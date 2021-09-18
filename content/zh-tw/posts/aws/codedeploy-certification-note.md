+++
title = "CodeDeploy Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 CodeDeploy 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "CODEDEPLOY",
]
date = "2021-09-18"
aliases = ["codedeploy-certification-note"]
images = ["images/aws.jpeg"]
+++

- CodeDeploy 是 AWS 部署服務，類似 Ansible, Chef, Puppet 等等
- 要部署的機器上面需要安裝 CodeDeploy Agent, Agent 會一直從 CodeDeploy 拉需要執行的動作
- Support Ec2, Lambda, ECS 



## Basic Step of EC2
---
- 需要部署的機器上面需要安裝 CodeDeploy Agent
- Agent 會一直嘗試從 CodeDeploy 拉需要做的工作
- CodeDeploy 會提供 appspec.ymal 
- 要安裝的程式會從 S3 (IAM Role 需要 S3 權限）或 Github 等地方拉下來，並開始安裝 
- CodeDeploy Agent 會回覆成功或失敗


## 部署 EC2 
---
- 如果 Source 從 S3 ， IAM Role 需要 S3 權限 和 需要有 Versioning 
- Deployment Type
    - In Place: 現有的機器會下線，安裝好新版本，再重新上線。 下線的數量取決於  Deployment configuration
        - Environment : Auto Scaling Group, EC2 或 On-premises  
    - Blue/Green: 會建立新的 Instance 取代舊的，必須要有 ELB 在 EC2 前面
- 預設 Deployment configuration
    - CodeDeployDefault.AllAtOnce
    - CodeDeployDefault.HalfAtTime
    - CodeDeployDefault.OneAtTime

### Trigger v.s. Cloud Watch Event
- Trigger 是 CodeDeploy 內建，可以根據特定事件發送 SNS 
- Cloud Watch Event 由 CloudWatch 監控事件，觸發其他 AWS 服務

### Rollback
- Manual Rollback : 手動直接部署舊的版本
- Automatic Rollback 有兩種選擇
    - Rollback when deployment failed : 任何一個部署失敗，就自動 rollback 全部
    - Rollback when alarms thresholds are met: 這邊整合 Cloud Watch Alarm 


## 部署 On-Premises Instance 
---
- 用 IAM User (只適合單台機器）
    - 建立有 S3 權限的 IM User ，並取得 access key
    - 在機器上建立 codedeploy.onpremises.yml 將 access key 放入
    - 安裝 AWS CLI & CodeDeploy Agent 
    - call register api
    ```bash
      aws deploy register-on-premises-instance --instance-name AssetTag12010298EX --iam-user-arn arn:aws:iam::444455556666:user/CodeDeployUser-OnPrem
    ```
    - 可以對 CodeDeploy 上的 on-premises instance 加上 tags ，並透過 tag 建立 deployment group
- 用 IAM Role 
    - 建立 IAM Role 擁有 S3 權限
    - 建立 IAM User 有 STS 權限，可以 assume-role 
    - 在機器上建立 codedeploy.onpremises.yml 
    ```YAML
    ---
    iam_session_arn: iam-session-arn
    aws_credentials_file: credentials-file
    region: supported-region
    ```
    - call register-on-premises-instance api
    ```bash
    aws deploy register-on-premises-instance --instance-name name-of-instance --iam-session-arn arn:aws:sts::account-id:assumed-role/role-to-assume/session-name
    ```
    - 可以對 CodeDeploy 上的 on-premises instance 加上 tags ，並透過 tag 建立 deployment group


## 部署 Lambda Function
-  Deployment configuration
    - Canary: 分兩次切流量，可以設定百分比決定一次要切多少流量到新的版本，當第一次正常沒問題就切剩下的全部流量到新的版本
    -  Linear: 固定時間區間和固定百分比增長流量到新的版本
    -  All-at-once: 一次全部切換


##  appspec.ymal
---
```YAML
version: 0.0
os: operating-system-name
files:
  source-destination-files-mappings
permissions:
  permissions-specifications
hooks:
  deployment-lifecycle-event-mappings
```

### EC2 or On-Premises Instance Hook
- Basic 
    - ApplicationStop: 服務停止前
    - DownloadBundle: 下載新版本的檔案到機器上臨時文件夾
    - BeforInstall : 開始安裝前，可以先安裝一些或執行程式
    - Install: 從臨時文件夾安裝到實際文件夾
    - AfterInstall : 可以修改一些設定或修改一些檔案權限等
    - ApplicationStart: 服務重啟
    - ValidateService: 驗證服務
- 如果有 ELB
    - BeforeBlockTraffic: 從 ELB 下掉之前 
    - BlockTraffic: 停止流量到 Instance
    - AfterBlockTraffic: : 從 ELB 下掉之後
    - BeforeAllowTraffic: 在上回 ELB 之前 
    - AllowTraffic: 開始接流量
    - AfterAllowTraffic: 在上回 ELB 之後 
- 實際順序
    -  BeforeBlockTraffic
    -  BlockTraffic
    -  AfterBlockTraffic
    -  ApplicationStop
    -  DownloadBundle
    -  BeforeInstall
    -  Install
    -  AfterInstall
    -  ApplicationStart
    -  ValidateService
    -  BeforeAllowTraffic
    -  AllowTraffic
    -  AfterAllowTraffic

![Center](/images/post/codedeploy-certification-note/CodeDeployHook.png#center) 


### Lambda Function Hook
- BeforeAllowTraffic
- BlockTraffic
- AfterBlockTraffic


### Environment Variable 	
- APPLICATION_NAME
- DEPLOYMENT_ID
- DEPLOYMENT_GROUP_NAME : 程式可以根據 Group 來判斷要做的事情，例如不同環境
- DEPLOYMENT_GROUP_ID
- LIFECYCLE_EVENT: Hook 名字


