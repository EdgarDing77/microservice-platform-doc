# Spring Cloud

[toc]



## Introduction

**老版Spring Cloud**

概括而言，springcloud的五大组件包括Netflix Eurek，Netflix Ribbon，Netflix Hystrix，Netflix Zuul和Spring Cloud Config。五个组件分别对应（1）服务发现（2）客服端负载均衡（3）断路器（4）服务网关和（5）分布式配置。各组件分工有别，协调统一。

- Spring Boot构建应用程序
- Spring Cloud Config管理微服务的配置，帮助开发人员确保服务的配置信息集中在单个存储库中，并在所有服务实例中都是版本控制和可重复的。
- Netflix Eurek是一个RESTful服务，将服务的位置从客户的使用中抽象出来，提供服务发现。
- Netflix Ribbon主要为客户一侧提供软件的负载均衡，客户端包括连接超时等等。
- Netflix Hystrix来实现客户端调用断路器模式、后备模式和舱壁模式。
- Netflix Zuul为所有的微服务建立了一个单一入口点。
- Spring Cloud Secruity和OAuth2实现服务验证和授权。
- Spring Cloud Stream和Apache Kafka将异步消息传递到微服务中。
- Spring Cloud Sleuth和Open Zipkin来实现日志关联、日志聚合和跟踪等常见日志记录模式。

Spring Cloud集成了各种微服务功能组件，并基于SpringBoot实现了这些组件的自动装配，从而提供了良好的开箱即用体验。

**新版Spring Cloud：**

Spring Cloud 为开发者提供了快速构建分布式系统中一些常见模式的工具（例如配置管理、服务发现、断路器、智能路由、微代理、控制总线、一次性令牌、全局锁、领导选举、分布式会话、集群状态）。分布式系统的协调导致了样板模式，使用 Spring Cloud 开发人员可以快速建立实现这些模式的服务和应用程序。它们将适用于任何分布式环境，包括开发人员自己的笔记本电脑、裸机数据中心和托管平台（如 Cloud Foundry）。

## Spring Cloud

**服务拆分注意事项：**

1. 不同服务，不要重复开发相同业务
2. 微服务数据独立，不要访问其他微服务的数据库
3. 微服务可以将自己的业务暴露为接口，供其他微服务调用

微服务的生命周期：

1. 装配
2. 引导
3. 发现
4. 监控

## 服务配置

将应用程序配置数据直接写入代码，若更改应用配置则需要重新编译与部署。为了避免该情况，配置与应用需要完全分离。

### 管理配置

四个原则：

- 分离：将服务配置与服务的实际物理部署完全分开。
- 抽象：将访问配置数据的功能抽象到一个服务接口种。应用程序使用基于REST的JSON服务来检索配置数据（即从文件中读取）。
- 集中：最小化用于保存配置信息的不同存储库，将应用程序的配置集中在尽可能小的存储库中。
- 稳定：高可用与冗余

### 配置管理架构

如上的四条原则如何应用：

1. 当一个服务实例出现，它将调用一个服务端点来读取所在环境的特定配置信息。配置管理的连接信息（连接凭据、服务端点等）将在微服务启动时被传递给微服务。
2. 实际的配置信息驻留在存储库中。基于配置存储库的实现，可以选择不同的实现来保存配置数据。配置存储库的实现可以选择源代码控制下的文件、关系型数据库或键值数据库等。
3. 应用程序配置数据的实际管理与应用部署的方式无关。
4. 进行配置管理更改的时候，必须通知使用该应用程序配置数据的服务，并刷新应用数据的缓存。

### 方案选择

- Etcd
  - Go开发，使用raft协议作为分布式计算模型。
  - 特点：非常快和可伸缩、可分布式、命令行驱动、易搭建与使用
- Eureka
- Consul
  - 类似于Etcd，在分布式计算模型上使用SWIM协议。
  - 特点：快速、可以与DNS集成
- Zookeeper
  - 一个提供分布式锁定功能的Apache项目，常用键值数据的管理解决方案
  - 特点：最古老、最久经测试的解决方案，使用最为复杂，可配置管理，但只有已经使用来zookeeper的时候才考虑
- Spring Cloud Config
  - 可以将Git、Eureka、Consul作为后端整合
  - 特点：非分布式键值存储，提供来对Spring和非Spring服务的紧密集成，可以使用多个后端来存储配置数据（文件共享系统、Eureka等）

## 服务路由

### Overview

「What」服务网关作为一个独立的应用程序，充当着所有微服务调用的过滤器与路由器的服务。

网关的核心概念就是路由配置和路由规则，而作为所有请求流量的入口，在实际生产中保证高可靠和高可靠，是尽量要避免重启的，所以实现动态路由是非常有必要的。

