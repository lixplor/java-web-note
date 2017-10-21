# Spring Boot

* Spring Boot内置了容器, 所以不再需要安装Tomcat或Jetty
* Spring Boot主要包含4个方面
    - starter
        - 有多种starter, 每种starter针对特定的需求提供了依赖的组合, 引入一个starter就相当于引入了相关的依赖
        - 为了简化以前Spring开发中繁琐的模板式配置文件
    - 自动配置
        - 自动配置免去了Bean的配置
    - CLI
        - 用于运行Groovy脚本
    - Actuator
        - 管理端点
        - 合理的异常处理, 默认`/error`映射端点
        - 获取应用信息的`/info`端点
        - Spring Security的审计事件框架
