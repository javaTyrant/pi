- 1、 Mapper接口是如何被加载 到 Configuration 中的？
- 2、 Mapper代理类是如何生成的？
- 3、 Mapper代理类是如何实现接口方法的？

### 一、加载 Mapper接口



### 二、 Mapper动态代理对象（MapperProxy）的创建



### 三、 MapperProxy 接口方法的实现



### 四、 个人总结

- 1、 Configuration 维护了一个 **MapperRegistry** 对象，该对象主要作用就是加载Mapper接口和获取MapperProxy。
- 2、 MapperRegistry 维护了一个key为 mapper接口class对象，value为 **MapperProxyFactory** 的map
- 3、 **MapperProxy** 是 通过  **MapperProxyFactory** 创建的。
- 4、 **MapperProxy** 实现 Mapper接口方法是委托 **MapperMethod** 执行的。
- 5、 **MapperMethod** 执行接口方法时是通过 **SqlCommand** 来判断要执行的具体 SQL节点，并且最终委托 **SqlSession**执行。
- 6、 **SqlCommand** 内部的 信息是通过从 **MappedStatement** 中获取的。





## 2021年9月28重新梳理.

#### 一个sql的执行过程

**1.代码中的mapper是一个接口,spring初始化的时候会生成一个动态代理.**

org.apache.ibatis.binding.MapperProxy@xxxx

2.然后进入代理方法,method

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  try {
    //是如果定义方法的类是个具体类就使用具体类的实现，如果是接口则往下执行。
    if (Object.class.equals(method.getDeclaringClass())) {
      return method.invoke(this, args);
    } else if (isDefaultMethod(method)) {//如果是默认方法.
      return invokeDefaultMethod(proxy, method, args);
    }
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }
  //获取mapperMethod
  final MapperMethod mapperMethod = cachedMapperMethod(method);
  //继续跟进去.
  return mapperMethod.execute(sqlSession, args);
}
```

获取缓存的mapperMethod.

```java
private MapperMethod cachedMapperMethod(Method method) {
  MapperMethod mapperMethod = methodCache.get(method);
   //为空新建
  if (mapperMethod == null) {
    mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
    methodCache.put(method, mapperMethod);
  }
   //不为空直接返回.
  return mapperMethod;
}
```

MapperMethod.execute.

```java
public Object execute(SqlSession sqlSession, Object[] args) {
  Object result;
  switch (command.getType()) {
    case INSERT: {
   Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.insert(command.getName(), param));
      break;
    }
    case UPDATE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
      break;
    }
    case DELETE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
      break;
    }
    case SELECT:
      if (method.returnsVoid() && method.hasResultHandler()) {
        executeWithResultHandler(sqlSession, args);
        result = null;
          //返回many
      } else if (method.returnsMany()) {
        result = executeForMany(sqlSession, args);
          //返回map
      } else if (method.returnsMap()) {
        result = executeForMap(sqlSession, args);
          //返回游标.
      } else if (method.returnsCursor()) {
        result = executeForCursor(sqlSession, args);
      } else {
        //看看convert方法.返回值呢?
        Object param = method.convertArgsToSqlCommandParam(args);
        //返回一个.
        result = sqlSession.selectOne(command.getName(), param);
      }
      break;
    case FLUSH:
      result = sqlSession.flushStatements();
      break;
    default:
      throw new BindingException("Unknown execution method for: " + command.getName());
  }
  if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
    throw new BindingException("Mapper method '" + command.getName() 
        + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
  }
  return result;
}
```

```java
public Object convertArgsToSqlCommandParam(Object[] args) {
  return paramNameResolver.getNamedParams(args);
}
```



```java
private <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler) {
  try {
    //根据statement获取MappedStatement
    MappedStatement ms = configuration.getMappedStatement(statement);
    //委托到执行器.
    return executor.query(ms, wrapCollection(parameter), rowBounds, handler);
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

**org.apache.ibatis.executor.CachingExecutor@503f5382**

