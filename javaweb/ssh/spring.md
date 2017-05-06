# Spring

* 业务层框架
* 可单独开发, 也可结合其他框架开发
* 核心: IOC, AOP


## 框架体系结构

* Spring框架提供20多个模块, 可根据程序要求引入需要的模块
    - 核心容器(Core Container)
        - Core: 框架基本组成部分, 包括IoC和依赖注入功能
        - Bean: 提供BeanFactory, 是一个工厂模式的复杂实现
        - Context: 是访问定义和配置的任何对象的媒介. ApplicationContext接口是Context的重点
        - SpEL: 提供了查询和操作一个对象图的表达式语言
    - 数据访问/集成(Data Access/Integration)
        - JDBC: 提供了JDBC抽象层
        - ORM: 对象关系映射API, 包括JPA, JDO, Hibernate, iBatis
        - OXM: 支持对JAXB, Castor, XMLBeans, JiBX, XStream的对象/XML映射
        - JMS: Java消息服务, 包含生产和消费的信息
        - Transaction: 为实现特殊接口的类和所有的POJO支持编程式和声明式的事务管理
    - Web
        - Web: 基本的面向Web集成功能
        - Web-MVC: 包含Spring MVC
        - Web-Socket: socket通信
        - Web-Portlet: 在portlet环境中实现MVC
    - 其他
        - AOP: 提供面向切面的编程实现
        - Aspects: 提供与AspectJ的集成, 这是一个成熟的AOP框架
        - Instrumentation: 提供类instumentation的支持和类加载器的实现
        - Messaging: 为STOMP提供了WebSocket协议
        - Test: 支持对具有JUnit或TestNG框架的Spring组件测试

