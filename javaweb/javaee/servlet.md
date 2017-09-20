# Servlet

## Servlet规范

* Servlet 2.4: 支持Tomcat 5
* Servlet 2.5: 支持JDK 5, Tomcat 6
* Servlet 3.0: 支持JDK 6, Tomcat 7, 支持注解替代web.xml
* Servlet 3.1

## Servlet体系结构

* Servlet体系结构

```
Servlet接口
    |_ GenericServlet抽象类
        |_ HttpServlet抽象类
            |_ 自定义Servlet实现类
```

* Servlet接口常用方法:
    - `void init(ServletConfig config)`: 初始化配置
    - `void service(ServletRequest req, ServletResponse res)`: 处理请求
    - `void destroy()`: 销毁
    - `ServletConfig getServletConfig()`: 获取当前Servlet配置对象
* GenericServlet抽象类方法
    - 除了service(), 其他都实现了
    - init()空实现
* HttpServlet方法:
    - 实现了`service()`方法, 转换为`doXxx()`方法
* Servlet生命周期: Servlet是单实例, 多线程的. 每个请求创建一个线程, 调用`service()`方法执行自己的业务逻辑
    - `init(ServletConfig conf)`
        - 初始化
        - 默认第一次访问时调用
    - `service(ServletRequest req, ServeltResponse res)`
        - 处理请求
        - 每次请求时调用
    - `destroy()`
        - 销毁Servlet实例
        - Servlet实例销毁时调用, 如Servlet被移除, 服务器关闭等
* 使用步骤:
    - 编写一个类
        - 继承`HttpServlet`
        - 重写`doXxx(HttpServletRequest req, HttpServletResponse res)`等Http动作方法
    - 编写配置文件
        - 在`WEB-INF`中创建`web.xml`
        - 注册Servlet: `<servlet>`
        - 映射url: `<servlet-mapping>`
    - 访问URL: `http://IP:端口号/项目名/Servlet映射的URL`

```java
public class Hello extends HttpServlet {

    @Override
    protected void doXxx(HttpServletRequest req, HttpServletResponse res) {

    }
}
```

```xml
<!-- web.xml -->
<!-- 注册Servlet -->
<servlet>
    <servlet-name>Servlet自定义名称</servlet-name>
    <servlet-class>Servlet实现类的全类名</servlet-class>
</servlet>

<!-- 映射Servlet到URL -->
<servlet-mapping>
    <servlet-name>Servlet自定义名称</servlet-name>
    <url-pattern>/要映射的URL</url-pattern>
</servlet-mapping>
```

## url-pattern配置

* 配置方式:
    - 完全路径匹配
        - 必须以`/`开始
        - 如`/login`, `/user/signup`
    - 目录匹配
        - 必须以`/`开始, 且以`*`结束
        - 如`/book/*`
    - 后缀名匹配
        - 以`*`开始, 以后缀名字符结束. 注意不能以`/`开头
        - 如`*.jsp`, `*.do`, `*.action`
* 优先级
    - `完全路径匹配 > 目录匹配 > 后缀名匹配`

## URL路径分类

* 相对路径: 以`.`或`..`开头
    - 相对于当前路径的路径
* 绝对路径:
    - 以`http://`开头
    - 以`/`开头
        - 客户端路径: 路径中带有项目名称
        - 服务端路径: 路径中不用带有项目名称. 一般在转发时使用

## load-on-startup标签

作用: 修改servlet的初始化时机. Servlet默认在第一次被访问时才初始化, 会影响效率. 可以设置此标签, 让Servlet在服务器启动时就初始化
取值: 正整数. 值越大, 优先级越低


## HttpServletRequest

* 获取请求行信息
    - `String getMethod()`: 获取请求方法
    - `String getProtocol()`: 获取协议和版本
    - `String getContextPath()`: 获取项目名称
    - `String getRequestURI()`: 获取从项目名到参数之间的内容, 如`/user/id`
    - `StringBuffer getRequestURL()`: 获取带协议的完整请求路径, 如`http://localhost/user/id`
    - `String getQueryString()`: 获取get请求的所有参数的字符串, 如`username=123&pwd=123`
