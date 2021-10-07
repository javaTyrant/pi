

MyBatis 会将 Mapper 映射文件中定义的 SQL 语句解析成 SqlSource 对象，其中的动态标签、SQL 语句文本等，会解析成对应类型的 SqlNode 对象。

在开始介绍 SqlSource 接口、SqlNode 接口等核心接口的相关内容之前，我们需要先来了解一下动态 SQL 中使用到的基础知识和基础组件。

### OGNL 表达式语言

MyBatis 为了提高 OGNL 表达式的工作效率，添加了一层 OgnlCache 来**缓存**表达式编译之后的结果（不是表达式的执行结果），OgnlCache 通过一个 ConcurrentHashMap<String, Object> 类型的集合（expressionCache 字段，静态字段）来**记录**OGNL 表达式编译之后的结果。通过缓存拿到表达式编译的结果之后，OgnlCache 底层还会依赖上述示例中的 OGNL 工具类以及 OgnlContext 完成表达式的执行。

### DynamicContext 上下文

在 MyBatis 解析一条动态 SQL 语句的时候，可能整个流程非常长，其中涉及多层方法的调用、方法的递归、复杂的循环等，其中**产生的中间结果需要有一个地方进行存储，那就是 DynamicContext 上下文对象**。

DynamicContext 中有两个核心属性：一个是 sqlBuilder 字段（StringJoiner 类型），用来记录解析之后的 SQL 语句；另一个是 bindings 字段，用来记录上下文中的一些 KV 信息。

DynamicContext 定义了一个 ContextMap 内部类，ContextMap 用来记录运行时用户传入的、用来替换“#{}”占位符的实参。在 DynamicContext 构造方法中，会**根据传入的实参类型决定如何创建对应的 ContextMap 对象**，核心代码如下：

```java
public DynamicContext(Configuration configuration, Object parameterObject) {

    if (parameterObject != null && !(parameterObject instanceof Map)) {

        // 对于非Map类型的实参，会创建对应的MetaObject对象，并封装成ContextMap对象

        MetaObject metaObject = configuration.newMetaObject(parameterObject);

        boolean existsTypeHandler = configuration.getTypeHandlerRegistry().hasTypeHandler(parameterObject.getClass());

        bindings = new ContextMap(metaObject, existsTypeHandler);

    } else {

        // 对于Map类型的实参，这里会创建一个空的ContextMap对象

        bindings = new ContextMap(null, false);

    }

    // 这里实参对应的Key是_parameter

    bindings.put(PARAMETER_OBJECT_KEY, parameterObject);

    bindings.put(DATABASE_ID_KEY, configuration.getDatabaseId());

}
```

ContextMap 继承了 HashMap 并覆盖了 get() 方法，在 get() 方法中有一个简单的降级逻辑：

- 首先，尝试按照 Map 的规则查找 Key，如果查找成功直接返回；
- 然后，再尝试检查 parameterObject 这个实参对象是否包含 Key 这个属性，如果包含的话，则直接读取该属性值返回；
- 最后，根据当前是否包含 parameterObject 相应的 TypeHandler 决定是返回整个 parameterObject 对象，还是返回 null。

后面在介绍 `<foreach>`、`<trim>` 等标签的处理逻辑中，你可以看到向 DynamicContext.bindings 集合中写入 KV 数据的操作，但是读取这个 ContextMap 的地方主要是在 OGNL 表达式中，也就是在 DynamicContext 中定义了一个静态代码块，指定了 OGNL 表达式读写 ContextMap 集合的逻辑，这部分读取逻辑封装在 ContextAccessor 中。除此之外，你还可以看到 ContextAccessor 中的 getProperty() 方法会将传入的 target 参数（实际上就是 ContextMap）转换为 Map，并先尝试按照 Map 规则进行查找；查找失败之后，会尝试获取“_parameter”对应的 parameterObject 对象，从 parameterObject 中获取指定的 Value 值。

### 组合模式

**组合模式（有时候也被称为“部分-整体”模式）是将同一类型的多个对象组合成一个树形结构。在使用这个树形结构的时候，我们可以像处理一个对象那样进行处理，而不用关心其复杂的树形结构。**







### SqlNode

![image.png](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20MyBatis%20%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-%E5%AE%8C/assets/Cgp9HWA-CCGAMA5bAADLCPKFfWg640.png)

