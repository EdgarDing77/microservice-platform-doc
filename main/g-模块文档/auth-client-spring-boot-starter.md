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
│   └── IdTokenClaimConstants.java
├── converter
│   └── CustomUserAuthenticationConverter.java
├── properties
│   ├── SecurityProperties.java
│   ├── TokenStoreProperties.java
│   └── concrete
│       ├── AuthProperties.java
│       ├── PermitProperties.java
│       ├── RenewProperties.java
│       ├── UrlPermissionProperties.java
│       └── ValidateCodeProperties.java
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



### 配置



- DefaultResourceServerConfig：默认资源服务器配置
- DefaultSecurityHandlerConfig：默认安全handler的配置





## Reference

