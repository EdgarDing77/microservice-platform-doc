# Spring Security

## Introduction

「Why」想要实现 OAuth2 协议并没有那么简单。OAuth2 协议涉及的技术体系非常复杂，需要综合考虑用户认证、密码加解密和存储、Token 生成和校验、分布式 Session 和公私钥管理，以及完成各个客户端的权限管理。这时就**需要引入专门的安全性开发框架**，而**Spring Security 就是这样一款开发框架**。

Spring Security 专门提供了 UAA（User Account and Authentication，用户账户和认证）服务，封装了 OAuth2 协议，用于管理用户账户、OAuth2 客户端以及用于鉴权的 Token。而 UAA 也只是 Spring Security 众多核心功能中的一部分。

## Spring Security Webflux中的请求处理流程

- 请求 ->
- NettyWebServer(持有ReactorHttpHandlerAdapter) ->
- ReactorHttpHandlerAdapter(持有HttpHandler) ->
- HttpWebHandlerAdapter(继承自HttpHandler，持有WebHandler) ->
- ExceptionHandlingWebHandler(持有FilteringWebHandler) ->
- FilteringWebHandler(继承自WebHandler，持有DefaultWebFilterChain) ->
- DefaultWebFilterChain(持有WebFilterChainProxy以及DispatcherHandler(用于Controller处理和Spring Cloud Gateway处理)) ->
- WebFilterChainProxy(持有SecurityWebFilterChain) ->
- SecurityWebFilterChain(由ServerHttpSecurity所build出来，唯一实现类为MatcherSecurityWebFilterChain，持有Spring Security配置的各种filter)。

### 各个Spring Security Filter依次为：

| Filter                                    | 作用                                                         | 相关配置                                                     |
| ----------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| HttpHeaderWriterWebFilter                 | 向response中加入各种header，比如与安全相关的`X-Frame-Options` | `http.headers()`                                             |
| CorsWebFilter                             | CORS相关                                                     | `http.cors()`                                                |
| CsrfWebFilter                             | CSRF安全配置                                                 | `http.csrf()`                                                |
| ReactorContextWebFilter                   | 用于根据session等创建SecurityContext                         | `http..securityContextRepository()`                          |
| AuthenticationWebFilter                   | 用于完成认证过程                                             | `http.formLogin()`, `http.httpBasic()`, `http.authenticationManager()`, `http.securityContextRepository()` |
| LoginPageGeneratingWebFilter              | 生成登录页面                                                 |                                                              |
| LogoutPageGeneratingWebFilter             | 生成登出之后的页面                                           |                                                              |
| SecurityContextServerWebExchangeWebFilter | 使ServerWebExchange.getPincipal()返回Authentication对象      |                                                              |
| ServerRequestCacheWebFilter               | 缓存request                                                  | `http.requestCache()`                                        |
| LogoutWebFilter                           | 处理登出                                                     | `http.logout()`                                              |
| ExceptionTranslationWebFilter             | 用于处理认证或授权失败的情况(主要决定于AuthorizationWebFilter)，如果认证失败(即SecurityConext中没有Authentication对象，更确切的说是ServerWebExchange.getPrincipal()返回空)，那么将执行ServerAuthenticationEntryPoint，如果授权失败将执行ServerAccessDeniedHandler | `http.exceptionHandling().authenticationEntryPoint()`, `http.exceptionHandling().accessDeniedHandler()` |
| AuthorizationWebFilter                    | 处理授权                                                     | `http.authorizeExchange().pathMatchers()`, `http.authorizeExchange().anyExchange()` |

## 初识

引入依赖：

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 单体应用

在软件系统中，我们把需要访问的内容定义为一种资源（Resource），而安全性的设计核心目标就是针对这些资源进行保护，确保对它们的访问是安全可控的，如对外暴露的HTTP Endpoint。

- 认证Authentication：你是谁
- 授权Authorization：你能做什么

而把以上结合，实现对资源的访问安全，在单体中就如下：

1. HTTP发起请求
2. 通过身份凭证进行认证
3. 获取用户角色进行访问权限的授权
4. 目标资源的访问

