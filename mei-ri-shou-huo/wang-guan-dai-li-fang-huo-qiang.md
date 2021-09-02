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

正向代理：

反向代理：

F5



正向代理例子：

客户端要访问Internet，则要通过代理服务器访问。即，客户端只能通过访问局域网内，甚至是同服务器中的nginx转发访问Internet。

客户端通过HTTP访问代理，代理通过监听不同的端口，分别处理HTTP和HTTPs请求。

```bash
server {
    resolver 114.114.114.114; #指定DNS服务器
    listen 80;
    location / {
    }
}
server {
    resolver 114.114.114.114; # 指定DNS服务器
    listen 443;
    location / {
    }
}
```



### 虚拟私有网络

Virtual Private Network\(VPN\)

代理模式

正向代理

隧道模式



网关



防火墙



