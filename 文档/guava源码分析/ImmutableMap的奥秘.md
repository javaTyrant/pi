我们先看下ImmutableMap的put方法

简单的用法

```java
  ImmutableMap<String, String> map = new ImmutableMap.Builder<String, String>()
                .put("1", "1")
                .put("2", "2")
                .build();
        //会报错,UnsupportedOperationException
        //map.put("a","a");不支持直接的put,妙啊
        System.out.println(map);
```



```java
 public Builder<K, V> put(K key, V value) {
   					//确保容量大小
            ensureCapacity(size + 1);
   					//根据入参构造entry
            Entry<K, V> entry = entryOf(key, value);
            // don't inline this: we want to fail atomically if key or value is null
   					//放入entries数组
            entries[size++] = entry;
   					//返回对象可以提供链式操作
            return this;
        }
```

```
Entry<K, V>[] entries;
```

我们来看下ImmutableMap这个Builder吧

有四个属性:

```java
Comparator<? super V> valueComparator;
Entry<K, V>[] entries;
int size;
boolean entriesUsed;
//构造器
public Builder() {
     this(ImmutableCollection.Builder.DEFAULT_INITIAL_CAPACITY);
}
```



