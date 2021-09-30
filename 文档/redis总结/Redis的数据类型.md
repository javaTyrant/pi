# Redis主要有5种数据类型，包括String，List，Set，Zset，Hash

每种的底层实现呢?

## String

字符串对象的编码可以是int，raw或者embstr。

简单动态字符串,为什么不直接用C的字符串呢?

- 当 int 编码保存的值不再是整数，或大小超过了long的范围时，自动转化为raw。

- 对于 embstr 编码，由于 Redis 没有对其编写任何的修改程序（embstr 是只读的），在对embstr对象进行修改时，都会先转化为raw再进行修改，因此，只要是修改embstr对象，修改后的对象一定是raw的，无论是否达到了44个字节。

## List

list 列表，它是简单的字符串列表，按照插入顺序排序，你可以添加一个元素到列表的头部（左边）或者尾部（右边），它的底层实际上是个链表结构。

　　当同时满足下面两个条件时，使用ziplist（压缩列表）编码：

　　1、列表保存元素个数小于512个

　　2、每个元素长度小于64字节

　　不能满足这两个条件的时候使用 linkedlist 编码。

　　上面两个条件可以在redis.conf 配置文件中的 list-max-ziplist-value选项和 list-max-ziplist-entries 选项进行配置。

## Set 集合

　集合对象的编码可以是 intset 或者 hashtable。

　　当集合同时满足以下两个条件时，使用 intset 编码：

　　1、集合对象中所有元素都是整数

　　2、集合对象所有元素数量不超过512

　　不能满足这两个条件的就使用 hashtable 编码。第二个条件可以通过配置文件的 set-max-intset-entries 进行配置。

## Zset 有序集合

有序集合的编码可以是 ziplist 或者 skiplist。

当有序集合对象同时满足以下两个条件时，对象使用 ziplist 编码：

　　1、保存的元素数量小于128；

　　2、保存的所有元素长度都小于64字节。

　　不能满足上面两个条件的使用 skiplist 编码。

## Hash

哈希对象的编码可以是 ziplist 或者 hashtable。



# 底层数据结构

## ziplist 和 linkedlist

ziplist是一个经过特殊编码的双向链表，它的设计目标就是为了提高存储效率。ziplist可以用于存储字符串或整数，其中整数是按真正的二进制表示进行编码的，而不是编码成字符串序列。它能以O(1)的时间复杂度在表的两端提供`push`和`pop`操作。

## skiplist 来给你手写一个跳表

