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


### @RequestMapping 映射请求路径

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


### 如何定义@RequestMapping注解的处理方法

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

### @RequestHeader 映射请求头属性

* 该注解适用于控制器方法的形参上
* 能够将一个请求头属性绑定到一个方法形参上
* 可以使用该注解应用在`Map<String, String>`, `MultiValueMap<String, String>`, 或`HttpHeaders`类型的参数上, 这样所有请求头属性值都会被填充到map中

```java
/*
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
*/

@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
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


### @ModelAttribute 注解方法和方法的形参

* 该注解可以应用在方法或方法的形参上
* 对方法使用注解
    - 作用: 添加一个或多个属性到model上, 如公共需要的属性或数据, 例如一个下拉列表所预设的几种状态. 这样的方法能接收与`@RequestMapping`注解相同的参数类型, 只不过不能直接被映射到具体的请求上.
    - 2种风格:
        - 方法通过返回值的方式默认地添加一个属性, 可以使用`@ModelAttribute("{key}")`来修改model中的键
        - 方法接收一个`Model`对象, 然后可以向其中添加任意数量的属性
    - 特点
        - 一个控制器可以拥有数量不限的`@ModelAttribute`方法
        - 注解了`@ModelAttribute`的方法会在`@RequestMapping`方法之前被调用
* 对形参使用注解
    - 作用: 从model中获取参数值.
    - 数据绑定: 如果model中找不到参数值, 则该参数会先被实例化, 然后被添加到model中. model中存在以后, 请求中所有名称匹配的参数都会填充到该参数中
    - 数据校验: 数据绑定过程中可能会出现一些错误, 如没有提供必要的字段, 类型转换错误等. 可以在注解了`@ModelAttribute`的参数后紧接着声明一个`BindingResult`参数. 或是添加一个`@Valid`注解到`@ModelAttribute`注解前

```java
// 注解到方法上, 风格1, 返回值被添加到model中的account属性值中
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

// 注解到方法上, 风格2, 添加多个属性到model中
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));
    // add more ...
}

// 注解到形参上
@RequestMapping(path = "/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {
    new PetValidator().validate(pet, result);
    if (result.hasErrors()) {
        return "petForm";
    }
}
```

### @SessionAttributes 让HTTP session保存model数据

* 该注解适用于控制器类上
* 用于声明某个特定处理器所使用的session属性. 一般用于在请求之间保存一些表单数据的bean

```java
@Controller
@RequestMapping("/editPet.do")
@SessionAttributes("pet")
public class EditPetForm {
    // ...
}
```

### 配置拦截`application/x-www-form-urlencoded`数据

* 由于Servlet中只支持通过`HTTP POST`方法提交表单, 但非浏览器客户端实际上也可以通过`PUT`或`PATCH`方法来提交表单, 为了处理这种情况, 需要配置拦截器拦截`content type`为`application/x-www-form-urlencoded`的请求
* 在`web.xml`文件中配置:

```xml
<filter>
    <filter-name>httpPutFormFilter</filter-name>
    <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpPutFormFilter</filter-name>
    <servlet-name>dispatcherServlet</servlet-name>
</filter-mapping>

<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
```

### @CookieValue 映射cookie

* 该注解适用于控制器方法的形参上
* 该注解能将HTTP的cookie值绑定到控制器方法的形参上
* 若注解的目标方法形参类型不是`String`类型, 则会自动转换为`String`类型

```java
// JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84

