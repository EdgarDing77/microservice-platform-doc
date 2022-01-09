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

## 概念

### OAuth2协议角色

**概念-角色**

| 名称       | 英文名               | 描述                                                         | web例子      |
| :--------- | :------------------- | :----------------------------------------------------------- | :----------- |
| 资源所有者 | resource owner       | 能够授予对受保护资源的访问权的实体。当资源所有者是一个人时，它就是用户。 | 用户         |
| 资源服务器 | resource server      | 承载受保护资源的服务器，能够使用访问令牌接受和响应受保护资源请求。 | 后端资源数据 |
| 客户端     | client               | 代表资源所有者及其授权发出受保护资源请求的应用程序。“客户端'' 并不意味着任何特定的实现特征(例如，应用程序是否在服务器、桌面或其他设备上执行)。 | 前端应用     |
| 授权服务器 | authorization server | 在成功认证资源所有者并获得授权后，服务器向客户端发出访问令牌。 | 后端授权     |

### OAuth2的Token

令牌是 OAuth2 协议中非常重要的一个概念，本质上也是一种**代表用户身份的授权凭证**，但与普通的用户名和密码信息不同，**令牌具有针对资源的访问权限范围和有效期**。如下所示就是一种常见的令牌信息：

```
{
    "access_token": "0efa61be-32ab-4351-9dga-8ab668ababae",
    "token_type": "bearer",
    "refresh_token": "738c42f6-79a6-457d-8d5a-f9eab0c7cc5e",
    "expires_in": 43199,
    "scope": "webclient"
}
```

上述令牌信息中的各个字段都很重要，我们展开分析。

- access_token：代表 OAuth2 的令牌，当访问每个受保护的资源时，用户都需要携带这个令牌以便进行验证。
- token_type：代表令牌类型，OAuth2 协议中有多种可选的令牌类型，包括 Bearer 类型、MAC 类型等，这里指定的 Bearer 类型是最常见的一种类型。
- expires_in：用于指定 access_token 的有效时间，当超过这个有效时间，access_token 将会自动失效。
- refresh_token：其作用在于当 access_token 过期后，重新下发一个新的 access_token。
- scope：指定了可访问的权限范围，这里指定的是访问 Web 资源的“webclient”。

OAuth2协议的授权工作流程：

![Drawing 3.png](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/oauth2-token.png)

## 四种授权模式

OAuth 2.0 是一个关于授权的开放的网络协议，是目前最流行的授权机制。

数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

由于授权的场景众多，OAuth 2.0 协议定义了获取令牌的四种授权方式，分别是：

- **「授权码模式」**：授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。
- **「简化模式」**：简化模式（implicit grant type）不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤，因此得名。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。
- **「密码模式」**：密码模式（Resource Owner Password Credentials Grant）中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。
- **「客户端模式」**：客户端模式（Client Credentials Grant）指客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证。严格地说，客户端模式并不属于OAuth框架所要解决的问题。在这种模式中，用户直接向客户端注册，客户端以自己的名义要求"服务提供商"提供服务，其实不存在授权问题。

> 多个系统之间不同域（跨域）则必须使用 *授权码* 或 *隐式* 模式。

常用的方式多为授权码与密码模式。授权模式分别通过使用不同的`grant_type`来区分。

### 授权模式

**当用户同意授权后，授权服务器返回的只是一个授权码，而不是最终的访问令牌**。在这种授权模式下，需要客户端携带授权码去换令牌，这就需要客户端自身具备与授权服务器进行直接交互的后台服务。

![Drawing 4.png](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/oauth2-authorizationcode.png)

这里要注意的是，**通过授权码向授权服务器申请令牌的过程是系统自动完成的，不需要用户的参与，用户需要做的就是在流程启动阶段同意授权**。

### 密码模式

**如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。**

![Drawing 5.png](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/oauth2-passwordcredentials.png)

密码模式比较简单，也更加容易理解。用户要做的就是提供自己的用户名和密码，然后客户端会基于这些用户信息向授权服务器请求令牌。授权服务器成功执行用户认证操作后将会发放令牌。

