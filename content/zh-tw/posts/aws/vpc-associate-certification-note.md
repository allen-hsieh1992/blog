+++
title = "VPC Associate Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Associate Certificate 時 VPC 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "VPC",
]
date = "2021-09-07"
aliases = ["vpc-associate-certification-note"]
images = ["images/aws.jpeg"]
+++

## Basic of VPC
---
- 每個 Region 預設上限五個 VPC
- 支援 IPv4 or IPv6 
- 每個 VPC 可以有五個 CIDR ，每個 CIDR 最大 /16 (65536 IP) , 最小 /28 (16 IP)
- VPC 是 私有網路，所以只有以下 Range 是允許的
    - 10.0.0.0/8
    - 172.16.0.0/12
    - 192.168.0.0/16
       {{< alert "要注意，VPC CIDR 不能跟其他已存在網住重疊" warning >}}
- AWS 有預留五個 IP 
   - ex: 10.0.0.0/24 => 256 個 IP 減去預設五個，實際只有251 個 IP 可使用
      - 10.0.0.0 : Network Address
      - 10.0.0.1 : 預設給 VPC Router
      - 10.0.0.2 : 預設給 DNS 
      - 10.0.0.3 : 先預留，暫時無使用
      - 10.0.0.255 : Network broadcast address，但 AWS 不支援 
     {{< alert "如果你需要 29 IP， Submask不可以選 /27 ， /27 雖然有 32 個 IP ，但扣除五個預留的以後，只剩下 27 個 IP" warning >}}
 


## Default VPC 
---
- 新帳號每個 Region 都會有 Default VPC，IP Range 172.31.0.0/16
- Default Subnet 的 Subnet Mask 是 /20 
- Default VPC 已經有 Main Route Table 而且設定好 Internet Gateway
- 在 Default VPC 啟用的 EC2 會自動 assign Public IPv4 IP
- Default Network Access List => Allow all inbound/outbound Traffic 
- 有預設 DHCP 

## Internet Gateway 
---
- 一個 VPC 只能連結一個 Internet Gateway
- Internet Gateway 同時也是 Network Address Translation (NAT)，有 IPv4 地址
- 支援 IPv4 & IPv6 




## Nat Gateway v.s Instance
---
- NAT 主要轉換 Private IP to Public IP，讓 Private Subnet EC2 可以單向連上網路
   - Private Subnet EC2 --> NAT --> IGW --> Internet 
- 都必須在 Public Subset
- 只支援 IPv4 

| 功能        | NAT  Gateway                   | NAT Instance                                                      |
|--------|------------------------|-----------------------------------------|
| Available |  AWS 託管服務，建立在一個 AZ，有 HA，但如果只有一個NAT gateway ，有可能因為一整個 AZ 掛掉而沒有網路 |  自己需要管控 Failover，如果只有一台壞掉就沒有網路 |
| 頻寬       |  從 5Gbps 到最高 45 Gbps.    |  取決於 Instance Type                                            |
| Maintains |  AWS 管理 | 需要自己升級內部軟體和操作系統等等 |
| Cost.     |  取決於用量和頻寬                 |  取決於 Instance Type                                            |
| Public IP | Elastic IP.                           |  Elastic IP or Public IP                                            |
| Security Group | X                              | V                                                                        |
| EC2 設定 | X | 需要關閉 EC2 的 Source/ Destination Check | 

## Egree Only Internet Gateway
- 只支援 IPv6
- 跟 NAT Gateway 相似，但 NAT Gateway 只支援 IPv4

## NCAL v.s Security Group
---
|.          | NCAL  | Security Group |
|------|------|-------------|
| Scope | 一個 Subnet 一個 NCAL  | 一個 Instance 最多五個 Security Group | 
| 特性 | Stateless: inbound & outbound 都需要特別設定  | Stateful: 只驗進出並記錄 request ，讓 response 可以返回 | 
| Default VPC 建立的 | 預設允許所有的 inbound & outbound  | X|
| 手動建立的 | 預設不允許任何   inbound & outbound |   預設沒有任何 Allow |
| 設定 | 可以設定 Allow 或 DENY ，另外有 rule 的概念 ，rule range (1-32766) 數字越低越優先，Ex: #100 Allow x, #200 DENY x 最後是 ALLOW |  只能設定 Allow 不能 DENY |


## VPC Peering
---
- 透過 AWS 私有網路連結兩個 VPC
    {{< alert "兩個VPC CIDR 不能重疊" warning >}}
- VPC Peering 不是 transitive 
   -  EX : VPC_A  <---VPC Peering---> VPC_B <---VPC Peering---> VPC_C
   -  VPC_A 不能透過 VPC_B 連到 VPC_C ，如果需要連線，就要在 VPC_A & VPC_C 中間再建立一個 VPC Peering 才可以互相連到
- VPC Peering 不限制在同一個 Region 也不限制在同一個 AWS 帳號下的 VPC 
- VPC Peering 設定完，還需要設定 Route Table 才可以連線       

## VPC Endpoint 
---
- 主要解決 VPC 內部的 Instance 可以透過 AWS 私有網路不必透過公有網路連到其他AWS 託管服務 如 S3, DynamoDB 等等
- 兩種 Type
   - Interface : 透過 AWS PrivateLink 去連到 其他 AWS 服務。
     - Endpoint : Server Side 需要使用 Load Balancer  
     - 而 Client Side 需要使用 Elastic Network Ineterface 
     - 中間透過 PrivateLink 去連起來
   - gateway : 提供特定目標，目前只有 S3, DynamoDB ，自動設定 Route Table 設定

## Flow Log
---
- 擷取傳入及傳出 VPC 中網路界面之 IP 流量相關資訊的功能
   -  VPC Flow log 
   -  Subnet Flow Log
   -  Elastic Network Interface Flow log 
- 日誌可以存在 S3 (可以整合 Athena 去 Query) / CloudWatch Log   

## DNS support in VPC
---
- enableDnsSupport 
  - 設定預設是 True
  - AWS DNS Server (169.254.169.253
- enableDnsHostname 
  - 手動建立的預設是 False, Default VPC 預設是 True
  - 如果 ture 會自動給 EC2 Instance 分派 Hostname 如果有 public ip   
- 如果想使用客製化 Private Route53 DNS Domain ，這兩個設定都必須打開

## VPN
---
- 在公司那邊建立  Customer Gateway （可以是軟體，或一個實體裝置）
- 在AWS這邊建立 Virtual Private Gateway ，並連結在VPC上
- 透過 site to site VPN Connection 將 VPN Gateway & customer gateway 建起連線

## Direct Connect 
---
- 建立專用網路連線到AWS，AWS 在不同地區都有不同的合作夥伴，所以連線方式像是 公司 --> 合作夥伴 --> AWS 
- 專線好處是 : 頻寬高，網路費用比較便宜，並且比較安全

