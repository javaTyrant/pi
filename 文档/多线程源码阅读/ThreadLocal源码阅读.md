## 1.类结构

```java
public class ThreadLocal<T>
```

**相关类**:

父子线程共享

```java
InheritableThreadLocal<T> extends ThreadLocal<T> 
```

```java
ThreadLocalRandom
```

## 2.核心变量

这个类只是一个门面类,调用set的时候,本质上是把他的内部类ThreadLocalMap保存到线程的threadLocals变量中

所以核心的变量在ThreadLocalMap里.

## 3.核心方法

### 3.1**set**

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

#### **3.1.1getMap**

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

#### **3.1.2createMap**:可以这个还是调用了内部类的构造器

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

### 3.2get

```java
public T get() {
  	//获取当前线程
    Thread t = Thread.currentThread();
  	//根据当前线程获取threadLocalMap,也就是线程的ThreadLocalMap变量
    ThreadLocalMap map = getMap(t);
  	//
    if (map != null) {
      	//根据当前的threadLocal获取Entry
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

map.getEntry(this)原理

```java
private Entry getEntry(ThreadLocal<?> key) {
  					//定位到索引
            int i = key.threadLocalHashCode & (table.length - 1);
  					//获取i位的entry
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }

```



## 4.内部类相关

### 4.1 ThreadLocalMap

#### 4.1.0 TLM的内部类Entry

包装的是ThreadLocal关联的value

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

### 4.1.0 get

```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    //获取当前的线程.
    Thread t = Thread.currentThread();
    //获取线程里的map.
    ThreadLocalMap map = getMap(t);
    //不为空的话
    if (map != null) {
        //获取这个threadLocal的entry.
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

### 4.1.0.1 getEntry

这个key是一个ThreadLocal.

```java
private Entry getEntry(ThreadLocal<?> key) {
    //很常见的写法了.
    int i = key.threadLocalHashCode & (table.length - 1);
    //获取i位置的.
    Entry e = table[i];
    //不为空 且 key相等. 
    if (e != null && e.get() == key)
        //
        return e;
    else
        // miss?
        return getEntryAfterMiss(key, i, e);
}
```

```java
//当'通过key取模运算的结果'无法获得准确的value时候(e.get() != key),
//其实也会通过while循环去遍历去获得该key(即ThreadLocal)的真实entry.
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
  //
  Entry[] tab = table;
  int len = tab.length;
  //这里是典型的线性地址探测法
  while (e != null) {
      ThreadLocal<?> k = e.get();
      if (k == key)
         return e;
       //如果
       if (k == null)
           expungeStaleEntry(i);
       else
           i = nextIndex(i, len);
           e = tab[i];
        }
          return null;
}
```



### 4.1.1 set

```java
public void set(T value) {
    //获取当前的线程
    Thread t = Thread.currentThread();
    //获取线程的map
    ThreadLocalMap map = getMap(t);
    //不为空
    if (map != null)
        map.set(this, value);
    else //创建map
        createMap(t, value);
}
```

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    //创建一个默认大小的Entry数组
    table = new Entry[INITIAL_CAPACITY];
    //HASH_INCREMENT = 0x61c88647;
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    //放入table. 
    table[i] = new Entry(firstKey, firstValue);
    //
    size = 1;
    //
    setThreshold(INITIAL_CAPACITY);
}
```

```java
//看看Entry的设计.weakReference的使用.
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    //ThreadLocal对应的value.
    Object value;
	//
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

```java
Reference(T referent, ReferenceQueue<? super T> queue) {
    this.referent = referent;
    //
    this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
}
```

```java
//
private void set(ThreadLocal<?> key, Object value) {
	// fast path是什么?
    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.
    Entry[] tab = table;
    //
    int len = tab.length;
    //
    int i = key.threadLocalHashCode & (len-1);
	//
    for (Entry e = tab[i];e != null;e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        //如果槽位的k == 设置的threadLocal
        if (k == key) {
            //设置value返回.
            e.value = value;
            return;
        }
		//如果k == null,说明被回收了?
        if (k == null) {
            //复杂的方法.
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    //
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```



