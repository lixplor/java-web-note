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
# 服务器列表(lb=load balance)
upstream server_lb {
    server localhost:8080;
    server localhost:8081;
}

server {
    listen       80;
    server_name  localhost;

    location / {
        root        html;
        proxy pass  http://server_lb;
        index       index.html index.htm;
    }

    error_page 500 502 503 504  /50x.html;
    location = /50x.html {
        root html;
    }
}
```

* `upstream`
* `weight`

## 搭建集群

* 利用Nginx转发请求到多个Tomcat实例
* 开启Tomcat的Cluster配置
