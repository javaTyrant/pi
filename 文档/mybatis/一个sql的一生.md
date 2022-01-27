问:

spring的sqlSessionTemplate的作用.

## 1.入口

其实是个接口.

org.apache.ibatis.binding.MapperProxy@6914bb4a

```java
billMapper.selectByOrderCodes(Collections.singletonList("SHZL-2021-000596322"));
```

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, args);
        }

        if (this.isDefaultMethod(method)) {
            return this.invokeDefaultMethod(proxy, method, args);
        }
    } catch (Throwable var5) {
        throw ExceptionUtil.unwrapThrowable(var5);
    }
	//获取mapperMethod.
    MapperMethod mapperMethod = this.cachedMapperMethod(method);
    //执行.
    return mapperMethod.execute(this.sqlSession, args);
}
```

```java
private MapperMethod cachedMapperMethod(Method method) {
  //先从缓存里获取
  MapperMethod mapperMethod = methodCache.get(method);
  //如果为空
  if (mapperMethod == null) {
    //new一个.interface,method,configuration.
    mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
    //put
    methodCache.put(method, mapperMethod);
  }
  return mapperMethod;
}
```

执行.

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
      //
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

<img src="C:\Users\lufengxiang\AppData\Roaming\Typora\typora-user-images\image-20211020171519974.png" alt="image-20211020171519974" style="zoom: 50%;" />

```java
private <E> Object executeForMany(SqlSession sqlSession, Object[] args) {
  List<E> result;
  //
  Object param = method.convertArgsToSqlCommandParam(args);
  //是否有rowBounds
  if (method.hasRowBounds()) {
    RowBounds rowBounds = method.extractRowBounds(args);
    result = sqlSession.<E>selectList(command.getName(), param, rowBounds);
  } else {
    //MapperMethod里转交给sqlSession了.
    //com.znlh.ifs.dal.ifs.mapper.BillMapper.selectByOrderCodes ,Object parameter
    //项目里是SQLSessionTemplate.
    result = sqlSession.<E>selectList(command.getName(), param);
  }
  // issue #510 Collections & arrays support
  if (!method.getReturnType().isAssignableFrom(result.getClass())) {
    if (method.getReturnType().isArray()) {
      return convertToArray(result);
    } else {
      return convertToDeclaredCollection(sqlSession.getConfiguration(), result);
    }
  }
  return result;
}
```

```java
public <E> List<E> selectList(String statement, Object parameter) {
//代理
    return this.sqlSessionProxy.selectList(statement, parameter);
}
```

<img src="C:\Users\lufengxiang\AppData\Roaming\Typora\typora-user-images\image-20211020172401753.png" alt="image-20211020172401753" style="zoom: 50%;" />

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  SqlSession sqlSession = getSqlSession(
      SqlSessionTemplate.this.sqlSessionFactory,
      SqlSessionTemplate.this.executorType,
      SqlSessionTemplate.this.exceptionTranslator);
  try {
    //public abstract java.util.List org.apache.ibatis.session.SqlSession.selectList(java.lang.String,java.lang.Object)
    //走到DefaultSqlSession里.
    Object result = method.invoke(sqlSession, args);
    //
    if (!isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
      // force commit even on non-dirty sessions because some databases require
      // a commit/rollback before calling close()
      sqlSession.commit(true);
    }
    return result;
  } catch (Throwable t) {
    Throwable unwrapped = unwrapThrowable(t);
    if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
      // release the connection to avoid a deadlock if the translator is no loaded. See issue #22
      closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
      sqlSession = null;
      Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException) unwrapped);
      if (translated != null) {
        unwrapped = translated;
      }
    }
    throw unwrapped;
  } finally {
    if (sqlSession != null) {
      closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
    }
  }
}
```



```java
public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType, 		PersistenceExceptionTranslator exceptionTranslator) {

  notNull(sessionFactory, NO_SQL_SESSION_FACTORY_SPECIFIED);
  notNull(executorType, NO_EXECUTOR_TYPE_SPECIFIED);

  SqlSessionHolder holder = (SqlSessionHolder) TransactionSynchronizationManager.getResource(sessionFactory);

  SqlSession session = sessionHolder(executorType, holder);
  if (session != null) {
    return session;
  }

  if (LOGGER.isDebugEnabled()) {
    LOGGER.debug("Creating a new SqlSession");
  }

  session = sessionFactory.openSession(executorType);

  registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);

  return session;
}
```

```java
@Override
public <E> List<E> selectList(String statement, Object parameter) {
  return this.selectList(statement, parameter, RowBounds.DEFAULT);
}
```

<img src="C:\Users\lufengxiang\AppData\Roaming\Typora\typora-user-images\image-20211020192616863.png" alt="image-20211020192616863" style="zoom:50%;" />

这里就进入PageHelp.

```java
@Override
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
  try {
    //
    MappedStatement ms = configuration.getMappedStatement(statement);
    //委托给Executor.哪个executor?org.apache.ibatis.executor.CachingExecutor@9736d7e
    //先进入代理.PageInterceptor.Executor是被代理了的.
    return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```



