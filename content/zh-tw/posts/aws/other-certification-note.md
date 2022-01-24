+++
title = "testing"
author = "Allen Hsieh"
categories = ["AWS"]
tags = [
    "testing",
]
draft=true
date = "2021-09-21"
aliases = ["other-certification-note"]
images = ["images/aws.jpeg"]
+++

## Catalog
---
- 給不會用的人使用， ADMIN 創建一個 protal 給 user 指定權限。
- 一個 Product 是一個 CloudFormation Template
- Profolios 是關聯 Product 和 操作的人，並且可以設定一些限制

## Inspector
---
- Two Type Assessments
    - Network Assessments (Agent 不是必要的) : 網路設定檢查，如果有安裝 Agent 可以知道所有的 Processes 的 port
    - Host Assessments (Agent 是必要的): 找到有弱點的軟體
- 可以決定要安裝的 Ec2 instance 或全部
- 可以透過 SSM run command 安裝 Inspector agent 。
- 整合 CloudWatch Event 
    - Schedule 定期執行，可以設定每次掃的區間
    - 每次有新的 EC2 機器，就自動執行
- 整合 SNS Topic : Event 包含 Run started, Run finished, Run state changed, Finding reported


## Health Dashboard
---
- Service Health Dashboard
- Personal Health Dashboard
    - 可以整合 CloudWatch Event 
        - 可以指定 Service
        - 可以指定 Category : scheduled change, issue, accountNotification 
    - AWS_RISK_CREDENTIALS_EXPOSED
    - https://github.com/aws/aws-health-tools/blob/master/automated-actions/AWS_RISK_CREDENTIALS_EXPOSED/README.md 

## Trust Advisor 
---
- Cost Optimization 
- Performance
- Security
- Fault Tolerance
- Service Limit
- 可以整合 CloudWatch Event ，只有 refresh status 這個 event 

Application Load Balancer 可以有 Weight , 但 Network Load Balancer 沒有
CloudWatch Load Balancer 不能 enable detailed log 
KMS Key 沒 Rotate ，用 Config 去監測
Config 有 cloudformation-stack-drift-detection-check 去檢測 CloudFormation 
CloudWatch 可以用 subscription filters 整合 Lambda

aws:runDocument plugin runs SSM documents stored in Systems Manager or on a local share. You can use this plugin with the aws:downloadContent plugin to download an SSM document from a remote location to a local share, and then run it.

xray-daemon.config is only for beanstalk 
ECS can include awslogs 
RDS
---
Read replica => 可以是其他 Region
multi-master  => same region different AZ

XRAY Deamon 是聽UDP Port 2000

OpsWorks lifecycle  Events: Setup, Configure, Deploy, UnDeploy and Shutdown

4