+++
title = "CodeBuild Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 CodeBuild 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "CODEBUILD",
]
date = "2021-09-13"
aliases = ["codebuild-certification-note"]
images = ["images/aws.jpeg"]
+++

## CodeBuild Basic
---
- AWS 託管 Build Service類似 Jenkins。可以編譯，跑單元測試，或者打包 package 到 artifacts.
- 根據 運行力 和 執行時間來計價
- Build Log 可以送到 CloudWatch or S3
- CodeBuild 是使用 Docker 技術，所以可以在 Local 執行。也可以客製化個人 Docker Image 去當 Build 環境
- Source 可以從 Github, CodeCommit, S3, Bitbucket 和 Github Enterprise 拉
- 可以設定 Build Timeout 或 Queue Timeout 
- 也可以設定特定 VPC 執行
- AWS 有預設 [Environment Variable](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html) ，也可以客製化個人的環境變數

## Cloud Watch 
---
- Monitor Metrix
    - type
        - Succeeded Builds Sum
        - Failed Builds Sum
        - Build Sum
        - Duration Average
    - 可以根據  Matrix 設定 Alert 通知
- CloudWatch Event
    - 可以設定 Scheduler 去觸發 Build 
    - 可以聽特定 Event 去 Trigger 其他 AWS 服務 類似 Lambda , SNS 等等

## Buildspec Example Java Package to Artifacts
---
```YAML
version: 0.2

env:
  variables:
    JAVA_HOME: "/usr/lib/jvm/java-8-openjdk-amd64"
  parameter-store:
    LOGIN_PASSWORD: /CodeBuild/dockerLoginPassword

phases:
  install:
    commands:
      - echo Entered the install phase...
      - apt-get update -y
      - apt-get install -y maven
    finally:
      - echo This always runs even if the update or install command fails 
  pre_build:
    commands:
      - echo Entered the pre_build phase...
      - docker login -u User -p $LOGIN_PASSWORD
    finally:
      - echo This always runs even if the login command fails 
  build:
    commands:
      - echo Entered the build phase...
      - echo Build started on `date`
      - mvn install
    finally:
      - echo This always runs even if the install command fails
  post_build:
    commands:
      - echo Entered the post_build phase...
      - echo Build completed on `date`

reports:
  arn:aws:codebuild:your-region:your-aws-account-id:report-group/report-group-name-1:
    files:
      - "**/*"
    base-directory: 'target/tests/reports'
    discard-paths: no
  reportGroupCucumberJson:
    files:
      - 'cucumber/target/cucumber-tests.xml'
    discard-paths: yes
    file-format: CUCUMBERJSON # default is JUNITXML
artifacts:
  files:
    - target/messageUtil-1.0.jar
  discard-paths: yes
  secondary-artifacts:
    artifact1:
      files:
        - target/artifact-1.0.jar
      discard-paths: yes
    artifact2:
      files:
        - target/artifact-2.0.jar
      discard-paths: yes
cache:
  paths:
    - '/root/.m2/**/*'
```

## Buildspec Example Build Docker Image to ECR
---
```YAML
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...          
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG      
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
```  

## 使用 Code Build 驗證 Code Commit Pull Request 

原始 AWS 文章 [Validating AWS CodeCommit Pull Requests with AWS CodeBuild and AWS Lambda](https://aws.amazon.com/tw/blogs/devops/validating-aws-codecommit-pull-requests-with-aws-codebuild-and-aws-lambda/)

![Center](/images/post/codebuild-certification-note/validating-aws-codecommit-pull-requests-with-aws-codebuild-and-aws-lambda.png#center)