### 微服务架构

而到了微服务架构下，情况就比单体复杂很多，因为涉及到了服务之间的调用关系。继续沿用“资源”这个概念，对应到微服务系统中，服务提供者充当的角色就是资源服务器。

把认证和授权结合起来：



![截屏2021-12-10 上午11.18.22](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/springsecurity-ms1.png)

通过将认证与授权进的过程进行集中化管理，因此这也是**授权中心**出现的缘由。而授权中心分发的Token就包含了**权限范围与有效期**。

客户端获取 Token 之后就可以基于这个 Token 发起对微服务的访问。这个时候，服务的提供者需要对这个 Token 进行认证，并根据 Token 的权限范围和有效期从授权中心获取该请求能够访问的特定资源。在微服务系统中，对外的资源表现形式同样可以理解为一个个 HTTP 端点。

关键点就在于构建用于生成和验证Token的授权中心，为此需要引入**OAuth2协议**。OAuth2 协议为我们在客户端程序和资源服务器之间设置了一个**授权层**，并确保 Token 能够在各个微服务中进行有效传递。

OAuth2 是一个相对复杂的协议，综合应用摘要认证、签名认证、HTTPS 等安全性手段，需要提供 Token 生成和校验以及公私钥管理等功能，同时需要开发者入驻并进行权限粒度控制。一般我们应该避免自行实现这类复杂的协议，倾向于借助于特定工具以免重复造轮子。而 Spring Security 为我们提供了实现这一协议的完整解决方案，我们可以使用该框架完成适用于微服务系统中的认证授权机制。

### 用户认证

#### 配置方式

在Spring Security中，初始化用户信息依赖的配置类是`WebSecurityConfigurer`接口，但通常不需要我们自己实现，而是通过继承`WebSecurityConfigurerAdapter`类来简化该配置类的使用方式。

```java
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
		// 允许HttpSecurity对所有HTTP Entpoint访问的HttpServletRequest进行限制
      http.authorizeRequests() 
        // 对所有请求进行认证
          	.anyRequest().authenticated()
        // 指定使用表单作为认证方式
            .and().formLogin()
        // 使用HTTP基础认证方式
            .and().httpBasic();
    }
}
```

#### Spring Security中的用户对象

Spring Security 中的用户对象用来描述用户并完成对用户信息的管理，涉及**UserDetails、GrantedAuthority、UserDetailsService 和 UserDetailsManager**这四个核心对象。

- UserDetails：描述 Spring Security 中的用户。
- GrantedAuthority：定义用户的操作权限。
- UserDetailsService：定义了对 UserDetails 的查询操作。
- UserDetailsManager：扩展 UserDetailsService，添加了创建用户、修改用户密码等功能。

#### 密码安全

**PasswordEncoder**提供了用于完成认证操作的PasswordEncoder组件，另一方面也包含了独立而完整的加密模块。

PasswordEncoder接口代表了一种密码编码器，其**核心作用于指定密码的具体加密方式**，以及如何将一段给定的加密字符串与明文之间完成匹配校验。

常见的几个PasswordEncoder接口：

- NoOpPasswordEncoder：以明文形式保留密码，不对密码进行编码。这种 PasswordEncoder 通常只用于演示，不应该用于生产环境。
- StandardPasswordEncoder：使用 SHA-256 算法对密码执行哈希操作。
- BCryptPasswordEncoder：使用 bcrypt 强哈希算法对密码执行哈希操作。
- Pbkdf2PasswordEncoder：使用 PBKDF2 算法对密码执行哈希操作。

具体应用方式通常仅需通过它的构造函数创建一个实例：

```java
PasswordEncoder p = new BCryptPasswordEncoder();
```

若只需要使用一种PasswordEncoder使用上述方式即可，若需要另一种替换，而如何更方便的替换？`DelegatingPasswordEncoder`扮演一种代理组件的角色，`DelegatingPasswordEncoder` 提供了一种途径，我们也可以根据需要创建符合自己需求的 `DelegatingPasswordEncoder`：

