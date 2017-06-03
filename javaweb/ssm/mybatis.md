# MyBatis

* 曾叫iBatis, 后迁移到Google code改为MyBatis

与Hibernate的区别:
* 对JDBC的封装, 比Hibernate更轻量
* 不是完全的ORM, 只是SQL语句和结果集的映射
* 直接使用SQL, 便于对SQL语句调优, 避免自动生成的SQL导致性能问题

解决了JDBC的几个缺点:
* 数据库连接频繁创建导致系统开销: 使用连接池
* SQL语句和Java代码混合不好维护: 使用XML映射分离
* SQL语句传参和条件变化的问题: 使用动态SQL语句解决
* 结果集和Java类型的转换: 使用ResultMap自动映射


## 常见JavaBean类型

* `POJO`: 不按MVC分层, 只是JavaBean, 有属性和getter和setter
* `domain`: 不按MVC分层, 只是JavaBean, 有属性和getter和setter
* `PO`: Persistent Object, 用于持久层, 还可以在增加或修改时, 从页面直接传入action中, 类名等于表名, 属性名等于表字段名, 有getter和setter
* `VO`: View Object, 表现层对象, 用于在高级查询中从页面接收传过来的各种参数, 扩展性强
* `BO`: Business Object, 用于service层, 现在基本不用


## 安装配置

* maven

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

### 日志配置

* 添加日志依赖
* 增加一项配置, name的取值可以是:
    - `SLF4J`
    - `LOG4J`
    - `LOG4J2`
    - `JDK_LOGGING`
    - `COMMONS_LOGGING`
    - `STDOUT_LOGGING`
    - `NO_LOGGING`

```xml
mybatis-config.xml
------------------
<configuration>
    <settings>
        ...
        <setting name="logImpl" value="LOG4J"/>
        ...
    </settings>
</configuration>
```


## 架构原理

* 配置文件
    - `mybatis-config.xml`: 核心配置文件, 配置运行环境
    - `XxxMapper.xml`: sql映射, 配置操作数据库的sql语句, 对应Java代码中的`XxxMapper.java`, 编写完成后需要在`mybatis-config.xml`中注册
* `SqlSessionFactory`: 会话工厂, 用于闯进SqlSession
* `SqlSession`: 会话, 操作数据库通过SqlSession进行
* `Executor`接口: 执行器, 操作数据库, 有2个实现类
    - 基本执行器
    - 缓存执行器
* `Mapped Statement`: 封装了mybatis配置和sql映射.
    - Mapper.xml中一个sql对应一个Mapped Statement对象, sql的id即Mapped statement的id
    - 定义了sql输入参数, 包括hashMap, 基本数据类型, POJO. Executor通过Mapped Statement在执行sql前将输入的java对象映射到sql中, 输入参数映射就是jdbc变成中对preparedStatement设置参数
    - 定义了sql输出结果, 包括hashMap, 基本数据类型, POJO. Executor通过Mapped Statement在执行sql后将输出结果映射至java对象中, 输出结果映射过程相当于jdbc变成中对结果的解析处理过程


## 快速入门

* 步骤:
    1. 创建SqlSessionFactory
        - xml方式: `mybatis-config.xml`
        - 代码方式
    2. 从SqlSessionFactory中获取SqlSession
    3. 创建POJO类
    4. 创建SQL映射文件
        - xml方式: `XxxMapper.xml`
        - 注解方式

```xml
mybatis-config.xml
----------------
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 加载外部配置文件的键值对, 用于填充xml中的变量 -->
    <properties resource="com/package/config.properties"/>

    <!-- 配置数据库 -->
    <environments default="development">
        <environment id="development">
            <!-- 事务处理 -->
            <transactionManager type="JDBC"/>
            <!-- 数据库连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

  <!-- 加载SQL映射 -->
    <mappers>
        <mapper resource="com/package/UserMapper.xml"/>
    </mappers>
</configuration>
```

```java
POJO
----
public class User {
    private int id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;

    // getter & setter
}
```

```xml
UserMapper.xml
---------
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.package.UserMapper">
    <!-- 定义sql语句 -->
    <select id="selectUser" parameterType="int" resultType="com.pakcage.domain.User">
        select * from User where id = #{id}
    </select>
</mapper>
```

```java
执行查询
-------
public class QueryUser {

    private SqlSessionFactory factory;

    private SqlSessionFactory createSqlSessionFactory() {
        String resource = "com/package/mybatis-config.xml";
        InputStream is = Resources.getResourceAsStream(resource);
        return new SqlSessionFactoryBuilder().build(is);
    }

    private void queryUser() {
        factory = createSqlSessionFactory();
        SqlSession session = factory.openSession();
        try {
            // 方式1: 使用命名空间+id
            // User user = (User) session.selectOne("com.package.UserMapper.selectUser", 1);

            // 方式2: 使用Mapper类
            UserMapper mapper = session.getMapper(UserMapper.class);
            User user = mapper.selectUser(1);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            session.close();
        }
    }
}
```


