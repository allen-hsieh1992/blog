+++
title = "CloudFormation Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 CloudFormation 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "CLOUDFORMATION",
]
date = "2021-09-12"
aliases = ["cloudformation-certification-note"]
images = ["images/aws.jpeg"]
+++


- 實踐 Infrastructure as Code，不需要手動建立資源，而且可以對 Source Code 做 Version Control。 
- CloudFormation 本身是免費的，只需要付建立的資源費用即可。 
- 可以透過 CloudFormation Designer 產生基本的架構圖
- Template 必須要上傳到 S3 ，Template 格式可以是 Yaml or Json
- Stack 是由一個 Template 產生的包含多個資源，Stack 名字當作識別。

## CloudFormation Template Section
---
- Parameters : 當要建立 Stack ，可以由外面設定動態參數
    - 可設定資訊
    - Type: String, Number, CommaDelimitedList, List<Type>, AWS Parameter
    - Description
    - Constraints
    - ConstraintDescription
    - Min/MaxLength
    - Defaults
    - AllowedValues (array)
    - AllowedPattern (regular expression)
    - NoEcho(Boolean)

```YAML
VpcId: !Ref MyVPC

Parameters:
  InstanceType:
    Type: 'AWS::SSM::Paramter::Value<String>'
    Default: /EC2/InstanceType
#there is public SSM parameters by AWS
```
- Conditions : 建立資源的條件
  - intrinsic function : Fn::And, Fn::Equals, Fn::If, Fn::Not, Fn::Or

```YAML
Conditions:
	CreateProdResources: !Equals [!Ref EnvType, prod] 

Resource:
  MountPoint:
    Type: "AWS::EC2::VolumeAttachment"
    Condition: CreateProdResources 
```

- Resource: 	要建立的 AWS 資源，此為必要欄位
    - 格式 : AWS::aws-product-name::data-type-name
- Mapping: 建立 Key Value Mapping 

```YAML
Mapping:
  RegionMap:
    user-east-1:
      "32" : "ami-aaa"
      "64" : "ami-bbb"
    user-east-2:
      "32" : "ami-ccc"
      "64" : "ami-ddd"

ImagesId: !FindInMap [RegionMap, !Ref "AWS:Region", 32] 
```

- Transforms: 主要用於 Serve-less Application Model (SAM) ，宣告這是 SAM 的格式
- Output: 可以將 Stack 建立的 Resource Output 出來，所以其他 Stack 可以建立 reference
    - 如果被其他 Stack reference ，那就不能 delete

```YAML
Outputs:
  StackSSHSecurityGroup:
    Description: The SSH Security Group for our Company
    Vaule: !Ref MyCompanyWideSSHSecurityGroup
  Export:
    Name: SSHSecurityGroup

Resource:
  MySecureInstance:
  Type: AWS::EC2::Instance
  Properties:
    AvailabilityZone: us-east-1a
    ImageId: ami-a5c7edb2
    InstanceType: t2.micro
    SecurityGroup:
      - !ImportValue SSHSecurityGroup
```
- Metadata: 提供額外有關 Template 資訊


##  Intrinsic Functions 
---
- Fn::Ref
   -  Parameters ⇒ 返回 parameter 對應的值 
   -  Resources ⇒ 返回資源的 physical ID 
-  Fn:GetAtt

```YAML
Resource:
  EC2Instance: 
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: ami-1234567
      InstanceType: t2.micro

NewVolume:
  Type: "AWS::EC2::Volume"
  Confition: CreateProdResources
  Properties:
    Size: 100
    AvailabilityZone:
    !GetAtt EC2Instance.AvailabilityZone
```

- Fn::Join

```YAML
"Fn::Join" : [ ",", [ "a", "b", "c" ] ] => "a,b,c"
```

- Fn::Sub

```YAML
Fn::Sub:
  - String
  - Var1Name: Var1Value
    Var2Name: Var2Value

Name: !Sub
  - www.${Domain}
  - { Domain: !Ref RootDomainName }	
```

- Fn::Split

```YAML
!Split [ "|" , "a|b|c" ] => ["a", "b", "c"]
```

- Fn::Select


```YAML
!Select [ "1", [ "apples", "grapes", "oranges", "mangoes" ] ]
```

- Fn::Base64
   - 可以用在 EC2 pass 整個 script 在 user data
   - user data script log ⇒ /var/log/cloud-init-output.log 

