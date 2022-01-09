# common-core

## Introduction

该模块位于djj-infrastructure下的，作为所有逻辑封装的通用模块，而通用逻辑封装为common-spring-boot-starter。此模块依赖于djj-log-spring-boot-starter（该模块是所有模块的底层依赖）。

该模块需要完成些什么？

- 所有业务模块、非业务模块都将依赖于该模块
- 认证信息 因为访问下游服务需要带着当前用户、角色、租户等信息
- 所有服务都需要访问用户的信息，因此需要在此配置UserService RPC调用
  - SysUser - username
  - LoginAppUser - username 
  - LoginAppUser - mobile
  - LoginAppUser - openId

- 分布式环境下，锁的实现
- 默认异常的定义
- 系统幂等性的实现
- Token参数解析、多租户Client参数解析
- 通用/默认常量
- 通用/默认配置
- 通用/默认注解定义

## 模块结构

此模块依赖于djj-log-spring-boot-starter（该模块是所有模块的底层依赖）。

```bash
.
├── annotation # 认证信息注入
│   ├── LoginClient.java 
│   └── LoginUser.java
├── config 
│   ├── BannerInitializer.java
│   ├── DefaultAsyncTaskConfig.java
│   ├── DefaultPasswordConfig.java
│   └── DefaultWebMvcConfig.java
├── constants
│   ├── CommonConstants.java
│   ├── ConfigConstants.java
│   ├── SecurityConstants.java
│   └── ServiceNameConstants.java
├── context
│   └── TenantContextHolder.java # TenantContext 通过TransmitableThreadLocal实现，实现父子线程之间数据传递
├── exception
│   ├── BusinessException.java
│   ├── DefaultExceptionAdvice.java
│   ├── IdempotencyException.java
│   └── LockException.java
├── feign
│   ├── UserService.java
│   └── fallback
│       └── UserServiceFallbackFactory.java
├── lock # 这里具体参考 {分布式锁}
│   ├── DLock.java
│   ├── DistributedLock.java
│   ├── Lock.java
│   └── LockAspect.java
├── model
│   ├── LoginAppUser.java
│   ├── SuperEntity.java
│   ├── SysMenu.java
│   ├── SysRole.java
│   ├── SysUser.java
│   └── api
│       ├── CodeEnum.java
│       ├── CommonResult.java
│       ├── PageResult.java
│       └── UserType.java
├── reolver
│   ├── ClientArgumentResolver.java
│   └── TokenArgumentResolver.java
├── service
│   ├── ISuperService.java
│   └── impl
│       └── ISuperServiceImpl.java
└── utils
    ├── CustomBanner.java
    ├── CustomThreadPoolTaskExecutor.java
    ├── JsonUtil.java
    ├── PwdEncoderUtil.java
    ├── ResponseUtil.java
    ├── RsaUtil.java
    └── WebfluxResponseUtil.java
```

## 详细

### 通用常量

- CommonConstants：全局公共常量
- ServiceNameConstans：记录添加的服务名称
- ConfigConstants：配置项常量
- SecurityConstants：Security权限常量

### 通用异常

继承`RuntimeException`实现。

- DefaultExceptioinAdvice：需要业务注入配置继承该类，例如：

  ```java
  @Configuration
  public class ExceptionAdvice extends DefaultExceptionAdvice {
  }
  ```

- BusinessException：通用业务异常
- IdempotencyException：幂等性异常

- LockException：分布式锁异常

### 实体model

主要有四个，都是不需要进行租户隔离的：

1. SysUser
2. SysRole
3. SysMenu
4. LoginAppUser extands SysUser：这里UserId为OpenId

实体表的构建具体查看[MyBatisPlus]()文章。

通用结果集封装为mode.api包下：

1. CodeEnum：成功0，错误1
2. UserType：前端USER，后端管理ADMIN
3. PageResult：分页实体类
4. CommonResult：通用结果集

### 获取当前登陆的信息

通过注解`@LoginUser`获取当前登陆人信息和`@LoginClient`获取当前租户id信息。

其中@LoginUser若需要获取所有的`SysUser`对象信息需要`@LoginUser(isFull = true) SysUser user`，不然默认只注入username和roles。

### 配置

- BannerInitializer：Spring Boot启动自定制。
- DefaultWebMvcConfig：默认Spring MVC拦截器，这里主要带入两个解析，用于解析上游传递来的用户信息token。以下两个方法实现`HandlerMethodArgumentResolver`接口。
  - TokenArgumentResolver
  - ClientArgumentResolver
- DefaultPasswordConfig：密码加密配置类，保证加密方式统一。
- DefaultAsyncTaskConfig：异步线程池配置。

### Feign

封装着对用户信息获取的RPC调用（这是每个服务都需要的）。

UserService：`@FeignClient(name = ServiceNameConstants.USER_SERVICE, fallbackFactory = UserServiceFallbackFactory.class, decode404 = true)`

- /users/name/{username}
- /users-anno/login
- /users-anno/mobile
- /users-anno/openId

### Service

因为MabatisPlus有着对CRUD的进一步封装IService，而这里再进行一层抽象的原因用于进行系统幂等性问题的解决。

这里参考：[mybatisplus]()，[系统幂等性]()

自定义通用Service方法实现：

```java
public class BaseServiceImpl<M extends BaseMapper<T>, T> extends ServiceImpl<M, T> implements
    IBaseService<T> {
}
```

说明：

- T：泛型`T`为任何实体类

- M ：Mapper接口类，继承该接口BaseMapper\<T>后无需编写mapper.xml文件，即可获得CRUD功能
- Wrapper：条件构造器

使用方式：在需要实现幂等性的serviceImpl上继承该类。





## 通用逻辑封装

重新创建一个模块djj-common-spring-boot-starter，该模块只需要添加一个类即可，每个application接受一个请求后的第一个拦截器（租户拦截器），接收上一个服务下来的请求：

```java
@ConditionalOnClass(Filter.class)
public class TenantFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        try {
            String tenantId = request.getParameter(CommonConstants.TENANT_ID_PARAM);
            boolean isEmpty = (tenantId == null || tenantId.length() == 0);
            if (isEmpty) {
                tenantId = request.getHeader(SecurityConstants.TENANT_HEADER);
            }
            if (!isEmpty) {
                TenantContextHolder.setTenant(tenantId);
            }
            filterChain.doFilter(request, response);
        } finally {
            TenantContextHolder.clear();
        }
    }
}
```

## Reference

