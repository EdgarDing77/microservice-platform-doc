# Alibaba Sentinel

## Introduction

> 随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Apache Dubbo、gRPC、Quarkus 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。同时 Sentinel 提供 Java/Go/C++ 等多语言的原生实现。
- **完善的 SPI 扩展机制**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

## 客户端弹性模式

「What」其作用就是，在远程服务发生错误或表现不佳的时候保护远程资源（另一个微服务调用或数据库查询）的客户端免于崩溃。

有四种客户端弹性模式：

1. 客户端负载均衡（client load balance）模式
2. 断路器（circuit breaker）模式
3. 后备（fallback）模式
4. 舱壁（bulkhead）模式

## 架构

### 基本原理

![Image](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/sentinel-basic-arch.png)

## 使用

### 接入

1. 修改pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

1. 接入限流埋点

- HTTP埋点：Sentinel Starter默认对所有HTTP服务提供限流埋点
- 自定义埋点：对某个特定方法进行限流或降级，可以通过`@SentinelResource`来完成自定义配置

1. 配置限流规则：两种配置方式：代码配置和控制太配置

## Reference

- [Github](https://github.com/alibaba/Sentinel)
- [wiki手册](https://github.com/alibaba/Sentinel/wiki)