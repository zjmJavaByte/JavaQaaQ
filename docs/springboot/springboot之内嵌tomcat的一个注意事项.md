通过`idea`或者`https://start.spring.io/`构建一个`springboot`项目，项目结构如下，其中`pom`内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.5.13</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.zjmByte</groupId>
	<artifactId>ArtConcurrentBook</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>ArtConcurrentBook</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>1.8</java.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<skipTests>true</skipTests>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>

```

我们可以看到，只有两个`dependency`，当我们通过`idea	`启动项目,发现启动后会自动暂停，如下所示：

![image-20220507225407700](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205072254759.png)

直接退出了！！！

然后百度搜索发现这位博主`https://blog.csdn.net/qq_46416934/article/details/124042525`给出了解决方法，但是没有说明问题原因。

然后我就加入如下依赖：

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

发现项目启动成功了！！！

​		于是我就寻找原因，经过一番研究，我发现和`springboot`自带的`tomcat`有关系，然后我将依赖改为如下：

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-tomcat</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
```

项目有自动退出，所以问题的原因就死`spring-boot-starter`并不代`tomcat`	容器，而是`spring-boot-starter-web`带有`tomcat`。