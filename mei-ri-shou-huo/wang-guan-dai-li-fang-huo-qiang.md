---
description: 源于两次使用CLB时遇到的坑。
---

# 网关、代理、防火墙

CLB/SLB，即云环境-负载均衡。作为流行云原生的2021年，经常接触云环境的各个组件产品。借实际使用中踩坑的例子，深入了解CLB产品。

## 踩坑CLB

1.长链接问题：CLB的http监听代理，默认设置30s超时时间。若需要使用长链接请求，选择tcp监听代理。

2.源IP会话保持：CLB的tcp监听代理，设置【源IP会话保持】监听策略，无法保持会话的源IP不变。若需要使用【源IP会话保持】，选择http监听代理。

## CLB实现原理

CLB作为云环境中计算资源的负载均衡产品，是反向代理的一种实现。他分为四层监听（传输层，tcp、udp协议），七层监听（应用层，http、https协议）。

四层监听，通过 LVS（Linux Virtual Server） + keepalived实现。

七层监听，通过 nginx等实现反向代理。

### CLB架构

（补充架构图）

* LVS集群：
* nginx集群：
* Key server：

### 四层监听 vs 七层监听



## 代理

### 正向代理

正向代理，是中介，是客户端访问所有服务的中介，是与正向代理关联的客户端访问Server的中介/转发者。（客户端代理）

> From Wikipedia
>
> a forward proxy, which is an intermediary for its associated clients to contact any server.
>
> a reverse proxy is an intermediary for its associated servers to be contacted by any client.

![&#x6B63;&#x5411;&#x4EE3;&#x7406;&#x56FE;&#x4F8B;](../.gitbook/assets/zheng-xiang-dai-li-.png)

正向代理图例说明：Client1指定访问Server1，并指定正向代理帮助Client1转发访问Server1。

#### 正向代理适用场景：

服务器处于内网，且无公网ip，希望与外网进行http/https通信，只能通过nat设备（SNAT）或proxy两种方式。（nat服务器有网段限制）

#### 代理服务器（正向）例子：

客户端要访问Internet，需要通过代理服务器访问。即，客户端只能通过访问局域网内，甚至是同服务器中的nginx转发访问Internet。

客户端通过HTTP访问代理，代理通过监听不同的端口，分别处理HTTP和HTTPs请求。

```bash
server {
    resolver 114.114.114.114; #指定DNS服务器
    listen 80;
    location / {
        proxy_pass http://$host$request_uri; #代理服务器的协议（http）和请求地址
        proxy_set_header HOST $host;
        proxy_buffers 256 4k;
        proxy_max_temp_file_size 0k;
        proxy_connect_timeout 30;
        proxy_send_timeout 60;
        proxy_read_timeout 60;
        proxy_next_upstream error timeout invalid_header http_502;
    }
}
server {
    resolver 114.114.114.114; # 指定DNS服务器
    listen 443;
    location / {
        proxy_pass httpss://$host$request_uri; #代理服务器的协议（https）和请求地址
        proxy_buffers 256 4k;
        proxy_max_temp_file_size 0k;
        proxy_connect_timeout 30;
        proxy_send_timeout 60;
        proxy_read_timeout 60;
        proxy_next_upstream error timeout invalid_header http_502;
    }
}
```

#### 测试正向代理服务器：

```bash
curl -I --proxy 192.168.0.1:80 www.baidu.com

curl -I --proxy 192.168.0.1:443 www.baidu.com

# 全局代理
vi /etc/profile
export http_proxy='192.168.0.1:80'
export https_proxy='192.168.0.1:443'
export ftp_proxy='192.168.0.1:80'

curl -I www.baidu.com:80
curl --head www.baidu.com:443
```

> 使用正向代理服务器时的全局代理问题：
>
> /etc/hosts文件设置的域名解析ip不生效。因为代理的域名解析由代理服务器配置。

### 反向代理

反向代理，是中介，是服务端被所有客服端访问的中介，是与反向代理关联的Server被所有client访问的中介/转发者。（服务端代理）

#### 反向代理适用场景

例如反向代理了三个Server，只有客户端访问这三个Server，代理服务器才会进行代理。

#### 代理服务器（反向）例子：

反向代理服务器一般架设在内网入口，实现动静分离、负载均衡等。

```bash
upstream {
    server 192.168.0.1 weight=1 max_fails=1 fail_timeout=30;
    server 192.168.0.2 down;
}
server {
    listen       10000;
    server_name  localhost;
    #拦截请求Server
    location /{
        proxy_pass http://webserver;
    }
    
    #拦截静态资源
    location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
        root /Usr/static;
    }
}
```



F5









### 虚拟私有网络

Virtual Private Network\(VPN\)

代理模式

正向代理

隧道模式



网关



防火墙