* 获取请求头信息
    - `String getHeader(String key)`: 获取指定头
    - `Enumeration getHeaders(String name)`: 获取多个指定头
    - `Enuermation getHeaderNames()`: 获取所有请求头的名称
    - `int getIntHeader(String key)`: 获取整形请求头
    - `long getDateHeader(String key)`: 获取时间请求头
* 获取请求参数
    - `String getParameter(String key)`: 获取指定参数
    - `String[] getParameterValues(String key)`: 获取同一个key的多个参数
    - `Map<String, String[] getParameterMap()`: 获取所有的参数key和value
    - `Enumeration getParameterNames()`: 获取所有参数key
* 获取客户端信息
    - `String getRemoteAddr()`: 获取请求发送方的IP地址
    - `String getRemoteHost()`: 获取请求发送方的主机名
    - `String getRemotePort()`: 获取请求发送方的端口号
* 作为域对象存取数据
    - `void setAttribute(String name, Object value)`: 保存数据键值对
    - `Object getAttribute(String name)`: 根据键获取值
    - `void removeAttribute(String name)`: 根据键删除键值对
* 获取Cookie
    - `Cookie[] getCookies()`: 获取所有Cookie
* 获取Session
    - `HttpSession getSession()`: 获取当前请求中的Session

## HttpServletResponse

* 响应码
    - `setStatus(int code)`: 针对1xx, 2xx, 3xx
    - `setError(int code)`: 针对4xx, 5xx
* 响应头
    - `setHeader(String key, String value)`: 设置字符串类型值响应头, 覆盖
    - `setIntHeader(String key, int value)`: 设置整型数字类型值的响应头, 覆盖
    - `setDateHeader(String key, long value)`: 设置时间类型值的响应头, 覆盖
    - `addHeader(String key, String value)`: 增加字符串类型值响应头, 追加
    - `addIntHeader(String key, int value)`: 增加整型数字类型值的响应头, 追加
    - `addDateHeader(String key, long value)`: 增加时间类型值的响应头, 追加
    - `sendRedirect(String location)`: 重定向到指定url
    - `setContentType(String type)`: 设置内容类型
* 写出响应
    - `PrintWriter getWriter()`: 字符输出流
        - `print(String text)`: 输出不换行
        - `println(String text)`: 输出并换行
    - `ServletOutputStream getOutputStream()`: 字节输出流
    - 注意:
        - getWriter和getOutputStream两个流互斥, 不能同时使用, 否则会报错
        - 响应完毕后, 服务器会自动关闭流
* 写出Cookie
    - `void addCookie(Cookie cookie)`: 向响应中添加一个Cookie, 写入到浏览器


## 请求的重定向和转发的区别

* 重定向
    - `response.setRedirect(url)`
    - 是response的方法
    - 可以请求站外资源
    - 告诉浏览器将请求重定向到其他位置, 让浏览器向新地址重新发送请求
    - 状态码`302`
    - 多次请求, 会影响Request域对象
    - 地址栏url改变
* 转发
    - `request.getRequestDispatcher("服务端路径").forward(request, response)`
    - 是request的方法
    - 不能请求站外资源
    - 在服务端内部转发该请求信息, 与浏览器无关
    - 状态码不变
    - 一次请求, 不会影响Request域对象
    - 地址栏url不变


## 缓存会话

* Cookie和Session的区别
    - Cookie保存在客户端, Session保存在服务端
    - Cookie保存的数据数量和大小受浏览器限制, Session保存的数据不受浏览器限制

### Cookie

* 作用: 将数据保存在客户端或浏览器
* 默认情况下, 浏览器关闭Cookie就会销毁, 但可以设置Cookie的过期时间, 等超时后才删除
* 浏览器能够保存的Cookie的大小和个数有限制
* `Cookie`类
    - 构造方法:
        - `Cookie Cookie(String name, String value)`: 使用键值对创建一个Cookie
    - 常用方法:
        - `String getName()`: 获取Cookie的名称
        - `String getValue()`: 获取Cookie的值
        - `void setDomain(String pattern)`: 指定Cookie的有效域名.
        - `void setPath(String uri)`: 指定访问哪个路径时才能获取到Cookie
        - `void setMaxAge(int expire)`: 设置Cookie的有效时间, 单位为秒. 如果设置为0, 则相当于让浏览器删除当前的Cookie(有效路径必须一致), 刷新页面即可