@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie) {
    //...
}
```

### 方法形参的类型转换

* 从请求参数, 路径变量, 请求头属性, cookie中获取到的`String`类型的值, 可能需要转换为形参的类型. 这一过程会自动转换.
* 简单的类型如int, long, Date内置了类型转换的支持
* 其他类型可以通过`WebDataBinder`实现, 或为`Formatters`配置一个`FormattingConversionService`实现

### 异步请求的处理

* 实现异步返回值有2种方式
    - 返回一个`java.util.concurrent.Callable`对象
        - 这些线程由SpringMVC来管理, 通过一个`TaskExecutor`在另外的线程中调用`Callable`, 当`Callable`返回时, 请求再携带`Callable`返回的值, 再次被分配到Servlet容器中恢复处理流程
    - 返回一个`DefferredResult`对象
        - 返回值可以由任何一个线程产生, 包括不是由SpringMVC管理的线程
* 实现异步请求的原理
    - 底层都依赖Servlet 3.0的异步请求处理特性:
        - 一个servlet请求`ServletRequest`可以通过调用`request.startAsync()`方法而进入异步模式. 这样做的主要结果就是该servlet以及所有的过滤器都可以结束,但其响应(response)会留待异步处理结束后再返回
        - 调用`request.startAsync()`方法会返回一个`AsyncContext`对象, 可用它对异步处理进行进一步的控制和操作. 比如说它也提供了一个与转向(forward)很相似的dispatch方法, 只不过它允许应用恢复Servlet容器的请求处理进程
        - `ServletRequest`提供了获取当前`DispatherType`的方式, 后者可以用来区别当前处理的是原始请求, 异步分发请求, 转向, 或是其他类型的请求分发类型
    - Callable异步请求的原理:
        - 控制器先返回一个`Callable`对象
        - Spring MVC开始进行异步处理, 并把该`Callable`对象提交给另一个独立线程的执行器`TaskExecutor`处理
        - `DispatcherServlet`和所有过滤器都退出Servlet容器线程, 但此时方法的响应对象仍未返回
        - `Callable`对象最终产生一个返回结果, 此时Spring MVC会重新把请求分派回Servlet容器, 恢复处理
        - `DispatcherServlet`再次被调用, 恢复对`Callable`异步处理所返回结果的处理
    - DeferredResult异步请求的原理:
        - 控制器先返回一个`DeferredResult`对象, 并把它存取在内存(队列或列表等)中以便存取
        - Spring MVC开始进行异步处理
        - `DispatcherServlet`和所有过滤器都退出Servlet容器线程, 但此时方法的响应对象仍未返回
        - 由处理该请求的线程对`DeferredResult`进行设值, 然后Spring MVC会重新把请求分派回Servlet容器, 恢复处理
        - `DispatcherServlet`再次被调用, 恢复对该异步返回结果的处理
* 异步请求中的异常处理
    - 当`Callable`中发生异常时, SpringMVC会返回一个`Exception`对象来替代正常的返回值, 然后容器恢复对此异步请求异常的处理
    - 当`DeferredResult`中发生异常时, 可以调用`Exception`实例的`setResult()`方法或是`setErrorResult()`方法
* 拦截异步请求
    - `HandlerInterceptor`可以实现`AsyncHandlerInterceptor`接口拦截异步请求. 当异步请求开始时, 会调用`afterConcurrentHandlingStarted()`方法, 而不是`postHandle()`或`afterCompletion()`方法
    - 如需更深入集成, 例如处理timeout事件, 则`HandlerInterceptor`需要注册一个`CallableProcessingInterceptor`或`DeferredResultProcessingInterceptor`拦截器
    - `DeferredResult`提供了`onTimeout(Runnable)`和`onCompletion(Runnable)`等方法
    - `Callable`对于请求超时和完成后的事件拦截, 可以将其封装在一个`WebAsyncTask`中


```java
// Callable方式
@RequestMapping(method=RequestMethod.POST)
public Callable<String> processUpload(final MultipartFile file) {
    // 返回一个Callable
    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };
}

// DeferredResult方式
@RequestMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    return deferredResult;
}

// In some other thread...
deferredResult.setResult(data);
```

* 异步请求的配置
    - 异步请求依赖于Servlet 3.0. 确保`web.xml`的版本在3.0或以上
    - 异步请求必须在`web.xml`中将`DispatcherServlet`下的子元素`<async-supported>true</async-supported>`设置为`true`
    - 所有参与异步请求处理的Filter必须配置为支持`ASYNC`类型的请求分发

```xml
web.xml
-------
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
            http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <filter>
        <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
        <filter-class>org.springframework.~.OpenEntityManagerInViewFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>

    <filter-mapping>
        <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>ASYNC</dispatcher>
    </filter-mapping>

</web-app>
```


### HTTP流(streaming)

* 用于在一个HTTP响应中推送多个事件
* 在方法中返回`ResponseBodyEmitter`对象可以发送多个对象
    - `ResponseBodyEmitter`可以放到`ResponseEntity`中, 从而可以自定义响应状态和响应头
* 在方法中返回一个`SseEmitter`对象, 可以实现服务端事件推送.
    - 它是`ResponseBodyEmitter`的子类, 提供了对`服务器端事件`的支持
    - IE不支持该技术
* 在方法中返回一个`StreamingResponseBody`来直接返回OutputStream流

```java
// ResponseBodyEmitter
@RequestMapping("/events")
public ResponseBodyEmitter handle() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();


