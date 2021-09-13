+++
title = "Route53 Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 Route53 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "ROUTE53",
]
date = "2021-09-05"
aliases = ["route53-certification-note"]
images = ["images/aws.jpeg"]
+++

- Route53 是 AWS 的 Domain Name System (DNS)，名字中有 53 的原因是因為 DNS 的 Port 是 53。
- Route53 也是一個 DNS Register ，可以在上面購買 Domain Name 
- TTL (Time to Live) : 單位為秒，告訴 Browser 去 Cache Response 的 Record 多少時間。
- 主要 Component 
  - Hosted Zone 
    - 包含所有 Record 資訊
    - 會自動建立 Name Server 和 SOA records
    - Public hosted Zone  :  解析互聯網的 Request
    - Private hosted Zone :  解析Amazon上內部的 Request
      -  VPC 需要設定 enableDnsHostnames & enableDnsSupport
  - Name Server
    - 主要轉換 Hostname to IP

## DNS Basic Records
---
| Record Name | 描述                                         |
| ----------- | ---------------------------------------------|
| A.          | Hostname to IPv4                             |
| AAAA.       | Hostname to IPv6                             |
| CNAME       | Hostname to Hostname                         |
| Alias       | Hostname to AWS resource                     |
| MX.         | Email                                        |
| PTR         | 反向解析 A record IPv4 to Hostname           |
| NS.         | Name Server                                  |
| SOA         | 主要記錄網域名稱伺服器的名稱和負責人資訊 |

- Alias V.S. Cname

| 功能       | Cname                                                                |Alias                               |
| -----------|----------------------------------------------------------------------|------------------------------------|
| Example    | app.domain.com => xxx.otherDomain.com                                | app.domain.com => elbxx.amazon.com |
| ROOT DOMAIN| 只能是 non root domain Ex: 不可以是 domain.com => xxx.otherDomain.com| non root & root domain 都可以      |

## Routing Policy 
---

| Policy                       |描述 |是否有 Health Check |
|------------------------------|-----------------------------------------------------------------------------|--------|
| Simple Routing Policy        | 如果有多筆值 ，會隨機返回一個 | X |
| Weight Routing Policy        | 根據 Weight (%) 來分流 |V|
| Latency Routing Policy       | AWS 會返回延遲最小的值 |V| 
| Failover Routing Policy      | 有 Primary 和 Secondary，如果 Primary health check pass 就會返回 Primary 否則返回 Secondary | V|
| GeoLocation Routing Policy   | 根據 User IP 地理位置 來決定返回哪個值 |V|
| Geo proximity Routing Policy | 根據 User 地址位置外加 Bias 值 Ex:us-west-1 & us-east-1 都在 us ，如果有設定 bias ， bias 高的會有比較多的 request | V |
| Multi Value Routing Policy   | 回覆多筆值 ，瀏覽器自己選一個值去發送請求 (request) | V |

## Health Check
- Type
   - endpoint IP or Domain
      - 支援協議 : TCP, HTTP, HTTPS
      - 預設檢查區間 : 30 秒，最快可以 10 秒
      - 進階可以設定根據 Response 內容前 5120 Bytes 是否有包含預期的 String
      - 有 N 次檢查不過 => Unhealthy Status
      - 已經有 N 次檢查過 => healthy Status
      - 可以產生 Latency 圖
      - 大約有 15 檢查者（checker) 從不同地方檢查 ，所以 endpoint 不能是 private 的 (如VPC內
   - calculated health check 根據其他 Health check 來計算
   - cloudWatch alarms

## Others
---
- 可以只用第三方 DNS Register 註冊的 Domain ，將第三方的 Domain 的 Name Server 設定為 Route53 的 Name Server。
- Router53 Domain 指向 S3
      - bucket name 需要跟 url 一樣
  - for http : 建立 Alias Record 指向 S3, 並且 S3 需要是 Public 和 開啟 website 功能
  - for https : 需要使用 CloudFormation

