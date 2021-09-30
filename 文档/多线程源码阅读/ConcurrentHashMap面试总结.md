- **ConcurrentHashMap的实现原理**
  - **ConcurrentHashMap1.7和1.8的区别？**
  - **ConcurrentHashMap使用什么技术来保证线程安全**
- **ConcurrentHashMap的put()方法**
  - **ConcurrentHashmap 不支持 key 或者 value 为 null 的原因？**
  - **put()方法如何实现线程安全呢？**
- **ConcurrentHashMap扩容机制**
- **ConcurrentHashMap的get方法是否要加锁，为什么？**
- **其他问题**
  - **为什么使用ConcurrentHashMap**
  - **ConcurrentHashMap迭代器是强一致性还是弱一致性？HashMap呢？**
  - **JDK1.7与JDK1.8中ConcurrentHashMap的区别**

**ConcurrentHashMap保证线程安全的方案是：**

- **JDK1.8：synchronized+CAS+HashEntry+红黑树；**
- **JDK1.7：ReentrantLock+Segment+HashEntry。**

**put过程:**

1.先校验kv不能为空

2.死循环处理:判断要不要初始化

3.如果当前的bucket为空,cas插入

4.是否在扩容:(fh = f.hash) == MOVED

5.对首节点加锁

因为`concurrenthashmap`它们是用于多线程的，并发的 ，如果`map.get(key)`得到了null，不能判断到底是映射的value是null,还是因为没有找到对应的key而为空，