#### 1. StaticTextSqlNode 和 MixedSqlNode

**StaticTextSqlNode 用于表示非动态的 SQL 片段**，其中维护了一个 text 字段（String 类型），用于记录非动态 SQL 片段的文本内容，其 apply() 方法会直接将 text 字段值追加到 DynamicContext.sqlBuilder 的最末尾。

**MixedSqlNode 在整个 SqlNode 树中充当了树枝节点，也就是扮演了组合模式中 Composite 的角色**，其中维护了一个 `List<SqlNode>` 集合用于记录 MixedSqlNode 下所有的子 SqlNode 对象。MixedSqlNode 对于 apply() 方法的实现也相对比较简单，核心逻辑就是遍历 `List<SqlNode>` 集合中全部的子 SqlNode 对象并调用 apply() 方法，由子 SqlNode 对象完成真正的动态 SQL 处理逻辑。

#### . TextSqlNode

**TextSqlNode 实现抽象了包含 “${}”占位符的动态 SQL 片段**。TextSqlNode 通过一个 text 字段（String 类型）记录了包含“${}”占位符的 SQL 文本内容，在 apply() 方法实现中会结合用户给定的实参解析“${}”占位符，核心代码片段如下：

```java
public boolean apply(DynamicContext context) {
    // 创建GenericTokenParser解析器，这里指定的占位符的起止符号分别是"${"和"}"
    GenericTokenParser parser = createParser(

            new BindingTokenParser(context, injectionFilter));

    // 将解析之后的SQL片段追加到DynamicContext暂存
    context.appendSql(parser.parse(text));

    return true;

}
```



这里**使用 GenericTokenParser 识别“${}”占位符**，在识别到占位符之后，会**通过 BindingTokenParser 将“${}”占位符替换为用户传入的实参**。BindingTokenParser 继承了TokenHandler 接口，在其 handleToken() 方法实现中，会根据 DynamicContext.bindings 这个 ContextMap 中的 KV 数据替换 SQL 语句中的“${}”占位符，相关的代码片段如下：

```java
public String handleToken(String content) {

    // 获取用户提供的实参数据

    Object parameter = context.getBindings().get("_parameter");

    if (parameter == null) { // 通过value占位符，也可以查找到parameter对象

        context.getBindings().put("value", null);

    } else if (SimpleTypeRegistry.isSimpleType(parameter.getClass())) {

        context.getBindings().put("value", parameter);

    }

    // 通过Ognl解析"${}"占位符中的表达式，解析失败的话会返回空字符串

    Object value = OgnlCache.getValue(content, context.getBindings());

    String srtValue = value == null ? "" : String.valueOf(value); 

    checkInjection(srtValue); // 对解析后的值进行过滤

    return srtValue; // 通过过滤的值才能正常返回

}
```

### SqlSource

经过上述一系列处理之后，SQL 语句最终会由 SqlSource 进行最后的处理。

**在 SqlSource 接口中只定义了一个 getBoundSql() 方法，它控制着动态 SQL 语句解析的整个流程**，它会根据从 Mapper.xml 映射文件（或注解）解析到的 SQL 语句以及执行 SQL 时传入的实参，返回一条可执行的 SQL。

![Drawing 3.png](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20MyBatis%20%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-%E5%AE%8C/assets/CioPOWBB-6CAFZlRAAEeTrexVm4358.png)

SqlSource 接口继承图

下面我们简单介绍一下这三个核心实现类的具体含义。

- DynamicSqlSource：当 SQL 语句中包含动态 SQL 的时候，会使用 DynamicSqlSource 对象。
- RawSqlSource：当 SQL 语句中只包含静态 SQL 的时候，会使用 RawSqlSource 对象。
- StaticSqlSource：DynamicSqlSource 和 RawSqlSource 经过一系列解析之后，会得到最终可提交到数据库的 SQL 语句，这个时候就可以通过 StaticSqlSource 进行封装了。

#### 1. DynamicSqlSource

DynamicSqlSource 作为最常用的 SqlSource 实现，**主要负责解析动态 SQL 语句**。

DynamicSqlSource 中维护了一个 SqlNode 类型的字段（rootSqlNode 字段），用于记录整个 SqlNode 树形结构的根节点。在 DynamicSqlSource 的 getBoundSql() 方法实现中，会使用前面介绍的 SqlNode、SqlSourceBuilder 等组件，完成动态 SQL 语句以及“#{}”占位符的解析，具体的实现如下：

