# SSH整合

## 概览

```
            Web层               Service层             DAO层
请求 ----->           -------->           ---------->
            struts2             Spring               Hibernate
            核心过滤器            IOC和AOP             持久化操作
            拦截器               AOP事务管理           ORM对象关系映射
            调用目标Action                            映射配置文件
```

## 配置文件

* struts2
    - 配置文件
        - `web.xml`: 过滤器
        - `struts.xml`: 配置action
* spring
    - 配置文件
        - `applicationContext.xml`: 配置IOC, AOP, 事务
        - `web.xml`: 监听器
        - `log4j.properties`: log4j日志配置
* hibernate
    - 配置文件
        - `hibernate.cfg.xml`: 数据库配置
        - `JavaBean类名.hbm.xml`: ORM映射配置


`web.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID"
    version="2.5">

    <display-name>项目名称</display-name>

    <!-- 配置Spring的web监听器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <!-- 配置struts2的过滤器 -->
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>

</web-app>
```

`struts.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
    "http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>



</struts>
```

`hibernate.cfg.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0-dtd">

<hibernate-configuration>

    <session-factory>
        <!-- 必选配置 -->
        <property name="hibernate-connection.driver.class">com.mysql.jdbc.Driver</property>
        <property name="hibernate-connection.url">jdbc:mysql:///db_url</property>
        <property name="hibernate-connection.username">root</property>
        <property name="hibernate-connection-password">123456</property>
        <property name="hibernate-dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- 可选配置 -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- C3P0配置 -->
        <property name="connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</property>

        <!-- 映射配置文件 -->
        <mapping resource="com/package/JavaBean类名.hbm.xml"/>
    </session-factory>

</hibernate-configuration>
```

`JavaBean类名.hbm.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0-dtd">

<hibernate-mapping>

    <class name="com.pakcage.JavaBean类名" table="数据表名">
        <!-- 配置主键映射 -->
        <id name="JavaBean id属性名" column="数据表主键字段名">
            <generator class="native"/>
        </id>

        <!-- 配置其他字段映射 -->
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
    </class>

</hibernate-mapping>
```

`applicationContext.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">



</beans>
```


## 整合

### 方案一: 使用`hibernate.cfg.xml`

* Web层
    - `XxxAction`
* Service层
    - `UserService`
* DAO层

```
src/
    |_ com.package/
        |_ dao/
            |_ UserDao.java
            |_ UserDaoImpl.java
        |_ domain/
            |_ User.java
        |_ web/
            |_ action/
                |_ UserAction.java
        |_ service/
            |_ UserService.java
            |_ UserServiceImpl.java
    |_ applicationContext.xml
    |_ hibernate.cfg.xml
    |_ log4j.properties
    |_ struts.xml
```


`UserAction.java`:

```java
public class UserAction extends ActionSupport implements ModelDriven<User> {
    private static final long serialVersionUID = 12324123213343324L;

    private User user = new User();
    private UserService userService;

    @Override
    public User getModel() {
        return this.user;
    }

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public String add() {
        // 保存客户
        // 方式1: 工厂, 太麻烦
        /*WebApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(ServletActionContext.getServletContext());
        UserService us = (UserService) context.getBean("userService");
        us.save(this.user);*/

        // 方式2: 注入
        this.userService.save(this.user);

        return NONE;
    }
}
```

`User.java`:

```java
public class User {
    private Long user_id;
    private String user_name;
    private Long user_create_id;
    private String user_source;

    // getter/setter
}
```

`UserService.java`:

```java
public interface UserService {

    public void save(User user);
}
```

`UserServiceImpl.java`:

```java
@Transactional
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void save(User user) {

    }
}
```

`UserDao.java`:

```java
public interface UserDao {

    public void save(User user);
}
```

`UserDaoImpl.java`:

```java
public class UserDaoImpl extends HibernateDaoSupport implements UserDao {

    @Override
    public void save(User user) {
        // 方式1: 模板类
        this.getHibernateTemplate().save(user);
    }
}
```

### 配置文件

`web.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID"
    version="2.5">

    <display-name>项目名称</display-name>

    <!-- 配置Spring的web监听器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <!-- 配置struts2的过滤器 -->
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>

</web-app>
```

`struts.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
    "http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>

    <!-- 配置包结构 -->
    <package name="crm" extends="struts-default" namespace="/">
        <!-- 配置用户的Action. 由Struts来管理action
        <action name="user_*" class="com.package.web.action.UserAction" method="{1}"></action>
        -->

        <!-- 配置用户的Action. 如果Action由Spring来管理, class只需要写id值 -->
        <action name="user_*" class="userAction" method="{1}">

        </action>
    </package>

</struts>
```

`hibernate.cfg.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0-dtd">

<hibernate-configuration>

    <session-factory>
        <!-- 必选配置 -->
        <property name="hibernate-connection.driver.class">com.mysql.jdbc.Driver</property>
        <property name="hibernate-connection.url">jdbc:mysql:///db_url</property>
        <property name="hibernate-connection.username">root</property>
        <property name="hibernate-connection-password">123456</property>
        <property name="hibernate-dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- 可选配置 -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- C3P0配置 -->
        <property name="connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</property>

        <!-- 映射配置文件 -->
        <mapping resource="com/package/JavaBean类名.hbm.xml"/>
    </session-factory>

</hibernate-configuration>
```

