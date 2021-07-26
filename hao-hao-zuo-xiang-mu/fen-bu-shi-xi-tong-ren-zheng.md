---
description: Distributed System Authentication
---

# 分布式系统认证

分布式系统：单体系统拆分为多个服务。每个服务可独立运行维护，提供功能。

* 分布式部署：每个服务可独立部署于不同环境，通过网络交互。
* 弹性伸缩：可部署服务集群，根据实际伸缩集群大小。
* 对内共享：系统内服务间可共享能力。
* 对外开放：服务可通过接口方式，对外开放能力。

分布式认证的要求

统一认证：  
客户端（web端、H5、移动端）、认证方式（用户密码、短信、二维码），采用统一的认证、授权、会话机制。

应用接入认证：  
一方应用（系统内部服务）、三方应用（外部系统），采用统一机制接入。

## 认证方式选型

单体系统时，session认证方式是最流行的认证方式。

### 基于Session的认证方式

Session的认证机制由Servlet规范定制，且Servlet容器已实现。

1. 用户登录成功后，sessionid和用户信息保存在服务端内存中，并将sessionid返回给客户端。
2. 客户端将sessionid保存在cookie（内存或硬盘），向服务端请求时，带上sessionid（通过请求头的cookie值）。
3. 服务端收到sessionid，根据id获取session对象。

分布式架构下使用Session需注意：

1. Session复制：Session需广播给所有服务。
   1. 占用大量网络带宽，所有服务带有session占资源性能低。
2. Session粘性：通过Nginx的 ip\_hash 机制，将 ip与服务绑定。无需广播Session。
   1. 服务故障或重启后，Session丢失。
3. Session共享：使用分布式缓存统一管理Session。
   1. Session基于cookie实现，客户端必须使用cookie，且无法跨域。
   2. 服务获取管理Session复杂，使用成本高。

### 基于token的认证方式

token一般由：uid（用户唯一标识）、timestamp（过期时间戳）、sign（数字签名，例如**MD5**不可逆编码）

1. 客户端登录成功后，服务端生成token并返回客户端。
2. 客户端本地存储token，每次请求时带上token（request header）。
3. 服务端验证token（时间戳、数字签名）。

分布式架构使用token的原因：

1. 满足不同客户端，存储token的方式无限制。
2. 适合第三方应用接入，可使用流行的开放协议Oauth2.0，JWT。
3. 每个服务无需存储token，也可验证token。

## 分布式架构认证技术方案

接入方（客户端）、网关、认证服务（签发token）、资源服务（需网关验证客户端认证和权限、认证服务验证用户认证和权限，才允许访问）。

（补充应用架构图）

### OAuth2.0标准

OAuth2.0是一个开放授权标准。开放授权的目的是，用户授权第三方应用访问内部系统的部分用户信息，而不需要提供用户名和密码或提供用户全部的信息。

（补充OAuth2.0流程图）

### 实现OAuth2.0标准的技术框架

spring-security 是spring的认证授权框架，本身支持OAuth2.0协议进行认证授权。（通过spring-security-oauth2实现）

spring-security-oauth2实现方式，由认证服务和资源服务组成

* 认证服务
  * 认证用户身份：/oauth/authrize
  * 颁发令牌：/oauth/token
* 资源服务
  * 校验令牌合法性：Oauth2AuthenticationProcessingFilter

### 认证授权服务

1. 添加 `@EnableAuthorizationServer` 注解
2. 通过 AuthorizationServerConfigurerAdapter 适配器，配置客户端信息、令牌/票据的管理方式（存储、刷新、有效期、端点url）、需认证授权的安全约束。
3. 
1.Client配置

四种授权模式

令牌访问端点《》授权类型

* 认证管理器（密码授权类型）
* UserDetailsService（刷新令牌授权类型）
* authorizationCodeService（授权码类型）
* implictitGrantService（隐式授权模式）
* tokenGranter（全自定义模式）

默认端点



