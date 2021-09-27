# ZAB协议

## 1.原子广播协议

zk的核心是一个原子的广播系统,这个系统让所有的服务器处于同步的状态

### 1.1Zab协议消息广播有以下4个步骤组成:

- Leader发送PROPOSAL给集群中所有的节点。
- 节点在收到PROPOSAL之后，把PROPOSAL落盘,发送一个ACK给Leader。
- Leader在收到大多数节点的ACK之后，发送COMMIT给集群中所有的Follower节点。
- 如果存在Observer节点，Leader同时发送INFORM信息给Observer服务节点同步数据，Observer只接收Leader的INFORM消息同步数据，不参与Leader选举和事务提交。

## 2.ZAB基本特性

#### 1.ZAB协议需要确保那些已经在Leader服务器上提交的事务最终被所有服务器都提交

#### 2.ZAB协议需要确保丢弃那些只在Leader服务器上被提出的事务。



## 3.一次原子广播的全过程源码解析

zkMain.processZKCmd(zkMain.cl)->cliCmd.parse(args).exec()->zk.create()->ReplyHeader r = cnxn.submitRequest(h, record, response, null);

通过这个单元测试我们可以发现zkMain.processZKCmd(zkMain.cl)这个是主要执行创建节点命令的方法

#### 3.1processZKCmd

```java
 protected boolean processZKCmd(MyCommandOptions co) throws CliException, IOException, InterruptedException {
        String[] args = co.getArgArray();
        String cmd = co.getCommand();
        if (args.length < 1) {
            //打印用法
            usage();
            throw new MalformedCommandException("No command entered");
        }
        //校验命令是否是合法的
        if (!commandMap.containsKey(cmd)) {
            usage();
            throw new CommandNotFoundException("Command not found " + cmd);
        }
        boolean watch = false;
        LOG.debug("Processing {}", cmd);
        //如果是退出命令
        if (cmd.equals("quit")) {
            //关闭zk
            zk.close();
            //推出
            System.exit(exitCode);
            //redo命令
        } else if (cmd.equals("redo") && args.length >= 2) {
            //第二个参数是啥
            Integer i = Integer.decode(args[1]);
            //redo的范围校验
            if (commandCount <= i || i < 0) { // don't allow redoing this redo
                throw new MalformedCommandException("Command index out of range");
            }
            //解析命令
            cl.parseCommand(history.get(i));
            if (cl.getCommand().equals("redo")) {
                throw new MalformedCommandException("No redoing redos");
            }
            //放入历史
            history.put(commandCount, history.get(i));
            processCmd(cl);
        } else if (cmd.equals("history")) {
            //历史命令
            for (int i = commandCount - 10; i <= commandCount; ++i) {
                if (i < 0) {
                    continue;
                }
                System.out.println(i + " - " + history.get(i));
            }
        } else if (cmd.equals("printwatches")) {
            //打印观察者
            if (args.length == 1) {
                System.out.println("printwatches is " + (printWatches ? "on" : "off"));
            } else {
                printWatches = args[1].equals("on");
            }
        } else if (cmd.equals("connect")) {
            //连接命令
            if (args.length >= 2) {
                connectToZK(args[1]);
            } else {
                connectToZK(host);
            }
        }

        // Below commands all need a live connection
        // 下面的命令都需要存活的连接,所以校验一次连接
        if (zk == null || !zk.getState().isAlive()) {
            System.out.println("Not connected");
            return false;
        }

        // execute from commandMap,获取zk内部的命令
        CliCommand cliCmd = commandMapCli.get(cmd);
        if (cliCmd != null) {
            cliCmd.setZk(zk);
            //解析执行,具体的子类执行具体的命令
            watch = cliCmd.parse(args).exec();
        } else if (!commandMap.containsKey(cmd)) {
            usage();
        }
        return watch;
    }
```

可见涉及到事务的变更的命令,都会通过这个执行

```java
 watch = cliCmd.parse(args).exec();
```

这是一个抽象的类,具体解析的方法有子类来实现

```java
public abstract CliCommand parse(String[] cmdArgs) throws CliParseException;
```

我们先来看下创建节点的方法吧,其实这几个方法都是大同小异

