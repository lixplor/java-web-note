# FreeMarker

* [中文文档](http://freemarker.foofun.cn/index.html)
* FreeMarker是一款模板引擎
* 在线编辑器: http://try.freemarker.org/


## 基本指令

* 表达式(插值, interpolation)

```
${...}
```

* FTL标签(指令)
    - 以`#`开头
    - 用户自定义标签使用`@`开头

```
<#...></#...>
```

* 注释

```
<#-- 注释内容 -->
```

### 条件渲染

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


### 列表渲染

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


### 引入其他模板

* include指令

```
格式
<#include "其他页面文件url">

示例
<#include "/hello.html">
<#include "/user.ftl">
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