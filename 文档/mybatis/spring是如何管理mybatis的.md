SqlSessionFactory实现了InitializingBean接口，也就是说，在该Bean初始化的时候，我执行InitializingBean中的afterPropertiesSet()方法。那么，我们直接进入SqlSessionFactory的afterPropertiesSet()方法：

```java
@Override
public void afterPropertiesSet() throws Exception {
  notNull(dataSource, "Property 'dataSource' is required");
  notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
  state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
            "Property 'configuration' and 'configLocation' can not specified with together");

  this.sqlSessionFactory = buildSqlSessionFactory();
}
```



```java
protected SqlSessionFactory buildSqlSessionFactory() throws IOException {
    
}
```

## InitializingBean

InitializingBean接口为bean提供了初始化方法的方式，它只包括afterPropertiesSet方法，凡是继承该接口的类，在初始化bean的时候会执行该方法。



## afterPropertiesSet的调用链

<img src="C:\Users\lufengxiang\AppData\Roaming\Typora\typora-user-images\image-20211009180655958.png" alt="image-20211009180655958" style="zoom:50%;" />

```java
protected void invokeInitMethods(String beanName, final Object bean, RootBeanDefinition mbd)
      throws Throwable {

   boolean isInitializingBean = (bean instanceof InitializingBean);
   if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
      if (logger.isDebugEnabled()) {
         logger.debug("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
      }
      if (System.getSecurityManager() != null) {
         try {
            AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
               @Override
               public Object run() throws Exception {
                  ((InitializingBean) bean).afterPropertiesSet();
                  return null;
               }
            }, getAccessControlContext());
         }
         catch (PrivilegedActionException pae) {
            throw pae.getException();
         }
      }
      else {
         ((InitializingBean) bean).afterPropertiesSet();
      }
   }
```

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
   if (System.getSecurityManager() != null) {
      AccessController.doPrivileged(new PrivilegedAction<Object>() {
         @Override
         public Object run() {
            invokeAwareMethods(beanName, bean);
            return null;
         }
      }, getAccessControlContext());
   }
   else {
      invokeAwareMethods(beanName, bean);
   }

   Object wrappedBean = bean;
   if (mbd == null || !mbd.isSynthetic()) {
      wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   }

   try {
      invokeInitMethods(beanName, wrappedBean, mbd);
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex);
   }

   if (mbd == null || !mbd.isSynthetic()) {
      wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
   }
   return wrappedBean;
}
```