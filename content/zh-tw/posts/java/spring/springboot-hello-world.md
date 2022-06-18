+++
title = "透過 Spring CLI 建立 Spring Boot Hello World"
author = "Allen Hsieh"
description = "前一陣子公司專案讓我有機會使用 Spring Boot 開發，但因為當時專案時間很趕，所以也一直專注在開發上當時開發所需要的技術。由於不熟 Spring Boot or Spring Core，所以用的東西都很基礎。所以我想趁現在好好的充實自己並嘗試寫一個 Spring Boot 系列的文章。"
featured = true
categories = ["JAVA", "SPRING"]
tags = [
"SPRING"
]
date = "2022-06-08"
aliases = ["springboot-hello-world"]
images = ["images/spring.png"]
+++

前一陣子公司專案讓我有機會使用 Spring Boot 開發，但因為當時專案時間很趕，所以也一直專注在開發上當時開發所需要的技術。由於不熟 Spring Boot or Spring Core，所以用的東西都很基礎。所以我想趁現在好好的充實自己並嘗試寫一個 Spring Boot 系列的文章。

## 環境確認
---
這系列的文章，會以以下的版本為主
- Java 11
- Spring Boot 2.7
- Apache Maven 3.8.2



## 建立第一個 Hello World
---
由於艾倫以前已經使用過 IDE 或 網頁版的 Spring Boot Initializr 初始化，所以這次會以還未使用過的 Spring CLI。

### 安裝 Spring CLI
---
```bash
$ brew tap spring-io/tap
$ brew install springboot

$ spring --version
Spring CLI v2.7.0
```
### 使用 Spring CLI 初始化專案
---
```bash 
$ spring init -dweb -j 11 -v 1.0.0 -a helloworld
Using service at https://start.spring.io
Content saved to 'helloworld.zip'
```
這邊對參數做一些小解釋
- -d: 是 Dependencies，指定專案所需要的 packages，目前先選擇最基本的 web starter
- -j: 是 Java 版本，這邊我們選擇 Java 11
- -v: 此專案的版本初始化
- -a: 是 artifact Id

更多的參數或者 dependencies 的選擇，可以透過 spring init -list 看到。

### 執行 Spring Boot 
---
建立一個新的資料夾如 demo，將剛剛 Spring CLI 所產生的 ZIP 檔案放入資料夾中，並解壓縮。<br>
在 Terminal 中移動到那個 folder 並在裡面執行 "./mvnw spring-boot:run" ，最後可以看到以下畫面
```bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.0)

2022-06-12 22:08:28.328  INFO 85059 --- [           main] com.example.helloworld.DemoApplication   : Starting DemoApplication using Java 17 on ChangdeMacBook-Pro.local with PID 85059 (/Users/allen/helloworld/target/classes started by allen in /Users/allen/helloworld)
2022-06-12 22:08:28.331  INFO 85059 --- [           main] com.example.helloworld.DemoApplication   : No active profile set, falling back to 1 default profile: "default"
2022-06-12 22:08:29.023  INFO 85059 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2022-06-12 22:08:29.035  INFO 85059 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2022-06-12 22:08:29.035  INFO 85059 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.63]
2022-06-12 22:08:29.107  INFO 85059 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-06-12 22:08:29.107  INFO 85059 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 721 ms
2022-06-12 22:08:29.395  INFO 85059 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2022-06-12 22:08:29.404  INFO 85059 --- [           main] com.example.helloworld.DemoApplication   : Started DemoApplication in 1.627 seconds (JVM running for 1.887)
```

從上面的訊息來看，可以看到 Spring Application 已經用 Tomcat 起來，且正在聽 8080 port。

這時我們嘗試打 8080 port 

```bash
 ~  curl http://127.0.0.
{"timestamp":"2022-06-12T14:10:55.321+00:00","status":404,"error":"Not Found","path":"/"}
```

會得到 404 Error Msg，這是正常的，因為目前我們還沒有建立任何的 endpoint ～ 但至少確認 Tomcat 有起來。


## Spring CLI 產生檔案

我們可以看到 Spring CLI 已經幫我們產生了以下的檔案
```bash
$ tree
.
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── helloworld
    │   │               └── DemoApplication.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── com
                └── example
                    └── helloworld
                        └── DemoApplicationTests.java
```

### DemoApplication.java
我們可以看到 Spring Boot 的進入點 SpringApplication.run。