```java
public BoundSql getBoundSql(Object parameterObject) {

    // 创建DynamicContext对象，parameterObject是用户传入的实参

    DynamicContext context = new DynamicContext(configuration, parameterObject);

    // 调用rootSqlNode.apply()方法，完成整个树形结构中全部SqlNode对象对SQL片段的解析

    // 这里无须关心rootSqlNode这棵树中到底有多少SqlNode对象，每个SqlNode对象的行为都是一致的，

    // 都会将解析之后的SQL语句片段追加到DynamicContext中，形成最终的、完整的SQL语句

    // 这是使用组合设计模式的好处

    rootSqlNode.apply(context);

    // 通过SqlSourceBuilder解析"#{}"占位符中的属性，并将SQL语句中的"#{}"占位符替换成"?"占位符

    SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);

    Class<?> parameterType = parameterObject == null ? Object.class : parameterObject.getClass();

    SqlSource sqlSource = sqlSourceParser.parse(context.getSql(), parameterType, context.getBindings());

    // 创建BoundSql对象

    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);

    context.getBindings().forEach(boundSql::setAdditionalParameter);

    return boundSql;

}
```

这里最终返回的 BoundSql 对象，包含了解析之后的 SQL 语句（sql 字段）、每个“#{}”占位符的属性信息（parameterMappings 字段 ，List`<ParameterMapping>` 类型）、实参信息（parameterObject 字段）以及 DynamicContext 中记录的 KV 信息（additionalParameters 集合，Map`<String, Object>` 类型）。

后面在讲解 StatementHandler、Executor 如何执行 SQL 语句的时候，我们还会继续介绍 BoundSql 的相关内容，到时候你可以跟这里联系起来学习。



#### 2. RawSqlSource

接下来我们看 SqlSource 的第二个实现—— RawSqlSource，它与 DynamicSqlSource 有两个不同之处：

- RawSqlSource 处理的是非动态 SQL 语句，DynamicSqlSource 处理的是动态 SQL 语句；
- RawSqlSource 解析 SQL 语句的时机是在初始化流程中，而 DynamicSqlSource 解析动态 SQL 的时机是在程序运行过程中，也就是运行时解析。

这里我们需要先来回顾一下前面介绍的 XMLScriptBuilder.parseDynamicTags() 方法，其中会**判断一个 SQL 片段**是否为动态 SQL，判断的标准是：**如果这个 SQL 片段包含了未解析的“${}”占位符或动态 SQL 标签，则为动态 SQL 语句**。但注意，如果是只包含了“#{}”占位符，也不是动态 SQL。

XMLScriptBuilder. parseScriptNode() 方法而 会**判断整个 SQL 语句**是否为动态 SQL，判断的依据是：**如果 SQL 语句中包含任意一个动态 SQL 片段，那么整个 SQL 即为动态 SQL 语句**。

总结来说，对于动态 SQL 语句，MyBatis 会创建 DynamicSqlSource 对象进行处理，而对于非动态 SQL 语句，则会创建 RawSqlSource 对象进行处理。

RawSqlSource 在构造方法中，会调用 SqlNode.apply() 方法将 SQL 片段组装成完整 SQL，然后通过 SqlSourceBuilder 处理“#{}”占位符，得到 StaticSqlSource 对象。这两步处理与 DynamicSqlSource 完全一样，只不过执行的时机是在 RawSqlSource 对象的初始化过程中（即 MyBatis 框架初始化流程中），而不是在 getBoundSql() 方法被调用时（即运行时）。

最后，RawSqlSource.getBoundSql() 方法实现是直接调用 StaticSqlSource.getBoundSql() 方法返回一个 BoundSql 对象。

通过前面的介绍我们知道，无论是 DynamicSqlSource 还是 RawSqlSource，底层都依赖 SqlSourceBuilder 解析之后得到的 StaticSqlSource 对象。StaticSqlSource 中维护了解析之后的 SQL 语句以及“#{}”占位符的属性信息（List`<ParameterMapping>` 集合），其 getBoundSql() 方法是真正创建 BoundSql 对象的地方，这个 BoundSql 对象包含了上述 StaticSqlSource 的两个字段以及实参的信息。