```java
public class PwdEncoderUtil {
    public static PasswordEncoder getDelegatingPasswordEncoder(String encodingId) {
        Map<String, PasswordEncoder> encoders = new HashMap<>();
        encoders.put("bcrypt", new BCryptPasswordEncoder());
        encoders.put("ldap", new org.springframework.security.crypto.password.LdapShaPasswordEncoder());
        encoders.put("MD4", new org.springframework.security.crypto.password.Md4PasswordEncoder());
        encoders.put("MD5", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("MD5"));
        encoders.put("noop", org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance());
// ... 等等PasswordEncoder实例添加进map

        DelegatingPasswordEncoder
            delegatingPasswordEncoder = new DelegatingPasswordEncoder(encodingId, encoders);
        delegatingPasswordEncoder.setDefaultPasswordEncoderForMatches(encoders.get(encodingId));
        return delegatingPasswordEncoder;
    }
}

public class DefaultPasswordConfig {
    /**
     * 选取指定的加密方式
     * @return
     */
    @Bean
    @ConditionalOnMissingBean
    public PasswordEncoder passwordEncoder() {
        return PwdEncoderUtil.getDelegatingPasswordEncoder("bycrypt");
    }
}
```

#### 拓展AuthenticationProvider

扩展 AuthenticationProvider 的过程就是提供一个自定义的 AuthenticationProvider 实现类。这里我们以最常见的用户名密码认证为例，梳理自定义认证过程所需要实现的步骤，如下所示：

![Drawing 3.png](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/springsecurity-AuthenticationProvider.png)



### 访问授权

#### 授权整体过程

在 Spring Security 中，存在一个叫 FilterSecurityInterceptor 的拦截器，它位于整个过滤器链的末端，核心功能是**对权限控制过程进行拦截**，以此判定该请求是否能够访问目标 HTTP 端点。FilterSecurityInterceptor 是整个权限控制的第一个环节，我们把它称为**拦截请求**。

我们对请求进行拦截之后，下一步就要获取该请求的访问资源，以及访问这些资源需要的权限信息。我们把这一步骤称为**获取权限配置**。在 Spring Security 中，存在一个 SecurityMetadataSource 接口，该接口保存着一系列安全元数据的数据源，代表权限配置的抽象。

当我们获取了权限配置信息后，就可以根据这些配置决定 HTTP 请求是否具有访问权限，也就是执行授权决策。Spring Security 专门提供了一个 AccessDecisionManager 接口完成该操作。而在 AccessDecisionManager 接口中，又把具体的决策过程委托给了 AccessDecisionVoter 接口。**AccessDecisionVoter 可以被认为是一种投票器，负责对授权决策进行表决**。

以上三个步骤构成了 Spring Security 的授权整体工作流程，可以用如下所示的时序图表示：

![Drawing 0.png](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/springsecurity-authorization-flow.png)

##### 一、拦截请求

作为一种拦截器，FilterSecurityInterceptor 实现了对请求的拦截。它的定义，如下所示：

```java
public class FilterSecurityInterceptor extends AbstractSecurityInterceptor implements Filter
```

FilterSecurityInterceptor 实现了 Servlet 的 Filter 接口，所以本质上也是一种过滤器，并实现了 Filter 接口的 invoke 方法。在它的 invoke 方法中，FilterSecurityInterceptor 自身并没有执行任何特殊的操作，只是获取了 HTTP 请求并调用了基类 AbstractSecurityInterceptor 中的 beforeInvocation() 方法对请求进行拦截。

AbstractSecurityInterceptor 中的 beforeInvocation() 方法非常长，我们把它裁剪之后，可以得到如下所示的主流程代码：

```java
protected InterceptorStatusToken beforeInvocation(Object object) {
	    …
	    //获取 ConfigAttribute 集合
        Collection< ConfigAttribute > attributes = this.obtainSecurityMetadataSource()
                 .getAttributes(object);
        …
        //获取认证信息
        Authentication authenticated = authenticateIfRequired();
        //执行授权
        try {
             this.accessDecisionManager.decide(authenticated, object, attributes);
        }
        catch (AccessDeniedException accessDeniedException) {
             …
        }
        …
}
```