```java
@Override
public boolean exec() throws CliException {
    boolean hasE = cl.hasOption("e");
    boolean hasS = cl.hasOption("s");
    boolean hasC = cl.hasOption("c");
    boolean hasT = cl.hasOption("t");
  	//参数校验
    if (hasC && (hasE || hasS)) {
        throw new MalformedCommandException("-c cannot be combined with -s or -e. Containers cannot be ephemeral or sequential.");
    }
    long ttl;
    try {
        ttl = hasT ? Long.parseLong(cl.getOptionValue("t")) : 0;
    } catch (NumberFormatException e) {
        throw new MalformedCommandException("-t argument must be a long value");
    }

    if (hasT && hasE) {
        throw new MalformedCommandException("TTLs cannot be used with Ephemeral znodes");
    }
    if (hasT && hasC) {
        throw new MalformedCommandException("TTLs cannot be used with Container znodes");
    }

    CreateMode flags;
    if (hasE && hasS) {
        flags = CreateMode.EPHEMERAL_SEQUENTIAL;
    } else if (hasE) {
        flags = CreateMode.EPHEMERAL;
    } else if (hasS) {
        flags = hasT ? CreateMode.PERSISTENT_SEQUENTIAL_WITH_TTL : CreateMode.PERSISTENT_SEQUENTIAL;
    } else if (hasC) {
        flags = CreateMode.CONTAINER;
    } else {
        flags = hasT ? CreateMode.PERSISTENT_WITH_TTL : CreateMode.PERSISTENT;
    }
    if (hasT) {
        try {
            EphemeralType.TTL.toEphemeralOwner(ttl);
        } catch (IllegalArgumentException e) {
            throw new MalformedCommandException(e.getMessage());
        }
    }

    String path = args[1];
    byte[] data = null;
    if (args.length > 2) {
        data = args[2].getBytes();
    }
    List<ACL> acl = ZooDefs.Ids.OPEN_ACL_UNSAFE;
    if (args.length > 3) {
        acl = AclParser.parse(args[3]);
    }
    try {
        //这个类只需要了解zk,不需要关键通信的细节
        String newPath = hasT
            ? zk.create(path, data, acl, flags, new Stat(), ttl)
            : zk.create(path, data, acl, flags);
        err.println("Created " + newPath);
    } catch (IllegalArgumentException ex) {
        throw new MalformedPathException(ex.getMessage());
    } catch (KeeperException.EphemeralOnLocalSessionException e) {
        err.println("Unable to create ephemeral node on a local session");
        throw new CliWrapperException(e);
    } catch (KeeperException.InvalidACLException ex) {
        err.println(ex.getMessage());
        throw new CliWrapperException(ex);
    } catch (KeeperException | InterruptedException ex) {
        throw new CliWrapperException(ex);
    }
    return true;
```

所以重点显而易见zk.create

所以zk这个类,我们必须要加以关注

```java
 public String create(
            final String path,
            byte[] data,
            List<ACL> acl,
            CreateMode createMode,
            Stat stat,
            long ttl) throws KeeperException, InterruptedException {
        final String clientPath = path;
        PathUtils.validatePath(clientPath, createMode.isSequential());
        EphemeralType.validateTTL(createMode, ttl);
        validateACL(acl);

        final String serverPath = prependChroot(clientPath);

        RequestHeader h = new RequestHeader();
        setCreateHeader(createMode, h);
        Create2Response response = new Create2Response();
        Record record = makeCreateRecord(createMode, serverPath, data, acl, ttl);
        //让cnxn提交请求,请求终于委托给cnxn了,ClientCnxn
        ReplyHeader r = cnxn.submitRequest(h, record, response, null);
        if (r.getErr() != 0) {
            throw KeeperException.create(KeeperException.Code.get(r.getErr()), clientPath);
        }
        if (stat != null) {
            DataTree.copyStat(response.getStat(), stat);
        }
        if (cnxn.chrootPath == null) {
            return response.getPath();
        } else {
            return response.getPath().substring(cnxn.chrootPath.length());
        }
    }
```

我们需要思考的是:ClientCnxn肯定有一个对应的serverCnxn,我们来找一下,因为请求最终都要到服务器嘛.

```java
public ReplyHeader submitRequest(
        RequestHeader h,
        Record request,
        Record response,
        WatchRegistration watchRegistration,
        WatchDeregistration watchDeregistration) throws InterruptedException {
    ReplyHeader r = new ReplyHeader();
    //这个方法肯定有处理的逻辑
    Packet packet = queuePacket(
            h, r, request, response, null,
            null, null, null,
            watchRegistration, watchDeregistration);
    //对packet加锁
    synchronized (packet) {
        if (requestTimeout > 0) {
            //有限等待
            // Wait for request completion with timeout
            waitForPacketFinish(r, packet);
        } else {
            //死等
            // Wait for request completion infinitely
            while (!packet.finished) {
                packet.wait();
            }
        }
    }
    if (r.getErr() == Code.REQUESTTIMEOUT.intValue()) {
        sendThread.cleanAndNotifyState();
    }
    return r;
```

