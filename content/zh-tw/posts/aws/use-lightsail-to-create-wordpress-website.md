+++
title = "使用 Lightsail 架設 WordPress 網站"
author = "Allen Hsieh"
description = "在大學時間，我曾經使用過 WordPress.com 只是最後覺得 WordPress.com 限制太多，想要達到我想要的效果，又需要額外花錢，所以最後就放棄了。這邊不會花太多時間解釋 WordPress com & org 的差別，如果有興趣的人，可以參考這邊文章 WordPress.org 和 WordPress.com 有什麼不一樣？。 2016年，AWS 宣布新的 Lightsail 服務上市。Lightsail 這邊使用 Bitnami 一鍵部署很快， 不用自己啟一個 EC2 & MySql Server 然後還要設定網路， WordPress 也會安裝好。這邊介紹直接用 Lightsail Console 頁面直接部署。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "LIGHTSAIL",
    "ROUTE53",
]
date = "2021-07-04"
aliases = ["use-lightsail-to-create-wordpress-website"]
images = ["images/aws.jpeg"]
+++

在大學時間，我曾經使用過 WordPress.com 只是最後覺得 WordPress.com 限制太多，想要達到我想要的效果，又需要額外花錢，所以最後就放棄了。這邊不會花太多時間解釋 WordPress com & org 的差別，如果有興趣的人，可以參考這邊文章 WordPress.org 和 WordPress.com 有什麼不一樣？。 2016年，AWS 宣布新的 Lightsail 服務上市。Lightsail 這邊使用 Bitnami 一鍵部署很快， 不用自己啟一個 EC2 & MySql Server 然後還要設定網路， WordPress 也會安裝好。這邊介紹直接用 Lightsail Console 頁面直接部署。

## Lightsail 架設 WordPress Server

### Step1 進入 Lightstail 服務，點下 Create instance

