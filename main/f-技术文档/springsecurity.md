# Spring Security

## Introduction

这里前置可以先看OAuth2。

「Why」想要实现 OAuth2 协议并没有那么简单。OAuth2 协议涉及的技术体系非常复杂，需要综合考虑用户认证、密码加解密和存储、Token 生成和校验、分布式 Session 和公私钥管理，以及完成各个客户端的权限管理。这时就**需要引入专门的安全性开发框架**，而**Spring Security 就是这样一款开发框架**。

Spring Security 专门提供了 UAA（User Account and Authentication，用户账户和认证）服务，封装了 OAuth2 协议，用于管理用户账户、OAuth2 客户端以及用于鉴权的 Token。而 UAA 也只是 Spring Security 众多核心功能中的一部分。

## 初识

引入依赖：

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



## Reference

- Spring Security学习：[学习地址](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/Spring%20Security%20%E8%AF%A6%E8%A7%A3%E4%B8%8E%E5%AE%9E%E6%93%8D/00%20%E5%BC%80%E7%AF%87%E8%AF%8D%20%20Spring%20Security%EF%BC%8C%E4%B8%BA%E4%BD%A0%E7%9A%84%E5%BA%94%E7%94%A8%E5%AE%89%E5%85%A8%E4%B8%8E%E8%81%8C%E4%B8%9A%E4%B9%8B%E8%B7%AF%E4%BF%9D%E9%A9%BE%E6%8A%A4%E8%88%AA.md)