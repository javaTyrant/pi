### 对于普通同步方法加锁时，锁是当前实例对象。

### 对于静态同步方法加锁时，锁是当前类的Class对象。

### 对于同步方法块加锁时，锁是Synchonized括号里配置的对象。

**Synchronized的实现方式：**
Synchonized是基于进入和退出Monitor对象来实现方法同步和代码块同步，但两者的实现细节不一样。Synchronized 用在方法上时，在字节码中是通过方法的 ACC_SYNCHRONIZED 标志来实现的。而代码块同步则是使用monitorenter和monitorexit指令实现的。
monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处，JVM要保证每个monitorenter必须有对应的monitorexit与之配对。任何对象都有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态。线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor的所有权，即尝试获得对象的锁，当获得对象的monitor以后，monitor内部的计数器就会自增（初始为0），当同一个线程再次获得monitor的时候，计数器会再次自增。当同一个线程执行monitorexit指令的时候，计数器会进行自减，当计数器为0的时候，monitor就会被释放，其他线程便可以获得monitor。

**Synchronized的优化**
Java SE 1.6为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”，在Java SE 1.6中，锁一共有4种状态，级别从低到高依次是：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态。

**偏向锁**
当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁。如果测试成功，表示线程已经获得了锁。如果测试失败，则需要再测试一下Mark Word中偏向锁的标识是否设置成1（表示当前是偏向锁），如果没有设置，则使用CAS竞争锁；如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程。

**轻量级锁**
线程在执行同步块之前，JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中。然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。

**重量级锁**
重量级锁是依赖对象内部的monitor锁来实现。当系统检查到锁是重量级锁之后，会把等待想要获得锁的线程进行阻塞，被阻塞的线程不会消耗cup。但是阻塞或者唤醒一个线程时，都需要操作系统来帮忙，需要从用户态转换到内核态，而转换状态是需要消耗很多时间。

#### 锁升级的过程

锁升级的方向是：无锁——>偏向锁——>轻量级锁——>重量级锁，并且膨胀方向不可逆。

### Happen before

1. 同一个线程中的，前面的操作 happen-before 后续的操作。（即单线程内按代码顺序执行。但是，在不影响在单线程环境执行结果的前提下，编译器和处理器可以进行重排序，这是合法的。换句话说，这一是规则无法保证编译重排和指令重排）。
2. 监视器上的解锁操作 happen-before 其后续的加锁操作。（Synchronized 规则）
3. 对volatile变量的写操作 happen-before 后续的读操作。（volatile 规则）
4. 线程的start() 方法 happen-before 该线程所有的后续操作。（线程启动规则）
5. 线程所有的操作 happen-before 其他线程在该线程上调用 join 返回成功后的操作。
6. 如果 a happen-before b，b happen-before c，则a happen-before c（传递性）。