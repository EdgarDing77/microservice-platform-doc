# djj-gateway

## Introduction

spring-cloud-gateway提供了一个用于在 Spring WebFlux 之上构建 API 网关的库。Spring Cloud Gateway 旨在提供一种简单而有效的方式来路由到 API 并为它们提供交叉关注点，例如：安全性、监控/指标和弹性。

本项目为3.0.4版本，依赖引入：

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

> Spring Cloud Gateway is built on [Spring Boot 2.x](https://spring.io/projects/spring-boot#learn), [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html), and [Project Reactor](https://projectreactor.io/docs). As a consequence, many of the familiar synchronous libraries (Spring Data and Spring Security, for example) and patterns you know may not apply when you use Spring Cloud Gateway. If you are unfamiliar with these projects, we suggest you begin by reading their documentation to familiarize yourself with some of the new concepts before working with Spring Cloud Gateway

## 基本概念

- Route：网关的基本构建块。它由ID, a destination URI, a collection of predicates, and a collection of filters。如果聚合predicate为真，则匹配路由。
- Predicate：This is a [Java 8 Function Predicate](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html). The input type is a [Spring Framework `ServerWebExchange`](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/server/ServerWebExchange.html). This lets you match on anything from the HTTP request, such as headers or parameters.
- Filter：These are instances of [`GatewayFilter`](https://github.com/spring-cloud/spring-cloud-gateway/tree/3.0.x/spring-cloud-gateway-server/src/main/java/org/springframework/cloud/gateway/filter/GatewayFilter.java) that have been constructed with a specific factory. Here, you can modify requests and responses before or after sending the downstream request.

**工作原理：**

![Spring Cloud Gateway Diagram](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/module/spring_cloud_gateway_diagram.png)

1. Client请求Spring Cloud Gateway
2. Gateway Handler Mapper决定该request匹配route，将发送到Gateway Web Handler
3. 该handler将request通过一个filter chain
4. 全部 “pre” filter logic is executed. 
5. 之后该proxy 请求将被生成，“post” logic开始执行

## 配置

```yml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      routes:
        - id: auth
          uri: lb://uaa-server
          predicates:
            - Path=/api-uaa/**
          filters:
            - StripPrefix=1
            - PreserveHostHeader
```

- filters：Route filters允许修改HTTP出入的一些行为。
  - StripPrefix=1：GatewayFilter采用一个参数parts，在这个parts参数在下游发送之前将会被剥离。具体例子：如上，请求参数`lb://uaa-server/api-uaa/xx`，将在通过网关后变成`lb://uaa-server/xx`。
  - PreserveHostHeader：此filter的目的用于设置路由过滤器检查的请求属性以确定是否应该发送origin host header，而不是Http客户端确定的host header。
- URI
  - lb：说明该URL具有lb方案，则它使用 Spring Cloud`ReactorLoadBalancer`将名称（`myservice`在此示例中）解析为实际主机和端口，并替换同一属性中的 URI。

这里具体说明下：

> If the URL has a `lb` scheme (such as `lb://myservice`), it uses the Spring Cloud `ReactorLoadBalancer` to resolve the name (`myservice` in this example) to an actual host and port and replaces the URI in the same attribute. The unmodified original URL is appended to the list in the `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` attribute. The filter also looks in the `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR` attribute to see if it equals `lb`

### **Global Filter**

请求统计分析埋点过滤器

```java
@Component
public class RequestStatisticsFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        Map<String, String> headers = request.getHeaders().toSingleValueMap();
        UserAgent userAgent = UserAgent.parseUserAgentString(headers.get("User-Agent"));
        //埋点 todo debug 修改
        PointUtil.debug("1", "request-statistics",
            "ip=" + ReactiveAddrUtil.getRemoteAddr(request)
                + "&browser=" + ReactiveAddrUtil.getBrowser(userAgent.getBrowser().name())
                + "&operatingSystem=" +
                ReactiveAddrUtil.getOperatingSystem(userAgent.getOperatingSystem().name()));

        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

生成日志链路追踪id，并传入header中，因此具有最高的优先级：

```java
@Component
public class TraceFilter implements GlobalFilter, Ordered {
    @Autowired
    TraceProperties traceProperties;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        if (traceProperties.getEnable()) {
            String traceId = MDCTraceUtil.createTraceId();
            MDC.put(CommonConstants.LOG_TRACE_ID, traceId);
            ServerHttpRequest httpRequest = exchange.getRequest().mutate()
                .header(CommonConstants.LOG_TRACED_ID_HEADER, traceId)
                .build();

            ServerWebExchange serverWebExchange = exchange.mutate()
                .request(httpRequest)
                .build();

            return chain.filter(serverWebExchange);
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE;
    }
}

```

### CORS webFilter

通过内置的CorsWebFilter支持，具体实现可以通过描述一个CorsWebFilter Bean或构造CorsConfigurationSource。

具体应用如下：

```java
@Configuration
public class CorsConfig {
    public static final String ALL = "*";

    @Order(Ordered.HIGHEST_PRECEDENCE)
    @Bean
    public CorsWebFilter corsFilter() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        // cookie 跨域
        corsConfiguration.setAllowCredentials(true);
        corsConfiguration.addAllowedMethod(ALL);
        corsConfiguration.addAllowedOriginPattern(ALL);
        corsConfiguration.addAllowedHeader(ALL);
        // 配置前端js允许访问自定义响应头
        corsConfiguration.addExposedHeader("setToken");
        UrlBasedCorsConfigurationSource source =
            new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsWebFilter(source);
    }
}
```

### ResourceServerConfig



## Auth

spring security设置要采用响应式配置，基于WebFlux中WebFilter实现，与Spring MVC的Security是通过Servlet的Filter实现类似，也是一系列filter组成的过滤链。
Reactor与传统MVC配置对应：

| webflux                                             | mvc                          | 作用                    |
| --------------------------------------------------- | ---------------------------- | ----------------------- |
| @EnableWebFluxSecurity                              | @EnableWebSecurity           | 开启security配置        |
| ServerAuthenticationSuccessHandler                  | AuthenticationSuccessHandler | 登录成功Handler         |
| ServerAuthenticationFailureHandler                  | AuthenticationFailureHandler | 登陆失败Handler         |
| ReactiveAuthorizationManager\<AuthorizationContext> | AuthorizationManager         | 认证管理                |
| ServerSecurityContextRepository                     | SecurityContextHolder        | 认证信息存储管理        |
| ReactiveUserDetailsService                          | UserDetailsService           | 用户登录                |
| ReactiveAuthorizationManager                        | AccessDecisionManager        | 鉴权管理                |
| ServerAuthenticationEntryPoint                      | AuthenticationEntryPoint     | 未授权Handler           |
| ServerAccessDeniedHandler                           | AccessDeniedHandler          | 拒绝访问异常处理Handler |

具体实现：

- CustomReactiveAuthenticationManager：认证管理，Token的检查
- AuthPermissionManager：实现了ReactiveAuthorizationManager\<AuthorizationContext>，用于认证管理实现URL级权限管理
- CustomServerAuthenticationSuccessHandler：登录成功Hander
- JsonServerAccessDeniedHandler：拒绝访问异常处理，403拒绝访问异常处理，转换为JSON
- JsonServerAuthenticationEntryPoint：未授权，401未授权异常处理，转换为JSON

## sc-gateway - swagger聚合

具体参考[knife4j]()文档，[官方文档](https://doc.xiaominfo.com/knife4j/action/springcloud-gateway.html#_2-1-3-3-%E6%96%87%E6%A1%A3%E8%81%9A%E5%90%88%E4%B8%9A%E5%8A%A1%E7%BC%96%E7%A0%81)。

- SwaggerAggProperties：自定义Properties
- SwaggerResourceConfig：使用springfox-swagger提供的swagger-resource接口，通过网关的注册中心动态发现所有的微服务文档
- SwaggerHandler：相关接口

## sc-gateway 自定义异常处理

### 前提

我们平时在用`SpringMVC`的时候，只要是经过`DispatcherServlet`处理的请求，可以通过`@ControllerAdvice`和`@ExceptionHandler`自定义不同类型异常的处理逻辑，具体可以参考`ResponseEntityExceptionHandler`和`DefaultHandlerExceptionResolver`，底层原理很简单，就是发生异常的时候搜索容器中已经存在的异常处理器并且匹配对应的异常类型，匹配成功之后使用该指定的异常处理器返回结果进行`Response`的渲染，如果找不到默认的异常处理器则用默认的进行兜底(个人认为，Spring在很多功能设计的时候都有这种“有则使用自定义，无则使用默认提供”这种思想十分优雅)。

`SpringMVC`中提供的自定义异常体系在`Spring-WebFlux`中并不适用，其实原因很简单，两者底层的运行容器并不相同。`WebExceptionHandler`是`Spring-WebFlux`的异常处理器顶层接口，因此追溯到子类可以追踪到`DefaultErrorWebExceptionHandler`是`Spring Cloud Gateway`的全局异常处理器，配置类是`ErrorWebFluxAutoConfiguration`。

### 为什么要自定义路由

网关在整个架构中的作用是：

1. 路由服务端应用的请求到后端应用。
2. (聚合)后端应用的响应转发到服务端应用。

假设网关服务总是正常的前提下：

对于第1点来说，假设后端应用不能平滑无损上线，会有一定的几率出现网关路由请求到一些后端的“僵尸节点(请求路由过去的时候，应用更好在重启或者刚好停止)”，这个时候会路由会失败抛出异常，一般情况是Connection Refuse。

对于第2点来说，假设后端应用没有正确处理异常，那么应该会把异常信息经过网关转发回到服务端应用，这种情况理论上不会出现异常。

其实还有第3点隐藏的问题，网关如果不单单承担路由的功能，还包含了鉴权、限流等功能，如果这些功能开发的时候对异常捕获没有做完善的处理甚至是逻辑本身存在BUG，有可能导致异常没有被正常捕获处理，走了默认的异常处理器`DefaultErrorWebExceptionHandler`，默认的异常处理器的处理逻辑可能并不符合我们预期的结果。

### 具体实现

> 以下为案例实现，并非具体实现，

我们可以先看默认的异常处理器的配置类`ErrorWebFluxAutoConfiguration`。注意到两个Bean实例`ErrorWebExceptionHandler`和`DefaultErrorAttributes`都使用了`@ConditionalOnMissingBean`注解，也就是我们可以通过自定义实现去覆盖它们。先自定义一个`CustomErrorWebFluxAutoConfiguration`（除了`ErrorWebExceptionHandler`的自定义实现，其他直接拷贝`ErrorWebFluxAutoConfiguration`）：

`ErrorWebExceptionHandler`的实现，可以直接参考`DefaultErrorWebExceptionHandler`，甚至直接继承`DefaultErrorWebExceptionHandler`，覆盖对应的方法即可。这里直接把异常信息封装成下面格式的`Response`返回，最后需要渲染成JSON格式。

确定三点：

1. 最后封装到响应体的对象来源于`DefaultErrorWebExceptionHandler#getErrorAttributes()`，并且结果是一个`Map<String, Object>`实例转换成的字节序列。
2. 原来的`RouterFunction`实现只支持HTML格式返回，我们需要修改为JSON格式返回(或者说支持所有格式返回)。
3. `DefaultErrorWebExceptionHandler#getHttpStatus()`是响应状态码的封装，原来的逻辑是基于异常属性`getErrorAttributes()`的status属性进行解析的。

自定义的`JsonErrorWebExceptionHandler`如下：

```java
public class JsonErrorWebExceptionHandler extends DefaultErrorWebExceptionHandler {

    public JsonErrorWebExceptionHandler(ErrorAttributes errorAttributes,
                                        ResourceProperties resourceProperties,
                                        ErrorProperties errorProperties,
                                        ApplicationContext applicationContext) {
        super(errorAttributes, resourceProperties, errorProperties, applicationContext);
    }

    @Override
    protected Map<String, Object> getErrorAttributes(ServerRequest request, boolean includeStackTrace) {
        // 这里其实可以根据异常类型进行定制化逻辑
        Throwable error = super.getError(request);
        Map<String, Object> errorAttributes = new HashMap<>(8);
        errorAttributes.put("message", error.getMessage());
        errorAttributes.put("code", HttpStatus.INTERNAL_SERVER_ERROR.value());
        errorAttributes.put("method", request.methodName());
        errorAttributes.put("path", request.path());
        return errorAttributes;
    }

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
        return RouterFunctions.route(RequestPredicates.all(), this::renderErrorResponse);
    }

    @Override
    protected int getHttpStatus(Map<String, Object> errorAttributes) {
        Integer httpStatus = (Integer) errorAttributes.remove("httpStatus");
        return httpStatus != null ? httpStatus : HttpStatus.INTERNAL_SERVER_ERROR.value();
    }
}
```

配置类`CustomErrorWebFluxAutoConfiguration`添加`JsonErrorWebExceptionHandler`：

```java
@Configuration
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.REACTIVE)
@ConditionalOnClass(WebFluxConfigurer.class)
@AutoConfigureBefore(WebFluxAutoConfiguration.class)
@EnableConfigurationProperties({ServerProperties.class, ResourceProperties.class})
public class CustomErrorWebFluxAutoConfiguration {

    private final ServerProperties serverProperties;

    private final ApplicationContext applicationContext;

    private final ResourceProperties resourceProperties;

    private final List<ViewResolver> viewResolvers;

    private final ServerCodecConfigurer serverCodecConfigurer;

    public CustomErrorWebFluxAutoConfiguration(ServerProperties serverProperties,
                                               ResourceProperties resourceProperties,
                                               ObjectProvider<ViewResolver> viewResolversProvider,
                                               ServerCodecConfigurer serverCodecConfigurer,
                                               ApplicationContext applicationContext) {
        this.serverProperties = serverProperties;
        this.applicationContext = applicationContext;
        this.resourceProperties = resourceProperties;
        this.viewResolvers = viewResolversProvider.orderedStream()
                .collect(Collectors.toList());
        this.serverCodecConfigurer = serverCodecConfigurer;
    }

    @Bean
    @ConditionalOnMissingBean(value = ErrorWebExceptionHandler.class, search = SearchStrategy.CURRENT)
    @Order(-1)
    public ErrorWebExceptionHandler errorWebExceptionHandler(ErrorAttributes errorAttributes) {
        JsonErrorWebExceptionHandler exceptionHandler = new JsonErrorWebExceptionHandler(
                    errorAttributes,
                    resourceProperties,
                    this.serverProperties.getError(),
                    applicationContext);
        exceptionHandler.setViewResolvers(this.viewResolvers);
        exceptionHandler.setMessageWriters(this.serverCodecConfigurer.getWriters());
        exceptionHandler.setMessageReaders(this.serverCodecConfigurer.getReaders());
        return exceptionHandler;
    }
}
```

很简单，这里把异常的HTTP响应状态码统一为`HttpStatus.INTERNAL_SERVER_ERROR(500)`，改造的东西并不多，只要了解原来异常处理的上下文逻辑即可。

## Issue

> [bean of type 'org.springframework.http.codec.ServerCodecConfigurer' that could not be found](https://stackoverflow.com/questions/52447223/bean-of-type-org-springframework-http-codec-servercodecconfigurer-that-could-n)



## Reference

- 官方：https://spring.io/projects/spring-cloud-gateway
- https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-cors-webfilter
- 自定义异常：https://www.cnblogs.com/throwable/p/10848879.html