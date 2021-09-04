+++
title = "使用 AWS Cognito CLI 建立 User Pool"
author = "Allen Hsieh"
description = "個人用戶管理對於許多網站 或 App 都是不可缺少的，大家有可能會經常煩惱要怎麼樣讓自己的服務，可以透過 Google 、Facebook 等第三方認證，或者擁有自己的使用者群。這時可以選擇考慮 AWS Cognito，它可以讓你快速的為你的服務建立使用者群，並且可以整合讓使用者擁有權限操作其他 AWS 服務 。由於使用 UI 建立 Cognito User Pool 的文章已經不少，所以這邊文章會介紹使用 AWS CLI cognito-idp 建立一個用戶可以註冊，並且自己透過 Mail 認證通過的 User Pool。"
featured = true
categories = ["AWS"]
tags = [
    "AWS",
    "COGNITO",
]
date = "2021-07-11"
aliases = ["create-cognito-user-pool-by-aws-cli"]
images = ["images/aws.jpeg"]
+++


個人用戶管理對於許多網站 或 App 都是不可缺少的，大家有可能會經常煩惱要怎麼樣讓自己的服務，可以透過 Google 、Facebook 等第三方認證，或者擁有自己的使用者群。這時可以選擇考慮 AWS Cognito，它可以讓你快速的為你的服務建立使用者群，並且可以整合讓使用者擁有權限操作其他 AWS 服務 。由於使用 UI 建立 Cognito User Pool 的文章已經不少，所以這邊文章會介紹使用 AWS CLI cognito-idp 建立一個用戶可以註冊，並且自己透過 Mail 認證通過的 User Pool。

## 前置作業
---
1. AWS CLI
2. AWS SES 驗證過的 Email

## 透過 AWS Console 設定 SES
---
### Step1 註冊 SES Mail

這邊要注意，Cognito 目前允許的 SES Region 只有三個，分別是 us-east-1、us-east-2、eu-west-1。所以在建立 Email 時，請確認要別確認 Region 喔，不然之後有可能會無法設定喔

請到 SES 下面，點選 Email Address，然後點下 Verify a New Email Address，並填寫你的 Mail