// StreamimgResponseBody
@RequestMapping("/download")
public StreamingResponseBody handle() {
    return new StreamingResponseBody() {
        @Override
        public void writeTo(OutputStream outputStream) throws IOException {
            // write...
        }
    };
}
```


### 处理器映射(handler mapping)

* 在旧版本的SpringMVC中, 需要定义多个`HandlerMapping`的bean来将请求映射到控制器方法上.
* 在新版本的SpringMVC中已经有了注解, 不再需要定义这些bean

### 处理器拦截器(HandlerInterceptor)

* 拦截器可用于对特殊功能进行操作, 如验证用户身份
* 配置拦截器, 需要实现`HandlerInterceptor`接口, 或继承`HandlerInterceptorAdapter`类, 重写3个方法:
    - `boolean preHandler()`: 在处理器执行前被执行. 返回值决定是否继续执行处理链中的部件, 返回`true`则处理器链会继续执行; 返回`false`则`DispatcherServlet`认为拦截器自身已经完成了请求的处理, 其余拦截器及处理链都不会再执行
    - `postHandle()`: 在处理器执行后被执行
    - `afterCompletion()`: 在整个请求处理完成后被执行


## 视图解析

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
* 处理视图的2个重要接口
    - `ViewResolver`: 视图解析器. 负责处理视图名与实际视图之间的映射关系
    - `View`: 视图. 负责准备请求, 并将请求的渲染交给某种具体的视图技术实现

### ViewResolver解析视图

* 常见视图解析器
    - `AbstractCachingViewResolver`: 一个抽象的视图解析器类, 提供了缓存视图的功能. 通常视图在能够被使用之前需要经过准备. 继承这个基类的视图解析器即可以获得缓存视图的能力
    - `XmlViewResolver`: 视图解析器接口`ViewResolver`的一个实现, 该类接受一个XML格式的配置文件. 该XML文件必须与Spring XML的bean工厂有相同的DTD。默认的配置文件名是`/WEB-INF/views.xml`
    - `ResourceBundleViewResolver`: 视图解析器接口`ViewResolver`的一个实现, 采用bundle根路径所指定的`ResourceBundle`中的bean定义作为配置. 一般bundle都定义在classpath路径下的一个配置文件中. 默认的配置文件名为`views.properties`
    - `UrlBasedViewResolver`: `ViewResolver`接口的一个简单实现. 它直接使用URL来解析到逻辑视图名, 除此之外不需要其他任何显式的映射声明. 如果你的逻辑视图名与你真正的视图资源名是直接对应的, 那么这种直接解析的方式就很方便, 不需要你再指定额外的映射
    - `InternalResourceViewResolver`: `UrlBasedViewResolver`的一个好用的子类. 它支持内部资源视图(具体来说, Servlet和JSP), 以及诸如`JstlView`和`TilesView`等类的子类. 可以使用`setViewClass(..)`为所有视图设置视图的类. 更多的细节, 请见UrlBasedViewResolver类的java文档
    - `VelocityViewResolver / FreeMarkerViewResolver`: `UrlBasedViewResolver`下的实用子类, 支持`Velocity`视图`VelocityView`(Velocity模板)和FreeMarker视图`FreeMarkerView`以及它们对应子类
    - `ContentNegotiatingViewResolver`: 视图解析器接口`ViewResolver`的一个实现, 它会根据所请求的文件名或请求的`Accept`头来解析一个视图

### 视图链

* Spring支持同时使用多个视图解析器, 因此可以配置一个解析器链
* 可以通过在`applicationContext.xml`中配置各种视图解析器即可, 通过`order`属性确定次序, order值越大, 在视图链的位置就越靠后
* 如果视图解析器不能返回一个视图, 则Spring会继续检查context中其他视图解析器. 如果所有视图解析器都不能返回一个视图, 则会抛出一个`ServletException`

```xml
applicationContext.xml
----------------------
<bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>

<bean id="excelViewResolver" class="org.springframework.web.servlet.view.XmlViewResolver">
    <property name="order" value="1"/>
    <property name="location" value="/WEB-INF/views.xml"/>
</bean>


views.xml
---------
<beans>
    <bean name="report" class="org.springframework.example.ReportExcelView"/>
