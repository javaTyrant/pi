# 好难啊这个类!!!!

## 1.核心变量



## 2.核心方法

fh的含义:

```java
static final int MOVED     = -1; // hash for forwarding nodes
static final int TREEBIN   = -2; // hash for roots of trees
static final int RESERVED  = -3; // hash for transient reservations
```

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
  	//key和value都不允许为空
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
  	//死循环
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
      	//需要初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
      	//该索引位置为null
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
          	//这边没有加锁,所以要cas确保只有一个节点能放进去
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
      	//MOVED:节点在扩容
        else if ((fh = f.hash) == MOVED)
          	//这个方法要吃透
            tab = helpTransfer(tab, f);
        else {
          	//保存旧的值
            V oldVal = null;
          	//对首节点加锁
            synchronized (f) {
              	//再次确保i的元素和f是相等的,为什么要二次判断了,防止其他的线程修改map吗
                if (tabAt(tab, i) == f) {
                  	//如果fh大于等于0
                    if (fh >= 0) {
                        binCount = 1;
                      	//死循环
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                          	//map中已经有这个key了,用之前的还是新value呢
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                          	//链表
                            Node<K,V> pred = e;
                          	//找到最后一个节点
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }//fh >= 0
                  	//红黑树
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
```



get方法

```java
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }

```

helpTransfer

总结下步骤吧:

[扩容资料](https://www.jianshu.com/p/39b747c99d32)

```java
    final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
      	//两个临时变量,sizeCtl
        Node<K,V>[] nextTab; int sc;
      	//tab不为空且节点的类型要是ForwardingNode且nextTab不为空
        if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
          	//根据tab的长度resizeStamp,rs的作用是什么呢
            int rs = resizeStamp(tab.length);
            while (nextTab == nextTable && table == tab &&
                   (sc = sizeCtl) < 0) {
              	//what?
              	// 如果 sizeCtl 无符号右移  16 不等于 rs （ sc前 16 位如果不等于标识符，则标识符变化了）
                // 或者 sizeCtl == rs + 1  （扩容结束了，不再有线程进行扩容）（默认第一个线程设置 sc ==rs 左移 16 位 + 2，
                // 当第一个线程结束扩容了，就会将 sc 减一。这个时候，sc 就等于 rs + 1）
                // 或者 sizeCtl == rs + 65535  （如果达到最大帮助线程的数量，即 65535）
                // 或者转移下标正在调整 （扩容结束）
                // 结束循环，返回 table
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
              	//what?
              	//如果以上都不是, 将 sizeCtl + 1, （表示增加了一个线程帮助其扩容）
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
```

resizeStamp:干啥的,为啥要这样干

> numberOfLeadingZeros（n）返回的是n的二进制标识的从高位开始到第一个非0的数字的之间0的个数，比如numberOfLeadingZeros（8）返回的就是28 ，因为0000 0000 0000 0000 0000 0000 0000 1000在1前面有28个0 
> RESIZE_STAMP_BITS 的值是16，1 << (RESIZE_STAMP_BITS - 1)就是将1左移位15位，0000 0000 0000 0000 1000 0000 0000 0000 
> 然后将两个数字再按位或，将相当于 将移位后的 两个数相加。
> 举个例子 
> 8的二进制表示是： 0000 0000 0000 0000 0000 0000 0000 1000 = 8
> 7的二进制表示是： 0000 0000 0000 0000 0000 0000 0000 0111 = 7
> 按位或的结果就是：0000 0000 0000 0000 0000 0000 0000 1111 =15
> 相当于 8 + 7 =15 
> 为什么会出现这种效果呢？因为8是2的整数次幂，也就是说8的二进制表示只会在某个高位上是1，其余地位都是0，所以在按位或的时候，低位表示的全是7的位值，所以出现了这种效果。

```java
//Returns the stamp bits for resizing a table of size n.
//Must be negative when shifted left by RESIZE_STAMP_SHIFT.
static final int resizeStamp(int n) {
  	return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
}

```



transfer:

精华啊!!!!太难了!!!硬着头皮上

> 简单点说： 比如有个数组长度32的，然后扩容的时候变成长度64，我们把扩容操作当做一个队列来看，每个线程来帮助扩容，其实就是去队列领取对应的长度。那么假设现在有4个线程来访问，第一个线程访问，看到要扩容，则领取队列中领取数组的0-15的数组数据，这里我们有个标志位a(从后往前减)，a=64-16=47,线程2过来，发现这个标志位为正在扩容，则去领取16-31的数组长度进行扩容 a=31,线程3来，一样，领取32-47的数组长度a=15，线程4来了，领取47-63的数组长度a=0。这就是帮助扩容。然后线程5来了，看到a=0,则这个时候，这个标志a其实已经移动到队列末尾了，也就没有要扩容的数据，所以就不用帮助扩容。大概意思就是这样

transferIndex:The next table index (plus one) to split while resizing.

sizeCtl:

```java
/**
 * Table initialization and resizing control.  When negative, the
 * table is being initialized or resized: -1 for initialization,
 * else -(1 + the number of active resizing threads).  Otherwise,
 * when table is null, holds the initial table size to use upon
 * creation, or 0 for default. After initialization, holds the
 * next element count value upon which to resize the table.
 */
```

nextTab:

```java
The next table to use; non-null only while resizing.
```

```java
//把tab往nextTab转移的过程  
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
     		//构建两个临时变量一个保存tab的长度,一个是stride,每个线程负责的大小
        int n = tab.length, stride;
     		//一个线程最少负责MIN_TRANSFER_STRIDE 16
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
     		//intiating nextTab什么时候为空呢?
        if (nextTab == null) {            
            try {
              	//构建一个二倍大小的新数组
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
              	//赋值给空数组
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
              	//sizeCto = max
                sizeCtl = Integer.MAX_VALUE;
              	//如果有异常直接return
                return;
            }
          	//nextTable赋值
            nextTable = nextTab;
          	//n旧的tab的长度
            transferIndex = n;
        }
     		//
        int nextn = nextTab.length;
  			//
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
  			//进展
        boolean advance = true;
  			//结束了吗
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) {
          	//f临时节点,fh
            Node<K,V> f; int fh;
          	//advance控制入口
            while (advance) {
                int nextIndex, nextBound;
              	//当i是0的时候,--i是几
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
              	//什么意思哦?
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
          	//
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
              	//完成扩容
                if (finishing) {
                    nextTable = null;
                  	//
                    table = nextTab;
                  	//why?
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
              	//锁住f
                synchronized (f) {
                  	//判断是链表?
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                      	//红黑树
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }//
    }
```



## 3.设计思路

### get为什么不需要加锁

### put时在哪加锁

## 4.内部类:

### ForwardingNode



## 5.思考

