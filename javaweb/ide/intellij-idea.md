# IntelliJ IDEA

## 下载

* 官网: [https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)
* 版本区别
    - Ultimate: 收费, 功能完善
    - Community: 免费, 很多功能需要手动实现, 很多文件格式不能识别

## 配置

* Ultimate激活
    - [神秘地址](http://idea.iteblog.com/key.php)


## 创建JavaWeb项目

* Ultimate步骤:
    - 选择`Create a new project`
    - 创建maven类型项目
        - 左侧列表选择`Maven`
        - 右侧勾选`Create from archetype`
        - 下方列表选择`org.apache.maven.archetype:maven-archetype-webapp`
        - `Next`
    - 填写项目信息
        - `GroupId`添加包名
        - `ArtifactId`填写项目名
        - `Next`
    - 查看maven仓库信息
        - `Maven home directory`: maven所在目录
        - `User settings file`: 本地maven配置文件
        - `Local repository`: 本地仓库路径
        - `Next`
    - 设置项目目录
        - `Project location`: 修改为项目目录的路径
        - `Finish`
        - 等待构建成功的提示    
* Community步骤:
    - 同Ultimate

### 关联本地Tomcat服务器

* Ultimate版:
    - 已经集成了服务器插件, 直接配置即可
    - 步骤
        - 选择菜单栏的`Run`
            - 选择`Edit Configuration`
        - 左侧点击加号, 选择最底下的`xx more`, 会出现Tomcat相关选项
            - 选择`Tomcat Server`, 然后选择`Local`
                - 右侧目前是`Server`选项卡
                    - `Name`写`Tomcat80`(表示用的Tomcat8.0版本)
                    - 点击`Application server`右侧的`Configure...`, 弹窗会检测出当前安装的Tomcat, 确定
                    - `After launch`可以配置服务器启动后自动打开的浏览器
                    - `Tomcat Server Setting`可以配置端口
                - 点击选项卡的`Deployment`
                    - 点击加号, 选择`Artifact`
                        - 选择`xxx:war exploded`, 确认
                    - 右侧出现`Application context`, 这是项目默认运行的地址
                    - `Apply`, `OK`
        - IDE主界面右上角会出现Tomcat的运行按钮, 点击三角运行
* Community版:
    - 没有自带服务器功能, 需要添加插件手动实现
    - 步骤
        - 选择菜单栏的`Run`
            - 选择`Edit Configuration`
        - 左侧点击加号, 选择`Maven`
            - 右侧`Name`写为`Tomcat80`
            - `Command line`: `tomcat7:run`(号码没错就是7)
        - 在`pom.xml`中添加插件:

```xml
在pom.xml中添加tomcat插件
---------
<project>
    ...
    <build>  
        <finalName>项目名称</finalName>  
        <plugins>  
            <!-- Tomcat插件, tomcat7插件也能用于8 -->
            <plugin>  
                <groupId>org.apache.tomcat.maven</groupId>  
                <artifactId>tomcat7-maven-plugin</artifactId>  
                <version>2.2</version>  
                <configuration>  
                    <port>8080</port>  
                    <path>/</path>  
                    <uriEncoding>UTF-8</uriEncoding>  
                    <server>tomcat80</server>  
                </configuration>  
            </plugin>  
        </plugins>  
    </build>
    ...
 </project>
```

### 一个完整的使用Maven的JavaWeb项目目录结构

```
project/
    |_ src/
        |_ main/
            |_ java/                       # 项目代码
                |_ your/package/name/
            |_ resources/                  # 资源目录
                |_ config/                 # 配置文件目录
                    |_ xxx.properties
                    |_ xxx.xml
                |_ static/                 # 静态资源
                    |_ css/
                    |_ images/
                    |_ js/
                |_ template/               # 模板引擎
            |_ webapp/                     # web服务
                |_ WEB-INF/                
                    |_ web.xml             # web配置
                |_ index.jsp               # 主页
        |_ test/                           # 测试代码
            |_ java/
    |_ pom.xml                             # maven配置
    |_ .gitignore                          # git忽略配置
```

### 关联远程Tomcat服务器

* Ultimate版
    - 已经集成了服务器插件, 直接配置即可
        - 步骤
            - 选择菜单栏的`Run`
                - 选择`Edit Configuration`
            - 左侧点击加号, 选择最底下的`xx more`, 会出现Tomcat相关选项
                - 选择`Tomcat Server`, 然后选择`Remote`
                    - 右侧目前是`Server`选项卡
                        - `Name`写`vps.xxx.com`(表示远程服务器地址)
                        - 点击`Application server`右侧的`Configure...`, 设置服务器安装的Tomcat, 确定
                        - 取消勾选`After launch`
                        - `Tomcat Server Setting`中
                            - `JMX port`: 设置远程
                            - `Type`选`sftp`
                            - `Host`点击`...`按钮创建一个sftp连接
                            - `Path from root`填`/`
                            - `Mapping as`填服务器中Tomcat的`webapps`目录绝对路径
                    - 点击选项卡的`Deployment`
                        - 点击加号, 选择`Artifact`
                            - 选择`xxx:war exploded`, 确认
                        - 右侧出现`Application context`, 这是项目默认运行的地址
                        - `Apply`, `OK`
            - IDE主界面右上角会出现Tomcat的运行按钮, 点击三角运行

### gitignore

```
#==================================
# Maven
#==================================
target/

#==================================
# IDEA
#==================================
.idea/
*.iml

#==================================
# Eclipse
#==================================
.settings/
.classpath/
.project

#==================================
# Log files
#==================================
logs/


```


## 使用maven部署项目到远程Tomcat服务器

* 配置远程Tomcat服务器
    - 必须有`manager`项目
    - 编辑`tomcat-user.xml`, 添加部署用户

```xml
<user
    username="deployer"
    password="123456"
    roles="manager-script"/>
```

* 配置maven-tomcat插件

```xml
pom.xml
-------
<properties>
    <tomcat.plugin.version>2.2</tomcat.plugin.version>

    <warPackageName>项目名称</warPackageName>
    <tomcat.deploy.server>服务器地址</tomcat.deploy.server>
    <tomcat.deploy.serverUrl>Tomcat地址/manager/text</tomcat.deploy.serverUrl>
</properties>

<build>
    <plugins>
        <!-- maven-tomcat插件 -->
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>${tomcat.plugin.version}</version>
            <configuration>
                <server>${tomcat.deploy.server}</server>
                <username>Tomcat用户名</username>
                <password>Tomcat用户密码</password>
                <url>${tomcat.deploy.serverUrl}</url>
                <path>/</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```

* 执行发布命令: `mvn tomcat:7 redeploy`


## 常见问题

* 创建`maven-archetype-webapp`项目, 没有`java`目录; 手动创建后也不检测代码, 不显示包结构
    - 没有目录可以手动创建
    - 不显示为包结构是因为当前IDE将`java`看做是普通目录, 右键, 选择`Mark Directory as`, 点击`Source Set`即可

* 找不到Tomcat提供的Servlet相关API
    - 添加依赖

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.0.1</version>
</dependency>
```
