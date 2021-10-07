tailMap的实现原理

Returns a view of the portion of this map whose keys are greater than or equal to fromKey.

The returned map is backed by this map, so changes in the returned map are reflected in this map, and vice-versa. The returned map supports all optional map operations that this map supports.

The returned map will throw an IllegalArgumentException on an attempt to insert a key outside its range.



```java
SortedMap<K,V> tailMap(K fromKey);
```



```java
public SortedMap<K,V> tailMap(K fromKey) {
    return tailMap(fromKey, true);
}
```



```java
public NavigableMap<K,V> tailMap(K fromKey, boolean inclusive) {
    return new AscendingSubMap<>(this,
                                 false, fromKey, inclusive,
                                 true,  null,    true);
}
```



```java
AscendingSubMap(TreeMap<K,V> m,
                boolean fromStart, K lo, boolean loInclusive,
                boolean toEnd,     K hi, boolean hiInclusive) {
    super(m, fromStart, lo, loInclusive, toEnd, hi, hiInclusive);
}
```



```java
NavigableSubMap(TreeMap<K,V> m,
                boolean fromStart, K lo, boolean loInclusive,
                boolean toEnd,     K hi, boolean hiInclusive) {
    if (!fromStart && !toEnd) {
        if (m.compare(lo, hi) > 0)
            throw new IllegalArgumentException("fromKey > toKey");
    } else {
        if (!fromStart) // type check
            m.compare(lo, lo);
        if (!toEnd)
            m.compare(hi, hi);
    }
    this.m = m;
    this.fromStart = fromStart;
    this.lo = lo;
    this.loInclusive = loInclusive;
    this.toEnd = toEnd;
    this.hi = hi;
    this.hiInclusive = hiInclusive;
}
```