![Spring框架结构图](http://img.w3cschool.cn/attachments/image/wk/wkspring/arch1.png)



## 环境配置

* JDK
* Spring框架官网:  [http://projects.spring.io/spring-framework/](http://projects.spring.io/spring-framework/)


## 快速入门

* 步骤:
    - 创建一个Service类
    - 创建`src/applicationContext.xml`, 添加schema约束, 声明Service类的bean
    - 创建工厂, 加载核心配置文件, 获取对象, 调用对象方法

```xml
配置文件: applicationContext.xml
-------------------------------
<?xml version="1.0" encoding="UTF-8" ?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
    xsi:shemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 使用bean标签 -->
    <bean id="userDao" class="xxx.package.UserDao"/>
    <bean id="userService" class="xxx.package.UserServiceImpl">
        <!-- 使用依赖注入实现属性初始化 -->
        <property name="username" value="John"/>
        <property name="userDao" ref="userDao"/>
    </bean>

</beans>
```

```java
使用spring方式
-------------
// 创建工厂, 加载配置文件
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
// 从工厂获取对象
UserServiceImpl usi = (UserServiceImpl) context.getBean("userService");
// 调用对象
usi.saveUser();
```


## 工厂类

* 早期使用`BeanFactory`, 现已由`ApplicationContext`代替
* `ApplicationContext`接口
    - 用于获取Bean对象
    - 有2个实现类:
        - `ClassPathXmlApplicationContext`: 加载类路径下的Spring配置文件
        - `FileSystemXmlApplicationContext`: 加载本地磁盘的Spring配置文件
* 2者区别
    - BeanFactory使用延迟加载, 第一次调用`getBean()`时才会初始化Bean
    - ApplicationContext在加载applicationContext.xml时就会创建Bean, 还提供了事件传递, Bean自动装配, 各种不同Context的实现等其他功能


## applicationContext.xml详解

* `<bean>`标签
    - `id`: 起到区分作用, 不允许出现特殊符号
    - `class`: 要创建对象的完整类名
    - `name`: 和id类似, 可以使用特殊符号
    - `scope`: bean的作用范围
        - `singleton`: 默认, 单例
        - `prototype`: 多例, 结合struts2的action
        - `request`: 应用在Web项目中, 每次Http请求都会胡藏剑一个新的bean
        - `session`: 应用在Web项目中, 同一个Http Session共享一个bean
        - `globalsession`: 应用在Web项目中, 多服务器之间的session
* Bean对象创建和销毁的2个生命周期属性
    - `init-method`: 当bean被载入到容器的时候, 调用init-method属性指定的方法
    - `destroy-method`: 当bean从容器中删除的时候调用destroy-method属性指定的方法


## 配置文件管理

* 可以使用多个配置文件
* 2种方式
    1. xml方式, 在主配置文件中引入其他配置文件
        - `<import resource="com/xxx/xxx.xml"/>`
    2. 代码方式
        - `new ClassPathXmlApplicationContext(String...)`


## IoC容器

* IoC, Inverse of Control, 控制翻转. 将对象的创建权反转给框架, 实现解耦
    - IoC的实现方式有2种:
        1. 依赖查找
        2. 依赖注入(DI)
* IoC和DI的关系: IoC是目的, DI是手段
* Spring的IoC实现方式:
    - `业务代码`和`资源`通过中间的`工厂类`和`配置文件`控制, 实现解耦
* Spring容器是Spring框架的核心. 容器会创建对象, 把他们链接在一起, 配置他们, 并管理他们的整个生命周期
* Spring容器使用依赖注入(DI)来管理组成一个应用程序的组件, 这些对象被称为`Spring Beans`
* 通过配置, 容器知道对哪些对象进行实例化, 配置和组装. 配置可以通过`XML`, ``注解``或``代码``来实现
* Spring提供了2种不同类型的容器:
    1. Spring BeanFactory容器: 为DI提供基本的支持, 使用`org.springframework.beans.factory.BeanFactory`接口来定义
    2. Spring ApplicationContext容器: 添加了更多的企业特定的功能, 由`org.springframework.context.ApplicationContext`接口定义
        - ApplicationContext容器包括BeanFactory容器的所有功能, 所以建议使用ApplicationContext

![Spring容器工作图](http://img.w3cschool.cn/attachments/image/wk/wkspring/ioc1.jpg)

### Spring BeanFactory容器

* 最简单的容器, 主要为依赖注入(DI)提供支持
* 使用`org.springframework.beans.factory.BeanFactory`接口来定义
* Spring中有大量对BeanFactory的实现, 如`XmlBeanFactory`类, 该容器从XML文件读取配置, 生成一个配置后的系统或应用

#### 使用BeanFactory的示例

* 步骤
    1. 创建一个名为`SpringExample`的工程并在`src`文件夹下新建一个名为`com.tutorialspoint`文件夹
    2. 点击右键, 选择 `Add External JARs` 选项, 导入 Spring 的库文件
    3. 在 `com.tutorialspoint` 文件夹下创建 `HelloWorld.java` 和 `MainApp.java` 两个类文件
    4. 在 `src` 文件夹下创建 `Bean` 的配置文件 `Beans.xml`
    5. 最后的步骤是创建所有 Java 文件和 Bean 的配置文件的内容, 按照如下所示步骤运行应用程序
* 解释:
    - 使用`XmlBeanFactory`来加载xml配置
    - 通过`XmlBeanFactory`的`getBean(String id)`方法来创建JavaBean
    - 这就是IoC, 我们并没有直接在类中new对象, 而是通过一个XML文件, 将Bean和要使用Bean对象的地方连接起来, 达到解耦的目的
* 示例

```xml
Beans.xml
---------
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
       <property name="message" value="Hello World!"/>
   </bean>

</beans>
```

```java
HelloWorld.java
---------------
package com.tutorialspoint;

public class HelloWorld {
   private String message;
   public void setMessage(String message){
       this.message  = message;
   }
   public void getMessage(){
       System.out.println("Your Message : " + message);
   }
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.xml.XmlBeanFactory;
import org.springframework.core.io.ClassPathResource;

public class MainApp {
   public static void main(String[] args) {
      XmlBeanFactory factory = new XmlBeanFactory(new ClassPathResource("Beans.xml"));
      HelloWorld obj = (HelloWorld) factory.getBean("helloWorld");
      obj.getMessage();
   }
}
```

### Spring ApplicationContext容器

* 是一个较高级的容器, 和BeanFactory类似, 用于加载配置文件中定义的Bean
* 还增加了企业所需要的功能, 如从属性文件解析文本信息, 将时间传递给指定的监听器
* 由`org.springframework.context.ApplicationContext`接口定义
* 常用的ApplicationContext实现类:
    - `FileSystemXmlApplicationContext`: 从XML文件中加载被定义的Bean. 需要提供XML文件的完整路径
    - `ClassPathXmlApplicationContext`: 从XML文件中加载被定义的Bean. 不用提供XML文件的完整路径, 只需要配置Classpath即可
    - `WebXmlApplicationContext`: 在Web应用程序的范围内加载XML文件中被定义的Bean

#### 使用ApplicationContext的示例

* 步骤:
    1. 创建一个名为 `SpringExample` 的工程, 在 `src` 下新建一个名为 `com.tutorialspoint` 的文件夹`src`
    2. 点击右键, 选择 `Add External JARs` 选项, 导入 Spring 的库文件
    3. 在 `com.tutorialspoint` 文件夹下创建 `HelloWorld.java` 和 `MainApp.java` 两个类文件
    4. 文件夹下创建 Bean 的配置文件 `applicationContext.xml`
    5. 最后的步骤是编辑所有 JAVA 文件的内容和 Bean 的配置文件, 按照以前我们讲的那样去运行应用程序
* 解释
    - 使用`FileSystemXmlApplicationContext`来加载xml配置
    - 通过`FileSystemXmlApplicationContext`的`getBean(String id)`方法来创建JavaBean
    - 这就是IoC, 我们并没有直接在类中new对象, 而是通过一个XML文件, 将Bean和要使用Bean对象的地方连接起来, 达到解耦的目的
* 示例

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
       <property name="message" value="Hello World!"/>
   </bean>

</beans>
```

```java
HelloWorld.java
---------------
package com.tutorialspoint;
public class HelloWorld {
   private String message;
   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = new FileSystemXmlApplicationContext
            (".../HelloSpring/src/applicationContext.xml");
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
   }
}
```

### Bean的定义

* Bean对象通过IoC容器来管理
* 所有的Bean都由容器提供的配置文件来创建, 即在XML中定义
* Bean的配置信息包括:
    - `class`属性: 必须. 指定创建bean的类名
    - `name`属性: 指定唯一的bean标识符. 与id类似
    - `scope`属性: 必须. 指定由特定bean定义创建对象的作用域
    - `constructor-arg`: 用于注入依赖关系
    - `properties`: 用于注入依赖关系
    - `autowiring mode`: 用于注入依赖关系
    - `lazy-initialization mode`: 延迟初始化bean, 也就是在第一次请求时创建一个bean实例, 而不是在启动时创建
    - `initialization`(`init-method`): 在bean的所有必须的属性被容器设置之后, 调用回调方法
    - `destruction`(`destroy-method`): 当包含该bean的容器被销毁时, 使用回调方法
* 配置Bean有3种方式:
    - XML配置文件
    - 注解
    - Java代码

* XML配置示例

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- A simple bean definition -->
   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with lazy init set on -->
   <bean id="..." class="..." lazy-init="true">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with initialization method -->
   <bean id="..." class="..." init-method="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with destruction method -->
   <bean id="..." class="..." destroy-method="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- more bean definitions go here -->

</beans>
```

### Bean的作用域

* 定义bean时, 必须声明作用域
* 5个作用域:
    - `singleton`: 该作用域将 bean 的定义的限制在每一个 Spring IoC 容器中的一个单一实例(默认)
    - `prototype`: 该作用域将单一 bean 的定义限制在任意数量的对象实例
    - `request`: 该作用域将 bean 的定义限制为 HTTP 请求. 只在 web-aware Spring ApplicationContext 的上下文中有效
    - `session`: 该作用域将 bean 的定义限制为 HTTP 会话. 只在web-aware Spring ApplicationContext的上下文中有效
    - `global-session`: 该作用域将 bean 的定义限制为全局 HTTP 会话. 只在 web-aware Spring ApplicationContext 的上下文中有效

### Bean的生命周期回调

* 初始化回调: `void afterPropertiesSet() throws Exception;`, 可在其中进行初始化操作
* 销毁回调: `void destroy() throws Exception;`, 可在对象销毁后进行一些收尾工作
* 配置默认初始化和销毁回调方法:
    - 我们可以通过默认配置来为所有Bean配置相同方法名的回调方法
    - `default-init-method`: 默认的初始化回调
    - `default-destroy-method`: 默认的销毁回调

```xml
applicationContext.xml
----------------------
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init"
    default-destroy-method="destroy">

   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

</beans>
```


#### 使用IoC方式定义回调方法

* 解释:
    - 我们在xml中配置了:
        - 当对象创建时, 调用对象的`init()`方法
        - 当对象销毁时, 调用对象的`destroy()`方法
* 示例:

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld"
       class="com.tutorialspoint.HelloWorld"
       init-method="init" destroy-method="destroy">
       <property name="message" value="Hello World!"/>
   </bean>

</beans>
```

```java
HelloWorld.java
---------------
package com.tutorialspoint;

public class HelloWorld {
   private String message;

   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
   public void init(){
      System.out.println("Bean is going through init.");
   }
   public void destroy(){
      System.out.println("Bean will destroy now.");
   }
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      AbstractApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
      context.registerShutdownHook();
   }
}
```

### Bean的后置处理器

* `BeanPostProcessor`接口也是定义回调方法的, 用于在所有的bean实例化的前后进行回调
* 对所有bean都生效

#### 使用BeanPostProcessor进行回调

* 解释
    - 定义了`InitHelloWorld`类, 实现了`BeanPostProcessor`接口, 重写了两个回调方法
    - 在xml中声明了`InitHelloWorld`这个处理器
    - 在创建所有的bean的前后时机, 都会调用两个回调
* 示例

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld"
       init-method="init" destroy-method="destroy">
       <property name="message" value="Hello World!"/>
   </bean>

   <bean class="com.tutorialspoint.InitHelloWorld" />

</beans>
```

```java
HelloWorld.java
---------------
package com.tutorialspoint;
public class HelloWorld {
   private String message;
   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
   public void init(){
      System.out.println("Bean is going through init.");
   }
   public void destroy(){
      System.out.println("Bean will destroy now.");
   }
}
```

```java
InitHelloWorld.java
-------------------
package com.tutorialspoint;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.BeansException;
public class InitHelloWorld implements BeanPostProcessor {
   public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("BeforeInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
   public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("AfterInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      AbstractApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
      context.registerShutdownHook();
   }
}
```

### Bean的继承

* Spring Bean的继承与Java类的继承无关, 但继承的概念是一样的. 用于将一个父bean作为模板, 继承所需的配置
* 使用`parent`属性来声明要继承的bean

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
      <property name="message1" value="Hello World!"/>
      <property name="message2" value="Hello Second World!"/>
   </bean>

   <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="helloWorld">
      <property name="message1" value="Hello India!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>

</beans>
```

#### Bean模板

* 可以创建一个Bean的模板, 供其他继承
* 模板不用指定`class`属性, 而要添加`abstract=true`的属性

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="beanTeamplate" abstract="true">
      <property name="message1" value="Hello World!"/>
      <property name="message2" value="Hello Second World!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>

   <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="beanTeamplate">
      <property name="message1" value="Hello India!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>

</beans>
```



## DI 依赖注入

* DI, Dependency Injection, 依赖注入, 动态将依赖对象注入. 用于实现IoC
* Spring提供了2种DI方式:
    - 构造函数DI
    - set方法DI

### 基于构造函数的DI

* 通过Bean的构造函数由容器来创建依赖
    - 在xml中bean的定义中, 使用`<constructor-arg ref="{beanId}"/>`来声明一个依赖. 这种方式使用构造方法来创建对象
    - 如果构造方法有多个参数, 则在定义其他bean时, 要将定义的顺序改为构造方法的参数顺序
* 解释
    - 在`TextEditor`类中有成员属性`SpellChecker`对象, 这是一种依赖, 要将其通过注入的方式解耦
    - 在`TextEditor`类的构造方法中, 我们看到若要创建`TextEditor`需要传入一个`SpellChecker`对象
    - 在xml中的`TextEditor`类的bean定义中, 使用`<constructor-arg ref="{beanId}"/>`为`TextEditor`类声明了一个依赖`SpellChecker`
    - 在`MainApp.java`使用ApplicationContext生成了一个`TextEditor`类, 内部先创建了`SpellChecker`对象的依赖
    - 在这里, 并不需要手动去new一个SpellChecker对象传入
* 示例

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <constructor-arg ref="spellChecker"/>
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

```java
TextEditor.java
---------------
package com.tutorialspoint;
public class TextEditor {
   private SpellChecker spellChecker;
   public TextEditor(SpellChecker spellChecker) {
      System.out.println("Inside TextEditor constructor." );
      this.spellChecker = spellChecker;
   }
   public void spellCheck() {
      spellChecker.checkSpelling();
   }
}
```

```java
SpellChecker.java
-----------------
package com.tutorialspoint;
public class SpellChecker {
   public SpellChecker(){
      System.out.println("Inside SpellChecker constructor." );
   }
   public void checkSpelling() {
      System.out.println("Inside checkSpelling." );
   }
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context =
             new ClassPathXmlApplicationContext("applicationContext.xml");
      TextEditor te = (TextEditor) context.getBean("textEditor");
      te.spellCheck();
   }
}
```

```bash
# 输出结果
Inside SpellChecker constructor.  # 先创建了依赖对象
Inside TextEditor constructor.
Inside checkSpelling.
```

### 基于set方法的DI

* 通过Bean的set方法来注入依赖对象
    - 在xml中bean的定义中, 使用`<property name="{objectName} ref="{beanId}"/>`来声明一个依赖. 这种方式使用set方法来创建对象
    - 如果构造方法有多个参数, 则在定义其他bean时, 要将定义的顺序改为构造方法的参数顺序
* 解释
    - 在`TextEditor`类中有成员属性`SpellChecker`对象, 这是一种依赖, 要将其通过注入的方式解耦
    - 在`TextEditor`类中可以通过`setSpellChecker(SpellChecker spellChecker)`方法来注入一个`SpellChecker`对象
    - 在xml中的`TextEditor`类的bean定义中, 使用`<property name="spellChecker" ref="spellChecker"/>`为`TextEditor`类声明了一个依赖`SpellChecker`
    - 在`MainApp.java`使用ApplicationContext生成了一个`TextEditor`类, 内部先创建了`SpellChecker`对象的依赖
    - 在这里, 并不需要手动去new一个SpellChecker对象传入
* 示例

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker" ref="spellChecker"/>
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

```java
TextEditor.java
---------------
package com.tutorialspoint;
public class TextEditor {
   private SpellChecker spellChecker;
   // a setter method to inject the dependency.
   public void setSpellChecker(SpellChecker spellChecker) {
      System.out.println("Inside setSpellChecker." );
      this.spellChecker = spellChecker;
   }
   // a getter method to return spellChecker
   public SpellChecker getSpellChecker() {
      return spellChecker;
   }
   public void spellCheck() {
      spellChecker.checkSpelling();
   }
}
```

```java
SpellChecker.java
-----------------
package com.tutorialspoint;
public class SpellChecker {
   public SpellChecker(){
      System.out.println("Inside SpellChecker constructor." );
   }
   public void checkSpelling() {
      System.out.println("Inside checkSpelling." );
   }  
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context =
             new ClassPathXmlApplicationContext("applicationContext.xml");
      TextEditor te = (TextEditor) context.getBean("textEditor");
      te.spellCheck();
   }
}
```

```bash
# 运行结果
Inside SpellChecker constructor. # 依赖被创建
Inside setSpellChecker.
Inside checkSpelling.
```


### 使用DI注入内部Bean

* 有可能一个Bean中还有另一个Bean作为属性, 这称为`内部Bean`
* 如果要注入的是这个内部Bean, 那么我们只需要在xml中的属性内部配置bean

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean using inner bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker">
         <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"/>
       </property>
   </bean>

</beans>
```

### 使用DI注入集合

* 当某个类的依赖不是某个对象, 而是一个集合(List, Set, Map, Properties), 我们把这种注入称为注入集合
* 对应的注入标签:
    - `List`: `<list></list>`
    - `Set`: `<set></set>`
    - `Map`: `<map></map>`
    - `Properties`: `<props></props>`
* 集合中的值:
    - 当集合中的值是基本类型时, 使用`<value></value>`, `<entry key="" value=""/>`, `<prop></prop>`来设置值
    - 当集合中的值是引用类型时, 我们可以使用`<ref bean=""/>`来设置值
    - 如果值为null, 则使用`<null/>`作为值, 如`<property name="email"><null/></property>`

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for javaCollection -->
   <bean id="javaCollection" class="com.tutorialspoint.JavaCollection">

   <!-- Passing bean reference  for java.util.List -->
         <property name="addressList">
            <list>
               <ref bean="address1"/>
               <ref bean="address2"/>
               <value>Pakistan</value>
            </list>
         </property>

      <!-- results in a setAddressList(java.util.List) call -->
      <property name="addressList">
         <list>
            <value>INDIA</value>
            <value>Pakistan</value>
            <value>USA</value>
            <value>USA</value>
         </list>
      </property>

      <!-- results in a setAddressSet(java.util.Set) call -->
      <property name="addressSet">
         <set>
            <value>INDIA</value>
            <value>Pakistan</value>
            <value>USA</value>
            <value>USA</value>
        </set>
      </property>

      <!-- results in a setAddressMap(java.util.Map) call -->
      <property name="addressMap">
         <map>
            <entry key="1" value="INDIA"/>
            <entry key="2" value="Pakistan"/>
            <entry key="3" value="USA"/>
            <entry key="4" value="USA"/>
         </map>
      </property>

      <!-- results in a setAddressProp(java.util.Properties) call -->
      <property name="addressProp">
         <props>
            <prop key="one">INDIA</prop>
            <prop key="two">Pakistan</prop>
            <prop key="three">USA</prop>
            <prop key="four">USA</prop>
         </props>
      </property>

   </bean>

</beans>
```

### 小结

* 注入Bean
    - `<bean>`
        - 属性: `<property name="属性名" value="属性值"/>`
        - 引用对象: `<property name="对象名" ref="对象bean标签的id"/>`
        - 构造方法: `<constructor-arg name="参数", value="参数值"/>`
        - setter: 同属性
* SpEL注入
    - `#{表达式}`
    - `<property name="username" value="#{'Tom'}"/>`
* 注入集合或数组
    - 数组, List集合: `<list></list>`
        - 普通元素: `<value></value>`
        - 对象元素: `<ref bean="对象id"/>`
    - Set集合: `<set></set>`
        - 普通元素: `<value></value>`
        - 对象元素: `<ref bean="对象id"/>`
    - Map集合: `<map></map>`
        - `<entry key="键" value="值"/>`
        - `<entry key-ref="键" value-ref="值"/>`
    - properties属性文件: `<property name="文件名"></property>`
        - `<props></props>`
            - `<prop key="键">值</prop`

```java
public class User {
    private String[] arr;
    public void setArr(String[] arr) {
        this.arr = arr;
    }
}
```

```xml
<bean id="user" class="xxx.User">
    <!-- 数组/集合的注入 -->
    <property name="arr">
        <list>
            <value>一</value>
            <value>二</value>
            <value>三</value>
            <!-- 如果元素是对象 -->
            <ref bean="引入对象的id"/>
        </list>
    </property>    
</bean>
```


## Bean的自动装配(自动连接, autowire)

* Spring可以对Bean进行自动装配. 主要是减少xml配置中的bean中ref属性的编写
* 自动装配的模式:
    - 无模式: 默认, 表示没有自动装配
    - `byName`: 按照属性的名称自动装配. 用于set方法DI
    - `byType`: 按照属性的类型自动装配. 用于set方法DI
    - `constructor`: 类似于byType, 但适用于构造函数DI
    - `autodetect`: 自动检测, 首先尝试通过构造函数来装配, 如果不行则尝试byType
* 自动装配的局限性:
    - 如果仍然存在显式的声明, 则依然会被覆盖
    - 不能自动装配基本类型, `String`和`Class`
    - 不如显式装配精确

### byName自动装配

* 由`属性的名称`指定自动装配, 即通过类中依赖的变量名称与xml中bean的`id`或`name`来匹配
* 用于set方法DI
* 在xml配置中的beans中将`auto-wire`属性设置为`byName`, 会将属性与配置文件中定义相同名称(`id`或`name`)的beans进行匹配和连接, 如果找到匹配项则注入bean, 否则会抛出异常


#### 示例

* 解释
    - `TextEditor`内部依赖`SpellChecker`, 且该依赖通过set方法传入
    - 普通方式下, xml配置文件中, 在声明`TextEditor`时内部需要声明`SpellChecker`属性, 并使用`ref`属性引用依赖的bean
    - 使用自动装配的`byName`方式后, 在声明`TextEditor`时只需要加入`autowire=byName`属性, 即可省略`spellChecker`属性的声明, Spring自动根据`TextEditor`类中的成员属性`spellChecker`在xml中寻找bean中`id`为`spellChecker`的这个bean来注入
* 示例

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <!-- 正常定义TextEditor方式, 内部有SpellChecker依赖, 要使用属性声明
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
       <property name="spellChecker" ref="spellChecker" />
       <property name="name" value="Generic Text Editor" />
   </bean>
    -->

   <!-- 使用autowire=byName进行属性自动装配, 免去声明依赖的属性 -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor" autowire="byName">
      <property name="name" value="Generic Text Editor" />
   </bean>

   <!-- 因为这个bean的id为spellChecker与TextEditor类中的依赖成员属性的名称相同, 所以注入这个bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"></bean>

</beans>
```

```java
TextEditor.java
---------------
package com.tutorialspoint;
public class TextEditor {
   private SpellChecker spellChecker;
   private String name;
   public void setSpellChecker( SpellChecker spellChecker ){
      this.spellChecker = spellChecker;
   }
   public SpellChecker getSpellChecker() {
      return spellChecker;
   }
   public void setName(String name) {
      this.name = name;
   }
   public String getName() {
      return name;
   }
   public void spellCheck() {
      spellChecker.checkSpelling();
   }
}
```

```java
SpellChecker.java
-----------------
package com.tutorialspoint;
public class SpellChecker {
   public SpellChecker() {
      System.out.println("Inside SpellChecker constructor." );
   }
   public void checkSpelling() {
      System.out.println("Inside checkSpelling." );
   }   
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
      TextEditor te = (TextEditor) context.getBean("textEditor");
      te.spellCheck();
   }
}
```

```bash
# 输出结果
Inside SpellChecker constructor.
Inside checkSpelling.
```

### byType自动装配

* 由`属性的类型`指定自动装配, 即使用类中依赖的类名与xml中bean的`class`匹配
* 用于set方法DI
* 在xml配置中将`autowire`属性设置为`byType`.

#### 示例

* 解释
    - `TextEditor`内部依赖`SpellChecker`, 且该依赖通过set方法传入
    - 普通方式下, xml配置文件中, 在声明`TextEditor`时内部需要声明`SpellChecker`属性, 并使用`ref`属性引用依赖的bean
    - 使用自动装配的`byType`方式后, 在声明`TextEditor`时只需要加入`autowire=byType`属性, 即可省略`spellChecker`属性的声明, Spring自动根据`TextEditor`类中的成员属性`spellChecker`的类`com.tutorialspoint.SpellChecker`, 在xml中寻找bean中`class`为`com.tutorialspoint.SpellChecker`的这个bean来注入
* 示例

```xml
applicationContext.xml
----------------------
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <!-- 正常定义TextEditor方式, 内部有SpellChecker依赖, 要使用属性声明
    <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker" ref="spellChecker" />
      <property name="name" value="Generic Text Editor" />
    </bean>
    -->

    <!-- 使用autowire=byType进行属性自动装配, 免去声明依赖的属性 -->
    <bean id="textEditor" class="com.tutorialspoint.TextEditor" autowire="byType">
        <property name="name" value="Generic Text Editor" />
    </bean>

    <!-- 因为这个bean的class为SpellChecker与TextEditor类中的依赖成员属性的类相同, 所以注入这个bean -->
    <bean id="SpellChecker" class="com.tutorialspoint.SpellChecker"></bean>