```YAML
Resource:
  EC2Instance: 
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: ami-1234567
      InstanceType: t2.micro
    UserData:
      Fn::Base64: |
        yum update -y
        yum install -y httpd
        systemctl start httpd
        systemctl enable httpd
        echo "Hello World from user data" > /var/www/html/index.html 
``` 

## cfn-init &  cfn-signal
---
- AWS::CloudFormation::Init 目的是讓 EC2 的設定可讀性更高， 需要放在 metadata section
- init 的 log 會寫在 /var/log/cfn-init.log
- cfn-init &  cfn-signal 流程
   -  CloudFormation 先建立 EC2 Instance ，如果有設定 WaitCondition 這時 CloudFormation 會等待 EC2 這邊的 signal
   - EC2 上面執行 User Data 時執行 cfn-init，這時 cfn-init 會去 CloudFormation 拉 metadata 的設定回來並執行 
   - cfn-signal 用於執行完 cfn-init 告訴 CloudFormation 成功與否  
- 沒有收到 Signal 怎麼辦
    - 確認 Script 是否有安裝
    - 可以disable rollback，進機器看  /var/log/cfn-init.log 和 /var/log/cloud-init-output.log
    - 由於 Signal 必須透過 Internet 通知 CloudFormation ，確認 EC2 可以 Access Internet. 


```YAML
Resource:
  EC2Instance: 
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: ami-1234567
      InstanceType: t2.micro
    UserData:
      Fn::Base64: |
        yum update -y aws-cfn-bootstrap
        # Start cfn-init
        /opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyInstance --region ${AWS::Region} || error_exit 'Failed to run cfn-init' 
        # Start cfn-signal to the wait condition
        /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource SampleWaitCondition --region ${AWS::Region}

Metadata:
  Comment: Install a simple Apache HTTP page
  AWS:CloudFormation::Init:
    config:
      packages:
        yum:
          httpd: []
      files:
        "/var/www/html/index.html";
        content: |
          <h1>Hello World from Ec2 Instance</h1>
        mode: '000644'
      commands:
        hello:
          command: "echo 'hello world'"
      services:
        sysvinit:
          httpd:
            enabled: 'true'
            ensureRunning: 'true'

SampleWaitCondition:
  CreationPolicy:
    ResourceSignal:
      Timeout: PT1M #等一分鐘
      Count:2 #要兩個成功的 Signal
    Type: AWS::CoudFormation::WaitCondition
```

## CloudFormation Rollback 
---
- Stack 建立失敗時，Stack 正常狀態會是 ROLLBACK_COMPLETE ，並且只能刪除不能更新
   - 預設 : 所有的資源會 rollback ，但 Stack 還在，可以看原因
       - OnFailure=Rollback
   - 可以 Disable rollback，所以可以做 troubleshooting 
       - OnFailure=DO_NOTHING
   - 可以刪除整個 Stack
       - OnFailure=Delete 
- Stack 更新失敗
    - Stack 如果 rollback 成功 Stack 狀態會是 UPDATE_ROLLBACK_COMPLETE ，可以嘗試更新或刪除
    - Stack 如果 rollback 失敗 Stack 狀態會是 UPDATE_ROLLBACK_FAIL，可以嘗試修復問題再繼續 rollback 

## Nested Stacks
---
- 可以重複使用其他 Stack 當作整個 Stack 一部分

```YAML
Resources:
  myStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL:
        xxx.com/...
      Parameters:
        key: value
```

## ChangeSet
---
- 可以在 CloudFormation 更新新的Template 之前先評估有什麼資源會變動，並確認是否要繼續執行，但執行了並不代表一定會成功。

## Drift
---
- CloudFormation 建立的資源是沒有預防人為手動更新的
- Drift 是用來檢查哪些資源跟當初 CloudFormation 建立的不一樣，並且告訴你哪些不同
{{< alert "不是所有的資源 Drift 都有支援。" warning >}}


## Deletion Policy
---
- 用來控制當 Stack 被刪除時特定資源要做的事情
- 三種 Policy
    - DeletionPolicy=Retain: 預防資源被刪除
    - DeletionPolicy=Snapshot: 先將資料備份再刪除
        - Support: EBS Volume, ElasticCache, Cluster, ElasticCache, ReplicationGroup, RDS DBInstance, RDS DBCluster, Redshift Cluster 
    - DeletionPolicy=Delete 刪除資源，但並非都會刪除成功，例如 S3 Bucket 裡面有 Object 就會刪除失敗
    - RDS 預設 Snapshot , 其他預設 Delete