* 获取和设置Cookie的相关方法:
    - 服务端向浏览器写入cookie: 使用Response对象
        - `httpServletResponse.addCookie(Cookie cookie)`
    - 服务端读取浏览器的cookie: 使用Request对象
        - `httpServletRequest.getCookies()`
* 常见应用场景
    - 记住用户名
    - 浏览记录


* 查找指定键的Cookie工具类

```java
public class CookieUtils {

    private CookieUtils() {}

    public static Cookie getCookie(Cookie[] cookies, String name) {
        if (cookies == null)
            return null;
        for (Cookie cookie : cookies) {
            if (name.equals(cookie.getName()))
                return cookie;
        }
        return null;
    }
}
```


### Session

* Session是基于Cookie的
* 作用: 将数据保存在服务端, 客户端或浏览器只保存一个与数据对应的sessionId
* 一般保存用户的私有信息
* 常见应用场景
    - 保持用户登录状态
    - 用户购物车
* `HttpSession`类
    - 获取Session
        - `httpServletRequest.getSession()`: 获取当前请求中的session
    - 获取Session信息
        - `String getId()`: 获取Session ID
    - 作为域对象存取数据
        - `void setAttribute(String name, Object value)`: 保存数据键值对
        - `Object getAttribute(String name)`: 根据键获取值
        - `void removeAttribute(String name)`: 根据键删除键值对
    - 销毁
        - `void invalidate()`: 将session声明为无效


## 域对象

* 都内置了Map集合来存储数据
* 都提供了`setAttribute()`和`getAttribute()`方法
* 都有自身特有的生命周期和作用域的对象
* 域对象使用原则: 能用作用域小的域对象, 就不用更大的域对象


* JSP中的4个域对象

| 域名称       | 对应对象      | 作用域             | 生命周期开始    | 生命周期结束           | 备注 |
|-------------|--------------|-------------------|---------------|----------------------| - |
|Application域|`ServletContext`| Web应用运行期间都有效| Web应用加载时创建|Web应用被移除或服务器关闭| - |
|Session域|`HttpSession`| 在一次会话中有效|第一次调用getSession()方法(访问JSP会调用因为有内置对象)|session过期(默认Tomcat设置30分钟); Session被声明为失效(调用invalidate()); 服务器非正常关闭(停电, 正常关闭Session会序列化到硬盘)|关闭浏览器默认会销毁Cookie, 可能导致保存SessionID的Cookie没有, 从而浏览器不会传递SessionID, 从而找不到Session中的信息. 注意此时Session并没有销毁, 只是找不到了|
|Request域|`HttpServletRequest`| 在当前请求中有效 | 接收到用户请求 | 处理完响应 | - |
|Page域| `PageContext` | 在当前JSP内有效 | JSP页面开始执行 | JSP页面执行完毕 | 仅对于JSP页面有效 |

* `ServletConfig`: 不是域对象. 是Servlet的配置对象
    - 作用:
        - 用于获取Servlet名称, 初始化参数
    - 常用方法:
        - `String getInitParameter(String name)`: 通过当前Servlet的初始化参数名获取参数值(web.xml中`servlet`中的`init-param`的键值对), 如果没有则返回null
        - `Enumeration getInitParameterNames()`: 获取初始化参数名的枚举
        - `ServletContext getServletContext()`: 获取ServletContext对象
        - `String getServletName()`: 获取当前Servlet的名称(在web.xml中配置的servlet-name)
