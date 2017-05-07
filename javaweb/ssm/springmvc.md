# SpringMVC

* 负责Web层, 替代Struts2. 是Spring框架中的一个组件
* 处理请求返回响应
* 基于MVC的设计思想

与struts2的区别:
* SpringMVC的入口是一个Servlet即前端控制器. Struts2的入口是一个Filter过滤器
* SpringMVC是基于方法开发的(一个请求url对应一个方法), 请求参数传递到方法的形参, 可以设计为单例或多例(建议单例). Struts2是基于类开发的(一个请求url对应一个类), 传递参数到类的属性, 只能设计为多例
* Struts采用ValueStack存储请求和响应数据, 通过OGNL存取数据. SpringMVC通过参数解析器将请求内容解析, 给方法形参赋值, 将数据和视图封装成ModelAndView对象, 最后又将ModelAndView中的模型数据通过request域传输到页面, JSP视图解析器默认使用JSTL


## 安装配置

* 官网: [http://projects.spring.io/spring-framework/](http://projects.spring.io/spring-framework/)
* maven

```xml
<dependencies>
    <!-- spring-context 核心依赖 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.7.RELEASE</version>
    </dependency>
    <!-- spring web mvc 依赖 -->
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-webmvc</artifactId>  
        <version>4.3.7.RELEASE</version>  
    </dependency>
</dependencies>
```


## 主要概念

* MVC
    - `模型(Model)`: 封装了应用程序数据, 通常他们由`POJO`类组成
    - `视图(View)`: 负责渲染模型数据, 一般来说它生成网页
    - `控制器(Controller)`: 负责处理用户请求, 并构建适当的模型, 并将模型传递给视图进行渲染

![原理图](http://www.yiibai.com/uploads/tutorial/20160116/1-1601161F914292.png)

* 核心工作流程

```
+--------------+                              +---------------+
| HTTP Request |                              | HTTP Response |
+------v-------+                              +--------^------+
       |                                               |
+------v-----------------------------------------------^------+
|                      DispatcherServlet                      |
+-----v---^--------------v--^-------------v--^-----------v--^-+
      |   |              |  |             |  |           |  |
+-----v---^------+  +----v--^----+  +-----v--^------+  +-v--^-+
| Handler Mapping|  | Controller |  | View Resolver |  | View |
+----------------+  +------------+  +---------------+  +------+
```

处理流程:
1. 接收到HTTP请求后, `DispatcherServlet`会查询`HandlerMapping`来调用相应的`Controller`
2. `Controller`接收请求并根据使用的`GET`或`POST`方法调用相应的服务方法. 服务方法基于定义的业务逻辑设置`模型数据`, 并将`视图名称`返回给`DispatcherServlet`.
3. `DispatcherServlet`根据视图名称从`ViewResolver`获取`视图`
4. `DispatcherServlet`将`模型数据`传递到最终的`视图`中, 在浏览器显示

主要类的职责:
* `DispatcherServlet`: 处理请求的各个环节. 将请求分发给合适的处理程序, 然后使用视图来返回响应结果
* `HandlerAdapter`: 在DispatcherServlet内部使用的适配器类, 由它实现DispatcherServlet对各种Controller的调用
* `Controller`: 调用业务逻辑生成Model
* `HandlerInterceptor`接口: 拦截器, 在调用Controller前后拦截
* `HandlerMapping`: 告诉DispatcherServlet调用哪个Controller
* `HandlerExecutionChain`: 调用链
* `ModelAndView`: 作为Model
* `ViewResolver`: 帮助DispatcherServlet解析正确的View来渲染页面
* `View`: 负责页面渲染

以上组件中, `HandlerMapping`, `Controller`, `ViewResolver`是`WebApplicationContext`的一部分


## 配置文件

* SpringMVC涉及2个配置文件:
    - `WebContent/WEB-INF/web.xml`, 该配置文件可以:
        - 配置由Spring默认的`DispatcherServlet`来拦截请求
        - 配置自定义的spring mvc配置文件路径
    - `{ProjectName}-servlet.xml`:
        - `DispatcherServlet`的配置文件, 用于创建定义的bean
        - 路径默认在`WebContent/WEB-INF/`下, 但可以通过`web.xml`自定义
        - 该文件可以:
            - 启用注解扫描
            - 设置搜索`@Controller`注解
            - 启用注解驱动
            - 配置InternalResourceViewResolver来定义解析视图名称的规则

```xml
WebContent/WEB-INF/web.xml
--------------------------
<web-app id="WebApp_ID" version="2.4"
    xmlns="http://java.sun.com/xml/ns/j2ee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
    http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <display-name>Spring MVC Application</display-name>

    <!-- 系统配置文件自定义位置
    <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>/WEB-INF/configs/spring/dispatcher-servlet.xml</param-value>
    </context-param>
    <listener>
       <listener-class>
          org.springframework.web.context.ContextLoaderListener
       </listener-class>
    </listener>-->

    <!-- 配置使用Spring MVC 默认的Servlet拦截所有.jsp请求 -->
    <servlet>
        <servlet-name>mvc-dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- DispatcherServlet对应的上下文配置, 默认为/WEB-INF/$servletName-servlet.xml, 为了便于维护, 修改为自定义位置-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/configs/spring/dispatcher-servlet.xml</param-value>
        </init-param>
        <!-- 设置优先级 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloWeb</servlet-name>
        <url-pattern>*.jsp</url-pattern>
    </servlet-mapping>

</web-app>
```

```xml
dispatcher-servlet.xml
----------------------
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- DispatcherServlet上下文, 只搜索@Controller注解的类, 不搜索其他注解的类 -->
    <context:component-scan base-package="com.package.demo">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 配置注解驱动, 启用HandlerMapping -->
    <mvc:annotation-driven/>

    <!-- 配置静态资源映射, css, js, images -->
    <mvc:resources mapping="/resources/*" location="/resources/"/>

    <!-- 配置ViewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name="prefix" value="/WEB-INF/jsp/" />
      <property name="suffix" value=".jsp" />
   </bean>

</beans>
```


## 定义控制器Controller

* 注解: 被注解的方法都叫做`服务方法`, 用于处理特定请求
    - 处理请求的注解
        - `@Controller`: 指定该类作为控制器
        - `@RequestMapping(String url)`: 用于类. 将整个类映射到URL, 当该类所有方法都是处理这个url时可以这样配置, 这样只需要在方法上配置其他属性即可, 如method等
        - `@RequestMapping(value = String url, method = RequestMethod method)`: 用于方法. 指定该方法是处理指定url的指定HTTP方法的. 如果类没有配置url, 则请求以`方法url`为准; 如果类也配置了url, 则请求为`类url/方法url`
            - `value`: 指示处理程序方法映射的url
            - `method`: 定义处理HTTP method请求的服务方法
    - 处理视图属性的注解
        - `@ModelAttribute("HelloWeb")`
    - 处理局部异常的注解
        - `@ExceptionHandler({ExceptionClass1, ExceptionClass2, ...})`: 指定该服务方法需要处理的异常
* 返回值
    - 返回字符串, 作为视图:
        - 返回一个String字符串, 它是渲染视图的名称
        - 如: `return "hello";`
    - 返回ModelAndView:
        - 通过构造一个ModelAndView对象来保存视图和数据并将其返回
        - 如: `return new ModelAndView("success", student);`
    - 返回Model:
        - 即返回一个对象
    - 返回ModelMap:
        - 将模型数据返回页面, 页面url不变. ModelMap以键值对方式存储多个数据
    - 返回Map:
        - 和ModelMap一样
    - 返回List:
        - 将模型数据返回页面, 页面url不变. 数据放在List中
    - 返回Set:
        - 将模型数据返回页面, 页面url不变. 数据放在Set中
    - 重定向请求:
        - 返回一个特殊的字符串`redirect:服务方法名`, 跳转到该方法处理
        - 如: `login()`方法中登录成功, 则重定向到成功页面, `login()`先`return "redirect:success";`, 然后`success()`方法中`return "loginSuccess";`, 跳转到`loginSuccess.jsp`页面
    - 重定向到静态资源:
        - 返回一个特殊的字符串`redirect:/静态资源目录/静态资源文件名.后缀`
        - 需要在`web.xml`中配置`<mvc:resources>`指定静态资源目录
        - 如: `return "redirect:/pages/final.htm";`
    - 转发:
        - 返回一个特殊的字符串`forward:{url}`, 将请求转发给指定url
    - Void:
        - 即没有返回值, 不写`return`. 此时页面还是请求路径的页面
* 注意
    - 在每个服务方法中, 会创建一个模型. 可以设置不同的模型属性, 这些属性将被视图访问来显示视图.
    - 定义的服务方法可以返回一个`String`, 作为渲染视图的名称

### 示例

```java
访问localhost:${port}/hello/mvc所进行的请求处理
@Controller
@RequestMapping("/hello")
public class HelloController {
    @RequestMapping(value = "/mvc", method = RequestMethod.GET)
    public String printHello(ModelMap model) {
        // 向模型中添加属性, 可在视图中获取进行显示
        model.addAttribute("message", "Hello Spring MVC Framework!");
        // 设置视图的名称, 会显示hello.jsp
        return "hello";
   }
}
```


## 定义视图

* Spring MVC支持多种视图:
    - JSP
    - HTML
    - PDF
    - Excel表格
    - XML
    - Velocity模板
    - XSTL
    - JSON
    - Atom
    - RSS源
    - JasperReports


```html
/WebContent/WEB-INF/hello/hello.jsp
-----------------------------------
<html>
    <head>
        <title>Hello Spring MVC</title>
    </head>
    <body>
        <!-- 这是从服务方法中向model添加的属性中获取到的, 因为模型传递到了视图 -->
        <h2>${message}</h2>
    </body>
</html>
```


## ModelAndView

* 代表了渲染视图所使用的Model和View, 通过ModelAndView对象来封装数据
* 构造方法:
    - `ModelAndView(String viewName)`: 返回视图名称
    - `ModelAndView(String viewName, Map model)`: 返回视图名称, 并使用Map携带Model数据
    - `ModelAndView(String viewName, String modelName, Object modelObject)`: 返回单个model时使用, 例如API, 并不需要返回视图


## 异常处理

* 处理局部异常
    - `@ExceptionHandler`注解
* 处理全局异常
    - 使用现有异常处理器: `SimpleMappingExceptionResolver`
    - 使用自定义全局异常处理器: 定义类实现`HandlerExceptionResolver`接口, 重写`public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)`方法
* 异常处理方式: dao, service, controller的所有异常都往上抛, 最终由全局异常处理器处理

```java
// 自定义异常
public class CustomException extends Exception {

    //异常信息
    public String message;

    public CustomException(String message) {
        super(message);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

}
```

```xml
xxx-servlet.xml
---------------
<!-- springmvc提供的简单异常处理器 -->
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
     <!-- 定义默认的异常处理页面 -->
    <property name="defaultErrorView" value="/WEB-INF/jsp/error.jsp"/>
    <!-- 定义异常处理页面用来获取异常信息的变量名，也可不定义，默认名为exception -->
    <property name="exceptionAttribute" value="ex"/>
    <!-- 定义需要特殊处理的异常，这是重要点 -->
    <property name="exceptionMappings">
        <props>
            <prop key="ssm.exception.CustomException">/WEB-INF/jsp/custom_error.jsp</prop>
        </props>
        <!-- 还可以定义其他的自定义异常 -->
    </property>
</bean>
```


## SpringMVC实现上传文件

* 步骤
    - 添加`commons-fileupload`依赖
    - 在`dispatcher-servlet.xml`中配置`CommonsMultipartResolver`上传文件解析器
    - 在JSP页面增加表单上传文件
    - 编写`Controller`处理文件上传, 读写流

* 文件上传依赖

```xml
pom.xml
-------
<!-- 文件上传所依赖的jar包 -->  
<dependency>  
    <groupId>commons-fileupload</groupId>  
    <artifactId>commons-fileupload</artifactId>  
    <version>1.3.2</version>  
</dependency>
```

* 在`dispatch-servlet.xml`中配置上传文件解析器

```xml
dispatch-servlet.xml
--------------------
<!-- 添加p标签 -->
xmlns:p="http://www.springframework.org/schema/p"

<!-- 上传文件的设置 ，maxUploadSize=-1，表示无穷大。uploadTempDir为上传的临时目录 -->  
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"    
    p:defaultEncoding="UTF-8"    
    p:maxUploadSize="5400000"    
    p:uploadTempDir="fileUpload/temp"    
></bean>
```

* 上传文件的JSP

```html
<body>
	<h2>上传文件</h2>
	<form method="post" action="/upload" enctype="multipart/form-data">
		<table>
			<tr>
				<td><input type="file" name="file"></td>
			</tr>
			<tr>
				<td><input type="submit" name="upload"></td>
			</tr>
		</table>
	</form>
</body>
```

* 处理上传的Controller

```java
@Controller
@RequestMapping("/upload")
public class FileUploadController {

	@RequestMapping(method = RequestMethod.POST)
	public String uploadFile(@RequestParam("file")CommonsMultipartFile file) {
		System.out.println(file.getOriginalFilename() + ", size=" + file.getSize());
        // todo 读写流
		return "upload-success";
	}
}
```
