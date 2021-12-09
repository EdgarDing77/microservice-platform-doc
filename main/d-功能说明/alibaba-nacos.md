# Alibaba Nacos

## Introduction

> Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。 Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

Nacos 支持几乎所有主流类型的“服务”的发现、配置和管理：

- [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)
- [Spring Cloud RESTful Service](https://spring.io/understanding/REST)

Nacos关键特性：

- **服务发现**
- **动态配置服务**
- **动态DNS服务**
- **服务及其元数据管理**

![Image](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/nacos-intro.png)

## 组件架构

### 基本架构

![Image](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/nacos-basic-arch.png)

### 逻辑架构

![Image](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/nacos-logic-arch.png)

### 领域模型

**数据模型：**

Nacos 数据模型 Key 由三元组唯一确定, Namespace默认是空串，公共命名空间（public），分组默认是 DEFAULT_GROUP，及：

- Namespace
- Group
- Service/DataId

**配置领域模型：**

![Image](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/nacos-domain-config-model.png)

## 部署

### 依赖环境

- 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac
  - 64 bit JDK 1.8+
- Maven 3.2.x+

### 下载

发行版地址：https://github.com/alibaba/nacos/releases

### 使用

**启动**

启动命令(standalone代表着单机模式运行，非集群模式):

```
sh startup.sh -m standalone
```

如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：

```
bash startup.sh -m standalone
```

**关闭**

```
sh shutdown.sh
```

## Nacos Spring Cloud

### 配置管理

1. 添加依赖：

```XML
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
  <version>${latest.version}</version>
</dependency>
```

**注意**：版本 [2.1.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-config) 对应的是 Spring Boot 2.1.x 版本。版本 [2.0.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-config) 对应的是 Spring Boot 2.0.x 版本，版本 [1.5.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-config) 对应的是 Spring Boot 1.5.x 版本。

更多版本对应关系参考：[版本说明 Wiki](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/版本说明)

1. 在 `bootstrap.properties` 中配置 Nacos server 的地址和应用名

spring.cloud.nacos.config.server-addr=127.0.0.1:8848

spring.application.name=example

说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```
${prefix}-{spring.profiles.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 ***\*`spring.profiles.active`\**** 为空时，对应的连接符 ***\*`-`\**** 也将不存在，dataId 的拼接格式变成 ****`${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

1. 通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新：

```XML
@RestController
@RequestMapping("/config")
@RefreshScope
public class ConfigController {

    @Value("${useLocalCache:false}")
    private boolean useLocalCache;

    @RequestMapping("/get")
    public boolean get() {
        return useLocalCache;
    }
}
```

1. 首先通过调用 [Nacos Open API](https://nacos.io/zh-cn/docs/open-api.html) 向 Nacos Server 发布配置：dataId 为`example.properties`，内容为`useLocalCache=true`

curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=example.properties&group=DEFAULT_GROUP&content=useLocalCache=true"

1. 运行 `NacosConfigApplication`，调用 `curl http://localhost:8080/config/get`，返回内容是 `true`。
2. 再次调用 [Nacos Open API](https://nacos.io/zh-cn/docs/open-api.html) 向 Nacos server 发布配置：dataId 为`example.properties`，内容为`useLocalCache=false`

curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=example.properties&group=DEFAULT_GROUP&content=useLocalCache=false"

1. 再次访问 `http://localhost:8080/config/get`，此时返回内容为`false`，说明程序中的`useLocalCache`值已经被动态更新了。

## 配置中心

#### 集成

引入jar：

```XML
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

**bootstrap.yml增加配置：**

```YAML
spring:
  cloud:
    nacos:
      config:
        server-addr: ip:prot  #nacos的地址
        file-extension: yml   #配置文件格式
        shared-dataids: common.yml #公共配置文件
```

### 核心功能

**全局公用配置**

```YAML
spring:
  cloud:
    nacos:
      config:
        shared-dataids: common.yml
        refreshable-dataids: common.yml
```

> shared-dataids 配置全局的配置名称
>  refreshable-dataids 配置需要自动刷新的全局配置(默认不支持自动刷新)

**自动刷新**

```java
// 在配置类上添加@RefreshScope
@RefreshScope
```

### 不同环境的配置文件隔离

1. Nacos本身的地址

- 直接通过运行参数设置
- 例如开发环境的nacos地址为127.0.0.1:8848，而生产环境的nacos地址为192.168.28.130，部署生产环境时只需要在项目的启动参数添加以下内容

```bash
//以下参数是本项目个性化的，能同时影响配置中心和注册中心的地址
-Dzlt.nacos.server-addr=192.168.28.130:8848
```

1. 业务配置项：总共有三种方式
2. 通过`spring.profile.active`
3. 通过Nacos的Group
4. 通过Nacos的Namespace（对应的是ID）

**多环境总结：**

**第一种**：通过`DataID`与`profile`实现。

- 优点：这种方式与Spring Cloud Config的实现非常像，用过Spring Cloud Config的用户，可以毫无违和感的过渡过来，由于命名规则类似，所以要从Spring Cloud Config中做迁移也非常简单。
- 缺点：这种方式在项目与环境多的时候，配置内容就会显得非常混乱。配置列表中会看到各种不同应用，不同环境的配置交织在一起，非常不利于管理。
- 建议：项目不多时使用，或者可以结合 `Group`对项目根据业务或者组织架构做一些拆分规划。

**第二种**：通过`Group`实现。

- 优点：通过 `Group`按环境讲各个应用的配置隔离开。可以非常方便的利用 `DataID`和 `Group`的搜索功能，分别从应用纬度和环境纬度来查看配置。
- 缺点：由于会占用 `Group`纬度，所以需要对 `Group`的使用做好规划，毕竟与业务上的一些配置分组起冲突等问题。
- 建议：这种方式虽然结构上比上一种更好一些，但是依然可能会有一些混乱，主要是在 `Group`的管理上要做好规划和控制。

**第三种**：通过`Namespace`实现。

- 优点：官方建议的方式，通过 `Namespace`来区分不同的环境，释放了 `Group`的自由度，这样可以让 `Group`的使用专注于做业务层面的分组管理。同时，Nacos控制页面上对于 `Namespace`也做了分组展示，不需要搜索，就可以隔离开不同的环境配置，非常易用。
- 缺点：没有啥缺点，可能就是多引入一个概念，需要用户去理解吧。
- 建议：直接用这种方式长远上来说会比较省心。虽然可能对小团队而言，项目不多，第一第二方式也够了，但是万一后面做大了呢？

> 多环境注意：不论用哪一种方式实现。对于指定环境的配置（`spring.profiles.active=DEV`、`spring.cloud.nacos.config.group=DEV_GROUP`、`spring.cloud.nacos.config.namespace=83eed625-d166-4619-b923-93df2088883a`），都不要配置在应用的`bootstrap.properties`中。而是在发布脚本的启动命令中，用`-Dspring.profiles.active=DEV`的方式来动态指定，会更加灵活！。

> Nacos使用注意

- Nacos本身的相关配置必须都放在`bootstrap.yml`文件中
- 如果在Nacos添加了应用的配置文件1. **应用读取配置后只会覆盖本地相同key的配置**2. **应用读取配置后会缓存起来，就算停掉Nacos也会生效**

## 服务注册发现

1. 引入jar

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

1. bootstrap.yml增加配置

```YAML
spring:
  cloud:
    nacos:
      discovery:
        server-addr: ip:prot  #nacos的地址
```

1. 工程启动类添加注解`@EnableDiscoveryClient`

## Reference

- [Github](https://github.com/alibaba/nacos)
- [Nacos手册](https://nacos.io/zh-cn/docs/what-is-nacos.html)
- [Spring Cloud for Nacos Config](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/Nacos-config)
- [Spring Cloud for Nacos Discovery](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/Nacos-discovery)