服务网关实现的横切关注点有如下：

- 静态路由：服务网关将所有的服务调用放置在单个URL和API网关后面。
- 动态路由：服务网关可以检查传入的服务请求，根据传入请求的数据和服务调用者的身份执行智能路由。
- 验证和授权：所有服务调用都需要经过服务网关，所以服务网关是检查服务调用者是否已经进行了验证并被授权进行服务调用的地方。
- 度量数据收集和日志记录：当服务调用通过服务网关的时，可以使用服务网关来收集数据和日志信息，同时可以使用服务网关确保用户请求提供关键信息确保日志统一。如服务调用次数和服务响应时间。

### 实现要点

实现动态路由的要点：

1. 网关启动时，动态路由的数据怎么加载进来；
2. 静态路由与动态路由以哪个为准，ps：静态路由指配置文件里写死的路由配置；
3. 监听动态路由的数据源变化；
4. 数据变化时如何通知Gateway刷新路由；

### 具体实现

`Spring Cloud Gateway` 中加载路由信息分别由以下几个类负责

1. PropertiesRouteDefinitionLocator：从配置文件中读取路由信息(如YML、Properties等)
2. RouteDefinitionRepository：从存储器中读取路由信息(如内存、配置中心、Redis、MySQL等)
3. DiscoveryClientRouteDefinitionLocator：从注册中心中读取路由信息(如Nacos、Eurka、Zookeeper等)

我们可以通过自定义 `RouteDefinitionRepository` 的实现类来实现动态路由的目的

#### 1、实现动态路由的加载

1. 创建一个Nacos的`RouteDefinitionRespository`实现类，重写`getRouteDefinitions()`方法实现路由信息获取。
2. 配置Nacos监听器，监听路由配置的变化。路由变化只需要往 `ApplicationEventPublisher` 推送一个 `RefreshRoutesEvent` 事件即可，gateway会自动监听该事件并调用 `getRouteDefinitions` 方法更新路由信息。

```java
/**
 * Description:
 * nacos路由数据源
 * @author:edgarding
 * @mail:edgarding97@gmail.com
 * @date:2021/11/24
 **/
@Slf4j
public class NacosRouteDefinitionRepository implements RouteDefinitionRepository {
    private static final String SCG_DATA_ID = "scg-route";
    private static final String SCG_GROUP_ID = "SCG_GATEWAY";

    private ApplicationEventPublisher publisher;
    private NacosConfigManager nacosConfigManager;

    public NacosRouteDefinitionRepository(ApplicationEventPublisher publisher, NacosConfigManager nacosConfigManager) {
        this.publisher = publisher;
        this.nacosConfigManager = nacosConfigManager;
        addListener();
    }

    @Override
    public Flux<RouteDefinition> getRouteDefinitions() {
        try {
            String content = nacosConfigManager.getConfigService().getConfig(SCG_DATA_ID, SCG_GROUP_ID,5000);
            List<RouteDefinition> routeDefinitions = getListByStr(content);
            return Flux.fromIterable(routeDefinitions);
        } catch (NacosException e) {
            log.error("NacosRouteDefinitionRepository: getRouteDefinitions by nacos error", e);
        }
        return Flux.fromIterable(CollUtil.newArrayList());
    }

    /**
     * 添加Nacos监听
     */
    private void addListener() {
        try {

            nacosConfigManager.getConfigService().addListener(SCG_DATA_ID, SCG_GROUP_ID, new Listener() {
                @Override
                public Executor getExecutor() {
                    return null;
                }

                @Override
                public void receiveConfigInfo(String configInfo) {
                    publisher.publishEvent(new RefreshRoutesEvent(this));
                }
            });
        } catch (NacosException e) {
            log.error("NacosRouteDefinitionRepository: nacos-addListener-error", e);
        }
    }

    @Override
    public Mono<Void> save(Mono<RouteDefinition> route) {
        return null;
    }

    @Override
    public Mono<Void> delete(Mono<String> routeId) {
        return null;
    }

    private List<RouteDefinition> getListByStr(String content) {
        if (StrUtil.isNotEmpty(content)) {
            return JsonUtil.toList(content, RouteDefinition.class);
        }
        return new ArrayList<>(0);
    }
}
```

#### 2、创建配置类

```java
@Configuration
@ConditionalOnProperty(prefix = "djj.gateway.dynamicRoute", name = "enable", havingValue = "true")
public class DynamicRouteConfig {

    @Autowired
    private ApplicationEventPublisher publisher;

    /**
     * Nacos实现方式
     */
    @Configuration
    @ConditionalOnProperty(prefix = "djj.gateway.dynamicRoute", name = "dataType", havingValue = "nacos", matchIfMissing = true)
    public class NacosDynamicRoute {
        @Autowired
        private NacosConfigProperties nacosConfigProperties;

        @Bean
        public NacosRouteDefinitionRepository nacosRouteDefinitionRepository() {
            return new NacosRouteDefinitionRepository(publisher, new NacosConfigManager(nacosConfigProperties));
        }
    }
}
```