可以看到，上述操作从配置好的 SecurityMetadataSource 中获取当前请求所对应的 ConfigAttribute，即权限信息。那么，这个 SecurityMetadataSource 又是怎么来的呢？

##### 二、获取权限配置

我们注意到在 FilterSecurityInterceptor 中定义了一个 FilterInvocationSecurityMetadataSource 变量，并通过一个 setSecurityMetadataSource() 方法进行注入，显然，这个变量就是一种 SecurityMetadataSource。

##### 三、执行授权决策

执行授权决策的前提是**获取认证信息**。

##### AccessDecisionManager

AccessDecisionManager 是用来进行授权决策的入口，其最核心的方法就是如下所示的 decide() 方法，前面我们已经看到了这个方法的执行过程：

```java
this.accessDecisionManager.decide(authenticated, object, attributes);
```

## 源码分析

构建操作为将通过 `apply` 方法应用进来的配置分别初始化和构建，链条为 `beforeInit -> init -> beforeConfigure -> configure -> performBuild`。`Spring Security` 中的 `AuthenticationManagerBuilder` （认证管理器生成配置）、`HttpSecurity` （过滤器管理器生成配置）、`WebSecurity` （过滤器生成配置） 都是继承 `AbstractConfiguredSecurityBuilder` 通过这个链条生成目标对象，这 3 个配置也是 `Spring Security` 的配置核心。

### 过滤器链

#### WebSecurityConfigurerAdapter配置

过滤器链生成过程中调用了 `WebSecurityConfigurerAdapter` 的 `init` 和 `configure` 方法。

`init` 方法首先调用了 `getHttp` 方法，用于生成 `AuthenticationManager` 和 `HttpSecurity` 实例。

#### 认证管理器AuthenticationManager的配置

来看看 `WebSecurityConfigurerAdapter` 关于认证管理器的组成：

```java
  /**
    * 认证管理器，管理多种认证方式（AuthenticationProvider），进行实际的认证调用
    */
  private AuthenticationManager authenticationManager;

  /**
    * 认证配置，装配认证方式，通过 @Autowired 自动注入
    */
  private AuthenticationConfiguration authenticationConfiguration;

  /**
    * 同于生成系统配置的认证管理器
    */
  private AuthenticationManagerBuilder authenticationBuilder;

  /**
    * 用于生成开发者可干预的认证管理器
    */
  private AuthenticationManagerBuilder localConfigureAuthenticationBldr;

  /**
    * true - 不使用可干预的认证管理器生成方式
    */
  private boolean disableLocalConfigureAuthenticationBldr;
```

#### 认证管理器的系统逻辑

`WebSecurityConfigurerAdapter` 中通过 `@Autowired` 注入了 `AuthenticationConfiguration`，此类的主要功能是为 `AuthenticationManagerBuilder` 装配 `AuthenticationProvider`，可以装配的认证配置逻辑分为两类：

- （1）在 `spring context` 中查找 `UserDetailsService` 等类的相关实现，包装成 `DaoAuthenticationProvider` 配置到 `AuthenticationManagerBuilder` 中。
  - `DaoAuthenticationProvider` 继承自 `AbstractUserDetailsAuthenticationProvider`，这个抽象类实现了 `AuthenticationProvider` 接口的 `authenticate` 方法，此方法会调用子类实现的 `retrieveUser` 方法。`DaoAuthenticationProvider` 的实现是调用注入进来的 `UserDetailsService` 的 `loadUserByUsername` 方法。
- （2）在 `spring context` 中查找 `AuthenticationProvider` 的实现，直接配置到 `AuthenticationManagerBuilder` 中，但是 `spring` 没有装配 `AuthenticationProvider` 的默认实现。

### 过滤管理器HttpSecurtiy配置

当认证管理器初始化完成，`WebSecurityConfigurerAdapter` 会继续配置 `HttpSecurity`，它用于配置 `web` 请求的安全配置，默认会应用到所有请求，开发者也可通过 `RequestMatcher` 配置例外。 来看看 `HttpSecurity` 的默认配置：

