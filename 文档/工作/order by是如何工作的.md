## 从一个问题看起

先看一个慢sql.

![image-20201222162205286](/Users/lumac/Library/Application Support/typora-user-images/image-20201222162205286.png)

这两个联表查询是没有走filterSort的.

但是我们多加一个skc排序之后,性能得到了显著的提升.extra里多了**Using index condition**,多了一个**Using filesort**.为什么只加一了一个skc的排序,性能就能如此的提升呢?是因为两张表是以skc关联的吗?

![image-20201222162253876](/Users/lumac/Library/Application Support/typora-user-images/image-20201222162253876.png)

## order by是如何工作的



## 什么时候会选择Using index condition



## 什么时候会用filesort