</beans>
```

```java
TextEditor.java
---------------
package com.tutorialspoint;
public class TextEditor {
   private SpellChecker spellChecker;
   private String name;
   public void setSpellChecker( SpellChecker spellChecker ) {
      this.spellChecker = spellChecker;
   }
   public SpellChecker getSpellChecker() {
      return spellChecker;
   }
   public void setName(String name) {
      this.name = name;
   }
   public String getName() {
      return name;
   }
   public void spellCheck() {
      spellChecker.checkSpelling();
   }
}
```

```java
SpellChecker.java
-----------------
package com.tutorialspoint;
public class SpellChecker {
   public SpellChecker(){
      System.out.println("Inside SpellChecker constructor." );
   }
   public void checkSpelling() {
      System.out.println("Inside checkSpelling." );
   }   
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context =
             new ClassPathXmlApplicationContext("applicationContext.xml");
      TextEditor te = (TextEditor) context.getBean("textEditor");
      te.spellCheck();
   }
}
```

```bash
# 输出结果
Inside SpellChecker constructor.
Inside checkSpelling.
```

### constructor自动装配

* 与byType类型类似
* 用于构造函数DI

#### 示例

* 解释
    - `TextEditor`内部依赖`SpellChecker`, 且该依赖通过构造方法传入
    - 普通方式下, xml配置文件中, 在声明`TextEditor`时内部需要声明`SpellChecker`属性, 并使用`ref`属性引用依赖的bean
    - 使用自动装配的`constructor`方式后, 在声明`TextEditor`时只需要加入`autowire=constructor`属性, 即可省略`spellChecker`属性的声明, Spring自动根据`TextEditor`类中的成员属性`spellChecker`的构造方法, 寻找对应的bean来注入
* 示例

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- 普通定义方式, 需要声明依赖spellChecker的构造函数
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <constructor-arg  ref="spellChecker" />
      <constructor-arg  value="Generic Text Editor"/>
   </bean>
   -->

   <!-- 由于使用autowire=constructor自动装配, 免去声明spellChecker构造函数, 会自动寻找对应的构造函数来注入 -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor" autowire="constructor">
      <constructor-arg value="Generic Text Editor"/>
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

```java
TextEditor.java
---------------
package com.tutorialspoint;
public class TextEditor {
   private SpellChecker spellChecker;
   private String name;
   public TextEditor( SpellChecker spellChecker, String name ) {
      this.spellChecker = spellChecker;
      this.name = name;
   }
   public SpellChecker getSpellChecker() {
      return spellChecker;
   }
   public String getName() {
      return name;
   }
   public void spellCheck() {
      spellChecker.checkSpelling();
   }
}
```

```java
SpellChecker.java
-----------------
package com.tutorialspoint;
public class SpellChecker {
   public SpellChecker(){
      System.out.println("Inside SpellChecker constructor." );
   }
   public void checkSpelling()
   {
      System.out.println("Inside checkSpelling." );
   }  
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context =
             new ClassPathXmlApplicationContext("applicationContext.xml");
      TextEditor te = (TextEditor) context.getBean("textEditor");
      te.spellCheck();
   }
}
```

```bash
# 输出结果
Inside SpellChecker constructor.
Inside checkSpelling.
```


## 基于注解的配置方式

* 注解方式更简单, 用于取代xml配置

### 开启注解方式

* 默认注解方式是关闭的, 需要在配置中开启
* 开启方式: (两者选一即可)
    - 方式1: `<context:annotation-config/>`
        - 开启注解配置功能, 同时注册了多个用于解析注解的解析器
    - 方式2: `<context:component-scan base-package="要扫描的包"/>`
        - 同方式1, 但注册了更多的解析器, 可以看做包含方式1

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <!-- 开启注解方式1 -->
    <context:annotation-config/>

    <!-- 开启注解方式2 -->
    <context:component-scan base-package="要扫描的包"/>

    <!-- 在开启注解的声明下方, 再定义bean -->

</beans>
```