* `ServletContext`: Application域对象. 一个web应用只有一个该对象, 无论有多少个Servlet
    - 作用:
        - 获取全局初始化参数(context-param)
        - 获取文件的MIME类型
        - 作为域对象存取数据
        - 读取web项目下的文件
    - 获取该对象的方法:
        - `GenericServlet`的`ServletContext getServletContext()`方法
    - 常用方法:
        - 获取全局初始化参数
            - `String getInitParameter(String name)`: 通过web应用的全局的初始化参数名获取参数值(web.xml中`web-app`中的`context-param`标签中的键值对)
            - `Enumeration getInitParameterNames()`: 获取全局初始化参数名的枚举
        - 获取文件MIME类型
            - `String getMimeType(String file)`: 获取文件的MIME类型(无论文件是否存在, 只通过文件后缀名判断)
        - 作为域对象存取数据
            - `void setAttribute(String name, Object value)`: 保存数据键值对
            - `Object getAttribute(String name)`: 根据键获取值
            - `void removeAttribute(String name)`: 根据键删除键值对
        - 读取项目中的文件
            - `InputStream getResourceAsStream(String path)`: 获取指定路径文件的输入流
            - `String getRealPath(String path)`: 获取一个路径在磁盘上的绝对路径
* `ServletHttpRequest`: Request域对象
    - 作为域对象存取数据
        - `void setAttribute(String name, Object value)`: 保存数据键值对
        - `Object getAttribute(String name)`: 根据键获取值
        - `void removeAttribute(String name)`: 根据键删除键值对
* `HttpSession`类
    - 获取Session
        - `httpServletRequest.getSession()`: 获取当前请求中的session
    - 获取Session信息
        - `String getId()`: 获取Session ID
    - 作为域对象存取数据
        - `void setAttribute(String name, Object value)`: 保存数据键值对
        - `Object getAttribute(String name)`: 根据键获取值
        - `void removeAttribute(String name)`: 根据键删除键值对
* `PageContext`:
    - 获取JSP中其他8个内置对象
        - `JspWriter getOut()`: 获取out对象
        - `Exception getException()`: 获取exception对象
        - `Object getPage()`: 获取page对象
        - `ServletRequest getRequest()`: 获取request对象
        - `ServletResponse getResponse()`: 获取response对象
        - `ServletConfig getServletConfig()`: 获取servletConfig对象
        - `ServletContext getServletContext()`: 获取servletContext对象
        - `HttpSession getSession()`: 获取session对象
    - 向JSP的4个域对象中存取数据
        - `void setAttribute(String name, Object value, int scope)`: 保存数据键值对, 同时指定保存到哪个域中, 值为PageContext类中的静态常量: `APPLICATION_SCOPE`, `SESSION_SCOPE`, `REQUEST_SCOPE`, `PAGE_SCOPE`
        - `Object getAttribute(String name, int scope)`: 根据键获取值, 同时指定从哪个域中获取, 值为PageContext类中的静态常量: `APPLICATION_SCOPE`, `SESSION_SCOPE`, `REQUEST_SCOPE`, `PAGE_SCOPE`
        - `void removeAttribute(String name, int scope)`: 根据键删除键值对, 同时指定从哪个域中删除, 值为PageContext类中的静态常量: `APPLICATION_SCOPE`, `SESSION_SCOPE`, `REQUEST_SCOPE`, `PAGE_SCOPE`
        - `Object findAttribute(String name)`: 从4个域中查找对应的键值对, 查找域的顺序为PAGE, REQUEST, SESSION, APPLICATION
    - 作为域对象存取数据
        - `void setAttribute(String name, Object value)`: 保存数据键值对
        - `Object getAttribute(String name)`: 根据键获取值
        - `void removeAttribute(String name)`: 根据键删除键值对


## 监听器

