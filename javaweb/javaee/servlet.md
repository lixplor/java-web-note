# Servlet

## Servlet规范

* Servlet 2.4: 支持Tomcat 5
* Servlet 2.5: 支持JDK 5, Tomcat 6
* Servlet 3.0: 支持JDK 6, Tomcat 7, 支持注解替代web.xml
* Servlet 3.1

## Servlet体系结构

* `Servlet接口`
    - `GenericServlet抽象类`
        - `HttpServlet抽象类`
            - 自定义Servlet实现类
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
* Servlet生命周期    
Servlet是单实例, 多线程的. 每个请求创建一个线程, 调用`service()`方法执行自己的业务逻辑
    - `init(ServletConfig conf)`
        - 初始化
        - 默认第一次访问时调用
    - `service(ServletRequest req, ServeltResponse res)`
        - 处理请求
        - 每次请求时调用
    - `destroy()`
        - 销毁Servlet实例
        - Servlet实例销毁时调用, 如Servlet被移除, 服务器关闭等
* `ServletConfig`
    - Servlet配置对象
    - 获取Servlet名称, 初始化参数

使用步骤:
* 编写一个类
    - 继承`HttpServlet`
    - 重写`doXxx(HttpServletRequest req, HttpServletResponse res)`等Http动作方法
* 编写配置文件
    - 在`WEB-INF`中创建`web.xml`
    - 注册Servlet
    - 映射url
* 访问

```java
public class Hello extends HttpServlet {

    @Override
    protected void doXxx(HttpServletRequest req, HttpServletResponse res) {

    }
}
```

```xml
<!-- 注册servelt -->
<servlet>
    <servlet-name>ServletName</servlet-name>
    <servlet-class>com.package.ClassName</servlet-class>
</servlet>

<!-- 绑定路径 -->
<servlet-mapping>
    <servlet-name>ServletName</servlet-name>
    <url-patter>/xxx</url-pattern>
</servlet-mapping>
```

### 获取参数

* `HttpServletRequest`对象
    - `getParameter(key)`: 获取指定key的value
* `HttpServletResponse`对象
    - `setContentType(String)`
    - `getWriter()`


## url-pattern配置

* 配置方式:
    - 完全匹配
        - 必须以`/`开始
        - 如`/login`, `/user/signup`
    - 目录匹配
        - 必须以`/`开始, 且以`*`结束
        - 如`/book/*`
    - 后缀名匹配
        - 以`*`开始, 以后缀名字符结束
        - 如`*.jsp`, `*.do`, `*.action`
* 优先级
    - `完全匹配 > 目录匹配 > 后缀名匹配`

## load-on-startup标签

作用: 修改servlet的初始化时机
取值: 正整数. 值越大, 优先级越低


## HttpServletRequest

* 操作请求行
    - `String getMethod()`: 获取请求方法
    - `String getRemoteAddr()`: 获取请求ip
    - `String getContextPath()`: 获取项目名称
    - `String getRequestURI()`: 获取从项目名到参数之间的内容, 如`/user/id`
    - `String getRequestURL()`: 获取带协议的完整请求路径, 如`http://localhost/user/id`
    - `String getQueryString()`: 获取get请求的所有参数的字符串, 如`username=123&pwd=123`
    - `String getProtocol()`: 获取协议和版本
* 操作请求头
    - `String getHeader(String key)`: 获取指定头
    - `Enumeration getHeaders(String name)`: 获取多个指定头
    - `Enuermation getHeaderNames()`: 获取所有请求头的名称
    - `int getIntHeader(String key)`: 获取整形请求头
    - `long getDateHeader(String key)`: 获取时间请求头
* 操作请求参数
    - `String getParameter(String key)`: 获取指定参数
    - `String[] getParameterValues(String key)`: 获取同一个key的多个参数
    - `Map<String, String[] getParameterMap()`: 获取所有的参数key和value
    - `Enumeration getParameterNames()`: 获取所有参数key
* 处理参数编码
    - get请求: 参数在地址栏, 使用utf-8进行url编码
    - post请求: 参数放在请求体中
    - 通用方法: `new String(参数.getBytes("iso-8859-1"), "utf-8")`
    - 针对于post请求: 将请求流编码设置为utf-8, `request.setCharacterEncoding("utf-8")`

### 重要请求头

* `user-agent`
* `referer`

## HttpServletResponse

* 响应码
    - `setStatus(int code)`: 针对1xx, 2xx, 3xx
    - `setError(int code)`: 针对4xx, 5xx
* 响应头
    - `setHeader(String key, String value)`: 设置字符串类型值响应头
    - `setIntHeader(String key, int value)`: 设置整形数字类型值的响应头
    - `setDateHeader(String key, long value)`: 设置时间类型值的响应头
    - `addHeader(String key, String value)`: 增加
    - `addIntHeader(String key, int value)`
    - `addDateHeader(String key, long value)`
    - `sendRedirect(Strin location)`: 重定向到指定url
    - `setContentType(String type)`
    - `Writer getWriter()`: 字符输出流
        - `print(String text)`
        - `println(String text)`
    - `ServletOutputStream getOutputStream()`: 字节输出流

> 注意:
> getWriter和getOutputStream两个流互斥, 不能同时使用
> 响应完毕后, 服务器会自动关闭流

## 响应头

* `location`: 用于302重定向的新url
* `refresh`: 定时刷新
* `Content-Type`: 设置文件mime类型, 设置响应流编码, 告诉浏览器用什么编码打开
* `Content-disposition`: 用于文件下载