### 使用注解

* 注解分类
    - 依赖注入注解:
        - `@Required`: 可用于bean依赖属性的set方法
        - `@Autowired`: 可用于bean依赖属性的set方法, 非set方法, 构造方法和属性
        - `@Qualifier`: 指定具体的bean
        - JSR-250 Annotations: Spring支持Java原生的注解
            - `@Resource`: 相当于`@Autowired`和`@Qualifier`一起使用
            - `@PostConstruct`: 相当于`init-method`
            - `@PreDestroy`: 相当于`destroy-method`
    - 配置注解:
        - `@Configuration`: 用在类上, 定义一个bean, 代替一个`<beans></beans>`
        - `@Bean`: 用在返回bean对象的方法上, 代替一个`<bean></bean>`

### @Required注解的使用方式

* `@Required`用在bean依赖的set方法上, 该依赖bean必须在xml中声明, 否则会抛出`BeanInitializationException`

#### 示例

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>

   <!-- Definition for student bean -->
   <bean id="student" class="com.tutorialspoint.Student">
      <property name="name"  value="Zara" />
      <!-- age属性使用了@Required注解, 必须声明 -->
      <property name="age"  value="11" />
   </bean>

</beans>
```

```java
Student.java
------------
package com.tutorialspoint;
import org.springframework.beans.factory.annotation.Required;
public class Student {
   private Integer age;
   private String name;
   @Required
   public void setAge(Integer age) {
      this.age = age;
   }
   public Integer getAge() {
      return age;
   }
   @Required
   public void setName(String name) {
      this.name = name;
   }
   public String getName() {
      return name;
   }
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
      Student student = (Student) context.getBean("student");
      System.out.println("Name : " + student.getName() );
      System.out.println("Age : " + student.getAge() );
   }
}
```

```bash
# 输出结果
Name : Zara
Age : 11
```

### @Autowired注解的使用

* `@Autowired`提供更多自动装配的设置
* `@Autowired`可用于多个地方:
    - 在set方法上使用: 免去在xml声明`<property>`元素, 会执行`byType`自动装配
    - 在构造方法上使用: 免去在xml中声明`<constructor-arg/>`, 也能通过构造方法自动装配
    - 在属性上使用: 可以免去编写依赖的set方法和构造方法, 并且不用在xml中声明依赖的属性
* `@Autowired(required=false)`: 相当于同时使用`@Autowired`和`@Required`, 表示仍然需要在xml中声明依赖的bean

#### 示例

* 用在set方法上

```java
package com.tutorialspoint;
import org.springframework.beans.factory.annotation.Autowired;
public class TextEditor {
   private SpellChecker spellChecker;
   @Autowired
   public void setSpellChecker( SpellChecker spellChecker ){
      this.spellChecker = spellChecker;
   }
   public SpellChecker getSpellChecker( ) {
      return spellChecker;
   }
   public void spellCheck() {
      spellChecker.checkSpelling();
   }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>

