+++
title = "CodeCommit Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 CodeCommit 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "CodeCommit",
]
date = "2021-09-13"
aliases = ["codecommit-certification-note"]
images = ["images/aws.jpeg"]
+++

## Basic 
- AWS 的 Git Version Control 服務，像是 Github
- 可以跟 CodeBuild 、 Jenkins 或者其他 CI 整合
- 可以透過 HTTPS or SSH connect，可以建立 IAM User 並在裡面設定 
	- ssh key 
	- https git credential (user name and password)  
- 可以透過 CloudTail 監控哪個 IAM User 在操作什麼


## Repository
---
- 每個 Repository 沒有 Size 限制
- 可以建立 Notification Rule ，根據設定的 Event 發出 SNS 
- 可以建立 Event Trigger 觸發 SNS 或 Lambda 
- 可以給予其他 AWS Account User 讀取權限
- 可以透過 KMS 加密 Repository 裡面的 File
	
## Pull Request 
---
- 可以整合 CodeGuru Review，在 pull request 建立時，可以透過 CodeGuru 去分析 Code 並抓出常見問題。  
- 建立  approval rules ，指定的 User 才可以 Merge pull request
```BASH
aws codecommit create-pull-request-approval-rule \
--pull-request-id 27 \
--approval-rule-name "Require two approved approvers" \
--approval-rule-content "{\"Version\": \"2018-11-08\",\"Statements\": [{\"Type\": \"Approvers\",\"NumberOfApprovalsNeeded\": 2,\"ApprovalPoolMembers\": [\"CodeCommitApprovers:123456789012:Nikhil_Jayashankar\", \"arn:aws:sts::123456789012:assumed-role/CodeCommitReview/*\"]}]}"
```
- 也可以建立 approval rules template 並重複使用，綁定在不同 repository 上
```BASH
aws codecommit create-approval-rule-template \
--approval-rule-template-name 2-approver-rule-for-main \
--approval-rule-template-description "Requires two developers from the team to approve the pull request if the destination branch is main" \
--approval-rule-template-content "{\"Version\": \"2018-11-08\",\"DestinationReferences\": [\"refs/heads/main\"],\"Statements\": [{\"Type\": \"Approvers\",\"NumberOfApprovalsNeeded\": 2,\"ApprovalPoolMembers\": [\"arn:aws:sts::123456789012:assumed-role/CodeCommitReview/*\"]}]}"
```

## 透過 IAM 限制 Push & Merge Master

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "codecommit:GitPush",
                "codecommit:DeleteBranch",
                "codecommit:PutFile",
                "codecommit:MergeBranchesByFastForward",
                "codecommit:MergeBranchesBySquash",
                "codecommit:MergeBranchesByThreeWay",
                "codecommit:MergePullRequestByFastForward",
                "codecommit:MergePullRequestBySquash",
                "codecommit:MergePullRequestByThreeWay"
            ],
            "Resource": "arn:aws:codecommit:us-east-2:111111111111:MyDemoRepo",
            "Condition": {
                "StringEqualsIfExists": {
                    "codecommit:References": [
                        "refs/heads/main", 
                     ]
                },
                "Null": {
                    "codecommit:References": "false"
                }
            }
        }
    ]
}
```
