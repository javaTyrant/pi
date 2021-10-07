思考下这几个类的关系.

### MapperRegistry

### MapperProxyFactory

### MapperProxy 

### MapperMethod

```java
public Object execute(SqlSession sqlSession, Object[] args) 
```



从**MapperRegistry**说起

```java
//哈,生成代理对象
@SuppressWarnings("unchecked")
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  //从缓存里获取Factory.
  final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
  if (mapperProxyFactory == null) {
    throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
  }
  try {
    //MapperRegistry -> MapperProxyFactory
    return mapperProxyFactory.newInstance(sqlSession);
  } catch (Exception e) {
    throw new BindingException("Error getting mapper instance. Cause: " + e, e);
  }
}
```

生成代理.

```java
public T newInstance(SqlSession sqlSession) {
  //封装成MapperProxy. MapperProxy实现了InvocationHandler
  //有了sqlSession,有了接口,有了方法.
  final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
  //
  return newInstance(mapperProxy);
}
```

最终是用了Proxy.

```java
@SuppressWarnings("unchecked")
protected T newInstance(MapperProxy<T> mapperProxy) {
  //用proxy生成代理对象.
  return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[]{mapperInterface}, mapperProxy);
}
```

MapperProxy的invoke方法肯定是核心了.

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  try {
    //如果不是接口,实现了方法,就直接调用.
    if (Object.class.equals(method.getDeclaringClass())) {
      return method.invoke(this, args);
    } else if (method.isDefault()) {
      //默认方法.
      if (privateLookupInMethod == null) {
        return invokeDefaultMethodJava8(proxy, method, args);
      } else {
        return invokeDefaultMethodJava9(proxy, method, args);
      }
    }
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }
  //到这里,就是我们业务代码中定义的mapper接口.需要生成代理实现.
  //MapperMethod负责执行sql.
  final MapperMethod mapperMethod = cachedMapperMethod(method);
  //switch case update or select .....
  return mapperMethod.execute(sqlSession, args);
}
```

**MapperMethod**

```java
//最终执行sql的地方.所以重要的工作就转移到SqlSession啦.
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
      } else if (method.returnsMany()) {
        result = executeForMany(sqlSession, args);
      } else if (method.returnsMap()) {
        result = executeForMap(sqlSession, args);
      } else if (method.returnsCursor()) {
        result = executeForCursor(sqlSession, args);
      } else {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = sqlSession.selectOne(command.getName(), param);
        if (method.returnsOptional()
          && (result == null || !method.getReturnType().equals(result.getClass()))) {
          result = Optional.ofNullable(result);
        }
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