   <!-- 不再需要property的set方法  -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"></bean>

</beans>
```

* 用在构造方法上

```java
package com.tutorialspoint;
import org.springframework.beans.factory.annotation.Autowired;
public class TextEditor {
   private SpellChecker spellChecker;
   @Autowired
   public TextEditor(SpellChecker spellChecker){
      System.out.println("Inside TextEditor constructor." );
      this.spellChecker = spellChecker;
   }
   public void spellCheck(){
      spellChecker.checkSpelling();
   }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>

   <!-- 不再需要constructor-arg  -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

* 用在属性上

```java
package com.tutorialspoint;
import org.springframework.beans.factory.annotation.Autowired;
public class TextEditor {
   @Autowired
   private SpellChecker spellChecker;
   public TextEditor() {
      System.out.println("Inside TextEditor constructor." );
   }  
   public SpellChecker getSpellChecker( ){
      return spellChecker;
   }  
   public void spellCheck(){
      spellChecker.checkSpelling();
   }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>

   <!-- property的set方法和构造方法都不要 -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

### @Qualifier注解的使用

* 相当于`@Autowired`和`@Required`一起使用
* 当需要创建多个具有相同类型的bean时, 如果只想用一个依赖为他们中的一个进行装配, 则可以使用`@Qualifier("{beanId}")`来指定具体使用哪个bean

#### 示例

* 解释
    - 对于同一个依赖`Student`, 在xml中配置了两种初始化方式, 属性的初始值不同
    - 在`Profile`类中使用`@Qualifier("student1")`指定了使用xml中id为`student1`的bean作为初始化

```xml
applicationContext.xml
---------
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>

