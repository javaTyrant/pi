非线程安全的:很简单的一个类

思考下.string为什么是不可变的,或者说不可变的字符串的优势何在

String不可变的原因包括 设计考虑,效率优化问题,以及安全性这三大方面. 

String的类结构是什么呢?

```java
String
    implements java.io.Serializable, Comparable<String>, CharSequence
```

# 类结构:

```java
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
```

# CharSequence:顾名思义就知道是什么意思了

```java
int length()
char charAt(int index);
CharSequence subSequence(int start, int end);
public String toString();  
//两个默认的方法
public default IntStream chars() {}
public default IntStream codePoints() {}
```

# AbstractStringBuilder

只有三个变量,char[]保存数据,count记录使用的char的个数,MAX_ARRAY_SIZE记录数组的最大长度

```java
/**
 * The value is used for character storage.
 */
char[] value;

/**
 * The count is the number of characters used.
 */
int count;

    /**
     * The maximum size of array to allocate (unless necessary).
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

只看个append吧

从StringBuilder开始看吧

```java
@Override
public StringBuilder append(CharSequence s) {
    super.append(s);
    return this;
}
```

#  super.append(s);

```java
  public AbstractStringBuilder append(char[] str) {
    		//字符串长度
        int len = str.length;
    		//长度校验,长度不够扩容数组
        ensureCapacityInternal(count + len);
    		//[a,b,c,0,0,0] e,f,g
    		//count = 3,len= 3
    		//arraycopy('efg',0,value,3,3)
    		//用的arraycopy,把str从0的位置复制到value里从count的位置开始,到len的位置结束
        System.arraycopy(str, 0, value, count, len);
        count += len;
    		//返回
        return this;
    }

```

# java.lang.AbstractStringBuilder#ensureCapacityInternal

```java
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}
```

# java.lang.System#arraycopy

```java
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

# 算法总结:实现strStr()

> 给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。
>

```java
public int strStr(String haystack, String needle) {
        int L = needle.length(), n = haystack.length();
        if (L == 0) return 0;
				
        int pn = 0;
        while (pn < n - L + 1) {
            // find the position of the first needle character
            // in the haystack string
          	//
            while (pn < n - L + 1 && haystack.charAt(pn) != needle.charAt(0)) ++pn;

            // compute the max match string
            int currLen = 0, pL = 0;
            while (pL < L && pn < n && haystack.charAt(pn) == needle.charAt(pL)) {
                ++pn;
                ++pL;
                ++currLen;
            }

            // if the whole needle string is found,
            // return its start position
            if (currLen == L) return pn - L;

            // otherwise, backtrack
            pn = pn - currLen + 1;
        }
        return -1;
    }
```

