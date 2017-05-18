# SpringMVC

* 负责Web层, 替代Struts2. 是Spring框架中的一个组件
* 处理请求返回响应
* 请求驱动
* 采用`前端控制器`设计模式
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


## DispatcherServlet

* `DispatcherServlet`负责将所有的请求都由其分发到所需的功能中
* `DispatcherServlet`采用了`前端控制器`的设计模式
* `DispatcherServlet`是一个Servelt, 继承自`HttpServlet`, 所以也要在`web.xml`中声明为Servlet, 并映射对应的URL
* 每个`DispatcherServlet`都持有一个自己的上下文对象`WebApplicationContext`
* `DispatcherServlet`在初始化过程中, Spring MVC会在应用的`WEB-INF`目录下查找一个名为`{servlet-name}-servlet.xml`的配置文件, 并创建其中定义的bean
  - `{servlet-name}`就是`web.xml`中配置`DispatcherServlet`的`<servlet-name></servlet-name>`的值

### WebApplicationContext中特殊的bean类型

* `HandlerMapping`: 处理器映射
    - 根据某些规则将进入容器的请求映射到具体的处理器以及一系列的前处理器和后处理器(即处理器拦截器)
* `HandlerAdapter`: 处理器适配器
    - 拿到请求所对应的处理器后, 适配器负责其调用该处理器, 使得`DispatcherServlet`无序关心具体的调用细节
* `HandlerExceptionResolver`: 处理器异常解析器
    - 负责将捕获的异常反射到不同的视图上去
* `ViewResolver`: 视图解析器
    - 负责讲一个代表逻辑视图名的字符串映射到实际的视图类型View上
* `LocaleResolver`: 地区解析器
    - 负责解析客户端所在地区信息和时区信息, 为国际化视图定制提供支持
* `LocaleContextResolver`: 地区上下文解析器
    - 负责解析客户端所在地区信息和时区信息, 为国际化视图定制提供支持
* `ThemeResolver`: 主题解析器
    - 负责解析web应用中可用的主题, 提供一些个性化定制的布局
* `MultipartResolver`:
    - 解析`multi-part`传输请求, 支持文件上传
* `FlashMapManager`: FlashMap管理器
    - 能够存储并取回两次请求之间的`FlashMap`对象, 后者可用于在请求之间传递数据, 通常是在请求重定向的情况下使用

### DispatcherServlet工作流程

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


## 控制器(Controller)的实现

* 控制器作为应用程序逻辑的处理入口, 负责调用开发人员实现的一些服务
* 控制器接收并解析用户的请求, 将其转换成一个模型交给视图, 由视图渲染出页面, 最终呈献给用户
* 控制器中的方法称为处理器方法(Handler method)

### @Controller 定义控制器

* 开启注解扫描:
    - 在xml中配置: `<context:component-scan base-package="org.springframework.samples.petclinic.web"/>`
    - `DispatcherServlet`会扫描所有注解了`@Controller`的类, 检测其中通过`@RequestMapping`注解配置的方法
* `@Controller`: 指定一个类作为控制器


## @RequestMapping 映射请求路径

* `@RequestMapping`注解可以将URL映射到2个地方:
    - 类: 将一个特定的请求路径映射到控制器上, 即定义该类处理统一的一个URL
    - 方法: 细化类的映射, 即定义不同方法处理同一个URL的子路径或不同HTTP方法

```java
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

    @Autowired
    public AppointmentsController(AppointmentBook appointmentBook) {
        this.appointmentBook = appointmentBook;
    }

    @RequestMapping(method = RequestMethod.GET)
    public Map<String, Appointment> get() {
        return appointmentBook.getAppointmentsForToday();
    }

    @RequestMapping(path = "/{day}", method = RequestMethod.GET)
    public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso = ISO.DATE) Date day, Model model) {
        return appointmentBook.getAppointmentsForDay(day);
    }
}
```

### URI模板

* `{变量}`: 在URI中使用该模板代替路径
* `@PathVariable`: 将URI模板与参数绑定
    - `@PathVariable 数据类型 模板变量名`
    - `@PathVariable("模板变量名") 数据类型 新变量名`
* 一个方法可以拥有任意数量的URI模板和`@PathVariable`注解
* URI模板支持使用正则表达式