## 作用域和生命周期

* `SqlSessionFactoryBuilder`
    - 作用域: 局部方法作用域
    - 生命周期: 随方法执行创建, 方法执行完毕销毁
* `SqlSessionFactory`
    - 作用域: 应用作用域
    - 生命周期: 随应用创建, 随应用销毁, 一般在运行期间只创建一个实例
* `SqlSession`
    - 作用域: 请求作用域, 或局部方法作用域
    - 生命周期: 随请求或方法创建和销毁
* Mapper实例
    - 作用域: 方法作用域
    - 生命周期: 随方法创建和销毁


## 配置文件

* 一般使用`mybatis-config.xml`作为配置文件名
* 配置文件的结构如下:

```
configuration                     配置
    properties                    属性
    settings                      设置
    typeAliases                   类型命名
    typeHandlers                  类型处理器
    objectFactory                 对象工厂
    plugins                       插件
    environments                  环境
        environment               环境变量
            transactionManager    事务管理器
            dataSource            数据源
    databaseIdProvider            数据库厂商标识
    mappers                       映射器
```

### `<properties>`属性

* 这种属性是可以通过外部文件配置的, 并且可以动态替换
* 通常用于引入外部配置文件, 填充xml配置中的占位符(通过`${xxx}`表示)
* 如果同一个属性在多个地方都配置过, 则加载优先级为:
    - `作为方法参数传递的属性 > resource/url属性指定的配置文件config.properties > <properties>标签`
* 属性值占位符的默认值
    - 3.4.2开始, 可以为占位符`${xxx}`设置一个默认值, 当指定占位符不存在时, 使用默认占位符
        - 格式: `value="${username:ut_user}"`, 如果`username`属性不存在, 则使用`ut_user`属性的值
        - 该特性默认是关闭的, 需要通过以下配置开启:
            - `<property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/>`
    - 默认值也可以使用三元运算符表示
        - 格式: `${tableName != null ? tableName : 'global_constants'}`

```xml
<!-- 配置属性 -->
<properties resource="org/mybatis/example/config.properties"> <!-- 引用外部配置文件 -->
  <property name="username" value="dev_user"/>  <!-- 定义了username属性的值 -->
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>

<!-- 使用属性 -->
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/> <!-- 通过占位符引用username -->
  <property name="password" value="${password}"/>
</dataSource>
```

### `<settings>`设置

* 是MyBatis极为重要的调整设置, 会改变MyBatis的运行时行为
* 在该标签内部使用`<setting name="设置参数" value="参数值"/>`
* 以下是设置参数功能表

