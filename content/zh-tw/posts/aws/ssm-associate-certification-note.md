+++
title = "SSM Associate Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Associate Certificate 時 SSM 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "SSM",
]
date = "2021-09-09"
aliases = ["ssm-associate-certification-note"]
images = ["images/aws.jpeg"]
+++

- 可以用來管理 EC2 或是 On-Prime 的機器
  - 需要安裝 SSM Agent 在機器上
  - AWS Linux AMI 和部分 Ubuntu AMI 預設已經裝好 Agent 了。
  - EC2 需要有 IAM Rule 的權限根 SSM 溝通
- Patch 自動化
- 支援 Windows or Linux
- 可以整合 CloudWatch 和 AWS Config

## Resource Group
---
- 不只是適用於 EC2，也可以適用於其他 AWS 服務如 S3, Lambda 等等
- 使用 AWS Tag 的功能，根據 Key Value 將 Resource grouping 起來。
  - Ex: Environment 的 value 等於 Production 時候
- 也可以根據 CloudFormation Stack 來 Grouping

## Documents
- 一個 Documents 定義一些 Parameters & Actions (執行一些 commands)
- Documents 可以是 Json 或 Yaml 格式
- 兩種Type Run Command (Script) & Automation (AWS API)

## Run Command
---
- Run command 可以配合 Resource Group 一次對多台機器執行 Documents (Script) ，也可以手動指定特定機器執行
- 有 Rate 控管可以控管一次同時更新幾台 resource 。也有 Error 控管， 更新過程中失敗多少 resource 就中止。 Resource 可以使用百分比或指定固定數量來定義。 
- 可以透過 IAM & CloudTail 知道誰執行的
- EC2 是透過 SSM Agent 執行的，EC2 可以不需要 Allow SSH
- 執行結果可以在 Console 上看到，或存到 S3 或 CloudWatch Logs  
- 可以透過 SNS 發送 Status
- 可以整合倍 EventBridge Trigger

## Automation
---
- 主要用於簡化維護和部署 EC2 或 AWS 其他服務
- Automation runbook 就是 Documents 主要執行一些 AWS Resource 的 API ，例如 EC2 的 stop。
- 可以被 Trigger 的方式
   - AWS Console, Cli or SDK
   - EventBridge
   - 透過 Maintain Windows 排成
   - AWS Config Remediation 

## Parameter Store
---
- 可以安全地存取 Configuration & Secrets  ，並且有版本控制
- 可以透過 KMS 加密
- 透過 IAM 和 Path 控管
- 可以整合 CloudFormation 
- 有 CloudWatch Event 
- get-parameters cli 如果有加密資料預設返回的是密文，需另透過 KMS 解密。但也可以透過  --with-decryption 這個參數直接返回明文，需要有 KMS 權限
- Parameter policy 可以設定 TTL 去 通知 或 delete parameters 
   - 三種 Policy
      - Expiration : 指定時間刪除 Parameter 	
      - ExpirationNotification : 可以設定快過期通知
      - NoChangeNotification  : 可以設定通知，如果 Parameters 在指定時間內都未更新 
   - Example
```json
{
   "Type":"Expiration",
   "Version":"1.0",
   "Attributes":{
      "Timestamp":"2018-12-02T21:34:33.000Z"
   }
}

{
   "Type":"ExpirationNotification",
   "Version":"1.0",
   "Attributes":{
      "Before":"15",
      "Unit":"Days"
   }
}

{
   "Type":"NoChangeNotification",
   "Version":"1.0",
   "Attributes":{
      "After":"20",
      "Unit":"Days"
   }
}
```
- 兩種付費模式

| 功能\模式 | Standard | Advanced |
|------|-----|-----|
| 每個 Region parameter 上限 | 10,000 | 100,000 |
| parameters size 上限 | 4k| 8k|
| parameters policy | x | v |
| storage cost | free | $0.05 per parameter / mother|
| standard throughput | free | $0.05/ 10,000 API |
| high  throughput (1000 api / second) | $0.05/10,000 API | $0.05/10,000 API |


## Inventory 
---
- 收集 EC2 or On-premises 機器上的資訊包含 :已安裝的軟體, OS drivers 和 設定 或 正在執行的 services。並可以透過這些資料做稽核
- 可以設定資料更新的時間區間為 分、秒或小時
- 可以將資料存在 S3 透過 Athena 來分析
- 可以建立客製化 Inventory ，拉特定資訊


## State Manager
---
- 定義一個 State  (Configration)，讓所有指定的機器維持在定義的 State
   - 例如 State 可以是 關閉 ssh (port 22)

## Patch Manager
---
- 自動更新和修補機器
- 可以做 OS 升級，Application 升級，或者一些 Security 相關的升級
- 可以是一次性，或者透過 Maintain Windows 來排程。
- 可以掃描所有管理的機器並產出 compliance report 
- Patch Baseline : 可以定義哪些 patch 應該 或 不應該 裝在機器上
- Patch Group : 將多台機器 Grouping 起來並執行 Patch Baseline 
   - 一台機器只能有一個 Patch Group，一個 Patch Group 只能綁定一個 Patch Baseline
   - 透過 AWS Tag key "Patch Group" 去 Grouping

## Maintenance Window
---
- 排程去執行任務 例如 更新 Instance 上的 操作系統、軟體等等
- Maintenance Window 包含了
   - 排程(Schedule)
   - 期間(Duration) 
   - 一組機器
   - 一組要執行的任務
- 任務(task) 可以是
  - Run Command
  - Automation 
  - Lambda
  - Step Function
