+++
title = "Build AWS Lambda Hello World Function By SAM"
author = "Allen Hsieh"
description = "以前當我第一次接觸 AWS Lambda Function 時，是在上 AWS Training 課程，使用 AWS Console 建立了一隻 Hello World。而 SAM (Serverless Application Model) 是一個開源的程式，主要在 CloudFormation 的基礎上擴展，專門用來開發 Serverless Application。我們可以透過 YAML 去定義我們的系統架構 像是 Endpoint 由哪個handler 處理，額外的 database (DynamoDB) 等等。這次的文章，主要是介紹使用 SAM 建立一個 Java Hello World 的 Lambda Function。"
featured = true
categories = ["AWS"]
tags = [
    "Lambda",
    "CloudFormation",
    "SAM",
]
date = "2021-07-03"
aliases = ["build-aws-lambda-hello-world-function-by-sam"]
images = ["images/aws.jpeg"]
+++

以前當我第一次接觸 AWS Lambda Function 時，是在上 AWS Training 課程，使用 AWS Console 建立了一隻 Hello World。而 SAM (Serverless Application Model) 是一個開源的程式，主要在 CloudFormation 的基礎上擴展，專門用來開發 Serverless Application。我們可以透過 YAML 去定義我們的系統架構 像是 Endpoint 由哪個handler 處理，額外的 database (DynamoDB) 等等。這次的文章，主要是介紹使用 SAM 建立一個 Java Hello World 的 Lambda Function。


## 事前準備
---