问一下,这里面并没有提交请求的方法呀.那么我们看看有没有往队列里添加请求的方法,要么这个方法直接发送请求,要么把请求加入队列

然后另一个线程在处理发送的逻辑.

然后我们发现abstract void sendPacket(Packet p) throws IOException;这个方法也是个抽象的方法,不用想一个是netty的发送方法,一个是nio的发送方法.ClientCnxnSocketNIO,ClientCnxnSocketNetty.

具体的处理细节,我们单独开一个文档.我们先看servercnxn的接受方法吧,毕竟我们先关心大的流程,再关心细节.

我们从这个类开始找:NIOServerCnxn.最关键的是servercnxn到处理器链的这个调用过程.

```java
/**
 * The main loop for the thread selects() on the connections and
 * dispatches ready I/O work requests, then registers all pending
 * newly accepted connections and updates any interest ops on the
 * queue.
 */
public void run() {
    try {
        while (!stopped) {
            try {
                select();
                processAcceptedConnections();
                processInterestOpsUpdateRequests();
            } catch (RuntimeException e) {
                LOG.warn("Ignoring unexpected runtime exception", e);
            } catch (Exception e) {
                LOG.warn("Ignoring unexpected exception", e);
            }
        }

        // Close connections still pending on the selector. Any others
        // with in-flight work, let drain out of the work queue.
        for (SelectionKey key : selector.keys()) {
            NIOServerCnxn cnxn = (NIOServerCnxn) key.attachment();
            if (cnxn.isSelectable()) {
                cnxn.close(ServerCnxn.DisconnectReason.SERVER_SHUTDOWN);
            }
            cleanupSelectionKey(key);
        }
        SocketChannel accepted;
        while ((accepted = acceptedQueue.poll()) != null) {
            fastCloseSock(accepted);
        }
        updateQueue.clear();
    } finally {
        closeSelector();
        // This will wake up the accept thread and the other selector
        // threads, and tell the worker thread pool to begin shutdown.
        NIOServerCnxnFactory.this.stop();
        LOG.info("selector thread exitted run method");
    }
```

doIO肯定是读取io的数据,然后执行相关的逻辑处理.

```java
 void doIO(SelectionKey k) throws InterruptedException {
        try {
            if (!isSocketOpen()) {
                LOG.warn("trying to do i/o on a null socket for session:0x" + Long.toHexString(sessionId));

                return;
            }
            if (k.isReadable()) {
                int rc = sock.read(incomingBuffer);
                if (rc < 0) {
                    handleFailedRead();
                }
                if (incomingBuffer.remaining() == 0) {
                    boolean isPayload;
                    if (incomingBuffer == lenBuffer) { // start of next request
                        incomingBuffer.flip();
                        isPayload = readLength(k);
                        incomingBuffer.clear();
                    } else {
                        // continuation
                        isPayload = true;
                    }
                    if (isPayload) { // not the case for 4letterword
                      	//读
                        readPayload();
                    } else {
                        // four letter words take care
                        // need not do anything else
                        return;
                    }
                }
            }
            if (k.isWritable()) {
                handleWrite(k);

                if (!initialized && !getReadInterest() && !getWriteInterest()) {
                    throw new CloseRequestException("responded to info probe", DisconnectReason.INFO_PROBE);
                }
            }
        } catch (CancelledKeyException e) {
            LOG.warn("CancelledKeyException causing close of session 0x" + Long.toHexString(sessionId));

            LOG.debug("CancelledKeyException stack trace", e);

            close(DisconnectReason.CANCELLED_KEY_EXCEPTION);
        } catch (CloseRequestException e) {
            // expecting close to log session closure
            close();
        } catch (EndOfStreamException e) {
            LOG.warn(e.getMessage());
            // expecting close to log session closure
            close(e.getReason());
        } catch (ClientCnxnLimitException e) {
            // Common case exception, print at debug level
            ServerMetrics.getMetrics().CONNECTION_REJECTED.add(1);
            if (LOG.isDebugEnabled()) {
                LOG.debug("Exception causing close of session 0x" + Long.toHexString(sessionId)
                        + ": " + e.getMessage());
            }
            close(DisconnectReason.CLIENT_CNX_LIMIT);
        } catch (IOException e) {
            LOG.warn("Exception causing close of session 0x" + Long.toHexString(sessionId) + ": " + e.getMessage());

            LOG.debug("IOException stack trace", e);
            close(DisconnectReason.IO_EXCEPTION);
        }
    }

```

