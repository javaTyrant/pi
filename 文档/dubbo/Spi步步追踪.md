## spi断点过程

```java
 ExtensionLoader<Animal> extensionLoader = ExtensionLoader.getExtensionLoader(Animal.class);
```

getExtensionLoader:

1.先做几个校验

2.从缓存EXTENSION_LOADERS中获取,获取不到直接new一个

看看构造器,构造器是私有的,还判断了type是不是==ExtensionFactory.class

显然不是,我们传的是Animal.class

```java
  private ExtensionLoader(Class<?> type) {
        this.type = type;
        //关键点:获取工厂类型,getAdaptiveExtension
        //如果type不是ExtensionFactory,那么先获取ExtensionFactory
        objectFactory = (type == ExtensionFactory.class ?
                null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
    }
```

所以要ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension()

然后这里又会调用一次构造器

此时objectFactory = null

然后调用getAdaptiveExtension,返回一个自适应扩展,这个自适应扩展,应该把扩展的方法全都封装到一起了.

这是个非常重要的方法:cachedAdaptiveInstance

缓存为空的化校验下createAdaptiveInstanceError,如果错误不为空,直接报错

否则加锁,再次get如果还为空,就创建一个AdaptiveExtension

所以我们的核心又到了createAdaptiveExtension.

然后把创建好的实例设置到cachedAdaptiveInstance里.

```java
 public T getAdaptiveExtension() {
        //先取缓存
        Object instance = cachedAdaptiveInstance.get();
        if (instance == null) {
            if (createAdaptiveInstanceError != null) {
                throw new IllegalStateException("Failed to create adaptive instance: " +
                        createAdaptiveInstanceError.toString(),
                        createAdaptiveInstanceError);
            }
            //对什么加锁:cachedAdaptiveInstance
            synchronized (cachedAdaptiveInstance) {
                instance = cachedAdaptiveInstance.get();
                if (instance == null) {
                    try {
                        //此时创建的是AdaptiveExtensionFactory
                        instance = createAdaptiveExtension();
                        cachedAdaptiveInstance.set(instance);
                    } catch (Throwable t) {
                        createAdaptiveInstanceError = t;
                        throw new IllegalStateException("Failed to create adaptive instance: " + t.toString(), t);
                    }
                }
            }
        }

        return (T) instance;
    }
```