1. 一個 AWS 帳號
2. 安裝 AWS CLI 並 設定好，SAM 的權限是根據 AWS CLI喔！所以 SAM 不用額外設定 Access Key ID & Secret Access Key
   - Mac & Linux 可以透過 pip 安裝，其他可以參考這篇[官網](https://docs.aws.amazon.com/zh_tw/cli/latest/userguide/cli-chap-install.html)安裝教學
   - 如果不知道如何獲取 Access Key ID & Secret Access Key 可以參考這篇[官網](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/id_credentials_access-keys.html)設定

```bash
# 安裝 AWS CLI
$ pip install awscli

# 確認 AWS CLI 安裝成功
$ aws --version
 aws-cli/1.19.105 Python/2.7.16 Darwin/20.4.0 botocore/1.20.105

 # AWS CLI 設定
 $ aws configure
 AWS Access Key ID [****************ASFC]:
 AWS Secret Access Key [****************fM7z]:
 Default region name [ap-northeast-1]:
 Default output format [json]:
```

3. Java11 JDK & Maven，這邊可以透過 Brew 安裝，如何安裝 Brew 看官網教學

```bash
# 安裝 open jdk11
$ brew install openjdk@11
# 確認 Java JDK 安裝成功  
$ java -version 
 openjdk version "11.0.10" 2021-01-19
 OpenJDK Runtime Environment (build 11.0.10+9)
 OpenJDK 64-Bit Server VM (build 11.0.10+9, mixed mode) 

# 安裝 maven
$ brew install maven
# 確認 Maven 安裝成功  
$ mvn -version
 Apache Maven 3.8.1 (05c21c65bdfed0f71a2f2ada8b84da59348c4c5d)
 Maven home: /usr/local/Cellar/maven/3.8.1/libexec
 Java version: 11.0.10, vendor: Oracle Corporation, runtime:   /usr/local/Cellar/openjdk@11/11.0.10/libexec/openjdk.jdk/Contents/Home
 Default locale: zh_TW_#Hant, platform encoding: UTF-8
 OS name: "mac os x", version: "11.3.1", arch: "x86_64", family: "mac" 
```

4. Docker ，由於 Docker 安裝比較麻煩，這邊就不寫安裝方式，我個人 MAC 是安裝 [Docker Desktop](https://docs.docker.com/desktop/mac/install/)

```bash
$ docker --version
 Docker version 20.10.7, build f0df350 
```

5. 今日主角，SAM CLI，這邊也是透過 Brew 去安裝

```bash
# 安裝 SAM CLI 
$ brew tap aws/tap
$ brew install aws-sam-cli
# 確認 SAM CLI 安裝成功
$ sam --version
 SAM CLI, version 1.25.0 
```


## SAM 初始化專案
---

```bash
$ sam init -r java11 -d maven --app-template hello-world -n demo 

# -r :  --runtime 的縮寫，執行環境的語言，這邊指定 java11 
# -d : --dependency-manager 的縮寫(軟體套件管理系統)，這邊指定 maven 
# --app-template : 官方已經有寫好一些 template 提供大家使用
# -n : --name 的縮寫，這個專案的名稱

# 如果想瞭解更多參數，和每個參數可以設定的選項，可以使用 
$ sam init help
```

透過 SAM 初始化，會建立以下的檔案

```bash
$ tree
 .
 ├── HelloWorldFunction
 │   ├── pom.xml
 │   └── src
 │       ├── main
 │       │   └── java
 │       │       └── helloworld
 │       │           └── App.java
 │       └── test
 │           └── java
 │               └── helloworld
 │                   └── AppTest.java
 ├── README.md
 ├── events
 │   └── event.json
 └── template.yaml 
```

Template.yaml 是我們設定 Resource 相關的資訊，由於太多這篇就不一一介紹，這邊會在產生出來的 Code 加一些簡單註解

```YAML
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  demo

  Sample SAM Template for demo

Globals:
  Function:
    Timeout: 20

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function # 這邊是指 Lambda Function
    Properties:
      CodeUri: HelloWorldFunction
      Handler: helloworld.App::handleRequest # 這邊指定 Handler
      #helloworld : java package
      #App : class name
      #handleRequest : handle request method
      Runtime: java11
      MemorySize: 512
      Environment: # 可以設定系統環境的地方
        Variables:
          PARAM1: VALUE
      Events: # Trigger 這個 Lambda Function 的 Event
        HelloWorld:
          Type: Api # Event 是由 API Gateway 來的
          Properties:
            Path: /hello # Api Gateway Endpoint 
            Method: get  # Http Method 

#Outputs 就請參考 CloudFormation
Outputs:
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
  HelloWorldFunction:
    Description: "Hello World Lambda Function ARN"
    Value: !GetAtt HelloWorldFunction.Arn
  HelloWorldFunctionIamRole:
    Description: "Implicit IAM Role created for Hello World function"
    Value: !GetAtt HelloWorldFunctionRole.Arn
```

這邊順便介紹主要處理邏輯的 Handler

```java
package helloworld;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URL;
import java.util.HashMap;
import java.util.Map;
import java.util.stream.Collectors;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyResponseEvent;

/**
 * Handler for requests to Lambda function.
 */
public class App implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {

    public APIGatewayProxyResponseEvent handleRequest(final APIGatewayProxyRequestEvent input, final Context context) {
        // response 的 header
        Map<String, String> headers = new HashMap<>();
        headers.put("Content-Type", "application/json");
        headers.put("X-Custom-Header", "application/json");

        APIGatewayProxyResponseEvent response = new APIGatewayProxyResponseEvent()
                .withHeaders(headers);
        try {
            // call amanzon api 取得系統 IP
            final String pageContents = this.getPageContents("https://checkip.amazonaws.com");
            
            // response body
            String output = String.format("{ \"message\": \"hello world\", \"location\": \"%s\" }", pageContents);

            return response
                    .withStatusCode(200)
                    .withBody(output);
        } catch (IOException e) {
            return response
                    .withBody("{}")
                    .withStatusCode(500);
        }
    }

    private String getPageContents(String address) throws IOException{
        URL url = new URL(address);
        try(BufferedReader br = new BufferedReader(new InputStreamReader(url.openStream()))) {
            return br.lines().collect(Collectors.joining(System.lineSeparator()));
        }
    }
}
```

所以從以上來看，這邊就是透過 Template.yaml 去指定 環境執行語言， API Endpoint 和 Handler 等等重要資訊。而 App.java 就是我們 Lambda Function 主要計算邏輯。目前透過這兩個 File 就可以產生出基本的 get hello 函示。

這邊要注意，hello-world Template 建立出的 pom 檔，裡面指定的 Compile Source 是 1.8 ，這邊建議改成 Java 11

```xml
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
</properties>
```

## SAM 本地測試
---

SAM 我個人覺得最方便的就是，它使用了 Docker 的技術，讓我們可以在本地也可以建立起一個環境做測試 API


```bash
$ sam local start-api
  
 Mounting HelloWorldFunction at http://127.0.0.1:3000/hello [GET]
 You can now browse to the above endpoints to invoke your functions. You do not need to restart/reload SAM CLI while working on your functions, changes will be reflected instantly/automatically. You only need to restart SAM CLI if you update your AWS SAM template
 2021-07-03 19:40:16  * Running on http://127.0.0.1:3000/ (Press CTRL+C to quit) 
```

從上面的 Output 可以看到，起了一個環境，並聽本地的3000 port，這邊會在開另一個terminal 來測試 curl 來測試結果

```bash
$ curl -v  http://127.0.0.1:3000/hello
 * Trying 127.0.0.1...
 * TCP_NODELAY set
 * Connected to 127.0.0.1 (127.0.0.1) port 3000 (#0)
 > GET /hello HTTP/1.1
 > Host: 127.0.0.1:3000
 > User-Agent: curl/7.64.1
 > Accept: */*
 > 
 * HTTP 1.0, assume close after body
 < HTTP/1.0 200 OK
 < X-Custom-Header: application/json
 < Content-Type: application/json
 < Content-Length: 57
 < Server: Werkzeug/1.0.1 Python/3.8.11
 < Date: Sat, 03 Jul 2021 11:47:10 GMT
 < 
 * Closing connection 0
 { "message": "hello world", "location": "114.44.114.30" }
```

在原本的 Terminal 可以看到以下資訊


```bash
Mounting /Users/allen/demo/.aws-sam/build/HelloWorldFunction as /var/task:ro,delegated inside runtime container
 END RequestId: 74bd804b-096b-473b-9c3f-2ba1cd847431
 REPORT RequestId: 74bd804b-096b-473b-9c3f-2ba1cd847431 Init Duration: 0.37 ms Duration: 2255.80 ms Billed Duration: 2300 ms Memory Size: 512 MB Max Memory Used: 512 MB 
 2021-07-03 19:44:13 127.0.0.1 - - [03/Jul/2021 19:44:13] "GET /hello HTTP/1.1" 200 - 
```

Sam 也可以直接一次性的執行某個 Function

```bash
$ sam local invoke "HelloWorldFunction"  
 Invoking helloworld.App::handleRequest (java11)
 Skip pulling image and use local one: amazon/aws-sam-cli-emulation-image-java11:rapid-1.25.0.
 

 Mounting /Users/allen/demo/.aws-sam/build/HelloWorldFunction as /var/task:ro,delegated inside runtime container
 END RequestId: 70223332-2a37-4094-bdf2-aa924df0d416
 REPORT RequestId: 70223332-2a37-4094-bdf2-aa924df0d416 Init Duration: 0.29 ms Duration: 2153.55 ms Billed Duration: 2200 ms Memory Size: 512 MB Max Memory Used: 512 MB 
 {"statusCode":200,"headers":{"X-Custom-Header":"application/json","Content-Type":"application/json"},"body":"{ \"message\": \"hello world\", \"location\": \"114.44.114.30\" }"} 
```

## SAM 部署到 AWS
---
這邊可以透過 sam deploy –guided  Deploy 到 AWS 上面，由於內容資訊比較多，所以這邊用圖片的方式，在第一次執行時這邊會需要填寫一些預設資料。

![Center](/images/post/build-aws-lambda-hello-world-function-by-sam/sam_deploy.png#center)

在填完預設資料後，SAM會一直執行，直到要你確認這次 Deploy 的改動 CloudFormation 的 changeset

![Center](/images/post/build-aws-lambda-hello-world-function-by-sam/sam_changeset.png#center)

CloudFormation 這時候會卡在 REVIEW_IN_PROGRESS

![Center](/images/post/build-aws-lambda-hello-world-function-by-sam/cloudFormation_review_in_progress.png#center)

這時Change Set 同意以後，這時看 CloudFormation 會正在建立 Resource， Status 是 CREATE_IN_PROGRESS

![Center](/images/post/build-aws-lambda-hello-world-function-by-sam/cloudFormation_create_in_progress.png#center)

SAM CLI 在 Deploy 完以後，會看到類似以下資訊

![Center](/images/post/build-aws-lambda-hello-world-function-by-sam/sam_deploy_successfully.png#center)

可以在 API Gateway 看到新的 Endpoint

![Center](/images/post/build-aws-lambda-hello-world-function-by-sam/sam_created_api_gateway.png#center)

也可以在 Lambda 看到 HelloWorld Lambda Function 建立好了

![Center](/images/post/build-aws-lambda-hello-world-function-by-sam/sam_created_lambda_function.png#center)

```bash
$ curl -v https://rla530b9o1.execute-api.ap-northeast-1.amazonaws.com/Prod/hello/       
 * Trying 13.35.7.43...
 * TCP_NODELAY set
 * Connected to rla530b9o1.execute-api.ap-northeast-1.amazonaws.com (13.35.7.43) port 443 (#0)
 * ALPN, offering h2
 * ALPN, offering http/1.1
 * successfully set certificate verify locations:
 *   CAfile: /etc/ssl/cert.pem
   CApath: none
 * TLSv1.2 (OUT), TLS handshake, Client hello (1):
 * TLSv1.2 (IN), TLS handshake, Server hello (2):
 * TLSv1.2 (IN), TLS handshake, Certificate (11):
 * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
 * TLSv1.2 (IN), TLS handshake, Server finished (14):
 * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
 * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
 * TLSv1.2 (OUT), TLS handshake, Finished (20):
 * TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
 * TLSv1.2 (IN), TLS handshake, Finished (20):
 * SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
 * ALPN, server accepted to use h2
 * Server certificate:
 *  subject: CN=*.execute-api.ap-northeast-1.amazonaws.com
 *  start date: May 17 00:00:00 2021 GMT
 *  expire date: Jun 15 23:59:59 2022 GMT
 *  subjectAltName: host "rla530b9o1.execute-api.ap-northeast-1.amazonaws.com" matched cert's "*.execute-api.ap-northeast-1.amazonaws.com"
 *  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
 *  SSL certificate verify ok.
 * Using HTTP2, server supports multi-use
 * Connection state changed (HTTP/2 confirmed)
 * Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
 * Using Stream ID: 1 (easy handle 0x7f9298809200)
 > GET /Prod/hello/ HTTP/2
 > Host: rla530b9o1.execute-api.ap-northeast-1.amazonaws.com
 > User-Agent: curl/7.64.1
 > Accept: */*
 > 
 * Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
 

 < HTTP/2 200 
 < content-type: application/json
 < content-length: 58
 < date: Sat, 03 Jul 2021 14:28:53 GMT
 < x-amzn-requestid: fa2fb89a-5379-4167-b13c-0a59269f71e0
 < x-amz-apigw-id: B5cU7GWTNjMFc6A=
 < x-custom-header: application/json
 < x-amzn-trace-id: Root=1-60e0741f-7e93aac373f51300700bbc59;Sampled=0
 < x-cache: Miss from cloudfront
 < via: 1.1 260a465bf4779e35dde8adbb89981df1.cloudfront.net (CloudFront)
 < x-amz-cf-pop: TPE52-C1
 < x-amz-cf-id: aGy5WF0naNegGjNJM1kaZpBnl5SDcg23eZMy8wYUrd0sxyQnHLnhgg==
 < 
 * Connection #0 to host rla530b9o1.execute-api.ap-northeast-1.amazonaws.com left intact
 { "message": "hello world", "location": "13.231.109.123" }* Closing connection 0 
```

## 清理 Sam 建立好的 AWS 環境
---

如果大家有 Deploy 到 AWS 上，最後記得要把 CloudFormation 建立出來的環境給刪除，以免產生額外的費用。這邊可以直接到 AWS CloudFormation 把 Stack 給刪除，或者使用 AWS CLI

```bash
$ aws cloudformation delete-stack --stack-name demo  
```

這邊要注意一下， SAM 會建立一個 S3 當作 Artifact ，所以需要先把 S3 裡面的資料手動清掉才可以 delete stack，否則 stack 會刪除失敗。以下是S3 沒有清空時的範例圖

![Center](/images/post/build-aws-lambda-hello-world-function-by-sam/cloudformation_delete_fail.png#center)
