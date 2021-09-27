# 使用



# 源码解读

CompletionStage
A stage of a possibly asynchronous computation, that performs an action or computes a value when another CompletionStage completes.

```java
类结构
class CompletableFuture<T> implements Future<T>, CompletionStage<T>
  
```

