---
title: '[Day 02] 第一個 Discord BOT (一)：註冊與取得 Token'
date: 2024-09-14 22:09:51
categories: 鐵人賽
tags: ithome Python Discord
---

今天來開始建立 Discord BOT ~

<!-- more -->

## 進度
![roadmap02](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_roadmap.jpg?alt=media&token=b665a226-e247-47a6-a175-ba48857bdfd0)

今後我會使用顏色來標記目前我們在這張 roadmap 中的哪一個地方，綠色表示當天會提到的主題，灰色則表示已經有介紹過了 (今天才第一天進入主題，沒有灰色 XD)。

今天的內容會包含兩個部分：
1. 註冊 Discord BOT，並把它加入 Discord 伺服器
2. 取得 Discord BOT 的 Token (備用，明天會用到)

## 1. 註冊 Discord BOT

第一階段，當然就是要先註冊一個新的 Discord BOT。

### 1-1 進入 Discord Developers Portal

進入 Discord Developers 網站的 Applications 頁面

網站連結：[https://discord.com/developers/applications](https://discord.com/developers/docs/intro)

![signup_01](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_signup_01.png?alt=media&token=65905a8b-b584-45dc-84dc-0d09b5542702)

> 如果沒登入過的話，需要先登入才能使用
> ![signup_00](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_signup_00.png?alt=media&token=d172cc79-af45-43d0-9557-7c01828a6bf5)

### 1-2 點擊右上角的「New Application」按鈕，開始創建 BOT

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_signup_01-2.png?alt=media&token=7b93d5e0-8a84-4413-9836-0782a726fa8b)

### 1-3 設定 Discord BOT 名稱

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_signup_02.png?alt=media&token=1b5fe040-bf62-487d-bdff-4d1cbc713361)

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_signup_03.png?alt=media&token=3f67d118-eab4-4014-8614-fe637a55dce8)

輸入名稱並同意規定後，點擊右下角的「Create」按鈕

### 1-4 完成

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_signup_04.png?alt=media&token=47d003e7-c197-4972-acb5-a04b4106722d)

### 1-5 追加設定

最後我們還需要把 Privileged Gateway Intents 的三個權限打開

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_signup_06.png?alt=media&token=e3de9bd1-bd12-4579-8b39-b7837701ba02)

修改完記得點擊「Save Changes」按鈕

## 2. 建立 Discord BOT 邀請連結

第二階段，就是要建立邀請連結，這樣大家才能把這個機器人加入 Discord 的伺服器中。

### 2-1 進入 OAuth2 分頁，開始設定權限

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_url_01.png?alt=media&token=01ae02bc-76f8-4b61-9489-1d4d32648ae3)

### 2-2 Scopes 勾選 bot

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_url_02.png?alt=media&token=769ab7a5-8e6f-4713-8ea8-a17b4fb36293)

### 2-3 Permissions 勾選 Administrator

為了方便後續操作，先要求管理者權限XD

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_url_03.png?alt=media&token=0aff2ac2-8efa-409e-9201-fd08a299cff8)

### 2-4 取得 Discord BOT 邀請連結

上面都設定完之後，下方會自動生成邀請連結

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_url_04.png?alt=media&token=b2691ee2-243c-46f7-ab88-5c0045706bee)

## 3. 加入伺服器

第三階段就是透過邀請連結，把 Discord BOT 邀請加入伺服器

### 3-1 點擊邀請連結

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_invite_01.png?alt=media&token=b22a30c6-2c83-4972-88cb-cd399615b227)

### 3-2 選擇伺服器

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_invite_02.png?alt=media&token=d0345b4f-85ad-45db-b844-e5dd5422637a)

完成後，點擊右下角的「繼續」

### 3-3 同意授權

點擊右下角的「授權」

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_invite_03.png?alt=media&token=396c65fd-3c42-4988-8bcb-ea0688a3814b)

### 3-4 新增 Discord BOT 成功

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_invite_04.png?alt=media&token=499b40f6-edcc-4ccf-93f3-fd2367664fe1)

之後就會在 Discord 看到通知

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_invite_05.png?alt=media&token=085d1160-ba95-4485-9217-48c6614c24b9)

同時，大家可以注意到，右上角已經可以看到機器人的身影了 (只是是離線的)

## 4. 取得 Token (備用)

前面三個階段就算是告一個段落了，不過趁現在 Discord Developer 頁面還開著，就順便先取得 Token，明天會用到~

### 4-1 進入 Bot 分頁

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_token_01.png?alt=media&token=804223a8-261a-4a86-9145-9cd767db2ca6)

### 4-2 重設 Token

點擊「Reset Token」

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_token_01-2.png?alt=media&token=bf2b079a-4df3-4cfd-b1b4-b556e3c39427)

之後會跳出確認訊息，點擊「Yes, do it!」按鈕

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_token_02.png?alt=media&token=09c248a3-ae97-42ec-b3e4-033a85e55426)

接著會要求輸入密碼做驗證

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_token_03.png?alt=media&token=95eb8875-061d-4f30-a4b9-380a3a3d6028)

輸入後按下「Submit」按鈕

### 4-3 取得 Token

重設完 Token 之後，Token 就會顯示在下方

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F02_token_05.png?alt=media&token=18068833-5d28-4c6e-a45c-ba710de94ab1)

為了減少 Token 外洩的風險，這個 Token 下次開啟網頁時就再也看不到了 (如果只是切去其他分頁的話還看的到)，記得趕快複製並儲存下來

> 注意！請妥善保存這個 Token，不要忘記也不要隨便公開

## 小結

今天我們順利地註冊好的 Discord BOT 並加到伺服器了，明天會開始使用 Token 對這個 Discord BOT 進行操作~
