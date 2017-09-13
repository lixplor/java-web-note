# Nginx

* 反向代理服务器
* 负载均衡
* 动静分离


## 反向代理

* 正向代理: 用户的代理服务器, 发出用户请求
* 反向代理: 以代理服务器来接收Internet上的连接请求, 然后将请求转发给内部网络上的服务器, 并将从内部服务器得到的结果返回给Internet上请求连接的客户端, 此时代理服务器对外就表现为一个服务器

## 负载均衡

Load balance, 建立在现有网络结构之上, 提供一种廉价, 有效, 透明的方法来扩展网络设备和服务器的带宽, 增加吞吐量, 加强网络数据处理能力, 提高网络的灵活性和可用性. 原理是将数据流量分摊到多个服务器上执行, 减轻每台服务器的眼里, 多台服务器功能完成工作任务, 从而提高了数据的吞吐量

## 动静分离

动态内容(jsp)和静态内容(html, 图片, 文件)分离, 静态内容存储在CDN服务器上, 加快访问加载速度


## 安装和配置

* 官网: [https://www.nginx.com](https://www.nginx.com)

### 配置

* `nginx/conf/nginx.conf`

```shell
# 配置反向代理中上游服务器列表(名字自己取, lb=load balance)
upstream server_lb {
    # 格式: server IP:端口号 weight=权重值;
    server localhost:8080 weight=5;  
    server localhost:8081 weight=5;
    #ip_hash;  # 配置同一个IP只会访问同一个服务器, 可用于session共享
}

# 配置Nginx服务器
server {
    # 默认端口号
    listen       80;
    # 服务器名称
    server_name  localhost;

    # 路由配置(/表示当访问根路径时的配置)
    location / {
        root        html;
        # 反向代理服务器URL
        proxy pass  http://server_lb;
        index       index.html index.htm;
    }

    # Nginx默认的错误处理页面
    error_page 500 502 503 504  /50x.html;
    location = /50x.html {
        root html;
    }
}
```


## 搭建集群

* 利用Nginx转发请求到多个Tomcat实例


## 集群架构的Session共享

* 用户登录后的状态应该不会因为集群服务器不同而导致Session不同, 所以需要在各个服务器中都使用相同的session
* 共享session的几种方式:
    - 同一个session只访问同一个服务器
        - 在nginx配置文件中的`upstream`中, 添加`ip_hash;`选项, 这样同一个IP只会访问同一个服务器
    - redis存储session
    - tomcat广播机制
        - 修改tomcat的`server.xml`, 打开集群配置`<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>`
        - 修改项目的`web.xml`, 添加`<distributable/>`标签
        - 在此标签中, 只要`<Membership>`中有相同的`address`和`port`, 则会被分配到同一个集群中, 共享session.
        - 同一个session被称为一个cluster, 可以有多个cluster