![Center](/images/post/use-lightsail-to-create-wordpress-website/Lightsail1.png#center)

### Step 2 選擇 地區、服務和方案。

![Center](/images/post/use-lightsail-to-create-wordpress-website/Lightsail2.png#center)

![Center](/images/post/use-lightsail-to-create-wordpress-website/Lightsail3.png#center)

由於我是新網站，所以這邊選用有一個月免費試用的 3.5 美金一個月的方案。


## 使用 bncert 建立 HTTPS

透過UI，可以直接連線到 Instance 上

![Center](/images/post/use-lightsail-to-create-wordpress-website/ssh_to_lightsail_instance.png#center)

使用 bncert 安裝憑證，我的機器一開始上面安裝有安裝 bncert。

```bash
# 安裝憑證
sudo /opt/bitnami/bncert-tool
# 如果沒有 bncert-tool 可以執行以下 command
wget -O bncert-linux-x64.run https://downloads.bitnami.com/files/bncert/latest/bncert-linux-x64.run
sudo mkdir /opt/bitnami/bncert
sudo mv bncert-linux-x64.run /opt/bitnami/bncert/
sudo chmod +x /opt/bitnami/bncert/bncert-linux-x64.run
sudo ln -s /opt/bitnami/bncert/bncert-linux-x64.run /opt/bitnami/bncert-tool
```

執行bncert-tool，一開始會詢問 Domain List 和 是否將 Http 近來的頁面全部轉到 Https

![Center](/images/post/use-lightsail-to-create-wordpress-website/bncert2.png#center)

之後會確認改動和填寫憑證上面的 mail

![Center](/images/post/use-lightsail-to-create-wordpress-website/bncert3.png#center)

執行結果

![Center](/images/post/use-lightsail-to-create-wordpress-website/bncert4.png#center)

## Router 53 購買域名
---

進入Router53 購買域名頁面，點選 Register Domain

![Center](/images/post/use-lightsail-to-create-wordpress-website/router53_1.png#center)

這邊可以搜尋你要的 domain，同時 Router 53 也會提供其他建議。

![Center](/images/post/use-lightsail-to-create-wordpress-website/router53_2.png#center)

確認要購買域名時，需要填寫個人資訊

![Center](/images/post/use-lightsail-to-create-wordpress-website/router53_3.png#center)

購買完以後就可以在 Rounter 53 上確認自己的 Domain 狀態

![Center](/images/post/use-lightsail-to-create-wordpress-website/router53_4.png#center)

申請完 Domain 需要一點時間喔～艾倫這邊等了大約兩天才下來。

## 建立新的 index 到 Google Search
---

這邊順便介紹一下，我目前在 Worldpress 使用的 SEO Plugin Yoast SEO ，安裝完以後每一個 Post & Page 下面可以設定一些 keyphrase、Meta description 等等資訊，也會提供每一個 Status 作為參考。

![Center](/images/post/use-lightsail-to-create-wordpress-website/seo_example.png#center)

這邊簡單一個方法可以確認 Google 是否有 index 你的網站，新的網站通常都沒有資訊，在 Google 打上 “site:domainName”

![Center](/images/post/use-lightsail-to-create-wordpress-website/google_site_result.png#center)

{{< alert "以艾倫的 allenhsieh1992.com ，由於我已經有設定幾天了，所以 Google 有 index 了，反則就會只有一個請使用 Google Search Console 的頁面" warning >}}



## Google Search Console 驗證網域
---

Google Search Console 是一個 Google 讓我們告訴他現在新的網域方法。目前有兩種驗證方法，這邊建議使用網域的新功能，因為使用網域前置字元，會把 HTTPS 和 HTTP 當作不同的網域處理。

![Center](/images/post/use-lightsail-to-create-wordpress-website/google_search_console_add_domain.png#center)

我一開始使用右邊的方式 "網址前置字元"，但因為我一開始架設網站時，不是先設定HTTPS，所以先驗證了HTTP。由於之後犯蠢登入不同的Gmail，所以以為HTTP 沒驗證成功，又驗證了一次HTTPS。事後發現 HTTP & HTTPS 是可以分開建立成功的，所以這邊會教學並建議使用"網域"的方式，並免大家跟我犯一樣的錯誤

![Center](/images/post/use-lightsail-to-create-wordpress-website/google_search_console_add_domain1.png#center)
![Center](/images/post/use-lightsail-to-create-wordpress-website/google_search_console_add_domain2.png#center)

### Step1 輸入要驗證的網域

![Center](/images/post/use-lightsail-to-create-wordpress-website/add_domain_step1.png#center)

### Step2 複製 Google 提供的 TXT Record 設定資訊

![Center](/images/post/use-lightsail-to-create-wordpress-website/add_domain_step2.png#center)

### Step3 在 Router 53 建立 Txt Record

![Center](/images/post/use-lightsail-to-create-wordpress-website/add_domain_step3.png#center)

{{< alert "Record Type 記得選 Txt 和 Value 貼上 Type2 的資訊" warning >}}

### Step4 回到 Step2 頁面點選驗證 ～

驗證成功就會在 Google Search Console 看到你的 Domain


## 上交 Sitemap 到 Google Search Console
---

Sitemap 主要包含了網站結構資訊，上交 Sitemap 可以讓 Google 更好地瞭解你的網站。這邊的 Sitemap 是由之前所提的 Yoast SEO Plugin 所產生的


### Step1 取得 Sitemap endpoint

在 Yoast 介面的 Features 下面有一個 XML Sitemaps 的設定，點選右邊的問號，在點下 See the xml sitemap，這時候就會直接打開 Sitemap 的頁面

![Center](/images/post/use-lightsail-to-create-wordpress-website/Sitemap.png#center)

Sitemap 實際內容就這個網域下所有的網頁，並且最後修改時間

![Center](/images/post/use-lightsail-to-create-wordpress-website/Sitemap2.png#center)

### Step2 上交 Sitemap 到 Google Search Console

![Center](/images/post/use-lightsail-to-create-wordpress-website/Sitemap3.png#center)

接著等待幾天後，就可以在 Google 透過 site 找到自己的網站

## 總結
---

這次架完以後，挺佩服那些沒有資訊背景又自己一手包辦，從零開始架設WorldPress 網站的人。這次的網站，雖然不如我內心所想要的外觀，無奈前端不熟，如果想要用出自己的外觀，又要花不少時間。所以艾倫我決定先增加這個網站的內容，將來這個網站如果我有動力繼續做下去，再花時間修改外觀。


