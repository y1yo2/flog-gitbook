# 分布式系统认证Demo

## 认证授权服务

四种授权服务：

* 授权码模式
* 用户密码
* 简单......

### 授权码模式

如何工作（工作原理）？如何配置使用？应用场景是什么？

security提供的，获取授权码的端点（url）+clientID, responseTYPE, scope, redirectURI&gt;&gt;授权码&gt;&gt;令牌

1.无用户信息系统（W系统），获取用户系统（Y系统）的用户信息。需要用户给权限。通过用户系统-认证授权服务，获取授权码。

url：/oauth/authorize

| 参数 | 值 |
| :--- | :--- |
| client\_id | 客户端标识 |
| response\_type | 返回格式；授权码填code |
| scope | 权限类型标识（自定义字符串） |
| redirect\_uri | 重定向地址；申请授权码成功后，浏览器请求该地址，并用code带上授权码 |

2.W系统拿到授权码后，再向Y系统-认证授权服务，获取令牌。

url：/oauth/token

| 参数 | 值 |
| :--- | :--- |
| client\_id | 客户端标识 |
| client\_secret | 客户端秘钥 |
| grant\_type | 授权类型；authorization\_code，授权码模式 |
| code | 授权码 |
| redirect\_uri | 重定向地址；？？？为什么和授权码一致 |

特点：一般用于系统服务间，相互认证授权。accesstoken不经浏览器或APP移动端，服务端交互，减少票据泄露的安全风险。

我的理解：

* 获取授权码，表示当前用户已在Y系统认证。
* 获取票据，表示有授权码，且合法的client（W系统），才允许获得认证和权限。

### 简单模式

如何工作？如何配置使用？适用场景是什么？

1.W系统，经过用户授权后，直接获取Y系统的票据，无需客户端秘钥。

url：/oauth/authorize

| 参数 | 值 |
| :--- | :--- |
| client\_id | 客户端标识 |
| response\_type | 返回格式；简单模式填token |
| scope | 权限类型标识（自定义字符串） |
| redirect\_uri | 重定向地址；申请授权码成功后，浏览器请求该地址，并用code带上授权码 |

返回：[https://www.baidu.com/\#access\_token=3af35e04-2713-4d74-bda7-7b79f3ffa62b&token\_type=bearer&expires\_in=43199](https://www.baidu.com/#access_token=3af35e04-2713-4d74-bda7-7b79f3ffa62b&token_type=bearer&expires_in=43199)

2.token以Hash的形式，存放在重定向uri的fragment中，给到浏览器（js通过监听浏览器地址变化，获取fragment）。

特点场景：用于没有服务端，无法使用授权码，单页面应用/系统。

我的理解：无需授权码，但需用户授权，即先完成用户认证。

### 密码模式

如何工作？如何配置使用？应用场景是什么？

1.W系统，通过用户名&密码，完成Y系统的认证授权，获取Y系统的票据。

url：/oauth/token

| 参数 | 值 |
| :--- | :--- |
| client\_id | 客户端标识 |
| client\_secret | 客户端秘钥 |
| grant\_type | 授权类型；password，密码模式 |
| username | 用户名 |
| password | 密码 |
| redirect\_uri | 重定向地址；申请授权码成功后，浏览器请求该地址，并用code带上授权码 |

特点场景：用于信任的（第一方）其他系统，用户名密码通过外部系统传入，有泄露用户名密码的风险。

我的理解：用户名密码完成认证，并默认授权，得到票据。

### 客户端模式

如何工作？如何配置使用？应用场景是什么？

1.W系统，通过client id和secret，完成Y系统的认证和授权，获得Y系统的票据。（无需用户授权）

url：/oauth/token

| 参数 | 值 |
| :--- | :--- |
| client\_id | 客户端标识 |
| client\_secret | 客户端秘钥 |
| grant\_type | 授权类型；client\_credentials，客户端证书模式 |
| redirect\_uri | 重定向地址；申请授权码成功后，浏览器请求该地址，并用code带上授权码 |

特点场景：绝对信任的系统，无需用户授权，用于一个系统内各个服务的认证授权。

我的理解：通过客户端标识及密码，完成认证授权。