```java
  private void readPayload() throws IOException, InterruptedException, ClientCnxnLimitException {
        if (incomingBuffer.remaining() != 0) { // have we read length bytes?
            int rc = sock.read(incomingBuffer); // sock is non-blocking, so ok
            if (rc < 0) {
                handleFailedRead();
            }
        }

        if (incomingBuffer.remaining() == 0) { // have we read length bytes?
            incomingBuffer.flip();
            packetReceived(4 + incomingBuffer.remaining());
            if (!initialized) {
                readConnectRequest();
            } else {
                //读请求
                readRequest();
            }
            lenBuffer.clear();
            incomingBuffer = lenBuffer;
        }
    }
```

快到关键点了

```java
 private void readRequest() throws IOException {
        //交给zkserver处理数据包了
        zkServer.processPacket(this, incomingBuffer);
    }
```

```java
ZooKeeperServer:
 //处理请求
    public void processPacket(ServerCnxn cnxn, ByteBuffer incomingBuffer) throws IOException {
        // We have the request, now process and setup for next
        InputStream bais = new ByteBufferInputStream(incomingBuffer);
        BinaryInputArchive bia = BinaryInputArchive.getArchive(bais);
        RequestHeader h = new RequestHeader();
        h.deserialize(bia, "header");

        // Need to increase the outstanding request count first, otherwise
        // there might be a race condition that it enabled recv after
        // processing request and then disabled when check throttling.
        //
        // Be aware that we're actually checking the global outstanding
        // request before this request.
        //
        // It's fine if the IOException thrown before we decrease the count
        // in cnxn, since it will close the cnxn anyway.
        cnxn.incrOutstandingAndCheckThrottle(h);

        // Through the magic of byte buffers, txn will not be
        // pointing
        // to the start of the txn
        incomingBuffer = incomingBuffer.slice();
        if (h.getType() == OpCode.auth) {
            LOG.info("got auth packet " + cnxn.getRemoteSocketAddress());
            AuthPacket authPacket = new AuthPacket();
            ByteBufferInputStream.byteBuffer2Record(incomingBuffer, authPacket);
            String scheme = authPacket.getScheme();
            ServerAuthenticationProvider ap = ProviderRegistry.getServerProvider(scheme);
            Code authReturn = KeeperException.Code.AUTHFAILED;
            if (ap != null) {
                try {
                    // handleAuthentication may close the connection, to allow the client to choose
                    // a different server to connect to.
                    authReturn = ap.handleAuthentication(
                            new ServerAuthenticationProvider.ServerObjs(this, cnxn),
                            authPacket.getAuth());
                } catch (RuntimeException e) {
                    LOG.warn("Caught runtime exception from AuthenticationProvider: " + scheme + " due to " + e);
                    authReturn = KeeperException.Code.AUTHFAILED;
                }
            }
            if (authReturn == KeeperException.Code.OK) {
                LOG.debug("Authentication succeeded for scheme: {}", scheme);
                LOG.info("auth success " + cnxn.getRemoteSocketAddress());
                ReplyHeader rh = new ReplyHeader(h.getXid(), 0, KeeperException.Code.OK.intValue());
                cnxn.sendResponse(rh, null, null);
            } else {
                if (ap == null) {
                    LOG.warn("No authentication provider for scheme: " + scheme
                            + " has " + ProviderRegistry.listProviders());
                } else {
                    LOG.warn("Authentication failed for scheme: " + scheme);
                }
                // send a response...
                ReplyHeader rh = new ReplyHeader(h.getXid(), 0, KeeperException.Code.AUTHFAILED.intValue());
                cnxn.sendResponse(rh, null, null);
                // ... and close connection
                cnxn.sendBuffer(ServerCnxnFactory.closeConn);
                cnxn.disableRecv();
            }
            return;
        } else if (h.getType() == OpCode.sasl) {
            processSasl(incomingBuffer, cnxn, h);
        } else {
            if (shouldRequireClientSaslAuth() && !hasCnxSASLAuthenticated(cnxn)) {
                ReplyHeader replyHeader =
                  new ReplyHeader(h.getXid(), 0, Code.SESSIONCLOSEDREQUIRESASLAUTH.intValue());
                cnxn.sendResponse(replyHeader, null, "response");
                cnxn.sendCloseSession();
                cnxn.disableRecv();
            } else {
                Request si = new Request(cnxn, cnxn.getSessionId(), h.getXid(), h.getType(), incomingBuffer, cnxn.getAuthInfo());
                int length = incomingBuffer.limit();
                if (isLargeRequest(length)) {
                    // checkRequestSize will throw IOException if request is rejected
                    checkRequestSizeWhenMessageReceived(length);
                    si.setLargeRequestSize(length);
                }
                si.setOwner(ServerCnxn.me);
                // Always treat packet from the client as a possible
                // local request.
                setLocalSessionFlag(si);
                //提交请求
                submitRequest(si);
            }
        }
    
```

