# Struts2

* Web层框架, 处理请求和响应
* 基于MVC设计模式的web应用框架, 本质上相当于一个Servlet, Struts2作为Controller
* 采用拦截器机制处理请求, 使得与Servlet API完全分开
* 前端控制器


## 环境搭建

### 下载

* 官网: [https://struts.apache.org/](https://struts.apache.org/)

### 导包

* 核心13个jar包

### 配置

* `web.xml`中配置`<filter>`, 使用struts2拦截所有请求:

```xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```


## 快速入门

* 创建`XxxAction`类, 处理请求
    - 该类方法必须符合如下要求:
        - 必须public
        - 必须有String类型返回值(实在没有返回的可以return null)
        - 方法名称任意, 但不能有参数列表
* 配置XxxAction
    - 创建`src/struts.xml`
    - 配置`<action>`标签

```jsp
<a href="${pageContext.request.contextPath}/login.action">登录</a>
```

```java
public class LoginAction {

    public String login(String username, String pwd) {
        // ...
        return null;
    }
}
```

```xml
src/struts.xml
--------------

<?xml version="1.0"... ?>
<!DOCTYPE struts ...>

<struts>
    <!-- 包结构 -->
    <package name="default" namespace="/" extends="struts-default">
        <!-- 配置action name:url的名称; class: Action的完整类名; method:Action类中对应的执行方法-->
        <action name="login" class="com.package.LoginAction" method="login">
            <<!-- 配置跳转的页面. 路径写法: 在struts2中无论转发还是重定向, 都不用谢项目名 -->
            <result name="ok">/demo/loginsuccess.jsp</result>
        </action>
    </package>
</struts>
```


## 执行流程

* 准备: 启动服务器, 核心过滤器创建, `init()`方法执行, 加载配置文件, 此时加载`struts.xml`配置
* 执行: 发送请求, `StrutsPrepareAndExecuteFilter`的`doFilter()`方法会执行, 经过一系列拦截器, 通过反射调用配置中指定的方法


## 配置文件的加载顺序

* 核心: `StrutsPrepareAndExecuteFilter`, 该过滤器有2个功能
    - Prepare: 预处理, 加载核心配置文件
    - Execute: 执行, 让部分拦截器执行
* StrutsPrepareAndExecuteFilter加载的配置文件(按加载顺序)
    - `init_DefaultProperties()`: 加载`org/apache/struts2/default.properties`
    - `init_TraditionalXmlConfigurations()`: 加载`struts-default.xml`, `struts-plugin.xml`, `struts.xml`
    - `init_LegacyStrutsProperties()`: 加载自定义的`struts.properties`
    - `init_CustomConfigurationProviders()`: 加载用户自定义配置提供者
    - `init_FilterInitParameters()`: 加载`web.xml`
    - 前三个配置文件是struts2默认的配置文件, 不用修改
    - 后三个配置文件可以允许自己修改常量, 后加载的配置会覆盖前面的配置
* 重要配置文件(按加载顺序)
    - `default-properties`: 在`org/apache/struts2/`目录下, struts2的常量值
    - `struts-default.xml`: struts2核心包下, 核心功能的配置(Bean, 拦截器, 结果类型等)
    - `struts.xml`: web应用的默认配置, 可以配置常量
    - `web.xml`: 配置前端控制器, 可以配置常量

* `struts.xml`
    - `<struts>`
        - `<package>`
            - `name`: 包名称, 包名不能相同
            - `namespace`: 命名空间. 决定action访问路径, `/`表示根目录
            - `extends`: 继承. 默认`struts-default`
            - `<action>`
                - `name`:
                - `class`: action类路径. 如果不配置, 默认会使用`ActionSupport`类
                - `method`: 执行的action类中的方法, 如果不配置, 则会执行action的默认方法`execute()`
                - `<result>`
                    - `name`: 结果的视图名称
                    - `type`: 跳转类型
                        - `chain`: 转发, action转发到action
                            - url写为`actionClass_method`格式
                        - `dispatcher`: 转发, action转发到jsp页面
                        - `redirect`: 重定向, action重定向到jsp页面
                        - `redirectAction`: 重定向, action重定向到action
                            - url写为`actionClass_method`格式
                        - `stream`: 下载流
                        - 还有其他
                    - 值: 跳转的url
            - `<global-result>`: 配置全局的result
                - `<result name="success">url</result>`


## 常量的配置

* 默认配置文件: struts项目中自带的`default.properties`文件, 不可修改
* 可修改的配置文件:
    - `struts.xml`
        - 示例: `<constant name="struts.action.extension" value="action,do,,"/>`
    - `struts.properties`
    - `web.xml`
        - 在`<filter>`标签内部, 增加以下两个标签进行配置:
            - `<param-name>name</param-name>`
            - `<param-value>value</param-value>`


## 使用`<include>`标签引入配置

分包管理配置文件:
* 在`struts.xml`中引入
    - 在`<struts>`标签内部使用`<include file="com/pakcage/xxx.xml">`


## Action类的三种编写方法

1. Action类作为一个POJO类(没有继承, 没有实现接口)
    - 在`struts.xml`配置action
2. Action类可以实现`Action`接口, 重写`execute()`方法
    - Action接口定义了5个常量, 对应5个逻辑视图跳转页面, 还定义了一个execute方法
        - `SUCCESS`: 成功
        - `INPUT`: 用于数据表单校验, 如果校验失败, 跳转INPUT视图
        - `LOGIN`: 登录
        - `ERROR`: 错误
        - `NONE`: 页面不跳转
    - `execute()`方法执行后返回以上5个常量之一
    - `struts.xml`中配置action
3. Action类可以继承`ActionSupport`类(使用较多)
    - `ActionSupport`类已经实现了`Action`及其他接口, 增加了很多功能, 便于开发
    - 重写`execute()`方法
    - `struts.xml`中配置action


## action的访问配置

* 传统访问方式
    - 每个请求配置一个`<action>`
* 动态方法访问方式
    - 开启配置常量: `struts.enable.DynamicMethodInvocation`设置为`true`
    - 配置`struts.xml`的action:
        - `<action class="xxx" name="user">`, 不用写`method`
    - 配置url, 使用`!`分隔类和方法名:  
        - `/user!save.action`
        - `/user!del.action`
* 通配符访问方式(使用较多)
    - action通配符配置, `{n}`表示占位符的位置, 通过其获取方法:
        - `<action class="xxxxx.ContactAction" name="contact_*" method="{1}"`
    - url配置:
        - `xxx/contact_save.action`: 正则匹配到`save`, 执行`save()`方法
        - `xxx/contact_del.action`: 正则匹配到`del`, 执行`del()`方法


## 获取参数的方式

* 使用Servlet API的方式
    - 完全解耦方式
        - `ActionContext ac = getContext()`
        - `Map<String, Object> params = ac.getParameters()`
        - `Map<String, Object> session = ac.getSession()`
        - `Map<String, Object> application = ac.getApplication()`
    - 使用`ServletActionContext`类的方式
        - `HttpServletRequest r = ServletActionContext.getRequest()`
        - `r.getSession()`
        - `r.getSession().getServletContext()`
* 使用Struts2的方式
    - 属性驱动方式:
        - 直接将action类当做Bean类方式(不好)
            - 将参数作为action类的属性, 提供setter即可
            - 注意中文参数的编码转换
        - 将参数属性封装到JavaBean中, 在action类中使用JavaBean
            - 页面: 使用`OGML表达式`
                - `<input>`标签的`name`属性格式: `javabean.javabean的属性`, 如`user.username`
            - action类: 定义JavaBean对象的属性, 提供该JavaBean的getter和setter
    - 模型驱动方式
        - 实现`ModelDriven`接口, 泛型是JavaBean, 重写方法`getModel()`方法
        - 手动实例化JavaBean对象
* 将数据封装到集合中
    - 场景: 比如同时添加多条相同参数的数据, 想要的数据是参数对象的集合
    - 默认采用属性驱动方式
        - list或set集合
            - 在action类中添加一个`List<JavaBean>`或`Set<JavaBean>`
            - 在页面`<input>`标签中的`name`属性, 使用OGNL表达式
                - `name=list[0].username`
                - `name=list[1].username`
        - map集合
            - action类添加`Map<String, JavaBean>`
            - 在页面`<input>`标签的`name`属性, 使用OGNL表达式
                - `name="map['keyone'].username"`
                - `name="map['keytwo'].username"`


## 拦截器

* 类似于nodejs. php的web开发框架中的`中间件`, 用于顺序处理请求
* 采用AOP实现
* 采用`责任链`模式
    - 责任链中, 多个对象由每个对象对齐下家对象的引用连接起来, 形成链
    - 责任链每一个节点, 都可以继续调用下一个节点, 也可以阻止流程继续执行
* 拦截器和过滤器的区别
    - 过滤器
        - 基于函数回调
        - 依赖servlet容器(tomcat, jetty服务器等)
        - 对所有的请求都起作用
    - 拦截器
        - 基于反射机制
        - 不依赖servlet容器
        - 只对Action请求起作用
* struts2中, 多个拦截器按照特定顺序, 组合成`拦截器栈`, 便可以顺序调用栈中的每一个拦截器
* 拦截器有多种抽象实现类, 实现了`Interceptor`接口, 通过继承可以实现不同功能
* 编写拦截器
    - 方式1:
        - 继承`AbstractInterceptor`, 重写`String intercept(ActionInvocation i)`方法
            - `ActionInvocation`
                - `String invoke()`: 执行下一个拦截器, 不写则后续拦截器就不会执行了
        - 配置拦截器
            - `<package>`标签内定义拦截器
                - `<interceptors>`: 用于包含拦截器
                    - `<interceptor name="拦截器名" class="拦截器完整类名">`: 定义拦截器
            - 在要使用拦截器的`<action>`标签内, 增加拦截器配置
                - `<interceptor-ref name="拦截器名"/>`
                - `<interceptor-ref name="defaultStack"/>`: 必须同时引入默认栈拦截器, 否则默认拦截器会不执行
    - 方式2: 直接定义拦截器栈
        - 在`<interceptors>`中
            - 定义拦截器栈: `<interceptor-stack name="自定义拦截器栈名">`
                - 引用自定义拦截器和默认拦截器: `<interceptor-ref name="拦截器名"/>`和`<interceptor-ref name="defaultStack"/>`
        - 在`<action>`标签中, 配置拦截器为自定义拦截器栈

```java
自定义拦截器
----------
public class LoginInterceptor extends AbstractInterceptor {

    @Override
    public String intercept(ActionInvocation ac) {
        User user = (User) ServletActionContext.getRequest().getSession().getAttribute("existUser");
        if(user == null) {
            // 没有登录, 不需要执行后续拦截器了
            return LOGIN;
        }
        // 继续执行后续拦截器
        return ac.invoke();
    }
}
```

```xml
方式1: 自定义拦截器
<interceptors>
    <!-- 自定义拦截器 -->
    <interceptor name="LoginInterceptor" class="xxx.LoginInterceptor"/>
</interceptors>

<!-- 配置到action中 -->
<action>
    <interceptor-ref name="LoginInterceptor"/>
    <!-- 必须同时引入默认栈, 否则默认拦截器不执行 -->
    <interceptor-ref name="defaultStack"/>
</action>
```

```xml
方式2: 自定义拦截器栈
------------------
<interceptors>
    <!-- 自定义拦截器 -->
    <interceptor name="LoginInterceptor" class="xxx.LoginInterceptor"/>

    <!-- 自定义拦截器栈 -->
    <interceptor-stack name="MyStack">
        <interceptor-ref name="LoginInterceptor"/>
        <interceptor-ref name="defaultStack"/>
    </interceptor-stack>
</interceptors>

<!-- 直接使用自定义拦截器栈 -->
<action>
    <interceptor-ref name="MyStack"/>
</action>
```


## OGNL表达式

* `OGNL`, Object-Graph Navigation Language, 对象图导航语言, 是一种表达式语言, 比EL强大
* Struts2使用OGNL作为默认的表达式语言
* 提供5大功能:
    - 调用对象的方法
    - 调用类的静态方法和属性
    - 访问`OGNL Context`和`ActionContext`
    - 支持赋值操作和链式调用
    - 操作集合对象

```java
// 创建上下文对象
OgnlContext context = new OgnlContext();
// 获取根对象
Object root = context.getRoot();

// 存储数据
context.put("key", "value");
// 获取数据
Object value = Ognl.getValue("#key", context, root);

// 调用方法
Object value = Ognl.getValue("'word'.length()", context, root);
```

* jsp中使用OGNL
    - 步骤
        1. 引入Struts2提供的标签库, s标签
        2. 使用提供的标签
    - 更多标签可以查看文档
* OGNL符号
    - `#`
        - 取值: `<s:property value="#user"/>`
        - 构建集合:
            - 文本和值相同: `<s:radio name="sex" list="{'男', '女'}"/>`
            - 文本和值不同: `<s:radio name="sex" list="#{'0':'男', '1':'女'}"/>`
    - `%`: 强制解析字符串为OGNL表达式
        - `<s:property value="%'haha'"/>`
    - `$`: 在配置文件中使用OGNL表达式从值栈取值, 类似于EL表达式
        - `<param name="contentType">${contentType}</param>`

```html
<%@ taglib prefix="s" uri="/struts-tags" %>

<!-- 从值栈中获取值, value属性值就使用OGNL表达式 -->
<s:property value="username"/>
```


## ValueStack 值栈

```
+-----------+------------+
|root(List) |context(Map)|
+-----------+------+-----+
|     obj   | key  |value|
+-----------+------+-----+
|           |      |     |
+-----------+------+-----+
|           |      |     |
+-----------+------+-----+
```

* 相当于struts2框架的数据中转站, 用于数据的存取
* `ValueStack`接口, 实现类为`OgnlValueStack`
* Action是多例的, 当请求来时, 创建Action实例, 创建一个ActionContext对象, 代表的是Action的上下文对象, 还会创建一个ValueStack对象
* 每个Action实例都有一个ValueStack对象(即一个请求, 一个ValueStack对象)
* 启动保存当前Action对象和其他相关对象
* struts框架把ValueStack对象保存在名为`struts.valueStack`的请求`request`的属性中
    - `ValueStack vs = (ValueStack) request.getAttribute("struts.valueStack")`
* ValueStack内部结构分析
    - 主要由两部分组成(还有其他)
        - `OgnlValueStack context`
            - map集合
            - struts2强各种各样的映射关系放入这里, 如
                - parameters: 包含当前请求的参数
                - request: 包含request对象所有属性
                - session: 包含session所有属性
                - application: 包含application所有属性
                - attr: 按`request, session, application`这个顺序检索某个属性
        - `CompoundRoot root`
            - List集合: ArrayList
            - struts2强动作和相关对象放入这里
* 取值:
    - 从root中取值: 不加警号, 直接变量名, `key`
    - 从context中取值: 警号加变量名, `#key`
* 值栈的创建和ActionContext的关系
    - 值栈对象是请求时创建的
    - ActionContext是绑定到当前线程上的(通过ThreadLocal实现), 每个拦截器或Action中获取到的ActionContext都是同一个对象
    - ActionContext对象中存在一个Map集合, 该Map集合和ValueStack的contextMap是同一个地址
    - ActionContext中可以获取到ValueStack的引用
* 获取值栈对象的3种方法(实际是2种, 1,2相同)
    1. `ValueStack vs = (ValueStack) ServletActionContext.getRequest().getAttribute("struts.valueStack");`
    2. `ValueStack vs = (ValueStack) ServletActionContext.getRequest().getAttribute(ServletActionContext.STRUTS_VALUESTACK_KEY);`
    3. `ValueStack vs = ActionContext.getContext().getValueStack();`(常用)
* 值栈存值
    - java代码中
        - `valueStack.push(Object obj)`: 将对象压入栈顶, 存到root中, 添加到0索引
        - `valueStack.set(String key, Object obj)`: 将键值对存入Map, 然后压入栈顶, 存到contextMap中, 也是添加到0索引
* 值栈取值
    - jsp中
        - `<s:debug />`: 在页面中点击即可查看值栈内容
        - `<s:property value="[0].top.属性"`: 取栈顶值
            - `[0].top`可以省略, 即`value="属性"`也能取到值
            - 如果是List, 则`[0].top[index]`
        - 迭代root栈取值: `<s:iterator >`
            - `value`: 要迭代的集合
            - `var`: 遍历出来的对象.
                - 写该属性, 则会把迭代产生的对象默认压入context栈, 取值记得使用`#`
                - 不写, 则默认压入root栈
        - 从context栈取值
            - `<s:property value="#request.key"/>`
            - `<s:property value="#session.key"/>`
            - `<s:property value="#application.key"/>`
            - `<s:property value="#attr.key"/>`
            - `<s:property value="#parameters.key"/>`
        - 使用EL表达式和JSTL标签库取值
            - 由于使用装饰器模式, 增强了getAttribute()方法, 实现取值
            - 引入依赖包
            - `<c:forEach item="${ ulist }" var="user">`
                - `${ user.username }`


## Struts2远程代码执行漏洞

* 2017年初
