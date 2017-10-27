# FreeMarker

* [中文文档](http://freemarker.foofun.cn/index.html)
* FreeMarker是一款模板引擎
* 在线编辑器: http://try.freemarker.org/


## 语法

* FTL语法会忽略表达式中多余的空格

### 数据类型

* 基本数据
    - 字符串
        - `"Foo"`
        - 字符串操作:
            - 插值: `hello ${username}`
            - 字符串拼接: `"hello" + ${user} + "!"`
            - 获取字符串中的字符: `${username[2]}`
            - 字符串切分:
                - 包含结尾: `${name[0..3]}`
                - 不包含结尾: `${name[0..<3]}`
                - 基于长度(最多到10, 至少到本身的长度): `${name[0..*10]}`
                - 去除开头: `${name[1..]}`
    - 数字
        - `123.45`
        - 不区分整数和小数, 如3/2结果是1.5
    - 布尔值
        - `true`, `false`
    - 日期/时间
        - 日期: 精确到天, 没有时间. 如`April 4, 2003`
        - 时间: 精确到毫秒, 没有日期. 如`10:19:18 PM`
        - 日期时间: 精确到毫秒, 如`April 4, 2003 10:19:18 PM`
* 容器
    - 哈希表: 键值对
        - `{"name":"mouse", "price":150}`
    - 序列: 即数组
        - `["foo", "bar", 123.45]`
        - `0..9`
        - `0..<10`
        - `0..!=10`
        - `0..`
    - 集合
* 子程序
    - 方法和函数
    - 用户自定义指令
* 节点: 配合XML


### 运算


* 算术运算
    - `(x * 1.5 + 10) % 2`
* 比较运算
    - `x == y`
    - `x > y`, `x gt y`
* 逻辑运算
    - `!login && (newUser || notInBlacklist)`


### 赋值

* `=`
* `+=`
* `-=`
* `*=`
* `/=`
* `%=`
* `++`
* `--`

### 获取变量值

* 直接使用变量
    - `username`
* 从哈希表中获取值
    - `user.name`
    - `user["name"]`
* 从序列中获取值
    - `products[5]`
* 特殊变量
    - `.main`

### 表达式(插值, interpolation)

* 插值的使用位置
    - 在文本区域中, 使用`${表达式}`方式
    - 在FTL中, 直接使用变量名即可: `<#if 表达式></#if>`
    - 使用插值时需要考虑插值的内容是否会是HTML标签或是script脚本, 避免XSS攻击
        - 可以使用`<#espace>${code}</#escape>`标签来直接显示代码
        - 对于特殊符号, 如果需要转义, 则可以使用`<#noescape></#noescape>`避免转换为显示代码

```
格式
${表达式}
<#标签 表达式></#标签>

示例
${username}
${(5 + 8) / 2}
<#if isLogin></#if>
```

### 内建函数

* `?`
    - `变量?html`: 返回变量的HTML转义版本, 如`&`会被`&amp;`代替
    - `变量?upper_case`: 返回变量的大写方式, 如`John`会被`JOHN`代替
    - `变量?cap_first`: 返回变量的首字母大写方式
    - `变量?length`: 返回变量字符的长度
    - `变量?size`: 返回变量集合的长度
    - 在`<#list>`中使用
        - `变量?index`: 从0开始的索引
        - `变量?couter`: 从1开始的索引
        - `变量?item_parity`: 返回字符串`odd`或`even`, 即奇偶, 可用于表格行渲染
* `.`
    - `变量?string("true时的返回值", "false时的返回值")`: 三元表达式返回字符串
    - `变量?item_cycle('true时的返回值', 'false时的返回值')`: item_parity的变体
    - `变量?join("分隔符")`: 使用分隔符连接集合的所有项目, 组成一个字符串
    - `变量?starts_with("字符")`: 是否以指定字符开头, 返回布尔值
* 内建函数可以链式操作
    - `username?upper_case?html`


### 处理不存在的变量

* 2种情况可以处理
    - 变量名不存在
    - 变量值为null
* 处理方式:
    1. 指定默认值
        - 格式: `${变量名!"默认值"}`
    2. 判断是否存在, 配合`<#if>`实现
        - 格式: `<#if user??></#if>`
* 注意:
    - 对于多级调用的变量, 需要使用小括号将多级变量括起来, 以免因为中间有变量不存在而停止调用: 
        - `(user.friend.age)!0`
        - `(user.friend.name)??`


### 处理空白

* 模板中的空白字符可以通过2种方式处理
    - 剥离空白
        - 开启指令: `<#ftl strip_whitespace=true>`
        - 效果: 对所有模板剥离空白, 具体看文档
    - compress指令
        - `<#compress>中间的代码会压缩掉空白字符</#compress>`
        - 效果: 具体看文档


### 预定义指令

* FTL标签(指令)
    - 预定义指令
        - 以`#`开头
        - 格式: `<#指令名称 参数></#指令名称>`
        - 如果标签没有嵌套内容, 则可以只使用开始标签
    - 用户自定义指令
        - 以`@`开头
        - 格式: `<@指令名称 参数></@指令名称>`
        - 如果标签没有嵌套内容, 则必须写为自封闭标签: `<@指令名称 参数/>`

```
<#...></#...>
```

* 注释

```
<#-- 注释内容 -->
```

#### 条件渲染

* if指令

```
格式
<#if 条件>
    条件满足时的代码
</#if>

示例
${user}<#if user == "Big Joe">
    , good morning!
</#if>
```

* else指令

```
格式
<#else>

示例
<#if a.price > b.price>
    a比b贵
<#else>
    b比a贵
</#if>
```

* elseif指令

```
格式
<#elseif>

示例
<#if a.price > b.price>
    a比b贵
<#elseif a.price == b.price>
    a和b一样贵
<#else>
    b比a贵
</#if>
```


#### 列表渲染

* list指令

```
格式
<#list 集合 as 元素></#list>


示例
<ul>
    <#list todos as todo>
        <li>${todo.priority}, ${todo.content}</li>
    </#list>
</ul>
```

* list配合items指令

```
格式: 当集合没有元素时不会输出
<#list 集合>
    <#items as 元素>...</#items>
</#list>

示例: 这种方式当集合没有元素时, 不会输出ul标签
<#list todos>
    <ul>
        <#items as todo>
            <li>${todo.priority}, ${todo.content}</li>
        </#items>
    </ul>
</#list>
```

* list配合sep分隔符
    - 只有当还有下一项时才会输出分隔符, 而最后一项不会输出

```
格式
<#sep></#sep>

示例
<#list todos as todo>
    ${todo.content}<#sep>, </#sep>
</#list>
```

* list配合else判断没有元素时的输出

```
<#list todos>
    <ul>
        <#items as todo>
            <li>${todo.content}<#sep>, </#sep>
        </#items>
    </ul>
<#else>
    <p>没有待办事项
</#list>
```


#### 引入其他模板

* include指令

```
格式
<#include "其他页面文件url">

示例
<#include "/hello.html">
<#include "/user.ftl">
```

#### 定义变量

* 简单变量: 
    - `<#assign x = 1>`
    - 这种方式定义的变量可以从模板的任意位置访问, 或从include引入的模板访问
* 局部变量: 
    - `<#local x = 1>`
    - 只能定义在宏定义体中, 只在宏内可见
* 循环变量
    - 由list指令自动创建
* 全局变量
    - `<#global x = 1>`
    - 尽量避免使用全局变量, 可能会导致冲突

```
简单变量
<#assign x = 1>
${x}

局部变量
<#macro test>
  <#local x = "local">
  ${x}
</#macro>

循环变量
<#list ["loop 1"] as x>
  ${x}
</#list>

全局变量

```

#### 命名空间

* 命名空间用于区分同名的变量或宏
* 通常可以在一个`.ftl`文件中专门定义变量或宏, 作为一个库, 以便引用
* 使用`<#import "ftl路径" as 命名空间名称>`来引入一个ftl文件中定义的变量或宏
    - 注意import和include是不同的
* 使用`命名空间名称.变量`或`命名空间名称.宏`来调用

```
<#import "/lib/my_test.ftl" as my>  引入命名空间
<#assign mail="fred@acme.com">      本文件中定义的同名变量

使用命名空间中的宏
<@my.copyright date="1999-2002"/>

以下两个变量不会冲突
${my.mail}
${mail}

如果想修改命名空间中定义的变量或宏也是可以的, 用in
<#assign mail="jsmith@other.com" in my>
```

#### 定义函数

* function指令

```
定义函数格式
<#function 函数名 参数1 参数2 ... 参数N>
    函数体
    <#return 返回值>
</#function>

调用函数
${函数名(参数1, 参数2)}

示例
<#function avg x y>
    <#return (x + y) / 2>
</#function>

${avg(10, 20)}
```



### 自定义指令

* 定义和使用自定义指令:

```
定义一个指令
<#marco 自定义指令名称 参数1 参数2>
    指令内容
</#macro>

使用自定义的指令
<@自定义指令名称></@自定义指令名称>

示例
<#macro greet name words>
    <font size="+2">Hello ${name}, ${words}</font>
</#greet>

<@greet name="John" words="good morning!"></@greet>
<@greet/>
```

* 嵌套内容
    - 使用`<#nested>`标签表示内容
```
格式
<#macro do_thrice>
    <#nested>
    <#nested>
    <#nested>
</#macro>

示例
<@do_thrice>
  Anything.
</@do_thrice>

效果
Anything.
Anything.
Anything.
```


### 模板配置

* 使用ftl指令进行配置
    - 语法: `<#ftl 参数=值 参数=值>`
    - 参数:
        - `encoding`: 模板的编码
        - `strip_whitespace`: 是否剥离空白, 值为true或false
        - `strip_text`: 是否移除模板中的顶级文本, 值为true或false
        - `strict_syntax`: 是否开启严格语法. 值为true或false
        - `ns_prefixeds`: 命名空间的前缀
        - `attributes`: 关联模板属性
    - 使用中括号替换尖括号
        - 在任意模板的开头定义一个`[#ftl]`即可
        - 开启中括号后所有模板指令不能再使用尖括号





