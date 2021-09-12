+++
title = "S3 Associate Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Associate Certificate 時 S3 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "S3",
]
date = "2021-09-12"
aliases = ["s3-associate-certification-note"]
images = ["images/aws.jpeg"]
+++

## Basic of S3
- 已 Object 為單位的儲存空間 ，而每個 Object 儲存在 Bucket
- 每 Object 的大小可以從 0 byptes to 5 TB，但 Bucket 沒有儲存中間上限
- 每個 S3 都會有獨立的 URL，這個是全球唯一的，所以 S3 建立後就無法修改 Region ，也不可以修改名字。
- 可以 Host 靜態頁面
-  如果要把 Object 變成公開的，需要先把 Bucket 設定為公開的，預設是私有的。
-  AWS CloudTrail  有存 API LOG 

## 取得 S3 資訊的方法
- API
- Amazon Direct Connect
- Storage Gateway 
- Kinesis Firehouse 
- Transfer Acceleration 
- Snow Family (SnowBall, SnowBall Edge, Snow Mobile) 

## 每個 Objects 包含基本資訊
- Key (name of the object)
- Value
- Version ID 
- Metadata
- Sub-resources
  - Access Control Lists and Bucket Policies
  - Torrent

## Data Consistency
- Read after Write consistency for PUTS of new Objects
  - 每當建立新的物件後，直接去讀取物件會存在，並不會讀取不完整的物件
- Eventual Consistency for overwrite PUTS and DELETES 
    -  因為 S3 一個物件存在多個地方，所以對舊有的物件做更新或刪除後立馬讀取的資訊有可能並非最新的。 
-  Strong Consistency for overwrite PUTS and DELETES 
    - 保證在更新物件以後，拉到的資料是最新的
 
## Tiered

- S3 Standard : 基本
- S3 Infrequently Access : 比 Standard 便宜，但需要額外收 retrieval fee ，適合存放比較不常使用的物件
- S3  One Zone Infrequently Access : 比 S3 IA 便宜，但只存在一個 AZ ，單點壞掉的風險比較大
- S3 Intelligent : 透過物件 access 頻繁度，自動區分 Standard or IA 
- Glacier : 
   - 每個物件在 Glacier 叫 Archive ，一個 Archive 空間上限是 40TB
   - Archive 存在 Vaults 裡面
   - 不能 real time 取得物件 ，都需要先做初始化 Retrieval 的動作
       - 初始化到完成可以下載需要時間，等完成以後可以透過 SNS 通知
       - 下載的 URL 是有時效性的，如果過期就需要再重新申請restore
   - 預設所有的物件都會被 AWS AES-256 加密
   - 也有 vault access policy 和 lock policy 
   - type
       - S3 Glacier 
           - Retrieval Options 
               - Expedited : 最快但最貴，可以是  On-Demand 的也可以是 Provisioned (1-5 mins)
               - Standard : 基本的 (3-5 hours)
               - Bulk : 最慢，但最便宜 (5 to 12 hours)
       - S3 Glacier Deep Archive 
           - Retrieval Options 
               - Standard : 基本的 (12 hours)
               - Bulk : 最慢，但最便宜 (48 hours)