```java
  /**
    * 创建 HttpSecurity 实例
    */
  protected final HttpSecurity getHttp() throws Exception {
    // ...

    http = new HttpSecurity(objectPostProcessor, authenticationBuilder, sharedObjects);
    if (!disableDefaults) {
      http
        // csrf 跨站请求伪造保护
        .csrf().and()
        // 配置异步支持
        .addFilter(new WebAsyncManagerIntegrationFilter())
        // security 异常处理
        .exceptionHandling().and()
        // 将请求的 header 写入响应的 header
        .headers().and()
        // session 管理器，可以配置一个用户仅有一个会话有效
        .sessionManagement().and()
        // 保存认证信息（session维度）
        .securityContext().and()
        // 保存 request cache
        .requestCache().and()
        // 匿名认证配置
        .anonymous().and()
        // 配置重载 servlet 相关安全方法
        .servletApi().and()
        // 表单登录页配置
        .apply(new DefaultLoginPageConfigurer<>()).and()
        // 匹配 /logout 做登出逻辑，成功后跳转登录页
        .logout();

      // ...
    }
    // HttpSecurity 扩展配置
    configure(http);
    return http;
  }

  /**
   * HttpSecurity 扩展配置
   */
  protected void configure(HttpSecurity http) throws Exception {
    http
      // 约束基于 HttpServletRequest 的请求
      .authorizeRequests()
        // 任何请求 需要认证
        .anyRequest().authenticated()
        .and()
      // 表单登录
      .formLogin().and()
      // http basic 认证
      .httpBasic();
  }
```

与配置认证管理器相同的是，在配置 `HttpSecurity` 的过程中，留有一个名为 `configure` 的方法供开发者配置。默认的配置方法拦截了所有请求，要求必须经过身份认证才能正确访问 `web` 资源，默认有表单登录和 `http basic` 两种认证方式可以选择。`HttpSecurity` 提供的大多数配置方法，都是通过过滤器实现的。

**具体配置：**

- form login表单登录：配置两个过滤器
  - `UsernamePasswordAuthenticationFilter`：在创建过滤器时默认使用 `/login POST` 作为表单登录请求，这个过滤器的过滤逻辑就是调用上文中配置的 `AuthenticationManager` 进行认证。
  - `DefaultLoginPageGeneratingFilter`：用于配置登录页面，登录页面默认的登录、登出、登录错误地址分别为 `/login /login?logout /login?error`，其初始化配置在 `HttpSecurity` 的默认配置中。过滤逻辑为当请求为这 3 个地址时，会生成一个表单登录的 `HTML` 并立即返回。
- http basic认证：`httpBasic` 方法配置了 `BasicAuthenticationFilter` 过滤器，其过滤逻辑是从取出 `Authorization` 头，请求头内容为 `username:password` 的 `Base64` 编码形式。在获取用户名、密码后，同样调用 `AuthenticationManager` 进行认证。
- csrf跨站请求伪造保护：`csrf` 方法配置了 `CsrfFilter`，其过滤逻辑为默认放行 `GET` 等请求，其它请求需要进行 `CsrfToken` 校验。访问请求走到这个过滤器时，如果没有携带 `CsrfToken`，会新生成并放入请求中。过滤器链继续走到 `DefaultLoginPageGeneratingFilter`，由于在 `DefaultLoginPageConfigurer` 配置时，从请求中会取出 `CsrfToken` 交给 `DefaultLoginPageGeneratingFilter`，所以 `CsrfToken` 会一并生成 `HTML` 表单，我们使用默认的登录页面就能正确提交表单。
- securityContext：`securityContext` 方法配置了 `SecurityContextPersistenceFilter`，其过滤逻辑为为每个会话创建一个 `SecurityContext`。
- authorizeRequests：在此过滤器的逻辑中，视图对此次访问进行权限验证，如果无权限，则会抛出 `AccessDeniedException`

### 小结