|设置参数|说明|有效值|默认值|
|-------|----|-----|------|
|cacheEnabled|该配置影响的所有映射器中配置的缓存的全局开关. |`true / false`|`true`|
|lazyLoadingEnabled|延迟加载的全局开关. 当开启时, 所有关联对象都会延迟加载.  特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态. |`true / false`|`false`|
|aggressiveLazyLoading|当开启时, 任何方法的调用都会加载该对象的所有属性. 否则, 每个属性会按需加载(参考lazyLoadTriggerMethods).|`true / false`|`false (true in ≤3.4.1)`|
|multipleResultSetsEnabled|是否允许单一语句返回多结果集(需要兼容驱动). |`true / false`|`true`|
|useColumnLabel|使用列标签代替列名. 不同的驱动在这方面会有不同的表现,  具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果. |`true / false`|`true`|
|useGeneratedKeys|允许 JDBC 支持自动生成主键, 需要驱动兼容.  如果设置为 true 则这个设置强制使用自动生成主键, 尽管一些驱动不能兼容但仍可正常工作(比如 Derby). |`true / false`|`False`|
|autoMappingBehavior|指定 MyBatis 应如何自动映射列到字段或属性.  NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集.  FULL 会自动映射任意复杂的结果集(无论是否嵌套). |`NONE, PARTIAL, FULL`|`PARTIAL`|
|autoMappingUnknownColumnBehavior|指定发现自动映射目标未知列(或者未知属性类型)的行为. `NONE`: 不做任何反应; `WARNING`: 输出提醒日志('org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 `WARN`); `FAILING`: 映射失败 (抛出 SqlSessionException)|`NONE`, `WARNING`, `FAILING`|`NONE`|
|defaultExecutorType|配置默认的执行器. SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句(prepared statements)； BATCH 执行器将重用语句并执行批量更新. |`SIMPLE`, `REUSE`, `BATCH`|`SIMPLE`|
|defaultStatementTimeout|设置超时时间, 它决定驱动等待数据库响应的秒数. |任意正整数	|Not Set (null)|
|defaultFetchSize|为驱动的结果集获取数量(fetchSize)设置一个提示值. 此参数只可以在查询设置中被覆盖. |任意正整数|Not Set (null)|
|safeRowBoundsEnabled|允许在嵌套语句中使用分页(RowBounds).  If allow, set the false.|`true / false`|`False`|
|safeResultHandlerEnabled|允许在嵌套语句中使用分页(ResultHandler).  If allow, set the false.|`true / false`|`True`|
|mapUnderscoreToCamelCase|是否开启自动驼峰命名规则(camel case)映射, 即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射. |`true / false`|`False`|
|localCacheScope|MyBatis 利用本地缓存机制(Local Cache)防止循环引用(circular references)和加速重复嵌套查询.  默认值为 SESSION, 这种情况下会缓存一个会话中执行的所有查询.  若设置值为 STATEMENT, 本地会话仅用在语句执行上, 对相同 SqlSession 的不同调用将不会共享数据. |`SESSION / STATEMENT`|`SESSION`|
|jdbcTypeForNull|当没有为参数提供特定的 JDBC 类型时, 为空值指定 JDBC 类型.  某些驱动需要指定列的 JDBC 类型, 多数情况直接用一般类型即可, 比如 NULL、VARCHAR 或 OTHER. |JdbcType enumeration. Most common are: NULL, VARCHAR and OTHER|OTHER|
|lazyLoadTriggerMethods|指定哪个对象的方法触发一次延迟加载. |A method name list separated by commas|equals,clone,hashCode,toString|
|defaultScriptingLanguage|指定动态 SQL 生成的默认语言. |A type alias or fully qualified class name.|org.apache.ibatis.scripting.xmltags.XMLLanguageDriver|
|callSettersOnNulls|指定当结果集中值为 null 的时候是否调用映射对象的 setter(map 对象时为 put)方法, 这对于有 Map.keySet() 依赖或 null 值初始化的时候是有用的. 注意基本类型(int、boolean等)是不能设置成 null 的. |`true / false`|`false`|
|returnInstanceForEmptyRow|当返回行的所有列都是空时, MyBatis默认返回null.  当开启这个设置时, MyBatis会返回一个空实例.  请注意, 它也适用于嵌套的结果集 (i.e. collectioin and association). (从3.4.2开始)|`true / false`|`false`|
|logPrefix|指定 MyBatis 增加到日志名称的前缀. |Any String|Not set|
|logImpl|指定 MyBatis 所用日志的具体实现, 未指定时将自动查找. |`SLF4J / LOG4J / LOG4J2 / JDK_LOGGING / COMMONS_LOGGING / STDOUT_LOGGING / NO_LOGGING`|Not set|
|proxyFactory|指定 Mybatis 创建具有延迟加载能力的对象所用到的代理工具. |`CGLIB / JAVASSIST`|`JAVASSIST (MyBatis 3.3 or above)`|
|vfsImpl|指定VFS的实现|自定义VFS的实现的类全限定名, 以逗号分隔. |Not set|
|useActualParamName|允许使用方法签名中的名称作为语句参数名称. 为了使用该特性, 你的工程必须采用Java 8编译, 并且加上-parameters选项. (从3.4.1开始)|`true / false`|`true`|
|configurationFactory|Specifies the class that provides an instance of Configuration. The returned Configuration instance is used to load lazy properties of deserialized objects. This class must have a method with a signature static Configuration getConfiguration(). (Since: 3.2.3)|A type alias or fully qualified class name|Not set|

### `<typeAliases>`类型别名

* 用于为Java类型设置一个简短的名字. 这样在其他需要写完整类名的地方都可以用更简短的方式来写
* 使用别名: 在该标签内部使用`<typeAliase alias="别名" type="完整类名"/>`
* 设置在指定包名下搜索JavaBean: 在该标签内部使用`<package name="包名"/>`
    - 这样就可以使用JavaBean类名的首字母小写方式作为别名, 如`com.xxx.bean.Author`就可以使用`author`
    - 如果在JavaBean类上使用了`@Alias("别名")`, 则要使用注解配置的别名
* MyBatis已经为常见Java类建立了类型别名, 大小写不敏感

|类型|别名|
|----|---|
|byte|_byte|
|long|_long|
|short|_short|
|int|_int|
|int|_integer|
|double|_double|
|float|_float|
|boolean|_boolean|
|String|string|
|Byte|byte|
|Long|long|
|Short|short|
|Integer|int|
|Integer|integer|
|Double|double|
|Float|float|
|Boolean|boolean|
|Date|date|
|BigDecimal|decimal|
|BigDecimal|bigdecimal|
|Object|object|
|Map|map|
|HashMap|hashmap|
|List|list|
|ArrayList|arraylist|
|Collection|collection|
|Iterator|iterator|

### `<typeHandlers>`类型处理器

* 类型处理器用于在为PreparedStatement设置参数和从结果集中取出值时进行Java类型转换
* 如果使用JSR-310(Date和Time的API), 可以使用`mybatis-typehandlers-jsr310`
* 自定义处理器:
    - MaBatis有默认的类型处理器, 如果有特殊的类型需要处理, 也可以自定义类型处理器. 方法如下:
        - 方式1: 实现`org.apache.ibatis.type.TypeHandler`接口, 重写所有方法
        - 方式2: 继承`org.apache.ibatis.type.BaseTypeHandler`类, 重写所需方法.
        - 使用`@MappedJdbcTypes(JdbcType.VARCHAR)`将该自定义处理器映射到一个JDBC类
        - 在该标签中使用`<typeHandler handler="自定义类型处理器完整类名"/>`
    - 自定义泛型类型处理器可以处理多种类型, 它需要创建一个接受泛型参数的构造方法
    - 设置在指定包查找类型处理器: 在该标签内添加`<package name="包名"/>`, 则会在该包内自动查找使用注解方式指定的JDBC类型
* 以下是默认的类处理器

|类处理器|Java类型|JDBC类型|
|-------|--------|-------|
|`BooleanTypeHandler`|`java.lang.Boolean`, `boolean`|数据库兼容的 `BOOLEAN`|
|`ByteTypeHandler`|`java.lang.Byte`, `byte`|数据库兼容的 `NUMERIC` 或 `BYTE`|
|`ShortTypeHandler`|`java.lang.Short`, `short`|数据库兼容的 `NUMERIC` 或 `SHORT` INTEGER|
|`IntegerTypeHandler`|`java.lang.Integer`, `int`|数据库兼容的 `NUMERIC` 或 `INTEGER`|
|`LongTypeHandler`|`java.lang.Long`, `long`|	数据库兼容的 `NUMERIC` 或 `LONG` INTEGER|
|`FloatTypeHandler`|`java.lang.Float`, `float`|数据库兼容的 `NUMERIC` 或 `FLOAT`|
|`DoubleTypeHandler`|`java.lang.Double`, `double`|数据库兼容的 `NUMERIC` 或 `DOUBLE`|
|`BigDecimalTypeHandler`|`java.math.BigDecimal`|数据库兼容的 `NUMERIC` 或 `DECIMAL`|
|`StringTypeHandler`|`java.lang.String`|`CHAR`, `VARCHAR`|
|`ClobReaderTypeHandler`|`java.io.Reader`|-|
|`ClobTypeHandler`|`java.lang.String`|`CLOB`, `LONGVARCHAR`|
|`NStringTypeHandler`|`java.lang.String`|`NVARCHAR`, `NCHAR`|
|`NClobTypeHandler`|`java.lang.String`|`NCLOB`|
|`BlobInputStreamTypeHandler`|`java.io.InputStream`|-|
|`ByteArrayTypeHandler`|`byte[]`|数据库兼容的字节流类型|
|`BlobTypeHandler`|`byte[]`|`BLOB`, `LONGVARBINARY`|
|`DateTypeHandler`|`java.util.Date`|`TIMESTAMP`|
|`DateOnlyTypeHandler`|`java.util.Date`|`DATE`|
|`TimeOnlyTypeHandler`|`java.util.Date`|`TIME`|
|`SqlTimestampTypeHandler`|`java.sql.Timestamp`|`TIMESTAMP`|
|`SqlDateTypeHandler`|`java.sql.Date`|`DATE`|
|`SqlTimeTypeHandler`|`java.sql.Time`|`TIME`|
|`ObjectTypeHandler`|`Any`|`OTHER` 或未指定类型|
|`EnumTypeHandler`|`Enumeration Type`|`VARCHAR`-任何兼容的字符串类型, 存储枚举的名称(而不是索引)|
|`EnumOrdinalTypeHandler`|`Enumeration Type`|任何兼容的 `NUMERIC` 或 `DOUBLE` 类型, 存储枚举的索引(而不是名称). |

```java
//自定义泛型处理器
public class GenericTypeHandler<E extends MyObject> extends BaseTypeHandler<E> {

  private Class<E> type;

  public GenericTypeHandler(Class<E> type) {
    if (type == null) throw new IllegalArgumentException("Type argument cannot be null");
    this.type = type;
  }
}
```

### `<objectFactory>`对象工厂

* MyBatis每次创建结果对象的新实例时, 都会使用一个实例来完成.
* 默认的对象工厂只是实例化目标类, 通过默认构造方法或带参构造方法.
* 可以定义类继承`DefaultObjectFactory`来重写其行为, 并在`mybatis-config.xml`中使用`<object type="自定义类">`来使其生效

### `<plugins>`插件

* 插件用于在映射的执行语句执行过程中的某一点进行拦截
* 允许使用插件拦截的方法包括:
    - `Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)`
    - `ParameterHandler (getParameterObject, setParameters)`
    - `ResultSetHandler (handleResultSets, handleOutputParameters)`
    - `StatementHandler (prepare, parameterize, batch, update, query)`
* 定义类实现`Interceptor`接口, 然后使用注解`@Intercepts(@Signature(type=拦截的方法类.class, method="INSERT|DELETE|UPDATE|SELECT"), args=参数类.class)`, 然后在`mybatis-config.xml`中使用`<plugins>`标签, 在其内部定义`<plugin interceptor="实现类">`

### `<environments>`环境配置

* 可以通过该标签配置用于开发, 测试, 生产等不同的环境, 来连接不同的环境下的数据库
* 但是每种环境都必须对应一个`SqlSessionFactory`实例, 在创建SqlSessionFactory时可以将环境和参数配置进去
* `<environments>`标签用于声明环境配置
    - 内部的`<environment id="">`用于定义一种环境配置, 该环境配置用id来注明, id随意命名, 但不能重复
        - `<transactionManager type="事务管理器类型">`用于定义该环境下的事务管理器配置
            - `JDBC`: 直接使用JDBC来管理事务
            - `MANAGED`: 不提交和回滚, 而让容器来管理事务
            - **注意**: 如果使用Spring+MyBatis, 则没有必要配置事务管理器, 因为Spring模块会使用自带的事务管理器来覆盖这个配置
        - `<dataSource type="数据源类型">`用于定义数据源和连接属性
            - `UNPOOLED`: 不使用数据库连接池. 每次请求都要打开和关闭连接
                - `driver`属性: 驱动的完整类名
                - `url`属性: 数据库URL
                - `username`属性: 数据库用户名
                - `password`属性: 数据库密码
                - `defaultTransactionIsolationLevel`属性: 默认连接事务隔离级别
            - `POOLED`: 使用数据库连接池
                - `driver`属性: 驱动的完整类名
                - `url`属性: 数据库URL
                - `username`属性: 数据库用户名
                - `password`属性: 数据库密码
                - `defaultTransactionIsolationLevel`属性: 默认连接事务隔离级别
                - `poolMaximumActiveConnetions`: 连接池最大活动连接数, 默认10
                - `poolMaximumIdleConnections`: 连接池最大空闲连接数
                - `poolMaximumCheckoutTime`: 连接池最大被检出事件, 默认20000毫秒
                - `poolTimeToWait`: 连接超时时间, 默认20000毫秒
                - `poolPingQuery`: 在查询时Ping数据库检测连接是否正常, 默认是`NO PING QUERY SET`
                - `poolPingEnabled`: 是否启用Ping检测, 默认`false`
                - `poolPingConnectionsNotUsedFor`: 配置`poolPingQuery`的使用品读, 默认0
            - `JNDI`: 适合EJB等有JNDI上下文引用的容器
                - `initial_context`: InitialContext的上下文路径
                - `data_source`: 数据源实例的上下文路径

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

### `<databaseIdProvider>`数据库厂商标识

* 用于为不同数据库厂商执行不同的SQL语句

```xml
<databaseIdProvider type="DB_VENDOR">
  <property name="SQL Server" value="sqlserver"/>
  <property name="DB2" value="db2"/>        
  <property name="Oracle" value="oracle" />
</databaseIdProvider>
```

### `<mappers>`映射器

* 用来告诉MyBatis在哪里寻找定义的`XxxMapper.xml`
* `<mappers>`用于声明映射器配置
    - `<mapper>`用于定义一个映射路径
        - `resource`属性用于从包名下寻找XML映射文件
        - `url`属性用于从`file://`路径下寻找映射文件
        - `class`属性用于从包名下寻找Java映射类
    - `<package name="包名"/>`用于在指定的包下寻找所有类型的映射文件, 省去一个一个配置的步骤

### 小结

`mybatis-config.xml`:
* `<configuration>`
    - `<properties resource="config.properties">`: 用于设置配置文件, 配置文件中定义的键值对可用于xml配置中
        - 加载优先级: `<properties>标签 < config.properties < 方法参数`
        - `<property name="" value=""/>`: 属性键值对
            - 属性值的默认值
                - 从3.4.2开始, 可以为占位符设置一个默认值:
                    - `value=${username:default_user}`, 表示如果`username`不存在的话, 则使用`default_user`
                    - 该特性默认是关闭的, 需要使用`<property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/>`设置默认值开启和关闭
                - 默认值也可以使用三元运算符来表示
                    - `value=${username != null? username : default_user}`
    - `<settings>`: 配置mybatis运行时行为
        - `<setting name="" value=""/>`: 配置键值对
    - `<typeAliases>`: 类别别名, 为Java类型设置一个别名
        - `<typeAlias name="" value=""/>`: 配置别名
        - `<package name=""/>`: 指定JabaBean所在包名
    - `<typeHandlers>`: 类型处理器, 用于设置将获取的值转换为Java类型的方式
    - `<objectFactory>`: 对象工厂, 用于配置对象初始值
        - `<property name="" value=""/>`
    - `<plugins>`: 插件, 用于拦截方法调用
        - `<plugin interceptor=""></plugin>`: 拦截插件
    - `<environments>`: 环境配置, 用于区分多种环境, 如开发, 测试, 生产
        - `<environment id="development|test|release"></environment>`: 一种环境的配置
            - `<transactionManager type=""></transactionManager>`: 事务管理器配置
            - `<dataSource>`: 数据库连接池配置
    - `<databaseIdProvider>`: 数据库厂商设置
    - `<mappers>`: sql映射
        - `<mapper resource|url|class=""/>`: 配置映射资源位置
        - `<package name=""/>`


## Mapper XML映射文件

`XxxMapper.xml`:
    - `<cache>`: 给定命名空间的缓存配置
    - `<cache-ref>`: 其他命名空间缓存配置的引用
    - `<resultMap>`: 描述如何从数据库结果集中来加载对象
    - `<sql>`: 可被其他语句引用的可重用语句块
    - `<insert>`: 映射插入语句
    - `<update>`: 映射更新语句
    - `<delete>`: 映射删除语句
    - `<select>`: 映射查询语句

### `<cache>`标签

配置二级缓存

属性:
* `eviction`: 回收策略
    - `LRU`: 默认. 最近最少使用. 最长时间不被使用的被移除
    - `FIFO`: 先进先出. 先进先移除
    - `SOFT`: 软引用. 移除基于垃圾回收器状态和软引用规则的对象
    - `WEAK`: 弱引用. 更积极的移除基于垃圾回收器状态和弱引用规则的对象
* `flushInterval`: 缓存刷新间隔, 单位毫秒
* `size`: 默认1024, 缓存对象数量
* `readOnly`: 缓存是否只读, 默认false.
* `type`: 指定自定义缓存的完整类名

### `<select>`标签

示例:

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
    SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

解释:
* 该语句的id是`selectPerson`, 可以通过该id找到该语句
* 该语句接受一个int或Integer类型的参数
* 该语句返回一个HashMap类型的对象, Map的key是列名, value是记录值
* 参数使用`#{id}`表示, 它在JDBC中会被`?`来替换

Select标签属性:
* `id`: 命名空间中的唯一标识, 用于引用该语句
* `parameterType`: 可选. 传入语句的参数的完整类名或别名. 默认是`unset`, 如果定义该属性也是可以的, 因为MyBatis会使用`TypeHandler`来推断传入的类型
* ~~`parameterMap`~~: 已过时
* `resultType`: 语句返回的结果的完整类名或别名. 如果结果是集合, 则应该是集合中包含的元素的类型, 而不是集合的类型. 不能与resultMap同时使用
* `resultMap`: 外部resultMap的命名引用. 不能与resultType同时使用
* `flushCache`: 当true时, 调用任何语句都会使本地缓存和二级缓存清空. 默认值false
* `useCache`: 当true时, 语句的结果被而今缓存. 默认对select标签为true
* `timeout`: 等待数据库返回请求结果的秒数, 超时会抛出异常
* `fetchSize`: 设置每次批量返回的结果行数
* `statementType`: SQL语句类型
    - `STATEMENT`: 使用Statement
    - `PREPARED`: 默认值, 使用PreparedStatement
    - `CALLABLE`: 使用CallableStatement
* `resultSetType`: 结果集类型, 默认`unset`
    - `FORWARD_ONLY`:
    - `SCROLL_SENSITIVE`
    - `SCROLL_INSENSITIVE`
* `databaseId`: 用于配置了databaseIdProvider时加载不同数据库
* `resultOrdered`: 仅针对嵌套结果select语句. 默认false. true时假设包含了嵌套结果集或是分组了, 这样当金返回一个主结果行时, 不会发生对前面结果集引用的情况
* `resultSets`: 仅对多结果集使用. 将列出语句执行后返回的结果集, 并为每个结果集给一个名称, 名称是逗号分隔的

示例:

```xml
<select
    id="selectPerson"
    parameterType="int"
    parameterMap="deprecated"
    resultType="hashmap"
    resultMap="personResultMap"
    flushCache="false"
    useCache="true"
    timeout="10000"
    fetchSize="256"
    statementType="PREPARED"
    resultSetType="FORWARD_ONLY">

<!-- 如果列名和属性不匹配, 可以使用别名 -->
<select id="selectPerson" resultType="Persen">
    select
        user_id            as "id",
        user_name          as "userName",
        hashed_password    as "hashedPassword"
    from table
    where id=#{id}
</select>
```


### `<insert>`, `<update>`, `<delete>`标签

insert, update, delete非常接近
属性:
* `id`: 命名空间中的唯一标识
* `parameterType`: 可选. 传入语句参数的完整类名或别名. 如果不定义, MyBatis会通过`TypeHandler`推断传入参数的类型
* ~~`parameterMap`~~: 已过时
* `flushCache`: 当true时, 调用任何语句都会使本地缓存和二级缓存清空. 默认值false
* `timeout`: 等待数据库返回请求结果的秒数, 超时会抛出异常
* `statementType`: 语句类型
    - `STATEMENT`: 默认, 使用Statement
    - `PREPARED`: 使用PreparedStatement
    - `CALLABLE`: 使用CallableStatement
* `useGeneratedKeys`: 仅对insert和update有用. 让mybatis使用JDBC的`getGeneratedKeys`方法来取出由数据库内部生成的主键. 默认值false
* `keyProperty`: 仅对insert和update有用. 唯一标记一个属性. MyBatis会通过`getGeneratedKeys`方法的返回值或通过insert语句的selectKey子元素设置它的键值. 可以使用`,`分隔多个列明
* `keyColumn`: 仅对insert和update有用. 通过生成的键值设置表中的列名. 仅在PostgreSQL中必须
* `databaseId`: 用于配置了databaseIdProvider时加载不同数据库

示例:

```xml
<insert
    id="insertAuthor"
    parameterType="domain.blog.Author"
    flushCache="true"
    statementType="PREPARED"
    keyProperty=""
    keyColumn=""
    useGeneratedKeys=""
    timeout="20">

<update
    id="updateAuthor"
    parameterType="domain.blog.Author"
    flushCache="true"
    statementType="PREPARED"
    timeout="20">

<delete
    id="deleteAuthor"
    parameterType="domain.blog.Author"
    flushCache="true"
    statementType="PREPARED"
    timeout="20">
```

### `<sql>`标签

用于定义可复用的SQL代码段
* 可以作为sql语句段被其他sql语句引用
* 可以是不完整的sql, 此时要注意拼接时的空格问题, 不要缺少空格
* 可以包含参数

```xml
<!-- 定义一段sql, 可以是不完整的sql -->
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>

<!-- 引用定义过的sql作为整体sql语句的一部分 -->
<select id="selectUsers" resultType="map">
    select
        <include refid="userColumns">
            <property name="alias" value="t1"/>
        </include>,
        <include refid="userColumns">
            <property name="alias" value="t2"/>
        </include>
    from some_table t1
    cross join some_table t2
</select>
```

### `<resultMap>`标签和`resultType`属性

* 用于将结果集映射为Java类型
* `<resultMap>`标签并不需要明确写明, 它会自动创建
    - 例如`<select>`标签, 它通过`resultType`属性指定结果集映射的类型

```xml
<!-- 不明确的resultMap -->
<select id="selectUsers" resultType="map">
    select id, username, password
    from table
    where id=#{id}
</select>

<!-- 明确的resultMap -->
<resultMap id="userResultMap" type="User">
    <id property="id" column="user_id"/>
    <result property="username" column="user_name"/>
    <result property="password" column="hashed_password"/>
    <!-- 一对多, 集合 -->
    <collection property="friends" ofType="User">
        ...
    </collection>
</resultMap>
<select id="selectUsers" resultMap="userResultMap">
    select user_id, user_name, hashed_password
    from table
    where -d=#{id}
</select>
```

## 参数

2种格式:
* `#{参数名}`: 作为sql中的占位符`?`, 可以防止sql注入
* `${字符串}`: 作为字符串(没有引号), 不会转义为占位符


## 动态SQL

* 为了避免拼接SQL语句时空格和逗号的错误问题
* 采用OGNL实现
* 标签:
    - `if`: 条件判断, 没有else
    - `choose(when, otherwise)`: 相当于switch
    - `where`: 只有当内部if标签至少有一个成立时才会出现where语句
    - `trim`:
    - `set`: 用于在set语句中消除不必要的逗号
    - `foreach`: 遍历集合, 通常用于构建IN条件语句
    - `bind`: 创建一个变量并将其绑定到Context

```xml
<!-- if标签 -->
<select id="findBlog" resultType="Blog">
    SELECT * FROM BLOG
    WHERE state = 'ACTIVE'
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
</select>

<!-- choose(when, otherwise)标签, 类似于switch -->
<select id="findActiveBlogLike" resultType="Blog">
SELECT * FROM BLOG WHERE state = ‘ACTIVE’
    <choose>
        <when test="title != null">
            AND title like #{title}
        </when>
        <when test="author != null and author.name != null">
            AND author_name like #{author.name}
        </when>
        <otherwise>
            AND featured = 1
        </otherwise>
    </choose>
</select>

<!-- where标签 -->
<select id="findActiveBlogLike" resultType="Blog">
    SELECT * FROM BLOG
    <where>
        <if test="state != null">
            state = #{state}
        </if>
        <if test="title != null">
            AND title like #{title}
        </if>
        <if test="author != null and author.name != null">
            AND author_name like #{author.name}
        </if>
    </where>
</select>

<!-- trim标签 -->
<trim prefix="WHERE" prefixOverrides="AND |OR ">
    ...
</trim>

<!-- set标签 -->
<update id="updateAuthorIfNecessary">
    update Author
        <set>
            <if test="username != null">username=#{username},</if>
            <if test="password != null">password=#{password},</if>
            <if test="email != null">email=#{email},</if>
            <if test="bio != null">bio=#{bio}</if>
        </set>
    where id=#{id}
</update>

<!-- foreach标签 -->
<select id="selectPostIn" resultType="domain.blog.Post">
    SELECT *
    FROM POST P
    WHERE ID in
    <foreach item="item" index="index" collection="list" open="(" separator="," close=")">
        #{item}
    </foreach>
</select>

<!-- bind标签 -->
<select id="selectBlogsLike" resultType="Blog">
    <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
    SELECT * FROM BLOG
    WHERE title LIKE #{pattern}
</select>
```


## Java API

* `SqlSessionFactoryBuilder`类
    - `SqlSessionFactory build()`: 创建工厂
* `SqlSessionFactory`类
    - `SqlSession openSession()`: 创建SqlSession
    - `Configuration getConfiguration()`: 获取配置
* `SqlSession`类
    - 语句执行
        - `T selectOne(String statement, Object param)`: 查询一条记录, 返回一个对象. 如果多余一个或没有返回, 则抛出异常
        - `List<E> selectList(String statement, Object param)`: 查询多条记录, 返回一个List集合
        - `List<E> selectList(String statement, Object param, RowBounds rowBounds)`: 通过RowBounds对象指定略过的记录数量和限制的返回结果的数量
        - `Map<K, V> selectMap(String statement, Object param, String mapKey)`: 查询多条记录, 返回Map集合
        - `Map<K, V> selectMap(String statement, Object param, String mapKey, RowBounds rowBounds)`: 通过RowBounds对象指定略过的记录数量和限制的返回结果的数量
        - `void select(String statement, Object param, ResultHandler<T> handler)`: 使用ResultHandler自定义结果集的处理方式
        - `void select(String statement, Object param, RowBounds bounds, ResultHandler<T> handler)`: 结合RowBounds和ResultHandler使用
        - `int insert(String statement, Object param)`: 插入记录
        - `int update(String statement, Object param)`: 更新记录
        - `int delete(String statement, Object param)`: 删除记录
    - 批量立即更新
        - `List<BatchResult> flushStatements()`: 立即执行批量更新语句
    - 事务控制
        - `void commit()`: 提交事务. 默认不会自动提交, 除非侦测到有插入更新或删除
        - `void commit(boolean force)`: 强制提交事务
        - `void rollback()`: 回滚事务
        - `void rollback(boolean force)`: 强制回滚
    - 清理session级别缓存
        - `void clearCache()`: 清理session级别缓存
    - 关闭SqlSession
        - `void close()`: 关闭SqlSession. 最好放在`finally`语句块中保证其调用
    - 映射器
        - `T getMapper(Class<T> type)`: 获取指定类型的映射器

## 映射器注解

* MyBatis3开始, 支持注解配置映射.



## SQL语句构造器

* MyBatis提供了便捷的SQL语句构造器, 避免复杂sql在编写时的错误
* 使用`SQL类`进行构建

```java
private String selectPersonSql() {
    return new SQL() {{
        SELECT("P.ID, P.USERNAME, P.PASSWORD, P.FULL_NAME");
        SELECT("P.LAST_NAME, P.CREATED_ON, P.UPDATED_ON");
        FROM("PERSON P");
        FROM("ACCOUNT A");
        INNER_JOIN("DEPARTMENT D on D.ID = P.DEPARTMENT_ID");
        INNER_JOIN("COMPANY C on D.COMPANY_ID = C.ID");
        WHERE("P.ID = A.ID");
        WHERE("P.FIRST_NAME like ?");
        OR();
        WHERE("P.LAST_NAME like ?");
        GROUP_BY("P.ID");
        HAVING("P.LAST_NAME like ?");
        OR();
        HAVING("P.FIRST_NAME like ?");
        ORDER_BY("P.ID");
        ORDER_BY("P.FULL_NAME");
    }}.toString();
}
```
