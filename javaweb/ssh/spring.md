# Spring

* 业务层框架
* 可单独开发, 也可结合其他框架开发
* 核心: IOC, AOP


## 下载安装

* 官网[http://projects.spring.io/spring-framework/](http://projects.spring.io/spring-framework/)


## 配置log4j

* log4j
    - 配置: `src/log4j.properties`
    - 使用: `Logger.getClass(Xxx.class).info("aaa");`

## 快速入门

* 步骤:
    - 创建一个Service类
    - 创建`src/applicationContext.xml`, 添加schema约束, 声明Service类的bean
    - 创建工厂, 加载核心配置文件, 获取对象, 调用对象方法

```xml
配置文件: applicationContext.xml
-------------------------------
<?xml version="1.0" encoding="UTF-8" ?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
    xsi:shemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 使用bean标签 -->
    <bean id="userDao" class="xxx.package.UserDao"/>
    <bean id="userService" class="xxx.package.UserServiceImpl">
        <!-- 使用依赖注入实现属性初始化 -->
        <property name="username" value="John"/>
        <property name="userDao" ref="userDao"/>
    </bean>

</beans>
```

```java
使用spring方式
-------------
// 创建工厂, 加载配置文件
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
// 从工厂获取对象
UserServiceImpl usi = (UserServiceImpl) context.getBean("userService");
// 调用对象
usi.saveUser();
```


## 工厂类

* 早期使用`BeanFactory`, 现已由`ApplicationContext`代替
* `ApplicationContext`接口
    - 用于获取Bean对象
    - 有2个实现类:
        - `ClassPathXmlApplicationContext`: 加载类路径下的Spring配置文件
        - `FileSystemXmlApplicationContext`: 加载本地磁盘的Spring配置文件
* 2者区别
    - BeanFactory使用延迟加载, 第一次调用`getBean()`时才会初始化Bean
    - ApplicationContext在加载applicationContext.xml时就会创建Bean, 还提供了事件传递, Bean自动装配, 各种不同Context的实现等其他功能


## applicationContext.xml详解

* `<bean>`标签
    - `id`: 起到区分作用, 不允许出现特殊符号
    - `class`: 要创建对象的完整类名
    - `name`: 和id类似, 可以使用特殊符号
    - `scope`: bean的作用范围
        - `singleton`: 默认, 单例
        - `prototype`: 多例, 结合struts2的action
        - `request`: 应用在Web项目中, 每次Http请求都会胡藏剑一个新的bean
        - `session`: 应用在Web项目中, 同一个Http Session共享一个bean
        - `globalsession`: 应用在Web项目中, 多服务器之间的session
* Bean对象创建和销毁的2个生命周期属性
    - `init-method`: 当bean被载入到容器的时候, 调用init-method属性指定的方法
    - `destroy-method`: 当bean从容器中删除的时候调用destroy-method属性指定的方法


## IOC 控制翻转

* IOC, Inverse of Control, 控制翻转. 将对象的创建权反转给框架, 实现解耦
* 实现方式:
    - `业务代码`和`资源`通过中间的`工厂类`和`配置文件`控制, 实现解耦


## DI 依赖注入

* DI, Dependency Injection, 依赖注入, 动态将依赖对象注入
* 注入Bean
    - `<bean>`
        - 属性: `<property name="属性名" value="属性值"/>`
        - 引用对象: `<property name="对象名" ref="对象bean标签的id"/>`
        - 构造方法: `<constructor-arg name="参数", value="参数值"/>`
        - setter: 同属性
* SpEL注入
    - `#{表达式}`
    - `<property name="username" value="#{'Tom'}"/>`
* 注入集合或数组
    - 数组, List集合: `<list></list>`
        - 普通元素: `<value></value>`
        - 对象元素: `<ref bean="对象id"/>`
    - Set集合: `<set></set>`
        - 普通元素: `<value></value>`
        - 对象元素: `<ref bean="对象id"/>`
    - Map集合: `<map></map>`
        - `<entry key="键" value="值"/>`
        - `<entry key-ref="键" value-ref="值"/>`
    - properties属性文件: `<property name="文件名"></property>`
        - `<props></props>`
            - `<prop key="键">值</prop`

```java
public class User {
    private String[] arr;
    public void setArr(String[] arr) {
        this.arr = arr;
    }
}
```

```xml
<bean id="user" class="xxx.User">
    <!-- 数组/集合的注入 -->
    <property name="arr">
        <list>
            <value>一</value>
            <value>二</value>
            <value>三</value>
            <!-- 如果元素是对象 -->
            <ref bean="引入对象的id"/>
        </list>
    </property>    
</bean>
```


## 配置文件管理

* 可以使用多个配置文件
* 2种方式
    1. xml方式, 在主配置文件中引入其他配置文件
        - `<import resource="com/xxx/xxx.xml"/>`
    2. 代码方式
        - `new ClassPathXmlApplicationContext(String...)`


## 注解方式

* 注解方式更简单, 用于取代xml配置
* 步骤:
    - 导入必要包, 包括`spring-aop-xxx.jar`
    - 在`applicationContext.xml`中开启注解扫描:
        - `<context:component-scan base-package="要扫描的包"/>`
    - 在类中
        - 类的注解: `@Component(value="别名id")`, 相当于`<bean id="别名id" class>`
* 注解
    - 组件(类)注入的注解
        - `@Component`. 有3个衍生注解, 目前作用一致, 后续会增强
            - `@Controller`: 作用在Web层
            - `@Service`: 作用在业务层
            - `@Repository`: 作用在持久层
        - `@Scope(value="作用类型")`: 设置bean的作用范围
    - 属性注入的注解, 可以不提供setter
        - `@Value(value="初始化值")`: 基本数据类型和字符串属性注入
        - `@Autowired`: 引用类型(对象). 按类型自动注入
            - `@Qualifier(value="id名称")`: 设置为按id名称注入
        - `@Resource(name="id名称")`: Java提供的注解, 相当于`@Autowired`和`@Qualifier`一起使用
    - 生命周期的注解
        - `@PostConstruct`: 相当于`init-method`
        - `@PreDestroy`: 相当于`destroy-method`


```xml
<beans>
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="要扫描的包"/>
</beans>
```

```java
@Component
public class UserServiceImpl implements UserService {

    public void hello() {
        // ..
    }
}
```

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService us = (UserService) ac.getBean("userService");
us.save();
```


## Spring整合JUnit单元测试

* 作用: 免去创建工厂, 加载配置文件, getBean()
* 步骤
    1. 引入`spring-test-xxx.jar`
    2. 通过注解实现
        - 类上
            - 设置JUnit测试: `@RunWith(SpringJUnit4ClassRunner.class)`
            - 设置配置文件: `@contextConfiguration("classpath:applicationContext")`
        - 成员
            - 引入依赖: `@Resource(name=userService)`
        - 方法
            - 执行测试: `@Test`

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class Demo {
    @Resource(name="userService")
    private UserService userService;

    @Test
    public void testHello() {
        // 不用再获取工厂加载配置文件getBean()了
        userService.hello();
    }
}
```


## AOP

* AOP, Aspect oriented programming, 面向切面编程, 目的是实现模块化编程, 取代传统纵向继承的重复性代码. 是OOP的延续
* 在不改变原代码的情况下, 增加功能
* 通过预编译方式, 在运行期间动态代理实现程序功能

### AOP的实现原理: 动态代理

* 使用JDK的`Proxy`类生成代理对象

```java
public static UserDao getProxy(final UserDao dao) {
    UserDao proxy = (UserDao) Proxy.newProxyInstance(dao.getClass().getClassLoader(), dao.getClass().getInterfaces(), new InvocationHandler() {
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // 代理代码
            if("save".equals(method.getName())) {
                System.out.println("...");
            }
            return method.invoke(dao, args);
        }
    });
    return proxy;
}
```

* 使用cglib方式进行动态代理


### AOP术语

* `Joinpoint`: 连接点. 指要被拦截到的点, 在spring中这些点指的是方法, 因为spring只支持方法类型的连接点
* `Pointcut`: 切入点. 指我们要对那些Joinpoint进行拦截的定义
* `Advice`: 通知/增强. 指拦截到Joinpoint之后所要做的事情就是通知.
    - 前置通知
    - 后置通知
    - 异常通知
    - 最终通知
    - 环绕通知
* `Introduction`: 引介. 一种特殊的通知, 在不修改代码的前提下, Introduction可以在运行期为类动态地添加一些方法或属性
* `Target`: 目标对象. 代理的目标对象
* `Weaving`: 织入. 指把增强应用到目标对象来创建新的代理对象的过程
* `Proxy`: 代理. 一个类被AOP织入增强后, 就产生一个结果代理类
* `Aspect`: 切面. 切入点和通知的结合


### xml配置方式

* 步骤
    - 导入依赖包
    - 创建业务类
    - 创建切面类, 定义方法作为advice
    - 创建`/src/applicationContext.xml`
        - 引入`aop`schema
        - 声明切面类和其他类的bean
        - 配置aop
            - `<aop:config></aop:config>`
                - 配置切面类: `<aop:aspect ref="切面类id"></aop:aspect>`
                    - 配置前置通知: `<aop:before method="方法名" pointcut="切入点表达式"/>`

```java
业务类, 要被切入的类
---------------
public class UserDao {

    public User getUser() {

    }
}
```

```java
切面类, 要切入的增强功能
--------------------
public class LogAspect {
    public void log() {
        // ...
    }
}
```

```xml
配置aop
------
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    <!-- 配置业务类dao -->
    <bean id="userDao" class="xxx.UserDao"/>

    <!-- 配置切面类 -->
    <bean id="logAspect" class="xxx.LogAspect"/>

    <!-- 配置aop -->
    <aop:config>
        <!-- 配置切面类 -->
        <aop:aspect ref="logAspect">
            <!-- 配置前置通知, 即在方法最前执行 -->
            <aop:before method="log" pointcut="execution(public User xxx.UserDao.getUser())"
        </aop:aspect>
    </aop:config>
</beans>
```

#### 切入点表达式

* 格式: `execution()`
    - 权限修饰符可以省略
    - 返回值必须写, 可以使用`*`进行通配
    - 包, 也可以写`*`通配一层, 使用`*..*`通配多层
    - 类, 也可以使用`*`通配, 一般`*DaoImpl`
    - 方法, 可以使用`*`通配, 如`save*()`
    - 参数, 可以使用`..`通配

#### 通知类型

* 前置通知
    - 在目标类的方法执行之前执行
    - 配置: `<aop:before method="方法名" pointcut-ref="切入点"/>`
    - 场景: 校验方法参数
* 后置通知
    - 在目标类的方法执行之后执行
    - 配置: `<aop:after-returning method="方法名" pointcut-ref="切入点"/>`
    - 场景: 修改方法返回值
* 异常通知
    - 在抛出异常后通知
    - 配置: `<aop:after-throwing method="方法名" pointcut-ref="切入点"/>`
    - 场景: 封装异常信息
* 最终通知
    - 在目标类的方法执行后执行, 即使之前出现了异常
    - 配置: `<aop:after method="方法名" prointcut-ref="切入点"/>`
    - 场景: 释放资源
* 环绕通知
    - 方法执行的前, 后都执行
    - 配置: `<aop:around method="方法名" pointcut-ref="切入点"/>`
    - 注意: 目标的方法默认不执行, 需要使用`ProceedingJoinPoint point`参数来让目标对象的方法执行


### 注解配置方式

* 步骤
    - 引入必要包
    - 创建`src/applicationContext.xml`
        - 开启注解自动代理
        - 声明bean
    - 创建实现类
    - 创建切面类, 使用注解


```java
public class UserDaoImpl implements UserDao {

}
```

```java
切面类
-----
@Aspect
public class LogAspect {
    @Before(value="execution(public * *..*.UserDaoImpl.save())")
    public void log() {
        // ...
    }
}
```

```xml
<beans>
    <!-- 开启自动注解 -->
    <aop:aspectj-autoproxy/>

    <!-- 配置目标对象 -->
    <bean id="userDao" class="xxx.UserDaoImpl"/>

    <!-- 配置切面类 -->
    <bean id="logAspect" class="xxx.LogAspect"/>

</beans>
```

通知注解类型:
* `@Before`
* `@After`
* `@AfterRunning`
* `@AfterThrowing`
* `@Around`

配置通用切入点
* `@Pointcut(value="切入表达式")`定义切入点方法, 方法为空实现
* 在其他切入点表达式引入

```java
public class MyAspect {
    //定义一个切入点
    @Pointcut(value="execution(public * xxx.save())")
    public void commonPointcut(){}

    @Before(value="MyAspect.commonPointcut()")    
    public void run() {
        // ...
    }
}
```


## JDBC模板

### 模板类

* Spring提供了内置的连接池, 也可以整合其他连接池

```xml
applicationContext.xml
----------------------
<!-- 配置连接池 -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql:///db_url"/>
    <property name="username" value="root"/>
    <property name="password" value="123445"/>
</bean>

<!-- 配置JDBC的模板类 -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

```java
// 创建mysql连接池
DriverManagerDataSource source = new DriverManagerDataSource();
source.setDriverClassName("com.mysql.jdbc.Driver");
source.setUrl("jdbc:mysql:///db_url");
source.setUsername("root");
source.setPassword("123456");

//创建模板类
JdbcTemplate t = new JdbcTemplate();
// 设置连接池
t.setDataSource(source);
// 操作
t.update("insert into t_account values (null, ?, ?)", "哈哈", 10000);
```

### 集成第三方连接池

* DBCP连接池
    - 引入jar包
        - `com.springsource.org.apache.commons.dbcp-x.x.x.jar`
        - `com.springsource.org.apache.commons.pool-x.x.x.jar`
    - 编写配置文件

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql:///db_url"/>
    <property name="username" value="root"/>
    <property name="password" value="1234"/>
</bean>
```

* C3P0连接池
    - 引入jar
        - `com.springsource.com.mchange.v2.c3p0.xxx.jar`
    - 编写配置文件

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql:///db_url"/>
    <property name="user" value="root"/>
    <property name="password" value="1234"/>
</bean>
```

### 使用模板类

* `update(String sql, Object... params)`: 增删改
* `queryForObject(String sql, RowMapper<T> mapper, Object... params)`: 查
* `query(String sql, RowMapper mapper, List<T> list)`: 查

```java
// 增
jdbcTemplate.update("insert into table values (null, ?, ?)", "Tom", 100);
// 改
jdbcTemplate.update("update table set name = ? where id = ?", "Tom", 100);
// 删
jdbcTemplate.update("delete from table where name = ? and id = ?", "Tom", 100);

// 查
class BeanMapper implements RowMapper<User> {
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setName(rs.getString("name"));
        user.setAge(rs.getInt("age"));
        return user;
    }
}
User user = jdbcTemplate.queryForObject("select * from t_user where age = ?", new BeanMapper(), 1);
List<User> users = jdbcTemplate.query("select * from t_user", new BeanMapper());
```


## 事务管理

* `PlatformTransactionManager`接口: 平台事务管理器, 有实现类
    - 实现类
        - `DataSrouceTransactionManager`: SpringJDBC模板类, MyBatis
        - `HibernateTransactionManager`: Hibernate
    - 方法
        - `void commit(TransactionStatus status)`
        - `TransactionStatus getTransaction(TransactionDefinition definition)`
        - `void rollback(TransactionStatus status)`
* `TransactionDefinition`接口: 事务定义信息(事务的隔离级别, 传播行为, 超时, 只读)
    - 事务隔离级别常量
        - `ISOLATION_DEFAULT`
        - `ISOLATION_READ_UNCOMMITTED`
        - `ISOLATION_READ_COMMITTED`
        - `ISOLATION_REPEATABLE_READ`
        - `ISOLATION_SERIALIZABLE`
    - 事务传播行为常量(一般不设置, 使用默认值)
        - `PROPAGATION_REQUIRED`: 默认. A中有事务, 使用A中的事务, 如果没有, B就会开启一个新的事务, 将A包含进来(即保证A, B在同一个事务中)
        - `PROPAGATION_SUPPORTS`: A中有事务, 使用A中的事务, 如果没有, 那么B也不使用事务
        - `PROPAGATION_MANDATORY`: A中有事务, 使用A中的事务, 如果没有, 抛出异常
        - `PROPAGATION_REQUIRES_NEW`: A中有事务, 将A的事务挂起, B创建一个新的事务(即保证A, B不在一个事务中)
        - `PROPAGATION_NOT_SUPPORTED`: A中有事务, 将A中的事务挂起
        - `PROPAGATION_NEVER`: A中有事务, 抛出异常
        - `PROPAGATION_NESTED`: 嵌套事务. 当A执行之后, 就会在这个位置设置一个保存点, 如果B没有问题, 执行通过, 如果B出现异常, 运行客户根据需求回滚(选择回滚到保存点或者最初状态)
* `TransactionStatus`接口: 事务的装填

### 事务配置

#### xml配置方式

* 代码实现
* aop声明式(推荐)

```xml
<!-- 声明式事务 -->
<!-- 配置通知 -->
<tx:advice id="myAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="方法名" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
<!-- 配置AOP -->
<aop:config>
    <!-- spring提供的通知 -->
    <aop:advisor advice-ref="myAdvice" pointcut="execution(public * com.xxx.pay())"/>
</aop:config>
```

#### 注解方式

步骤:
1. xml中开启注解
2. 类上加注解: `@Transactional`

```xml
<!-- 开启事务注解 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

```java
@Transactional
public class UserServiceImpl implements UserService {

}
```



## Spring Struts整合

* 解决重复加载配置文件的问题
    - 引入`spring-web-x.x.x.jar`包
    - 配置`ContextLoaderListener`监听器, 加载配置文件
    - 使用`WebApplicationContext`创建工厂

```xml
<!-- 配置Spring的监听器 -->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

```java
// 使用WebApplicationContext
WebApplicationContext wac = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
UserService us = (UserService) wac.getBean("userService");
userService.save();
```
