原理:

A thread-safe variant of java.util.ArrayList in which all mutative operations (add, set, and so on) are implemented by making a fresh copy of the underlying array.

This is ordinarily too costly, but may be more efficient than alternatives when traversal operations vastly outnumber mutations, and is useful when you cannot or don't want to synchronize traversals, yet need to preclude interference among concurrent threads. 

```java
public boolean add(E e) {
  	//先加锁
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
      	//复制
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}
```

```java
final void setArray(Object[] a) {
    array = a;
}
```

```java
public E get(int index) {
    return elementAt(getArray(), index);
}
```

```java
static <E> E elementAt(Object[] a, int index) {
    return (E) a[index];
}
```

