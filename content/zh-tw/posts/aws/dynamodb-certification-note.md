+++
title = "DynamoDB Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 DynamoDB 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "DynamoDB",
]
date = "2021-09-21"
aliases = ["dynamodb-certification-note"]
images = ["images/aws.jpeg"]
+++

## Basic of DynamoDB 
- DynamoDB 是 AWS NOSQL Database. 
- DynamoDB 的資料存在 SSD ，所以有很好的 performance
- Data 會自動備份到不同的 AZ ，預防 single point failure
- DynamoDB 以 table 為單位，裡面每筆的 Row 為 Item ，而每個 Item 的 Column 為 Attribute 
- DynamoDB 有兩種 Type 的 Primary Key ，在 DB 需要是唯一值
    - Partition Key  
    - Composite Key ( Partition Key + Sort Key)
        - 同一個  Partition Key 會存在一起，根據 Sort Key 去排序
- 可以設定 TTL (Time to Live ) to Item，所以當時間到 Item 會自動刪除。TTL 需要存要刪除的時間 Timestamp 

## Secondary Index
---
- Projection 代表一組 Attribute 複製到 secondary Index
- Types
    - Global Secondly Index
        - Partition Key 和 Sort Key 可以都不一樣
        - queries or scans 只能拉有 projected 到  secondary Index 的 attribute
    - Local Secondary Index 
        - 一樣的 Partition Key 但是不一樣的 Sort Key => 所以 Primary Key  一定要是 Composite Key 
        - 只能在建立 Table 時建立
        - query or scan 如果沒有 projected 到  secondary Index 的 attribute， DynamoDB 會自動從 table 拉出來

## Read Consistency
---
- Eventually Consistent Reads: 最終會一制性，因為 Data Source 存在不只一個地方，有可能讀到舊的 Data Source
- Strongly Consistent Reads: 讀到的資料一定是最新的，有更高的延遲，不支援 global secondary index
- Transition: 支援 ACID (atomicity, consistency, isolation, & durability) Transition ，上限一次 25 個 write

## Query v.s Scan
---
- Query 
    - 是透過 primary key 去找搜尋
    - 撈出來的資料是根據 Sort Key 去排序 ， 可以透過設定 ScanIndexForward = false 得到相反的順序的結果
    - 預設是 Eventually Consistent ，可以設定是 Strongly Consistent
- Scan 
    - 是拉出所有的 Items (full table scan)，然後可以透過 Filter 一一過濾
    - performance 比 Query 差
    - 可以用小的 page size 改進 performance  
    - 可以設定 Parallel 去加強 performance，但要注意 DynamoDB loading 是否很重 

## DynamoDB Capacity  
---
- Provisioned Capacity 吞吐量 (Throughput) 
    - DynamoDB 吞吐量定義單位為 Capacity Unit ，區分為 Read, Write 兩個   
        - 一個 Write Capacity Unit 等於 1 KB/1 Second 
        - 一個 Read Capacity Unit 等於
            -  1 個 Strongly Consistent Reads 4 KB / 1 Second 
            -  2 個 Eventually Consistent Reads 4 KB / 1 Second 
    -  如果超出定義的 Capacity Unit ，DynamoDB 會噴 ProvisionedThroughputExceededException 。
    -  建議時做 exponential backoff retry，意思就是隨著 retry 次數的增長，每個request 中間的時間區隔也會增長
-  On-Demand Capacity
    - AWS 會根據流量自動 Scaling 
    - 很難預測花費

## DynamoDB Accelerator (DAX)
---
- DAX 是 AWS 管理的 in memory cache cluster ，可以加快 Read 的 Perofrmance 
- 在寫入資料時，會同時寫入 DAX 和 DynamoDB Table，所以寫入後直接 Read 會 Hit 到 cache
- 不支援 Strongly consistent reads 

## DynamoDB Stream
--- 
- 會抓到所有修改 Item 的 Event ，可以 Trigger Lambda 
- 資料會加密和保存 24 小時
- 有獨立的 Endpoint 

## AWS CLI create table example
---
```BASH
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```
- KeyType : 
    - Hash: Partition Key
    - RANGE:       Sort Key

##  IAM Policy Conditions
---
- 限制 IAM User 只能操作特定的 Item ，以下面為例 dynamodb:LeadingKeys 是指 partition key，當 IAM UserID 等於 partition key 時才可以執行 Policy 裡面的 Action。

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowAccessToOnlyItemsMatchingUserID",
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetItem",
                "dynamodb:BatchGetItem",
                "dynamodb:Query",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem",
                "dynamodb:BatchWriteItem"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-west-2:123456789012:table/GameScores"
            ],
            "Condition": {
                "ForAllValues:StringEquals": {
                    "dynamodb:LeadingKeys": [
                        "${www.amazon.com:user_id}"
                    ],
                    "dynamodb:Attributes": [
                        "UserId",
                        "GameTitle",
                        "Wins",
                        "Losses",
                        "TopScore",
                        "TopScoreDateTime"
                    ]
                },
                "StringEqualsIfExists": {
                    "dynamodb:Select": "SPECIFIC_ATTRIBUTES"
                }
            }
        }
    ]
}
```
