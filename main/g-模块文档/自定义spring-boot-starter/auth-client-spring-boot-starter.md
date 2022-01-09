# auth-client-spring-boot-starter

## Introduction

针对djj-auth-client-spring-boot-starter认证客户端通用逻辑封装的说明。

该模块被所有业务逻辑所依赖。

## 项目结构

```bash
.
├── AuthClientAutoConfigure.java # 鉴权自动配置
├── config
│   ├── DefaultResourceServerConfig.java
│   └── DefaultSecurityHandlerConfig.java
├── constants
│   ├── AuthoritiesEnum.java
│   └── OpenIdTokenClaimConstants.java
├── converter
│   └── CustomUserAuthenticationConverter.java
├── properties
│   ├── SecurityProperties.java
│   ├── TokenStoreProperties.java
│   ├── AuthProperties.java
│   ├── PermitProperties.java
│   ├── RenewProperties.java
│   ├── UrlPermissionProperties.java
│   └── ValidateCodeProperties.java
├── service
│   └── impl
│       └── DefaultPermissionServiceImpl.java
├── store
│   ├── AuthJwtTokenStore.java
│   ├── AuthRedisTokenStore.java
│   ├── CustomRedisTokenStore.java
│   └── ResJwtTokenStore.java
├── token
│   ├── CustomWebAuthenticationDetails.java
│   ├── MobileAuthenticationToken.java
│   ├── OpenIdAuthenticationToken.java
│   └── TenantUsernamePasswordAuthenticationToken.java
└── utils
    ├── AuthUtil.java
    └── JwtUtil.java
```

## 详解

### 常量

AuthoritiesEnum列举了三个角色：

- ADMIN(“ROLE_ADMIN”)
- USER(“ROLE_USER”)
- ANONYMOUS(“ROLE_ANONYMOUS”)

OpenIdTokenClaimConstans：OpenId协议常用的常量

### 配置

- DefaultResourceServerConfig：默认资源服务器配置
- DefaultSecurityHandlerConfig：默认安全handler的配置，这里主要配置如下几个Bean：
  - AuthenticationEntryPoint：启动身份验证方案，实现根据需求修改ServletResponse上的Header
  - OAuth2WebSecurityExpressionHandler：可以处理默认的Web安全表达式
  - OAuth2AccessDeniedHandler：处理spring security oauth处理失败返回消息格式


资源服务器配置：

```java
    @Override
    public void configure(HttpSecurity http) throws Exception {
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>.AuthorizedUrl authorizedUrl = setHttp(http)
            .authorizeRequests()
            .antMatchers(securityProperties.getIgnore().getUrls()).permitAll()
            .antMatchers(HttpMethod.OPTIONS).permitAll()
            .anyRequest();
        setAuthenticate(authorizedUrl);
				// 会话管理配置 SpringSecurity只会在需要时创建一个HttpSession
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
            .and()
                .httpBasic().disable()
                .headers().frameOptions().disable() // 这种情况下，浏览器拒绝当前页面加载任何Frame页面
            .and()
                .csrf().disable(); 
    }
```

> 重要的是：该配置只能控制Spring Security如何创建与使用session，而不是控制整个应用程序。如果我们不明确指定，Spring Security可能不会创建session，但是我们的应用程序可能会创建session（一般spring应用的session管理交由Spring Session进行）！

自定义Properties如下：

- SecurityProperties

  - AuthProperties：认证配置
    - RenewProperties：续权配置
    - UrlPermissionProperties：URL级权限配置

  - PermitProperties：放权的URL白名单，实现URL级权限控制
  - ValidateCodeProperties：验证码配置

- TokenStoreProperties：Token配置 具体的TokenStore配置在store包目录下

具体环境配置参考[环境说明]()。

### converter

这里修改修改自`{@link org.springframework.security.oauth2.provider.token.DefaultUserAuthenticationConverter}`，jwt返回的principal改为返回SysUser，增加拓展字段id，通过id获取当前用户信息与权限。

具体CustomUserAuthenticationConverter修改如下：

```java
    @Override
    public Authentication extractAuthentication(Map<String, ?> map) {
        if (map.containsKey(USERNAME)) {
            Object principal = map.get(USERNAME);
            Collection<? extends GrantedAuthority> authorities = getAuthorities(map);
            if (userDetailsService != null) {
                UserDetails user = userDetailsService.loadUserByUsername((String) map.get(USERNAME));
                authorities = user.getAuthorities();
                principal = user;
            } else {
                Integer id = (Integer)map.get("id");
                LoginAppUser user = new LoginAppUser();
                user.setUsername((String)principal);
                user.setId(Long.valueOf(id));
                principal = user;
            }
            return new UsernamePasswordAuthenticationToken(principal, "N/A", authorities);
        }
        return null;
    }
```

### 自定义实现TokenStore

