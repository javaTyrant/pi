Spring循环依赖的情况总结如下：

1. 不能解决的情况： 1. 构造器注入循环依赖 2. `prototype` field属性注入循环依赖
2. 能解决的情况： 1. field属性注入（setter方法注入）循环依赖

![image-20210202184809998](/Users/lumac/Library/Application Support/typora-user-images/image-20210202184809998.png)

 对Bean的创建最为核心三个方法解释如下：

- `createBeanInstance`：实例化，其实也就是调用对象的**构造方法**实例化对象
- `populateBean`：填充属性，这一步主要是对bean的依赖属性进行注入(`@Autowired`)
- `initializeBean`：回到一些形如`initMethod`、`InitializingBean`等方法

从对`单例Bean`的初始化可以看出，循环依赖主要发生在**第二步（populateBean）**，也就是field属性注入的处理。

在Spring容器的整个声明周期中，单例Bean有且仅有一个对象。这很容易让人想到可以用缓存来加速访问。

**三级缓存**

1. `singletonObjects`：用于存放完全初始化好的 bean，**从该缓存中取出的 bean 可以直接使用**
2. `earlySingletonObjects`：提前曝光的单例对象的cache，存放原始的 bean 对象（尚未填充属性），用于解决循环依赖
3. `singletonFactories`：单例对象工厂的cache，存放 bean 工厂对象，用于解决循环依赖

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	...
	@Override
	@Nullable
	public Object getSingleton(String beanName) {
		return getSingleton(beanName, true);
	}
	@Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
	...
	public boolean isSingletonCurrentlyInCreation(String beanName) {
		return this.singletonsCurrentlyInCreation.contains(beanName);
	}
	protected boolean isActuallyInCreation(String beanName) {
		return isSingletonCurrentlyInCreation(beanName);
	}
	...
}
```

1. 先从`一级缓存singletonObjects`中去获取。（如果获取到就直接return）
2. 如果获取不到或者**对象正在创建中**（`isSingletonCurrentlyInCreation()`），那就再从`二级缓存earlySingletonObjects`中获取。（如果获取到就直接return）
3. 如果还是获取不到，且允许singletonFactories（allowEarlyReference=true）通过`getObject()`获取。就从`三级缓存singletonFactory`.getObject()获取。**（如果获取到了就从**`**singletonFactories**`**中移除，并且放进**`**earlySingletonObjects**`**。其实也就是从三级缓存**`**移动（是剪切、不是复制哦~）**`**到了二级缓存）**

`getSingleton()`从缓存里获取单例对象步骤分析可知，Spring解决循环依赖的诀窍：**就在于singletonFactories这个三级缓存**。这个Cache里面都是`ObjectFactory`，它是解决问题的关键。

```java
// 它可以将创建对象的步骤封装到ObjectFactory中 交给自定义的Scope来选择是否需要创建对象来灵活的实现scope。  具体参见Scope接口
@FunctionalInterface
public interface ObjectFactory<T> {
	T getObject() throws BeansException;
}
```

>  经过ObjectFactory.getObject()后，此时放进了二级缓存`earlySingletonObjects`内。这个时候对象已经实例化了，`虽然还不完美`，但是对象的引用已经可以被其它引用了。 

**此处说一下二级缓存`earlySingletonObjects`它里面的数据什么时候添加什么移除？？?**

**添加**：向里面添加数据只有一个地方，就是上面说的`getSingleton()`里从三级缓存里挪过来 **移除**：`addSingleton、addSingletonFactory、removeSingleton`从语义中可以看出添加单例、添加单例工厂`ObjectFactory`的时候都会删除二级缓存里面对应的缓存值，是互斥的