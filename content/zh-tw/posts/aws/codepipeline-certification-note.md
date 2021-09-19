+++
title = "CodePipeline Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 CodePipeline 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "CodePipeline",
]
date = "2021-09-18"
aliases = ["codepipeline-certification-note"]
images = ["images/aws.jpeg"]
+++



## Basic 
---
- Stage 裡面可以有許多 Action， Stage 跟 Stage 之間是可以關聯的
    -    runOrder 可以用來決定順序性， 預設是1 ，同樣的 runOrder 代表同時跑
- Artifacts: S3 
    - 預設使用 AWS/S3 key 加密，但可以選擇使用 KMS 
    - 也可以建立一個 S3 的 Action ， Copy artifacts 到其他  S3 Bucket
- CodePipeline 可以透過 ColudFormation 建立

    
    
## Action
---
- Approval Action: 可以設定需要  Manual Approval 才可以進去執行

- Source Action  : GitHub, [CodeCommit](/posts/aws/codecommit-certification-note/), [S3](/posts/aws/s3-certification-note/), ECR
    - CodePipeline 監測改變選擇
        - 根據 CloudWatch Events
        - CodePipeline 週期性去檢查 	
- Build Action : [CodeBuild](/posts/aws/codebuild-certification-note/), and Jenkins
- Deploy Action: AWS [CodeDeploy](/posts/aws/codedeploy-certification-note/), [Beanstalk](/posts/aws/beanstalk-certification-note), [CloudFormation](/posts/aws/cloudformation-certification-note), ECS 
    - 可以建立不同 Region 的同時部署
       - CloudFormation 
           - 可以執行 
               -   Create or update a stack
               -   Delete a stack 
               -   Replace a failed stack 
               -   Create or replace a change set
               -   execute a change set
           - 特定指定 S# template   
           
- Invoke Action : 觸發 Lambda Function
- Test Action: [CodeBuild](/posts/aws/codebuild-certification-note/), Jenkins and or  open source
    - 可以設定 Input artifacts and output  artifacts 

### Example

{{< mermaid >}}
sequenceDiagram
   CodePipeline->>+CodeCommit: Trigger
	loop
        CodeCommit->> CodeCommit: package source code
   end
   CodeCommit->>+S3 : push to artifacts
   CodeCommit->>+CodePipeline:Success
   CodePipeline->>+CodeBuild: Trigger
 	CodeBuild->>+S3: pull artifacts     
 	loop
        CodeBuild->> CodeBuild: build
   end
 	CodeBuild->>+S3: push to artifacts     
	CodeBuild->>+CodePipeline:Success
	CodePipeline->>+CodeDeploy: Trigger
   CodeDeploy->>+Instance: Deploy
	 

{{< /mermaid >}}



