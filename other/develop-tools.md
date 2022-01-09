# Develop Tools

开发工具Intellij Idea

IntelliJ IDEA 是一个智能的上下文感知 IDE，可以使用 Java 和其他 JVM 语言（例如 Kotlin、Scala 和 Groovy）开发各种应用程序。 得益于强大的集成工具、对 JavaScript 和相关技术的支持以及对 Spring、Spring Boot、Jakarta EE、Micronaut、Quarkus、Helidon 等热门框架的高级支持，IntelliJ IDEA Ultimate 还可以帮助您开发全栈 Web 应用程序。 此外，您可以使用 JetBrains 开发的免费插件扩展 IntelliJ IDEA，这让您可以利用其他编程语言，包括 Go、Python、SQL、Ruby 和 PHP。
数据库工具Datagrip

DataGrip 一个是多引擎的数据库环境。 如果 DBMS 有 JDBC 驱动程序，那么您可以通过 DataGrip 连接它。 它可以提供用来为支持的引擎创建和修改对象所需的数据库内省和各种工具如：MySQL、PostgreSQL、MariaDB、Apache Hive、MongoDB、H2等等。

容器管理平台Rancher

Rancher是业界唯一完全开源的企业级容器管理平台，为企业用户提供在生产环境中落地使用容器所需的一切功能与组件。Rancher2.0基于Kubernetes构建。使用Rancher，DevOps团队可以轻松测试、部署和管理应用程序，运维团队可以部署、管理和维护一切Kubernetes集群，无论集群运行在何基础设施之上。

应用性能测试JMeter

  Apache JMeter是100%纯JAVA桌面应用程序，被设计为用于测试客户端/服务端结构的软件(例如web应用程序)。它可以用来测试静态和动态资源的性能，例如：静态文件，Java Servlet,CGI Scripts,Java Object,数据库和FTP服务器等等。JMeter可用于模拟大量负载来测试一台服务器，网络或者对象的健壮性或者分析不同负载下的整体性能。同时，JMeter可以帮助你对你的应用程序进行回归测试。通过你创建的测试脚本和assertions来验证你的程序返回了所期待的值。为了更高的适应性，JMeter允许你使用正则表达式来创建这些assertions.

## Git

### .gitignore文件

忽略文件的原则是：

1. 忽略操作系统自动生成的文件，比如缩略图等；
2. 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的`.class`文件；
3. 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

```.gitignore
#开头         #注释，被git忽略
*.class      #忽略所有.class结尾的文件
！Test.class  #Test.class除外
/bin/        #忽略bin目录下的所有文件
/conf/*.txt  #忽略/conf/1.txt但是不包括/conf/sub/2.txt
```

在项目根目录下增加.gitignore文件。

若在项目开发过程中后期想把某些文件加入忽略规则，按照上述方法定义后发现并未生效。原因是.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。

解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：

```
git rm -r --cached .
git add .
git commit -m 'update .gitignore'1.2.3.
```

## Java

### javadoc 标签

javadoc 工具软件识别以下标签：

| **标签**      |                        **描述**                        |                           **示例**                           |
| :------------ | :----------------------------------------------------: | :----------------------------------------------------------: |
| @author       |                    标识一个类的作者                    |                     @author description                      |
| @deprecated   |                 指名一个过期的类或成员                 |                   @deprecated description                    |
| {@docRoot}    |                指明当前文档根目录的路径                |                        Directory Path                        |
| @exception    |                  标志一个类抛出的异常                  |            @exception exception-name explanation             |
| {@inheritDoc} |                  从直接父类继承的注释                  |      Inherits a comment from the immediate surperclass.      |
| {@link}       |               插入一个到另一个主题的链接               |                      {@link name text}                       |
| {@linkplain}  |  插入一个到另一个主题的链接，但是该链接显示纯文本字体  |          Inserts an in-line link to another topic.           |
| @param        |                   说明一个方法的参数                   |              @param parameter-name explanation               |
| @return       |                     说明返回值类型                     |                     @return explanation                      |
| @see          |               指定一个到另一个主题的链接               |                         @see anchor                          |
| @serial       |                   说明一个序列化属性                   |                     @serial description                      |
| @serialData   | 说明通过writeObject( ) 和 writeExternal( )方法写的数据 |                   @serialData description                    |
| @serialField  |             说明一个ObjectStreamField组件              |              @serialField name type description              |
| @since        |               标记当引入一个特定的变化时               |                        @since release                        |
| @throws       |                 和 @exception标签一样.                 | The @throws tag has the same meaning as the @exception tag.  |
| {@value}      |         显示常量的值，该常量必须是static属性。         | Displays the value of a constant, which must be a static field. |
| @version      |                      指定类的版本                      |                        @version info                         |