   <!-- Definition for profile bean -->
   <bean id="profile" class="com.tutorialspoint.Profile">
   </bean>

   <!-- Definition for student1 bean -->
   <bean id="student1" class="com.tutorialspoint.Student">
      <property name="name"  value="Zara" />
      <property name="age"  value="11"/>
   </bean>

   <!-- Definition for student2 bean -->
   <bean id="student2" class="com.tutorialspoint.Student">
      <property name="name"  value="Nuha" />
      <property name="age"  value="2"/>
   </bean>

</beans>
```

```java
Student.java
------------
package com.tutorialspoint;
public class Student {
   private Integer age;
   private String name;
   public void setAge(Integer age) {
      this.age = age;
   }   
   public Integer getAge() {
      return age;
   }
   public void setName(String name) {
      this.name = name;
   }  
   public String getName() {
      return name;
   }
}
```

```java
Profile.java
------------
package com.tutorialspoint;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
public class Profile {
   @Autowired
   @Qualifier("student1")
   private Student student;
   public Profile(){
      System.out.println("Inside Profile constructor." );
   }
   public void printAge() {
      System.out.println("Age : " + student.getAge() );
   }
   public void printName() {
      System.out.println("Name : " + student.getName() );
   }
}
```

```java
MainApp.java
------------
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
      Profile profile = (Profile) context.getBean("profile");
      profile.printAge();
      profile.printName();
   }
}
```

```bash
# 输出结果
Inside Profile constructor.
Age : 11
Name : Zara
```

### @Configuration和@Bean注解的使用

* `@Configuration`: 用在类上, 代替xml中的`<beans></beans>`
* `@Bean`: 用在返回bean对象的方法上, 代替xml中的`<bean></bean>`
    - 声明生命周期方法: `@Bean(initMethod="{初始化方法名}", destroyMethod="{销毁方法名}")`
* `@Import({Class})`: 从另一个配置中加载Bean的定义
* 需要使用`AnnotationConfigApplicationContext`来创建bean

#### 示例

* java注解的配置

```java
HelloWorldConfig.java
---------------------
package com.tutorialspoint;
import org.springframework.context.annotation.*;
@Configuration
public class HelloWorldConfig {
   @Bean
   public HelloWorld helloWorld(){
      return new HelloWorld();
   }
}
```

* 等同于xml中的配置

```xml
applicationContext.xml
---------
<beans>
   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld" />
</beans>
```

* 创建bean

```java
public static void main(String[] args) {
    /*
    // 方式1: 空参创建ApplicationContext, 然后注册配置
    AnnotationConfigApplicationContext ctx =
       new AnnotationConfigApplicationContext();
       ctx.register(AppConfig.class, OtherConfig.class);
       ctx.register(AdditionalConfig.class);
       ctx.refresh();
    */
    // 方式2: 直接指定配置类
    ApplicationContext ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class);
    // 创建bean
    HelloWorld helloWorld = ctx.getBean(HelloWorld.class);
    // 调用方法
    helloWorld.setMessage("Hello World!");
    helloWorld.getMessage();
}
```


### 小结

* 注解方式更简单, 用于取代xml配置
* 步骤:
    - 导入必要包, 包括`spring-aop-xxx.jar`
    - 在`applicationContext.xml`中开启注解扫描:
        - `<context:component-scan base-package="要扫描的包"/>`
    - 在类中
        - 类的注解: `@Component(value="别名id")`, 相当于`<bean id="别名id" class>`
* 注解
    - 组件(类)注入的注解
        - `@Component`. 有3个衍生注解, 目前作用一致, 后续会增强
            - `@Controller`: 作用在Web层
            - `@Service`: 作用在业务层
            - `@Repository`: 作用在持久层
        - `@Scope(value="作用类型")`: 设置bean的作用范围
    - 属性注入的注解, 可以不提供setter
        - `@Value(value="初始化值")`: 基本数据类型和字符串属性注入
        - `@Autowired`: 引用类型(对象). 按类型自动注入
            - `@Qualifier(value="id名称")`: 设置为按id名称注入
        - `@Resource(name="id名称")`: Java提供的注解, 相当于`@Autowired`和`@Qualifier`一起使用
    - 生命周期的注解
        - `@PostConstruct`: 相当于`init-method`
        - `@PreDestroy`: 相当于`destroy-method`


```xml
<beans>
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="要扫描的包"/>
</beans>
```

```java
@Component
public class UserServiceImpl implements UserService {

    public void hello() {
        // ..
    }
}
```

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService us = (UserService) ac.getBean("userService");
us.save();
```

## Spring Context事件回调

* `ApplicationContext`负责bean的完整生命周期, 有一系列的事件回调
* 设置回调监听器
    - 定义类实现`ApplicationListener<事件类型>`接口, 重写`public void onApplicationEvent(ContextStartedEvent event)`方法
        - 可以定义多个类各自实现一个事件接口, 或者一个类实现多个事件接口
    - 在xml配置文件中声明: `<bean id="cStartEventHandler" class="com.tutorialspoint.CStartEventHandler"/>`
* 回调事件
    - `ContextRefreshedEvent`: `ApplicationContext` 被初始化或刷新时, 该事件被发布. 这也可以在 `ConfigurableApplicationContext` 接口中使用 `refresh()` 方法来发生
    - `ContextStartedEvent`: 当使用 `ConfigurableApplicationContext` 接口中的 `start()` 方法启动 `ApplicationContext` 时, 该事件被发布. 你可以调查你的数据库, 或者你可以在接受到这个事件后重启任何停止的应用程序
    - `ContextStoppedEvent`: 当使用 `ConfigurableApplicationContext` 接口中的 `stop()` 方法停止 `ApplicationContext` 时, 发布这个事件. 你可以在接受到这个事件后做必要的清理的工作
    - `ContextClosedEvent`: 当使用 `ConfigurableApplicationContext` 接口中的 `close()` 方法关闭 `ApplicationContext` 时, 该事件被发布. 一个已关闭的上下文到达生命周期末端; 它不能被刷新或重启
    - `RequestHandledEvent`: web特有事件, 告诉所有bean的HTTP请求已经完成

### 自定义事件

* 步骤:
    - 定义事件类, 继承`ApplicationEvent`
    - 定义事件发布类, 实现`ApplicationEventPublisherAware`接口, 重写方法
    - 在xml配置文件中声明事件发布类: `<bean id="customEventPublisher" class="com.tutorialspoint.CustomEventPublisher"/>`

```java
package com.tutorialspoint;
import org.springframework.context.ApplicationEvent;
public class CustomEvent extends ApplicationEvent{
   public CustomEvent(Object source) {
      super(source);
   }
   public String toString(){
      return "My Custom Event";
   }
}
```

```java
package com.tutorialspoint;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
public class CustomEventPublisher
   implements ApplicationEventPublisherAware {
   private ApplicationEventPublisher publisher;
   public void setApplicationEventPublisher
              (ApplicationEventPublisher publisher){
      this.publisher = publisher;
   }
   public void publish() {
      CustomEvent ce = new CustomEvent(this);
      publisher.publishEvent(ce);
   }
}
```








## AOP

