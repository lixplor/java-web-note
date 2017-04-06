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
    - `mapper.xml`: sql映射, 配置操作数据库的sql语句, 需要在`mybatis-config.xml`中配置
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
        - xml方式: `JavaBean类名.xml`
        - 注解方式

```xml
mybatis-config.xml
----------------
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 参数 -->
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

`mybatis-config.xml`:
* `<configuration>`
    - `<properties resource="config.properties">`: 用于设置配置文件, 配置文件中定义的键值对可用于xml配置中
        - 加载优先级: `<properties>标签 < config.properties < 方法参数`
        - `<property name="" value=""/>`: 属性键值对
            - 默认值
                - 从3.4.2开始, 可以设置默认值: `value=${username:default_user}`
                - 使用`<property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/>`设置默认值开启和关闭
    - `<settings>`: 配置mybatis运行行为
        - `<setting name="" value=""/>`: 配置键值对. 具体属性见官网
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
* 该语句返回一个HashMap类型的对象
* 参数使用`#{id}`表示, 它在JDBC中会被`?`来替换

Select标签属性:
* `id`: 命名空间中的唯一标识, 用于引用该语句
* `parameterType`: 可选. 传入语句的参数的完整类名或别名
* `resultType`: 语句返回的结果的完整类名或别名. 如果结果是集合, 则是集合可以包含的类型. 不能与resultMap同时使用
* `resultMap`: 外部resultMap的命名引用. 不能与resultType同时使用
* `flushCache`: 当true时, 调用任何语句都会使本地缓存和二级缓存清空. 默认值false
* `useCache`: 当true时, 语句的结果被而今缓存. 默认对select标签为true
* `timeout`: 等待数据库返回请求结果的秒数, 超时会抛出异常
* `fetchSize`: 设置每次批量返回的结果行数
* `statementType`: 语句类型
    - `STATEMENT`: 默认, 使用Statement
    - `PREPARED`: 使用PreparedStatement
    - `CALLABLE`: 使用CallableStatement
* `resultSetType`: 结果集类型
    - `FORWARD_ONLY`
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
* `parameterType`: 可选. 传入语句参数的完整类名或别名
* `flushCache`: 当true时, 调用任何语句都会使本地缓存和二级缓存清空. 默认值false
* `timeout`: 等待数据库返回请求结果的秒数, 超时会抛出异常
* `statementType`: 语句类型
    - `STATEMENT`: 默认, 使用Statement
    - `PREPARED`: 使用PreparedStatement
    - `CALLABLE`: 使用CallableStatement
* `useGeneratedKeys`: 仅对insert和update有用. 让mybatis使用JDBC的`getGeneratedKeys`方法来取出由数据库内部生成的主键
* `keyProperty`: 仅对insert和update有用. 唯一标记一个属性.
* `keyColumn`: 仅对insert和update有用. 通过生成的键值设置表中的列明
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

* MyBatis3开始, 支持注解配置映射. 但是不如xml强大, 而且java在注解方面也有些限制

参见[官方文档](http://www.mybatis.org/mybatis-3/zh/java-api.html)


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
