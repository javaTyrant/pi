## Recycle的内部类

```java
 public final T get() {
   			//校验
        if (maxCapacityPerThread == 0) {
            return newObject((Handle<T>) NOOP_HANDLE);
        }
        //线程封闭的
        Stack<T> stack = threadLocal.get();
        //为什么叫defaultHandle呢
        DefaultHandle<T> handle = stack.pop();
        if (handle == null) {
            handle = stack.newHandle();
            //抽象的方法,最终会掉到实现类,ObjectPool实现
            handle.value = newObject(handle);
        }
        return (T) handle.value;
    }

```

get的流程

先从本地threadLocal获取Stack,然后从stack中弹出handle,handle为空初始化一个,handle不为空直接返回handle的value.

从这个流程可以大致看出来这三个内部类的关系.

### Handle



### DefaultHandle



### Stack



### WeakOrderQueue



### 