如果對 Spring Boot 啟動時做了什麼事情有興趣，可以看看這篇文章 [從 SpringBootApplication 談談 Spring Boot 啓動時都做了哪些事？](https://www.twblogs.net/a/5ef3f904efdbac7f8b1043f2)

```Java
package com.example.helloworld;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```

### Pom.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.7.0</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>helloworld</artifactId>
	<version>1.0.0</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>11</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
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
			</plugin>
		</plugins>
	</build>

</project>
```

從上面看到，我們繼承了 starter parent，且 dependencies 如預期裡面包含了 web starter 跟我們一開始初始化專案時所要求的一樣。

最後這邊可以看到，有裝 spring boot maven plugin，這是我們執行 mvn spring-boot:run 所需要的 plugin。


### Maven dependency tree

當我們下 "./mvnw dependency:tree" 時，我們可以看到整個專案所需要的所包含的 package，也可以看到 package 中所拉到的其他 package 或者我們叫 transitive dependencies。

以下面的結果為範例，我們可以看到 spring-boot-starter-web 裡面有包含 spring-boot-starter-tomcat。

```bash
$  ./mvnw dependency:tree
[INFO] com.example:helloworld:jar:1.0.0
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:2.7.0:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:2.7.0:compile
[INFO] |  |  +- org.springframework.boot:spring-boot:jar:2.7.0:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.7.0:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.7.0:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.2.11:compile
[INFO] |  |  |  |  \- ch.qos.logback:logback-core:jar:1.2.11:compile
[INFO] |  |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.17.2:compile
[INFO] |  |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.17.2:compile
[INFO] |  |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.36:compile
[INFO] |  |  +- jakarta.annotation:jakarta.annotation-api:jar:1.3.5:compile
[INFO] |  |  \- org.yaml:snakeyaml:jar:1.30:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-json:jar:2.7.0:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.13.3:compile
[INFO] |  |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.13.3:compile
[INFO] |  |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.13.3:compile
[INFO] |  |  +- com.fasterxml.jackson.datatype:jackson-datatype-jdk8:jar:2.13.3:compile
[INFO] |  |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.13.3:compile
[INFO] |  |  \- com.fasterxml.jackson.module:jackson-module-parameter-names:jar:2.13.3:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-tomcat:jar:2.7.0:compile
[INFO] |  |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:9.0.63:compile
[INFO] |  |  +- org.apache.tomcat.embed:tomcat-embed-el:jar:9.0.63:compile
[INFO] |  |  \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:9.0.63:compile
[INFO] |  +- org.springframework:spring-web:jar:5.3.20:compile
[INFO] |  |  \- org.springframework:spring-beans:jar:5.3.20:compile
[INFO] |  \- org.springframework:spring-webmvc:jar:5.3.20:compile
[INFO] |     +- org.springframework:spring-aop:jar:5.3.20:compile
[INFO] |     +- org.springframework:spring-context:jar:5.3.20:compile
[INFO] |     \- org.springframework:spring-expression:jar:5.3.20:compile
```

## 將 Tomcat 換成 Jetty 

從 dependency tree 我們已經看到 tomcat 是由 web start 拉入的，所以我們可以透過 exclusion 將 tomcat starter 從 web starter 中排除，並手動拉入 jetty starter。

```XML
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
		<exclusions>
			<!-- 去除Tomcat容器 -->
			<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<!-- 增加Jetty容器 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-jetty</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

這時我們重新執行 ./mvnw spring-boot:run 

```bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.0)

2022-06-12 22:49:19.224  INFO 87278 --- [           main] com.example.helloworld.DemoApplication   : Starting DemoApplication using Java 17 on ChangdeMacBook-Pro.local with PID 87278 (/Users/allen/helloworld/target/classes started by allen in /Users/allen/helloworld)
2022-06-12 22:49:19.226  INFO 87278 --- [           main] com.example.helloworld.DemoApplication   : No active profile set, falling back to 1 default profile: "default"
2022-06-12 22:49:19.740  INFO 87278 --- [           main] org.eclipse.jetty.util.log               : Logging initialized @1029ms to org.eclipse.jetty.util.log.Slf4jLog
2022-06-12 22:49:19.857  INFO 87278 --- [           main] o.s.b.w.e.j.JettyServletWebServerFactory : Server initialized with port: 8080
2022-06-12 22:49:19.860  INFO 87278 --- [           main] org.eclipse.jetty.server.Server          : jetty-9.4.46.v20220331; built: 2022-03-31T16:38:08.030Z; git: bc17a0369a11ecf40bb92c839b9ef0a8ac50ea18; jvm 17+0
2022-06-12 22:49:19.896  INFO 87278 --- [           main] o.e.j.s.h.ContextHandler.application     : Initializing Spring embedded WebApplicationContext
2022-06-12 22:49:19.897  INFO 87278 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 633 ms
2022-06-12 22:49:19.958  INFO 87278 --- [           main] org.eclipse.jetty.server.session         : DefaultSessionIdManager workerName=node0
2022-06-12 22:49:19.958  INFO 87278 --- [           main] org.eclipse.jetty.server.session         : No SessionScavenger set, using defaults
2022-06-12 22:49:19.959  INFO 87278 --- [           main] org.eclipse.jetty.server.session         : node0 Scavenging every 600000ms
2022-06-12 22:49:19.966  INFO 87278 --- [           main] o.e.jetty.server.handler.ContextHandler  : Started o.s.b.w.e.j.JettyEmbeddedWebAppContext@69da0b12{application,/,[file:///private/var/folders/dx/18qvmm_55hz74r3xdx9wn6x40000gn/T/jetty-docbase.8080.12428220257398218881/],AVAILABLE}
2022-06-12 22:49:19.966  INFO 87278 --- [           main] org.eclipse.jetty.server.Server          : Started @1256ms
2022-06-12 22:49:20.187  INFO 87278 --- [           main] o.e.j.s.h.ContextHandler.application     : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-06-12 22:49:20.187  INFO 87278 --- [           main] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-06-12 22:49:20.188  INFO 87278 --- [           main] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
2022-06-12 22:49:20.219  INFO 87278 --- [           main] o.e.jetty.server.AbstractConnector       : Started ServerConnector@17ca8b92{HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
2022-06-12 22:49:20.220  INFO 87278 --- [           main] o.s.b.web.embedded.jetty.JettyWebServer  : Jetty started on port(s) 8080 (http/1.1) with context path '/'
2022-06-12 22:49:20.230  INFO 87278 --- [           main] com.example.helloworld.DemoApplication   : Started DemoApplication in 1.274 seconds (JVM running for 1.52)
```

我們可以看到 Application 現在由 Jetty 9.4 起來～
