# Spring WebFlux

## Introduction

Spring WebFlux 是 Spring Framework 5.0中引入的新的**响应式web框架**。与Spring MVC不同，它不需要Servlet API，是完全异步且非阻塞的，并且通过Reactor项目实现了Reactive Streams规范。

Spring WebFlux 用于创建基于事件循环执行模型的完全异步且非阻塞的应用程序。

（PS：所谓异步非阻塞是针对服务端而言的，是说服务端可以充分利用CPU资源去做更多事情，这与客户端无关，客户端该怎么请求还是怎么请求。）

Reactive Streams是一套用于构建高吞吐量、低延迟应用的规范。而Reactor项目是基于这套规范的实现，它是一个完全非阻塞的基础，且支持背压。Spring WebFlux基于Reactor实现了完全异步非阻塞的一套web框架，是一套响应式堆栈。

- [spring-webmvc + Servlet + Tomcat] 命令式的、同步阻塞的

- [spring-webflux + Reactor + Netty] 响应式的、异步非阻塞的

## Illustration

![知识点架构](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/webflux-intro.png)

### 响应式编程

**「What」响应式编程（reactive programming）**是一种基于**数据流（data stream）**和**变化传递（propagation of change）**的**声明式（declarative）的编程范式**。

变化传递（propagation of change）通过一个例子进行理解：

- 在命令式编程下：式子`a=b+c`意味着通过a的值是由b和c计算而来，而若后序b和c的值发生了变化**不会影响到**a值。
- 在响应式编程下，式子`a:=b+c`，若后序b和c的值发生了变化，**会影响到**a的值。

数据流（data stream）和声明式（declarative）理解：

- 可以通过Java的Stream流来理解，Stream的三个步骤（创建stream、执行中间操作、执行最终操作）
- 而本来数据是需要我们自行处理的，后来我们把要**处理的数据抽象**出来（变成数据流），然后**通过API去处理**数据流中的数据（是声明式的）

### 异步非阻塞

因为**响应式编程它是异步**的，也可以理解成**变化传递**它是异步执行的。

### SpringMVC与WebFlux

![spring mvc 和 webflux venn](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/spring-mvc-and-webflux-venn.png)

反应式和非阻塞式的主要预期好处是能够使用少量固定数量的线程和更少的内存进行扩展。这使得应用程序在负载下更具弹性，因为它们以更可预测的方式扩展。然而，为了观察这些好处，您需要有一些延迟（包括缓慢和不可预测的网络 I/O 的混合）。这就是反应堆开始显示其优势的地方，并且差异可能是巨大的。

Spring MVC 和 Spring WebFlux 都支持带注释的控制器，但在并发模型和阻塞和线程的默认假设上有一个关键的区别。

- 在 Spring MVC（以及一般的 servlet 应用程序）中，假设应用程序可以阻塞当前线程（例如，用于远程调用）。出于这个原因，servlet 容器在请求处理期间使用一个大线程池来吸收潜在的阻塞。

- 在 Spring WebFlux（以及一般的非阻塞服务器）中，假设应用程序不会阻塞。因此，非阻塞服务器使用一个小的、固定大小的线程池（事件循环工作者）来处理请求。

## Reference

- https://segmentfault.com/a/1190000021038373
- 官网：https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html