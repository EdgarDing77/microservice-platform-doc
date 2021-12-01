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

项目编译：`mvn clean package -Dmaven.test.skip=true`

Nacos控制台启动：`sh bin/startup.sh -m standalone`

Sentinel控制台启动：`java -jar -Dserver.port=6999 sentinel-dashboard.jar`

## 配置说明



## 难点记录

- 分布式ID生成器解决方案
- 



## 其它说明

banner.txt logo来源：https://devops.datenkollektiv.de/banner.txt/index.html

Feign：http://www.wangqingzheng.com/huaweiyun/63/274363.html