* 主要用于监听3个域对象(ServletContext, HttpSession, ServletRequest)
* 8个监听器
    - 监听域对象的创建和销毁. 定义类实现接口, 并在web.xml中注册
        - `ServletContextListener`: 用于加载框架配置文件, 定时任务
            - `void contextInitialized(ServletContextEvent sce)`: ServletContext启动时调用
            - `void contextDestroyed(ServletContextEvent sce)`: ServletContext销毁时调用
        - `HttpSessionListener`:
            - `void sessionCreated(HttpSessionEvent se)`: 当Session创建时调用
            - `void sessionDestroyed(HttpSessionEvent se)`: 当Session销毁时调用
        - `ServletRequestListener`:
            - `void requestInitialized(ServletRequestEvent sre)`: 当ServletRequest对象初始化时调用
            - `void requestDestroyed(ServletRequestEvent sre)`: 当ServletRequest对象销毁时调用
    - 监听域对象的属性添加, 删除, 修改. 定义类实现接口, 并在web.xml中注册
        - `ServletContextAttributeListener`
            - `void attributeAdded(ServletContextAttributeEvent scab)`: 当属性添加到ServletContext时调用
            - `void attributeRemoved(ServletContextAttributeEvent scab)`: 当属性从ServletContext删除时调用
            - `void attributeReplaced(ServletContextAttributeEvent scab)`: 当ServletContext中的属性被修改时调用
        - `HttpSessionAttributeListener`
            - `void attributeAdded(HttpSessionBindingEvent se)`: 当属性添加到session时调用
            - `void attributeRemoved(HttpSessionBindingEvent se)`: 当属性从session删除时调用
            - `void attributeReplaced(HttpSessionBindingEvent se)`: 当session中的属性被修改时调用
        - `ServletRequestAttributeListener`
            - `void attributeAdded(ServletRequestAttributeEvent srae)`: 当属性添加到request时调用
            - `void attributeRemoved(ServletRequestAttributeEvent srae)`: 当属性从request删除时调用
            - `void attributeReplaced(ServletRequestAttributeEvent srae)`: 当request中的属性被修改时调用
    - 监听HttpSession中的JavaBean的状态改变(绑定(将JavaBean存入session), 解除绑定(将JavaBean从session中删除), 钝化(序列化到磁盘), 活化(从磁盘反序列化)). 让JavaBean实现接口, 而不用配置到web.xml中
        - `HttpSessionBindingListener`
            - `void valueBound(HttpSessionBindingEvent event)`: 当对象绑定到session时调用
            - `void valueUnbound(HttpSessionBindingEvent event)`: 当对象从session解绑时调用
        - `HttpSessionActivationListener`
            - `void sessionDidActivate(HttpSessionEvent se)`: 当session活化时调用
            - `void sessionWillPassivate(HttpSessionEvent se)`: 当session钝化时调用

* 在web.xml中配置监听器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app ...>
    ...
    <listener>
        <listener-class>监听器全类名</listener-class>
    </listener>
</web-app>
```


## 过滤器

* 用于过滤客户端向服务器发送的请求和响应, 进行一些处理
* 过滤器的生命周期
    - 创建: 服务器启动时创建过滤器
    - 销毁: 服务器关闭时销毁过滤器
* `javax.servlet.Filter`接口
    - `void init(FilterConfig config)`: 当过利器初始化时调用
    - `void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)`: 每次请求时调用
    - `void destroy()`: 当过滤器销毁时调用
* `javax.servlet.FilterChain`接口
    - `void doFilter(ServletRequest request, ServletResponse response)`: 传递请求和响应到下一个过滤器, 如果后面没有过滤器, 则传递到请求的资源(Servlet或JSP)进行处理
    - 注意:
        - 过滤器链中的执行顺序:
            - 先执行带有`<url-pattern>`的过滤器
            - 再执行带有`<filter-name>`的过滤器
            - 如果以上条件相同, 则按照web.xml中`<filter-mapping>`的配置顺序执行
        - 过滤器链在过滤请求时经过的过滤器顺序和返回响应时经过的过滤器顺序相反
* `javax.servlet.FilterConfig`接口
    - `String getFilterName()`: 获取过滤器在web.xml中配置的名称
    - `String getInitParameter(String name)`: 根据键获取初始化参数值, 如果没有则返回null
    - `Enumeration getInitParameterNames()`: 获取初始化参数名的枚举
    - `ServletContext getServletContext()`: 获取ServletContext对象


* 在web.xml中配置过滤器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app ...>
    ...
    <filter>
        <filter-name>过滤器名称</filter-name>
        <filter-class>监听器全类名</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>过滤器名称</filter-name>
        <url-pattern>使用过滤器的URL路径</url-pattern>
    </filter-mapping>
</web-app>
```


## Servlet 3.0

* Servlet 3.0相对于2.5的区别:
    - 支持注解开发, 可以替代web.xml
    - 支持文件上传
    - 支持异步请求
