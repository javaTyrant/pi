## 1.PrepRequestProcessor

。请求预处理器。在Zookeeper中，那些会改变服务器状态的请求称为事务请求（创建节点、更新数据、删除节点、创建会话等），PrepRequestProcessor能够识别出当前客户端请求是否是事务请求。对于事务请求，PrepRequestProcessor处理器会对其进行一系列预处理，如创建请求事务头、事务体、会话检查、ACL检查和版本检查等。

## 2.ProposalRequestProcessor

事务投票处理器。Leader服务器事务处理流程的发起者，对于非事务性请求，ProposalRequestProcessor会直接将请求转发到CommitProcessor处理器，不再做任何处理，而对于事务性请求，除了将请求转发到CommitProcessor外，还会根据请求类型创建对应的Proposal提议，并发送给所有的Follower服务器来发起一次集群内的事务投票。同时，ProposalRequestProcessor还会将事务请求交付给SyncRequestProcessor进行事务日志的记录。

## 3.SyncRequestProcessor

事务日志记录处理器。用来将事务请求记录到事务日志文件中，同时会触发Zookeeper进行数据快照。

## 4.AckRequestProcessor

负责在SyncRequestProcessor完成事务日志记录后，向Proposal的投票收集器发送ACK反馈，以通知投票收集器当前服务器已经完成了对该Proposal的事务日志记录。

## 5.CommitProcessor

事务提交处理器。对于非事务请求，该处理器会直接将其交付给下一级处理器处理；对于事务请求，其会等待集群内针对Proposal的投票直到该Proposal可被提交，利用CommitProcessor，每个服务器都可以很好地控制对事务请求的顺序处理。

incoming committed requests和 locally submitted requests.

This RequestProcessor matches the incoming committed requests with the locally submitted requests. The trick is that locally submitted requests that change the state of the system will come back as incoming committed requests, so we need to match them up. Instead of just waiting for the committed requests, we process the uncommitted requests that belong to other sessions.
The CommitProcessor is multi-threaded. Communication between threads is handled via queues, atomics, and wait/notifyAll synchronized on the processor. The CommitProcessor acts as a gateway for allowing requests to continue with the remainder of the processing pipeline. It will allow many read requests but only a single write request to be in flight simultaneously, thus ensuring that write requests are processed in transaction id order.

- 1 commit processor main thread, which watches the request queues and assigns requests to worker threads based on their sessionId so that read and write requests for a particular session are always assigned to the same thread (and hence are guaranteed to run in order). - 0-N worker threads, which run the rest of the request processor pipeline on the requests. If configured with 0 worker threads, the primary commit processor thread runs the pipeline directly.
  Typical (default) thread counts are: on a 32 core machine, 1 commit processor thread and 32 worker threads.
  Multi-threading constraints: - Each session's requests must be processed in order. - Write requests must be processed in zxid order - Must ensure no race condition between writes in one session that would trigger a watch being set by a read request in another session
  The current implementation solves the third constraint by simply allowing no read requests to be processed in parallel with write requests.

## 6.ToBeAppliedRequestProcessor

该处理器有一个toBeApplied队列，用来存储那些已经被CommitProcessor处理过的可被提交的Proposal。其会将这些请求交付给FinalRequestProcessor处理器处理，待其处理完后，再将其从toBeApplied队列中移除。

## 7.FinalRequestProcessor

## 关注点:不同角色的处理器链路

处理器的类结构

```
extends ZooKeeperCriticalThread implements RequestProcessor 
```

## PrepRequestProcessor:源码解读

核心变量

```java
//
private static final Logger LOG = LoggerFactory.getLogger(PrepRequestProcessor.class);
//已提交的请求
LinkedBlockingQueue<Request> submittedRequests = new LinkedBlockingQueue<>();
//下一个处理器
private final RequestProcessor nextProcessor;
//当前服务器的信息
ZooKeeperServer zks;
```

这个类主要两个关注点:1.run方法,2.processRequest

run方法的调用点:

```java
//设置请求链的顺序
protected void setupRequestProcessors() {
    //PrepRequestProcessor->SyncRequestProcessor->RequestProcessor
    RequestProcessor finalProcessor = new FinalRequestProcessor(this);
    RequestProcessor syncProcessor = new SyncRequestProcessor(this, finalProcessor);
    ((SyncRequestProcessor) syncProcessor).start();
    firstProcessor = new PrepRequestProcessor(this, syncProcessor);
    ((PrepRequestProcessor) firstProcessor).start();
}
```