```java
// 直接使用模板变量名
@RequestMapping(path = "/owner/{ownerId}", method = RequestMethod.GET)
public String findOwner(@PathVariable String ownerId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    model.addAttribute("owner", owner);
    return "displayOwner";
}

// 不使用模板变量名, 使用新的变量名
@RequestMapping(path = "/owner/{ownerId}", method = RequestMethod.GET)
public String findOwner(@PathVariable("ownerId") String theOwner, Model model) {
    Owner owner = ownerService.findOwner(theOwner);
    model.addAttribute("owner", owner);
    return "displayOwner";
}

// 使用正则的URI模板
@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{extension:\\.[a-z]+}")
    public void handle(@PathVariable String version, @PathVariable String extension) {
        // 代码部分省略...
    }
}
```

### Path Pattern路径模板

* 支持Ant风格的路径模式, 如`/myPath/*.do`
* 当 URL 同时匹配多个模板时, 模板会进行排序以便找到最匹配的
    - 匹配原则:
        - 原则1: URI模板变量的数目和通配符数量的 **总和** 最少的那个路径模板更匹配
            - 如: `/hotels/{hotel}/*`和`/hotels/{hotel}/**`, 前者有1个变量和1个通配符, 后者有2个变量和2个通配符, 所以前者总和最少更匹配
        - 原则2: 如果两个模板的URI模板数量和通配符数量总和一致, 则路径更长的模板更匹配
            - 如: `/foo/bar*`和`/foo/*`, 两者都只有1个通配符, 前者路径更长更匹配
        - 原则3: 如果两个模板的数量和长度均一致, 则具有更少通配符的模板更匹配
            - 如: `/hotels/{hotel}`和`/hotels/*`, 前者有1个变量, 后者有1个通配符, 数量总和和长度一致, 但前者通配符更少更匹配
        - 其他原则:
            - 默认通配模式: `/**`比其他所有模式都更不匹配, 如`/api/{a}/{b}/{c}`比`/**`更匹配
            - 更多可参考`AntPatternComparator`和`AntPathMatcher`

### 后缀模式匹配

* Spring MVC 默认采用`".*"`的后缀模式来进行路径匹配, 如`/person`也会被映射到`/person.*`
* 可以关闭默认的后缀模式匹配, 避免产生歧义或安全问题
* RFD(Reflected file download)攻击依赖于浏览器跳转到下载页面, 将特定格式的响应当做可执行脚本
    - `@ResponseBody`和`@ResponseEntity`方法都有风险
    - 防范方法: 在请求头增加一行`Content-Disposition:inline;filename=f.txt`指定固定的下载文件名

### 矩阵变量

* `Matrix Variable`, 矩阵参数, 在路径中包含键值对, 如`/cars;color=red,blue;year=2012`
* 需要在配置中启用矩阵变量: `<mvc:annotation-driven enable-matrix-variables="true"/>`
* 使用`@MatrixVariable`注解来获取路径中的矩阵变量
    - `name`: 键
    - `pathVar`: 路径
    - `required`: 是否必须
    - `defaultValue`: 默认值
* 也可以通过Map将矩阵变量中的键值对都保存起来: `@MatrixVariable Map<String, String> matrixVars`

```java
// GET /owners/42;q=11/pets/21;q=22

@RequestMapping(path = "/owners/{ownerId}/pets/{petId}", method = RequestMethod.GET)
public void findPet(
    @MatrixVariable(name="q", pathVar="ownerId") int q1,
    @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22

}
```

### 可消费媒体类型

* 可以在`@RequestMapping`中使用`consumes="xxx"`指定匹配请求的`Content-Type`头来缩小映射范围, 如: `@RequestMapping(method = RequestMethod.POST, consumes="application/json")`
* 媒体类型表达式中可以使用`!`来表示相反, 如: `consume=!text/plain`匹配所有不含`text/plain`的请求

### 可生产媒体类型

* 可以在`@RequestMapping`中使用`produces="xxx"`指定匹配请求的`Accept`头来缩小映射范围, 如: `@RequestMapping(method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_UTF8_VALUE)`
* 媒体类型表达式中可以使用`!`来表示相反, 如: `consume=!text/plain`匹配所有不含`text/plain`的请求

### 使用请求头筛选来匹配请求

