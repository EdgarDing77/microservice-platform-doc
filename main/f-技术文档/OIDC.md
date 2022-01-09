# OIDC

## Introduction

「What」`OIDC` 是 OpenID Connect 的简称，OIDC=(Identity, Authentication) + OAuth 2.0。它在 `OAuth2` 上构建了一个身份层，是一个基于 OAuth2 协议的身份认证标准协议。我们都知道 OAuth2 是一个授权协议，它无法提供完善的身份认证功能，OIDC 使用 OAuth2 的授权服务器来为第三方客户端提供用户的身份认证，并把对应的身份认证信息传递给客户端，且完全兼容 OAuth2。

## 核心概念

OAuth2 提供了 `Access Token` 来解决授权第三方 `客户端` 访问受保护资源的问题；OIDC 在这个基础上提供了 `ID Token` 来解决第三方客户端标识用户身份认证的问题。OIDC 的核心在于 OAuth2 的授权流程中，一并提供用户的身份认证信息 `ID Token` 给到第三方 `客户端`，`ID Token` 使用 `JWT` 格式来包装。

**「OIDC协议授权返回示例」**：

```
{
    "resp_code": 200,
    "resp_msg": "ok",
    "datas": {
        "access_token": "d1186597-aeb4-4214-b176-08ec09b1f1ed",
        "token_type": "bearer",
        "refresh_token": "37fd65d8-f017-4b5a-9975-22b3067fb30b",
        "expires_in": 3599,
        "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwOi8vemx0MjAwMC5jbiIsImlhdCI6MTYyMTY5NjU4MjYxNSwiZXhwIjoxNjIxNjk2NjQyNjE1LCJzdWIiOiIxIiwibmFtZSI6IueuoeeQhuWRmCIsImxvZ2luX25hbWUiOiJhZG1pbiIsInBpY3R1cmUiOiJodHRwOi8vcGtxdG1uMHAxLmJrdC5jbG91ZGRuLmNvbS_lpLTlg48ucG5nIiwiYXVkIjoiYXBwIiwibm9uY2UiOiJ0NDlicGcifQ.UhsJpHYMWRmny45K0CygXeaASFawqtP2-zgWPDnn0XiBJ6yeiNo5QAwerjf9NFP1YBxuobRUzzhkzRikWGwzramNG9na0NPi4yUQjPNZitX1JzlIA8XSq4LNsuPKO7hS1ALqqiAEHS3oUqKAsjuE-ygt0fN9iVj2LyL3-GFpql0UAFIHhew_J7yIpR14snSh3iLVTmSWNknGu2boDvyO5LWonnUjkNB3XSGD0ukI3UEEFXBJWyOD9rPqfTDOy0sTG_-9wjDEV0WbtJf4FyfO3hPu--bwtM_U0kxRbfLnOujFXyVUStiCKG45wg7iI4Du2lamPJoJCplwjHKWdPc6Zw"
    }
}
```

> 可以看到与普通的 OAuth2 相比返回的信息中除了有 access_token 之外还多出了 **「id_token」** 属性。

## Id Token

ID Token 是一个安全令牌，由授权服务器提供的包含用户信息的 JWT 格式的数据结构，得益于 JWT（JSON Web Token）的自包含性，紧凑性以及防篡改机制，使得 ID Token 可以安全的传递给第三方客户端程序并且容易被验证。

**「id_token包含以下内容」**：

```
{
  "iss": "http://xxx",
  "iat": 1621696582615,
  "exp": 1621696642615,
  "sub": "1",
  "name": "管理员",
  "login_name": "admin",
  "picture": "http://xxx/头像.png",
  "aud": "app",
  "nonce": "t49bpg"
}
```

- **「iss」**：令牌颁发者
- **「iat」**：令牌颁发时间戳
- **「exp」**：令牌过期时间戳
- **「sub」**：用户id
- **「name」**：用户姓名
- **「login_name」**：用户登录名
- **「picture」**：用户头像
- **「aud」**：令牌接收者，OAuth应用ID
- **「nonce」**：随机字符串，用来防止重放攻击

### 与JWT的Access Token区别

是否可以直接使用 JWT 方式的 Access Token 并在 Payload 中加入用户信息来代替 ID Token 呢？

虽然在 Access Token 中可以加入用户的信息，并且是防篡改的，但是用户的每次请求都需要携带着 Access Token，这样不但增加了带宽，而且很容易泄露用户的信息。

###  与 UserInfo 端点的区别

通常 OIDC 协议都需要另外提供了一个 `Get /userinfo` 的 Endpoint，需要通过 Access Token 调用该 Endpoint 来获取详细的用户信息，这个方法和 ID Token 同样都可以获取用户信息，那两者有什么区别呢？

相比较于 `Get /userinfo` 的接口使用 ID Token 可以减少远程 API 调用的额外开销；使用那个主要是看 **「需求」**，当你只需要获取用户的基本信息直接使用 ID Token 就可以了，并不需要每次都通过 Access Token 去调用 `Get /userinfo` 获取详细的用户信息。

## OIDC流程

下面我们看一个 OIDC 协议常用的场景，就是具有 **「独立用户体系」** 系统间的单点登录，意思指的是用户数据并不是统一共用的，而是每个系统都拥有自己独立的用户数据，所以流程最后增加了一步 **「自动注册用户」**。

![img](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/oidc-flow.png)

大部分的流程与 OAuth2 的授权码模式相同这里就不多讲述了，其中下面两个步骤需要说明一下：

- 解析 ID Token 的公钥可以是预先提供给第三方系统也可以是提供接口获取。
- **「自动注册用户」** 指的是第一次单点登录的时候，由于用户信息不存在需要在本系统中生成该用户数据；例如你从未在 CSDN 中注册也可以使用微信来登录该网站。

## Spring Security实现

先说一下扩展最终的目标是需要达到以下效果：

- 授权码模式：`/oauth/authorize?client_id={client_id}&redirect_uri={redirect_uri}&response_type=code`
- OIDC 模式：`/oauth/authorize?client_id={client_id}&redirect_uri={redirect_uri}&response_type=code id_token`

> 目标是要通过在 response_type 中的传值来控制是否使用 OIDC 模式，如果使用则在 response_type 中增加 id_token 的值。

由于需要在 OAuth2 返回的内容中添加 ID Token 属性，所以实现这个扩展的关键就是需要通过 Security 的 `TokenEnhancer` 来为 Token 添加自定义字段。





## Reference