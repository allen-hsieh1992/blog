+++
title = "Lambda Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 Lambda 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "Lambda",
]
date = "2021-09-19"
aliases = ["lambda-certification-note"]
images = ["images/aws.jpeg"]
+++

- Lambda 是 AWS 無伺服器的運算服務，以下幾點為 AWS 端的責任
    - Capacity Provisioning 
    - OS Patching
    - Auto Scaling
    - High Availability
- Basic Setting
    -   Memory usage 會影響 CPU 
    -   timeout 設定，最多十五分鐘
    -   role to execute the function  
-   收費根據以下三點
    -  Request 數量
    -  Lambda Function 處理 request 的時間
    -  執行 Lambda Function 的資源 Memory & CPU size  

    
## Components of Lambda
---
- Function: Source Code 存在 S3
- Runtimes: 執行環境
- Layers: 可以放 dependency file. 預設限制 75 GB and 5 Layers
- Event Source: 觸發 Lambda 的 Event  
- Downstream resources: 這個 Lambda Function 要 Access 的資源
- Log Streams: Lambda 已經整合 CloudWatch
- Environment variables
- Versions and aliases  

### Event Source
- Data Source
    - S3, DynamoDB, Kinesis, Cognito, Aurora
- Endpoint
    - API Gateway, Step Function, IOT
- Repository 
    - CloudWatch, CloudFormation, CloudTail, CodeCommit
- Message Service
    - SES, SNS, SQS 
    
### CloudWatch Log Streams
- Invocations: 五分鐘區間，Lambda 被建立的次數
- Duration: 執行時間，有提供平均、最大和最小  
- Error count and success rate (%): 成功和失敗比率
- Throttles: 多少個 request 失敗是因為瓶頸
- IteratorAge: For stream event sources, the age of the last item in the batch when Lambda received it and invoked the function.
- Async delivery failures:  非同步時多少 request 進入錯誤 和寫到 dead-letter queue，可以放到 SNS or SQS. 
- Concurrent executions: 同時有多少個 ㄑunction 正在執行

### Environment variables
- 可以透過 KMS 加
- 可也從 SSM 取得

### Versions and aliases
- 只可以更改或update $LATEST 
- Aliases 可以設定 Weight 來分流到不同版本

## Security
---
- IAM Resource Policy: 執行 Lambda Function 的權限
- IAM Execution Role: Lambda call 其他 AWS 服務需要的權限
- Lambda 可以在特定的 VPC 內運行

## Concurrency
---
- 預設每個 Region 上限是 1000，所以 Lambda Function 共用的，可申請調高
- 當達到瓶頸時，Lambda 會返回 429 too many requests 
- 可以設定以下兩種  Concurrency
    - Reserved Concurrency: 特定 Function 保留執行數量，也是這個 Function 最大上限，但其他 Function 不能使用這數量
    - Provisioned Concurrency: 預先初始化的數量

## Synchronous v.s. Asynchronous
---
- Synchronous: Lambda 會回覆 Response 執行結果
- Asynchronous: Lambda 會回覆 Client 已經收到，並會執行，失敗會進入到 dead-letter queue

## Warm Start v.s. Cold Start 
---
![Center](/images/post/lambda-certification-note/coldAndWarnStart.png#center)

## 整合其他 AWS 服務
---
- Amazon EFS : Lambda Function 可以 mount EFS 當 Local File System
- Amazon Gateway API : 提供 Http or Https API
- X-Ray: 追中整個 Request Data Flow. 
- AWS CodePipeline:  客製化 Task
- Lambda@Edge: 整合 CloudFront ，可以在Edge 做客製化例如透過Lambda 在Edge 就做身份驗證
- Parameter Store : 當多個 Lambda Function 需要共用參數。