|  | S3 Standard | S3 Intelligent | S3 Standard IA | S3 one zone IA | S3 Glacier | S3 Glacier Deep Archive | 
|-----|----------|-----------|-------------|------------|----------|--------------------|
| Durability | 99.99999999999 (11's 9) |  99. (11's 9) | 99. (11's 9) | 99. (11's 9) | 99. (11's 9) | 99. (11's 9) | 
| Availability | 99.99% | 99.9% | 99.9% | 99.5% | 99.99% | 99.99%|
| SLA | 99.9% | 99% | 99% | 99% | 99.9% | 99.9% |
| AZ | >= 3 | >= 3  | >= 3 | =1 | >= 3 | >= 3 |
| 最小 Object Size收費單位 (小於這單位以這單位收錢） | N/A | N/A | 128KB | 128KB | 40KB | 40KB | 
| 最小收費存取時間 (存取時間小於這單位以這單位收錢) | N/A | 30 days | 30 days | 30 days | 90 days | 180 days |
| Retrieval fee | N/A | N/A | per GB  | per GB | per GB | per GB |

## 功能
### Lifecycle Management
- 可以設定現在版本或過去版本的Object N 天以後移動到不同的 Tiered
- 可以設定最新的版本只保留多久或多久後永久刪除
- 刪除超時未完成的 Multi-part 上傳
- 可以根據不同的 prefix or tag 設定


###  Versioning 
- Bucket Level 功能
- 當物件被刪除時，會有 delete marker ，如果我們將 delete marker 刪除，物件又會回來
- 不可以關閉(disable) 只能暫停 (suspend)
- 物件大小計算是 全部 Versioning 的總和
- 新物件都會有 Version Id ，但如果是在開啟 Versioning 功能之前就上傳的物件會是 null


###  Encryption
- Server Side Encryption
   - 3 type   
		- SSE-AES : AWS 管理的 Key 
		 - header "x-amz-server-side-encrpytion" : "AES256"
		- SSE-KMS : 透過 KMS Key
		 -  header "x-amz-server-side-encrpytion" : "aws:kms"
		- SSE-C  : 自己提供的 Key，S3 不會存取這個 Key ，透過 request Header 將 Key 提供給 S3 加解密
   - 可以在 Bucket Level 開啟 Default Encryption 有 SSE-AES & SSE-KMS 可以選擇
   - 可以透過 Bucket Policy 根據 header 強制要 Encryption 
- Client Side Encryption 


### Transfer Acceleration 
- 透過 CloudFront Edge 加速上傳速度，會有不同的 endpoint Ex xxx.s3-accerate.amazonaws.com

### Cross Region Replication (CRR) or Same Region Replication (SRR)
- 自動備份到不同的 Region (不需要同 Account) ，但開啟功能前已存在的物件是不會備份的。
- 需要開啟 Versioning 的功能
- 如果是加密物件，備份到的 Bucket 也需要有加密 Key 的權限。
- 如果要備份 delete mark，需要額外設定
- 刪除 Version ID 不會備份去避免遭到惡意病毒攻擊
- 如果 Bucket1 備份到 Bucket2, 而 Bucket2 備份到 Bucket3, Bucket1 的備份不會到 Bcuket3 

### Pre-Signed URL
- 只能透過 CLI 或 SDK 產生，提供短暫有效的 URL 
- URL 擁有的權限繼承產生 URL 的人
- 預設Timeout 一小時
```bash
aws s3 presign s3:://mybucket/myobject --expires-in 300 (sec)
```

### MFA for Delete 
- 刪除物件需要 MFA 驗證
- 需要打開 Versioning 的功能
- 只有 Root Account 可以打開或關閉這功能 ，只能透過 CLI or SDK 設定，沒辦法透過 Console

### Object Lock
- 主要達到 WORM (write object once, read many)
- 有兩種模式
   - Government model : 只有特殊權限的人，才可以修改物件
   - Compliance model : 在特定的時間內，任何人都無法修改物件
- Glacier 也有 Glacier Vault Lock 

###  Bucket Policies 
- 可以給予其他 AWS Account 權限
```Json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": ["arn:aws:iam::111122223333:user/Alice",
                "arn:aws:iam::111122223333:root"]
      },
      "Action": "s3:*",
      "Resource": ["arn:aws:s3:::my_bucket",
                   "arn:aws:s3:::my_bucket/*"]
    }
  ]
}
```

### ACL
```xml
<?xml version="1.0" encoding="UTF-8"?>
<AccessControlPolicy xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Owner>
    <ID>Owner-canonical-user-ID</ID>
    <DisplayName>display-name</DisplayName>
  </Owner>
  <AccessControlList>
    <Grant>
      <Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="CanonicalUser">
        <ID>Owner-canonical-user-ID</ID>
        <DisplayName>display-name</DisplayName>
      </Grantee>
      <Permission>FULL_CONTROL</Permission>
    </Grant>
    
    <Grant>
      <Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="CanonicalUser">
        <ID>user1-canonical-user-ID</ID>
        <DisplayName>display-name</DisplayName>
      </Grantee>
      <Permission>WRITE</Permission>
    </Grant>

    <Grant>
      <Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="CanonicalUser">
        <ID>user2-canonical-user-ID</ID>
        <DisplayName>display-name</DisplayName>
      </Grantee>
      <Permission>READ</Permission>
    </Grant>

    <Grant>
      <Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="Group">
        <URI>http://acs.amazonaws.com/groups/global/AllUsers</URI> 
      </Grantee>
      <Permission>READ</Permission>
    </Grant>
    <Grant>
      <Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="Group">
        <URI>http://acs.amazonaws.com/groups/s3/LogDelivery</URI>
      </Grantee>
      <Permission>WRITE</Permission>
    </Grant>

  </AccessControlList>
</AccessControlPolicy>
```

### Access log 
- 可以存放 Access Log 到另一個 S3 Bucket 
- 如果要監控 BucketA ，不要將 BucketA 的 access log 寫到 BucketA 自己，會造成無限循環 

### S3 CORS (CROS Origin Resource Sharing)
- Origin 包含 protocol , domain, port 
   -  EX: https://www.google.com => protocol: https, domain: www.google.com, port 443
- 如果沒有 CORS Header ， Browser 會阻擋其他網站來 request 我們網站
   - Origin Header : Access-Control-Allow-Origin : origin
   - Method Header : Access-Control-Allow-Methods : http_method (GET, PUT)
- 範例 如果 S3 BucketA 裡面的頁面要拉 S3 BucketB 的圖案，那 BucketB 需要打開 CORS Allow BucketA 來讀取。

```json
[
    {
        "AllowedOrigins": [" https://www.example.com "],
        "AllowedMethods": ["GET"],
        "MaxAgeSeconds": 3000,
        "ExposeHeaders": []        
        "AllowedHeaders": [Authorization"]
    }
]
```

### Inventory 
- 可以產特定的 Bucket 每日或每週的 Metadata 報表存放到另一個同 Region 的 Bucket
- 報表格式可以是  CVS, ORC 或 Apache Parquet 
- 可以用來 Auditing 加密狀態、統計 Object 數量等等


### S3 & Glacier Select 
 - 如果 Object 是 Json 或 CVS 的格式，可以透過 SQL 的指令方式取出資料
 - 可以減少流量的傳輸，因為已經在server side 過濾，並減少 Client Side 的 CPU (如果Client 要自己過濾）

### S3 Analytics
- 分析多少物件從 Standard 轉到 Standard_IA
- Report 每天更新，第一開始需要 24 - 48 小時初始化

### Event Notification
- 當S3 更新物件或者是新增物件等等事件，可以發出 Event 給 Lambda, SNS, or SQS. 

### S3 Access Endpoint 
- 每個 Access Endpoint 都有獨立的 DNS 和 Policy 去限制誰可以access point policy 這個 S3 
- access point 流量可以從網路 或從 VPC 

### S3 Batch Operation
- 一個 request 可以做多個指令
- 建立一個 Job ，裡邊包含要變動的物件從 (Inventory Report) 或特定格式的 CSV，然後指定特定的 Operation
   - 複製到其他定方
   - Trigger Lambda Function
   - 換 Tag 或 刪除 Tag 
   - 換 ACL
   - 對 Glacier 物件申請恢復
   - 加 或修改 Lock 

## Performance
---
- S3 每多一個 prefix 
   - 3500 PUT/COPY/POST/DELETE  qps
   - 5,500 GET/HEAD qps
   - Ex:  
	 - https://s3.amazonaws.com/mybucket/prefix1/prefix2/myObject1 
	 - https://s3.amazonaws.com/mybucket/prefix1/prefix3/myObject2
	 - https://s3.amazonaws.com/mybucket/prefix3/myObject3  
	 - https://s3.amazonaws.com/mybucket/prefix4/myObject4
 	 - 如果 request 平均留到不同 prefix => 那會有 3500 * 4 的PUT/COPY/POST/DELETE  qps
- 如果 S3 的物件有透過 KMS 加密，qps 上限有可能受到 KMS 影響。 KMS 有每個region 有 quota qps，可以申請提高
- Multipart Upload  
  - 建議超過 100 MB 就要使用  Multipart Upload ，如果物件 Size 超過 5GB 一定要使用
-  Byte Range Fetch 
  - 可以同時拉不同 Range 的 Object 下來，最後組成一個完成的 Object 

  
 ## 整合其他 AWS 服務
 ---
 - DataSync : 將機器上大量物件上傳到 S3 ，並且可以週期性的上傳，需安裝 data sync agent 
 - Athena : 
     - 跟 S3 Select 一樣也是使用 SQL 的方式去拉取 S3 物件但是可以同時對大量的物件 ，S3 Select 一次只對單個物件。
     - Athena 是 Serveless，先將資料讀進 DB，然後再從 DB Query 資料出來 
 - Macie: 使用 Machine Learning 去監測，是否有存取敏感性資料 如 個資 PII, 信用卡等
 

