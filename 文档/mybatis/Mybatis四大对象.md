### **Executor** MyBatis的执行器，用于执行增删改查操作

- **Executor** (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)

### StatementHandler 数据库的处理对象，用于执行SQL语句

- **StatementHandler** (prepare, parameterize, batch, update, query)

### ParameterHandler 处理SQL的参数对象

- **ParameterHandler** (getParameterObject, setParameters)

### ResultSetHandler 处理SQL的返回结果集

- **ResultSetHandler** (handleResultSets, handleOutputParameters)

### 问题

分页原理:分页插件的实现原理，就是针对原有的 `select xxx` 查询语句通过代码动态的为其增加 `limit` 的分页查询语句，以此达到不编写分页语句，直接通过代码定义就能达到分页的效果。

pagehelp通过拦截哪个对象来实现分页的?

鉴于此能够知道，分页插件的实现需要在 SQL 执行前对SQL进行查询语句拼接，而通过拦截 `Executor` 和 `StatementHandler` 便能够实现 SQL 执行前的动态编辑。

通常来说直接拦截 `Executor` 即可，因为在 `Executor` `query` 执行是，持有 `MappedStatement` 和 `ResultHandler` 的引用，相当于有了 SQL 更改和结果集操作的能力。

pageHelp 通过拦截 `Executor#query` 实现的分页插件。

