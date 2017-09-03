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

* 服务器配置: `tomcat/conf/server.xml`
* 用户配置: `tomcat/conf/tomcat-users.xml`
* Web应用配置: `tomcat/conf/web.xml`

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