这个处理器是一直工作的,任务的添加和任务的处理是解耦的.简单然而很妙.

```java
@Override
public void run() {
    try {
        //死循环处理
        while (true) {
            //
            ServerMetrics.getMetrics().PREP_PROCESSOR_QUEUE_SIZE.add(submittedRequests.size());
            //不断的重队列中取出请求进行处理
            Request request = submittedRequests.take();
            ServerMetrics.getMetrics().PREP_PROCESSOR_QUEUE_TIME
                    .add(Time.currentElapsedTime() - request.prepQueueStartTime);
            long traceMask = ZooTrace.CLIENT_REQUEST_TRACE_MASK;
            if (request.type == OpCode.ping) {
                traceMask = ZooTrace.CLIENT_PING_TRACE_MASK;
            }
            if (LOG.isTraceEnabled()) {
                ZooTrace.logRequest(LOG, traceMask, 'P', request, "");
            }
            if (Request.requestOfDeath == request) {
                break;
            }
            request.prepStartTime = Time.currentElapsedTime();
            //预处理
            pRequest(request);
        }
    } catch (RequestProcessorException e) {
        if (e.getCause() instanceof XidRolloverException) {
            LOG.info(e.getCause().getMessage());
        }
        handleException(this.getName(), e);
    } catch (Exception e) {
        handleException(this.getName(), e);
    }
    LOG.info("PrepRequestProcessor exited loop!");
}
```

```java
public void processRequest(Request request) {
    //这里会添加请求
    request.prepQueueStartTime = Time.currentElapsedTime();
    submittedRequests.add(request);
    ServerMetrics.getMetrics().PREP_PROCESSOR_QUEUED.add(1);
}
```

pRequest:预处理的过程