![Center](/images/post/create-cognito-user-pool-by-aws-cli/Ses_register_1.png#center)

當你填完以後，這時可以你的 Email 在 list 中，而 Verification Status 還會在 Pending 喔～ AWS 會寄 Mail 到你的郵箱，請去填寫的 Email 認證。

![Center](/images/post/create-cognito-user-pool-by-aws-cli/Ses_register_2.png#center)

### Step2 驗證 Mail

這時候當入剛剛輸入的 Mail 中，應該會有一封 Email Address Verification Request 的 Mail ，這時候請點選裡面的 Link 完成驗證

![Center](/images/post/create-cognito-user-pool-by-aws-cli/Ses_register_3.png#center)

### Step3 確認信箱已驗證

這時候重新進入 SES 頁面，會發現 Verification Status 是 verified

![Center](/images/post/create-cognito-user-pool-by-aws-cli/Ses_register_4.png#center)

如果你也在嘗試練習，請點選 UI 的 Email ，這時候可以看到 Identity ARN。請將 Identity ARN 記錄下來，後面建立 User Pool 會使用到。

![Center](/images/post/create-cognito-user-pool-by-aws-cli/Ses_register_5.png#center)


## 使用 AWS CLI cognito-idp 建立 User Pool & Client

aws [cognito-idp](https://docs.aws.amazon.com/cli/latest/reference/cognito-idp/index.html) command 主要是讓我們來管理 User Pool 的，裡面有分兩種不同的 level 的 command ，分別是 ADMIN 和 一般 User 的， 所有 admin 的 command 都會是 admin- 開頭的。以下是附上 AWS 官方的圖片，一開始 User 在 Cognito 註冊是在 已註冊狀態(Registered) 是不能 Login 的，必須要是認證狀態(Confirmed) ，才能 Login。而目前只有兩種方式才能讓 User 變成認證的，第一種就是由 ADMIN 這邊認證，而第二種就是 User 可以自己認證透過電話或 Mail。這邊要示範透過 Mail 認證

![Center](/images/post/create-cognito-user-pool-by-aws-cli/amazon-cognito-sign-in-confirm-user.png#center)

### Step1 使用 AWS cognito-idp 列出目前所擁有的 User Pool

使用 list-user-pools command 列出目前擁有的全部 User Pool，目前艾倫的 User Pool 都已經清空，所以沒有。

```bash
$ aws cognito-idp list-user-pools --max-results 10

#Result
{
    "UserPools": []
}
```

### Step2 使用 cognito-idp 建立 User Pool

如果正在使用這邊文章做範例請紀錄 ID，後面建立 Client 需要使用

這邊使用 create-user-pool command

* --pool-name User Pool 的名字
* --auto-verified-attributes 想要驗證的欄位，這邊是 Mail
* --username-attributes 這邊特別指定 User Name 可以透過 Email 替代 ，這邊可以不強制 User Name 使用 Email，也可以讓 User 自定義自己的 Uesr Id
* --email-configuration 這邊要定義 Cognito 寄出的 Mail Server，艾倫這邊使用 SES 寄出，SourceArn 記得改成你的 SES ARN

這邊的 Result 由於太長，所以艾倫有做一些刪減。Result 中的 VerificationMessageTemplate ，裡面寫 CONFIRM_WITH_CODE， 當 User Sign up 時， Cognito 會產生一組驗證碼寄送到 Mail 中，這樣 User 就可以使用驗證碼來自己驗證。另外可以看到下面 PasswordPolicy，至少要一個大小寫英文字母，並且要有符號和數字，最後長度必須要八以上，這些條件都是可以客製化的，不過可以看到，艾倫在建立 User Pool 時，沒有帶入特定參數，所以這些目前是使用預設的。


```bash
$ aws cognito-idp create-user-pool \
 --pool-name helloWorld \
 --auto-verified-attributes email \
 --username-attributes "email" \
 --email-configuration=SourceArn="arn:aws:ses:us-east-1:129824596365:identity/allen.hsieh.aws@gmail.com",ReplyToEmailAddress="allen.hsieh.aws@gmail.com"

# Result
{
     "UserPool": {
         "Name": "helloWorld",
         "VerificationMessageTemplate": {
             "DefaultEmailOption": "CONFIRM_WITH_CODE"
         },
         "EmailConfiguration": {
             "EmailSendingAccount": "COGNITO_DEFAULT",
             "ReplyToEmailAddress": "allen.hsieh.aws@gmail.com",
             "SourceArn": "arn:aws:ses:us-east-1:129824596365:identity/allen.hsieh.aws@gmail.com"
         },
         "Policies": {
             "PasswordPolicy": {
                 "RequireNumbers": true,
                 "RequireLowercase": true,
                 "RequireSymbols": true,
                 "RequireUppercase": true,
                 "TemporaryPasswordValidityDays": 7,
                 "MinimumLength": 8
             }
         },
         "Id": "ap-northeast-1_DpExb5BW8",              "Arn: "arn:aws:cognito-idp:ap-northeast-1:129824596365:userpool/ap-northeast-1_DpExb5BW8"
}
```

### Step3 確認 User Pool

再次使用 Step1 的 list-user-pools Command，這時後會看到 User Pool。

```bash
$ aws cognito-idp list-user-pools --max-results 10 

# Result                                                                                                         
 {
     "UserPools": [
         {
             "CreationDate": 1625996882.861,
             "LastModifiedDate": 1625996882.861,
             "LambdaConfig": {},
             "Id": "ap-northeast-1_DpExb5BW8",
             "Name": "helloWorld"
         }
     ]
 }
```

### Step4 建立 User Pool 的 Client

一個 User Pool 可以有多個 Client ，例如想要區分 App, Web 等等

如果正在使用這邊文章做範例請紀錄 ClientId，後面建立 User 需要使用

* --user-pool-id 這邊需要 Step3 or Step2 的 ID (User Pool 的 ID)
* --explicit-auth-flows 特別指定驗證方式， USER_PASSWORD_AUTH 是同意使用 User & Password 去做驗證，這邊是因為後面方便要 Demo 使用 Cli 去做 Login 的關係，否則正式環境 App 建議使用 (SRP)[https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-authentication-flow.html] 驗證。
* --client-name: Client 的名字

```bash
$ aws cognito-idp create-user-pool-client \
   --user-pool-id  ap-northeast-1_DpExb5BW8 \
   --explicit-auth-flows  USER_PASSWORD_AUTH \
   --client-name cli

# Result
{
     "UserPoolClient": {
         "UserPoolId": "ap-northeast-1_DpExb5BW8",
         "LastModifiedDate": 1625999636.182,
         "ClientId": "400d11al9p25pmjg3igflo5h4i",
         "AllowedOAuthFlowsUserPoolClient": false,
         "TokenValidityUnits": {},
         "ExplicitAuthFlows": [
             "USER_PASSWORD_AUTH"
         ],
         "RefreshTokenValidity": 30,
         "CreationDate": 1625999636.182,
         "EnableTokenRevocation": true,
         "ClientName": "cli"
     }
```

## 建立 & 驗證新 User

這邊需要使用建立 Client 時的 Client Id ，如果忘了拷貝下來，可以使用 list-user-pool-clients 取得。

```bash
$ aws cognito-idp list-user-pool-clients --user-pool-id ap-northeast-1_DpExb5BW8                                                                        

# Result
 {
     "UserPoolClients": [
         {
             "ClientName": "cli",
             "UserPoolId": "ap-northeast-1_DpExb5BW8",
             "ClientId": "400d11al9p25pmjg3igflo5h4i"
         }
     ]
 }
```

### Step1 建立新 User

* --client-id 從 Create Client 時拿掉
* --username 這邊要填寫 User Name ，由於艾倫建立的 User Pool 強制要求需要使用 Email 當 User Name，所以要使用 Email 喔
* --password 密碼，記得要 Follow 之前的 PasswordPolicy

```bash
$ aws cognito-idp sign-up \
   --client-id 400d11al9p25pmjg3igflo5h4i \
   --username  allen.hsieh.aws@gmail.com \
   --password  Aa123456!

# Result
{
     "UserConfirmed": false,
     "UserSub": "c57c5fb9-7fbd-406d-9bb3-19f88cfc9ac0",
     "CodeDeliveryDetails": {
         "AttributeName": "email",
         "Destination": "a***@g***.com",
         "DeliveryMedium": "EMAIL"
     }
 }
```

這時後透過 AWS Console 進入 User Pool 裡面的 Users & Groups 裡面，可以看到剛剛註冊的用戶，這邊可以看到 Account Status 是 UNCONFIRMED


![Center](/images/post/create-cognito-user-pool-by-aws-cli/user-pool-user1.png#center)

### Step2 嘗試 Login

Step1 註冊 Command 是 Sign-up，這邊是驗證，而並非登入，所以 Command 不是 Login ，而是 initiate-auth

* --auth-flow 驗證方式，這邊使用 USER_PASSWORD_AUTH 單純使用 UserName & Password 。

這邊要特別注意， Client 必須要 Allow 可以使用 USER_PASSWORD_AUTH，艾倫在 Create Client 時有特別特定

* --auth-parameters 這邊是 USERNAME & PASSWORD

結果會是噴 Exception ，原因是 Account Status 還是 UNCONFIRMED

```bash
aws cognito-idp initiate-auth \
 --auth-flow USER_PASSWORD_AUTH \
 --client-id 400d11al9p25pmjg3igflo5h4i \
 --auth-parameters USERNAME=allen.hsieh.aws@gmail.com,PASSWORD=Aa123456!

# Result
An error occurred (UserNotConfirmedException) when calling the InitiateAuth operation: User is not confirmed. 
```

### Step3 驗證 User

Cognito 會寄送 verification code 到你的 mail ，信箱內應該要有一封 email title 叫做 Your verification code，如果沒有看到，建議先到垃圾郵箱裡面查看。


![Center](/images/post/create-cognito-user-pool-by-aws-cli/verification_code_mail.png#center)

這邊使用 confirm-sign-up 去認證 ，注意正常認證成功是不會有任何 result

```bash
aws cognito-idp confirm-sign-up \
 --client-id 400d11al9p25pmjg3igflo5h4i \
 --username  allen.hsieh.aws@gmail.com \
 --confirmation-code 328625
```

如果沒有收到 Email 或覺得麻煩 ，想要直接認證的話，可以先使用 ADMIN 的權限認證

```bssh
aws cognito-idp admin-confirm-sign-up \
  --user-pool-id ap-northeast-1_DpExb5BW8 \
  --username  allen.hsieh.aws@gmail.com
```

### Step4 確認驗證

這時後透過 UI，可以看到 Account Status 已經是 CONFIRMED

![Center](/images/post/create-cognito-user-pool-by-aws-cli/user-pool-user2.png#center)

### Step5 再次 Login

這時候，再次 call initiate-auth 就會成功，而且這時候 Result 會有三個 Token，分別是 IdToken, AccessToken 和 RefreshToken，這三個 Token 都是使用 JWT Token。RefreshToken 就如字面的意思，主要用來更新 Token。Response 中有 ExpiresIn 這個是告知 Token 過期時間，這邊單位是秒。IdToken 主要有包含客戶的一些資訊（例如 email, phone)，可以知道這個人是誰。而 AccessToken 主要是給你 call 一些不需要知道個人資訊的 API，API 只是單純要知道這個 User 擁有權限就可以使用 AccessToken。這邊也可以使用 JWT.io 將 Token decode 回來，看看裡面有什麼資訊。

```bash
aws cognito-idp initiate-auth \
 --auth-flow USER_PASSWORD_AUTH \
 --client-id 400d11al9p25pmjg3igflo5h4i \
 --auth-parameters USERNAME=allen.hsieh.aws@gmail.com,PASSWORD=Aa123456!

# Result
{
     "AuthenticationResult": {
         "ExpiresIn": 3600,
         "IdToken": "eyJraWQiOiJhTmJjOWZpbUw5TjUxWWk2VkV6VlUwZ1EyZUdkRFVPNElQVURYYmN4MVhFPSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiJjNTdjNWZiOS03ZmJkLTQwNmQtOWJiMy0xOWY4OGNmYzlhYzAiLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiaXNzIjoiaHR0cHM6XC9cL2NvZ25pdG8taWRwLmFwLW5vcnRoZWFzdC0xLmFtYXpvbmF3cy5jb21cL2FwLW5vcnRoZWFzdC0xX0RwRXhiNUJXOCIsImNvZ25pdG86dXNlcm5hbWUiOiJjNTdjNWZiOS03ZmJkLTQwNmQtOWJiMy0xOWY4OGNmYzlhYzAiLCJvcmlnaW5fanRpIjoiNGVlYmIyMTQtMzBlOC00N2Q3LWJjOGMtODA3NjM5ZTljMzU2IiwiYXVkIjoiNDAwZDExYWw5cDI1cG1qZzNpZ2ZsbzVoNGkiLCJldmVudF9pZCI6IjM3Nzk5N2RlLTFhMWYtNDA5ZS1iNjM1LWVjNzk2ZTBjZTEwYiIsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNjI2MDAzNjU5LCJleHAiOjE2MjYwMDcyNTksImlhdCI6MTYyNjAwMzY1OSwianRpIjoiOWZkN2M1MjYtNTBkMy00NWFmLTg5YmUtOWMxNWMzMjMzMjUxIiwiZW1haWwiOiJhbGxlbi5oc2llaC5hd3NAZ21haWwuY29tIn0.Asp7HF1mpPzWUE3l_7S8EH1nY8x9y3o2NK_8fBb_-Hz0PPXTkPGTvjqooEbHRLGGWAf8d7jwoBbsqV3jhFN-UHvrfcAYNRX_DAOeyxi4tCDTQXDk_Ozu8bCU9Clmb5zyg8Jo_SBAZdnKJewbWLEuhqi9_g7AHQrI_sufec3G6L8Ufr3mdfm-eTGtMcICmgkCxImd_zVgWSGkT5gUgcN0DvFzu6cqWC6kjaM3UoUx1uJ-iLpgA2rXzoBEKI2MqGWlIFSJcxL-IFtZyBvU2k4WNDrT5wN1s30UBa6_zeP_-8ZOp-BdU5D4omWltJfns16fqsPBmhbVoyuycamhxv_htQ",
         "RefreshToken": "eyJjdHkiOiJKV1QiLCJlbmMiOiJBMjU2R0NNIiwiYWxnIjoiUlNBLU9BRVAifQ.BJz_JPV5gRBtqiMLhiQroXg0GsTg33jpT9JOlpT1COkmYCTQBfYblnkoGb4tNCL7y7JWreHJDAFXRafaq4fYkOU9gob0FSxUv3MhVs8V1AKiWunM-iytuKFjNALe2Xlmsa70hBcjiEda5agWMdoYG8Q4eL59-ogmJUgsn46h69-B2cxnWx7RQQ9RwJpauLV8zEEKCUYkVE7C5umlceJjQI0OWN76fcoWoCdx-IUgAJEBf72PJ0KBWeZ1p4ElSMWFtg3Egso-OcES1rzW1gE1qG7N_Hcn_58td8fnQAZEONMxPbHzH2C6AaUPF1NaLcQl9mRSs7KgODZkGFQ6FmAc8Q.jitGP9kMECesDsZj.QODsHRk2PYwx9LktJrkbGz__7i2eh7Fz7ueI5qrgrR3uK0nC5jU06RVuMQqn9m_EnlWh0dj9HEU_e9j5XMrbeRA4ZCJkL3wlsZmfRFfF04pAhjO7H7_pxsq_-gGw-5ZNGyh0osd3MICYIYBoJ90RrRHetKKfna9X_hktv8f9A63lqjZLSWDFhAWmXYD_UiLsi7Pf9JAw5g9w3MGYxQ2RN_KRmoaxQg213_6r4fZjqpH0QDpo7Y7bsMu82kGusv8HXwP-O4l7pE6IMTH8k2oX2q2Yg85o-fwEjFZmIiCwnkQfOK607I9qUIN1w8ACtfW6KbN8AXch3CXsnyYeZbprAtgzwYJCERIImfv6UakKwvqlzBCIr3o_idkHnBJrn4HnVhlNTzttl6cyySvO3ombFdHWfn8KbztaAVAZwazQC5WNTm6sCwvz5oR30A-i2taDeT7gwXe_CSg-YiSG8ODKCcTBOeCmCKOpx2Z7maMBP1dAsKqHQ_0-Ozfp_VPJ57TsjTVyLiUCwOCKlZPcMJbKz_svgkc7yTHv9EAYklOLDC3eEEvIIuhXqkmdW0CEvlwvfQLXiRvMbrVPxxGvOrHyIN7f7MSEY8NjZZ-LaXxCg7ilCHEk_pNGn8tru6r_FjTs657tVnxRg0CL5yOu-heKwcYXrK1syS5YRWYi4DYwkBI2_2ymPOlWxZY94cM6LfJpW7EVFxvF7ZUu3aMm4F2s5lQsFuWo6LNIm7umdpm6hJyP6lsgzn3VHsHXBN1WQZDqqUWmVSOcAVQjzvYVaisin90r5LSwMqGE2cxgSzlykE7tSwzZbK8I3sO-iUwdZoaVQYHqSKEa9dwRUc_y3yJxfo81vaLvy73FFg2QnTUA8l6mjziRk0-pG3aM6ODneCGewXm0lyZ6sTbzk2euT6ohrj0CgdJ3kDe1IS0IyV8G5wd_bxE0a5U5IVMnPMexZo-HeJxMbiRKh7dFZ3lCoLvJ3r9vfOGIJCMCo5u-nCC9KkQDJQY5P8mWbKRh4pO6XmPMIj4c96reguvnXmm-eA09y89hQm9AkOafnjMDhYRNEISr0WO1zP5GQRpY98BRdz5T-e8xzQKufoerg3Ogalciw_r6FJgjUJeyfG9PEQlEQXz4nvfLFI5DwHXs768g-_02i0tRZ-mVl8cXRMQ2Sa8L-w2ptvznmayCLEL8rMdggGEHQZSuQKP5pmQu9Mq-nP4mWjR2GWey-UAMLHTZ1l-VsoextWfwdsZaTRJQNXUmOpHgZHHYb8yAOae6nQP60CNGA9DnvAkalrhaNZlOdXG5paDG8hUz1wNFsOLunEe3fhon573QivWx0dUOmxf2DGCqlDC1GrB2UvY6lw.1uaq1g1lQ9BBIMa3X5VG1Q",
         "TokenType": "Bearer",
         "AccessToken": "eyJraWQiOiJKcXVVM0N3dk1YNW1oMTNhb0ZrRnp6VTJ5VytIdXcwUnR4cGdRSXZEVFIwPSIsImFsZyI6IlJTMjU2In0.eyJvcmlnaW5fanRpIjoiNGVlYmIyMTQtMzBlOC00N2Q3LWJjOGMtODA3NjM5ZTljMzU2Iiwic3ViIjoiYzU3YzVmYjktN2ZiZC00MDZkLTliYjMtMTlmODhjZmM5YWMwIiwiZXZlbnRfaWQiOiIzNzc5OTdkZS0xYTFmLTQwOWUtYjYzNS1lYzc5NmUwY2UxMGIiLCJ0b2tlbl91c2UiOiJhY2Nlc3MiLCJzY29wZSI6ImF3cy5jb2duaXRvLnNpZ25pbi51c2VyLmFkbWluIiwiYXV0aF90aW1lIjoxNjI2MDAzNjU5LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAuYXAtbm9ydGhlYXN0LTEuYW1hem9uYXdzLmNvbVwvYXAtbm9ydGhlYXN0LTFfRHBFeGI1Qlc4IiwiZXhwIjoxNjI2MDA3MjU5LCJpYXQiOjE2MjYwMDM2NTksImp0aSI6ImMyMDlhYWQyLTYxZDEtNDgwNy1hMmFlLWJjODNhNmJiY2I2NyIsImNsaWVudF9pZCI6IjQwMGQxMWFsOXAyNXBtamczaWdmbG81aDRpIiwidXNlcm5hbWUiOiJjNTdjNWZiOS03ZmJkLTQwNmQtOWJiMy0xOWY4OGNmYzlhYzAifQ.IuD31XgD2C93v0H9PUCCIWCBYUVX7QdSq3S3MQlR0LhWI7Ai-w5yEI7KVT0Zy6U943sqsvoagU3ykNS756ivfJbPzI1un4L6d8Y4bZBR9ZY4QRylLx7s3sDY4gnNd02KFGcwYQxj_MGm3SDXQF9xFyDJ6_gP7j1XXgIWknhvrN1-QzEaNqGDiY5v97jT1pLLGvlAe5VuUkcbptMdhStM10fAWww-xMVO_2W3mZcCJwUQ5NR6CaGo2HP_EwkkZ5_Qy7w8q3yTzdrNlX67e6TCHdU5Yu2p-bceSU3_2X3IeHfUhAOobhR_BuntsoSdXLJgnJcCYYg2BiHcUY2jIf8h6A"
     },
     "ChallengeParameters": {}
 }
```

![Center](/images/post/create-cognito-user-pool-by-aws-cli/jwt_decode_example.png#center)

## 總結

AWS Cognito 真的很方便，對於還沒有自己的用戶管理App，可以快速的部署。內部還有許多功能例如 Host Login UI 、客製化 Lambda trigger 或者使用第三方認證。艾倫之後有空的話，也會寫使用第三方認證如 Facebook 或 Gmail，或者建立 Serverless Application (API Gateway 驗證）相關文章～


