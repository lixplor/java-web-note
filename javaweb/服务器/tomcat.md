# Tomcat

## 安装和配置

* 官网[http://tomcat.apache.org/](http://tomcat.apache.org/)
* 安装: 解压到任意目录下
* 配置环境变量
    - 配置JAVA环境变量
        - `JAVA_HOME`: 必须. 指向JDK根目录, 通常是`java.jdk`.
        - `JRE_HOME`: 可选. 指向JRE根目录, 通常是`java/jdk/jre`.
        - 如果同时配置了`JAVA_HOME`和`JRE_HOME`, 则优先使用`JRE_HOME`
    - 配置Tomcat环境变量
        - `CATALINA_HOME`: 必须. 指向`tomcat`根目录
        - `CATALINA_BASE`: 可选. 指向配置目录, 一般用于Tomcat多实例

## 目录结构

* Tomcat目录结构

```bash
tomcat/
    |_ bin/                     # 可执行文件
    |_ conf/                    # 配置文件
        |_ web.xml              # 配置web应用的默认行为, web应用可以用自己的web.xml覆盖
        |_ tomcat-users.xml     # 配置tomcat的用户
        |_ server.xml           # tomcat核心配置文件, 配置tomcat的初始化行为
        |_ context.xml          # 配置项目上下文环境
    |_ lib/                     # 依赖库
    |_ logs/                    # 日志
    |_ temp/                    # Tomcat产生的临时文件
    |_ webapps/                 # 存放web项目
    |_ work/                    # JSP文件在运行时产生的java和class文件
```

* 动态web项目的目录结构:

```bash
# Servlet 2.5版本
project/
    |_ 静态资源/
        |_ html/
        |_ css/
        |_ js/
        |_ image/
    |_ WEB-INF/        # 浏览器访问不到
        |_ lib/        # 项目依赖库
        |_ classes/    # 项目生成的class文件
        |_ web.xml     # 项目配置文件
```

## 启动和停止

* `tomcat/bin/startup.sh`: 启动Tomcat
* `tomcat/bin/shutdown.sh`: 停止Tomcat


## 配置

* 参考[官方文档](https://tomcat.apache.org/tomcat-7.0-doc/config/index.html)
    - Tomcat配置文件是无约束的XML
    - 标签和属性都是大小写敏感的
    - 支持Apache Ant风格的变量替换

* 服务器配置: `tomcat/conf/server.xml`
    - `<Server>`: 根标签. 代表整个Catalina Servlet容器. 其属性表示整个Servlet容器的特性
        - 属性:
            - `className`: 实现了`org.apache.catalina.Server`接口的实现类的全类名. 如果不指定, 则使用标准实现类`org.apache.catalina.core.StandardServer`
            - `address`: 服务器等待关闭命令的TCP/IP地址. 如果不指定, 则使用`localhost`
            - `port`: 服务器等待关闭命令的TCP/IP地址. 设置为`-1`可以禁用关闭端口
            - `shutdown`: 通过TCP/IP连接到指定端口号所接收到的命令字符串, 用于关闭Tomcat
        - 内部标签:
            - `<GlobalNamingResources>`: 为Server配置JNDI全局资源
            - `<Service>`: 代表一个或多个共享相同Engine的Connector, 用来处理请求. Server标签中可以有多个Service标签.
                - 属性:
                    - `className`: `org.apache.catalina.Service`接口实现类的全类名. 如果不指定, 则使用标准实现类`org.apache.catalina.core.StandardService`
                    - `name`: Service的显示名称, 会被用于日志信息. Server中每个Service的名称必须唯一
                - 内部标签:
                    - `<Connector>`: 代表处理请求创建响应的组件. Engine可以为Service处理所有请求, Host可以为特定的虚拟主机处理所有请求, Context可以为特定的web应用处理所有请求
                        - 分类:
                            - HTTP Connector: 支持HTTP/1.1协议的Connector. 可以让Catalina作为一个独立web服务器, 可以执行Servlet和JSP页面. 一个Connector实例会监听一个指定TCP端口的连接. 一个Service中可以配置一个或多个Connector, 这些Connector会将请求转发到相关的Engine来处理请求并返回响应. 每个请求都需要一个单独的线程. 如果当前的请求处理线程不够用, 则会创建新的线程, 直到上限(`maxThreads`属性). 此时如果仍有额外的并发的请求, 则这些请求会被保存在Connector创建的server socket中, 直到配置的上限(`acceptCount`属性). 此时如果仍有额外的并发请求, 则会接收到`connection refused`错误, 直到有足够的资源来处理这些请求
                            - AJP Connector
                        - 属性: (必要项为粗体)
                            - 通用属性:
                                - `allowTrace`: 布尔值, 启用或禁用TRACE HTTP方法. 如果没有指定, 默认为false
                                - `asyncTimeout`: 异步请求的默认超时时间毫秒值. 如果没有指定, 默认为10000(10秒)
                                - `enableLookups`: 设置为`true`则可以调用`request.getRemoteHost()`方法来执行DNS查询以便返回远程客户端的真实主机名. 设置为`false`则跳过DNS查询并返回IP地址的字符串(可以提升效率). 默认是禁用的
                                - `maxHeaderCount`: 容器允许的一个请求中能够包含的请求头的最大数量. 如果请求包含的请起头超出该限制, 则请求会被拒绝. 负数表示没有限制. 如果没有指定, 默认为100
                                - `maxParameterCount`: 允许容器自动解析的参数键值对(GET加上POST)的最大数量. 超出该限制的参数或键值对会被忽略. 负数表示没有限制. 如果不指定, 默认为10000. 可以使用`FailedRequestFilter`来拒绝超出限制的请求
                                - `maxPostSize`: 容器处理的FORM表单的POST请求的最大字节数. 设置为0则表示禁用该限制. 如果没有指定, 默认为2097152(2MB). 可以使用`FailedRequestFilter`来拒绝超出限制的请求
                                - `maxSavePostSize`: 容器在FORM表单或CLIENT-CERT验证时允许保存/缓冲的POST请求的最大字节数. 设置为`-1`可以禁用该限制, 设置为`0`可以禁用验证期间的POST数据的保存. 如果没有指定, 默认为4096(4KB)
                                - `parseBodyMethods`: 以逗号分隔的HTTP方法列表. 用于RESTful应用通过POST形式来支持PUT请求. 默认是`POST`
                                - **`port`**: Connector创建server socket等待连接的TCP端口号. 设置为`0`则Tomcat会随机选择一个可用的端口
                                - `protocol`: 设置处理请求的协议. 默认是`HTTP/1.1`
                                - `proxyName`: 如果该Connector用于代理配置, 则要使用该属性来指定`request.getServerName()`时返回的服务器名称.
                                - `proxyPort`: 如果该Connector用于代理配置, 则要使用该属性来指定`request.getServerPort()`时返回的服务器端口号
                                - `redirectPort`: 如果该Connector用于支持非SSL的请求, 并且请求要求SSL通信, Catalina则会自动将该请求重定向到该端口
                                - `scheme`: 设置调用`request.getScheme()`返回的协议名称. 默认为`http`
                                - `secure`: 设置为`true`则在调用`request.isSecure()`时返回`true`. 你应该将此属性设置在支持SSL的Connector中. 默认值为`false`
                                - `URIEncoding`: 指定URI编码的字符集. 如果不指定, 默认`ISO-8859-1`
                                - `useBodyEncodingForURI`: 设置contentType中指定的编码集是否应用于URI中的查询参数, 而不使用URIEncoding. 该属性用于Tomcat 4.1.x的兼容性, 也可以使用`request.setCharacterEncoding()`方法来设置URL中的参数的编码. 默认值为`false`. 注意: 该属性仅对请求中的参数字符串有效, 而对路径无效
                                - `useIPVHosts`: 设置为`true`来让Tomcat使用请求中的IP地址确定发送请求的主机. 默认为`false`
                                - `xpoweredBy`: 设置为`true`来让Tomcat为Servlet标准使用推荐的请求头. 默认`false`
                            - HTTP Connector属性:
                                - `acceptCount`: 当全部请求处理线程被使用后, 存储请求的队列的最大长度. 超出队列长度的请求都会被拒绝. 默认`100`
                                - `acceptorThreadCount`: 接受连接的线程数量. 在多核心CPU机器上可以增大该值, 通常不需要超过2个. 对于大量的非keep alive的连接, 也可以增加该值. 默认`1`
                                - `acceptorThreadPriority`: 接受线程的优先级. 默认值`5`
                                - `address`: 对于拥有多个IP地址的服务器, 该属性可以指定哪个地址用于监听哪个端口. 默认端口会被用于服务器关联的所有IP
                                - `allowedTrailerHeaders`: ...
                                - `bindOnInit`: 控制Connector使用的socket何时被绑定. 默认在Connector初始化时绑定, 在Connector销毁时解绑. 如果设置为`false`, socket会在Connector启动时绑定, 在停止时解绑
                                - `compressibleMimeType`: 指定要使用HTTP压缩的MIME类型. 默认值为`text/html,text/xml,text/plain,text/css,text/javascript,application/javascript`
                                - `compression`: Connector可以使用HTTP/1.1 GZIP压缩来节省服务器带宽. 可用值为`on`(允许文本数据压缩), `force`(所有数据压缩), `off`(禁用压缩), 数字值等同于`on`. 默认为`off`
                                - `compressionMinSize`: 如果conpression属性设置为`on`, 则该属性用来指定数据压缩后的最小体积. 如果没指定, 默认`2048`
                                - `connectionLinger`: ...
                                - `connectionTimeout`: Connector接收到一个连接后等待的最大毫秒值. 设置为`-1`表示没有超时, 默认值为`60000`(60秒). 注意默认的`server.xml`中设置为了`20000`. 该属性也会被用于读取请求体, 如上传文件, 除非将`disableUploadTimeout`设置为`false`
                                - `connectionUploadTimeout`: 指定上传数据的超时毫秒数. 仅在`disableUploadTimeout`设置为`false`后生效
                                - `disableUploadTimeout`: 设置Servlet容器禁止在上传数据时使用更长的连接超时. 如果没有指定, 默认为`true`表示禁用更长的超时
                                - `executor`: 对于Executor标签的引用名称. 如果设置了该属性并且该名称存在, 则Connector会使用该Executor, 其他线程属性会被忽略. 如果不指定共享的executor, 则Connector会使用一个私有的内部的executor来提供线程池
                                - `executorTerminationTimeoutMillis`: ...
                                - `keepAliveTimeout`: Connector在关闭连接前等待另一个HTTP请求的超时毫秒数. 默认值为`connectionTimeout`的属性值. 设置为`-1`表示没有超时
                                - `maxConnections`: 服务器允许接收和处理的最大连接数. 当达到该上限值时, 服务器会接受连接, 但不进行处理. 额外的连接会被阻塞, 直到低于该上限值.
                                - `maxCookieCount`: 一个请求中允许的cookie的最大数量. 负数表示没有限制. 如果没有指定, 默认为`200`
                                - `maxExtensionSize`: ...
                                - `maxHttpHeaderSize`: HTTP请求头和响应头的最大字节数. 如果没指定, 默认为`8192`(8KB)
                                - `maxKeepAliveRequests`: 服务器关闭前可以保持的HTTP最大连接数. 设置为`1`则禁用HTTP/1.0 keep alive, HTTP/1.1 keep alive和pipelining. 设置为`-1`则没有限制. 如果没指定, 默认为`100`
                                - `maxSwallowSize`: ...
                                - `maxThreads`: Connector创建的请求处理线程的最大数量, 它决定了能够处理的最大并发请求数. 如果没执行, 默认为`200`
                                - `maxTrailerSize`: ...
                                - `minSpareThreads`: 保持运行的线程的最小数量. 如果没有指定, 默认`10`
                                - `noCompressionUserAgents`: 值为匹配UA请求头的正则表达式, 设置这些UA不会使用压缩. 默认值是空字符串, 表示禁用
                                - `processorCache`: 协议处理器缓存Processor对象来提升性能. `-1`表示无限制, 默认`200`
                                - `restrictedUserAgents`: 值为匹配UA请求头的正则表达式, 设置这些UA不使用HTTP/1.1或HTTP 1.0 keep alive. 默认是空字符串, 表示禁用
                                - `server`: 重写HTTP响应的Server响应头. 默认是`Apache-Coyote/1.1`
                                - `socketBuffer`: 提供socket输出缓冲的缓冲区大小的字节数. `-1`表示禁用缓冲区, 默认值为`9000`
                                - `SSLEnabled`: 设置为`true`则为该Connector启用SSL通信. 默认值`false`
                                - `tcpNoDelay`: 设置为`true`则TCP_NO_DELAY选项会设置在server socket中, 在大多数场景下会提升性能. 默认为`true`
                                - `threadPriority`: 请求处理线程的优先级. 默认为`5`
                                - `upgradeAsyncWriteBufferSize`: 单次无法完成异步写入时使用的缓冲区的字节数. 默认为`8192`
                    - `<Engine>`: 表示与某个Service关联的请求处理机制. 它接收并处理Connector的所有请求, 并将响应返回Connector并最终返回给客户端
                        - 属性:
                            - `backgroundProcessorDelay`: 表示backgroundProcess方法的执行到其子容器执行之间的延迟秒数. 默认`10`即10秒
                            - `className`: `org.apache.catalina.Engine`接口实现类的全类名. 如果不指定, 默认为`org.apache.catalina.core.StandardEngine`
                            - `defaultHost`: 默认主机名. 该名称必须匹配Host标签中的`name`属性
                            - `jvmRoute`: 用于负载均衡场景, 来启用session affinity
                            - `name`: 该Engine的逻辑名称, 用于日志和错误信息. 如果一个Server中有多个Service标签, 则每个Engine的名称必须唯一
                            - `startStopThreads`: 该Engine用于启动子Host标签的线程数量. `0`表示使用`Runtime.getRuntime().availableProcessors()`的值, 负数表示使用`Runtime.getRuntime().availableProcessors() + value`的值. 如果没有指定, 默认为`1`
                        - 内部标签:
                            - `<Realm>`: 域. 配置域可以指定允许何种用户或角色在不同的Host和Context之间共享
                            - `<Host>`: 表示一个虚拟主机, 是一个服务器网络名称与服务器的组合. 一个Engine中可以有多个Host标签. Host标签中可以有Context标签.
                                - 属性:
                                    - **`appBase`**: 虚拟主机的应用目录. 是发布web应用的目录, 路径可以是绝对路径, 也可以是相对于`$CATALINA_BASE`目录的相对路径. 如果没指定, 默认使用`webapps`
                                    - `xmlBase`: 虚拟主机的XML目录. 是包含XML描述符的目录, 路径可以是绝对路径, 也可以是相对于`$CATALINA_BASE`目录的相对路径. 如果没指定, 默认使用`conf/<engine_name>/<host_name>`
                                    - `createDirs`: 设置为`true`则Tomcat会在启动时创建`appBase`和`xmlBase`目录. 默认为`true`. 如果无法创建, 则会打印错误信息, 但不会终止启动流程
                                    - `autoDeploy`: 指示Tomcat是否定期检查新的web应用. 如果设置为`true`, 则Tomcat会定期检查appBase和xmlBase目录并部署新的web应用或XML描述符. 更新web应用或XML描述符会出发web应用重新加载. 默认为`true`
                                    - `backgroundProcessorDelay`: 表示backgroundProcess方法的执行到其子容器执行之间的延迟秒数. 默认`-1`
                                    - `className`: `org.apache.catalina.Host`接口实现类的全类名. 如果没指定, 默认为`org.apache.catalina.core.StandardHost`
                                    - `deployIgnore`: 设置了`autoDeploy`和`deployOnStartup`后要忽略的路径的正则表达式. 一般用于忽略版本控制相关的目录. 正则表达式路径相对于`appBase`目录
                                    - `deployOnStartup`: 是否在Tomcat启动时自动部署web应用. 默认`true`
                                    - `failCtxIfServletStartFails`: ...
                                    - **`name`**: 虚拟主机的网络名称, 是注册在DNS上的. Tomcat会将其转换为小写字母. Engine中至少要有一个Host的name匹配Engine中`defaultHost`的值
                                    - `startStopThreads`: 该Host用于启动子Host标签的线程数量. `0`表示使用`Runtime.getRuntime().availableProcessors()`的值, 负数表示使用`Runtime.getRuntime().availableProcessors() + value`的值. 如果没有指定, 默认为`1`
                                    - `undeployOldVersions`: 是否自动将旧的不用的web应用的版本卸载. 必须将`autoDeploy`设置为`true`后才有效. 默认`false`
                                    - `copyXML`: 设置为`true`则在应用部署时将应用中的XML描述符(位于`/META-INF/context.xml`)复制到`xmlBase`路径中. 默认`false`
                                    - `deployXML`: 设置为`false`则禁用解析XML描述符. 默认`true`
                                    - `errorReportValveClass`: 用于错误报告的Java类名, 是`org.apache.catalina.Valve`接口实现类的全类名, 默认`org.apache.catalina.valves.ErrorReportValve`
                                    - `unpackWARs`: 设置为`true`则将`appBase`目录中的WAR文件自动解压. 设置为`false`则直接从该WAR文件运行web应用
                                    - `workDir`: 用于应用临时读写的目录. 如果没指定, 默认`$CATALINA_BASE/work`
```xml
<?xml version="1.0" encoding="utf-8"?>
<Server port="8005" shutdown="SHUTDOWN">

  <Listener className="org.apache.catalina.startup.VersionLoggerListener"/>  
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on"/>  
  <Listener className="org.apache.catalina.core.JasperListener"/>  
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"/>  
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener"/>  
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"/>  

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container" type="org.apache.catalina.UserDatabase" description="User database that can be updated and saved" factory="org.apache.catalina.users.MemoryUserDatabaseFactory" pathname="conf/tomcat-users.xml"/>
  </GlobalNamingResources>  

  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1" URIEncoding="utf-8" connectionTimeout="20000" redirectPort="8443"/>  
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>  
    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
      </Realm>  
      <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="localhost_access_log." suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b"/>
      </Host>
    </Engine>
  </Service>
</Server>
```

* Web应用配置: `tomcat/conf/web.xml`

```xml

```

* 用户配置: `tomcat/conf/tomcat-users.xml`

```xml

```


## 日志文件

* 在Tomcat目录下的`logs`目录中, 存放Tomcat运行过程中的所有日志
    - `catalina.out`: Tomcat运行时的日志, 所有`System.out/err`输出都会在这里
    - `catalina.yyyy-MM-dd.log`: Tomcat运行时的日志, 按照日期拆分为单个文件
    - `host-manager.yyyy-MM-dd.log`: Tomcat自带管理页面的日志
    - `manager.yyyy-MM-dd.log`: Tomcat自带管理页面日志
    - `localhost.yyyy-MM-dd.log`: 本机的一些异常日志
    - `localhost_access_log.yyyy-MM-dd.txt`: Web服务访问日志

## 项目发布和虚拟目录映射

Tomcat通过使用映射的虚拟目录找到项目的目录. 所以在发布项目时, 既可以将项目放在虚拟目录中, 也可以通过修改虚拟目录配置找到项目目录.

* 发布项目方式:
    - 方式1: 将项目放到`tomcat/webapps`下
    - 方式2: 创建单独的配置文件
        - 在`tomcat/conf/引擎目录/主机目录`下创建以`虚拟路径URL.xml`为名的配置文件, 文件名就是虚拟路径URL
        - 在配置文件中增加xml文档声明和`<Context docBase="项目在磁盘上的绝对路径"/>`
    - 方式3: 修改`server.xml`配置虚拟路径映射(注意这种方式如果配置文件有错会导致Tomcat无法启动)
        - 修改`tomcat/conf/server.xml`配置文件
        - 在其中`name`属性为`localhost`的`<Host>`标签中, 新增`<Context />`标签
        - 配置Context标签的属性: `<Context path="虚拟路径URL" docBase="项目在磁盘上的绝对路径" />`
            - 如: `<Context path="/bookshop" docBase="~/project/bookshop" />`


## 项目的访问

* 访问路径: `协议://IP:端口号/项目名/index.html`

## 设置是否以目录方式列出项目下的文件

* 修改`web.xml`配置

```xml
<servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
        <param-name>debug</param-name>
        <param-value>0</param-value>
    </init-param>
    <init-param>
        <!-- 设置是否列出目录 -->
        <param-name>listings</param-name>
        <!-- true列出目录, false不列出 -->
        <param-value>true</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```
