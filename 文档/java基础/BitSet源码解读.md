## 1.使用

BitSet的使用场景

```java
如统计40亿个数据中没有出现的数据，将40亿个不同数据进行排序等。
现在有1千万个随机数，随机数的范围在1到1亿之间。现在要求写出一种算法，将1到1亿之间没有在随机数中的数求出来如统计40亿个数据中没有出现的数据，将40亿个不同数据进行排序等。
```

## 2.原理java



## 3.源码

```java
//long数组.一个long数组保存64个bits
private long[] words;

```

```java
public void set(int bitIndex) {
    //越界校验
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);
	//确定是在哪个数组.
    int wordIndex = wordIndex(bitIndex);
    //
    expandTo(wordIndex);
	//
    words[wordIndex] |= (1L << bitIndex); // Restores invariants
	//
    checkInvariants();
}
```



```java
private static int wordIndex(int bitIndex) {
    //6
    return bitIndex >> ADDRESS_BITS_PER_WORD;
}
```



```java
private void expandTo(int wordIndex) {
    // + 1
    int wordsRequired = wordIndex+1;
    //使用中 < 需要的
    if (wordsInUse < wordsRequired) {
        //
        ensureCapacity(wordsRequired);
        //
        wordsInUse = wordsRequired;
    }
}
```

 1、首先保证没有越界,否则返回false
 2、然后通过1L << bitIndex,将bitIndex位设置为1
 3、将2中左移的值和words中索引为wordIndex的值进行&操作
 4、如果bitIndex位的值为1,那么&操作后该位的值就为1,此时对应的words元素值不为0,所以返回true。
       如果bitIndex位的值为0,那么&操作后该位的值就为0,此时对应的words元素值为0,所以返回false。

```java
public boolean get(int bitIndex) {
    //
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);
	//
    checkInvariants();
	//获取索引位
    int wordIndex = wordIndex(bitIndex);
    //1L << bitIndex 1 * 2 的 bitIndex次方.
    return (wordIndex < wordsInUse) && ((words[wordIndex] & (1L << bitIndex)) != 0);
}
```

