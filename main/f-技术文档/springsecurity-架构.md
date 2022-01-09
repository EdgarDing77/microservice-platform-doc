# Spring Security架构

## 一、Introduction

「What」**Spring Security** 是一个能够为基于 Spring 的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在 Spring 应用上下文中配置的 Bean，充分利用了 Spring IoC（Inversion of Control 控制反转），DI（Dependency Injection 依赖注入）和 AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。

Spring Security 拥有以下特性：

- 对身份验证和授权的全面且可扩展的支持
- 防御会话固定、点击劫持，跨站请求伪造等攻击
- 支持 Servlet API 集成
- 支持与 Spring Web MVC 集成

**整体架构如下：**

![spring security arch](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/spring-security-arch.jpg)

## 二、核心组件

### **SecurityContextHolder**

SecurityContextHolder：最基本的对象，它是我们存储当前应用程序安全上下文的详细信息，其中包括当前使用应用程序的主体的详细信息。如当前操作的用户是谁，该用户是否已经被认证，他拥有哪些角色权限等。

- 默认情况下，`SecurityContextHolder` 使用 `ThreadLocal` 来存储这些详细信息，这意味着 Security Context 始终可用于同一执行线程中的方法，即使 Security Context 未作为这些方法的参数显式传递。*而要实现父子线程传递可以修改ThreadLocal，利用TTL。*

> **TransmittableThreadLocal** 是Alibaba开源的、用于解决 **“在使用线程池等会缓存线程的组件情况下传递ThreadLocal”** 问题的 InheritableThreadLocal 扩展。若希望 TransmittableThreadLocal 在线程池与主线程间传递，需配合 **TtlRunnable** 和 **TtlCallable** 使用。

获取当前用户的信息：（因为身份信息与当前执行线程绑定，所以可以在SecurityContextHolder进行获取）

```java
Object principal = SecurityContextHolder.getContext()
  .getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
  String username = ((UserDetails)principal).getUsername();
} else {
  String username = principal.toString();
}
```

调用`getContext()`返回的对象是`SecurityContext`接口的一个实例，对应`SecurityContext`接口定义如下：

```java
// org/springframework/security/core/context/SecurityContext.java
public interface SecurityContext extends Serializable {
	Authentication getAuthentication();
	void setAuthentication(Authentication authentication);
}
```

`getContext()`调用返回的对象是`SecurityContext`接口的实例。这是保存在线程本地存储中的对象。正如我们将在下面看到的那样，Spring Security 中的大多数身份验证机制都会返回`UserDetails`的实例作为主体。

### **Authentication**

在 SecurityContext 接口中定义了 getAuthentication 和 setAuthentication 两个抽象方法，当调用 getAuthentication 方法后会返回一个 Authentication 类型的对象。

Authentication：是一个接口，定义如下：

```java
// org/springframework/security/core/Authentication.java
public interface Authentication extends Principal, Serializable {
  // 权限信息列表，默认是GrantedAuthority接口的一些实现类，通常是代表权限信息的一系列字符串。
	Collection<? extends GrantedAuthority> getAuthorities();
  // 密码信息，用户输入的密码字符串，在认证过后通常会被移除，用于保障安全。
	Object getCredentials();
	Object getDetails();
  // 最重要的身份信息，大部分情况下返回的是UserDetails接口的实现类，也是框架中的常用接口之一。
	Object getPrincipal();
	boolean isAuthenticated();
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

以上的 Authentication 接口是 spring-security-core jar 包中的接口，直接继承自 Principal 类，而 Principal 是位于 java.security 包中，由此可知 Authentication 是 spring security 中核心的接口。**通过这个 Authentication 接口的实现类，我们可以得到用户拥有的权限信息列表，密码，用户细节信息，用户身份信息，认证信息等。**

#### UserDetailsService

从上面可以发现，可以从Authentication获取主体，一个`Object`，但是大多情况下，都可以转换成`UserDetails`对象。

 `UserDetails`是 Spring Security 中的核心接口。它代表一种原理，但以一种可扩展的和特定于应用程序的方式。将`UserDetails`视为您自己的用户数据库和`SecurityContextHolder`内部的 Spring Security 所需的适配器。作为您自己的用户数据库中某些内容的表示，通常您会将`UserDetails`强制转换为应用程序提供的原始对象，以便可以调用特定于业务的方法(例如`getEmail()`，`getEmployeeNumber()`等)。

那么如何提供`UserDetails`对象呢？通过`UserDetailsService`接口：

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```

