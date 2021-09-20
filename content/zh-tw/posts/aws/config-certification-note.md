+++
title = "Config Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 Config 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "Config",
]
date = "2021-09-20"
aliases = ["config-certification-note"]
images = ["images/aws.jpeg"]
+++

## Basic 
---
- Resource Type : 
    - Config 是 Region Level 服務，預設存取當下 Region 下 Config 支援服務的全部設定，可以指定特定服務
    - 可以存取 Global 服務的設定
- 所有 Config 紀錄存取到 S3 
- 單一 Resource 可以看 Configuration Timeline detailed 包含
    - 和其他服務的關係
    - 改動的地方
    - CloudTail 整合 Event 
 


## Rules
---
- 每一個 Rule 每個月有固定費用 ex 1 USD/Month 
- AWS 已經有提供許多常見的 Rule ， 也可以客製化自己的 Rule
        - rule 是透過 Lambda 執行去檢查 
- Trigger Type
    - Configuration Change : 每次更動都檢查
    - Periodic: 週期性檢查
- 檢查 Scope 
    - Resource Scope : 特定 Resource 例如 EC2 
    - Tags: 特定 Tag
    - All Changes
- Compliance Status: Noncompliant or compliant
- 單一 Resource 可以看 Compliance Timeline
       - 可以從 Timeline 看到 Rule 的 Status 改變 
       - 和其他服務的關係
       - 改動的地方

## SNS
--- 
- AWS Config 有 Streaming 的功能，可以將 Event 通知到 SNS
- 會通知的事件包含
    - Resource 的 Config 變動
    - 歷史紀錄 或 Snapshot 上傳到 S3 
    - Compliance 的結果
    - Compliance Rule 開始審核
    - AWS Config 跟 S3 互動失敗，例如上傳 Snapshot
    
## CloudWatch Event
---
- 可以偵測到 Event
    - AWS API call via CloudTail
    - Config Configuration Item Change
    - Config Rules Compliance Change
    - Config Rules Re-evaluation Status
    - Config Configuration Snapshot Delivery Status
    - Config Configuration History Delivery Status

## SSM 
---
- Config 整合 [System Manager Automation](/posts/aws/ssm-certification-note/#automation) 可以嘗試自動修復 Noncompliant 資源

## Advance Query Example
--- 
- 可以使用 SQL Query 

```SQL
SELECT
    resourceId,
    resourceType,
    configuration.instanceType,
    configuration.placement.tenancy,
    configuration.imageId,
    availabilityZone
WHERE
    resourceType = 'AWS::EC2::Instance'
    AND configuration.imageId = 'ami-12345'
```    
## Aggregator View
---
- 可以整合多個 AWS Account 或多個 Region 下的 AWS Config 資訊
- 需要由一個 Main Config 建立 View，其他 Config 授權給 Main Config 就算是同一個 Account 但不同 Region。
    - 例如要多個Region ，Main Config 在 JP 建立 Aggregator 並整合 US Config，在 US Config 下需要授權給 JP Config
![Center](/images/post/config-certification-note/configAggregate.png#center)

## S3 允許 Config 權限的 Bucket Policy
---
```YAML
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSConfigBucketPermissionsCheck",
      "Effect": "Allow",
      "Principal": {
        "Service": [
         "config.amazonaws.com"
        ]
      },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::targetBucketName"
    },
    {
      "Sid": "AWSConfigBucketExistenceCheck",
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "config.amazonaws.com"
        ]
      },
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::targetBucketName"
    },
    {
      "Sid": "AWSConfigBucketDelivery",
      "Effect": "Allow",
      "Principal": {
        "Service": [
         "config.amazonaws.com"    
        ]
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::targetBucketName/[optional] prefix/AWSLogs/sourceAccountID-WithoutHyphens/Config/*",
      "Condition": { 
        "StringEquals": { 
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}   
```
