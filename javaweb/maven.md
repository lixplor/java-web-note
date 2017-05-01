# Maven

* Apache Maven基于项目对象模型(POM)的概念
* Maven项目的结构和内容在一个名为`pom.xml`的XML文件中声明
* 作用:
    - 依赖管理
    - 项目构建(清理, 编译, 测试, 报告, 打包, 部署)

## 安装配置

* 官网[https://maven.apache.org/](https://maven.apache.org/)
* 安装步骤
    - 手动方式
        - 确认安装了JDK 1.7或以上, 并添加到环境变量中
        - 下载压缩包
        - 解压压缩包到任意目录
            - `unzip apache-maven-3.3.9-bin.zip`或`tar xzvf apache-maven-3.3.9-bin.tar.gz`
        - 添加maven的`bin/`目录到环境变量
            - `export PATH="/xxx/apache-maven-3.3.9/bin:$PATH"`
            - `source ./bash_profile`
        - 验证
            - `mvn -v`
        - 配置
            - `maven/conf/settings.xml`
            - `$UserHome/.m2/`也行, 可以实现maven升级同时配置不丢失
    - homebrew方式
        - `brew install maven`
        - 验证
            - `mvn -v`
        - 配置
            - `vim ~/.m2/settings.xml`


## maven仓库

* 3种仓库
    - 本地仓库: 本地电脑的仓库, 
    - 远程仓库: 非官方的maven仓库, 有java.net, 开源中国maven或自建
    - 中央仓库: 官方maven仓库
* 当添加依赖时, 仓库的搜索顺序为: `本地仓库 > 中央仓库 > 远程仓库`
    
### 本地仓库

* 用于存储所有项目的依赖关系到本地文件夹
* 默认情况下, 本地仓库默认为`.m2`目录
    - Uxin/OSX: `~/.m2`
    - Windows: `C:\Documents and Settings\{your-username}\.m2`
* 修改默认仓库为其他目录
    - 编辑` {M2_HOME}\conf\setting.xml`文件
    - 修改`<localRepository></localRepository>`的值为其他目录
    - 执行`mvn archetype:generate -DgroupId=com.yiibai -DartifactId=NumberGenerator -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`保存
    
### 远程仓库

* 添加远程仓库: 在`pom.xml`文件中添加你需要的远程仓库

```xml
pom.xml
-------
<project ...>
    <repositories>
        <!-- 添加远程仓库信息 -->
        <repository>
            <id>java.net</id>
            <url>https://maven.java.net/content/repositories/public/</url>
        </repository>
        <repository>
            <id>JBoss repository</id>
            <url>http://repository.jboss.org/nexus/content/groups/public/</url>
        </repository>
    </repositories>
</project>
```


## maven依赖机制

* 传统方式下我们需要自己下载jar包然后导入到项目, 每次版本更新我们都需要下载新版本的jar包重新导入
* maven方式下, 我们只需要知道依赖库的坐标, 然后会自动下载指定版本的依赖jar包, 如果不指定版本, 则下载仓库中最新版本的依赖

* 坐标

```xml
<groupId>log4j</groupId>
<artifactId>log4j</artifactId>
<version>1.2.14</version>
```

* 在`pom.xml`中添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.14</version>
    </dependency>
</dependencies>
```


## maven项目目录结构

* web项目

```
project/
    |_ src/                       # 源码
        |_ main/                  # 项目代码
            |_ java/              # java代码
            |_ resource/          # 资源文件
            |_ webapp/            # 服务器代码
                |_ index.jsp      
                |_ WEB-INF/       
                    |_ web.xml     
        |_ test/                  # 测试代码
            |_ java/              # java代码
            |_ resource/          # 资源文件
    |_ target/                    # 编译生成的文件(mvn compile生成)
    |_ pom.xml                    # maven项目配置
```

* java项目

```
project/
    |_ src/                       # 源码
        |_ main/                  # 项目代码
            |_ java/              # java代码   
        |_ test/                  # 测试代码
            |_ java/              # java代码
    |_ target/                    # 编译生成的文件(mvn compile生成)
    |_ pom.xml                    # maven项目配置
```


## POM

* POM, Project Object Model, 对象项目模型, 是Maven的基本单位, 是一个XML文件, 保存在项目根目录下的`pom.xml`文件中
* 每个项目应该有一个单一的POM文件
* 所有POM文件项目元素必须有三个必填字段:
    - groupId: 项目所属的组织的名称
    - artifactId: 项目名
    - version: 项目版本号
* 配置包括:
    - 依赖
    - 插件
    - 目标
    - 构建配置
    - 项目版本
    - 开发人员
    - 邮件列表
    
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/
2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 h
ttp://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <!-- 必须的三要素 -->
  <groupId>com.companyname.project-group</groupId>
  <artifactId>project</artifactId>
  <version>1.0</version>
  
  <!-- 构建信息 -->
  <build>
    <sourceDirectory>C:\MVN\project\src\main\java</sourceDirectory>
    <scriptSourceDirectory>src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>C:\MVN\project\src\test\java</testSourceDirectory>
    <outputDirectory>C:\MVN\project\target\classes</outputDirectory>
    <testOutputDirectory>C:\MVN\project\target\test-classes</testOutputDirectory>
    <resources>
      <resource>
        <mergeId>resource-0</mergeId>
        <directory>C:\MVN\project\src\main\resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <mergeId>resource-1</mergeId>
        <directory>C:\MVN\project\src\test\resources</directory>
      </testResource>
    </testResources>
    <directory>C:\MVN\project\target</directory>
    <finalName>project-1.0</finalName>
    
    <!-- 插件管理 -->
    <pluginManagement>
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
  
  <!-- 仓库管理 -->
  <repositories>
    <repository>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <id>central</id>
      <name>Maven Repository Switchboard</name>
      <url>http://repo1.maven.org/maven2</url>
    </repository>
  </repositories>
  
  <!-- 插件仓库 -->
  <pluginRepositories>
    <pluginRepository>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <id>central</id>
      <name>Maven Plugin Repository</name>
      <url>http://repo1.maven.org/maven2</url>
    </pluginRepository>
  </pluginRepositories>
  
  <!-- 报告生成目录 -->
  <reporting>
    <outputDirectory>C:\MVN\project\target/site</outputDirectory>
  </reporting>
</project>
```


## maven生命周期

* 生命周期设置构建项目的执行顺序


* 典型的Maven生命周期: 

|阶段    |处理    |描述                                    |
|--------|--------|----------------------------------------|
|准备资源|资源复制|资源复制可以进行定制                    |
|编译    |执行编译|源代码编译在此阶段完成                  |
|打包    |打包    |创建JAR/WAR包如在 pom.xml 中定义提及的包|
|安装    |安装    |这一阶段在本地/远程Maven仓库安装程序包  |

* clean生命周期(执行`mvn clean`)
    - `pre-clean`
    - `clean`
    - `post-clean`

* 默认的生命周期(23个阶段):

|生命周期阶段         |描述                                                     |
|---------------------|---------------------------------------------------------|
|validate             |验证项目是否正确，并且所有必要的信息可用于完成构建过程   |
|initialize           |建立初始化状态，例如设置属性                             |
|generate-sources     |产生任何的源代码包含在编译阶段                           |
|process-sources      |处理源代码，例如，过滤器值                               |
|generate-resources   |包含在包中产生的资源                                     |
|process-resources    |复制和处理资源到目标目录，准备打包阶段                   |
|compile              |编译该项目的源代码                                       |
|process-classes      |从编译生成的文件提交处理，例如：Java类的字节码增强/优化  |
|generate-test-sources|生成任何测试的源代码包含在编译阶段                       |
|process-test-sources |处理测试源代码，例如，过滤器任何值                       |
|test-compile         |编译测试源代码到测试目标目录                             |
|process-test-classes |处理测试代码文件编译生成的文件                           |
|test                 |运行测试使用合适的单元测试框架(JUnit)                    |
|prepare-package      |执行必要的任何操作的实际打包之前准备一个包               |
|package              |提取编译后的代码，并在其分发格式打包，如JAR，WAR或EAR文件|
|pre-integration-test |完成执行集成测试之前所需操作。例如，设置所需的环境       |
|integration-test     |处理并在必要时部署软件包到集成测试可以运行的环境         |
|pre-integration-test |完成集成测试已全部执行后所需操作。例如，清理环境         |
|verify               |运行任何检查，验证包是有效的，符合质量审核规定           |
|install              |将包安装到本地存储库，它可以用作当地其他项目的依赖       |
|deploy               |复制最终的包到远程仓库与其他开发者和项目共享             |


## maven命令

```shell
# 清理项目
mvn clean

# 编译项目
mvn compile

# 测试
mvn test

# 打包
mvn package

# 打jar包或war包或发布到本地仓库
mvn install
```


## Maven构建环境配置

* 用于针对不同的环境(开发/生产)来自定义构建
* 配置文件有3种:
    - 按项目配置: 定义在项目根目录的`pom.xml`
    - 按用户配置: 定义在用户的Maven配置文件中(`%USER_HOME%/.m2/settings.xml`)
    - 全局配置: 定义在全局的Maven配置文件中(`%M2_HOME%/conf/settings.xml`)

    
### 配置方式

1. 配置pom.xml的profile

```xml
<profiles>
    <profile>
        <!-- 本地开发环境 -->
        <id>dev</id>
        <properties>
            <profiles.active>dev</profiles.active>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <!-- 测试环境 -->
        <id>test</id>
        <properties>
            <profiles.active>test</profiles.active>
        </properties>
    </profile>
    <profile>
        <!-- 生产环境 -->
        <id>product</id>
        <properties>
            <profiles.active>product</profiles.active>
        </properties>
    </profile>
</profiles>
```

2. 创建配置文件
    - 针对不同环境创建不同的配置文件, 配置文件都作为资源文件放在`src/main/resources`目录下. 配置文件使用键值对编写一些变量的配置参数
        - `env-dev.properties`: 开发环境
        - `env-test.properties`: 测试环境
        - `env-product.properties`: 生产环境

3. 配置构建资源

```xml
<build>
　　<filters>
        <filter>${project.basedir}/src/main/resources/environment/env-${profiles.active}.properties</filter>
　　</filters>
　　<resources>
    　　<resource>
        　　<directory>src/main/resources</directory>
        　　<filtering>true</filtering>
    　　</resource>
　　</resources>
</build>
```

4. 执行构建

```shell
# mvn package -P${env}
mvn package -Pdev      # 开发环境
mvn package -Ptest     # 测试环境
mvn package -Pproduct  # 生产环境
```
