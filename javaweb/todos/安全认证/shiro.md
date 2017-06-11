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
