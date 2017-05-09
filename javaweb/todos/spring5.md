# Spring 5.0

* Spring 5.0 M1 引入了Reactive方式的函数式Web框架, 极大的简化了代码的编写和配置.


## Spring 5 代码示例

* Spring 5 的函数式编程示例, 实现了Controller使用非阻塞方式从远程服务器获取数据并返回

```java
@GetMapping("/accounts/{id}/alerts")
public Flux<Alert> getAccountAlerts(@PathVariable Long id) {

  return this.repository.getAccount(id)
      .flatMap(account ->
          this.webClient
              .perform(get("/alerts/{key}", account.getKey()))
              .extract(bodyStream(Alert.class)));
}
```


## 什么是Reactive编程方式

* Reactive编程方式的特点是非阻塞, 事件驱动, 应用使用了小数量的带有背压的线程作为关键因素来确保生产者不会压倒消费者. 
* Reactive Streams标准(在Java 9中被采用)使得不同层次和库之间得以沟通. 


## Spring Web Reactive 和 Spring Web MVC的区别

* Spring 5中
	* Spring MVC仍然用于Servelt容器, 它可以运行在任何的Servlet 3.1栈中, 包括JavaEE 7的服务器Tomcat, Jetty. 
	* Spring Web Reactive除了能够运行在Servlet 3.1容器外, 还支持Undertow, Netty等非Servlet容器