## 请求的重定向和转发的区别

* `重定向`
    - `response.setRedirect(url)`
    - 是response的方法
    - 可以请求站外资源
    - 浏览器发送的, 将请求重定向到其他位置
    - 状态码`302`
    - 多次请求
    - 地址栏url改变

* `转发`
    - `request.getRequestDispatcher("内部路径").forward(request, response)`
    - 是request的方法
    - 不能请求站外资源
    - 服务器内部转发
    - 状态码不变
    - 一次请求
    - 地址栏url不变

## 域对象

* 都内置了Map集合来存储数据
* 都提供了`setAttribute()`和`getAttribute()`方法
* 都有自身特有的生命周期和作用域的对象
* 域对象使用原则: 能用作用域小的域对象, 就不用更大的域对象


| 域对象       | 对应对象              | 作用域               | 生命周期开始     | 生命周期结束             | 备注 |
|-------------|----------------------|---------------------|-----------------|-------------------------| - |
| Application | `ServletContext`     | Web应用运行期间都有效 | Web应用加载时创建 | Web应用被移除或服务器关闭  | - |
| Session     | `HttpSession`        | 在一次会话中有效      | 创建session      | session过期或被声明为失效 | - |
| Request     | `HttpServletRequest` | 在当前请求中有效      | 接收到用户请求    | 处理完响应               | - |
| Page        | `PageContext`        | 在当前JSP内有效       | JSP页面开始执行  | JSP页面执行完毕           | 仅对于JSP页面有效 |



## 功能

### 登录流程

* 数据库和表
* web页面
* Servlet

流程:
* `login.html`
    - 提交`username`和`pwd`
* `LoginServlet`
    - 接收到请求信息
    - 调用`UserService.login(username, pwd)`
* `UserService`
    - 在`login(username, pwd)`方法中调用`UserDao.getUser(username, pwd)`
* `UserDao`
    - 查询数据库
    - 返回`User`对象, 一直返回到`LoginServlet`
* `LoginServlet`
    - 判断返回的`User`对象是否为空
        - 空: 用户不存在
        - 非空: 允许登录

### 自动跳转

通过设置响应头的`refresh`字段实现
* `refresh: 延时秒数;url=要跳转的路径`

```java
response.setHeader("refresh", "3;url=/login.html");
```

### 统计用户登录成功次数

数据库增加一个登录成功次数字段, 每次登录成功将该字段值取出, 加1, 然后保存
`ServletContext`

### 文件下载

* 获取下载请求的文件名称
    - `request.getParameter(key, value)`
    - 注意如果有中文的话, 需要处理编码
* 设置文件mime类型
    - `String mime = this.getServletContext().getMimeType(filename)`
    - `response.setContentType(mime)`
* 设置下载头信息: `content-disposition`
    - `response.setHeader("Content-Disposition", "attachment;filename=" + filename)`
    - 注意文件名, 如果包含中文, 则需要utf-8编码. 对于FireFox, 需要使用Base64编码
* 提供下载文件的对拷流
    - `InputStream is = context.getResourceAsStream()`
    - `ServletOutputStream os = response.getOutputStream()`
* 问题: 并发请求下载同一个文件怎么办?

### Response生成验证码

图片标签src请求验证码

```html
<img alt="验证码" src="/code" title="看不清?点击更换" onclick="changeImg(this)"/>

<script type="text/javascript">
    function changeImg(obj) {
        // 通过请求的不同, 实现刷新
        obj.src = "/code?t=" + new Date().getTime();
    }
</script>
```

```java
public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	// 使用java图形界面技术绘制一张图片
	int charNum = 4;
	int width = 30 * 4;
	int height = 30;

	// 1. 创建一张内存图片
	BufferedImage bufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

	// 2.获得绘图对象
	Graphics graphics = bufferedImage.getGraphics();

	// 3、绘制背景颜色
	graphics.setColor(Color.YELLOW);
	graphics.fillRect(0, 0, width, height);

	// 4、绘制图片边框
	graphics.setColor(Color.BLUE);
	graphics.drawRect(0, 0, width - 1, height - 1);

	// 5、输出验证码内容
	graphics.setColor(Color.RED);
	graphics.setFont(new Font("宋体", Font.BOLD, 20));

	// 随机输出4个字符
	Graphics2D graphics2d = (Graphics2D) graphics;
	String s = "ABCDEFGHGKLMNPQRSTUVWXYZ23456789";
	Random random = new Random();
	// session中要用到
	String msg = "";
	int x = 5;
	for (int i = 0; i < 4; i++) {
		int index = random.nextInt(32);
		String content = String.valueOf(s.charAt(index));
		msg += content;
		double theta = random.nextInt(45) * Math.PI / 180;
		// 让字体扭曲
		graphics2d.rotate(theta, x, 18);
		graphics2d.drawString(content, x, 18);
		graphics2d.rotate(-theta, x, 18);
		x += 30;
	}

	// 6、绘制干扰线
	graphics.setColor(Color.GRAY);
	for (int i = 0; i < 5; i++) {
		int x1 = random.nextInt(width);
		int x2 = random.nextInt(width);

		int y1 = random.nextInt(height);
		int y2 = random.nextInt(height);
		graphics.drawLine(x1, y1, x2, y2);
	}

	// 释放资源
	graphics.dispose();

	// 图片输出 ImageIO
	ImageIO.write(bufferedImage, "jpg", response.getOutputStream());
}
```

### referer防盗链判断

referer为null, 地址栏输入的
