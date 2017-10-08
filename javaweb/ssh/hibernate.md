# Hibernate

* 持久层ORM框架, 操作数据库
* 简化了开发, 提高效率
* 提供额外功能, 如缓存
* 基本概念:
    - session: 与数据库的会话. 
        - session与connection的关系: session与connection是多对一关系, 每个session都有一个与之对应的connection, 一个connection不同时刻可以供多个session使用

## 环境搭建

### 下载

* 官网: [http://hibernate.org/](http://hibernate.org/)

### 添加依赖包

* jar包方式
    - 在`WEB-INF/lib`下, 加入Hibernate的`require/*`, `log4j`, `slf4j`
    - 注意hibernate依赖于Log4J, slf4j, 不要忘记导包
* maven方式

```xml
<dependencies>
    <!-- Hibernate -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>5.2.11.Final</version>
    </dependency>

    <!-- Log4J -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.16</version>
    </dependency>

    <!-- SLF4J -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-nop</artifactId>
        <version>1.6.4</version>
    </dependency>
<dependencies>
```


## 配置文件

* 对象关系映射文件: 创建与JavaBean同名的xml, 如`User.hbm.xml`(hbm = hibernate mapping)
    - `<hibernate-mapping>`: 根标签
        - `<class>`标签: 将类与数据库表建立映射关系
            - 属性
                - `name`: 类的完整路径
                - `table`: 表名(如果类名和表名一致, 则可以省略该属性)
                - `catalog`: 数据库名称(如果配置了则可以省略)
            - 内部标签
                - `<id>`标签: 将类中的属性与表中的主键建立映射, id标签用于配置主键
                    - 属性
                        - `name`: 属性名
                        - `column`: 表中字段名(如果类中属性名和表中字段名一致, 则可以省略)
                        - `length`: 字段的长度, 如果数据库已经建好, 则可以省略. 如果没有建好数据库, 最好指明
                        - `type`: 指定类型
                    - 内部标签
                        - `<generator>`: 主键生成策略
                            - `increment`: 代理主键. 由hibernate维护一个变量, 每次生成主键时自动递增. 适用于short, int, long类型的主键
                            - `identity`: 代理主键. 由底层数据库生成标识符. 需要数据库支持自动增长, 如MySQL. 适用于short, int, long
                            - `sequence`: 代理主键. 根据底层数据库序列生成标识符. 需要数据库支持序列, 如Oracle
                            - `native`: 代理主键. 根据底层数据库自动选择identity, sequence, hilo. 不建议使用
                            - `uuid`: 代理主键. 采用128位UUID算法生成标识符. 建议使用
                            - `assigned`: 自然主键. 由Java程序生成标识符, 即手动设置
                - `<property>`标签: 将类中的普通属性与表中的字段建立映射
                    - `name`: 类中属性名
                    - `column`: 表中字段名
                    - `length`: 数据长度
                    - `type`: java/hibernate/SQL中的数据类型
                        - Hibernate: `string`
                        - Java: `java.lang.String`
                        - SQL: `VARCHAR`
                        - 默认是hibernate中的数据类型
                    - `sql-type`: sql数据类型

```xml
<?xml version"1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping ...>
<hibernate-mapping>
    <!-- 配置类和表的映射 -->
    <class name="com.package.User" table="tb_user">
        <!-- 配置id, name是JavaBean属性, column是对应的表的字段 -->
        <id name="id" column="id">
            <!-- 主键策略 -->
            <generator class="native"/>
        </id>
        <!-- 配置其他属性 -->
        <property name="name" column="name" type="string" length="25"/>
    </class>
</hibernate-mapping>
```

* Hibernate核心配置文件: `hibernate.cfg.xml`
    - `<hibernate-configuration>`: 配置根标签
        - `<session-factory>`: 数据库连接配置, 一个数据库对应一个标签
            - `<property>`: 配置项标签
                - `name`属性值:
                    - `hibernate.connection.driver_class`: 数据库连接驱动类全名
                    - `hibernate.connection.url`: 数据库连接URL
                    - `hibernate.connection.username`: 数据库连接用户名
                    - `hibernate.connection.password`: 数据库连接密码
                    - `hibernate.dialect`: 数据库连接使用的方言类
                        - MySQL使用`org.hibernate.dialect.MySQLDialect`
                    - `hibernate.show_sql`: 是否在控制台打印执行的SQL语句
                        - true
                        - false
                    - `hibernate.format_sql`: 是否格式化SQL语句
                        - true
                        - false
                    - `hibernate.hbm2dl.auto`: 生成数据表结构策略
                        - `create-drop`: 重新创建表后再删除
                        - `create`: 重新创建表, 一般在测试时使用
                        - `update`: 如果数据库中有表则不创建, 如果没有表则创建. 如果映射不匹配则更新已存在的表(开发中用这个)
                        - `validate`: 只使用已存在的表, 并校验JavaBean和表字段是否一致
            - `<mapping>`: 映射配置文件, 一个配置文件对应一个标签
                - `resource`属性值: 配置文件全类名(格式: `包/名/配置文件名.hbm.xml`)
            

```xml
<?xml version"1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <!-- 必须先配置SessionFactory标签, 一个数据库对应一个sessionFactory标签 -->
    <session-factory>
        <!-- 必须要配置的参数有5个: 4大连接参数, 1个数据库的方言 -->
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql:///xxx</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">123456</property>
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- 配置数据库连接池 -->
        <property name="hibernate.connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.timeout">120</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>

        <!-- 可选配置 -->
        <!-- 在控制台打印单行sql语句 -->
        <property name="hibernate.show_sql">true</property>
        <!-- 在控制台打印格式化的sql语句 -->
        <property name="hibernate.format_sql">true</property>
        <!-- 生成数据库表结构 create-drop, create, update, validate -->
        <property name="hibernate.hbm2dl.auto">update</property>
        <!-- 设置事务是否自动提交 true自动提交, false手动提交-->
        <property name="hibernate.connection.autocommit">true</property>

        <!-- 映射配置文件, 需要引入映射的配置文件 -->
        <mapping resource="com/package/User.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```

## 编写代码

### 创建JavaBean

* 创建JavaBean(可选): 如`User.java`
    - 基本类型使用包装类, 因为默认值是`null`, 有助于判断
    - xml也可以自动生成JavaBean

### 编写代码使用hibernate

* 使用步骤
    - 加载配置文件
        - 此步骤会自动寻找`hibernate.cfg.xml`, 并读取其中的`<mapping>`寻找映射
    - 创建SessionFactory对象, 生成Session对象
    - 开启事务
    - 编写sql代码
    - 提交事务
    - 释放资源
* Hibernate相关API
    - `Configuration`类: 用于加载配置文件
        - `Configuration Configuration()`
        - `void config()`: 加载默认名称的配置文件
        - `void config(String filename)`: 加载指定名称的配置文件
    - `SessionFactory`接口: 初始化Hibernate, 作为数据源代理, 创建Session对象. 一个项目通常只需要一个SessionFactory, 当有多个数据库时, 可以为每个数据库指定一个SessionFactory
        - `Session openSession()`: 获取session
        - `Session getCurrentSession()`: 获取当前线程中ThreadLocal保存的的Session, 用于session与ThreadLocal绑定的情况
    - `Session`接口: 负责CRUD, 调用时创建, 用完后销毁. 非线程安全
        - `save(Object obj)`: 添加记录
        - `saveOrUpdate(Object obj)`: 添加或更新已有记录
        - `delete(Object obj)`: 通过JavaBean的主键删除指定记录
        - `update(Object obj)`: 通过JavaBean的主键更新指定记录
            - 注意更新时, 要先查询记录, 然后更新查询出来的记录
        - `get(Class clazz, int id)`: 通过主键查询记录
        - `SQLQuery createSQLQuery(String sql)`: 获取可以操作SQL的SQLQuery对象
        - `Query createQuery(String hql)`: HQL查询并获取Query对象
            - 条件查询:
                - `from JavaBean where field > ?`
                - `from JavaBean where field > :变量占位符`
        - `Criteria createCriteria(Class c)`: Criteria条件查询
        - `Transaction beginTransaction()`: 开启事务, 并获取事务对象
    - `Query`接口: 查询数据库. 可以使用HQL或SQL
        - `void setFirstResult(int offset)`: 设置开始索引
        - `void setMaxResults(int size)`: 设置本次查询返回的条数
        - `void setParameter(String key, Object value)`: 设置参数值
        - `void setInteger(index, value)`: 设置hql占位符位置的值
        - `void setString(index, value)`: 设置hql占位符位置的值
        - `T uniqueResult()`: 查询唯一结果, 可以映射为一个JavaBean对象
        - `List<T> list()`: 将查询结果转换为List集合
    - `SQLQuery`接口: 执行SQL
        - `void addEntity(Class c)`: 将查询结果封装到指定类的对象中
        - `void setParameter(String key, Object value)`: 设置参数值
        - `T uniqueResult()`: 查询唯一结果, 可以映射为一个JavaBean对象
        - `List<T> list()`: 将查询结果转换为List集合
    - `Criteria`接口: 条件查询
        - `Criteria add(Criterion c)`: 添加查询条件
        - `List<T> list()`: 按照Criteria对象内部的条件执行条件查询
    - `Restrictions`类: 工具类, 用于创建Criterion
        - `Criterion eq(String key, Object value)`: 等于
        - `Criterion gt(column, value)`: 大于
        - `Criterion ge()`: 大于等于
        - `Criterion lt()`: 小于
        - `Criterion le()`: 大于等于
        - `Criterion between()`: 介于
        - `Criterion like()`: 模式匹配
        - `Criterion in()`: 在列表中
        - `Criterion and()`: 条件与
        - `Criterion or(Criterion... c)`: 条件或
        - `Criterion isNull()`: 为空
        - `List<T> list()`: 查询所有记录的所有字段
    - `Projections`类: 投影查询
        - `static ProjectionList projectionList()`: 创建一个投影列表
        - `static Projection sum(String propertyName)`: 求和
        - `static Projection max(String propertyName)`: 求最大值
        - `static Projection min(String propertyName)`: 求最小值
        - `static Projection avg(String propertyName)`: 求平均值
        - `static Projection count(String propertyName)`: 计数
    - `ProjectionList`类: 投影列表, 可以添加聚合条件
        - `ProjectionList add(Projection p)`: 添加一个投影聚合条件
    - `Transaction`接口: 管理实务
        - `commit()`: 提交事务
        - `rollback()`: 回滚事务
* 注意: 
    - Hibernate默认不会自动提交事务, 必须手动提交事务
    - 如果没有开启事务, 那么Session的每个操作都作为一个单独的事务


```java
// 加载配置文件
Configuration config = new Configuration();
config.configure();  // 默认加载class目录下的hibernate.cfg.xml
// 创建SessionFactory
SessionFactory factory = config.buildSessionFactory();
// 创建Session对象
Session session = factory.openSession();
// 开启事务
Transaction tr = session.beginTransaction();
// 执行数据库操作
session.save(user);
// 提交事务
tr.commit();
// 释放资源
session.close();
factory.close();
```


## 持久化

### 持久化类

* 持久化类: 
    - JavaBean与表建立了映射关系, 即`JavaBean + JavaBean.hbm.xml`
* 编写规则:
    - 提供一个无参数的public构造方法(用于反射)
    - 提供一个标识属性, 映射数据表主键字段(唯一标识OID, 用于确定表中记录)
    - 所有属性提供public的setter和getter
    - 标识属性尽量使用包装类型

### 持久化对象的三个状态

* 瞬时态(transient): 也叫临时态或自由态
    - 没有持久化标识OID
    - 没有被Session管理
    - 如: Java代码中创建的对象, 而不是数据库查询出的, 一般只用来传递信息, 使用完毕后会被JVM回收
* 持久态(persistent)
    - 有持久化标识OID
    - 有被Session管理
    - 如: Java代码中创建的对象, 使用session.save()保存到了缓存中. 数据库中可能有也可能没有
* 脱管态(detached): 也叫游离态或离线态, 指持久态对象失去了与session的关联
    - 有持久化标识OID
    - 没有被Session管理
    - 如: Java代码中创建的对象, 使用session.save()保存到了缓存中, 提交了事务, 关闭了会话session.close(), session不再管理, 但存在OID. 数据库种可能有也可能没有
* 判断状态的依据:
    - 是否有OID
    - 判断是否与session关联
* 三种状态的获取和脱离:
    - 瞬时态:
        - 获取:
            - new对象
        - 脱离:
            - 垃圾回收
    - 持久态
        - 获取:
            - `get()`
            - `load()`
            - `find()`
            - `iterate()`
            - 等等
        - 脱离:
            - 垃圾回收
    - 脱管态:
        - 获取:
            - 无法直接获取
        - 脱离:
            - 垃圾回收
* 三种状态的转换:
    - 瞬时态:
        - 瞬时态 -> 持久态
            - `save()`
            - `saveOrUpdate()`
        - 瞬时态 -> 脱管态:
            - 手动设置OID
    - 持久态:
        - 持久态 -> 瞬时态
            - `delete()`
        - 持久态 -> 脱管态
            - `evict()`: 清除一级缓存中指定的一个对象
            - `close()`: 清空以及缓存
            - `clear()`: 关闭, 清空一级缓存
    - 脱管态:
        - 脱管态 -> 瞬时态
            - 将OID删除
        - 脱管态 -> 持久态
            - `update()`
            - `saveOrUpdate()`
            - `lock()`



```
+------+    new    +--------------------+
| 对象 |  ------->  | 瞬时态(Transient)  | -----------------+
+------+           +--v--------------^--+                  |
   |                  |              |                     |
   get()              save()         delete()              |
   load()             saveOrUpdate() |                     |
   find()             |              |                     |
   iterate()          |              |                     v
   |               +--v--------------^--+                  垃圾回收
   +------------>  | 持久态(Persistent)  |                  ^
                   +--v--------------^--+                  |
                      |              |                     |
                      evict()        update()              |
                      close()        saveOrUpdate()        |
                      clear()        lock()                |
                      |              |                     |
                   +--v--------------^--+                  |
                   | 脱管态(Detached)    | -----------------+
                   +--------------------+
```


## 缓存

* 缓存: 内存中存储数据的空间, 用于提升获取数据的效率
* 一级缓存: 自带, 不可取消. 生命周期与session一致, 是session级别的缓存. 属于事务缓存
* 二级缓存: 默认没有开启, 需要配置开启. 二级缓存可以再多个session中共享数据, 是sessionFactory级别的缓存. 属于进程缓存

### 一级缓存: Session

* Session中定义了一系列的集合来存储数据, 构成session缓存
    - `private transient ActionQueue actionQueue;`: 一个队列, 用来记录crud操作
    - `private transient StatefulPersistenceContext persistenceContext;`: 真正的缓存
* 作用:
    - 当通过hibernate种的session调用其API如save, get, update操作后, 会将持久化对象保存到session中, 下次查询就会先查缓存中是否有数据(通过OID判断), 如果有则不会查询数据库. 达到减少数据库访问的目的
* Session
    - `clear()`: 清空session缓存
    - `evict(Object entity)`: 从session缓存中清除指定的实体对象
    - `refresh()`: 重新查询数据库, 并使用查询结果更新session缓存和快照
    - `update()`: 针对脱管对象
    - `saveOrUpdate()`: 如果对象是瞬时态则执行save操作, 如果对象是脱管态则执行update操作, 如果对象是持久态则直接返回
    - `delete()`: 删除脱管对象, 先删除session缓存, 再删除数据库中的数据
* Session的快照机制
    - 持久态根据缓存更新的原理
        - session操作后, 缓存区和快照区都会更新数据
        - 当修改数据后, 缓存区的数据会更新
        - commit之前, 会对比缓存区和快照区数据是否一致
            - 如果一致, 正常
            - 如果不一致, 则修改数据库中的值, 并更新快照区数据

### 二级缓存: SessionFactory

* 二级缓存分为2种:
    - 内置缓存: Hibernate自带的SessionFactory. 数据是只读的
    - 外置缓存: 第三方缓存插件, 缓存介质可以是内存或硬盘
* Hibernate支持的外置缓存插件
    - EHCache
    - OpenSymphony
    - SwarmCache
    - JBossCache



## 多表操作

### 建表原则

* 一对一
    - 唯一外键对应方式: 在任意一方添加UNIQUE的外键关联另一方的主键
    - 主键对应方式: 一方的主键作为另一方的主键, 即两个表的主键值相同
* 一对多
    - 在多方添加外键关联一方主键
* 多对多
    - 创建中间表, 分别添加两个多方的外键列, 并分别关联两个多方的主键列


### 一对多操作

* JavaBean的定义
    - 一方对多方的持有, 使用存储多方对象的集合表示
        - hibernate中集合默认使用`Set`, 并且必须手动初始化
    - 多方对一方的持有, 使用一方对象表示
    
```java
// 一方
public class User {
    private Integer id;
    private Set<Contactor> contactors = new HashSet<>();  // 对多, 必须手动初始化
}
```

```java
// 多方
public class Contactor {
    private Integer id;
    private User user;   // 对一
}
```

* hbm映射配置文件

```xml
<!-- 一方 -->
<set name="多方属性名">
    <key column="外键字段"/>
    <one-to-many class="多方完整类名"/>
</set>
```

```xml
<!-- 多方 -->
<many-to-one name="一方属性名" class="一方类完整路径" column="外键字段"/>
```

### 多对多操作

* JavaBean的定义

```java
// 多方
public class User {
    private Integer id;
    private Set<Contactor> contactors = new HashSet<>();  // 对多, 必须手动初始化
}
```

```java
// 多方
public class Contactor {
    private Integer id;
    private Set<User> users = new HashSet<>();  // 对多, 必须手动初始化
}
```

* hbm配置文件

```xml
<!-- 多 -->
<set name="多方类属性" table="多方表名">
    <key column="当前对象在中间表的外键名称"/>
    <many-to-many class="集合存入对象的完整路径" column="集合中对象在中间表的外键的字段名称"/>
</set>

<!-- 另一个多, 同上 -->
<set name="多方类属性" table="多方表名">
    <key column="当前对象在中间表的外键名称"/>
    <many-to-many class="集合存入对象的完整路径" column="集合中对象在中间表的外键的字段名称"/>
</set>
```

### 多表关系下的级联操作

* 在hbm配置文件中, `cascade`属性, 可以设置级联操作的方式. `inverse`属性可以设置对外键的维护方
    - `cascade`属性可以使用多个值, 用逗号分隔: `cascade="save-update, delete"`
        - `none`: 不使用级联
        - `save-update`: 级联保存或更新. 如果是游离态对象则会执行update
        - `delete`: 级联删除
        - `delete-orphan`: 孤儿删除(只能应用在一对多关系)
        - `all`: 除了delete-orphan的所有情况(包含save-update, delete)
        - `all-delete-orphan`: 包含了delete-orphan的所有情况(包含save-update, delete, delete-orphan)
    - `inverse`属性可以设置级联操作中外键的维护
        - 作用: 只在双向关联的情况下有效, 用于确定由哪方来维护外键. 通常用于避免产生多余数据
        - `<set>`标签加上`inverse="true"`属性, inverse用于维护外键
            - `true`: 放弃, 由对方来维护外键
            - `false`: 默认值, 不放弃, 由本方来维护外键
        - 注意在多对多关系时, 必须要有一方放弃外键的维护
* 保存数据的方式:
    - 手动双向保存
        - 一表和多表的数据都保存
    - 单向级联保存(有方向性)
        - 保存一表, 级联保存多表
            - 在`<set>`标签中增加`cascade="save-update"`属性
            - 这时只存一表, 即可自动保存多表
        - 或者保存多表, 级联保存一表
            - 在`<many-to-one>`标签中增加`cascade="save-update"`属性
            - 这时只存多表, 即可自动保存多表
    - 双向级联保存
        - 一表和多表都增加`cascade`属性
        - 一般都设置双向关联
* 删除数据的方式:
    - 手动删除: 有外键引用时, 必须先把外键设置为空, 才能删除
    - 级联删除(有方向性)
        - 使用`cascade="delete"`属性, 级联删除
        - 或使用`cascade="delete-orphan"`属性, 删除与当前对象解除关系的对象 
    - 孤儿删除
        - 一对多关系中, 将一方认为是父方, 将多方认为是子方. 在解除了父子关系的时候, 将子方记录删除


## 注解

* 注解用于简化xml配置文件
* 使用注解需要在`hibernate.cfg.xml`中进行配置
    - 配置PO类注解: `<mapping class="com.package.ClassName"/>`
* PO类注解
    - `@Entity`: 声明一个实体
        - 注解类
    - `@Table`: 声明表名和类名的对应
        - 注解类
        - 如: `@Table(name="表名", catalog="")`
    - `@Id`: 声明一个属性是主键
        - 注解属性
    - `@GeneratedValue`: 声明主键生成策略
        - 注解主键属性
        - 如: `@GeneratedValue(strategy=GenerationType.IDENTITY)`, 相当于`native`
    - `@GenericGenerator`: 声明主键生成器
        - 注解主键属性
        - 如: `@GenericGenerator(name="自定义生成器名称", strategy="uuid")`, 然后使用`@GeneratedValue(generator="自定义的生成器名称")`
    - `@Column`: 声明列与属性的对应
        - 注解属性
        - 如: `@Column(name="列名", length=长度, nullable=true)`
        - 注意: 即使不使用该注解, 也会按照类的属性生成映射关系
    - `@JoinColumn`: 声明外键列
        - 注解属性
        - 如: `@JoinColumn(name="外键列名")`
    - `@Temporal`: 声明属性对应的数据库中的日期类型
        - 注解属性
        - 如: `@Temporal(TemporalType.TIMESTAMP)`
    - `@OneToOne`: 声明一对一关系
        - 注解属性
        - 如: `@OneToOne(targetEntity=另一方类.class, mappedBy="维护外键一方的属性名")`
    - `@OneToMany`: 在一方类中声明一对多关系
        - 注解属性
        - 如: `@OneToMany(targetEntity=集合中多方类.class, mappedBy="维护外键一方的属性名")`
    - `@ManyToOne`: 在多方类中声明多对一关系
        - 注解属性
        - 如: `@ManyToOne(targetEntity=一方类.class)`
        - 注意: 多对一不存在`mappedBy`属性
    - `@ManyToMany`: 声明多对多关系
        - 注解属性
        - 如: `@ManyToMany(targetEntity=集合中多方类.class, mappedBy="维护外键一方的属性名")`
    - `@JoinTable`: 声明多对多关系中的中间表映射关系
        - 注解属性
        - 如: `@JoinTable(name="中间表名", joinColumns={@JoinColumn(name="当前类的表在中间表的外键字段名")}, inverseJoinColumns={@JoinColumn(name="对方类的表在中间表的外键字段名")})`
    - `@Cascade`: 声明级联操作方式
        - 注解属性
        - 如: `@Cascade(CascadeType.SAVE_UPDATE)`
    - `@PrimaryKeyJoinColumn`: 声明一对一使用主键对应方式
        - 注解属性
    - `@NamedQuery`: 声明预先定义的HQL语句
        - 注解类
        - 如: `@NamedQuery(name="自定义名称", query="HQL语句")`
    - `@NamedNativeQuery`: 声明预先定义的SQL语句
        - 注解类
        - 如: `@NamedNativeQuery(name="自定义名称", query="SQL语句", resultMapping="自定义SQL查询结果集映射名称")`
    - `@SqlResultSetMapping`: 声明SQL查询结果集映射到哪个类
        - 注解类
        - 如: `@SqlResultSetMapping(name="userSetMapping", entities={@EntityResult(entityClass=User.class, fields={@FieldResult(name="id", column="id"), @FieldResult(name="name", column="name")})})`


* 一对一示例: 唯一外键对应

```java
// 一方实体类
@Entity
@Table(name = "t_user")
public class User {

    @Id   // 指定为主键
    @GenericGenerator(name = "uuidGen", strategy = "uuid")  // 设置主键生成器, 使用uuid作为主键值
    @GeneratedValue(generator = "uuidGen")  // 主键生成策略使用自定义的uuid生成器
    private String id;

    private String name;

    @OneToOne(targetEntity = IDCard.class, mappedBy = "user")  // 设置一对一关系, 外键由user维护
    private IDCard idCard;
}
```

```java
// 一方实体类
@Entity
@Table(name = "t_idcard")
public class IDCard {

    @Id   // 指定为主键
    @GenericGenerator(name = "uuidGen", strategy = "uuid") // 设置主键生成器, 使用uuid作为主键值
    @GeneratedValue(generator = "uuidGen")  // 主键生成策略使用自定义的uuid生成器
    private String id;

    private String cardNum;

    @OneToOne(targetEntity = User.class)  // 设置一对一关系
    @JoinColumn(name = "c_user_id")  // 设置对应的外键字段名
    @Cascade(CascadeType.SAVE_UPDATE)  // 级联策略
    private User user;
}
```

* 一对一示例: 主键对应

```java
// 一方实体类
@Entity
@Table(name = "t_user")
public class User {

    @Id   // 指定为主键
    @GenericGenerator(strategy = GenerationType.IDENTITY) 
    private String id;

    private String name;

    @OneToOne
    @PrimaryKeyJoinColumn  // 设置主键映射
    private IDCard idCard;
}
```

```java
// 一方实体类
@Entity
@Table(name = "t_idcard")
public class IDCard {

    @Id   // 指定为主键
    @GenericGenerator(name = "foreignKeyGen", strategy = "foreign", parameters = {@Parameter(name = "property", value = "user")}) // 设置主键生成器, 使用user的主键
    @GeneratedValue(generator = "foreignKeyGen")  // 主键生成策略使用自定义的生成器
    private String id;

    private String cardNum;

    @OneToOne
    @PrimaryKeyJoinColumn  // 设置主键映射
    private User user;
}
```

* 一对多示例

```java
// 一方实体类
@Entity
@Table(name = "t_user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTIFY)
    private Integer id;

    private String name;

    @OneToMany(targetEntity = Order.class, mappedBy = "user")
    private Set<Order> orders = new HashSet<>();
}
```

```java
// 多方实体类
@Entity
@Table(name = "t_order")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTIFY)
    private Integer id;

    private String name;

    @ManyToOne(targetEntity = User.class)
    private User user;
}
```

* 多对多示例

```java
// 多方表实体类
@Entity
@Table(name="t_student")
public class Student {

    @Id  // 该字段为主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // 主键生成策略为native
    private Integer id;

    private String name;

    @ManyToMany(targetEntity = Subject.class, mappedBy = "students") // 多对多映射, 外键由对方维护
    private Set<Subject> subjects = new HashSet<>();
}
```

```java
// 多方表实体类
@Entity
@Table(name="t_subject")
public class Subject {

    @Id  // 该字段为主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // 主键生成策略为native
    private Integer id;

    private String name;

    @ManyToMany(targetEntity = Student.class) // 多对多映射, 外键由对方维护
    @JoinTable(name="t_student_subject_mapping", joinColumns={@JoinColumn(name="c_subject_id")}, inverseJoinColumns={@JoinColumn(name="c_student_id")})
    @Cascade(CascadeType.SAVE_UPDATE)
    private Set<Student> students = new HashSet<>();
}
```


## 查询

* Hibernate提供的查询方式:
    - 唯一标识(OID)查询方式: 按照对象的OID查询
        - `User user = session.get(User.class, 2);`
        - `User user = session.load(User.class, 2);`
    - 对象图导航(OGNL)查询方式: 根据已加载的对象导航到其他对象(通过一个对象获取其关联的另一个对象)
        - `Order order = user.getOrders().get(0)`, 通过User对象找到Order对象
    - HQL查询方式(推荐): 面向对象的HQL查询语言
        - Hibernate Query Language, 与SQL类似, 但更加面向对象
        - 步骤:
            - 获取session
            - 编写HQL语句
            - 调用`session.createQuery(hql)`创建`Query`对象
            - 调用`Query`对象的`setParameter(String key, Object value)`设置查询条件参数
            - 调用`list()`返回所有查询结果, 调用`uniqueResult()`返回一个查询结果
        - 语法: 
            - HQL语句格式:
                - `SELECT/UPDATE/DELETE ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... ASC/DESC`
            - 命名参数:
                - 格式: `:参数名`
                - 作用: 作为参数占位符, 有名字可以使用
                - 设置参数值: `query.setParameter(key, value)`
            - 位置参数:
                - 格式: `?`
                - 作用: 作为参数占位符, 只能通过索引设置参数
                - 设置参数值: `query.setString(0, "hello")`
        - 查询方式
            - 链式调用
                - `session.createQuery("from Customer").list().size()`
            - 别名方式:
                - `session.createQuery("from Customer c")`
                - 别名: `session.createQuery("select c from Customer c")`
            - 条件查询: 
                - 创建查询: 
                    - `session.createQuery("from User u where u.id > ?")`
                - 设置参数值:
                    - `setParameter(int index, Object value)`: 设置指定index位置的`?`占位符的值
                    - `setParameter(String key, Object value)`: 设置指定命名参数key变量的值
            - 投影查询: 查询指定字段
                - 普通方式返回的是对象数组: 
                    - `List<Object[]> session.createQuery("select column1, column2 from User").list()`
                - 投影方式步骤:
                    1. 在JavaBean中声明包含要查询字段的带参构造方法(注意同时保留空参构造方法)
                    2. HQL语句中创建对象: 
                        - `List<User> = session.createQuery("select new 类(column1, column2) from User").list();`
            - 排序查询: 
                - `session.createQuery("from User u order by u.id asc")`
            - 分页查询:
                - `query.setFirstResult(int offset)`: 从哪条记录开始, 从0开始
                - `query.setMaxResults(int limit)`: 每页需要的记录条数
                - 每页10个第2页: `List<User> users = session.createQuery("from User").setFirstResult(10).setMaxResults(10).list();`
            - 聚合查询:
                - `count()`:
                    - `List<Number> list = session.createQuery("select count(*) from User").list();`
                    - `int count = list.get(0).intValue();`
                - `sum()`
                    - `List<Number> list = session.createQuery("select sum(id) from User").list();`
                    - `Long sum = list.get(0).longValue();`
                - `avg()`
                - `max()`
                - `min()`
            - 命名查询:
                - 将HQL定义好, 并起名, 查询时按照名字查找HQL语句并执行
                - 定义位置:
                    - 每个JavaBean的hbm配置文件中
                        - `<query name="自定义名称">HQL语句</query>`
                    - 或注解中
                        - `@NamedQuery(name="自定义名称", query="HQL语句")`
    - QBC查询方式(推荐, 用于条件查询): 封装了字符串形式查询的API, 提供面向对象的查询接口
        - Query By Criteria, 条件查询
        - `Criteria c = session.createCriteria(类.class);`
        - `Criteria`类的API
            - 排序: 
                - `criteria.addOrder(Order.asc|desc(column))`
            - 分页
                - `criteria.setFirstResult(int offset)`
                - `criteria.setMaxResults(int limit)`
            - 条件: 
                - `criteria.add(Restrictions r)`: 添加条件
                    - `Restrictions.eq()`: 等于
                    - `Restrictions.gt()`: 大于
                    - `Restrictions.ge()`: 大于等于
                    - `Restrictions.lt()`: 小于
                    - `Restrictions.le()`: 大于等于
                    - `Restrictions.between()`: 介于
                    - `Restrictions.like()`: 模式匹配
                    - `Restrictions.in()`: 在列表中
                    - `Restrictions.and()`: 条件与
                    - `Restrictions.or()`: 条件或
                    - `Restrictions.isNull()`: 为空
            - 聚合:
                - 设置投影聚合: 
                    - `Criteria setProjection(Projections p)`
                        - `Projections.count()`
                        - `Projections.sum()`
                        - `Projections.max()`
                        - `Projections.min()`
                        - `Projections.avg()`
                - 清空Projection: 
                    - `criteria.setProjection(null);`
                    - 设置一种Projection后, 后续查询都会使用该Project, 如需更换查询, 必须清空Projection
            - 离线条件查询:
                - 脱离session来创建查询, 可以不局限于service层查询
                - 创建离线条件查询对象: 
                    - `DetachedCriteria dc = DetachedCriteria.fromClass(类.class);`
                - 执行查询: 
                    - `List<T> list = dc.getExecutableCriteria(session).list();`
    - SQL查询方式: 直接使用SQL语句查询
        - 创建查询: 
            - `SQLQuery q = session.createSQLQuery(String sql);`
        - 默认返回Object数组: 
            - `List<Object[]> list = q.list();`
            - 可以通过设置返回类型修改返回List的泛型: `q.addEntity(类.class)`
        - 支持命名查询, 可以将SQL定义好, 按名称调用
            - 定义位置:
                - 每个JavaBean的hbm配置文件中
                    - `<sql-query name="自定义名称">SQL语句</sql-query>`
                - 或注解中
                    - `@NamedNativeQuery(name="自定义名称", query="SQL语句", resultMapping="自定义SQL查询结果集映射名称")`
                    - `@SqlResultSetMapping(name="userSetMapping", entities={@EntityResult(entityClass=User.class, fields={@FieldResult(name="id", column="id"), @FieldResult(name="name", column="name")})})`

### 使用HQL进行多表查询

* 操作分类(Hibernate中有迫切连接的概念, SQL中没有)
    - 交叉连接
        - 交叉连接会产生笛卡尔积
        - 语法: `... CROSS JOIN ...`
        - 示例: `SELECT * FROM t_1 CROSS JOIN t_2;`
    - 内连接
        - 显式内连接
            - 查询的结果是`Object[]`
            - 语法: `INNER JOIN ... WITH ...`
            - 示例: `FROM User u INNER JOIN u.orders WITH u.id = 1;`
        - 隐式内连接
            - 查询的结果是`Object[]`. 语法和SQL中不一样, 使用`.`来关联
            - 语法: ``
            - 示例: `FROM Order o WHERE o.u.id = 1;`
        - 迫切内连接
            - 查询的结果会直接封装到PO类中
            - 语法: `... INNER JOIN FETCH ...`
            - 示例: `SELECT DISTINCT u FROM User u INNER JOIN FETCH u.orders;`
    - 外连接
        - 左外连接
            - 查询的结果是`Object[]`
            - 语法: `... LEFT OUTER JOIN ...`
            - 示例: `FROM User u LEFT OUTER JOIN u.orders;`
        - 迫切左外连接
            - 查询的结果会直接封装到PO类只能够
            - 语法: `... LEFT OUTER JOIN FETCH ...`
            - 示例: `SELECT DISTINCT u FROM User u LEFT OUTER JOIN FETCH u.orders WHERE u.id = 1;`
        - 右外链接
            - 查询的结果是`Object[]`
            - 语法: `... RIGHT OUTER JOIN ...`
            - 示例: `FROM User u RIGHT OUTER JOIN u.orders;`
* 内连接查询
    - HQL: `from User u inner join fetch u.orders`
    - 两个问题的处理:
        - 返回列表泛型是Object[]的问题, 想要变成类型: 使用`fetch`关键字
            - fetch: 迫切连接, 把数据封装到对象中
        - 查询结果重复的问题: 将List集合放入Set中去重
* 左外连接查询
    - HQL: `from User u left join fetch u.orders`
* 右外连接查询
    - HQL: `from User u right join fetch u.orders`


## 事务管理

* 事务: 一组操作, 必须同时成功, 或同时不成功
* 特性:
    - 原子性: 事务不可分割
    - 一致性: 事务执行的前后数据的完整性保持一致
    - 隔离性: 一个事务执行的过程中, 不应该受到其他的事务的干扰
    - 持久性: 事务一旦提交, 数据就永久保持到数据库中
* 如果不考虑隔离性, 会引发的:
    - 读的问题
        - 问题
            - 脏读: 一个事务读到了另一个事物未提交的数据
            - 不可重复读: 一个事务读到了另一个事物已经提交的update数据, 导致多次查询结果不一致
            - 虚读(幻读): 一个事务读到了另一个事物已经提交的insert数据, 导致多次查询结果不一致
        - 解决
            - 通过设置数据库的隔离级别来解决上述读的问题   
                - 在`hibernate.cfg.xml`配置文件中使用标签来配置`hibernate.connection.isolation = 4`
                    - `1`: 未提交读(Read uncommitted isolation): 以上的读的问题都有可能发生
                    - `2`: 已提交读(Read committed isolation): 避免脏读. 但是不可重复读, 虚读都有可能发生
                    - `4`: 可重复读(Repeatable read isolation): 避免脏读和不可重复读. 但虚读有可能发生
                    - `8`: 串行化(Serializable isolation): 以上读的情况都可以避免
    - 写的问题
        - 问题:
            - 丢失更新, 如两个事物同时修改一条记录
        - 解决:
            - 悲观锁(MySQL自带的, 不怎么用)
                - 在SQL语句后添加`for update`
            - 乐观锁(使用较多)
                - 给JavaBean添加新的属性`version`, 提供getter和setter, 每次操作判断版本
                - 然后在映射的配置文件中添加`<version name='version'/>`标签即可
* Hibernate对session的3种管理方式
    1. Session对象的生命周期与ThreadLocal绑定
        - 原理: 底层使用的是ThreadLocal, 将Connection与当前线程绑定
        - 步骤:
            - `hibernate.cfg.xml`配置:
                - `<property name="hibernate.current_session_context_class">thread</property>`
            - 更换代码中获取session的方法:
                - `Session session = sessionFactory.getCurrentSession();`
            - 注意此方式不需要关闭session, 在线程结束会自动关闭session, 如果调用`session.close()`则会报错
    2. Session对象的生命周期与JTA事务绑定(分布式事务管理)
    3. Hibernate委托程序代码来管理Session的生命周期, 即通过调用代码获取session和销毁session



## 性能优化

* HQL语句优化
    - 使用参数绑定
        - 可以让数据库一次解析SQL, 对后续的重复请求直接使用已经生成的执行计划, 节省CPU计算量和内存
        - 可以避免SQL注入
    - 少使用NOT
        - 如果where子句中包含not, 则执行时该字段的索引会失效
    - 尽可能使用where, 少用having
        - having在查询出所有记录后才对结果集进行过滤, 效率较低
    - 减少子查询
        - 子查询越多效率越低
    - 尽量使用表别名提高阅读性
* Session缓存优化
    - 缓存太大需要及时清理, 可以调用`session.evict()`或`session.clear()`
* 查询策略优化
    - 延迟加载
        - 作用: 先获取代理对象, 当真正用到该对象的属性时, 才会发送SQL语句. 提高程序执行效率
        - 调用session的查询方法使用不同的策略:
            - `session.get()`: 立即加载
            - `session.load()`: 延迟加载
        - 2种延迟加载策略:
            - 类级别的延迟加载
                - 通过session直接查询某一类对应的数据
                - session对象的load方法默认就是延迟加载: 
                    - `User u = session.load(User.class, 1L);`
                - 开启或关闭延迟加载的配置方式:
                    - 在hbm配置中的`<class>`标签增加属性`lazy="是否延迟加载"`
                        - `true`: 延迟加载
                        - `false`: 立即加载
                    - 或使用注解`@Proxy(lazy=是否延迟加载)`
                        - `true`: 延迟加载
                        - `false`: 立即加载
            - 关联级别的延迟加载
                - 默认就是延迟加载
* 抓取策略优化
    - hbm配置关联方的抓取策略:
        - `<set>`标签的配置策略
            - `fetch`: 控制SQL语句的生成格式
                - `select`: 默认, 发送查询语句
                - `join`: 连接查询, 发送一条迫切左外连接, **配置了join, lazy就会失效**
                - `subselect`: 子查询. 发送一条子查询, 查询其关联对象
            - `lazy`: 查找关联对象的时候是否采用延迟加载
                - `true`: 默认, 使用延迟加载
                - `false`: 不使用延迟加载
                - `extra`: 极为延迟
        - `<many-to-one>`或`<one-to-one>`标签的配置策略:
            - `fetch`: 控制SQL格式
                - `select`: 默认, 发送基本select语句查询
                - `join`: 发送迫切左外连接查询
            - `lazy`: 控制加载关联对象是否采用延迟
                - `proxy`: 默认, 代理方式, 是否延迟由一表的以下情况决定:
                    - 如果一表的`<class>`上的lazy为`true`, 则proxy的值就是`true`, 延迟加载
                    - 否则就是`false`, 不延迟加载
                - `false`: 不采用
    - 或采用注解方式配置抓取策略:
        - `@Fetch(FetchMode.SELECT)`
        - 多方: `@LazyCollection(LazyCollectionOption.TRUE)`
        - 一方: `@LazyToOne(LazyToOneOption.FALSE)`
    - 常见配置组合
        - 一方类中对多方属性的注解:
            - 组合1: 先查当前类信息, 需要用到多方时才会查询
                - `@Fetch(FetchMode.SELECT)`
                - `@LazyCollection(LazyCollectionOption.TRUE)`
            - 组合2: 查当前类信息, 并同时查询多方信息
                - `@Fetch(FetchMode.SELECT)`
                - `@LazyCollection(LazyCollectionOption.FALSE)`
            - 组合3: 先查当前类信息, 需要用到多方的某个部分, 则只查询某个部分, 不会全查
                - `@Fetch(FetchMode.SELECT)`
                - `@LazyCollection(LazyCollectionOption.EXTRA)`
            - 组合4: 立即查询, 且是迫切左外连接方式
                - `@Fetch(FetchMode.JOIN)`
                - `@LazyCollection(LazyCollectionOption.FALSE)`
            - 组合5: 生成子查询, 多方使用延迟加载
                - `@Fetch(FetchMode.SUBSELECT)`
                - `@LazyCollection(LazyCollectionOption.TRUE)`
            - 组合6: 生成子查询, 多方立即加载
                - `@Fetch(FetchMode.SUBSELECT)`
                - `@LazyCollection(LazyCollectionOption.FALSE)`
            - 组合7: 生成子查询, 需要用到多方的某个部分, 则只查询某个部分, 不会全查
                - `@Fetch(FetchMode.SUBSELECT)`
                - `@LazyCollection(LazyCollectionOption.EXTRA)`
        - 多方类中对一方属性的注解:
            - 组合1: 先查当前类信息, 需要用到一方时才会查询
                - `@Fetch(FetchMode.SELECT)`
                - `@LazyToOne(LazyToOneOption.PROXY)`
                - 需要一方类使用注解开启延迟加载: `@Proxy(lazy = true)`
            - 组合2: 查当前类信息, 并同时查询一方信息
                - `@Fetch(FetchMode.SELECT)`
                - `@LazyToOne(LazyToOneOption.PROXY)`
                - 需要一方类使用注解关闭延迟加载: `@Proxy(lazy = false)`
            - 组合3: 查当前类信息, 并同时查询一方信息
                - `@Fetch(FetchMode.SELECT)`
                - `@LazyToOne(LazyToOneOption.FALSE)`
            - 组合4: 立即查询, 且是迫切左外连接方式
                - `@Fetch(FetchMode.JOIN)`
                - `@LazyToOne(LazyToOneOption.FALSE)`
    - 批量抓取策略
        - 多表映射关系中, 主键一方是父表, 有外键一方是子表. 
        - 无论查询来自哪方, 都要在父方设置批量抓取策略:
            - hbm中增加`batch-size`属性设置
            - 或使用注解`@BatchSize(size = 数值)`