`JavaBean类名.hbm.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0-dtd">

<hibernate-mapping>

    <class name="com.pakcage.JavaBean类名" table="数据表名">
        <!-- 配置主键映射 -->
        <id name="JavaBean id属性名" column="数据表主键字段名">
            <generator class="native"/>
        </id>

        <!-- 配置其他字段映射 -->
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
    </class>

</hibernate-mapping>
```

`applicationContext.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 加载hibernate.cfg.xml -->
    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="configLocation" value="classpath:hibernate.cfg.xml"/>
    </bean>

    <!-- 配置平台事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <!-- 开启事务注解 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    <!-- 配置用户模块. Action必须配置为多例prototype -->
    <bean id="userAction" class="com.package.web.action.UserAction" scope="prototype">
        <!-- 注入Service属性 -->
        <property name="userService" ref="userService"/>
    </bean>
    <bean id="userService" class="com.package.service.UserServiceImpl">
        <!-- 注入Dao属性 -->
        <property name="userDao" ref="userDao"/>
    </bean>
    <bean id="userDao" class="com.package.dao.UserDaoImpl">
        <!-- 注入sessionFactory属性 -->
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
</beans>
```


### 方案二: 不使用`hibernate.cfg.xml` (推荐)

主要使用`applicationContext.xml`整合`hibernate.cfg.xml`的配置

* Web层
    - `XxxAction`
* Service层
    - `UserService`
* DAO层

```
src/
    |_ com.package/
        |_ dao/
            |_ UserDao.java
            |_ UserDaoImpl.java
        |_ domain/
            |_ User.java
        |_ web/
            |_ action/
                |_ UserAction.java
        |_ service/
            |_ UserService.java
            |_ UserServiceImpl.java
    |_ applicationContext.xml
    |_ log4j.properties
    |_ struts.xml
```


`UserAction.java`:

```java
public class UserAction extends ActionSupport implements ModelDriven<User> {
    private static final long serialVersionUID = 12324123213343324L;

    private User user = new User();
    private UserService userService;

    @Override
    public User getModel() {
        return this.user;
    }

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public String add() {
        // 保存客户
        // 方式1: 工厂, 太麻烦
        /*WebApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(ServletActionContext.getServletContext());
        UserService us = (UserService) context.getBean("userService");
        us.save(this.user);*/

        // 方式2: 注入
        this.userService.save(this.user);

        return NONE;
    }
}
```

`User.java`:

```java
public class User {
    private Long user_id;
    private String user_name;
    private Long user_create_id;
    private String user_source;

    // getter/setter
}
```

`UserService.java`:

```java
public interface UserService {

    public void save(User user);
}
```

`UserServiceImpl.java`:

```java
@Transactional
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void save(User user) {

    }
}
```

`UserDao.java`:

```java
public interface UserDao {

    public void save(User user);
}
```

`UserDaoImpl.java`:

```java
public class UserDaoImpl extends HibernateDaoSupport implements UserDao {

    @Override
    public void save(User user) {
        // 方式1: 模板类
        this.getHibernateTemplate().save(user);
    }
}
```

### 配置文件

`web.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID"
    version="2.5">

    <display-name>项目名称</display-name>

    <!-- 配置Spring的web监听器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <!-- 配置struts2的过滤器 -->
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>

</web-app>
```

`struts.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
    "http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>

    <!-- 配置包结构 -->
    <package name="crm" extends="struts-default" namespace="/">
        <!-- 配置用户的Action. 由Struts来管理action
        <action name="user_*" class="com.package.web.action.UserAction" method="{1}"></action>
        -->

        <!-- 配置用户的Action. 如果Action由Spring来管理, class只需要写id值 -->
        <action name="user_*" class="userAction" method="{1}">

        </action>
    </package>

</struts>
```

`JavaBean类名.hbm.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0-dtd">

<hibernate-mapping>

    <class name="com.pakcage.JavaBean类名" table="数据表名">
        <!-- 配置主键映射 -->
        <id name="JavaBean id属性名" column="数据表主键字段名">
            <generator class="native"/>
        </id>

        <!-- 配置其他字段映射 -->
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
        <property name="JavaBean属性名" column="数据表字段名"/>
    </class>

</hibernate-mapping>
```

`applicationContext.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- C3P0连接池配置 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPolledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql:///db_url"/>
        <property name="user" value="root"/>
        <property name="password" value="123456"/>
    </bean>

    <!-- 加载hibernate配置 -->
    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <!-- 加载连接池 -->
        <property name="dataSource" value="dataSource"/>
        <!-- 加载方言 -->
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql">true</prop>
                <prop key="hibernate.hbm2dll.auto">update</prop>
            </props>
        </property>
        <!-- 引入映射配置 -->
        <property name="mappingResources">
            <list>
                <value>com/package/domain/User.hbm.xml</value>
            </list>
        </property>
    </bean>

    <!-- 配置平台事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <!-- 开启事务注解 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    <!-- 配置用户模块. Action必须配置为多例prototype -->
    <bean id="userAction" class="com.package.web.action.UserAction" scope="prototype">
        <!-- 注入Service属性 -->
        <property name="userService" ref="userService"/>
    </bean>
    <bean id="userService" class="com.package.service.UserServiceImpl">
        <!-- 注入Dao属性 -->
        <property name="userDao" ref="userDao"/>
    </bean>
    <bean id="userDao" class="com.package.dao.UserDaoImpl">
        <!-- 注入sessionFactory属性 -->
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
</beans>
```
