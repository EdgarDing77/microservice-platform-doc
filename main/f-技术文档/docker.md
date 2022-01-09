# Docker

## Introduction

**Docker** 是一个[开放源代码](https://zh.wikipedia.org/wiki/開放原始碼)[软件](https://zh.wikipedia.org/wiki/軟體)，是一个[开放平台](https://zh.wikipedia.org/wiki/開放平臺)，用于开发应用、交付（shipping）应用、运行应用。 Docker允许用户将基础设施（Infrastructure）中的应用单独分割出来，形成更小的颗粒（容器），从而提高交付软件的速度。

## docker命令

例子：

```bash
docker run -d -p 8080:8080 edgarding/microservice-platform-demo/djj-docker-demo
```

- -p：端口映射，格式为`{主机端口:容器端口}`
- -it：为容器分配伪输入终端(-t)，且以交互模式运行(-i)
- -d：后台运行容器

## 使用Docker部署Spring Boot

docker-maven-plugin的版本已经在根pom里管理了，所以工程不需要加版本号。

Maven插件导入：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.2.2</version>
    </plugins>
</build>
```

具体配置：

```xml
<plugin>
	<groupId>com.spotify</groupId>
	<artifactId>docker-maven-plugin</artifactId>
	<configuration>
		<imageName>${docker.image.prefix}/${project.artifactId}</imageName>
		<imageTags>
			<imageTag>${project.version}</imageTag>
<!--<imageTag>latest</imageTag>-->
		</imageTags>
		<forceTags>true</forceTags>
		<baseImage>${docker.baseImage}</baseImage>
		<volumes>${docker.volumes}</volumes>
		<env>
			<JAVA_OPTS>${docker.java.opts}</JAVA_OPTS>
		</env>
		<entryPoint>["java ","$JAVA_OPTS ${docker.java.security.egd} ","-jar ","/${project.build.finalName}.jar"]</entryPoint>
		<resources>
			<resource>
				<targetPath>/</targetPath>
				<directory>${project.build.directory}</directory>
				<include>${project.build.finalName}.jar</include>
			</resource>
		</resources>
	</configuration>
</plugin>
```

**`configuration`属性解释**

- imageName：为镜像名
- imageTags：配置镜像tag，这里指定了两个tag(最新的版本号和latest)
- forceTags：强制在每次新的构建上覆盖镜像tags
- baseImage：基础镜像
- volumes：配置挂载目录
- env：配置环境变量
- entryPoint：配置执行命令
- resources：配置目标jar包的位置

父工程下properties参数：

```xml
        <docker-maven-plugin.version>1.2.2</docker-maven-plugin.version>
        <docker.baseImage>openjdk:8-jre-alpine</docker.baseImage>
        <docker.volumes>/tmp</docker.volumes>
        <docker.image.prefix>edgarding/microservice-platform-demo</docker.image.prefix>
        <docker.java.security.egd>-Djava.security.egd=file:/dev/./urandom</docker.java.security.egd>
        <docker.java.opts>-Xms128m -Xmx128m</docker.java.opts>
```

**参数解释**

- docker.baseImage：为jre8的基础镜像
- docker.volumes：是在容器里挂载的目录
- docker.image.prefix：仓库地址/项目名
- docker.java.security.egd：加快随机数产生过程
- docker.java.opts：设置JVM启动参数

### 推送Docker镜像

1. 创建Repo，登录docker hub，创建如下docker hub repo：`edgarding/microservice-platform-demo:tagname`
2. 配置pom.xml

```xml
<configuration>
  <repository>longyonggang/config-server</repository>
  <tag>${project.version}</tag>
  <buildArgs>
    <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
  </buildArgs>
</configuration>
```

- repostiory格式：`{username}/{repo_name}`

3. 配置认证

## 使用

You can build an image with the above configurations by running this command.

```
mvn clean package docker:build
```

To push the image you just built to the registry, specify the `pushImage` flag.

```
mvn clean package docker:build -DpushImage
```

To push only specific tags of the image to the registry, specify the `pushImageTag` flag.

```
mvn clean package docker:build -DpushImageTag
```

1. 创建一个Docker DEV environment（可有可无，尚不支持Intellij Idea）
2. `docker login`
3. 执行`mvn docker:build`

这里注意，由于打包的是target，需要注意编译好先。

**注意：**

- 端口问题，处于容器网络内

## Reference

- https://github.com/spotify/docker-maven-plugin#usage
- https://docs.docker.com/docker-hub/access-tokens/
- https://www.jdon.com/57143