```java
public void submitRequest(Request si) {
    enqueueRequest(si);
}
```

```java
public void enqueueRequest(Request si) {
    if (requestThrottler == null) {
        synchronized (this) {
            try {
                // Since all requests are passed to the request
                // processor it should wait for setting up the request
                // processor chain. The state will be updated to RUNNING
                // after the setup.
                while (state == State.INITIAL) {
                    wait(1000);
                }
            } catch (InterruptedException e) {
                LOG.warn("Unexpected interruption", e);
            }
            if (requestThrottler == null) {
                throw new RuntimeException("Not started");
            }
        }
    }
  	//请求还是要加以控制点,不然突发的请求是会出事的.
    requestThrottler.submitRequest(si);
}

```

```java
private final LinkedBlockingQueue<Request> submittedRequests = new LinkedBlockingQueue<>();
public void submitRequest(Request request) {
    if (stopping) {
        LOG.debug("Shutdown in progress. Request cannot be processed");
        dropRequest(request);
    } else {
      	//最终请求加入到了这个队列里
        submittedRequests.add(request);
    }
}
```

然后RequestThrottler的run方法会取请求出来

```java
@Override
public void run() {
    try {
        while (true) {
            if (killed) {
                break;
            }
						//取出请求
            Request request = submittedRequests.take();
            if (Request.requestOfDeath == request) {
                break;
            }

            if (request.mustDrop()) {
                continue;
            }

            // Throttling is disabled when maxRequests = 0
            if (maxRequests > 0) {
                while (!killed) {
                    if (dropStaleRequests && request.isStale()) {
                        // Note: this will close the connection
                        dropRequest(request);
                        ServerMetrics.getMetrics().STALE_REQUESTS_DROPPED.add(1);
                        request = null;
                        break;
                    }
                    if (zks.getInProcess() < maxRequests) {
                        break;
                    }
                    throttleSleep(stallTime);
                }
            }

            if (killed) {
                break;
            }

            // A dropped stale request will be null
            if (request != null) {
                if (request.isStale()) {
                    ServerMetrics.getMetrics().STALE_REQUESTS.add(1);
                }
                //泪目啊,这么久终于逮到你了
                zks.submitRequestNow(request);
            }
        }
    } catch (InterruptedException e) {
        LOG.error("Unexpected interruption", e);
    }
    int dropped = drainQueue();
    LOG.info("RequestThrottler shutdown. Dropped {} requests", dropped);
}
```

```java
public void submitRequestNow(Request si) {
    if (firstProcessor == null) {
        synchronized (this) {
            try {
                // Since all requests are passed to the request
                // processor it should wait for setting up the request
                // processor chain. The state will be updated to RUNNING
                // after the setup.
                while (state == State.INITIAL) {
                    wait(1000);
                }
            } catch (InterruptedException e) {
                LOG.warn("Unexpected interruption", e);
            }
            if (firstProcessor == null || state != State.RUNNING) {
                throw new RuntimeException("Not started");
            }
        }
    }
    try {
        touch(si.cnxn);
        boolean validpacket = Request.isValid(si.type);
        if (validpacket) {
            //PrepRequestProcessor处理,提交给请求链了
            firstProcessor.processRequest(si);
            if (si.cnxn != null) {
                //requestsInProcess自增1
                incInProcess();
            }
        } else {
            LOG.warn("Received packet at server of unknown type " + si.type);
            // Update request accounting/throttling limits
            requestFinished(si);
            new UnimplementedRequestProcessor().processRequest(si);
        }
    } catch (MissingSessionException e) {
        if (LOG.isDebugEnabled()) {
            LOG.debug("Dropping request: " + e.getMessage());
        }
        // Update request accounting/throttling limits
        requestFinished(si);
    } catch (RequestProcessorException e) {
        LOG.error("Unable to process request:" + e.getMessage(), e);
        // Update request accounting/throttling limits
        requestFinished(si);
    }
```

# 选举算法



