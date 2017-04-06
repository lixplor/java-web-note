# JSP

* JSP, Java Server Pages, 是一种动态网页开发技术
* 使用JSP标签在HTML网页中插入Java代码
* JSP是一种Java Servlet, 主要用于实现Java web应用程序的用户界面部分. 除了解释阶段外, JSP网页几乎可以被当成一个普通的Servlet对待

## 运行环境

* JDK
* 支持JSP的Web服务器, 也叫做容器, 如Tomcat, Apache


## JSP结构

* Web服务器通过JSP引擎(容器)处理JSP页面
* 容器截获对JSP页面的请求

![图示](http://www.runoob.com/wp-content/uploads/2014/01/jsp-arch.jpg)


## JSP原理

Web服务器使用JSP创建网页的步骤:
* 浏览器发送HTTP请求给Web服务器
* Web服务器识别出这是JSP网页的请求, 将该请求传递给JSP引擎. 通过URL或.jsp文件来完成
* JSP引擎从磁盘中载入JSP文件, 然后将它们转换为Servlet. 这种转换只是简单地将所有模板文本改用`println()`语句, 并且将所有的JSP元素转化为Java代码
* JSP引擎将Servlet编译为class文件, 将原始请求传递给Servlet引擎
* Web服务器的某组件将会调用Servlet引擎, 然后载入并执行Servlet类. 执行过程中, Servlet产生HTML格式的输出并将其内嵌与HTTP response中上交给Web服务器
* Web服务器以静态HTML网页的形式将HTTP response返回到浏览器中

![流程图](http://www.runoob.com/wp-content/uploads/2014/01/jsp-processing.jpg)


## JSP生命周期

JSP的生命周期和Servlet非常相似:
* 编译: Servlet容器编译Servlet源文件, 生成Servlet类
    - 当浏览器请求JSP页面时, JSP引擎会首先去检查是否需要编译这个文件. 如果这个文件没有被编译过, 或者在上次编译后背更改过, 则编译这个JSP文件
    - 编译的3个步骤:
        - 解析JSP文件
        - 将JSP文件转为Servlet
        - 编译Servlet
* 初始化: 加载与JSP对应的Servlet类, 创建其实例, 并调用其初始化方法
    - 容器载入JSP文件后, 会先调用`jspInit()`方法.
    - 初始化只执行一次
* 执行: 调用JSP对应的Servlet实例的服务方法
    - 处理请求相关的交互行为, 直到被销毁
    - JSP网页初始化完毕后, 会调用`_jspService()`方法, 该方法需要一个`HttpServletRequest`对象和一个`HttpServletResponse`对象作为参数.
    - `_jspService()`方法在每个request中被调用一次并且负责产生与之相对应的response, 并且它还负责产生所有7个HTTP方法的响应, 如GET, POST等
* 销毁: 调用与JSP对应的Servlet实例的销毁方法, 然后销毁Servlet实例
    - `jspDestroy()`是销毁的方法

![JSP生命周期](http://www.runoob.com/wp-content/uploads/2014/01/jsp_life_cycle.jpg)


## JSP语法

* `<% ... %>`: 代码片段
* `<%! ... %>`: 声明
* `<%= ... %>`: 表达式
* `<%@ ... %>`: 指令, 用于设置JSP页面属性
    - `<%@ page ... %>`: 定义页面依赖属性
        - 属性:
            - `buffer`: 指定out对象使用缓冲区的大小
            - `autoFlush`: 控制out对象的缓存区
            - `contentType`: 指定当前JSP页面的MIME类型和字符编码
            - `errorPage`: 指定当JSP页面发生异常时需要转向的错误处理页面
            - `isErrorPage`: 指定当前页面是否可以作为另一个JSP页面的错误处理页面
            - `extends`: 指定Servlet从哪一个类继承
            - `import`: 导入要使用的Java类
            - `info`: 定义JSP页面的描述信息
            - `isThreadSafe`: 指定对JSP页面的访问是否为线程安全
            - `language`: 定义JSP页面所用的脚本语言, 默认是Java
            - `session`: 指定JSP页面是否使用session
            - `isELIgnored`: 指定是否执行EL表达式
            - `isScriptingEnabled`: 确定脚本元素能否被使用
    - `<%@ include ... %>`: 包含其他文件
        - `file`: 文件相对url地址
    - `<%@ taglib ... %>`: 引入标签库的定义
        - `uri`: 标签库的位置
        - `prefix`: 标签库的前缀, 如`<s:scope>`
* `<%-- ... --%>`: 注释, 在浏览器查看源代码时看不到

### 代码片段

```html
<% 代码片段 %>

<jsp:scriptlet>
    代码片段
</jsp:scriptlet>
```

### 声明

```html
<%! 声明; ... %>

<%! int i= 0; Circle a = new Circle(2.0); %>
```

### 表达式

```html
<%= 表达式 %>

<jsp:expression>
    表达式
</jsp:expression>

<%= new Date().toLocaleString() %>
```

### 指令

```html
<%@ 指令 属性="属性值" %>
```

### 中文编码

```html
<!-- 添加在文件头部 -->
<%@ page language="java" contentType="text/html; charset="UTF-8" pageEncoding="UTF-8" %>
```


### 控制流语句

* 条件判断
    - `if...else...`
    - `switch...case`
* 循环
    - `for`
    - `while`
    - `do...while`

#### if...else

```html
<% if(...) { %>
    ...
<% } else { %>
    ...
<% } %>

<% if(day == today) { %>
    <p>今天</p>
<% } else { %>
    <p>不是今天</p>
<% } %>
```

#### switch...case

```html
<%
switch(...) {
    case ...:
        ...
        break;
    default:
        ...
}
%>

<%
switch(day) {
    case 0:
        out.println("星期日");
        break;
    default:
        out.println("不知道");
}
%>
```

#### for

```html
<% for(...;...;...) { %>
    ...
<% } %>

<% for(int i = 0; i < 10; i++) { %>
    <li>这是<%= i %></li>
<% } %>
```

#### while

```html
<% while(...) { %>
    ...
<% } %>

<% while(a < 10) { %>
    <li>dsfsda</li>
<% } %>
```


## JSP动作标签

* 使用xml语法结构控制Servlet引擎
* `<jsp:行为名称 属性="属性值" />`
* 预定义行为标签:
    - `<jsp:include/>`: 用于在当前页面中包含静态或动态资源
    - `<jsp:useBean/>`: 寻找和初始化一个JavaBean组件
    - `<jsp:setProperty/>`: 设置JavaBean组件的值
    - `<jsp:getProperty/>`: 将JavaBean组件的值插入到output中
    - `<jsp:forward/>`: 从一个JSP文件向另一个文件传递一个包含用户请求的request对象
    - `<jsp:plugin/>`: 用于在生成的HTML页面中包含Applet和JavaBean对象
    - `<jsp:element/>`: 动态创建一个XML元素
    - `<jsp:attribute/>`: 定义动态创建的XML元素的属性
    - `<jsp:body/>`: 定义动态创建的XML元素的主体
    - `<jsp:text/>`: 封装模板数据


## JSP的9个隐含对象

* `request`: HttpServletRequest类的实例
* `response`: HttpServletResponse类的实例
* `out`: PrintWriter类的实例
* `session`: HttpSession类的实例
* `application`: ServletContext类的实例
* `config`: ServletConfig类的实例
* `pageContext`: PageContext类的实例, 提供对JSP页面所有对象以及命名空间的访问
* `page`: 类似于`this`
* `Exception`: Exception类的实例, 代表发生错误的JSP页面中对应的异常对象


## 常见问题

### 指令和动作标签的区别

指令在编译时生效, 动作标签在请求处理阶段起作用