（1） `Spring Security` 开箱即用，拥有完善的默认配置机制，基于过滤器对 `web` 应用进行保护。 

（2） 如果开发者需要对 `Spring Security` 自动配置进行干预，可以继承 `WebSecurityConfigurerAdapter` 并实现它的 3 个 `configure` 方法：

- `void configure(AuthenticationManagerBuilder auth)`：配置认证管理器，开发者需要实现 `UserDetailsService` 接口，编写自定义认证逻辑，并将接口实现注册到 `Spring` 容器，在此方法中指定认证逻辑实现。

```java
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
      auth.userDetailsService(userDetailsService).passwordEncoder(new BCryptPasswordEncoder());
  }
```

- `void configure(HttpSecurity http)`：配置过滤器管理器，开发者在此方法中对默认的 `HttpSecurity` 进行修改：

```java
  protected void configure(HttpSecurity http) throws Exception {
      http
              // 表单登录
              .formLogin().and()
              // 关闭 csrf 保护
              .csrf().disable()
              // 任何请求都需要认证
              .authorizeRequests().anyRequest().authenticated();
  }
```

- `void configure(WebSecurity web)`：请求忽略配置，开发者在此可以配置不需要进行安全认证的请求：

```java
  public void configure(WebSecurity web) throws Exception {
      // 忽略指定url的请求（不走过滤器链）
      web.ignoring().mvcMatchers("/**");
  }
```

（3）重要的定义

- `springSecurityFilterChain`：`Spring Security` 过滤器链。
- `AuthenticationManager`：认证管理器，负责对用户身份进行认证。
- `AuthenticationProvider`：认证逻辑具体实现，由认证管理器调用。
- `UserDetailsService`：通过 `username` 认证，包装成 `AuthenticationProvider` 使用。
- `SecurityContextPersistenceFilter`：从会话中加载有效认证信息或创建默认认证信息上下文。
- `FilterSecurityInterceptor`：最终决定是否放行请求，如果需要认证而未认证，或没有相应的权限，都会判断请求失败。
- `FilterChainProxy`：`Spring Security` 过滤器代理，关于安全的过滤逻辑均在此过滤器中执行，执行完成后才回到 `Spring MVC` 过滤器链中继续执行。

## Spring Security OAuth2

### OAuth2.0 Provider

OAuth2 Provider负责提供暴露OAuth2保护的资源。配置涉及建立可以独立地或代表用户访问其受保护资源的OAuth 2.0客户端。Provider通过管理和验证用于访问受保护资源的OAuth 2.0令牌来实现此操作。在适用的情况下，Provider还必须为用户提供一个接口，以确认客户端可以授予对受保护资源的访问（即确认页面）。

### Provider接口

- [`AuthorizationEndpoint`](https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/endpoint/AuthorizationEndpoint.html) is used to service requests for authorization. Default URL: `/oauth/authorize`.
- [`TokenEndpoint`](https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/endpoint/TokenEndpoint.html) is used to service requests for access tokens. Default URL: `/oauth/token`.
- The [`OAuth2AuthenticationProcessingFilter`](https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/authentication/OAuth2AuthenticationProcessingFilter.html) is used to load the Authentication for the request given an authenticated access token.





## Reference

- Spring Security学习：[学习地址](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/Spring%20Security%20%E8%AF%A6%E8%A7%A3%E4%B8%8E%E5%AE%9E%E6%93%8D/00%20%E5%BC%80%E7%AF%87%E8%AF%8D%20%20Spring%20Security%EF%BC%8C%E4%B8%BA%E4%BD%A0%E7%9A%84%E5%BA%94%E7%94%A8%E5%AE%89%E5%85%A8%E4%B8%8E%E8%81%8C%E4%B8%9A%E4%B9%8B%E8%B7%AF%E4%BF%9D%E9%A9%BE%E6%8A%A4%E8%88%AA.md)
- 官方文档：https://projects.spring.io/spring-security-oauth/docs/oauth2.html
- http://semlinker.com/spring-security-arch/
- https://www.jianshu.com/p/06596fbc02ec