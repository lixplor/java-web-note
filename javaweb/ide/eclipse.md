# Eclipse

## 安装配置

* 官网: [https://eclipse.org/](https://eclipse.org/)
* 配置
    - workspace: 所有项目所在的工作目录

## maven配置

* 下载最新版Eclipse, 已经集成了Maven
* 配置maven: 在`setting`中的`maven`标签中, 选择`installations`, 手动添加本地安装的maven
* 显示maven仓库界面: `Window > Show View > Other > Maven > Maven Repositories`


## 创建项目

* 创建普通动态web项目步骤:
    - `New`
        - 选择`Dynamic Web Project`
            - `Project name`: 项目名称
            - `Project location`: 项目所在目录
            - `Target runtime`: 应用容器, 在这里选择Tomcat服务器
                - 如果没有, 则点击`New Runtime...`
                    - 选择相应版本的服务器
                    - 设置服务器安装目录
                    - 设置JRE
            - `Dynamic web module version`: 选择服务器支持的Servlet标准
        - 目录页面可以提前配置目录, 此步可以直接过
        - Context配置上下文环境, 此步可以直接过
* 创建Maven项目步骤:
    - `New`
        - 选择`Maven Project`
            - 勾选`Create a simple project`, 可以跳过框架选择
                - 注意: 这种方式创建完毕后缺少`web.xml`, 需要补上
            - 填写项目信息
                - `Group Id`: 包名
                - `Artifact Id`: 项目名
                - `Version`: 初始版本
                - `Package`: 打包方式
                    - `jar`: java项目
                    - `pom`: maven模块项目
                    - `war`: java web项目
            - 完成后会报错, 补充`WEB-INF/web.xml`文件即可
