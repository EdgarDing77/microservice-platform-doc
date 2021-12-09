# Oauth2

## Introduction

「What」**开放授权**（OAuth，Open Authorization）是一个[开放标准](https://zh.wikipedia.org/wiki/开放标准)，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和[密码](https://zh.wikipedia.org/wiki/密码)提供给第三方应用。

OAuth 代表资源所有者为客户端提供对服务器资源的“安全委托访问”。它指定了一个流程，让资源所有者在不提供凭据的情况下授权第三方访问其服务器资源。OAuth专为与HTTP一起使用而设计，本质上允许在资源所有者批准的情况下，由授权服务器向第三方客户端颁发[访问令牌](https://en.wikipedia.org/wiki/Access_token)。然后第三方使用访问令牌访问由资源服务器托管的受保护资源。特别是，OAuth 2.0 为 Web 应用程序、桌面应用程序、手机和智能设备提供了特定的授权流程。

OAuth2与传统实现方案的比较：

| 传统方法             |        OAuth2        |
| -------------------- | :------------------: |
| 密码平台保管         |     密码用户保管     |
| 授权缺少范围和有效期 | 授权拥有范围和有效期 |
| 多次授权             |       一次授权       |

## 流程

![Oauth 2.0 授权流程的高级概述。](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/func/oauth2-abstract-flow.png)

OAuth 2.0 的运行流程:

1. （A）用户打开客户端以后，客户端要求用户给予授权。
2. （B）用户同意给予客户端授权。
3. （C）客户端使用上一步获得的授权，向认证服务器申请令牌。
4. （D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
5. （E）客户端使用令牌，向资源服务器申请获取资源。
6. （F）资源服务器确认令牌无误，同意向客户端开放资源。

> 假设我们做了一个自己的服务平台，如果不使用 OAuth2 登录方式，那么我们需要用户先完成注册，然后用注册号的账号密码或者用手机验证码登录。而使用了 OAuth2 之后，相信很多人使用过、甚至开发过公众号网页服务、小程序，当我们进入网页、小程序界面，第一次使用就无需注册，直接使用微信授权登录即可，大大提高了使用效率。因为每个人都有微信号，有了微信就可以马上使用第三方服务，这体验不要太好了。而对于我们的服务来说，我们也不需要存储用户的密码，只要存储认证平台返回的唯一ID 和用户信息即可。
>
> 以上是使用了 OAuth2 的授权码模式，利用第三方的权威平台实现用户身份的认证。当然了，如果你的公司内部有很多个服务，可以专门提取出一个认证中心，这个认证中心就充当上面所说的权威认证平台的角色，所有的服务都要到这个认证中心做认证。

这其实就是个单点登录的功能，仅需一次登录，可以在授权范围内的服务与接口进行访问。

## 四种授权模式

OAuth2有四种授权模式：

1. 授权码模式（authorization code）：用在服务端应用之间，这种最安全也最复杂
2. 简单模式（implicit）：用在移动app或webapp（这些app是在用户的设备上，如在手机上调微信来进行认证授权）
3. 密码模式（resource owner password credentials）：应用直接都是受信任的（都是由一家公司开发）
4. 客户端模式（client credentials）：同用在应用间API访问

> 多个系统之间不同域（跨域）则必须使用 *授权码* 或 *简单* 模式。

## 实现



## OpenID

### 介绍

> **OpenID** is an [open standard](https://en.wikipedia.org/wiki/Open_standard) and [decentralized](https://en.wikipedia.org/wiki/Decentralized) [authentication](https://en.wikipedia.org/wiki/Authentication) [protocol](https://en.wikipedia.org/wiki/Communications_protocol) promoted by the non-profit [OpenID Foundation](https://en.wikipedia.org/wiki/OpenID#OpenID_Foundation). It allows users to be authenticated by cooperating sites (known as [relying parties](https://en.wikipedia.org/wiki/Relying_party), or RP) using a third-party identity provider (IDP) service, eliminating the need for [webmasters](https://en.wikipedia.org/wiki/Webmaster) to provide their own *ad hoc* login systems, and allowing users to log into multiple unrelated websites without having to have a separate identity and password for each.

### 技术概述

> An *end user* is the entity that wants to assert a particular identity. A *[relying party](https://en.wikipedia.org/wiki/Relying_party)* (RP) is a web site or application that wants to verify the end user's identifier. Other terms for this party include "service provider" or the now obsolete "consumer". An identity provider, or *OpenID provider* (OP) is a service that specializes in registering OpenID URLs or XRIs. OpenID enables an end user to communicate with a relying party. This communication is done through the exchange of an identifier or *OpenID*, which is the [URL](https://en.wikipedia.org/wiki/Uniform_Resource_Locator) or [XRI](https://en.wikipedia.org/wiki/Extensible_Resource_Identifier) chosen by the end user to name the end user's identity. An identity provider provides the OpenID authentication (and possibly other identity services). The exchange is enabled by a *user-agent*, which is the program (such as a browser) used by the end user to communicate with the relying party and OpenID provider.



## Reference

- 官方：https://oauth.net/2/
- Oauth2简化：https://aaronparecki.com/oauth-2-simplified/
- https://juejin.cn/post/6927169846836723725