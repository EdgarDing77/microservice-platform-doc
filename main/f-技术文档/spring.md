# Spring

## Introduction

这里主要记载Spring的一些问题与细节。

## IoC

### 介绍

Spring框架提供的很重要的一个作用就是IoC容器，而这里的容器就是指为某种特定组件的运行提供一个必要的软件环境，例如Tomcat就是一个Servlet容器，可以为Servlet提供运行时。通常来说，使用容器运行组件，除了提供一个组件的运行时，还提供了许多底层服务，如Servlet的底层实现了TCP连接、HTTP协议解析等复杂服务，若没有容器来提供这些服务，也没法使我们简单的编写Servlet代码。

Spring的核心就是IoC容器，可以管理所有轻量级的JavaBean组件，提供底层服务包括组件的生命周期管理、配置和组装服务、AOP支持等。

### IoC原理

IoC（Inverse of Control：控制反转）是一种设计思想，**就是将原本在程序中手动创建对象的控制权，交给由Spring框架管理。**IoC容器是Spring用来实现IoC的载体，IoC容器实际上就是个Map，Map中存放的是各种对象。

将对象之间的相互依赖关系交给IoC容器管理，并由IoC容器完成对象的注入。即很大程度的简化了应用的开发，把应用从复杂的依赖关系中解放出来。**IoC容器就像一个工厂，当我们需要创建一个对象的时候，只需要配置好配置文件/注解，完全不用考虑对象如何被创建。**

>  在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。
>
>  Spring 时代我们一般通过 XML 文件来配置 Bean，后来开发人员觉得 XML 文件来配置不太好，于是 SpringBoot 注解配置就慢慢开始流行起来。

#### 依赖倒置原理

把原本的高层建筑依赖底层建筑“倒置”过来，变成底层建筑依赖高层建筑。高层建筑决定需要什么，底层去实现这样的需求，但是高层并不用管底层是怎么实现的。这样就不会出现前面的“牵一发动全身”的情况。

**控制反转（Inversion of Control）** 就是**依赖倒置**原则的一种代码设计的思路。具体采用的方法就是所谓的**依赖注入（Dependency Injection）**，它解决的最主要的一个问题就是：将组件的创建+配置与组件的使用分离，并且，由IoC容器负责管理组件生命周期。

#### 核心思想

**Inverseion of Control的最核心的思想就是，资源不由使用资源的双方管理，而由不使用资源的第三方管理。**

这里有个很棒的例子：

就是甲方要达成某种目的不需要直接依赖乙方，只需要将目的告诉给第三个。例如甲方需要一台电脑，乙方卖一台电脑，乙方要将电脑卖出去也不需要找到甲方，也只需要将电脑寄托到淘宝（电商平台）去告诉我要卖一台电脑。这样甲乙双方进行交易活动，都不需要自己去找互相，相当于程序内部开放接口，而卖家由第三方作为参数传入。甲乙不相互依赖，只有在交易的时候产生联系，反之亦然，解决了甲乙直接进行交易之间的不信任，因为交易由第三方负责联系。

- 而所谓的依赖注入，就是甲方开发接口，在需要的时候，将乙方注入进来。

- 而所谓的控制反转，就是甲乙双方不互相依赖，交易活动的进行不依赖于互相，整个活动的进行全部由第三方负责管理。

### 使用

Spring 作者 Rod Johnson 设计了两个接口用以表示容器。

- **BeanFactory**
- **ApplicationContext**

`BeanFactory` 粗暴简单，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 “**低级容器**”。

`ApplicationContext` 可以称之为 “**高级容器**”。因为他比 BeanFactory 多了更多的功能。他继承了多个接口。因此具备了更多的功能。例如资源的获取，支持多种消息（例如 JSP tag 的支持），对 BeanFactory 多了工具级别的支持等待。所以你看他的名字，已经不是 BeanFactory 之类的工厂了，而是 “应用上下文”， 代表着整个大容器的所有功能。该接口定义了一个 refresh 方法，此方法是所有阅读 Spring 源码的人的最熟悉的方法，用于刷新整个容器，即重新加载/刷新所有的 bean。



