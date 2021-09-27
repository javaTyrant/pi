# 类结构

```java
final class Optional<T> 
```

# 核心变量

```java
//空对象
private static final Optional<?> EMPTY = new Optional<>();
//value
private final T value;
```

# 核心方法

get可能是会报空指针的

```java
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```

```java
  public T orElse(T other) {
        return value != null ? value : other;
    }
```

```java
  public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

```java
  public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```



# 总结

很简单的一个类,注意下方法的抽象就可以了.

