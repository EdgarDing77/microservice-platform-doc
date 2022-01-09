# Spring WebFlux

## Introduction

Spring WebFlux 是 Spring Framework 5.0中引入的新的**响应式web框架**。与Spring MVC不同，它不需要Servlet API，是完全异步且非阻塞的，并且通过Reactor项目实现了Reactive Streams规范。

Spring WebFlux是Spring MVC的并行版本，并支持无阻塞响应式。并使用Netty作为内置服务器来运行响应式应用程序。

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

在响应式系统中，任何操作都可以被看作是一种事件，而这些事件构成了数据流。这个数据流对于技术栈而言是一个全流程的概念。也就是说，**无论是从底层数据库，向上到达服务层，最后到 Web 层，亦或是在这个流程中所包含的任意中间层组件，整个数据传递链路都应该是采用事件驱动的方式来进行运作**。这样，我们就可以不采用传统的同步调用方式来处理数据，而是由位于数据库上游的各层组件自动来执行事件。这就是响应式编程的核心特点。

### 异步非阻塞

因为**响应式编程它是异步**的，也可以理解成**变化传递**它是异步执行的。

### SpringMVC与WebFlux

![spring mvc 和 webflux venn](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/tech/spring-mvc-and-webflux-venn.png)

反应式和非阻塞式的主要预期好处是能够使用少量固定数量的线程和更少的内存进行扩展。这使得应用程序在负载下更具弹性，因为它们以更可预测的方式扩展。然而，为了观察这些好处，您需要有一些延迟（包括缓慢和不可预测的网络 I/O 的混合）。这就是反应堆开始显示其优势的地方，并且差异可能是巨大的。

Spring MVC 和 Spring WebFlux 都支持带注释的控制器，但在并发模型和阻塞和线程的默认假设上有一个关键的区别。

- 在 Spring MVC（以及一般的 servlet 应用程序）中，假设应用程序可以阻塞当前线程（例如，用于远程调用）。出于这个原因，servlet 容器在请求处理期间使用一个大线程池来吸收潜在的阻塞。
- 在 Spring WebFlux（以及一般的非阻塞服务器）中，假设应用程序不会阻塞。因此，非阻塞服务器使用一个小的、固定大小的线程池（事件循环工作者）来处理请求。

## 简单案例

依赖导入：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>  
```

请注意这里的 spring-boot-starter-webflux 就是构成响应式 Web 应用程序开发的基础。基于 WebFlux 构建响应式 Web 服务的编程模型，开发人员有两种选择。第一种是使用基于 Java 注解的方式，而第二种则是使用函数式编程模型。

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public Mono<String> hello() {
        return Mono.just("Hello World");
    }
}
```

以上代码只有一个地方值得注意，即 hello() 方法的返回值从普通的 String 对象转化为了一个 Mono 对象。这点是完全可以预见的，使用 Spring WebFlux 与 Spring MVC 的不同之处，在于前者使用的类型都是 Reactor 中提供的 Flux 和 Mono 对象，而不是普通的 POJO。

我们知道传统的 Spring MVC 构建在 Java EE 的 Servlet 标准之上，该标准本身就是阻塞式和同步的。而 Spring WebFlux 则是构建在响应式流以及它的实现框架 Project Reactor 之上的一个开发框架，因此可以基于 HTTP 协议来构建异步非阻塞的 Web 服务。

最后，我们来看一下位于底部的容器支持。当你使用 Spring WebFlux 时，你会注意到它默认采用了 Netty 作为运行时容器。这是因为 Spring MVC 是运行在传统的 Servlet 容器之上，而 Spring WebFlux 则需要支持异步的运行环境，比如 Netty、Undertow 以及 Servlet 3.1 之上的 Tomcat 和 Jetty。这里多说一句，因为在 Servlet 3.1 中引入了异步 I/O 支持。

## 响应式编程

新的响应式流式 API 由 Netflix，Pivotal，Lightbend，RedHat，Twitter 和 Oracle 等工程师创建，现已成为 Java 9 的一部分。它定义了四个接口：

- Publisher：根据从订阅者收到的需求向订阅者发出一系列事件。

  -  一个发布者可以为多个订阅者提供服务。它只有一个方法：Publisher.java

    ```java
    public interface Publisher<T> 
    {
    	public void subscribe(Subscriber<? super T> s);
    }
    ```

- Subscriber：接收和处理发布者发出的事件。 请注意，在调用`Subscription#request(long)`发出信号表示需求之前，不会收到任何通知。有四种方法处理各种响应：

  - ```java
    public interface Subscriber<T> 
    {
    	public void onSubscribe(Subscription s);
    	public void onNext(T t);
    	public void onError(Throwable t);
    	public void onComplete();
    }
    ```

- Subscription：定义“发布者”和“订阅者”之间的一对一关系。 只能由单个“订阅者”使用一次。 它既用于表示对数据的需求，又用于取消需求（并允许清除资源）。

  - ```java
    public interface Subscription<T> 
    {
    	public void request(long n);
    	public void cancel();
    }
    ```

- Processor：表示由“订阅者”和“发布者”组成的处理阶段，并服从两者的契约。

  - ```java
    public interface Processor<T, R> extends Subscriber<T>, Publisher<R> 
    {
    }
    ```

Spring Webflux 使用项目反应器作为反应库。 Reactor 是 Reactive Streams 库，因此，它的所有运算符都支持无阻塞背压。 它是与 Spring 紧密合作开发的。

Spring WebFlux 大量使用两个发布者：

- `Mono`：返回 0 或 1 个元素。

  ```java
  Mono<String> mono = Mono.just("Alex");
  Mono<String> mono = Mono.empty();
  ```

- `Flux`：返回`0…N`个元素。 `Flux`可以是无穷无尽的，这意味着它可以永远保持发光。 它还可以返回一系列元素，然后在返回所有元素后发送完成通知。

  ```java
  Flux<String> flux = Flux.just("A", "B", "C");
  Flux<String> flux = Flux.fromArray(new String[]{"A", "B", "C"});
  Flux<String> flux = Flux.fromIterable(Arrays.asList("A", "B", "C"));
  
  //To subscribe call method
  
  flux.subscribe();
  ```

在 Spring WebFlux 中，我们称为响应式 API /函数，它们返回 mono 和 flux，而您的控制器将返回 mono 和 flux。 当您调用返回单声道或流量的 API 时，它将立即返回。 当它们可用时，函数调用的结果将通过单声道或磁通传递给您。

> 要构建真正的非阻塞应用程序，我们必须致力于将其所有组件创建/使用为非阻塞程序，即客户端，控制器，中间服务甚至数据库。 如果其中之一阻止了请求，我们的目标将会失败。



## Reference

- https://segmentfault.com/a/1190000021038373
- https://www.kancloud.cn/apachecn/howtodoinjava-zh/1952700
- 官网：https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html
- 响应式编程实现：https://projectreactor.io/