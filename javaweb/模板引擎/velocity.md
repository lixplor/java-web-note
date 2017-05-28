# Velocity

* Apache开发的基于Java的模板引擎.
* 使用velocity特定的语法`VTL`, 能够在一段模板中引用Java对象的属性, 达到以模板+数据模型来生成特定文本的目的


## 添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.velocity</groupId>
        <artifactId>velocity</artifactId>
        <version>1.7</version>
    </dependency>
    <dependency>
        <groupId>org.apache.velocity</groupId>
        <artifactId>velocity-tools</artifactId>
        <version>2.0</version>
    </dependency>
</dependencies>
```


## 配置VelocityViewServlet

* 在`web.xml`中配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
    <display-name>Demo VM</display-name>
    <servlet>
        <servlet-name>velocity</servlet-name>
        <servlet-class>org.apache.velocity.tools.view.VelocityViewServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>velocity</servlet-name>
        <url-pattern>*.vm</url-pattern>
    </servlet-mapping>
</web-app>
```

### 自定义配置文件: `velocity.properties`

* 自定义配置文件可以参考velocity包中`org.apache.velocity.runtime.defaults.velocity.properties`. 同目录下的`directive.properties`定义了velocity的常用指令.
* 自定义配置文件通常放在`WEB-INF`目录下, 取名为`velocity.properties`. 然后在`web.xml`中的velocity配置的`<servlet>`中增加`<init-param>`来设置配置文件
    - `<param-name>org.apache.velocity.properties</param-name>`
    - `<param-value>自定义velocity.properties文件的路径</param-value>`

```xml
web.xml
-------
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
    <display-name>Demo VM</display-name>
    <servlet>
        <servlet-name>velocity</servlet-name>
        <servlet-class>org.apache.velocity.tools.view.VelocityViewServlet</servlet-class>
        <!-- 配置自定义velocity的配置文件路径 -->
        <init-param>
            <param-name>org.apache.velocity.properties</param-name>
            <param-value>/WEB-INF/velocity.properties</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>velocity</servlet-name>
        <url-pattern>*.vm</url-pattern>
    </servlet-mapping>
</web-app>
```

### 配置编码

* velocity默认使用`ISO-8859-1`编码, 可以在`velocity.properties`中配置输出和输出的编码

```property
input.encoding=utf-8
output.encoding=utf-8
```

### foreach 计数器配置

```property
directive.foreach.counter.name = velocityCount      # 配置计数器变量名称
directive.foreach.counter.initial.value = 1         # 配置计数器变量起始值
directive.foreach.maxloops = -1                     #
directive.foreach.iterator.name = velocityHasNext   # 迭代器变量名
```

### `#parse()`解析深度配置

```property
directive.parse.max.depth = 10
```

### 宏定义配置

```property
velocimacro.library = VM_global_library.vm  # 指定宏定义库
velocimacro.max.depth = 20                  # 指定宏定义嵌套深度
```

### 启用严格模式

```property
velocimacro.arguments.strict = false
```

### 配置日志

* `velocity.log`是velocity引擎的运行日志

```property
runtime.log.logsystem.class = org.apache.velocity.runtime.log.AvalonLogChute,org.apache.velocity.runtime.l og.Log4JLogChute,org.apache.velocity.runtime.log.CommonsLogLogChute,org.apac he.velocity.runtime.log.ServletLogChute,org.apache.velocity.runtime.log.JdkL ogChute     # 配置日志引擎
runtime.log.logsystem.class = org.apache.velocity.runtime.log.NullLogChute # 使用该类则关闭日志
runtime.log = velocity.log   # 日志文件名
```

## toolbox工具箱

* `velocity-tools`提供了实用的Java类
* 使用前需要配置`toolbox.xml`, 通常该配置文件放在`WEB-INF`下, 然后在`web.xml`中的`<servlet>`配置`toolbox.xml`文件的路径