这是在SpringSecurity中为用户加载信息的最常用方法，只需要有关用户的信息，就会在整个框架中使用。

成功认证后，将使用`UserDetails`来构建存储在`SecurityContextHolder`中的`Authentication`对象(有关此[below](https://www.docs4dev.com/docs/zh/spring-security/5.1.2.RELEASE/reference/overall-architecture.html#tech-intro-authentication)的更多信息)。

Spring Security提供了许多`UserDetailsService`的实现：

- `InMemoryDaoImpl`：使用内存实现爱你
- `JdbcDaoImpl`：使用JDBC实现
- 当然，更多倾向于自定义实现

> 关于`UserDetailsService`常常会有一些困惑。它纯粹是用于用户数据的 DAO，除了将数据提供给框架内的其他组件外，不执行其他功能。特别是，它不会对用户进行身份验证，这是由`AuthenticationManager`完成的。在许多情况下，如果您需要自定义身份验证过程，则直接使用[implement AuthenticationProvider](https://www.docs4dev.com/docs/zh/spring-security/5.1.2.RELEASE/reference/overall-architecture.html#core-services-authentication-manager)更有意义。

**这里补充下：**

**Authentication 的 getCredentials() 与 UserDetails 中的 getPassword() 需要被区分对待，前者是用户提交的密码凭证，后者是用户正确的密码，认证器其实就是对这两者进行比对。**

#### GrantedAuthority

`Authentication`提供的另一个重要方法是`getAuthorities()`。此方法提供了`GrantedAuthority`个对象的数组。

`GrantedAuthority`是授予委托人的权限。此类权限通常是“角色”，例如`ROLE_ADMINISTRATOR`或`ROLE_HR_SUPERVISOR`。稍后将这些角色配置为 Web 授权，方法授权和域对象授权。 Spring Security 的其他部分能够解释这些权限，并希望它们存在。 `GrantedAuthority`对象通常由`UserDetailsService`加载。

### 小结

SpringContextHolder用于存储SpringContext，通过调用SpringContext对象中的方法，可以获取Authentication对象，利用该对象可以进一步获取已认证用户的详细信息UserDetails。

回顾一下，到目前为止，我们已经看到了 Spring Security 的主要组成部分：

- `SecurityContextHolder`，以提供对`SecurityContext`的访问权限。
- `SecurityContext`，以保存`Authentication`以及可能特定于请求的安全信息。
- `Authentication`，以特定于 Spring Security 的方式表示主体。
- `GrantedAuthority`，以反映授予主体的应用程序范围的权限。
- `UserDetails`，以提供必要的信息以从应用程序的 DAO 或其他安全数据源构建 Authentication 对象。
- `UserDetailsService`，以便在传入基于`String`的用户名(或证书 ID 等)时创建`UserDetails`。

现在，您已经了解了这些重复使用的组件，下面让我们仔细看看身份验证的过程。

## 三、身份认证

### Spring Security中的身份认证？

1. 用户名和密码被过滤器获取到，封装成 `Authentication`, 通常情况下是 `UsernamePasswordAuthenticationToken` 这个实现类。

2. `AuthenticationManager` 身份管理器负责验证这个 `Authentication`

3. 认证成功后，`AuthenticationManager` 身份管理器返回一个被填充满了信息的（包括上面提到的权限信息，身份信息，细节信息，但密码通常会被移除）`Authentication` 实例。

4. `SecurityContextHolder` 安全上下文容器将第 3 步填充了信息的 `Authentication`，通过 SecurityContextHolder.getContext().setAuthentication(…) 方法，设置到其中。

如下认证例子：

```java
public class AuthenticationExample {
    private static AuthenticationManager am = new SampleAuthenticationManager();

    public static void main(String[] args) throws Exception {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));

        while(true) {
            System.out.println("Please enter your username:");
            String name = in.readLine();
            System.out.println("Please enter your password:");
            String password = in.readLine();
            try {
                Authentication request = new UsernamePasswordAuthenticationToken(name, password);
                Authentication result = am.authenticate(request);
                SecurityContextHolder.getContext().setAuthentication(result);
                break;
            } catch(AuthenticationException e) {
                System.out.println("Authentication failed: " + e.getMessage());
            }
        }
        System.out.println("Successfully authenticated. Security context contains: " +
            SecurityContextHolder.getContext().getAuthentication());
    }
}

class SampleAuthenticationManager implements AuthenticationManager {
    static final List<GrantedAuthority> AUTHORITIES = new ArrayList<GrantedAuthority>();

    static {
        AUTHORITIES.add(new SimpleGrantedAuthority("ROLE_USER"));
    }

    @Override
    public Authentication authenticate(Authentication auth) throws AuthenticationException {
        // 输入与输出相同即可认证通过
        if (auth.getName().equals(auth.getCredentials())) {
            return new UsernamePasswordAuthenticationToken(auth.getName(),
                auth.getCredentials(), AUTHORITIES);
        }
        throw new BadCredentialsException("Bad Credentials");
    }
}
```

在这里，编写了一个Servlet，要求用户输入username和password，这里实现`AuthenticationManager`将所有username于password相同的用户进行认证，并未每个用户分配一个角色：

```bash
Please enter your username:
djj
Please enter your password:
123456
Authentication failed: Bad Credentials
Please enter your username:
djj
Please enter your password:
djj
Successfully authenticated. Security context contains: UsernamePasswordAuthenticationToken [Principal=djj, Credentials=[PROTECTED], Authenticated=true, Details=null, Granted Authorities=[ROLE_USER]]
```

> 请注意，您通常不需要编写任何此类代码。该过程通常在内部进行，例如在 Web 身份验证过滤器中。我们在此处仅包含了代码，以表明在 Spring Security 中实际上构成身份验证的问题有一个非常简单的答案。 `SecurityContextHolder`包含完全填充的`Authentication`对象时，将对用户进行身份验证。

### Web应用中的身份验证

典型的Web应用程序身份验证流程如下：

1. 您访问主页，然后单击链接。
2. 向服务器发送请求，服务器确定您已请求受保护的资源。
3. 由于您目前尚未通过身份验证，因此服务器会发回响应，指示您必须进行身份验证。响应将是 HTTP 响应代码，或重定向到特定网页。
4. 根据身份验证机制，您的浏览器将重定向到特定网页，以便您可以填写表格，或者浏览器将以某种方式检索您的身份(通过 BASIC 身份验证对话框，cookie，X.509 证书等)。 )。
5. 浏览器将响应发送回服务器。这将是包含您填写的表单内容的 HTTP POST 或包含身份验证详细信息的 HTTPHeaders。
6. 接下来，服务器将决定所提供的凭据Token是否有效。如果有效，则将进行下一步。如果它们无效，通常会要求您的浏览器再试一次(因此您返回到上面的(2))。
7. 您尝试引起身份验证过程的原始请求将被重试。希望您已获得足够授权的身份验证，以访问受保护的资源。如果您具有足够的访问权限，则请求将成功。否则，您将收到一个 HTTP 错误代码 403，表示“禁止”。

Spring Security 有不同的类负责上述大多数步骤。主要参与者(按照使用 Sequences)是`ExceptionTranslationFilter`，`AuthenticationEntryPoint`和“身份验证机制”，它们负责调用上一节中看到的`AuthenticationManager`。

#### ExceptionTranslationFilter

`ExceptionTranslationFilter`是 Spring Security 过滤器，负责检测引发的任何 Spring Security 异常。此类异常通常由`AbstractSecurityInterceptor`引发，该`AbstractSecurityInterceptor`是授权服务的主要提供者。

我们将在下一节中讨论`AbstractSecurityInterceptor`，但是现在我们只需要知道它会产生 Java 异常，并且对 HTTP 或对主体进行身份验证一无所知。而是`ExceptionTranslationFilter`提供此服务，具体负责返回错误代码 403(如果主体已通过身份验证，因此仅缺少足够的访问权限-按照上面的第七步)，或者启动`AuthenticationEntryPoint`(如果主体未通过身份验证并且因此我们需要开始第三步)。

#### AuthenticationEntryPoint

`AuthenticationEntryPoint`负责上述列表中的第(3)步。可以想象，每个 Web 应用程序将具有默认的身份验证策略(嗯，可以像配置 Spring Security 中的几乎所有其他功能一样配置它，但是现在让我们保持简单)。每个主要认证系统都有自己的`AuthenticationEntryPoint`实现，该实现通常执行步骤 (3) 中所述的操作之一。

## 四、核心服务

### AbstractSecurityInterceptor

那么，什么是“安全对象”？ Spring Security 使用该术语来指代任何可以对其应用安全性(例如授权决策)的对象。最常见的示例是方法调用和 Web 请求。

每个受支持的安全对象类型都有其自己的拦截器类，该类是`AbstractSecurityInterceptor`的子类。重要的是，到`AbstractSecurityInterceptor`被调用时，如果主体已通过身份验证，则`SecurityContextHolder`将包含有效的`Authentication`。

`AbstractSecurityInterceptor`提供了一致的工作流来处理安全的对象请求，通常：

1. 查找与当前请求关联的“配置属性”（这里配置属性指一些角色名称等）
2. 将安全对象，当前`Authentication`和配置属性提交给`AccessDecisionManager`以进行授权决策
3. (可选)更改发生调用的`Authentication`
4. 允许进行安全对象调用(假设已授予访问权限)
5. 一旦调用返回，则调用`AfterInvocationManager`(如果已配置)。如果调用引发异常，则`AfterInvocationManager`将不会被调用。

### AuthenticationManager

> **该章节以AuthenticationManger、ProviderManager和AuthenticationProvider出发，这是Spring Security的核心类。**

`AuthenticationManager`接口是认证相关的核心接口，也是认证的出发点。

- AuthenticationManager：`AuthenticationManager`只是一个接口，因此实现可以是我们选择的任何东西，但是实际上它如何工作？如果我们需要检查多个身份验证数据库或不同身份验证服务(例如数据库和 LDAP 服务器)的组合，该怎么办？

- ProviderManager：Spring Security 中的默认实现称为`ProviderManager`，而不是处理身份验证请求本身，它委派给已配置的`AuthenticationProviders`  列表（`List<AuthenticationProvider>`），存放多种认证方式*（委托者模式的应用Delegate）*，依次查询每个`AuthenticationProviders` 以查看其是否可以执行认证。
- AuthenticationProvider：每个Provider程序都将引发异常或返回完全填充的`Authentication`对象。

也就是说，核心的认证入口始终只有一个：AuthenticationManager，不同的认证方式：用户名 + 密码（UsernamePasswordAuthenticationToken），邮箱 + 密码，手机号码 + 密码登录则对应了三个 AuthenticationProvider。在默认策略下，只需要通过一个 AuthenticationProvider 的认证，即可被认为是登录成功。`ProviderManager` 中的 List，会依照次序去认证，认证成功则立即返回，若认证失败则返回 null，下一个 AuthenticationProvider 会继续尝试认证，如果所有认证器都无法认证成功，则 `ProviderManager` 会抛出一个 ProviderNotFoundException 异常。

![img](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/spring-security-architecture.png)

### PasswordEncoder

Spring Security 的`PasswordEncoder`接口用于执行密码的单向转换，以允许安全地存储密码。鉴于`PasswordEncoder`是一种单向转换，因此在密码转换需要采用两种方式(即存储用于向数据库进行身份验证的凭据)时并不适用。通常，`PasswordEncoder`用于存储需要与认证时用户提供的密码进行比较的密码。

默认的`PasswordEncoder`现在类似于`BCryptPasswordEncoder`。但是，这忽略了三个现实问题：

1. 有许多使用旧密码编码的应用程序无法轻松迁移
2. 密码存储的最佳做法将再次更改。
3. 作为一个框架，Spring Security 不能经常进行重大更改

相反，Spring Security 引入了`DelegatingPasswordEncoder`，它通过以下方式解决了所有问题：

1. 确保使用当前密码存储建议对密码进行编码
2. 允许以现代和旧式格式验证密码
3. 允许将来升级编码

使用自定义加密模式：

```java
public class DefaultPasswordConfig {
    public static PasswordEncoder getDelegatingPasswordEncoder(String encodingId) {
        Map<String, PasswordEncoder> encoders = new HashMap<>();
        encoders.put("bcrypt", new BCryptPasswordEncoder());
        encoders.put("ldap",
            new org.springframework.security.crypto.password.LdapShaPasswordEncoder());
        encoders.put("MD4", new org.springframework.security.crypto.password.Md4PasswordEncoder());
        encoders.put("MD5",
            new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("MD5"));
        encoders.put("noop",
            org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance());
        encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
        encoders.put("scrypt", new SCryptPasswordEncoder());
        encoders.put("SHA-1",
            new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("SHA-1"));
        encoders.put("SHA-256",
            new org.springframework.security.crypto.password.MessageDigestPasswordEncoder(
                "SHA-256"));
        encoders.put("sha256",
            new org.springframework.security.crypto.password.StandardPasswordEncoder());
        encoders.put("argon2", new Argon2PasswordEncoder());

        Assert.isTrue(encoders.containsKey(encodingId),
            encodingId + " is not found in idToPasswordEncoder");

        DelegatingPasswordEncoder
            delegatingPasswordEncoder = new DelegatingPasswordEncoder(encodingId, encoders);
      
        delegatingPasswordEncoder.setDefaultPasswordEncoderForMatches(encoders.get(encodingId));
        return delegatingPasswordEncoder;
    }

    /**
     * 选取指定的加密方式
     *
     * @return
     */
    @Bean
    @ConditionalOnMissingBean
    public PasswordEncoder passwordEncoder() {
        return getDelegatingPasswordEncoder("bycrypt");
    }
}

```

### Other

#### springSecurityFilterChain

SpringSecurityFilterChain 作为 SpringSecurity 的核心过滤器链在整个认证授权过程中起着举足轻重的地位，每个请求到来，都会经过该过滤器链。



## Reference

- https://www.docs4dev.com/docs/zh/spring-security/5.1.2.RELEASE/reference/overall-architecture.html#architecture-%E5%92%8C-implementation
- https://www.slideshare.net/JesusPerezFranco/spring-security-5
- https://www.cnkirito.moe/spring-security-1/#1-6-%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A7%88%E5%9B%BE
- http://semlinker.com/spring-security-arch/#2-1-SecurityContextHolder%EF%BC%8CSecurityContext-%E5%92%8C-Authentication
- http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/Spring%20Security%20%E8%AF%A6%E8%A7%A3%E4%B8%8E%E5%AE%9E%E6%93%8D/18%20%20%E6%8A%80%E6%9C%AF%E8%B6%8B%E5%8A%BF%EF%BC%9A%E5%A6%82%E4%BD%95%E4%B8%BA%20Spring%20Security%20%E6%B7%BB%E5%8A%A0%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B%E7%89%B9%E6%80%A7%EF%BC%9F.md
- https://mp.weixin.qq.com/s?__biz=MjM5OTI2NDMwMg==&mid=2247484306&idx=1&sn=600d45706cc363b5b6d97393791042eb&chksm=a73f6ad39048e3c5d8e0cc165a4a44730068768a146a0d6324e4430953f04dc52273f0895241&scene=178&cur_album_id=1390224084755349506#rd



