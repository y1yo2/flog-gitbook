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





