因为比较简单,我们直接上源码

```java
//入参,需要一个Key,一个function,把这个Key转成其他的格式
default V computeIfAbsent(K key,Function<? super K, ? extends V> mappingFunction) {
  	//非空判断
    Objects.requireNonNull(mappingFunction);
    V v;
  	//如果map里没有这个key
    if ((v = get(key)) == null) {
        V newValue;
      	//
        if ((newValue = mappingFunction.apply(key)) != null) {
          	//放入map里
            put(key, newValue);
            return newValue;
        }
    }
		//如果有key,返回v
    return v;
}
```

