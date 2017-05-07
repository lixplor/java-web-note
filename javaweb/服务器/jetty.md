# Jetty

* Jetty是Eclipse出品的Java Servlet容器, 作用等同于Tomcat
* 适用于分布式项目
* 比Tomcat更轻量
* 可以集成到IDEA中


## 安装配置

* 官网: [http://www.eclipse.org/jetty/](http://www.eclipse.org/jetty/)


## IDE集成

* Eclipse
    - 安装Jetty插件运行环境
        - `Help > Eclipse Marketplace`
        - 搜索`Jetty`, 找到`Eclipse Jetty x.x.x`, 点击`Install`
        - 安装完毕后重启Eclipse
    - 配置Jetty
        - 右键`Run as > Run Configuration > Jetty Webapp > 你的项目`
        - 可以配置端口, JRE环境


## 添加Jetty的maven插件

* 确保创建了maven项目
* 在[官网](http://www.eclipse.org/jetty/documentation/current/jetty-maven-plugin.html)查看最新的版本号

```xml
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.4.5.v20170502</version>
</plugin>
```

* mvn启动jetty: `mvn jetty:run`
* Eclipse中: `右键 > Run as > Run with Jetty`


## 常见问题

*  Jetty访问报错`A full JDK (not just JRE) is required`
    - 原因: JDK没有选对, 可能是Eclipse默认的版本
    - 解决: 右键`Run as > Run Configuration > Jetty Webapp > 你的项目`, 找到`JRE`标签, 勾选`Alternate JRE`, 选择本地机器安装的最新版JDK. Apply并保存即可
