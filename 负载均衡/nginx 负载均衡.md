nginx 负载均衡

用官网的话说，它充当着网络流中“交通指挥官”的角色，
 - “站在”服务器前处理所有服务器端和客户端之间的请求，从而最大程度地提高响应速率和容量利用率，同时确保任何服务器都没有超负荷工作。
 - 如果单个服务器出现故障，负载均衡的方法会将流量重定向到其余的集群服务器，以保证服务的稳定性。
 - 当新的服务器添加到服务器组后，也可通过负载均衡的方法使其开始自动处理客户端发来的请求。


nginx

- 正向代理： 代转发请求。隐藏了客户端的信息，服务器不知道真正发出请求的是谁
- 反向代理  来自不同客户端的请求实际上都发到了代理服务器，再由代理服务器按照规则把请求分发给各个服务器
代理的是服务端，隐藏了服务器的信息，在反向代理的过程中，客户端并不知道具体是哪台服务器处理了自己的请求。

负载均衡常用算法
1. 轮询
2. 加权轮询
3. IP哈希 可以保证同ip的请求发到相同的服务器，或者相同hash的不同ip映射到一个服务器，在一定程度上
  - 解决集群部署环境下Session不共享的问题
  - 文件分片上传再合并的问题。
  - 还可以用于灰度发布，将一部分ip下的请求转发到部署新服务的服务器，另一部分不变。
4. 其他
  - url 哈希
  - 最小连接数  新请求，分发给连接数最小的一台服务器


## nginx 配置

- 轮询服务器设置

```
upstream api-server { # 服务器集群
  server localhost:8000 weight=10;
  server localhost:8001 weight=20;
  server localhost:8003 backup; # 默认是1  backup是标记为备用服务器，只有其他的主服务器都不可用时才启用
}
```

- 代理地址

```
location /api {
  root html;
  index index.html index.htm;
  proxy_pass http://api-server; # 指向自定义的服务器集群, 此处也可以是https
}
```


### 最少连接负载均衡

```
upstream api-server { # 服务器集群
  least_conn;
  server localhost:8000;
  server localhost:8001;
  server localhost:8003； 
}
```


### IP Hash

不支持权重设置，如果某台被永久移除，需要在后面加down来标记

```
upstream api-server { # 服务器集群 
  ip_hash;
  server localhost:8000;
  server localhost:8001 down; # down 表示这个server暂时不参与负载
  server localhost:8003； 
}
```

### fair  第三方

按照后端服务器的响应时间来分配请求

```
upstream api-server { # 服务器集群 
  server localhost:8000;
  server localhost:8001;
  server localhost:8003;
  fair;
}
```