---
title: "[Day 12] 登入驗證 (一)：Basic Auth"
date: 2023-09-25 23:26:32
categories: 鐵人賽
tags: ithome Python FastAPI
---
在大多數情況，後端 API 並不會隨便對外開放，需要有足夠的權限才可以訪問，有的只要一般會員就好，有的則是要管理員層級才可以。而辨識身分的方法，就是我們接下來要討論的主題。
<!-- more -->

## 一般操作流程
讓我們來快速回想一下平常是怎麼登入的？

最常見的流程是：
1. 進入登入畫面
2. 輸入帳號、密碼
3. 送出

接下來的一段時間內，就可以自由地使用會員功能，直到我們登出，或是系統設定的登入時間結束
> 時間長短往往取決於對安全的要求，尤其是跟金錢有關的服務，例如：網路銀行

## 那為什麼後端可以知道我們是會員呢？
這是因為最一開始我們有先註冊過，後端資料庫內有儲存我們的帳號與密碼的相關資訊，因此，只要比對資料庫，就知道這次的登入是否成功。

> 注意！資料庫是不能直接儲存密碼的，它需要透過一些轉換方式 (後面會介紹) 轉換過後，才能儲存進資料庫。這麼做是為了避免資料外洩時，用戶的密碼直接赤裸裸地出現在大家面前。

這邊來看一下極簡單的例子
```python
# main.py
from fastapi import FastAPI, Form

app = FastAPI()

fake_user_db = [
    {
        "username": "ithome",
        "password": "secret"
    }, {
        "username": "ironman",
        "password": "password"
    }
]

@app.post("/login")
async def login(username: str = Form(), password: str = Form()):
    is_user = False
    for user in fake_user_db:
        if username == user["username"] and password == user["password"]:
            is_user = True
            break

    return {"is_user": is_user}
```

輸入錯誤的帳密，後端就會告訴你失敗
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_postman_1.PNG?alt=media&token=8dab51ff-cc77-412f-970c-811fd8916f17)

輸入正確的才會回傳成功
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_postman_2.PNG?alt=media&token=17e14722-3c4f-469a-b386-198b6989d395)

> 現在這個範例可以說是漏洞百出，只是為了快速展示概念用的，請大家不要真的拿去實作

再往下思考一下，如果每次都要在 Request body 攜帶帳號密碼，實在很不方便，同時也限制了 HTTP method。因此，後來討論出來的公認做法，就是在 Request header 夾帶身分認證的資訊。

> 會使用「身分認證的資訊」這個詞的原因是，並不是所有方法都是直接使用帳號、密碼

## 怎麼在 Header 夾帶身分認證資訊？
既然是公認的作法，自然也有標準的格式，也就是
```shell
Authorization: <type> <credentials>
```
其中 `type` 是指認證方式，而 `credentials` 則是該認證格式的身分資訊，這邊介紹一下兩種常見的認證格式

### Basic Authentication
是最基本的格式，作法就是將帳號與密碼用 `:` 接起來後，用 Base64 編碼 (encode) 轉換，最後在 Header 帶入
```shell
Authorization: Basic <credentials>
```

舉個例子：
假設帳號是 `ithome`，密碼是 `secret`，那接起來就是 `ithome:secret`，再經過 Base64 encode 得到 `aXRob21lOnNlY3JldA==`，因此在 Header 帶入
```shell
Authorization: Basic aXRob21lOnNlY3JldA==
```

後端只要再 Base64 編碼轉換回去就可以得到帳號、密碼了

> 有很多免費工具可以幫我們做 Base64 轉換，例如 [這個網站](https://www.base64encode.org/)

### JSON Web Tokens (JWT)

今天先看一下範例就好，細節明天再介紹~ XD
```shell
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.tImCzvIkqaNmGB5mMAG1DZRnZO56sjoYO5nU2YUdRK4
```

## 如何在 FastAPI 實作 Basic Authentication

知道概念後，其實要土法煉鋼自己寫出 Basic Authentication (以下簡稱 Basic Auth) 也不難，但重複造輪子的事還是能避免就避免，時間可是很寶貴的。

```python
# main.py
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app = FastAPI()

security = HTTPBasic()

@app.get("/users/me")
def read_current_user(credentials: Annotated[HTTPBasicCredentials, Depends(security)]):
    return {"username": credentials.username, "password": credentials.password}
```

> 注意這個範例只有最基本的 Basic Auth，也就是說，它只要求使用者提供帳號、密碼，並沒有做驗證

接著使用 Postman 進行測試。

Postman 有一個很貼心的功能，它把登入驗證的部份獨立到 Auth 頁籤下，我們只要選擇好對應的 type，並輸入對應的資訊，它就會幫我們把它做對應的轉換並放入 Header，不需要再自己去做 Base64 encode。

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_postman_3.PNG?alt=media&token=893ad6a8-db83-4775-a544-8a5d45a1714f)

之後在 `Username` 和 `Password` 輸入帳號、密碼，並送出。

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_postman_4.PNG?alt=media&token=b3e35a73-4ed0-43ea-bafc-62f37eab01f9)

就會收到後端回傳的帳號、密碼，代表 FastAPI 真的有幫我們解析出帳號與密碼。

如果沒有在 Header 放登入資訊，則會收到 401 錯誤。

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_postman_5.PNG?alt=media&token=2ceff9a1-82f8-4eed-8c9b-58149bd2f692)

> 如同上方所說，這個範例是沒有驗證帳號、密碼的，官網有更完整的[範例](https://fastapi.tiangolo.com/advanced/security/http-basic-auth/#check-the-username)，大家有興趣的話可以去看看~

## Swagger UI

接下來讓我們看看 API 文件的變化，此時文件右側有一個 Authorize 按鈕。
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_swagger_1.PNG?alt=media&token=f69d0f06-5bc6-408e-a196-141f207a8a6a)

點擊之後就會看到一個彈出視窗，要求輸入帳號、密碼
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_swagger_2.PNG?alt=media&token=f4d9ad67-a1ee-435a-b893-66dea6a03e8d)

接著我們隨便輸入一組帳號、密碼，並按下 Autherize 按鈕
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_swagger_3.PNG?alt=media&token=dddfe9d9-66f6-47c3-9507-405b55c4280a)

這樣我們就進入登入狀態了，後續在這份文件上測試 API 都會自動帶有 Basic Auth 的 Header
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_swagger_4.PNG?alt=media&token=3eff2536-8fc7-45c0-87d2-c5beec0bc9da)

回到 API 文件就會發現右邊的鎖的 icon 發生變化了~
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_swagger_5.PNG?alt=media&token=e10cd992-4652-4097-86f5-0c89a7f8b669)

這是測試剛剛的 API 的結果
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_swagger_7.PNG?alt=media&token=134e0833-de45-4aa1-9dc4-613f85f7a289)

但如果不按右上的 Autherize 按鈕，就直接去測試 API，它也會出現彈出視窗要求輸入帳號、密碼
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F12_swagger_6.PNG?alt=media&token=1d0376dc-21d8-4529-b8e5-0208020443e5)

## 重點回顧

今天我們主要介紹了 Basic Auth 的概念與做法，明天會繼續介紹 JWT 的那一長串的身分資訊是怎麼來的？以及要怎麼在 FastAPI 實作 JWT 的登入驗證？ 

