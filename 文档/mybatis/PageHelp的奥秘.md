

```java
PageHelper.startPage(1, 2);
```

PageMethod

创建一个page变量.保存num和size.

```java
public static <E> Page<E> startPage(int pageNum, int pageSize) {
    return startPage(pageNum, pageSize, DEFAULT_COUNT);
}
```

```java
/**
 * 开始分页
 *
 * @param pageNum      页码
 * @param pageSize     每页显示数量
 * @param count        是否进行count查询
 * @param reasonable   分页合理化,null时用默认配置
 * @param pageSizeZero true且pageSize=0时返回全部结果，false时分页,null时用默认配置
 */
public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count, Boolean reasonable, Boolean pageSizeZero) {
    Page<E> page = new Page<E>(pageNum, pageSize, count);
    page.setReasonable(reasonable);
    page.setPageSizeZero(pageSizeZero);
    //当已经执行过orderBy的时候
    Page<E> oldPage = getLocalPage();
    if (oldPage != null && oldPage.isOrderByOnly()) {
        page.setOrderBy(oldPage.getOrderBy());
    }
    //ThreadLocal的set.
    setLocalPage(page);
    return page;
}
```

LocalPage.是一个ThreadLocal.

```java
protected static final ThreadLocal<Page> LOCAL_PAGE = new ThreadLocal<Page>();
```

获取page是在哪里的呢?

清除本地变量是在哪里的呢?

**ExecutorUtil**

```java
public static  <E> List<E> pageQuery(Dialect dialect, Executor executor, MappedStatement ms, Object parameter,
                             RowBounds rowBounds, ResultHandler resultHandler,
                             BoundSql boundSql, CacheKey cacheKey) throws SQLException {
    //判断是否需要进行分页查询
    if (dialect.beforePage(ms, parameter, rowBounds)) {
        //生成分页的缓存 key
        CacheKey pageKey = cacheKey;
        //处理参数对象
        parameter = dialect.processParameterObject(ms, parameter, boundSql, pageKey);
        //调用方言获取分页 sql
        String pageSql = dialect.getPageSql(ms, boundSql, parameter, rowBounds, pageKey);
        BoundSql pageBoundSql = new BoundSql(ms.getConfiguration(), pageSql, boundSql.getParameterMappings(), parameter);

        Map<String, Object> additionalParameters = getAdditionalParameter(boundSql);
        //设置动态参数
        for (String key : additionalParameters.keySet()) {
            pageBoundSql.setAdditionalParameter(key, additionalParameters.get(key));
        }
        //执行分页查询
        return executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, pageKey, pageBoundSql);
    } else {
        //不执行分页的情况下，也不执行内存分页
        return executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, cacheKey, boundSql);
    }
}  
```



```sql
Select * FROM bill WHERE del_flag = 0 AND order_code IN
(?) LIMIT ? 
```