虽然 OAuth2 协议解决的是授权问题，但它也应用到了认证的概念，这是因为只有验证了用户的身份凭证，我们才能完成对他的授权。所以说，**OAuth2 实际上是一款技术体系比较复杂的协议，综合应用了信息摘要、签名认证等安全性手段，并需要提供令牌以及背后的公私钥管理等功能**。

### 隐式模式

有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。**

1. A网站提供一个链接，要求用户跳转到B网站，授权用户数据给A网站使用

   ```javascript
   https://b.com/oauth/authorize?
     response_type=token&
     client_id=CLIENT_ID&
     redirect_uri=CALLBACK_URL&
     scope=read
   ```

2. 用户跳转到B后，登录后同意给予A网站授权，这时B网站就会跳回`redirect_uri`参数指定的跳转网站，并把令牌作为URL参数，传给A

```javascript
https://a.com/callback#token=ACCESS_TOKEN
```

上面 URL 中，`token`参数就是令牌，A 网站因此直接在前端拿到令牌。

注意，令牌的位置是 URL 锚点（fragment），而不是查询字符串（querystring），这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。

> Implicit Flow 返回 access token 时，不应该用 query (?token=TOKEN-xxx) 而是 fragment (#TOKEN-xxx)
>
> 这在 RFC 6749 里是有规定的 https://tools.ietf.org/html/rfc6749#section-4.2
>
> 在 implicit 中使用 fragment 传递 token 是很巧妙的设计，具体原因可见：https://www.chrisyue.com/?p=3178

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

### 凭证式

**最后一种方式是凭证式（client credentials），适用于没有前端的命令行应用，即在命令行下请求令牌。**

第一步，A 应用在命令行向 B 发出请求。

> ```javascript
> https://oauth.b.com/token?
>   grant_type=client_credentials&
>   client_id=CLIENT_ID&
>   client_secret=CLIENT_SECRET
> ```

上面 URL 中，`grant_type`参数等于`client_credentials`表示采用凭证式，`client_id`和`client_secret`用来让 B 确认 A 的身份。

第二步，B 网站验证通过以后，直接返回令牌。

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

## 实现

依赖引入：

```xml
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
</dependency>
```

### 自定义授权模式

虽然OAuth2定义了4中国呢开始模式，但是实际开发过程中，远远无法满足各种业务需求，因此需要去拓展，如增加图形验证码、手机验证码、手机号密码登录等场景。

常见的做法都是通过增加`Filter`的方式来拓展Spring Security授权，但是这样做的方式有两个问题：

1. 脱离了OAuth2的管理
2. 不灵活：如系统使用「密码模式」授权，网页版需要增加图形验证码校验，但是手机端APP又不需要，使用增加Filter的方式就比较麻烦了。

所以目前在 `Spring Security` 中比较优雅和灵活的扩展方式就是通过自定义 `grant_type` 来增加授权模式。

### 实现思路

在拓展之前首先了解Spring Security的整个授权流程，以**密码模式**为例：

![图片](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/oauth2-password-flow.png)

#### 流程分析

整个授权关键分为两个部分：

一、关于授权类型`grant_type`的解析

1. 每种`grant_type`都会有一个对应的`TokenGranter`实现类
2. 所有`TokenGranter`实现类都通过`CompositeTokenGranter`中的`tokenGranters`集合存起来。
3. 然后通过判断`grant_type`参数定位具体使用哪个`TokenGrant`实现类来处理授权。

二、关于授权登录逻辑

1. 每一个授权方式都对应一个`AuthenticationProvider`实现类来实现。
2. 所有`AuthenticationProvider`实现类都通过`ProviderManager`中的`providers`集合存起来。
3. `TokenGranter` 类会 new 一个 `AuthenticationToken` 实现类，如 `UsernamePasswordAuthenticationToken` 传给 `ProviderManager` 类。
4. 而 `ProviderManager` 则通过 `AuthenticationToken` 来判断具体使用那个 `AuthenticationProvider` 实现类来处理授权。
5. 具体的登录逻辑由 `AuthenticationProvider` 实现类来实现，如 `DaoAuthenticationProvider`。

#### 拓展分析

根据上面的流程，扩展分为以下两种场景

**场景一**：只对原有的授权逻辑进行增强或者扩展，如：用户名密码登录前增加图形验证码校验。

该场景需要定义一个新的 `grantType` 类型，并新增对应的 `TokenGranter` 实现类 **添加扩展内容**，然后加到 `CompositeTokenGranter` 中的 `tokenGranters` 集合里即可。 

**场景二：**新加一种授权方式，如：手机号加密码登录。

该场景需要实现以下内容：

1. 定义一个新的 `grantType` 类型，并新增对应的 `TokenGranter` 实现类加到 `CompositeTokenGranter` 中的 `tokenGranters` 集合里
2. 新增一个 `AuthenticationToken` 实现类，用于存放该授权所需的信息。
3. 新增一个 `AuthenticationProvider` 实现类 **实现授权的逻辑**，并重写 `supports` 方法绑定步骤二的 `AuthenticationToken` 实现类。

#### 代码实现

以场景二新增手机号加密码授权方式为例：

1. 创建AuthenticationToken，用于存储手机号与密码信息：

```java
public class MobileAuthenticationToken extends AbstractAuthenticationToken {
 private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

 private final Object principal;
 private Object credentials;

 public MobileAuthenticationToken(String mobile, String password) {
  super(null);
  this.principal = mobile;
  this.credentials = password;
  setAuthenticated(false);
 }

 public MobileAuthenticationToken(Object principal, Object credentials,
          Collection<? extends GrantedAuthority> authorities) {
  super(authorities);
  this.principal = principal;
  this.credentials = credentials;
  super.setAuthenticated(true);
 }

 @Override
 public Object getCredentials() {
  return this.credentials;
 }

 @Override
 public Object getPrincipal() {
  return this.principal;
 }

 @Override
 public void setAuthenticated(boolean isAuthenticated) {
  if (isAuthenticated) {
   throw new IllegalArgumentException(
     "Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
  }
  super.setAuthenticated(false);
 }

 @Override
 public void eraseCredentials() {
  super.eraseCredentials();
 }
}
```

2. 创建AuthenticationProvider

创建`MobileAuthenticationProvider`类，实现登录逻辑，并绑定`MobileAuthenticationToken`类：

```java
@Setter
public class MobileAuthenticationProvider implements AuthenticationProvider {
    private ZltUserDetailsService userDetailsService;
    private PasswordEncoder passwordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication) {
        MobileAuthenticationToken authenticationToken = (MobileAuthenticationToken) authentication;
        String mobile = (String) authenticationToken.getPrincipal();
        String password = (String) authenticationToken.getCredentials();
        UserDetails user = userDetailsService.loadUserByMobile(mobile);
        if (user == null) {
            throw new InternalAuthenticationServiceException("手机号或密码错误");
        }
        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new BadCredentialsException("手机号或密码错误");
        }
        MobileAuthenticationToken authenticationResult = new MobileAuthenticationToken(user, password, user.getAuthorities());
        authenticationResult.setDetails(authenticationToken.getDetails());
        return authenticationResult;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return MobileAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

3. 创建TokenGranter

创建`MobilePwdGranter`类，并定义grant_type为`mobile_password`：

```java
public class MobilePwdGranter extends AbstractTokenGranter {
    private static final String GRANT_TYPE = "mobile_password";

    private final AuthenticationManager authenticationManager;

    public MobilePwdGranter(AuthenticationManager authenticationManager, AuthorizationServerTokenServices tokenServices
            , ClientDetailsService clientDetailsService, OAuth2RequestFactory requestFactory) {
        super(tokenServices, clientDetailsService, requestFactory, GRANT_TYPE);
        this.authenticationManager = authenticationManager;
    }

    @Override
    protected OAuth2Authentication getOAuth2Authentication(ClientDetails client, TokenRequest tokenRequest) {
        Map<String, String> parameters = new LinkedHashMap<>(tokenRequest.getRequestParameters());
        String mobile = parameters.get("mobile");
        String password = parameters.get("password");
        parameters.remove("password");

        Authentication userAuth = new MobileAuthenticationToken(mobile, password);
        ((AbstractAuthenticationToken) userAuth).setDetails(parameters);
        userAuth = authenticationManager.authenticate(userAuth);
        if (userAuth == null || !userAuth.isAuthenticated()) {
            throw new InvalidGrantException("Could not authenticate mobile: " + mobile);
        }

        OAuth2Request storedOAuth2Request = getRequestFactory().createOAuth2Request(client, tokenRequest);
        return new OAuth2Authentication(storedOAuth2Request, userAuth);
    }
}
```

4. 加入到CompositeTokenGranter中的集合里：

```java
// 添加手机号加密码授权模式
tokenGranters.add(new MobilePwdGranter(authenticationManager, tokenServices, clientDetailsService, requestFactory));
```







### 微服务集成OAuth2授权机制

在OAuth2中，单个微服务的定位就是资源服务器。Spring Security 框架为此提供了专门的 @EnableResourceServer 注解。通过**在 Bootstrap 类中添加 @EnableResourceServer 注解**，相当于声明该服务中的所有内容都是受保护的资源，一旦我们在微服务中添加了 @EnableResourceServer 注解，该服务就会对所有的 HTTP 请求进行验证以确定 Header 部分中是否包含 Token 信息。如果没有 Token 信息，就会直接限制访问；如果有 Token 信息，则通过访问 OAuth2 服务器进行 Token 的验证。

## OpenID

### 介绍

> **OpenID** is an [open standard](https://en.wikipedia.org/wiki/Open_standard) and [decentralized](https://en.wikipedia.org/wiki/Decentralized) [authentication](https://en.wikipedia.org/wiki/Authentication) [protocol](https://en.wikipedia.org/wiki/Communications_protocol) promoted by the non-profit [OpenID Foundation](https://en.wikipedia.org/wiki/OpenID#OpenID_Foundation). It allows users to be authenticated by cooperating sites (known as [relying parties](https://en.wikipedia.org/wiki/Relying_party), or RP) using a third-party identity provider (IDP) service, eliminating the need for [webmasters](https://en.wikipedia.org/wiki/Webmaster) to provide their own *ad hoc* login systems, and allowing users to log into multiple unrelated websites without having to have a separate identity and password for each.

### 技术概述

> An *end user* is the entity that wants to assert a particular identity. A *[relying party](https://en.wikipedia.org/wiki/Relying_party)* (RP) is a web site or application that wants to verify the end user's identifier. Other terms for this party include "service provider" or the now obsolete "consumer". An identity provider, or *OpenID provider* (OP) is a service that specializes in registering OpenID URLs or XRIs. OpenID enables an end user to communicate with a relying party. This communication is done through the exchange of an identifier or *OpenID*, which is the [URL](https://en.wikipedia.org/wiki/Uniform_Resource_Locator) or [XRI](https://en.wikipedia.org/wiki/Extensible_Resource_Identifier) chosen by the end user to name the end user's identity. An identity provider provides the OpenID authentication (and possibly other identity services). The exchange is enabled by a *user-agent*, which is the program (such as a browser) used by the end user to communicate with the relying party and OpenID provider.

## oauth_client_details表

字段解释：https://blog.csdn.net/wangxuelei036/article/details/109491215



## Reference

- 官方：https://oauth.net/2/
- Oauth2简化：https://aaronparecki.com/oauth-2-simplified/
- https://juejin.cn/post/6927169846836723725
- 阮一峰：https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html
- [Spring Security如何增加OAuth2协议授权模式](https://mp.weixin.qq.com/s?__biz=MjM5OTI2NDMwMg==&mid=2247484306&idx=1&sn=600d45706cc363b5b6d97393791042eb&chksm=a73f6ad39048e3c5d8e0cc165a4a44730068768a146a0d6324e4430953f04dc52273f0895241&scene=178&cur_album_id=1390224084755349506#rd)