## Termination Protection
---
- 用於保護 Stack 不會被不小心刪除

## Creation Policy & Update Policy
---
- 像 Auto Scaling Group 可以需要啟多個 Ec2 Instance，這時 Creation Policy 就可以設定在 Auto Scaling Group，要收時間內收多少個 Signal 才算成功
- UpdatePolicy
  - 如果直接更新 template 中的 user data，Auto Scaling Group 裡面的 EC2 還會是舊的 user data。
  - UpdatePolicy Attribute 可以使用在三種 AWS Resource
      - auto scaling group 有三種 Policy ，前兩種需要設定 WillReplace = true
          - AutoScalingReplacingUpdate: 建立全新的 Auto Scaling Group 取代舊的
          - AutoScalingRollingUpdate: 在 Auto Scaling Group，將現有的 EC2 Instance 停掉，並建立新的取代
          - AutoScalingScheduledAction 

          ```YAML
          Resources:
            AutoScalingGroup:
            Type: AWS::AutoScaling::AutoScalingGroup
            Properties:
              AvailabilityZones:
                Fn::GetsAZs:
                  Ref: "AWS::Region"
              LaunchConfigurationName:
                Ref: LaunchConfig
              Desiredcapacity: '3''
              MinSize: '1'
              MaxSize: '4'
            CreationPolicy:
              ResourceSignal:
              Count: '3'
              Timeout: PT15M
            UpdatePolicy:
              ## Example1 start
              AutoScalingRollingUpdate:
                MinInstancesInService: '1'
                MaxBatchSize: '2'
                #how much time to wait for the signal
                PauseTime: PT1M
                WaitOnResourceSignals: 'true'
              ## 預防CloudFormation 有排程去 update min,max or desired size 
              AutoScalingScheduledAction:
                IgnoreUnmodifiedGroupSizeProperties: 'true
              ## Example1 end
              ## Example1 and Example2 不會同時存在
          
              ## Example 2 start
              ## 會建立全新的 Auto Scaling Group 取代舊得
              AutoScalingReplacingUpdate:
                WillReplace: 'true'
              ## Example 2 end
          
          LaunchConfig:
            Type: AWS::AutoScaling::LaunchConfiguration
            Properties:
              ImageId: ami-aaa
              InstanceType: t2.micro
              UserData:
                Fn::Base64: |
                  yum update -y
                  yum install -y httpd
                  systemctl start httpd
                  systemctl enable httpd
                  echo "Hello World from user data" > /var/www/html/index.html 	
          ```
          
      - lambda alias
          - CodeDeployLambdaAliasUpdate
          ```YAML
          UpdatePolicy:
            CodeDeployLambdaAliasUpdate:
              AfterAllowTrafficHook: String
              ApplicationName: String
              BeforeAllowTrafficHook: String
              DeploymentGroupName: String
          ```

      - elastic cache replication group 
          - UseOnlineResharding
          ```YAML
          UpdatePolicy:
          EnableVersionUpgrade: Boolean
          ```

## Depends ON
---
- 控制資源建立的順序，以下面為例，EC2 Instance 需要在 RDS 建立好以後才建立


```YAML
Resources:
  Ec2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId:
        Fn::FindInMap:
        - RegionMap
        - Ref: AWS::Region
        - AMI
    DependsOn: myDB
  myDB:
    Type: AWS::RDS::DBInstance
```


## Stack Policy
---
- 像是 IM Policy ，可以用來控制哪些 Resource 可以做什麼，哪些不能做什麼

```JSON
{
  "Statement" : [
    {
      "Effect" : "Allow",
      "Action" : "Update:*",
      "Principal": "*",
      "Resource" : "*"
    },
    {
      "Effect" : "Deny",
      "Action" : "Update:*",
      "Principal": "*",
      "Resource" : "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```
## StackSets
---
- 可以一次更動多個 Region 或多個 Account 的 Stack 
- 只有 Administrator Account 可以建立 StackSets
- Trust Account 可以更新 StackSets
- 每次更動 StackSets 設定，全部的 Stack 都會被更新
- 可以刪除整個 StackSets 或單獨刪除 StackSets 裡面的一個 Stack
