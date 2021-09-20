+++
title = "CloudTail Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 CloudTail 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "CloudTail",
]
date = "2021-09-20"
aliases = ["cloudtail-certification-note"]
images = ["images/aws.jpeg"]
+++

- CloudTail 主要用於監控和稽核 AWS 資源的 API Call

## Trails
---
- 每個 Region 上限五個
- Type
    - All Region : 所有的 Region Event Log 存在一個 S3 Bucket
    - One Region : 指定特定 Region Event Log 
- Organization Trail : 可以 Log 整個 AWS Organization 的 Event，只有 management account. 
- 也可以 Cross Account Trail，原理是設定 S3 Bucket Policy 給予每個 AWS Account 權限，每個 Tail 的 Log 放入同一個 S3  
- 可以發送 SNS 
    
## Events
---
- Type 
    - Management events: AWS 服務操作，又分為 Read, Write 兩種 Type
        - 預設Log， 最近 90 的 Log 是免費的，可以直接在 Event History 內查看
        -  Type example
            - Read: EC2 的 DescribeSecurityGroups
            - Write: EC2 的 TerminateInstances  
    - Data event : 操作特定 AWS 服務裡的資源，例如 S3 GetObject, DynamoDB PutItem or Lambda Invoke
        -  預設沒啟用，須額外收費
    -  如果是全球性的服務例如 IAM, STS 等等，Log 是放到 US East (N. Virginia) Region
    -  Event History 可以根據時間區間或以下屬性做 filter
        -  Event name, User name, Resource name, Event source, Event ID, and Resource type.  

## Security 
- 有 Log File Validation ，可以檢查刪除和修改
    - 刪除會顯示 not found 錯誤
    - 修改會顯示 hash value doesn't match  
- 可以整合 CloudWatch Log ，去監控特定事件，發出 Alert
- S3 預設使用 AWS AES key 加密，可以改用 KMS
- AWSCloudTrail_FullAccess 限制只有少數人，擁有這個權限
