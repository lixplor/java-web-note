# Shiro

* Apache出品的安全框架, 提供认证, 授权, 加密, 会话管理, 缓存等功能


## Shiro架构

![架构图](http://wiki.jikexueyuan.com/project/shiro/images/1.png)

* `Authentication`: 身份认证/登录模块, 验证用户是否拥有相应的身份
* `Authorization`: 授权模块, 用于验证权限, 确认某个已认证的用户是否拥有某个权限去做对应的事
* `Session Manager`: 会话管理模块, 用户登陆后就是一次会话, 在没有退出前, 它的所有信息都会在会话中
* `Cryptography`: 加密模块, 保护数据安全, 如密码加密存储到数据库
* `Web Support`: Web支持模块, 用于继承到Web环境
* `Caching`: 缓存模块, 如用户登录后, 用户的信息, 角色, 权限不必每次重新查询
* `Concurrency`: 支持多线程并发验证. 在一个线程中开启另一个线程, 能把权限自动传递过去
* `Testing`: 测试支持模块
* `Run As`: 在允许的情况下, 让一个用户假装另一个用户的身份进行访问
* `Remember me`: 记住用户登录信息

![交互流程图](http://wiki.jikexueyuan.com/project/shiro/images/2.png)

* `Subject`: 主体, 代表当前用户, 用户不一定是具体的人, 与当前应用交互的任何东西都是Subject, 如爬虫, 机器人等. 所有操作都委托给SecurityManager
* `SecurityManager`: 安全管理器, 所有与安全有关的操作都会与SecurityManager. 管理所有Subject, 是Shiro的核心
* `Realm`: 域, Shiro从Realm获取安全配置信息(如用户, 角色, 权限). 这部分由开发者实现


## 集成

* 使用maven, 添加`junit`, `commons-logging`, `shiro-core`的依赖

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.9</version>
    </dependency>
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.1.3</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.2.2</version>
    </dependency>
</dependencies>
```


## 身份验证

* Shiro中, 用户需要提供`principals`和`credentials`来验证用户身份. 一般对应用户名和密码
* `principals`: 身份, 主体的标识属性
    - 可以是任何东西, 如用户名, 邮箱等.
    - 必须唯一
    - 一个主体可以有多个`principals`, 但只能有一个`Primary principals`
* `credentials`: 证明或凭证, 如密码, 数字证书等

```java
@Test
public void testHelloworld() {
    //1、获取SecurityManager工厂，此处使用Ini配置文件初始化
    SecurityManager  Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
    //2、得到SecurityManager实例 并绑定给SecurityUtils   
    SecurityManager securityManager = factory.getInstance();
    SecurityUtils.setSecurityManager(securityManager);
    //3、得到Subject及创建用户名/密码身份验证Token（即用户身份/凭证）
    Subject subject = SecurityUtils.getSubject();
    UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");
    try {
        //4、登录，即身份验证
        subject.login(token);
    } catch (AuthenticationException e) {
        //5、身份验证失败
    }
    Assert.assertEquals(true, subject.isAuthenticated()); //断言用户已经登录
    //6、退出
    subject.logout();
}
```


## 授权

* 授权: 也叫访问控制, 即在应用中控制谁能访问哪些资源
* 授权中的关键对象
    - `主体`: 访问应用的用户, 在Shiro中使用Subject代表该用户. 用户只有授权后才允许访问相应的资源
    - `资源`: 在应用中用户可以访问的任何东西, 如JSP页面, 查看编辑某些数据, 访问某个业务方法, 打印文本等都是资源. 用户需要授权后才能访问
    - `权限`: 安全策略中的原子授权单位, 通过权限我们可以表示在应用中用户没有操作某个资源的权利. 即权限表示在应用中用户能不能访问某个资源
    - `角色`: 角色代表操作集合, 可以理解为权限的集合, 一般情况下我们会赋予用户角色而不是权限, 即这样用户可以拥有一组权限, 赋予权限时比较方便.
    - `隐式角色`: 直接通过角色来验证用户有没有操作权限
    - `显式角色`: 在程序中通过权限控制谁能访问某个资源, 角色聚合一组权限集合
* 授权方式
    1. 代码判断
        - `if (subject.hasRole("admin")) {}`
    2. 注解
        - `@RequiresRoles("admin")`
    3. JSP标签
        - `<shiro:hasRole name="admin">`


## 编码和加密

* 编码解码
    - BASE64
        - `Base64.encodeToString(str.getBytes());`
        - `Base64.decodeToString(base64Encoded);`
    - 十六进制
        - `Hex.encodeToString(str.getBytes());`
        - `Hex.decode(base64Encoded.getBytes());`
* 散列算法
    - `new Md5Hash(str, salt).toString();`
    - `new Sha256Hash(str, salt).toString();`
    - `new SimpleHash("SHA-1", str, salt).toString();`
    - `DefaultHashService`
* 加密解密
    - AES
* 密码加密和验证
    - `PasswordService`
    - `CredentialsMatcher`
* 密码重试次数限制
    - 继承`HashedCredentialsMatcher`

```java
public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
    String username = (String)token.getPrincipal();
    //retry count + 1
    Element element = passwordRetryCache.get(username);
    if(element == null) {
        element = new Element(username , new AtomicInteger(0));
        passwordRetryCache.put(element);
    }
    AtomicInteger retryCount = (AtomicInteger)element.getObjectValue();
    if(retryCount.incrementAndGet() > 5) {
        //if retry count > 5 throw
        throw new ExcessiveAttemptsException();
    }
    boolean matches = super.doCredentialsMatch(token, info);
    if(matches) {
        //clear retry count
        passwordRetryCache.remove(username);
    }
    return matches;
}
```


## Spring集成

* `spring-shiro.xml`

```xml
<!-- 会话Cookie模板 -->
<bean id="sessionIdCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
    <constructor-arg value="sid"/>
    <property name="httpOnly" value="true"/>
    <property name="maxAge" value="180000"/>
</bean>
<!-- 会话管理器 -->
<bean id="sessionManager"
class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
    <property name="globalSessionTimeout" value="1800000"/>
    <property name="deleteInvalidSessions" value="true"/>
    <property name="sessionValidationSchedulerEnabled" value="true"/>
    <property name="sessionValidationScheduler" ref="sessionValidationScheduler"/>
    <property name="sessionDAO" ref="sessionDAO"/>
    <property name="sessionIdCookieEnabled" value="true"/>
    <property name="sessionIdCookie" ref="sessionIdCookie"/>
</bean>
<!-- 安全管理器 -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
<property name="realm" ref="userRealm"/>
    <property name="sessionManager" ref="sessionManager"/>
    <property name="cacheManager" ref="cacheManager"/>
</bean>


<!-- 基于Form表单的身份验证过滤器 -->
<bean id="formAuthenticationFilter"
class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter">
    <property name="usernameParam" value="username"/>
    <property name="passwordParam" value="password"/>
    <property name="loginUrl" value="/login.jsp"/>
</bean>
<!-- Shiro的Web过滤器 -->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"/>
    <property name="loginUrl" value="/login.jsp"/>
    <property name="unauthorizedUrl" value="/unauthorized.jsp"/>
    <property name="filters">
        <util:map>
            <entry key="authc" value-ref="formAuthenticationFilter"/>
        </util:map>
    </property>
    <property name="filterChainDefinitions">
        <value>
            /index.jsp = anon
            /unauthorized.jsp = anon
            /login.jsp = authc
            /logout = logout
            /** = user
        </value>
    </property>
</bean>
```

* `web.xml`

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        classpath:spring-beans.xml,
        classpath:spring-shiro-web.xml
    </param-value>
</context-param>
<listener>
   <listener-class>
org.springframework.web.context.ContextLoaderListener
</listener-class>
</listener>

<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```


## Session管理

* Shiro提供的session管理不依赖于底层容器(如tomcat), 不论JavaSE还是JavaEE都可以使用
* 提供会馆管理, 会话事件监听, 会话存储/持久化, 容器无关集群, 失效/过期支持, 对Web透明支持, SSO单点登录支持
* 使用Shiro的会话管理, 可以直接替换Web容器的会话管理
* Shiro提供的3个会话管理器默认实现类
    - `DefaultSessionManager`: 是`DefaultWebSecurityManager`使用的默认实现, 用于Web环境, 直接使用Servlet容器的session
    - `ServletContainerSessionManager`: 是`DefaultWebSecurityManager`使用的默认实现, 可以替代`ServletContainerSessionManager`, 自己维护session, 废弃了Servlet容器的session管理
    - `DefaultWebSessionManager`: 用于Web环境, 替代`ServletContainerSessionManager`, 自己维护session, 废弃了Servlet容器的session管理
* 类和方法
    * `SecurityUtils`
        * `static Subject getSubject()`: 获取Subject
    * `Subject`
        * `Session getSession()`: 获取会话
        * `static logout()`: 自动调用session的stop方法销毁会话
    * `Session`
        * `getId()`: 获取session唯一标识
        * `getHost()`: 获取当前Subject的主机地址
        * `getTimeout()`: 获取过期时间
        * `setTimeout(long millis)`: 设置过期时间
        * `getStartTimestamp()`: 获取session启动时间
        * `getLastAccessTime()`: 获取最后访问时间
        * `touch()`: 更新session最后访问时间. 如果是web项目, 则每次进入ShiroFilter都会自动调用该方法
        * `stop()`: 销毁会话
    * `SecurityManager`
        * `Session start(SessionContext context)`: 启动会话
        * `Session getSession(SessionKey key)`: 根据session的key获取session
        * `boolean isServletContainerSessions()`: 是否使用Servelet容器的会话
    * `ValidatingSessionManager`
        * `void validateSessions()`: 验证所有会话是否过期



### 获取session

```java
login("classpath:shiro.ini", "zhang", "123");
Subject subject = SecurityUtils.getSubject();
Session session = subject.getSession();
```


## 记住我功能

* `sessionIdCookie`：maxAge=-1 表示浏览器关闭时失效此 Cookie；
* `rememberMeCookie`：即记住我的 Cookie，保存时长 30 天；

```xml
spring-shiro-web.xml
---------------------

<bean id="sessionIdCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
    <constructor-arg value="sid"/>
    <property name="httpOnly" value="true"/>
    <property name="maxAge" value="-1"/>
</bean>
<bean id="rememberMeCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
    <constructor-arg value="rememberMe"/>
    <property name="httpOnly" value="true"/>
    <property name="maxAge" value="2592000"/><!-- 30天 -->
</bean>

<!-- rememberMe管理器 -->
<bean id="rememberMeManager"
class="org.apache.shiro.web.mgt.CookieRememberMeManager">
    <property name="cipherKey" value="
\#{T(org.apache.shiro.codec.Base64).decode('4AvVhmFLUs0KTA3Kprsdag==')}"/>
     <property name="cookie" ref="rememberMeCookie"/>
</bean>

<!-- 安全管理器 -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
  <property name="rememberMeManager" ref="rememberMeManager"/>
</bean>

<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="filterChainDefinitions">
        <value>
            /login.jsp = authc
            /logout = logout
            /authenticated.jsp = authc
            /** = user
        </value>
    </property>
</bean>
```