* AOP, Aspect oriented programming, 面向切面编程, 目的是实现模块化编程, 取代传统纵向继承的重复性代码. 是OOP的延续
* OOP的最小单元是类, AOP的最小单元是切面
* 目的: 在不改变原代码的情况下, 增加功能
* 原理: 通过预编译方式, 在运行期间动态代理实现程序功能

### AOP的实现原理: 动态代理

* 动态代理有2种实现方式:
    1. 使用JDK的`Proxy`类生成代理对象
    2. 使用cglib方式进行动态代理

```java
// 使用JDK的`Proxy`类生成代理对象
public static UserDao getProxy(final UserDao dao) {
    UserDao proxy = (UserDao) Proxy.newProxyInstance(dao.getClass().getClassLoader(), dao.getClass().getInterfaces(), new InvocationHandler() {
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // 代理代码
            if("save".equals(method.getName())) {
                System.out.println("...");
            }
            return method.invoke(dao, args);
        }
    });
    return proxy;
}
```


### AOP术语

这些并不是Spring特有的术语, 而是AOP的相关概念

* `Aspect`: 切面, 一个切面具有一组提供横切需求的API
* `Join point`: 连接点. 指要被拦截到的点, 在spring中这些点指的是方法, 因为spring只支持方法类型的连接点
* `Pointcut`: 切入点. 一组一个或多个连接点, 通知在这里执行. 指我们要对那些Join point进行拦截的定义
* `Advice`: 通知. 指拦截到Join point之前或之后所要做的事情就是通知.
    - Spring提供5种通知:
        - `前置通知`: 在目标类的方法执行之前执行
        - `后置通知`: 在目标类的方法执行之后执行
        - `异常通知(抛出异常后通知)`: 在抛出异常后通知
        - `最终通知(返回后通知)`: 在目标类的方法执行后执行, 即使之前出现了异常
        - `环绕通知`: 方法执行的前, 后都执行
* `Introduction`: 引介. 一种特殊的通知, 在不修改代码的前提下, Introduction可以在运行期为类动态地添加一些方法或属性
* `Target object`: 目标对象. 代理的目标对象
* `Weaving`: 织入. 指把通知应用到目标对象来创建新的代理对象的过程
* `Proxy`: 代理. 一个类被AOP织入通知后, 就产生一个结果代理类
* `Aspect`: 切面. 切入点和通知的结合


### 自定义切面

* 自定义切面有2种方式:
    1. xml配置
    2. `@AspectJ`注解

### xml配置自定义切面

可以在xml配置中配置AOP

#### 切入点表达式

* 格式: `execution()`
    - 权限修饰符可以省略
    - 返回值必须写, 可以使用`*`进行通配
    - 包, 也可以写`*`通配一层, 使用`*..*`通配多层
    - 类, 也可以使用`*`通配, 一般`*DaoImpl`
    - 方法, 可以使用`*`通配, 如`save*()`
    - 参数, 可以使用`..`通配

#### 通知类型

* 前置通知
    - 在目标类的方法执行之前执行
    - 配置: `<aop:before method="切面方法名" pointcut-ref="切入点id"/>`
    - 场景: 校验方法参数
* 后置通知
    - 在目标类的方法执行之后执行
    - 配置: `<aop:after-returning method="切面方法名" pointcut-ref="切入点id"/>`
    - 场景: 修改方法返回值
* 异常通知
    - 在抛出异常后通知
    - 配置: `<aop:after-throwing method="切面方法名" pointcut-ref="切入点id"/>`
    - 场景: 封装异常信息
* 最终通知
    - 在目标类的方法执行后执行, 即使之前出现了异常
    - 配置: `<aop:after method="切面方法名" prointcut-ref="切入点id"/>`
    - 场景: 释放资源
* 环绕通知
    - 方法执行的前, 后都执行
    - 配置: `<aop:around method="切面方法名" pointcut-ref="切入点id"/>`
    - 注意: 目标的方法默认不执行, 需要使用`ProceedingJoinPoint point`参数来让目标对象的方法执行

#### 使用步骤

* 步骤
    - 导入依赖包: spring-aop
    - 创建要被切入的类
    - 创建切面类, 定义方法作为advice
    - 创建`/src/applicationContext.xml`
        - 引入`aop`schema
            - `xmlns:aop="http://www.springframework.org/schema/aop"`
            - `http://www.springframework.org/schema/aop`
            - `http://www.springframework.org/schema/aop/spring-aop-3.0.xsd`
        - 声明`切面类`和`要被切入类`的bean
        - 配置aop
            - `<aop:config></aop:config>`
                - 配置切面类: `<aop:aspect ref="{切面类id}"></aop:aspect>`
                - 配置切入和通知的2种方式:
                    - 方式1, 分别配置切入点和通知:
                        - 配置切入点: `<aop:pointcut id="userDao" expression="execution(public User xxx.yyy.zzz())"/>`
                        - 配置前置通知: `<aop:before method="{切面方法名}" pointcut-ref="userDao"/>`
                    - 方式2, 在通知中配置切入点:
                        - 配置前置通知: `<aop:before method="{切面方法名}" pointcut="execution(public User xxx.yyy.zzz())"/>`

#### 示例

* 业务类, 要被切入的类

```java
UserDao.java
---------------
public class UserDao {

    public User getUser() {
        // 将该方法作为切入点
    }
}
```

* 切面类, 要切入的增强功能

```java
LogAspect.java
--------------------
public class LogAspect {
    public void log() {
        // 通知方法
    }
}
```

* 配置aop

```xml
applicationContext.xml
----------------------
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">

    <!-- 配置要被切入的类 -->
    <bean id="userDao" class="xxx.UserDao"/>

    <!-- 配置切面类 -->
    <bean id="logAspect" class="xxx.LogAspect"/>

    <!-- 配置aop -->
    <aop:config>
        <!-- 配置切面类 -->
        <aop:aspect id="myAspect" ref="logAspect">
            <!-- 配置切入点, 即要执行通知的位置. 这里将UserDao的getUser()方法作为切入点 -->
            <aop:pointcut id="userDao" expression="execution(public User xxx.UserDao.getUser())"/>
            <!-- 配置前置通知, 即在getUser方法的最前执行切面的log方法 -->
            <aop:before method="log" pointcut-ref="userDao"/>
        </aop:aspect>
    </aop:config>
</beans>
```


### @AspectJ 注解配置方式

也可以通过注解的方式配置AOP

#### 开启aop注解功能

* 在xml中添加配置: `<aop:aspectj-autoproxy/>`

#### 相关注解

* 配置切面
    - `@Aspect`: 定义类为切面类
* 配置切面
    - `@Pointcut(value="切入表达式")`: 定义方法作为切入点方法
    - `@Before("切入点方法名")`: 前置通知
    - `@After("切入点方法名")`: 后置通知
    - `@AfterRunning("切入点方法名")`: 返回后通知
    - `@AfterThrowing("切入点方法名")`: 抛出异常后通知
    - `@Around("切入点方法名")`: 环绕通知

#### 使用步骤

* 步骤
    - 引入必要包: aspectjxxx
    - 创建`src/applicationContext.xml`
        - 开启注解自动代理: `<aop:aspectj-autoproxy/>`
        - 声明bean
    - 创建切面类
        - 使用`@Aspect`注解标记类作为切面类
        - 使用`@Pointcut("切入点表达式")`注解切入点方法, 本方法为空实现, 表达式指向要切入的实际方法
        - 使用`@Before("切入点方法名")`注解标记方法, 作为前置通知
        - 使用`@After("切入点方法名")`注解标记方法, 作为后置通知
        - 使用`@AfterReturning("切入点方法名")`注解标记方法, 作为返回后通知
        - 使用`@AfterThrowing("切入点方法名")`注解标记方法, 作为抛出异常后通知
    - 创建被切入的类