* Tomcat 7及以上版本支持Servlet 3.0
* 使用注解代替web.xml
    - `@WebServlet()`: 注解当前类为一个Servlet
        - 注解的属性
            - `urlPatterns="URL路径"`: 匹配该Servlet的URL路径
            - `loadOnStartup=优先级数字`: 启动优先级
            - `initParams={@WebInitParam注解的参数元素}`: 初始化参数
                - `@WebInitParam(name="键", value="值")`: 参数键值对
            - `asyncSupported=布尔值`: 是否支持异步请求
    - `@WebListener`: 注解当前类为一个Listener
    - `@WebFilter()`: 注解当前类为一个Filter
        - 注解的属性
            - `urlPatterns="URL路径"`: 匹配该过滤器的URL路径
            - `asyncSupported=布尔值`: 是否支持异步请求
    - `@MultipartConfig`: 注解当前Servlet支持文件上传API
* Servlet 3.0中新增API
    - `HttpServletRequest`中新增
        - `Part getPart(String name)`: 获取文件上传部分的内容
        - `AsyncContext startAsync(ServletRequest req, ServletResponse resp)`: 获取异步请求上下文
    - `Part`
        - `String getName()`: 获取name属性值
        - `String getHeader(String name)`: 获取指定头的值
        - `long getSize()`: 获取上传的文件的大小
        - `InputStream getInputStream()`: 获取上传文件输入流
    - `AsyncContext`
        - `start(Runnable r)`: 启动异步任务
        - `ServletRequest getRequest()`: 获取请求对象
        - `ServletResponse getResponse()`: 获取响应对象


## 中文乱码处理方法

* Request获取的请求参数乱码
    - 针对GET请求:
        - 乱码原因: GET请求参数在URL中, Tomcat对URL编码默认使用ISO-8859-1, 所以中文乱码.
        - 解决方式:
            - 方式1: 将参数字符串重新转码, `new String(参数.getBytes("iso-8859-1"), "utf-8")`
            - 方式2: 使用URLEncoder和URLDecoder编解码
            - 方式3: 修改Tomcat配置文件`server.xml`将`Connector`的`URIEncoding`属性值设置为`UTF-8`
    - 针对POST请求:
        - 乱码原因: POST请求参数在请求体中, 请求体字符缓冲区默认是ISO-8859-1
        - 解决方式: 修改请求的字符缓冲区编码, `request.setCharacterEncoding("utf-8");`
* Response写出的内容乱码
    - 页面乱码:
        - 字节流输出中文乱码
            - 乱码原因: 代码写出的字符串的编码和浏览器解析页面的编码不一致. Java中字符串获取字节数组默认按照ANSI本地编码
            - 解决方案:
                - 将代码写出的字符串编码和页面Content-Type响应头中的编码统一:
                    - 设置代码中字符串的编码: `getOutputStream().write("中文".getBytes("utf-8"));`
                    - 设置要求浏览器使用的编码: `response.setContentType("text/html; charset=utf-8")`
        - 字符流输出中文乱码
            - 乱码原因: 代码写出的字符串的编码和浏览器解析页面的编码不一致. 字符流中有缓冲区, 默认是ISO-8859-1
            - 解决方案:
                - 将response对象中的字符缓冲区编码和页面Content-Type响应头中的编码统一:
                    - 设置代码中字符串的编码: `response.setCharacterEncoding("utf-8");`
                    - 设置要求浏览器使用的编码: `response.setContentType("text/html; charset=utf-8")`
    - 下载文件名乱码:
        - IE浏览器:
            - 乱码原因: IE浏览器对于文件名使用URL编码
            - 解决方式: 通过request对象获取`User-Agent`请求头, 判断是否包含`Firefox`字符串, 不是则使用`String URLEncoder.encode(String s, String charsetName)`对文件名使用utf-8字符集进行URL编码
        - Firefox浏览器:
            - 乱码原因: Firefox浏览器对于文件名使用Base64编码
            - 解决方式: 通过request对象获取`User-Agent`请求头, 判断是否包含`Firefox`字符串, 是则使用Base64对文件名进行编码
* 以上方式可以利用Filter统一对请求和响应进行处理
