## Stack的问题

看一下类结构其实就是很显然的了

```java
public
class Stack<E> extends Vector<E>
```

再看看文档呢?

A more complete and consistent set of LIFO stack operations is provided by the Deque interface and its implementations, which should be used in preference to this class. For example:
Deque<Integer> stack = new ArrayDeque<Integer>();

## ArrayDeque源码解读

