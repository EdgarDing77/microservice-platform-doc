# djj-uaa

## Introduction

认证中心的实现说明。

## 项目结构



## 具体配置

### 授权服务器配置

配置AuthorizationServerConfigurerAdapter需要注意的地方有三：

1. 密码校验方式 （PasswordEncoder）

2. AuthenticationManager 

3. UserDetailsService

实现流程如下：

新建`AuthorizationServerConfig`继承`AuthorizationServerConfigurerAdapter`，重写三个config方法：（AuthorizationServerConfigurerAdapter实现AuthorizationServerConfigurer）：

```java
public interface AuthorizationServerConfigurer {

	/**
	 * Configure the security of the Authorization Server, which means in practical terms the /oauth/token endpoint. The
	 * /oauth/authorize endpoint also needs to be secure, but that is a normal user-facing endpoint and should be
	 * secured the same way as the rest of your UI, so is not covered here. The default settings cover the most common
	 * requirements, following recommendations from the OAuth2 spec, so you don't need to do anything here to get a
	 * basic server up and running.
	 * 
	 * @param security a fluent configurer for security features
	 */
	void configure(AuthorizationServerSecurityConfigurer security) throws Exception;

	/**
	 * Configure the {@link ClientDetailsService}, e.g. declaring individual clients and their properties. Note that
	 * password grant is not enabled (even if some clients are allowed it) unless an {@link AuthenticationManager} is
	 * supplied to the {@link #configure(AuthorizationServerEndpointsConfigurer)}. At least one client, or a fully
	 * formed custom {@link ClientDetailsService} must be declared or the server will not start.
	 * 
	 * @param clients the client details configurer
	 */
	void configure(ClientDetailsServiceConfigurer clients) throws Exception;

	/**
	 * Configure the non-security features of the Authorization Server endpoints, like token store, token
	 * customizations, user approvals and grant types. You shouldn't need to do anything by default, unless you need
	 * password grants, in which case you need to provide an {@link AuthenticationManager}.
	 * 
	 * @param endpoints the endpoints configurer
	 */
	void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception;

}

```







## 详解

这里具体实现几种模式：

- 授权码模式（authorization code）
- 隐式模式（implicit）：用在移动app或webapp（这些app是在用户的设备上，如在手机上调微信来进行认证授权）
- 密码模式（resource owner password credentials）：应用直接都是受信任的（都是由一家公司开发）
- 客户端凭证模式（client credentials）：同用在应用间API访问

自定义实现的认证模式：

- password_code：username+password+validateCode实现的授权模式



所有实现模式：

- 默认模式

  - “refresh_token”：刷新令牌模式

  - “implicite”：隐式模式

  - “client_credentials”：客户端模式

  - “password”：密码模式

- 自定义模式：

  - “password_code”：用户名+密码+验证码模式



## Reference