#### 3、添加Nacos路由配置

DataID: scg-route

GroupID: SCG_GATEWAY

测试样例：

```json
[
  {
      "id": "csdn",
      "predicates": [{
          "name": "Path",
          "args": {
                  "pattern": "/csdn/**"
          }
      }],
      "uri": "https://www.csdn.net/",
      "filters": []
  },
  {
      "id": "github",
      "predicates": [{
          "name": "Path",
          "args": {
                  "pattern": "/github/**"
          }
      }],
      "uri": "http://github.com/",
      "filters": []
  }
]
```

## 服务安全

一个安全的应用程序涉及多层保护：

- 确保有正确的用户控制，以便可以确认用户有着正确的权限。
- 保持运行服务的基础设施是打过补丁，并且将漏洞的风险降到最低。
- 实现网络访问控制，让少量已授权的服务器能访问服务，并使服务只能通过定义良好的端口进行访问。

**Spring Cloud Security：**

### **OAuth2**

#### 简介

**「What」OAuth2是一个基于令牌的安全验证和授权框架**。

- 基于令牌的安全框架，允许用户使用第三方认证服务进行验证。
- 若验证成功，将出示一个令牌，该令牌必须与每个请求一起发送，验证服务可以通过该令牌进行验证。
- 主要目的：在调用多个服务的时候，用户不需要在处理请求的时候为每个服务都提供自己的凭证信息就能完成验证。
- 优势：方便与第三方进行集成。

我们使用安全框架的最大意义就在于保护我们的资源，让我们的资源能够在我们希望他被访问到的时候才能够被访问，而存放我们资源的地方就是**资源服务器**。前面已经说过，资源服务器是围绕着授权服务器进行的，在 oauth2 中当有了授权服务器以后，才会有资源服务器，这样说虽然有点绝对，但是如果没有授权服务器，资源服务器其实也就没有太大的存在意义的了，那么还不如就作为一个普通的 web 应用即可。

#### 概览

OAuth2的安全性分解：

1. 受保护资源
2. 资源所有者
3. 应用程序
4. OAuth2验证服务器

OAuth2的四种授权类型：

- 密码password
- 客户端凭据client credential
- 授权码authorization code grant
- 隐式implicit

OAuth2角色：

- 资源拥有者（Resource Owner）：能够访问受保护资源的实体。当资源的所有者是一个人，被称为终端用户。
- 资源服务器（Resource Server）：服务器托管受保护的资源，能够接收和响应使用访问令牌保护资源的请求。
- 客户端（Client）：应用程序使资源所有者的请求有授权访问受保护资源。
- Authorization server：服务器在成功认证资源所有者和获得授权之后发出访问令牌给客户端。

### OpenID Connect协议

#### 简介