```xml
web.xml
-------
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
    <display-name>Demo VM</display-name>
    <servlet>
        <servlet-name>velocity</servlet-name>
        <servlet-class>org.apache.velocity.tools.view.VelocityViewServlet</servlet-class>
        <!-- 配置自定义velocity的配置文件路径 -->
        <init-param>
            <param-name>org.apache.velocity.properties</param-name>
            <param-value>/WEB-INF/velocity.properties</param-value>
        </init-param>
        <!-- 配置toolbox.xml -->
        <init-param>
            <param-name>org.apache.velocity.toolbox</param-name> <param-value>/WEB-INF/toolbox.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>velocity</servlet-name>
        <url-pattern>*.vm</url-pattern>
    </servlet-mapping>
</web-app>
```

```xml
toolbox.xml
-----------
<?xml version="1.0" encoding="UTF-8"?>
<toolbox>
    <tool>
        <key>date</key>
        <scope>request</scope>
        <class>org.apache.velocity.tools.generic.DateTool</class>
        <parameter name="format" value="yyyy-MM-dd HH:mm:ss"/>
    </tool>
</toolbox>
```

```vm
## 使用小工具的key作为变量名即可使用工具
$date   ## 会输出相应格式的时间
```

### 自定义toolbox

* 步骤
    - 定义一个工具类, 提供相关方法
    - 在`toolbox.xml`中添加该类
    - 在velocity模板中调用变量名及其方法即可

```java
public class TimeUtils {

    public static String getCurrentTimestamp() {
        return "" + System.currentTimeMillis();
    }
}
```

```xml
toolbox.xml
-----------
<?xml version="1.0" encoding="UTF-8"?>
<toolbox>
    <tool>
        <key>timeutils</key>
        <scope>application</scope>
        <class>com.lixplor.velocityhelloworldtest.controller.TimeUtils</class>
    </tool>
</toolbox>
```

```vm
$timeutils.getCurrentTimestamp()   ## 显示时间戳
```


## Velocity和Spring结合

* 在`web.xml`中, `<servlet>`需要使用SpringMVC的`DispatcherServlet`, 而不能使用velocity的`VelocityViewServlet`
* SpirngMVC的视图文件使用`.vm`模板, 在控制器方法中返回模板文件名的字符串, 会自动渲染
* 模板文件中的变量的值, 通过控制器的`model.addAttribute(key, value)`方式传递
* 对于表单提交的参数, 可以通过velocity提供的`springBind`宏来将表单的参数绑定到变量的属性中, 从而提交表单时, 将该变量作为一个bean提交到控制器
* velocity默认没有将session域中的对象绑定到视图模型. 如果希望vm模板能够直接访问session域, 可在初始化`VelocityViewResolver`时加入`<property name="exposeSessionAttributes" value="true" />`. 同理也可以开启对request域的绑定


## VTL语法

* 注释

```vm
## 单行注释

#*
  多行
  注释
*#
```

* 引用变量, 属性, 方法: `$xxx`
    - 变量: 输出变量的`toString()`方法
    - 属性: 属性必须可以访问, 或通过`get`方法可以访问
    - 方法: 显示该方法的输出结果

```vm
## 输出变量. 相当于输出user.toString(). 如果变量不存在, 或变量的值为null, 则会直接输出$user
$user

## 输出存在的变量. 如果变量不存在或变量的值为null, 则不输出任何东西
$!user

## 引用对象的属性
$user.password

## 引用对象的方法
$user.sayHello()
$user.print($user)
```

* 如果将变量设置为null, 变量的值并不会改变为null. 在判断时要特别注意

```vm
#set($a="aaa")                  ## $a有初始值
#set($a=$nullMethod.setNull())  ## 该方法将$a设置为null
<p>$a</p>                       ## 仍然输出aaa
```

* 创建变量: `#set()`

```vm
## 创建一个变量
#set($person = "Tom")
$person               ## 输出Tom

## 创建一个集合
#set($list = ["a", "b", "c"])
#set($map = {"a":"aaa", "b":"bbb", "c":"ccc"})

## 获取元素
$list.get(0)
$list[0]
$map.get("a")
$map["a"]

## 修改元素
$list[0] = "d"

## 调用集合的方法
$list.isEmpty()
```