#### 示例

```java
public class UserDaoImpl implements UserDao {

    public void save() {
        // 要被切入的方法
    }
}
```

```java
切面类
-----
@Aspect
public class LogAspect {

    //定义一个切入点方法, 该方法指示要切入UserDaoImpl的save()方法
    @Pointcut(value="execution(public * *.UserDaoImpl.save())")
    public void commonPointcut(){}

    @Before(value="commonPointcut()") // 指向切入点方法
    public void log() {
        // 前置通知方法
    }
}
```

```xml
<beans>
    <!-- 开启使用注解进行aop -->
    <aop:aspectj-autoproxy/>

    <!-- 配置目标对象 -->
    <bean id="userDao" class="xxx.UserDaoImpl"/>

    <!-- 配置切面类 -->
    <bean id="logAspect" class="xxx.LogAspect"/>

</beans>
```


## JDBC模板

### JdbcTemplate类

* Spring提供了内置的连接池, 也可以整合其他连接池

```xml
applicationContext.xml
----------------------
<!-- 配置连接池 -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql:///db_url"/>
    <property name="username" value="root"/>
    <property name="password" value="123445"/>
</bean>

<!-- 配置JDBC的模板类 -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

```java
// 创建mysql连接池
DriverManagerDataSource source = new DriverManagerDataSource();
source.setDriverClassName("com.mysql.jdbc.Driver");
source.setUrl("jdbc:mysql:///db_url");
source.setUsername("root");
source.setPassword("123456");

//创建模板类
JdbcTemplate t = new JdbcTemplate();
// 设置连接池
t.setDataSource(source);
// 操作
t.update("insert into t_account values (null, ?, ?)", "哈哈", 10000);
```

### 集成第三方连接池

* DBCP连接池
    - 引入jar包
        - `com.springsource.org.apache.commons.dbcp-x.x.x.jar`
        - `com.springsource.org.apache.commons.pool-x.x.x.jar`
    - 编写配置文件

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql:///db_url"/>
    <property name="username" value="root"/>
    <property name="password" value="1234"/>
</bean>
```

* C3P0连接池
    - 引入jar
        - `com.springsource.com.mchange.v2.c3p0.xxx.jar`
    - 编写配置文件

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql:///db_url"/>
    <property name="user" value="root"/>
    <property name="password" value="1234"/>
</bean>
```

### 使用模板类

* `update(String sql, Object... params)`: 增删改
* `queryForObject(String sql, RowMapper<T> mapper, Object... params)`: 查
* `query(String sql, RowMapper mapper, List<T> list)`: 查

```java
// 增
jdbcTemplate.update("insert into table values (null, ?, ?)", "Tom", 100);
// 改
jdbcTemplate.update("update table set name = ? where id = ?", "Tom", 100);
// 删
jdbcTemplate.update("delete from table where name = ? and id = ?", "Tom", 100);

// 查
class BeanMapper implements RowMapper<User> {
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setName(rs.getString("name"));
        user.setAge(rs.getInt("age"));
        return user;
    }
}
User user = jdbcTemplate.queryForObject("select * from t_user where age = ?", new BeanMapper(), 1);
List<User> users = jdbcTemplate.query("select * from t_user", new BeanMapper());
```


## 事务管理

* `PlatformTransactionManager`接口: 平台事务管理器, 有实现类
    - 实现类
        - `DataSrouceTransactionManager`: SpringJDBC模板类, MyBatis
        - `HibernateTransactionManager`: Hibernate
    - 方法
        - `void commit(TransactionStatus status)`
        - `TransactionStatus getTransaction(TransactionDefinition definition)`
        - `void rollback(TransactionStatus status)`
* `TransactionDefinition`接口: 事务定义信息(事务的隔离级别, 传播行为, 超时, 只读)
    - 事务隔离级别常量
        - `ISOLATION_DEFAULT`
        - `ISOLATION_READ_UNCOMMITTED`
        - `ISOLATION_READ_COMMITTED`
        - `ISOLATION_REPEATABLE_READ`
        - `ISOLATION_SERIALIZABLE`
    - 事务传播行为常量(一般不设置, 使用默认值)
        - `PROPAGATION_REQUIRED`: 默认. A中有事务, 使用A中的事务, 如果没有, B就会开启一个新的事务, 将A包含进来(即保证A, B在同一个事务中)
        - `PROPAGATION_SUPPORTS`: A中有事务, 使用A中的事务, 如果没有, 那么B也不使用事务
        - `PROPAGATION_MANDATORY`: A中有事务, 使用A中的事务, 如果没有, 抛出异常
        - `PROPAGATION_REQUIRES_NEW`: A中有事务, 将A的事务挂起, B创建一个新的事务(即保证A, B不在一个事务中)
        - `PROPAGATION_NOT_SUPPORTED`: A中有事务, 将A中的事务挂起
        - `PROPAGATION_NEVER`: A中有事务, 抛出异常
        - `PROPAGATION_NESTED`: 嵌套事务. 当A执行之后, 就会在这个位置设置一个保存点, 如果B没有问题, 执行通过, 如果B出现异常, 运行客户根据需求回滚(选择回滚到保存点或者最初状态)
* `TransactionStatus`接口: 事务的装填

### 事务配置

#### xml配置方式

* 代码实现
* aop声明式(推荐)

```xml
<!-- 声明式事务 -->
<!-- 配置通知 -->
<tx:advice id="myAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="方法名" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
<!-- 配置AOP -->
<aop:config>
    <!-- spring提供的通知 -->
    <aop:advisor advice-ref="myAdvice" pointcut="execution(public * com.xxx.pay())"/>
</aop:config>
```

#### 注解方式

步骤:
1. xml中开启注解
2. 类上加注解: `@Transactional`

```xml
<!-- 开启事务注解 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

```java
@Transactional
public class UserServiceImpl implements UserService {

}
```


## 配置log4j

* log4j
    - 配置: `src/log4j.properties`
    - 使用: `Logger.getClass(Xxx.class).info("aaa");`


## 配置JUnit单元测试

* 作用: 免去创建工厂, 加载配置文件, getBean()
* 步骤
    1. 引入`spring-test-xxx.jar`
    2. 通过注解实现
        - 类上
            - 设置JUnit测试: `@RunWith(SpringJUnit4ClassRunner.class)`
            - 设置配置文件: `@contextConfiguration("classpath:applicationContext")`
        - 成员
            - 引入依赖: `@Resource(name=userService)`
        - 方法
            - 执行测试: `@Test`

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class Demo {
    @Resource(name="userService")
    private UserService userService;

    @Test
    public void testHello() {
        // 不用再获取工厂加载配置文件getBean()了
        userService.hello();
    }
}
```


## Spring Struts整合

* 解决重复加载配置文件的问题
    - 引入`spring-web-x.x.x.jar`包
    - 配置`ContextLoaderListener`监听器, 加载配置文件
    - 使用`WebApplicationContext`创建工厂

```xml
<!-- 配置Spring的监听器 -->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

```java
// 使用WebApplicationContext
WebApplicationContext wac = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
UserService us = (UserService) wac.getBean("userService");
userService.save();
```
