+++
title = "Java Jersey Client 取得每個 Request Duration"
author = "Allen Hsieh"
description = "這新專案我選擇使用了之前沒使用過的 Jersey Client~ 由於希望分析打到其他服務的 P99 latency，所以我研究了一下怎麼去 log 下來所有的 duration (如果不知道什麼是 P99 latency 可以參考這篇文章 “What is P99 latency?“)。這邊分享一下，我最後使用的方法，由於不太熟 Jersey Client，所以最後也是花了一點時間才找到使用 ClientRequestFilter & ClientResponseFilter 來解決。"
featured = true
categories = ["JAVA"]
tags = [
]
date = "2021-06-30"
aliases = ["java-jersey-client-log-request-duration"]
images = ["images/java.jpeg"]
+++


這新專案我選擇使用了之前沒使用過的 Jersey Client ~ 由於希望分析打到其他服務的 P99 latency，所以我研究了一下怎麼去 log 下來所有的 duration (如果不知道什麼是 P99 latency 可以參考這篇文章 "[What is P99 latency](https://stackoverflow.com/questions/12808934/what-is-p99-latency)")。這邊分享一下，我最後使用的方法，由於不太熟 Jersey Client，所以最後也是花了一點時間才找到使用 ClientRequestFilter & ClientResponseFilter 來解決。

## 前置作業
---

這邊使用 Maven Project 作為範例

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>jerseyClient</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.core</groupId>
            <artifactId>jersey-client</artifactId>
            <version>2.33</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.inject</groupId>
            <artifactId>jersey-hk2</artifactId>
            <version>2.28</version>
        </dependency>
    </dependencies>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

</project>
```

```java
package com.demo;

import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.client.WebTarget;
import javax.ws.rs.core.MediaType;
import java.util.concurrent.TimeUnit;

public class MAIN {
    public static void main(String[] args) {
        Client client = ClientBuilder.newBuilder()
                .connectTimeout(30, TimeUnit.SECONDS)
                .build();
        //Filter 在後面的範例中
        WebTarget webTarget = client.target("https://google.com").register(new Filter());
        webTarget
                .request(MediaType.APPLICATION_JSON_TYPE)
                .get(String.class);
    }
}
```

## ClientResponseFilter
---

實作 Interface javax.ws.rs.client.ClientResponseFilter 可以處理所有的每個 Response 的結果 ~ 可以拿到 ClientRequestContext & ClientRequestContext，以下是範例。


```java
package com.demo;

import javax.ws.rs.client.ClientRequestContext;
import javax.ws.rs.client.ClientResponseContext;
import javax.ws.rs.client.ClientResponseFilter;
import java.io.IOException;

public class Filter implements ClientResponseFilter {
    @Override
    public void filter(ClientRequestContext requestContext,
                       ClientResponseContext responseContext) throws IOException {
        System.out.println(String.format("requestUri=\"%s\",  method=\"%s\", responseStatus=\"%s\", requestTime=\"%s\", responseTime=\"%s\"",
                requestContext.getUri(),
                requestContext.getMethod(),
                responseContext.getStatus(),
                requestContext.getDate(),
                responseContext.getDate()));
    }
}
```

這邊可以看到，ClientRequestContext ＆ ClientResponseContext 都有 getDate() 的函式返回一個 java.util.Date！天真的我一開始以為可以靠這兩個 getDate() 算出這個 response 的 duration，但實際執行結果卻出乎我的意料，requestTime 竟然是 null。

```bash
requestUri="https://google.com",  method="GET", responseStatus="200", requestTime="null", responseTime="Wed Jun 30 22:41:53 CST 2021"
```

## ClientRequestFilter
---

當時有點犯蠢，想說 requestTime 是 Null 就差點放棄，不知道怎麼完全沒想到竟然有 ClientResponseFilter ，那怎麼會沒有 ClientRequestFilter 呢！所以這邊最後靠著 ClientRequestFilter ，在每個 request 送出去前，把系統當下時間塞進去 requestContext ，這樣就可以在 response 回來時拉到當初 request 送出時間。

```java
package com.demo;

import javax.ws.rs.client.ClientRequestContext;
import javax.ws.rs.client.ClientRequestFilter;
import javax.ws.rs.client.ClientResponseContext;
import javax.ws.rs.client.ClientResponseFilter;
import java.io.IOException;

public class Filter implements ClientResponseFilter, ClientRequestFilter {
    private static final String REQUEST_TIME = "REQUEST_TIME";

    @Override
    public void filter(ClientRequestContext requestContext) throws IOException {
        long currentTimeMillis = System.currentTimeMillis();
        requestContext.setProperty(REQUEST_TIME, currentTimeMillis);
    }
    
    @Override
    public void filter(ClientRequestContext requestContext,
                       ClientResponseContext responseContext) throws IOException {
        long requestTime = (Long) requestContext.getProperty(REQUEST_TIME);
        long currentTimeMillis = System.currentTimeMillis();
        long duration = currentTimeMillis - requestTime;

        System.out.println(String.format("requestUri=\"%s\",  method=\"%s\", responseStatus=\"%s\", duration=\"%d\"",
                requestContext.getUri(),
                requestContext.getMethod(),
                responseContext.getStatus(),
                duration));
    }
}
```

透過 ClientResponseFilter & ClientRequestFilter 就可以簡單算出每個 request duration ~ 希望有幫助到看到這邊文章的人


