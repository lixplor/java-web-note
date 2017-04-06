# Maven

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
    - 本地仓库: 本地电脑的仓库
    - 远程仓库: 自行搭建的maven仓库
    - 中央仓库: 官方maven仓库


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


## maven生命周期




## maven项目目录结构

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