```java
public Object intercept(Invocation invocation) throws Throwable {
    try {
        Object[] args = invocation.getArgs();
        MappedStatement ms = (MappedStatement)args[0];
        Object parameter = args[1];
        RowBounds rowBounds = (RowBounds)args[2];
        ResultHandler resultHandler = (ResultHandler)args[3];
        //获取代理对象
        Executor executor = (Executor)invocation.getTarget();
        CacheKey cacheKey;
        BoundSql boundSql;
        //
        if (args.length == 4) {
            boundSql = ms.getBoundSql(parameter);
            cacheKey = executor.createCacheKey(ms, parameter, rowBounds, boundSql);
        } else {
            cacheKey = (CacheKey)args[4];
            boundSql = (BoundSql)args[5];
        }
		//
        this.checkDialectExists();
        List resultList;
        if (!this.dialect.skip(ms, parameter, rowBounds)) {
            if (this.dialect.beforeCount(ms, parameter, rowBounds)) {
                //
                Long count = this.count(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                if (!this.dialect.afterCount(count, parameter, rowBounds)) {
                    Object var12 = this.dialect.afterPage(new ArrayList(), parameter, rowBounds);
                    return var12;
                }
            }
			//
            resultList = ExecutorUtil.pageQuery(this.dialect, executor, ms, parameter, rowBounds, resultHandler, boundSql, cacheKey);
        } else {
            resultList = executor.query(ms, parameter, rowBounds, resultHandler, cacheKey, boundSql);
        }

        Object var16 = this.dialect.afterPage(resultList, parameter, rowBounds);
        return var16;
    } finally {
        this.dialect.afterAll();
    }
}
```



```java
@Override
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
  Statement stmt = null;
  try {
    //获取配置
    Configuration configuration = ms.getConfiguration();
    //获取StatementHandler->
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
    //生成Statement
    stmt = prepareStatement(handler, ms.getStatementLog());
    //执行
    return handler.<E>query(stmt, resultHandler);
  } finally {
    closeStatement(stmt);
  }
}
```

RoutingStatementHandler

```java
@Override
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
  //PreparedStatementHandler
  return delegate.<E>query(statement, resultHandler);
}
```



```java
@Override
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
  PreparedStatement ps = (PreparedStatement) statement;
  //是个代理类.PreparedStatementLogger
  ps.execute();
  return resultSetHandler.<E> handleResultSets(ps);
}
```

**PreparedStatementLogger** extends BaseJdbcLogger implements InvocationHandler 

```java
@Override
public Object invoke(Object proxy, Method method, Object[] params) throws Throwable {
  try {
    if (Object.class.equals(method.getDeclaringClass())) {
      return method.invoke(this, params);
    }          
    if (EXECUTE_METHODS.contains(method.getName())) {
      if (isDebugEnabled()) {
        debug("Parameters: " + getParameterValueString(), true);
      }
      clearColumnInfo();
      if ("executeQuery".equals(method.getName())) {
        ResultSet rs = (ResultSet) method.invoke(statement, params);
        return rs == null ? null : ResultSetLogger.newInstance(rs, statementLog, queryStack);
      } else {
        return method.invoke(statement, params);
      }
    } else if (SET_METHODS.contains(method.getName())) {
      if ("setNull".equals(method.getName())) {
        setColumn(params[0], null);
      } else {
        setColumn(params[0], params[1]);
      }
      return method.invoke(statement, params);
    } else if ("getResultSet".equals(method.getName())) {
      ResultSet rs = (ResultSet) method.invoke(statement, params);
      return rs == null ? null : ResultSetLogger.newInstance(rs, statementLog, queryStack);
    } else if ("getUpdateCount".equals(method.getName())) {
      int updateCount = (Integer) method.invoke(statement, params);
      if (updateCount != -1) {
        debug("   Updates: " + updateCount, false);
      }
      return updateCount;
    } else {
      //public abstract boolean java.sql.PreparedStatement.execute() throws java.sql.SQLException
      //这里就开始走到jdk的方法了.
      return method.invoke(statement, params);
    }
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }
}
```







SqlSessionTemplate的作用.

Thread safe, Spring managed, SqlSession that works with **Spring transaction** management to ensure that that the actual SqlSession used is the one associated with the current Spring transaction**. In addition, it manages the session life-cycle, including closing, committing or rolling back the session as necessary based on the Spring transaction configuration.**
The template needs a SqlSessionFactory to create SqlSessions, passed as a constructor argument. It also can be constructed indicating the executor type to be used, if not, the default executor type, defined in the session factory will be used.
This template converts MyBatis PersistenceExceptions into unchecked DataAccessExceptions, using, by default, a MyBatisExceptionTranslator.
Because SqlSessionTemplate is thread safe, a single instance can be shared by all DAOs; there should also be a small memory savings by doing this. This pattern can be used in Spring configuration files as follows:
   <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
   <constructor-arg ref="sqlSessionFactory" />
 </bean>

See Also:
SqlSessionFactory, MyBatisExceptionTranslator
Author:
Putthibong Boonbong, Hunter Presnall, Eduardo Macarron