「What」TokenStore：Persistence interface for OAuth2 tokens(对于OAuth2令牌持久化接口)。
TokenStore 的默认实现有三种：

- InMemoryTokenStore
- JdbcTokenStore
- JwtTokenStore

这里通过Redis去实现更好的TokenStore，同时保留JwtTokenStore，但是基于非对称加密，也需要进行更改。

具体的认证架构参考[服务认证架构设计]()，有网络隔离方式：

- Redis方式：使用redis进行token的持久化与验证

- Jwt方式：uaa中心使用私钥resJwt加密，其他服务使用公钥，对token进行解密。

采取策略的方式具体参考[登录认证]()。

#### JwtTokenStore方式

jwt是将信息存放在令牌中，那么就得考虑其安全性，因此，OAuth2提供了JwtAccessTokenConverter实现，添加jwtSigningKey，以此生成秘钥，以此进行签名，只有jwtSigningKey才能获取信息。

这里通过AuthJwtTokenStore进行私钥加密，ResJwtTokenStore为公钥对Token进行解密。

当获取pubkey失败，则通过访问授权服务器获取非对称加密公钥：

```java
    /**
     * 通过访问授权服务器获取非对称加密公钥 Key
     * @return 公钥 Key
     */
    private String getKeyFromAuthorizationServer() {
        if (StrUtil.isNotEmpty(this.resource.getJwt().getKeyUri())) {
            final HttpHeaders headers = new HttpHeaders();
            final String username = this.resource.getClientId();
            final String password = this.resource.getClientSecret();
            if (username != null && password != null) {
                final byte[] token =
                    Base64.getEncoder().encode((username + ":" + password).getBytes());
                headers.add("Authorization", "Basic " + new String(token));
            }
            final HttpEntity<Void> request = new HttpEntity<>(headers);
            final String url = this.resource.getJwt().getKeyUri();
            return (String) new RestTemplate()
                .exchange(url, HttpMethod.GET, request, Map.class).getBody()
                .get("value");
        }
        return null;
    }
```

请求的URL：`http://127.0.0.1:9900/api-uaa/tokens/key`

请求方式：GET

请求头：

| 参数名        | 参数值                          | 是否必须 | 类型   | 说明                                                         |
| :------------ | :------------------------------ | :------- | :----- | :----------------------------------------------------------- |
| Authorization | Basic {clientId}:{clientSecret} | 是       | string | {clientId}:{clientSecret} 的值必需使用base64加密，clientId为应用id，clientSecret为应用密钥 |

返回结果集：

```json
{
    "datas": null,
    "resp_code": 0,
    "resp_msg": "-----BEGIN PUBLIC KEY-----{data}-----END PUBLIC KEY-----"
}
```

#### RedisTokenStore方式

通过实现TokenStore的CustomRedisTokenStore来针对Token方式进行存储。

具体实现参考[登录认证]()。

### DefaultPermissionServiceImpl

具体实现应该放入到网关中进行具体配置管理（请求抵达的第一个服务）。

请求权限判断Service，实现URL级权限认证，具体配置如下：

```yml
djj:
	security:
		auth:
			urlPermission:
      	enable: false # 默认为false
      	includeClientIds: # 白名单，需要url认证
      	exclusiveClientIds: # 黑名单，不需要url级认证
      	ignoreUrls: # 配置只进行登录认证，而无需url级认证的API
```

其中若同时存在于白名单和黑名单，黑名单优先。具体逻辑实现：

```java
    private boolean isNeedAuth(String clientId) {
        boolean result = true;
        //白名单
        List<String> includeClientIds = securityProperties.getAuth().getUrlPermission().getIncludeClientIds();
        //黑名单
        List<String> exclusiveClientIds = securityProperties.getAuth().getUrlPermission().getExclusiveClientIds();
        if (includeClientIds.size() > 0) {
            result = includeClientIds.contains(clientId);
        }
        if(exclusiveClientIds.size() > 0) {
            result = !exclusiveClientIds.contains(clientId);
        }
        return result;
    }
```

有关URL级权限认证参考[服务认证架构设计]()。

```java
public boolean hasPermission(Authentication authentication, String requestMethod,String requestUrl){
  
}
```

以上为资源权限判断逻辑，具体如下：

1. 前端跨域OPTIONS检视服务器方法，直接放行
2. 判断是否为无效token，若为无效直接false
3. 判断是否开启URL级权限验证
4. 超级管理员admin无需认证
5. 判断应用的黑白名单
6. 判断不进行URL权限的API，所有登录用户都能访问
7. 保存租户信息，根据用户权限进行URL级匹配

## Reference

- https://blog.csdn.net/DuShiWoDeCuo/article/details/78929333
- https://learnku.com/articles/30704
- JWT公私钥：https://cloud.tencent.com/developer/news/837117
- redisTokenStore实现：https://www.cnblogs.com/hellxz/p/12044482.html
