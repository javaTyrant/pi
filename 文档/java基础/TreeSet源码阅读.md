## 类结构

```java
 TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
```

## 核心变量

backing map

```java
private transient NavigableMap<E,Object> m;
```

```java
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
```

## 构造器

默认的构造器:用treeMap实现

所以核心的话还是要看TreeMap

```java
public TreeSet() {
    this(new TreeMap<E,Object>());
}
```

自定义底层map

```java
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}
```

## 核心方法

```java
public boolean add(E e) {
  	//只用到了treemap的key
    return m.put(e, PRESENT)==null;
}
```

## 补充

Set

```java
boolean add(E e);
boolean addAll(Collection<? extends E> c);

boolean remove(Object o);
boolean removeAll(Collection<?> c);
void clear();

int size();
boolean isEmpty();

boolean contains(Object o);
boolean containsAll(Collection<?> c);

Object[] toArray();
<T> T[] toArray(T[] a);
@Override
default Spliterator<E> spliterator() {
  	return Spliterators.spliterator(this, Spliterator.DISTINCT);
}

int hashCode();
boolean equals(Object o);


```

AbstractSet

这个set只实现了三个方法

```java
public boolean equals(Object o) {
  	//直接判断==
    if (o == this)
        return true;
		//不是set直接返回true
    if (!(o instanceof Set))
        return false;
  	//如果是set
    Collection<?> c = (Collection<?>) o;
  	//如果size不等,返回false
    if (c.size() != size())
        return false;
    try {
      	//是否全部包含
        return containsAll(c);
    } catch (ClassCastException unused)   {
        return false;
    } catch (NullPointerException unused) {
        return false;
    }
}
```



```java
public int hashCode() {
    int h = 0;
    Iterator<E> i = iterator();
    while (i.hasNext()) {
        E obj = i.next();
        if (obj != null)
          	//累加hashcode
            h += obj.hashCode();
    }
    return h;
}
```

删除全部

```java
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    boolean modified = false;

    if (size() > c.size()) {
        for (Iterator<?> i = c.iterator(); i.hasNext(); )
            modified |= remove(i.next());
    } else {
        for (Iterator<?> i = iterator(); i.hasNext(); ) {
            if (c.contains(i.next())) {
                i.remove();
                modified = true;
            }
        }
    }
    return modified;
}
```

