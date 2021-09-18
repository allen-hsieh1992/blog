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

- Beanstalk 是 PAAS 服務，可以透過 Config (.ebextensions) 設定去建立環境，例如基本的 Three Tire Architecture : Client -> Server -> DB，而我們只需要專注在 Application 上的開發 
- Beanstalk 本身是免費的，只需要付原本底下的資源費用 例如 EC2, ELB 等等
- 支援多個 Programming Language ，也支援 Docker
- 有 Deployment 策略 和 Rollback 策略

## 可控制
---
- instance type
- databases
- Auto Scaling Group
- Elastic Load Balancer (Https)

## Basic Of Components 
- Application
- Application Version 
    - Deploy Code 放在 S3，指向特定 S3 Object Version
    - Version 有上限，可以透過 Application Life Cycle Policy 去刪除舊的 Version
- Environment Tire
    - Web Server: 
    - Worker: 在上面執行一些程式，例如可以是SQS 的 Puller ，將 Queue 裡面資料拉下來處理。有 Auto Scaling Group 可以根據 SQS Queen Number 去做 Auto Scaling  
- Environment Types
    - 單一 Instance Environment 
    - Load-balancing, Autoscaling Environment: for web server
       -  Autoscaling Only : for worker Tire

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

