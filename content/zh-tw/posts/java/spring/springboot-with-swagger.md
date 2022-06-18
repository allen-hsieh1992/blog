+++
title = "Spring Boot Hello World"
author = "Allen Hsieh"
description = "前一陣子公司專案讓我有機會使用 Spring Boot 開發，但因為當時專案時間很趕，所以也一直專注在開發上當時開發所需要的技術。由於不熟 Spring Boot or Spring Core，所以用的東西都很基礎。所以我想趁現在好好的充實自己並嘗試寫一個 Spring Boot 系列的文章。"
featured = true
categories = ["JAVA", "SPRING"]
tags = [
"SPRING"
]
date = "2022-06-08"
aliases = ["springboot-with-swagger"]
images = ["images/spring.png"]
draft=true
+++

上次分享了如何透過 [Spring CLI 建立 Hello World Application]({{< relref "/content/zh-tw/posts/java/springboot-hello-world.md" >}})

這邊文章將延續產生 status endpoint 和產生 Swagger 頁面


## 建立第一個 Endpoints
---
建立一個新的 package 並建立 statusController.java

```Java
package com.example.springboot_demo.contoller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController //1
public class statusController {

    @RequestMapping("/status") //2
    public String  status() {
        return "ok";
    }
}
```

備註1 : RestController 等於 @Controller + @ResponseBody， 
- ResponseBody: Spring 會自動幫你把 Response 轉換成 Json 格式
- Controller: 告訴 Spring 這是一個 request 進入點的 Class ，負責調度請求(RequestDispatcher) 

備註2 : RequestMapping 告訴 SpringBoot endpoint 設定，如 path 、 http method 等等，後面會提到更多，目前先知道當打 /status 這個 endpoint 時，SpringBoot 會找到這個函式並執行。


### 測試

現在將 Web Server 啟動，並打 status endpoint，我們會得到我們剛剛所寫的 "OK"

```bash
$ curl http:/127.0.0.1:8080/status
ok
```