* 格式: `params="键=值"`
* 允许的筛选方式:
    - 键: `params="myParam"`
    - 键取反: `params="!myParam"`
    - 键值对: `params="myParam=myValue"`

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

    @RequestMapping(path = "/pets/{petId}", method = RequestMethod.GET, params="myParam=myValue")
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
        // 实际实现省略
    }
}
```


## 如何定义@RequestMapping注解的处理方法

* 方法支持的参数类型
    - `ServletRequest`或`HttpServletRequest`: Servlet请求或相应对象
    - `HttpSession`: 会话对象. 要求session必须存在, 不能为null
    - `org.springframework.web.context.request.WebRequest`或`org.springframework.web.context.request.NativeWebRequest`: 用于存取一般的请求参数和请求/会话范围的attribute
    - `java.util.Locale`: 请求的地区信息, 通过地区解析器获得, 在SpringMVC中是`LocaleResolver`或`LocaleContextResolver`
    - `java.io.InputStream`或`java.io.Reader`: 与Servlet拿到的输出流是一样的
    - `org.springframework.http.HttpMethod`: HTTP的请求方法
    - `java.security.Principal`: 被认证用户信息
    - `@PathVariable`注解的参数: URI模板的路径变量
    - `@MatrixVariable`注解的参数: URI模板的矩阵变量
    - `@RequestParam`注解的参数: 请求参数
    - `@RequestHeader`注解的参数: HTTP请求头
    - `@RequestBody`注解的参数: HTTP请求体
    - `@RequestPart`注解的参数: `multipart/form-data`请求内容
    - `HttpEntity<?>`: HTTP请求实体, 包括HTTP请求头和请求体
    - `java.util.Map`或`org.springframework.io.Model`或`org.springframework.ui.ModelMap`: 增强model
    - `org.springframework.web.servlet.mvc.support.RedirectAttributes`: 指定重定向下要使用到的属性集和flash属性
    - 命令或表单对象: 将请求参数绑定到bean
    - `org.springframework.validation.Errors`或`org.springframework.validation.BindingResult`: 验证结果对象, 存储前面的命令或表单对象的验证结果
        - 这两个类的参数必须紧跟在其所绑定的验证对象后面, 顺序不能错
    - `org.springframework.web.bind.support.SessionStatus`: 标记当前表单已经处理结束, 出发一些清理操作
    - `org.springframework.web.util.UriComponentsBuilder`: 请求URL的构造信息对象, 可获取主机名, 端口号, 资源类型, 上下文路径, servlet映射中的literal part等
* 方法支持的返回值类型
    - `ModelAndView`对象: model隐含填充了命令对象, 注解了`@ModelAttribute`字段的存取器被调用所返回的值
    - `Model`对象: 视图名称默认由`RequestToViewNameTranslator`决定, model隐含填充了命令对象以及注解了`@ModelAttribute`字段的存取器被调用所返回的值
    - `Map`对象: 用于暴露model, 其中视图名称默认由`RequestToViewNameTranslator`决定, model隐含填充了命令对象以及注解了`@ModelAttribute`字段的存取器被调用所返回的值. handler方法也可以增加一个`Model`类型的方法参数来增强model
    - `String`对象: 该字符串会被解析成一个逻辑视图名. model默认填充了命令对象以及注解了`@ModelAttribute`字段的存取器被调用所返回的值. handler方法也可以增加一个`Model`类型的方法参数来增强model
    - `void`: 如果处理器方法中已经对response响应数据进行了处理(如在方法参数中定义一个`ServletResponse`或`HttpServletResponse`并直接向该响应写内容), 则可以返回void
    - 如果处理器方法注解了`@ResponseBody`, 则返回类型江北写到HTTP响应提中, 而返回值会被`HttpMessageConverters`转换成方法声明的参数类型
    - `HttpEntity<?>`或`ResponseEntity<?>`对象: 用于提供对Servlet HTTP响应头和相应内容的存取. 对象会被`HttpMessageConverters`转换成响应流
    - `HttpHeaders`对象: 返回一个不含响应体的response
    - `Callable<?>`对象: 异步返回方法值时使用. 该过程由SpringMVC自身的线程来管理
    - `DeferredResult<?>`对象: 当方法的返回值交由线程自身决定时使用
    - `ListenableFuture<?>`对象: 当方法的返回值交由线程自身决定时使用
    - `ResponseBodyEmitter`对象: 异步地向响应体中同时写多个对象
    - `SseEmitter`对象: 异步地向响应体中写服务器端事件(Server-sent Events)
    - `StreamingResponseBody`对象: 异步地向响应对象的输出流中写内容
    - 其他任何返回类型: 会被处理成model的一个属性并返回给视图, 该属性的名称为方法级的`@ModelAttibute`所注解的字段名(或以返回类型的类名作为默认的属性名).

### @RequestParam 映射请求参数至控制器方法形参

* 该注解适用于映射URL中的请求参数
* 使用位置: 控制器方法的形参前
* 格式:
    - `public 返回值类型 控制器方法名(@RequestParam("{GET请求参数的key}") 数据类型 变量名) {}`
        - 将请求的参数映射到控制器方法的形参上. 默认该请求参数是必须的
    - `public 返回值类型 控制器方法名(@RequestParam(path="{GET请求参数的key}", request={参数是否必须}) 数据类型 变量名) {}`
        - 将请求的参数映射到控制器方法的形参上, 同时指定该参数是否必须提供
* 若`@RequestParam`注解的参数类型是`Map<String, String>`或`MultiValueMap<String, String>`, 则该Map中会自动填充所有的请求参数

```java
@Controller
@RequestMapping("/pets")
@SessionAttributes("pet")
public class EditPetForm {
    @RequestMapping(method = RequestMapping.GET)
    public String setupForm(@RequestParam("petId") int petId, ModelMap model) {
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }
}
```

### @RequestBody 映射请求体

* 该注解适用于映射请求体中的参数
* 使用位置: 控制器方法的形参前
* 格式:
    - `public 返回值类型 控制器方法名(@RequestBody 数据类型 变量名) {}`
        - 将请求体映射到控制器方法的形参上
* 请求体到方法形参的转换由`HttpMessageConverter`完成, 它负责将HTTP请求信息转换为对象, 或是将对象转换为一个HTTP响应体. 对于该注解, `RequestMappingHandlerAdapter`提供了集中默认的`HttpMessageConverter`的支持:
    - `ByteArrayHttpMessageConverter`用以转换字节数组
    - `StringHttpMessageConverter`用以转换字符串
    - `FormHttpMessageConverter`用以将表格数据转换成`MultiValueMap<String, String>`或从`MultiValueMap<String, String>`中转换出表格数据
    - `SourceHttpMessageConverter`用于`javax.xml.transform.Source`类的互相转换
* 使用`@RequestBody`的方法形参还可以被`@Valid`注解, 从而被已配置的`Validator`示例来对该参数进行验证

```java
@RequestMapping(path = "/something", method = RequestMethod.PUT)
public void handle(@RequestBody String body, Writer writer) throws IOException {
    writer.write(body);
}
```

### @ResponseBody 映射响应体

* 该注解用于映射返回的响应体
* 使用位置: 方法上
* 对象到响应体的转换也是使用`HttpMessageConverter`

```java
@RequestMapping(path = "/something", method = RequestMethod.PUT)
@ResponseBody
public String helloWorld() {
    return "Hello World";  // 因为使用了@ResponseBody, 所以返回字符串, 而不是字符串所代表的的视图
}
```

### @RestController 创建Rest控制器

* 该注解用于将控制器类定义为Rest控制器, 从而让所有控制器方法返回的都是对象
* 使用位置: 控制器类上
* 作用: 如果希望实现Rest返回, 不需要在每个控制器方法上都加上`@ResponseBody`来返回对象, 只需要对控制器的类使用`@RestController`的注解
* 相当于结合了`@Controller`和`@ResponseBody`

### 使用HttpEntity和ResponseEntity

* `HttpEntity`: 获取请求体中的内容, 还可以存取请求头. 与`@RequestBody`相似
* `ResponseEntity`: 获取响应体中的内容, 还可以存取响应头. 与`@ResponseBody`相似
* 也是使用`HttpMessageConverter`来将请求流和响应流进行转换

```java
@RequestMapping("/something")
public ResponseEntity<String> handle(HttpEntity<byte[]> requestEntity) throws UnsupportedEncodingException {
    // 从HttpEntity中获取请求头
    String requestHeader = requestEntity.getHeaders().getFirst("MyRequestHeader");
    // 从HttpEntity中获取请求体
    byte[] requestBody = requestEntity.getBody();

    // do something with request header and body

    HttpHeaders responseHeaders = new HttpHeaders();
    responseHeaders.set("MyResponseHeader", "MyValue");
    // 创建ResponseEntity并设置响应头
    return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED);
}
```


### 小节

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
