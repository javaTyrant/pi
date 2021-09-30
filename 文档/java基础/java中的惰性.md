## 类结构

```java
class ThreadLocal<T>
```

还有个:父子线程不是线程隔离的

```java
class InheritableThreadLocal<T> extends ThreadLocal<T> 
```

**注意**

Thread这个类里有两个变量,跟这个类息息相关

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;

/*
 * InheritableThreadLocal values pertaining to this thread. This map is
 * maintained by the InheritableThreadLocal class.
 */
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

## 内部类

注意下两个类的区别

```java
static class ThreadLocalMap
```

### 核心变量

### 内部类

1.Entry注意这个Entry是WR

```java
static class Entry extends WeakReference<ThreadLocal<?>>
```

```java

```



```java
static final class SuppliedThreadLocal
```

## 核心变量



## 核心方法

毫无疑问,get,set方法是最重要的两个方法

```java
public void set(T value) {
  	//获取当前线程
    Thread t = Thread.currentThread();
  	//获取map
    ThreadLocalMap map = getMap(t);
    if (map != null)
      	//不为空set
        map.set(this, value);
    else
      	//为空初始化map再set
        createMap(t, value);
}
```

getMap:获取线程里的变量,就这么简单

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

createMap:还是修改thread的变量

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

new ThreadLocalMap:可见threadLocalMap才是这个类的核心

```java
  ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
}
```



```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```



### 答疑

#### 为什么Entry extends WeakReference<ThreadLocal<?>>

因为这个ThreadLocalMap并非用户定义，**用户只是new了一个ThreadLocal对象，所以当用户定义的ThreadLocal对象不再使用之后，ThreadLocal对象及其指向的T对象都应该可以被回收**。可是由于很多线程中的ThreadLocalMap对象中保存了ThreadLocal对象和T对象的Entry（键值对）中的ThreadLocal对象回收可能会受到阻碍。因此这里看代码可以看到ThreadLocal对象作为key的时候首先被WeakReference包裹了一下的.