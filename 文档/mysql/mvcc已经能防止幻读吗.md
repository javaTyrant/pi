## MVCC

#### **是什么**?[Multi-Version Concurrency Control](http://en.wikipedia.org/wiki/Multiversion_concurrency_control)

MVCC是在并发访问数据库时，通过对数据做多版本管理，避免因为写锁的阻塞而造成读数据的并发阻塞问题。

通俗的讲就是MVCC通过保存数据的历史版本，根据比较版本号来处理数据的是否显示，从而达到读取数据的时候不需要加锁就可以保证事务隔离性的效果

#### **实现原理**

核心知识点:

##### 1.事务版本号

每次事务开启前都会从数据库获得一个自增长的事务ID，可以从事务ID判断事务的执行先后顺序。

##### 2.表的隐藏列

**DB_TRX_ID:** 记录操作该数据事务的事务ID；

**DB_ROLL_PTR：**指向上一个版本数据在undo log 里的位置指针；

**DB_ROW_ID:** 隐藏ID ，当创建表没有合适的索引作为聚集索引时，会用该隐藏ID创建聚集索引;

##### 3.undo log

Undo log 主要用于记录数据被修改之前的日志，在表信息修改之前先会把数据拷贝到undo log 里，当事务进行回滚时可以通过undo log 里的日志进行数据还原。

##### undo log 的用途

（1）保证事务进行rollback时的原子性和一致性，当事务进行回滚的时候可以用undo log的数据进行恢复。

（2）用于MVCC快照读的数据，在MVCC多版本控制中，通过读取undo log的历史版本数据可以实现不同事务版本号都拥有自己独立的快照数据版本。

##### 4.read view

在innodb 中每个SQL语句执行前都会得到一个read_view。副本主要保存了当前数据库系统中正处于活跃（没有commit）的事务的ID号，其实简单的说这个副本中保存的是系统中当前不应该被本事务看到的其他事务id列表。

##### **Read view 的几个重要属性**

**trx_ids:** 当前系统活跃(未提交)事务版本号集合。

**low_limit_id:** 创建当前read view 时“当前系统最大**事务版本号**+1”。

**up_limit_id:** 创建当前read view 时“系统正处于**活跃事务**最小版本号”

**creator_trx_id:** 创建当前read view的事务版本号；