这里只会提及注解的方式：

- @Autowired：使用`@Autowired`就相当于把指定类型的Bean注入到指定的字段中
  - @Autowired(required = false)：Spring如果没有找到对应类型的Bean，它会抛出`NoSuchBeanDefinitionException`异常，可以通过该方式，若找不到就忽略
- @Compoent：相当于定义了一个Bean
  - @Scope：默认为单例，可以修改成`@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)`，即`@Scope("prototype")`
- @CompoentScan：过把所有标注为`@Component`的Bean自动创建出来，并根据`@Autowired`进行装配
- @Configuration：标注是一个配置类
- @Bean：通过在`@Configuration`的配置类中创建第三方Bean
  - @Qualifier(“name”)：若Bean的类型相同，而容器只会创建一个实例，就需要使用别名来进行区分：`@Bean("name")`，这时候在使用的时候需要通过`@Qualifier("name")`来指定注入的Bean
  - @Primary：或者通过该注解设置为主要Bean
- @Order：场景：有些时候，我们会有一系列接口相同，不同实现类的Bean。因为Spring是通过扫描classpath获取所有的Bean，这时候可以通过该注解定义Bean添加顺序。
- @PostContruct和`init()`：调用标记有`@PostConstruct`的`init()`方法进行初始化。
- @PreDestory和`shutdown()`：而销毁时，容器会首先调用标记有`@PreDestroy`的`shutdown()`方法。

## AOP

### 介绍

AOP(Aspect-Oriented Programming：面向切面编程)，通过**预编译方式和运行期动态代理**实现程序功能的统一维护的一种技术。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

**主要功能：让关注点代码与业务代码分离**。

**面向切面编程：对很多功能具有重复代码抽取，再在运行的时候往业务方法上动态注入“切面类代码”**。

在Java上，对于AOP的织入由三种方式：

1. 编译器：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并拓展编译器，AspectJ就扩展了Java编译器，使用关键字aspect来实现织入；
2. 类加载器：在目标类被装载到JVM时，通过一个特殊的类加载器，对目标类的字节码重新“增强”；
3. 运行期：目标对象和切面都是普通Java类，通过JVM的动态代理功能或者第三方库实现运行期动态织入。