* 变量和字符的连接

```vm
#set($name = "Tom")
My name is $name. Nice to see you!    ## 输出My name is Tom. Nice to see you!
```

* 特殊字符的转义

```vm
## 示例
#set($a = "aaa")
\$a     ## 输出$a
\\$a    ## 输出\aaa
\\\$a   ## 输出\$a
\\\\$a  ## 输出\\aaa
```

* 条件判断: `#if()...#elseif()...#else...#end`
    - 判断只能比较相同类型的变量, 如果`==`两端变量类型不同, 则返回`false`
    - 对于字符串类型, 字符串内容相同则返回`true`
    - 对于Java对象, 调用`equals()`方法进行比较
    - 可以通过重写类的`equals()`和`hashCode()`改变比较的结果
    - 可以使用`&&`和`||`

```vm
#set($a = 10)

#if($a > 0)
    <p>bigger than 0</p>
#elseif($a > 5)
    <p>bigger than 5</p>
#elseif($a > 10)
    <p>bigger than 10</p>
#else
    <p>others</p>
#end
```

* 循环: `#foreach($var in $list)...#end`
    - 迭代过程中, 可以使用整型内置变量`velocityCount`获取迭代次数, 值默认从`1`开始, 可以在配置文件中修改.
        - 通过该值可以实现表格不同行的颜色区分
    - 可以使用布尔类型内置变量`velocityHasNext`来判断是否还有下一个元素
        - 可用于判断迭代是否完成

```vm
## 迭代list
#set($list = ["a", "b", "c"])
<ul>
    #foreach($item in $list)
        <li>$item</li>
    #end
</ul>

## 迭代map
#set($map = ["a":"aaa", "b":"bbb"])
<ul>
    #foreach($value in $map)             ## 直接获取value
        <li>$value</li>
    #end
</ul>
<ul>
    #foreach($key in $map.keySet())      ## 获取key
        <li>$key : $map.get($key)</li>   ## 通过map的get方法获取value
    #end
</ul>
```

* 跳出循环: `#break`
    - 结束循环操作
    - 适用于在循环过程中满足某种条件下结束循环
    - 结束循环后, 循坏以后的内容仍然会输出, 与`#stop`不同

```vm
#set($list = ["a", "b", "c"])
<ul>
    #foreach($item in $list)
        #if($item == "b")
            #stop
        #end
        <li>$item</li>
    #end
</ul>
<p>该行可以输出, 因为使用break</p>
```

* 停止模板引擎: `#stop`
    - 停止模板引擎, 停止位置之后的所有内容都不会被输出
    - 适用于处理异常情况

```vm
#set($list = ["a", "b", "c"])
<ul>
    #foreach($item in $list)
        #if($item == "b")
            #stop
        #end
        <li>$item</li>
    #end
</ul>
<p>该行不会被输出, 因为stop了</p>
```

* 引入文本文件: `#include(String... paths)`和`#parse(String... paths)`
    - `#include()`引入的文本不会被模板引擎渲染, 按照字符串显示
    - `#parse()`引入的文本会被模板引擎渲染

```vm
#include("/WEB-INF/views/module1.vm", "/WEB-INF/views/module2.vm")
#parse("/WEB-INF/views/module1.vm", "/WEB-INF/views/module2.vm")
```

* 宏定义: `#macro()...#end`
    - 与C语言类似, 可用于设置常量, 封装函数
    - 宏定义的参数可以是变量, List, Map, Java基本类型, 字符串

```vm
## 定义格式
#macro(宏名称 参数)
    代码操作
#end

## 调用格式
#宏名称(参数)

## 示例, 定义一个宏, 并调用
#macro(show_table $list)
    <table border="1">
        #foreach($item in $list)
            <tr>
                <td>$item</td>
            </tr>
        #end
    </table>
#end

#set($list = ["a", "b", "c"])
#show_table($list)
```
