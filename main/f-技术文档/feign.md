# Feign

## Introduction

Feign是一款Netflix开源的轻量级的Restful Http Client，实现了负载均衡和Rest调用的开源框架，封装了Ribbon（Spring Cloud 2020版本后替代为LoadBalancer）和RestTemplate，实现了面向接口编程，进一步降低了项目耦合度。

> [Feign](https://github.com/OpenFeign/feign) is a declarative web service client. It makes writing web service clients easier. To use Feign create an interface and annotate it. It has pluggable annotation support including Feign annotations and JAX-RS annotations. Feign also supports pluggable encoders and decoders. Spring Cloud adds support for Spring MVC annotations and for using the same `HttpMessageConverters` used by default in Spring Web. Spring Cloud integrates Eureka, Spring Cloud CircuitBreaker, as well as Spring Cloud LoadBalancer to provide a load-balanced http client when using Feign.

Feign目的在于使编写Java HTTP客户端变得更加简单，Feign简化了RestTemplate代码，实现负载均和。

## Feign vs OpenFeign

### Feign

Feign是用来做客户端负载均衡和服务调用的。Feign 支持的注解和用法参考官方文档：https://github.com/OpenFeign/feign官方文档。使用 Feign 的注解定义接口，然后调用这个接口，就可以调用服务注册中心的服务。

### OpenFegin

Feign本身并不支持Spring MVC注解，为了方便使用，Spring Cloud孵化了OpenFeign。OpenFeign的@FeignClinent支持了Spring MVC的注解，可以解析Spring MVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，同时实现负载均衡调用逻辑。

## 负载均衡

Spring Cloud 2020版本之前，Feign默认集成Ribbon，Nacos也很好的兼容了Fegin，默认实现了负载均衡的效果。

Spring Cloud 2020版本之后，移除了对Netfix的Ribbon，Hystrix和Zuul的支持，替代方案如下：

| Netflix产品 | 推荐替代产品              |
| ----------- | ------------------------- |
| Hystrix     | Resilience4j/Sentinel     |
| Ribbon      | Spring Cloud Loadbalancer |
| Zuul 1      | Spring Cloud Gateway      |

## loadbalancer通用逻辑封装

本项目用Sentinel代替Hystrix，Spring Cloud Loadbalancer代替Ribbon，Spring Cloud代替Zuul。

依赖导入：

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
<!--自定义Http连接池-->
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
</dependency>
```

具体配置：

```yaml
djj:
	rest-template:
		maxTotal: 200 # 最大链接数
		maxPreRoute: 50 # 同路由最大并发数
		readTimeout: 30000 # 读取超时时间 单位ms
		connectTimeout: 10000 # 链接超时时间 单位ms
```

**拦截器注解：**

- @EnableFeignInterceptor：开启feign拦截器传递数据给下游服务，包含基础数据和http的相关数据
- @EnableBaseFeignInterceptor：开启feign拦截器传递数据给下游服务，只包含基础数据

**拦截器配置：**

- `FeignInterceptorConfig`：使用feign client访问别的微服务时，将上游传过来的client等信息放入header传递给下一个服务。
- `FeignHttpInterceptorConfig`：使用feign client访问别的微服务时，将上游传过来的access_token、username、roles等信息放入header传递给下一个服务。

## Reference

- 官方文档：https://spring.io/projects/spring-cloud-openfeign
- openfeign3.0.4doc：https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/
- Feign 支持的注解和用法参考官方文档：https://github.com/OpenFeign/feign官方文档。
- https://codeantenna.com/a/KzgcDFh2Rn