### 4.1.2 remove

```java
/**
 * Remove the entry for key.
 */
private void remove(ThreadLocal<?> key) {
    //
    Entry[] tab = table;
    //
    int len = tab.length;
    //
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        //必须的校验.
        if (e.get() == key) {
            e.clear();
            expungeStaleEntry(i);
            return;
        }
    }
}
```

```java
public void clear() {
    this.referent = null;
}
```

```java
原理:

/**
 * Expunge a stale entry by rehashing any possibly colliding entries
 * lying between staleSlot and the next null slot.  This also expunges
 * any other stale entries encountered before the trailing null.  See
 * Knuth, Section 6.4
 *
 * @param staleSlot index of slot known to have null key
 * @return the index of the next null slot after staleSlot
 * (all between staleSlot and this slot will have been checked
 * for expunging).
 */
private int expungeStaleEntry(int staleSlot) {
    //临时变量保存下table
    Entry[] tab = table;
    //获取tab的长度.
    int len = tab.length;

    // expunge entry at staleSlot
    //把这个槽位的值设置成null
    tab[staleSlot].value = null;
    //tab位设置成null
    tab[staleSlot] = null;
    //长度--
    size--;
	//为什么需要rehash呢?
    // Rehash until we encounter null
    Entry e;
    //
    int i;
    //
    for (i = nextIndex(staleSlot, len);(e = tab[i]) != null;i = nextIndex(i, len)) {
        //
        ThreadLocal<?> k = e.get();
        //如果槽位的ThreadLocal为null了,被gc回收了
        if (k == null) {
            //value赋值成null
            e.value = null;
            //槽位设置为null
            tab[i] = null;
            //长度--
            size--;
        } else {
            //如果k不为null.
            int h = k.threadLocalHashCode & (len - 1);
            //什么情况会进入这个分支? 那么就说明当前的i是'不对'的(通过key计算得出的Entry[]下标可能获取不到准确的value).
            if (h != i) {
                //设空.
                tab[i] = null;
                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                //找到下一个为null的h.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                //把e设置.
                tab[h] = e;
            }
        }
    }
    //the index of the next null slot after staleSlot.
    return i;
}

private static int nextIndex(int i, int len) {
   //如果没有越界就返回下一个否则就返回第0个.
   return ((i + 1 < len) ? i + 1 : 0);
}
```

## 思考

**1.为什么要把ThreadLocalMap放到线程的属性呢?**

唯一的可能就是线程在任何想用的时候都可以直接get,如果放在其他地方,还要存在共享变量里.所以我们设计的时候也要真正的想明白,变量到底是哪个类的属性.

**2.Entry为什么要继承 WeakReference呢?**

**3.Netty的fastThreadLocal的优化思路呢?**

**4.为什么threadLocal会有内存泄露的风险呢?**

'ThreadLocalMap'是'Entry[]'去保存value,
而'Entry'继承了'WeakReference', 即Entry的key是弱引用.
当'ThreadLocal'没有外部强引用时, 那么会被GC回收, 然而该key对应的value却不会被回收.
若是当前线程一直不结束的话, 会存在一条强引用链, value也会一直累加导致内存泄露:

也就是value的生命周期是和线程一样的.

### 补充

```java
public class WeakReference<T> extends Reference<T> {
    /**
     * Creates a new weak reference that refers to the given object.  The new
     * reference is not registered with any queue.
     *
     * @param referent object the new weak reference will refer to
     */
    public WeakReference(T referent) {
        super(referent);
    }

    /**
     * Creates a new weak reference that refers to the given object and is
     * registered with the given queue.
     *
     * @param referent object the new weak reference will refer to
     * @param q the queue with which the reference is to be registered,
     *          or <tt>null</tt> if registration is not required
     */
    public WeakReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }

}
```

