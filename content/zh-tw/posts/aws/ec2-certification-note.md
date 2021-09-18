+++ 
title = "EC2 Certification Note"
author = "Allen Hsieh"
description = "準備 AWS Certificate 時 EC2 相關的筆記，如果有資訊太舊或不正確請留言給我謝謝。"
featured = true
categories = ["AWS"]
tags = [
    "EC2",
]
date = "2021-09-03" 
aliases = ["ec2-certification-note"]
images = ["images/aws.jpeg"]
+++
## EC2 Purchase Option
---
### On-Demand Instances
- 用多少付多少，不需要提前預付也也不需要合約，根據秒來計費
- 不同 Region 有不同的 Instance & CPU 限制數量，可以透過 AWS Support Center 提高上限

### Reserved Instances
- 需要一年或三年的合約 ，以小時計費
- 付款選項
  - All Upfront (一次付清)
  - Partial Upfront (部分先付清）
  - No Upfront (用多少付多少）
- 方案類別
  - Standard (標準) 
    - 折扣較多，但無法更改 Instance Type
    - 可以在 Reserved Instance Marketplace 販賣
  - Convertible (可轉換) 
    - 折扣較少，但隨時可以硬體升降級
    - 不可以在 Reserved Instance Marketplace 販賣
- 範圍 (Scope)
  - Zonal Instance
    - 只能在指定的 AZ
    - 可以預留 capacity
  - Regional
    - 只能在指定的 Region
    - 同一 Region, 不同 AZ 可以一起折扣
    - 同一系列EC2 根據 [正規化因素](https://docs.aws.amazon.com/zh_tw/AWSEC2/latest/UserGuide/apply_ri.html#ri-normalization-factor) 在不同硬體規格可以有折扣，
      - 例如 t2.medium 正規化因素為 2 而 t2.small 為 1 ，當你購買一個 t2.medium 的 Reserved Instances，可以折扣在兩個 t2.small 的 instance。

### Spot Instance

- 競價模式: 請求時會自定義一個每小時最高可接受的價格(Spot Price)，而 Spot Price 會隨著市場而變動，當最高價格高於 Spot price ，就有機會獲得 Instance。當然當 Spot Price 低於最高價格， AWS 會隨時終止你的 Instance，所以跑在 Spot Instance 上的服務，需要設計可以隨時可以被終止
  - 如果因 AWS 而終止，未滿一小時的費用， AWS 不會收取任何費用。
- 每個 Region 可以限制 20 Spot Instance，可向 AWS 申請調高
- Spot Instance 被終止 EC2 能狀態有三種選擇 :
  - Hibernate
  - Stop
  - Terminate (default)
- Spot Block Mode : 可以指定一到六小時內，不會被 AWS 終止
- Spot Request Type 
	- one-time 
	- persistent
  {{< alert "在 persistent 啟用的狀態下終止 Instance，AWS 會重新分配一個 Instance 給你。" warning >}}
- Spot Instance 在被終止前兩分鐘 (grace period)，會有 warning，可以透過 Cloud Watch 或者 透過 EC2 metadata 取得
  - AWS 建議每五秒檢查一次 warning
  - EC2 取得 metadata，如果沒有任何 Action 會是 404
  ```
  curl : http://169.254.169.254/lasest/meta-data/spot/instance-action
  ```

### Dedicated Hosts & Instance

Dedicated Hosts & Dedicated Instance 都是只在實體機上執行你的程式，但兩者有點小小的差別

- Dedicated Hosts 是固定一台 EC2 給你使用，而 Dedicated Instance 只保證這台 EC2 上面的程式不會有其他人的，所以當你重啟 EC2 時，有可能是另一台 EC2。
- Dedicated Hosts 可以裝自己的 Software Licenses 而 Dedicated Instance 不行。
  - Note Dedicate Hosts 有自動容錯功能，發現EC2 壞掉時會自動更換一台新的 EC2，這邊可以整合 AWS License Manager 自動部署新的 Software Licenses
- Dedicated Hosts 如果購買 On-Demand 是已小時計算
- Dedicate Hosts 比 Dedicate Instance 貴

### Capacity Reservations

因為 AWS 在 AZ 裡面的實體硬體是有限的，所以是有可能當你啟動 EC2 Instance 時是有可能沒有足夠資源的，這時候只能等待其他人釋放 Instance 或者 選擇不同類型的 Instance。Capacity Reservations 可以讓你先提前先預付某些資源，這樣就不用擔心要啟用 EC2 Instance時沒有資源。在啟用新的 EC2 Instance 時可以直接選擇從 Capacity Reservations 裡面直接扣除，就不需要額外再付費了。但要注意，在 Capacity Reservations 的狀態是啟用時，就已經產生收費。

### Saving Plan

- Compute Savings Plans
  - 最高省66%
  - 折扣會自動套用在任何 EC2 Instance 上，不管是什麼操作系統、Instance Type 或者在任何 Region
- EC2 Instance Savings Plans
  - 最高省72%
  - 折扣限制在同一個 Region 下的 Instance Family
- 可以和 Capacity Reservations 混著使用。

### Spot Feet

- Spot Feet 會根據設定 (launch pools) 嘗試要到足夠的 Spot Instance ，剩餘的會使用 On-Demand Instance 補齊。
- 可以選擇的策略
  - lowestPrice: 從 Pool 中選出最便宜的方案
  - diversified: 分散到多個 Pool 中
  - capacityOptimized: AWS 會優化最佳 Capacity 去啟用 Spot Instance

## Instance Type
---
| Type | 描述     |
|------|----------|
| R    | RAM      |
| C    | CPU      |
| M    | General  |
| I    | I/O      |
| G    | GUP      |
| T    | can burst|


- Burstable Instance (T2/T3) : 當機器閒置時 AWS 會給 Burst Credits (有上限），之後有大量的 CPU 需求時可以得到額外的CPU 並從 Credit 中扣除。
  -  T3 可以無限 Burst ，但當然會需要支付額外的費用

## Instance Lifecycle
---
![Center](/images/post/aws-ec2-certification-note/ec2_instance_lifecycle.png#center)

- 以下狀態會收費 :
  - running
  - stop-Hibernate
- 使用 instance store 當 root volume 的 Instance 是不能 stop 的
- User Data : EC2 啟用時執行的 Script
  - 在 EC2 裡面取得 User Data 資訊 
  ```
  http://169.254.169.254/latest/user-data
  ```
  - 16 KB 大小限制
  {{< alert "當你停止(Stop) EC2 Instance，之後修改 User Data 以後，再重新啟動 (Start) EC2 Instance，這時候 EC2 Instance 還是會執行舊的 User Data" warning >}}

- Hibernate : （只限制在使用 EBS 的 Instance) : 在 Instance 停止時，將當下的RAM裡面的狀態複製一份到 EBS，之後重新啟用時，會將 EC2 還原到停止時的狀態。
  - EBS 需要是加密的
  - Hibernate 不能超過六十天
  - Instance Ram 必須小於 150 GB

![Center](/images/post/aws-ec2-certification-note/hibernation-flow.png#center)

|特徵\行為	| Reboot     | Stop/Start | Hibernate| Terminate|
|---------|------------|------------|----------|----------|
|實體機器	|同一台實體機器 |時通常是不同實體機器| 時通常是不同實體機器| X|
|IPV4 Address| 兩個固定一樣	| Private IPV4: 一樣 <br> Public IPV4: 新的，除非有 Elastic IP address| Private IPV4: 一樣<br> Public IPV4: 新的 除非有 Elastic IP address	 | X|
|Instance store volumes	|資料保留	|資料刪除	|資料刪除| 資料刪除|
|Root device volume	| 資料保留| 資料保留	|資料保留|資料預設刪除|
|RAM	|資料刪除	|資料刪除	|資料保留在 EBS	|資料刪除|

## Placement Group
---
- Cluster : 所有的 Instance 放在同一個 AZ ，以提供更快的網路速度
- Spread : 將所有的 Instance 放在不同的實體硬體，一個 AZ 最多七個 Instance
- Partitioned : 將 Ec2 Instance 按照自己的邏輯分開(partition) ，每個 AZ 上限七個 partition


## Storage
---
| 特徵\Type| EBS                                                                               | Instance Store                            |
|----------|-----------------------------------------------------------------------------------|-------------------------------------------|
|啟用時間  | 約一分鐘	                                                                       | 正常少於五分鐘                            |
|數據保存  |預設: 當 Ec2 被 Terminate 時， EBS 也會被砍掉。但可以設定不砍掉，並存留現在有的資料| 資料會隨著 Ec2 被被 Terminate 時一起刪除  |
|修改設定  |在 EC2 停止時可以被修改  	                                                       | 固定的不能修改                            |
|SnapShot  |支援 snapshot	                                                               | 不支援 snapshot                           |

## Monitor
---
- AWS 提供的 Metrics 
  - 包含資訊
     - CPU Utilization
     - Burst Credit 使用 和 餘額
     - Network In/Out (bytes)
     - Status Check
        - Instance Status : 檢查 EC2 VM 是否正常
        - System Status : 檢查硬體設備
     {{< alert "RAM 不包含在 AWS 提供的 Metrics" warning >}}
 - 模式
    - Basic : 免費，Metrics 每五分鐘間隔
    - Detailed : 需要付費，Metrics 每一分鐘間隔
- 客製化，自己裝 Unified CloudWatch Agent，將想要的 Metrics 或 Log 推到 Cloud Watch 上
   - CloudWatch Agent 設定可以統一方在 SSM Parameter Store 上
   - 可以取得更詳細的資訊，例如每個 Process 的 CPU 使用量

## Others
---
- Elastic IP address : 固定 IPv4 Address, 一個 Region 限制五個
- Enhanced Network 
      - Elastic Network Adapter: 可以提升網路數度至  100 Gbps
      - Virtual Function: 可以提升網路數度至 10 Gbps
- Elastic Fabric Adapter : 高效能運算使用


## Troubleshooting
---
### 不能啟用新的EC2
- InstanceLimitedExceeded Error : 每一個帳號在每一個 Region 都有一些限制，此錯誤代表達到上限，可以跟 AWS 提出將上限提高
- InsufficientInstanceCapacity Error : 代表現在 AWS 在這個 AZ 沒有足夠的資源提供，可以等一段時間或者換其他AZ 嘗試。
- Instance 直接 Terminate 有可能有以下幾個原因
    - 達到 EBS Volume Limit
    - EBS snapshot 壞掉了
    - EBS 有加密，但沒有 KMS 權限
    - AMI 有問題

    
### 不能 SSH 到 EC2 
- "Unprotected private key file" error : ssh key 需要是400 權限
- "Too many authentication failures" 有可能是 Username 不對
- "Connection timeout" 有可能是
    - Security Group 沒有設定對
    - NACL 沒有設定對
    - Route Table 沒有設定對
    - 沒有 Public IPv4 
    - CPU Loading 太高，機器無法回應