</beans>
```

### 视图重定向和转发

* 应用场景: 有时, 我们想要在视图渲染之前, 先把一个HTTP重定向请求发送回客户端. 比如, 当一个控制器成功地接受到了POST过来的数据, 而响应仅仅是委托另一个控制器来处理(比如一次成功的表单提交)时, 我们希望发生一次重定向. 在这种场景下, 如果只是简单地使用内部转发, 那么意味着下一个控制器也能看到这次POST请求携带的数据, 这可能导致一些潜在的问题, 比如可能会与其他期望的数据混淆等. 此外, 另一种在渲染视图前对请求进行重定向的需求是, 防止用户多次提交表单的数据. 此时若使用重定向, 则浏览器会先发送第一个POST请求; 请求被处理后浏览器会收到一个重定向响应, 然后浏览器直接被重定向到一个不同的URL, 最后浏览器会使用重定向响应中携带的URL发起一次GET请求. 因此, 从浏览器的角度看, 当前所见的页面并不是POST请求的结果, 而是一次GET请求的结果. 这就防止了用户因刷新等原因意外地提交了多次同样的数据. 此时刷新会重新GET一次结果页, 而不是把同样的POST数据再发送一遍
* 重定向的方式:
    - 返回`RedirectView`. 在控制器中创建并返回一个`RedirectView`实例. 这会使`DispatcherServlet`放弃使用一般的视图解析机制. 紧接着`RedirectView`会调用`HttpServletResponse.sendRedirect()`方法发送一个HTTP重定向响应给客户端浏览器
    - 返回`redirect:`前缀. 如`redirect:{视图名}`
    - 返回`forward:`前缀. 如`forward:{视图名}`



## FlashAttribute

* 用于通过一个请求为另一个请求存储有用的属性
* 使用场景: 在重定向的时候最常使用, 比如常见的 `POST/REDIRECT/GET` 模式. Flash属性会在重定向前被暂时地保存起来(通常是保存在session中), 重定向后会重新被下一个请求取用并立即从原保存地移除
* 2个操作Flash属性的类
    - `FlashMap`: 存储flash属性
    - `FlashMapManager`: 存储, 取回, 管理`FlashMap`实例
* 对Flash属性的支持默认是启用的. 控制器通常不需要直接使用`FlashMap`, 一般通过`@RequestMapping`接收一个`RedirectAttribute`类型的参数, 然后向其中添加flash属性
* 在并发情况下, `RedirectView`会自动为一个`FlashMap`实例记录其目标重定向URL的路径和查询参数, 来匹配信息, 减少并发问题
* 一般只在重定向的场景下推荐使用flash属性


## URI构造

* `UriComponentsBuilder`: 提供构造URI的机制
* `UriComponents`: 提供加密URI的机制
    - 是一个不可变对象, `expand()`和`encode()`操作都会返回一个新的实例
* `ServletUriComponentsBuilder`: 提供静态的工厂方法, 从Servlet请求中获取URL信息

```java
// 通过URI模板字符串来填充并加密一个URI
UriComponents uriComponents = UriComponentsBuilder.fromUriString(
        "http://example.com/hotels/{hotel}/bookings/{booking}").build();

URI uri = uriComponents.expand("42", "21").encode().toUri();

// 通过URI组件创建并加密一个URI
UriComponents uriComponents = UriComponentsBuilder.newInstance()
        .scheme("http").host("example.com").path("/hotels/{hotel}/bookings/{booking}").build()
        .expand("42", "21")
        .encode();

// ServletUriComponentsBuilder
ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromRequest(request)
        .replaceQueryParam("accountId", "{id}").build()
        .expand("123")
        .encode();

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromServletMapping(request)
                    .path("/accounts").build()
```

### 为控制器和方法指定URI

* `MvcUriComponentsBuilder`
    - `fromMethodName()`
    - `fromMethodCall()`

```java
@Controller
@RequestMapping("/hotels/{hotel}")
public class BookingController {
    @RequestMapping("/bookings/{booking}")
    public String getBooking(@PathVariable Long booking) {
    // ...
    }
}


UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodName(BookingController.class, "getBooking", 21).buildAndExpand(42);
URI uri = uriComponents.encode().toUri();


UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);
URI uri = uriComponents.encode().toUri();
```

## 用于国际化的地区信息(Locales)

* 地区信息相关的解析器
    - `LocaleResolver`: 提供对用户地区信息的消息解析能力.
        - 当请求被处理时, `DispatcherServlet`会寻找一个地区解析器. 如果找到了, 就会使用它来设置地区相关信息
    - `LocaleContextResolver`接口: 除了能获取到LocaleResolver的信息, 还能获取到时区信息
    - `AcceptHeaderLocaleResolver`: 获取`accept-language`请求头
    - `CookieLocaleResolver`: 获取cookie中的`Locale`或`TimeZone`信息
        - xml中支持的属性:
            - `cookieName`: 默认值`classname+LOCALE`. cookie名
            - `cookieMaxAge`: 默认值`Integer.MAX_INT`. cookie在客户端保存的最长时间, 如果为-1, 则cookie在浏览器关闭后就失效
            - `cookiePath`: 默认值`/`. 限制了cookie仅对站点下的某些特定路径可见. 如果制定了cookiePath, namecookie将仅对该路径及其自路径下的所有站点可见
    - `SessionLocaleResolver`: 从session中获取用户请求的`Locale`和`TimeZone`
        - 与外部session管理机制没有关系. 仅能简单的从当前请求`HttpServletRequest`相关的`HttpSession`对象中, 获取对应属性
* 获取请求中的地区信息
    - `RequestContext.getLocale()`: 获取LocaleResolver能解析到的地区信息
    - `RequestContext.getTimeZone()`: 获取时区.
        - 时区信息是被Spring的`ConversionService`下注册的日期/时间转换器`Converter`和格式化对象`Formatter`所使用的
* 地区信息相关的拦截器  
    - `LocaleChangeInterceptor`: 地区变更拦截器. 检测请求参数中的地区变化

```xml
CookieLocaleResolver配置
-----------------------
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">
    <property name="cookieName" value="clientlanguage"/>
    <!-- 单位为秒。若设置为-1，则cookie不会被持久化（客户端关闭浏览器后即被删除） -->
    <property name="cookieMaxAge" value="100000">
</bean>
```


## 主题(theme)

* theme用于为整站应用皮肤或主题, theme是一些列的静态资源的集合, 主要是样式表和图片
* 自定义`ThemeResource`, 或配置`ResourceBundleThemeSource`的基本名前缀(base name prefix), 可以在`applicationContext.xml`中注册一个名字为`themeSource`的bean, 会自动检测并使用它
    - 默认情况下基本名前缀是空值, 即从根classpath路径加载, 如`/WEB-INF/classes`
* `org.springframework.ui.context.support.ResourceBundleThemeSource`是处理主题的默认实现, 它会从`classpath`的根路径下去加载配置文件
    - 主题处理逻辑: 应用主题需要实现`org.springframework.ui.context.ThemeSource`接口. `WebApplicationContext`接口继承了`ThemeSource`接口
    - `ResourceBundleThemeSource`方式配置主题:
        - 在配置文件`xxx.properties`中定义属性的键值对
        - 在JSP中, 可以使用`<spring:theme code="background" />`标签来定义
* `themeResolver`是默认的主题解析器. `DispatcherServlet`会自动查找这个名称的bean来确定`ThemeResolver`的实现
    - 一些`ThemeResolver`接口的实现
        - `FixedThemeResolver`: 选择一个固定的主题, 这时通过设置`defaultThemeName`属性实现的
        - `SessionThemeResolver`: 请求相关的主题保存在用户的HTTP session. 对于每个session来说, 只需要被设置一次, 但不能在session之间保存
        - `CookieThemeResolver`: 选中的主题被保存在客户端的cookie中
* `ThemeChangeInterceptor`: 主题变更拦截器


### 示例

* 定义主题属性

```html
<!-- 配置文件 -->
styleSheet=/themes/cool/style.css
background=/themes/cool/img/coolBg.jpg

<!-- JSP中使用spring:theme标签 -->
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<html>
    <head>
        <link rel="stylesheet" href="<spring:theme code=''styleSheet''/>" type="text/css"/>
    </head>
    <body style="background=<spring:theme code=''background''/>">
        ...
    </body>
</html>
```

## 文件上传

* Spring内置支持`multipart`上传
    - 默认是关闭的. 如需开启, 需要在`applicationContext.xml`中注册一个`MultipartResolver`
* 配置第三方文件上传库`commons-fileupload`
    - 添加依赖
    - 注册解析器
    - 使用`MultipartHttpServletRequest`对象获取文件
* Servlet 3.0开启文件上传
    - 在`web.xml`中的`DispatcherServlet`配置中添加`multipart-config`标签, 然后注册`StandardServletMultipartResolver`解析器到`applicationContext.xml`

```xml
applicationContext.xml中注册解析器
--------------------------------
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 支持的其中一个属性，支持的最大文件大小，以字节为单位 -->
    <property name="maxUploadSize" value="100000"/>
