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
* `Realm`: 域, Shiro从Realm获取安全配置信息(如用户, 角色, 权限)


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