OIDC是一个OAuth2上层的简单身份层协议。它允许客户端验证用户的身份并获取基本的用户配置信息。OIDC使用JSON Web Token（JWT）作为信息返回，通过符合OAuth2的流程来获取，更多详细 [10 分钟理解什么是 OAuth 2.0 协议](https://deepzz.com/post/what-is-oauth2-protocol.html)。

OAuth2与资源访问和共享有关，而OIDC与用户身份验证有关。

其目的是为您提供多个站点的登录名。每次需要使用OIDC登录网站时，都会被重定向到登录的OpenID网站，然后再回到该网站。例如，如果选择使用Google帐户登录Auth0，这就使用了OIDC。成功通过Google身份验证并授权Auth0访问您的信息后，Google会将有关用户和执行的身份验证的信息发送回Auth0。此信息在JWT中返回，包含ID Token或者Access Token。

JWT包含Claims，它们是有关实体（通常是用户）的Claims（例如名称或电子邮件地址）和其他元数据。OIDC规范定义了一组标准的权利要求。这组标准声明包括姓名，电子邮件，性别，出生日期等。但是，如果要获取有关用户的信息，并且当前没有最能反映此信息的标准声明，则可以创建自定义声明并将其添加到令牌中。

较OAuth2，OIDC有一些不同的概念：

- OpenID Provider（OP），实现OIDC的OAuth2授权服务器
- Relying Party（RP），使用OIDC的OAuth2客户端
- End-User（EU），用户
- ID Token，JWT格式的授权Claims
- UserInfo Endpoint，用户信息接口，通过ID Token访问时返回用户信息，此端点必须为HTTPS

#### 协议流程

从理论上来讲，OIDC协议遵循以下步骤：

1. RP发送认证请求到OP
2. OP验证End-User并颁发授权
3. OP用ID Token（通常是Access Token）进行响应
4. RP携带Access Token发送请求到UserInfo Endpoint
5. UserInfo Endpoint返回End-User的Claims

#### ID Token

这里预先解释ID Token的含义，OIDC对OAuth2进行的主要扩展（用户用户身份验证）就是ID Token，为JWT格式。其中包含授权服务器对用户验证的Claims和其它请求的Claims。

在ID Token中，以下Clams适用于使用OIDC的所有OAuth2：

- iss，必须，发行机构Issuer，大小写敏感的URL，不能包含query参数
- sub，必须，用户身份Subject，Issuer为End-User分配的唯一标识符，大小写敏感不超过255 ASCII自符
- aud，必须，特别的身份Audience，必须包含OAuth2的client_id，大小写敏感的字符串/数组
- exp，必须，iat到期时间Expire，参数要求当前时间在该时间之前，通常可以时钟偏差几分钟，unix时间戳
- iat，必须，JWT颁发时间Issuer at time，unix时间戳
- auth_time，End-User验证时间，unix时间戳。当发出max_age或auth_time Claims时，必须。
- nonce，用于将Client session和ID Token关联，减轻重放攻击，大小写敏感字符串
- acr，可选，Authentication Context Class Reference，0 End-User不符合ISO/IEC 28115 level 1，不应该授权对任何货币价值的资源访问。大小写敏感的字符串。
- amr，可选，Authentication Methods References，JSON字符串数组，身份验证的表示符，如可能使用了密码和OTP身份验证方式
- azp，可选，Authorized party，被授权方。如果存在必须包含OAuth2的Client ID，仅当ID Token有单个Audience且与授权方不同时，才需要此Claim

ID Token可能包含其它Claims，任何未知的Claims都必须忽略。ID Token必须使用JWS进行签名，并分别使用JWS和JWE进行可选的签名和加密，从而提供身份验证、完整性、不可抵赖性和可选的机密性。如果对ID Token进行了加密，则必须先对其签名，结果是一个嵌套的JWT。ID Token不能使用nonce作为alg值，除非所使用的响应类型没有从Authorization Endpoint返回任何ID Token（如Authorization Code Flow），并且客户端在注册时显示请求使用nonce。

#### 授权

身份验证遵循以下三种方式；授权码方式（response_type=code）、隐式方式（response_type=id_token token或response_type=id_token）、混合方式。

下表是三种方式的特征：

| 属性                         | 授权码 | 隐式 | 混合 |
| ---------------------------- | ------ | ---- | ---- |
| Token从authorization端点返回 | no     | yes  | no   |
| Token从token端点返回         | yes    | no   | no   |
| Token未显示给浏览器          | yes    | no   | no   |
| 能够验证客户端               | yes    | no   | yes  |
| 可以刷新Token                | yes    | no   | yes  |
| 一次交流                     | no     | yes  | no   |
| 服务器到服务器               | yes    | no   | no   |

response_type对应的身份验证方式：

| response_type       | 方式   |
| ------------------- | ------ |
| code                | 授权码 |
| id_token            | 隐式   |
| id_token token      | 隐式   |
| code id_token       | 混合   |
| code token          | 混合   |
| code id_token token | 混合   |

除了由OAuth2定义的“response_type”之外，所有code均在 OAuth2多种响应类型编码实践。

> 注意OAuth2为隐式类型定义token的响应类型，但OIDC不会使用此响应类型，因为不会返回ID Token。

**授权码方式**

使用授权码方式时，所有Token从Token端点返回。授权码将授权code返回给客户端，然后客户端可以将其直接交换为ID Token和Access Token。这样的好处是不会向User-Agent及可能访问User-Agent的其它恶意应用公开任何Token。授权服务器还可以在交换Access Token的授权code之前对客户端进行身份验证。授权code适用于可以安全的维护其自身和授权服务器之间的客户端机密的客户端。

执行以下步骤：

1. 客户端（RP）准备一个包含所需请求参数的身份验证请求
2. 客户端将（RP）请求发送到授权服务器（OP）
3. 授权服务器（OP）对用户（EU）进行身份验证
4. 授权服务器（OP）获得用户同意/或授权
5. 授权服务器（OP）使用授权码将用户发送回客户端（RP）
6. 客户端（RP）使用Token Endpoint的授权码来请求响应
7. 客户端（RP）收到响应，该响应Body中包含ID Token和Access Token
8. 客户端（RP）验证ID Token并检索用户的标识符

## Reference

