## 类结构

```java
class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

## 核心变量

稍微的注意下,哪些是构造器可以改变的,哪些是内部属性,通过普通的方法修改的.

```java
//最核心的变量,最底层的数据结构,这个数组既可以放链表又可以放红黑树,所以链表和红黑树这两个类必须要继承Node
1.transient Node<K,V>[] table;
2.size
3.transient Set<Map.Entry<K,V>> entrySet;
4.int threshold;
5.transient int modCount;
6.final float loadFactor;

7.static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
8.static final int MAXIMUM_CAPACITY = 1 << 30;
9.static final float DEFAULT_LOAD_FACTOR = 0.75f;
10.static final int TREEIFY_THRESHOLD = 8;
11.static final int UNTREEIFY_THRESHOLD = 6;
12.static final int MIN_TREEIFY_CAPACITY = 64;
```

## 核心方法

最核心的肯定是put和get方法

然后就是resize方法

入口:需要两个参数一个key,一个value

```java
   public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

putVal方法,参数多一些

```java
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
   			//临时的容器tab,临时的node节点,两个临时变量
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
          	//tab为空或者长度为0,resize,返回resize后的长度
            n = (tab = resize()).length;
   			//此时tab已经不为空了
   			//如果判断索引处是否已经有数据了
        if ((p = tab[i = (n - 1) & hash]) == null)
          	//为空直接new个node放入i
          	//newNode就是简单的调用了node构造器的方法
            tab[i] = newNode(hash, key, value, null);
        else {
          	//索引处已经有值了,两种情况,链表或者红黑树
            Node<K,V> e; K k;//构造两个临时变量
          	//p:原来的节点,如果插入的key和原来的key是相同的
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
              //保存的node  
              e = p;
          	//红黑树
            else if (p instanceof TreeNode)
              	//这边的实现有点复杂,干
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
              	//链表操作,链表需要循环遍历,
                for (int binCount = 0; ; ++binCount) {
                  	//找到尾节点,经典的链表操作
                    if ((e = p.next) == null) {
                      	//插入
                        p.next = newNode(hash, key, value, null);
                      	//链表过长需要treeify,树化
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                  	//移动指针
                    p = e;
                }
            }
          	//
            if (e != null) { // existing mapping for key
              	//p节点的value,旧的value
                V oldValue = e.value;
              	//不存在才替换策略
                if (!onlyIfAbsent || oldValue == null)
                  	//新value替换老value
                    e.value = value;
              	//节点插入后操作,钩子方法
              	//让实现类去做额外的操作,唯一的LinkedHashMap
                afterNodeAccess(e);
              	//返回老的value
                return oldValue;
            }
        }
   			//e为空才会走到下面的逻辑,e为空说明了新key是不存在的
   			//e为空链表的找到尾结点那步会把e设置成null,红黑树的逻辑肯定也是一样
   			//修改次数+1
        ++modCount;
   			//检查需不需要resize
        if (++size > threshold)
            resize();
				//钩子方法
        afterNodeInsertion(evict);
   			//返回空
        return null;
    }
```

**resize**方法

这个扩容的对各种临界值的判断是不是值得我们去学习呢?

1.resize的时候先处理newCap和newThr.

2.然后再处理元素的转移.

**链表转移算法**

> 我们首先准备了两个链表 `lo` 和 `hi`, 然后我们顺序遍历该存储桶上的链表的每个节点, 如果 `(e.hash & oldCap) == 0`, 我们就将节点放入`lo`链表, 否则, 放入`hi`链表.  这样设计的目的何在呢?为什么要需要两个呢

上面我们已经弄懂了链表拆分的代码, 但是这个拆分条件看上去很奇怪, 这里我们来稍微解释一下:

首先我们要明确三点:

1. oldCap一定是2的整数次幂, 这里假设是2^m
2. newCap是oldCap的两倍, 则会是2^(m+1)
3. hash对数组大小取模`(n - 1) & hash` 其实就是取hash的低`m`位

  结论:

```java
//返回resize后的node
final Node<K,V>[] resize() {
  	//保存原table
    Node<K,V>[] oldTab = table;
  	//计算原table的大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
  	//原来的门槛
    int oldThr = threshold;
  	//新的table容量,新的门槛
    int newCap, newThr = 0;
  	//思考下这几个ifelse流程的顺序
  	//旧的容量大于0
    if (oldCap > 0) {
      	//大于了边界值,并没有报错
        if (oldCap >= MAXIMUM_CAPACITY) {
          	//
            threshold = Integer.MAX_VALUE;
						//返回旧的大小,那么新增的数据是覆盖还是丢失呢
            return oldTab;
        }
      	//正常的扩容,检查扩容后会不会大于阈值
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
          	//可以扩容一倍的时候,扩容一倍
          	//门槛也要扩容,newCap在if里double了
            newThr = oldThr << 1; // double threshold
    }
  	//oldCap=0时判断oldThr
    else if (oldThr > 0) // initial capacity was placed in threshold
      	//newCap = 老门槛
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
      	//oldThr<=0
      	//两个值都要重新赋值,默认的16
        newCap = DEFAULT_INITIAL_CAPACITY;
      	//16*0.75= 12
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
  	//什么时候新的门槛会==0
    if (newThr == 0) {
      	//
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];//新建一个node
  	//老的table已经保存了,所以可以放心的复制
    table = newTab;
  	//开始处理链表的复制.链表里有红黑树,也要做相应的处理.
    if (oldTab != null) {
      	//一个桶一个桶处理
        for (int j = 0; j < oldCap; ++j) {
          	//临时node
            Node<K,V> e;
          	//j的位置不为空
            if ((e = oldTab[j]) != null) {
              	//oldTab置空
                oldTab[j] = null;
              	//如果这个桶只有一个元素
                if (e.next == null)
                  	//放入newTab的e.hash & (newCap - 1)位置
                  	//这个跟老的tab位置的关系呢
                    newTab[e.hash & (newCap - 1)] = e;
              	//红黑树的转移
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                  	//链表的转移,这几个变量可见构造了两个链表,lo,hi,分别有两个指针
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                      	//两种情况
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                      	//第一种可能放置的地方,放j
                      	//loHead新构建的链表
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                      	//第二种j+oldCap,新构建的链表hiHead
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## 内部类

#### 1.数据的容器之Node

注意下访问修饰符,比较简单的一个node

```java
static class Node<K,V> implements Map.Entry<K,V> {
  			//hash是final类型的
        final int hash;
  			//key
        final K key;
  			//value
        V value;
  			//下一个节点
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```



## 注意点

##### 1.什么时候树化

##### 2.tabSizeFor

##### 3.(n - 1) & hash

##### 4.和LinkedHashMap的关系,哪些钩子方法

##### 5.扩容时要重新计算hash值吗,为什么?

##### 6.链表扩容demo

例如:
我们假设 oldCap = 16, 即 2^4,
16 - 1 = 15, 二进制表示为 `0000 0000 0000 0000 0000 0000 0000 1111`
可见除了低4位, 其他位置都是0（简洁起见，高位的0后面就不写了）, 则 `(16-1) & hash` 自然就是取hash值的低4位,我们假设它为 `abcd`.

以此类推, 当我们将oldCap扩大两倍后, 新的index的位置就变成了 `(32-1) & hash`, 其实就是取 hash值的低5位. 那么对于同一个Node, 低5位的值无外乎下面两种情况:

```
0abcd
1abcd
```

其中, `0abcd`与原来的index值一致, 而`1abcd` = `0abcd + 10000` = `0abcd + oldCap`

故虽然数组大小扩大了一倍，但是同一个`key`在新旧table中对应的index却存在一定联系： 要么一致，要么相差一个 `oldCap`。(妙啊!)

而新旧index是否一致就体现在hash值的第4位(我们把最低为称作第0位), 怎么拿到这一位的值呢, 只要:

```
hash & 0000 0000 0000 0000 0000 0000 0001 0000
```

上式就等效于

```
hash & oldCap
```

故得出结论:

> 如果 `(e.hash & oldCap) == 0` 则该节点在新表的下标位置与旧表一致都为 `j`
> 如果 `(e.hash & oldCap) == 1` 则该节点在新表的下标位置 `j + oldCap`
>
> 

###### 6.通过这个类的源码,思考下如果是自己设计这个类,我会怎么设计.



**引用**

[resize参考了这篇](https://segmentfault.com/a/1190000015812438)

[hash的奥秘]()