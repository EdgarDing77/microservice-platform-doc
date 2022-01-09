# microservice-platfrom-doc

## Introduction

该项目为微服务平台设计与实现的项目文档。

主要针对解决微服务和业务开发时常见的**非功能性需求**。

目录结构如下：

- main - 项目文档主体
  - journal - 项目记录日志
  - issue - 项目问题记录
- other - 相关知识
- images - 图片、思维导图等

## 使用说明

几个必要模块：

- djj-gateway
- djj-uaa(User Account and Authentication，用户账户和认证)

项目编译：`mvn clean package -Dmaven.test.skip=true`

Nacos控制台启动：`sh bin/startup.sh -m standalone`  ｜ `sh bin/shutdown.sh`

Sentinel控制台启动：`java -jar -Dserver.port=6999 sentinel-dashboard.jar`

Redis控制台启动：`redis-server`

项目编译查看：[docker](./main/f-技术文档/docker.md)

日志存放位置：`../logs`

## 模块结构

- djj-uaa：uaa-server:8077
- djj-gateway：sc-gateway:8099
- bussiness：{`:9001+`}
  - user-center：user-center:9001
- djj-register
  - nacos：8848
- djj-sentinel：6999
- djj-web：
  - djj-back-end：9000

## 模块说明

自定义spring-boot-starter模块全部存于djj-infrastructure中。

## 其它说明

banner.txt logo来源：https://devops.datenkollektiv.de/banner.txt/index.html

用户类型：

- USER
- ADMIN

响应说明：

基本响应结构：

```
{
    "data": null,
    "resp_code": 1,
    "resp_msg": "返回信息"
}
```

