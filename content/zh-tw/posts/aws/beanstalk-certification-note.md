+++
title = "Beanstalk Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 Beanstalk 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "Beanstalk",
]
date = "2021-09-12"
aliases = ["beanstalk-certification-note"]
images = ["images/aws.jpeg"]
+++


## Basic
---
- Beanstalk 是 PAAS 服務，可以透過 Config (.ebextensions) 設定去建立環境，例如基本的 Three Tire Architecture : Client -> Server -> DB，而我們只需要專注在 Application 上的開發 
- Beanstalk 本身是免費的，只需要付原本底下的資源費用 例如 EC2, ELB 等等
- 支援多個 Programming Language ，也支援 Docker
- 有 Deployment 策略 和 Rollback 策略
- 可以備份現在的設定 (Save Configuration)
- 環境設定的順序 高到低
    - 直接環境設定
    - Save Configuration
    - .ebextensions 設定
    - Default Value 
- 可以設定 Management Updated 
    - 可以排成固定時間去做 patch
    - 可以選擇是直接更新舊的，還是先建立一台新的機器安裝好去取代 
- Swap URL ，透過此功能做到 Blue/Green Deployment ，底層是更改 Router53 Record
- Multi-Container 底層是用ECS 做到

## Basic Of Components
--- 
- Application
- Application Version 
    - Deploy Code 放在 S3，指向特定 S3 Object Version
    - Version 有上限，可以透過 Application Life Cycle Policy 去刪除舊的 Version
        - 可以設定上限多少 Version 
        - 要存和保留多少天
        - 也可以設定 Source 是否要保留在 S3
- Environment Tire
    - Web Server: 
    - Worker: 在上面執行一些程式，例如可以是SQS 的 Puller ，將 Queue 裡面資料拉下來處理。有 Auto Scaling Group 可以根據 SQS Queen Number 去做 Auto Scaling  
- Environment Types
    - 單一 Instance Environment 
    - Load-balancing, Autoscaling Environment: for web server
       -  Autoscaling Only : for worker Tire



## . ebextensions
---
### 建立 Resource
- 注意 Resource 如果透過 Beanstalk 一起建立的話，砍掉環境時也會跟著一起砍掉，所以 Storage 的部分最好分開建立，例如 RDS。

```YAML
Resources:
  DynamoDBTable:
    Type: AWS::DynamoDB::Table
    Properties:
      KeySchema:
         HashKeyElement:
           AttributeName: id
           AttributeType: S
      # create a table with the least available rd and wr throughput
      ProvisionedThroughput:
         ReadCapacityUnits: 1
         WriteCapacityUnits: 1

  NotificationTopic:
    Type: AWS::SNS::Topic

Outputs:
  NotificationTopicArn:
    Description: Notification topic ARN
    Value: { "Ref" : "NotificationTopic" }

option_settings:
  aws:elasticbeanstalk:application:environment:
    # these are assigned dynamically during a deployment
    NOTIFICATION_TOPIC: '`{"Ref" : "NotificationTopic"}`'
    DYNAMODB_TABLE: '`{"Ref" : "DynamoDBTable"}`'
    AWS_REGION: '`{"Ref" : "AWS::Region"}`'
```
### Command v.s Container Command 
---
- commands : 是在 Ec2 Instance 上執行，並且在 Application Code unpacked之前，按照英文名字順序執行
- container_commands : 是在 Application code unpacked 之後，但在部署之前，可以用來修改 source code 
    - leader_only : 由 Beanstalk 決定單一 instance 當 Leader 執行，並且會比其他沒有 leader_only 的 command 早執行，主要用在只能或只需要執行一次的 Command。
```YMAL
  create_hello_world_file:
    command: touch hello-world.txt
    cwd: /home/ec2-user
    
container_commands:
  modify_index_html:
    command: 'echo " - modified content" >> index.html'
```    

## Deployment Policy
---

| Policy | 描述 | 推失敗影響 | 推機速度 | 推幾時是否有 Downtime | 是否需要修改 DNS Record | Rollback 方式 |
|------|-----|--------|-------|-------------|----------------|---------------|
| All at once | 一次全部機器直接推機 |  Downtime | 最快 | 有 | 否 | 需重新推舊的Version |
| Rolling | 一次下掉幾台線上機器，裝好新的code 在重新上線。| 只有正在推的那幾台會壞掉，線上有可能會部分機器是新的 Code ，部分機器是舊的 Code | 比 All at once 慢 | 無 | 否 | 需重新推舊的Version|
| Rolling with an additional batch | 跟 Rolling 差不多，差別在於是先建立全新的機器推完機以後，在取代線上的| 除了一開始推壞沒上線已外，其他跟 Rolling Policy 一樣 | 比 Rolling 慢 | 無 | 否 | 需重新推舊的Version |
| Immutable | 建立全新的 Auto Scaling Group，然後將流量轉到新的 Auto Scaling Group | 比較沒有，因為可以先測試全新的 Auto Scaling Group 才切換 | 慢 | 無 | 無 | 切回流量到舊的 Auto Scaling Group |
| Traffic Splitting | 根據權重慢慢的將流量切到新的機器上 | 取決於流量的權重 | 慢 | 無 | 無 | 將權重切回到舊的 |
| Blue/Green | 透過 DNS Record 切換流量 | 比較小，因為是全新的一組機器 Ready 才切 | 慢 | 無 | 是 | 重新設定 DNS Record 指向舊的機器 |  