最简单的就是第三种返回式，Spring的AOP实现就是基于JVM的动态代理。由于JVM的动态代理要求必须实现接口，如果一个普通类没有业务接口，就需要通过[CGLIB](https://github.com/cglib/cglib)或者[Javassist](https://www.javassist.org/)这些第三方库实现。

### 术语&案例

AOP中的常用术语：

- Aspect：切面，即一个横跨多个核心逻辑的功能，或者成为系统关注点
- Joinpoint：连接点，定义在应用程序流程的何处插入Aspect的执行
- Pointcut：切入点，即一组连接点的集合
- Advice：增强，指特定Joinpoint上执行的动作
- Introduction：引介，指一个已有的Java对象动态地增加新的接口
- Weaving：织入，将Aspect整合到程序的执行流中
- Interceptror：拦截器，一种实现Advice的方法
- Traget Object：目标对象，即真正执行业务的核心逻辑对象
- AOP Proxy：AOP代理，是客户端持有的增强后的对象引用

```java
@Aspect
@Component
public class LoggingAspect {
    // 在执行UserService的每个方法前执行:
    @Before("execution(public * com.itranswarp.learnjava.service.UserService.*(..))")
    public void doAccessCheck() {
        System.err.println("[Before] do access check...");
    }

    // 在执行MailService的每个方法前后执行:
    @Around("execution(public * com.itranswarp.learnjava.service.MailService.*(..))")
    public Object doLogging(ProceedingJoinPoint pjp) throws Throwable {
        System.err.println("[Around] start " + pjp.getSignature());
        Object retVal = pjp.proceed();
        System.err.println("[Around] done " + pjp.getSignature());
        return retVal;
    }
}
```

拦截器类型：

- @Before：这种拦截器先执行拦截代码，再执行目标代码。如果拦截器抛异常，那么目标代码就不执行了；
- @After：这种拦截器先执行目标代码，再执行拦截器代码。无论目标代码是否抛异常，拦截器代码都会执行；
- @AfterReturning：和@After不同的是，只有当目标代码正常返回时，才执行拦截器代码；
- @AfterThrowing：和@After不同的是，只有当目标代码抛出了异常时，才执行拦截器代码；
- @Around：能完全控制目标代码是否执行，并可以在执行前后、抛异常后执行任意拦截代码，可以说是包含了上面所有功能。

### 使用

- @Transactional：使用AOP时，被装配的Bean最好自己能清清楚楚地知道自己被安排了。例如，Spring提供的`@Transactional`就是一个非常好的例子。如果我们自己写的Bean希望在一个数据库事务中被调用，就标注上`@Transactional`。

## 常见注解

### @Transactional

添加位置：

1. 接口实现类 或 接口方法上 而不是接口类中
2. 访问权限：public的方法才起作用。Spring AOP的本质决定（**本质跟AOP调用有关：在事务代理对象中有判断代理的方法必须是public修饰，否则返回null**）。

系统设计：将标签放置在需要进行事务管理的方法上，而不是放在所有接口实现类上：只读的接口就不需要事务管理，由于配置了@Transactional就需要AOP拦截及事务的处理，可能影响系统性能。

错误使用：

> 1.接口中A、B两个方法，A无@Transactional标签，B有，上层通过A间接调用B，此时事务不生效。
>
> 2.接口中异常（运行时异常）被捕获而没有被抛出。
>   默认配置下，spring 只有在抛出的异常为运行时 unchecked 异常时才回滚该事务，
>   也就是抛出的异常为RuntimeException 的子类(Errors也会导致事务回滚)，
>   而抛出 checked 异常则不会导致事务回滚 。可通过 @Transactional rollbackFor进行配置。
>
> 3.多线程下事务管理因为线程不属于 spring 托管，故线程不能够默认使用 spring 的事务,也不能获取spring 注入的 bean 。
>   在被 spring 声明式事务管理的方法内开启多线程，多线程内的方法不被事务控制。
>   一个使用了@Transactional 的方法，如果方法内包含多线程的使用，方法内部出现异常，不会回滚线程中调用方法的事务。

@Transactional 实质是使用了 JDBC 的事务来进行事务控制的
@Transactional 基于 Spring 的动态代理的机制

> @Transactional 实现原理：
>
> 1) 事务开始时，通过AOP机制，生成一个代理connection对象，
>    并将其放入 DataSource 实例的某个与 DataSourceTransactionManager 相关的某处容器中。
>    在接下来的整个事务中，客户代码都应该使用该 connection 连接数据库，
>    执行所有数据库命令。
>    [不使用该 connection 连接数据库执行的数据库命令，在本事务回滚的时候得不到回滚]
>     （物理连接 connection 逻辑上新建一个会话session；
>    DataSource 与 TransactionManager 配置相同的数据源）
>
> 2) 事务结束时，回滚在第1步骤中得到的代理 connection 对象上执行的数据库命令，
>    然后关闭该代理 connection 对象。
>     （事务结束后，回滚操作不会对已执行完毕的SQL操作命令起作用）

## Spel

### 介绍

SpEL（Spring Expression Language），即Spring表达式语言，是比JSP的EL更强大的一种表达式语言。为什么要总结SpEL，因为它可以在运行时查询和操作数据，尤其是数组列表型数据，因此可以缩减代码量，优化代码结构。

具体使用参考[官方文档](https://docs.spring.io/spring-framework/docs/3.0.x/reference/expressions.html)。



## Reference

- https://www.liaoxuefeng.com/wiki/1252599548343744/1266265100383840
- IoC原理解析：https://mp.weixin.qq.com/s/0zRks2Cz36S8N70Uonb0OA
- HowToDoInJava：https://www.kancloud.cn/apachecn/howtodoinjava-zh/1952630
- Spel官方：https://docs.spring.io/spring-framework/docs/3.0.x/reference/expressions.html