</bean>


<!-- Servlet 3.0 注册的解析器 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver">
</bean>
```

### 使用内置支持处理表单上传

* 接收浏览器发送的`enctype="multipart/form-data"`表单上传文件

```html
<html>
    <head>
        <title>Upload a file please</title>
    </head>
    <body>
        <h1>Please upload a file</h1>
        <form method="post" action="/form" enctype="multipart/form-data">
            <input type="text" name="name"/>
            <input type="file" name="file"/>
            <input type="submit"/>
        </form>
    </body>
</html>
```

* 创建控制器
    - 控制器方法参数可以使用`MultipartHttpServletRequest`或`MultipartFile`来获取文件
    - Servlet 3.0中, 参数使用`javax.servlet.http.Part`类型

```java
@Controller
public class FileUploadController {
    @RequestMapping(path = "/form", method = RequestMethod.POST)
    public String handleFormUpload(@RequestParam("name") String name, @RequestParam("file") MultipartFile file) {
        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

### 使用内置支持处理客户端文件上传

* 使用`@RequestPart`注解来获取`meta-data`

```java
@RequestMapping(path = "/someUrl", method = RequestMethod.POST)
public String onSubmit(@RequestPart("meta-data") MetaData metadata, @RequestPart("file-data") MultipartFile file) {
    // ...
}
```

### 使用Commons FileUpload处理文件上传

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


## 异常处理

* 异常处理方式: dao, service, controller的所有异常都往上抛, 最终由全局异常处理器处理

### 处理局部异常

* 在控制器中定义一个处理异常的方法, 并使用`@ExceptionHandler(异常类.class)`注解该方法, 如果控制器方法抛出异常, 则会进入该异常处理方法中进行处理

```java
@Controller
public class SimpleController {

    // @RequestMapping methods omitted ...

    @ExceptionHandler(IOException.class)
    public ResponseEntity<String> handleIOException(IOException ex) {
        // prepare responseEntity
        return responseEntity;
    }
}
```

### 处理全局异常

* 2种方式
    - 使用现有异常处理器:
        - `SimpleMappingExceptionResolver`: 将异常映射到一个表示错误的视图
        - `DefaultHandlerExceptionResolver`: 可以将异常转换为状态码
            - `BindException`: 400, 请求无效
            - `ConversionNotSupportedException`: 500, 服务器内部错误
            - `HttpMediaTypeNotAcceptableException`: 406, 不接受
            - `HttpMediaTypeNotSupportedException`: 415, 不支持的媒体类型
            - `HttpMessageNotReadableException`: 400, 请求无效
            - `HttpMessageNotWritableException`: 500, 服务器内部错误
            - `HttpRequestMethodNotSupportedException`: 405, 不支持的方法
            - `MethodArgumentNotValidException`: 400, 无效请求
            - `MissingServletRequestParameterException`: 400, 无效请求
            - `MissingServletRequestPartException`: 400, 无效请求
            - `NoHandlerFoundException`: 404, 请求资源未找到
            - `NoSuchRequestHandlingMethodException`: 404, 请求资源未找到
            - `TypeMismatchException`: 400, 请求无效
            - `MissingPathVariableException`: 500, 服务器内部错误
            - `NoHandlerFoundException`: 404, 请求未找到
    - 使用自定义全局异常处理器:
        - 定义类实现`HandlerExceptionResolver`接口, 重写`public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)`方法


### 自定义异常

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


### 自定义错误页面

* 在`web.xml`中定义一个错误页面`<error-page>`
    - Servlet 3.0以前, 错误元素必须显示指定映射到一个具体的错误码或异常类型
    - Servlet 3.0起, 不需要再映射

```xml
web.xml
-------
<error-page>
    <location>/error</location>
</error-page>
```

* 通过控制器定义错误页面

```java
@Controller
public class ErrorController {

    @RequestMapping(path = "/error", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public Map<String, Object> handle(HttpServletRequest request) {

        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));

        return map;
    }
}
```

* 使用JSP

```html
<%@ page contentType="application/json" pageEncoding="UTF-8"%>
{
    status:<%=request.getAttribute("javax.servlet.error.status_code") %>,
    reason:<%=request.getAttribute("javax.servlet.error.message") %>
}
```