```java
protected void pRequest(Request request) throws RequestProcessorException {
    // LOG.info("Prep>>> cxid = " + request.cxid + " type = " +
    // request.type + " id = 0x" + Long.toHexString(request.sessionId));
    request.setHdr(null);
    request.setTxn(null);

    try {
        //请求的类型
        switch (request.type) {
            //如果是创建操作
            case OpCode.createContainer:
            case OpCode.create:
            case OpCode.create2:
                //创建一个创建请求类
                CreateRequest create2Request = new CreateRequest();
                pRequest2Txn(request.type, zks.getNextZxid(), request, create2Request, true);
                break;
            case OpCode.createTTL:
                CreateTTLRequest createTtlRequest = new CreateTTLRequest();
                pRequest2Txn(request.type, zks.getNextZxid(), request, createTtlRequest, true);
                break;
            case OpCode.deleteContainer:
            case OpCode.delete:
                DeleteRequest deleteRequest = new DeleteRequest();
                pRequest2Txn(request.type, zks.getNextZxid(), request, deleteRequest, true);
                break;
            case OpCode.setData:
                SetDataRequest setDataRequest = new SetDataRequest();
                pRequest2Txn(request.type, zks.getNextZxid(), request, setDataRequest, true);
                break;
            case OpCode.reconfig:
                ReconfigRequest reconfigRequest = new ReconfigRequest();
                ByteBufferInputStream.byteBuffer2Record(request.request, reconfigRequest);
                pRequest2Txn(request.type, zks.getNextZxid(), request, reconfigRequest, true);
                break;
            case OpCode.setACL:
                SetACLRequest setAclRequest = new SetACLRequest();
                pRequest2Txn(request.type, zks.getNextZxid(), request, setAclRequest, true);
                break;
            case OpCode.check:
                CheckVersionRequest checkRequest = new CheckVersionRequest();
                pRequest2Txn(request.type, zks.getNextZxid(), request, checkRequest, true);
                break;
            case OpCode.multi:
                MultiOperationRecord multiRequest = new MultiOperationRecord();
                try {
                    ByteBufferInputStream.byteBuffer2Record(request.request, multiRequest);
                } catch (IOException e) {
                    request.setHdr(new TxnHeader(request.sessionId, request.cxid, zks.getNextZxid(), Time.currentWallTime(), OpCode.multi));
                    throw e;
                }
                List<Txn> txns = new ArrayList<>();
                //Each op in a multi-op must have the same zxid!
                long zxid = zks.getNextZxid();
                KeeperException ke = null;

                //Store off current pending change records in case we need to rollback
                //存储当前挂起的更改记录，以防我们需要回滚
                Map<String, ChangeRecord> pendingChanges = getPendingChanges(multiRequest);

                for (Op op : multiRequest) {
                    Record subrequest = op.toRequestRecord();
                    int type;
                    Record txn;

                    /* If we've already failed one of the ops, don't bother
                     * trying the rest as we know it's going to fail and it
                     * would be confusing in the logfiles.
                     */
                    if (ke != null) {
                        type = OpCode.error;
                        txn = new ErrorTxn(Code.RUNTIMEINCONSISTENCY.intValue());
                    } else {
                        /* Prep the request and convert to a Txn */
                        try {
                            pRequest2Txn(op.getType(), zxid, request, subrequest, false);
                            type = request.getHdr().getType();
                            txn = request.getTxn();
                        } catch (KeeperException e) {
                            ke = e;
                            type = OpCode.error;
                            txn = new ErrorTxn(e.code().intValue());

                            if (e.code().intValue() > Code.APIERROR.intValue()) {
                                LOG.info("Got user-level KeeperException when processing {} aborting"
                                                + " remaining multi ops. Error Path:{} Error:{}",
                                        request.toString(),
                                        e.getPath(),
                                        e.getMessage());
                            }

                            request.setException(e);

                            /* Rollback change records from failed multi-op */
                            //回滚改变,如果出现异常的话
                            rollbackPendingChanges(zxid, pendingChanges);
                        }
                    }

                    // TODO: I don't want to have to serialize it here and then
                    //       immediately deserialize in next processor. But I'm
                    //       not sure how else to get the txn stored into our list.
                    try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
                        BinaryOutputArchive boa = BinaryOutputArchive.getArchive(baos);
                        txn.serialize(boa, "request");
                        ByteBuffer bb = ByteBuffer.wrap(baos.toByteArray());
                        txns.add(new Txn(type, bb.array()));
                    }
                }

                request.setHdr(new TxnHeader(request.sessionId, request.cxid, zxid, Time.currentWallTime(), request.type));
                request.setTxn(new MultiTxn(txns));

                break;

            //create/close session don't require request record
            case OpCode.createSession:
            case OpCode.closeSession:
                if (!request.isLocalSession()) {
                    pRequest2Txn(request.type, zks.getNextZxid(), request, null, true);
                }
                break;

            //All the rest don't need to create a Txn - just verify session
            case OpCode.sync:
            case OpCode.exists:
            case OpCode.getData:
            case OpCode.getACL:
            case OpCode.getChildren:
            case OpCode.getAllChildrenNumber:
            case OpCode.getChildren2:
            case OpCode.ping:
            case OpCode.setWatches:
            case OpCode.checkWatches:
            case OpCode.removeWatches:
            case OpCode.getEphemerals:
            case OpCode.multiRead:
                zks.sessionTracker.checkSession(request.sessionId, request.getOwner());
                break;
            default:
                LOG.warn("unknown type " + request.type);
                break;
        }
    } catch (KeeperException e) {
        if (request.getHdr() != null) {
            request.getHdr().setType(OpCode.error);
            request.setTxn(new ErrorTxn(e.code().intValue()));
        }

        if (e.code().intValue() > Code.APIERROR.intValue()) {
            LOG.info("Got user-level KeeperException when processing {} Error Path:{} Error:{}",
                    request.toString(),
                    e.getPath(),
                    e.getMessage());
        }
        request.setException(e);
    } catch (Exception e) {
        // log at error level as we are returning a marshalling
        // error to the user
        LOG.error("Failed to process " + request, e);

        StringBuilder sb = new StringBuilder();
        ByteBuffer bb = request.request;
        if (bb != null) {
            bb.rewind();
            while (bb.hasRemaining()) {
                sb.append(Integer.toHexString(bb.get() & 0xff));
            }
        } else {
            sb.append("request buffer is null");
        }

        LOG.error("Dumping request buffer: 0x" + sb.toString());
        if (request.getHdr() != null) {
            request.getHdr().setType(OpCode.error);
            request.setTxn(new ErrorTxn(Code.MARSHALLINGERROR.intValue()));
        }
    }
    request.zxid = zks.getZxid();
    ServerMetrics.getMetrics().PREP_PROCESS_TIME.add(Time.currentElapsedTime() - request.prepStartTime);
    //请求转移给下一个处理器
    nextProcessor.processRequest(request);
}
```

