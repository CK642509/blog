---
title: "[Day 11] Middleware 與 CORS"
date: 2023-09-25 23:26:32
categories: 鐵人賽
tags: ithome Python FastAPI
---
今天來聊聊 middleware，一個非必要，但十分好用的設定。
<!-- more -->

## 什麼是 Middleware？
我們直接看這張網路上找到的簡單示意圖
![](https://i0.wp.com/hoclaravel.net/wp-content/uploads/2021/01/Tim-hieu-ve-Middleware-trong-Laravel.jpeg?w=679&ssl=1)

簡單來說，就是在 FastAPI 的基本流程
1. 收到請求
2. API 處理
3. 回傳回應

的中間 (也就是 API 處理前後)，多經過了一層處理。

## 為什麼需要 Middleware？
其實 middleware 不是必要的 (這也是為什麼這個主題可以這麼後面才介紹)，但是有它的話，在許多應用情境上會很方便，例如：紀錄日誌、錯誤處理等。
> 這兩個主題後續都會介紹到

## FastAPI 要怎麼實做 Middleware
我們先來看一下這個[官網](https://fastapi.tiangolo.com/tutorial/middleware/)給的範例，相信會更好懂 middleware 負責的階段。
```python
# main.py
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response

@app.get("/hello")
def read_main():
    return {"message": "Hello World from main app"}
```
這邊使用的是最基本的 http middleware，所有的 request 都會經過這個 middleware。程式碼中的 `call_next(request)` 對應的是 API 的部份，等 API 要做的事情處理完畢後，它會傳的 response 就會回到這個 http middleware。因此，我們只要在 `call_next(request)` 前後紀錄一下時間，就可以知道每個 API 實際上所花費的時間了。除此之外，也可以在這層去客製化調整 header，讓 API 的花費時間回傳到前端。

> middleware 是可以有很多層的，如果要繼續添加，可以繼續使用 `@app.middleware()`，或是 `@app.add_middleware()`

## Middleware 的其他寫法
有趣的是，雖然上面的 middleware 寫法是 [FastAPI 官網](https://fastapi.tiangolo.com/tutorial/middleware/#before-and-after-the-response)提供的，也可以正確執行，但如果大家真的有去嘗試上面那段官網給的範例 code，或許就會注意到 `@app.middleware("http")` 這個寫法是「不鼓勵的 (discouraged) 寫法」...
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F11_vscode_1.PNG?alt=media&token=1f384247-6063-4fab-a784-c7e17612452a)

> 我查不到建議寫法修改的確切時間，只能從這個 [Issue](https://github.com/encode/starlette/issues/1481) 得知至少是 2022 年 2 月以前就改了

FastAPI 的 Middleware 是由 Starlette 建立起來的，讓我們來看看 Starlette 的[文件](https://www.starlette.io/middleware/)的範例
```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.base import BaseHTTPMiddleware

class CustomHeaderMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        response.headers['Custom'] = 'Example'
        return response

routes = ...

middleware = [
    Middleware(CustomHeaderMiddleware)
]

app = Starlette(routes=routes, middleware=middleware)
```

簡單來說，就是在建立 `app` 時，把 middlware 以 `list` 的方式給設定好，這也方便我們管理多個 middleware 時的觸發順序。

所以，如果要做到跟上面 FastAPI 一樣功能，較鼓勵的寫法是
```python
# main.py
import time
from fastapi import FastAPI, Request
from fastapi.middleware import Middleware
from starlette.middleware.base import BaseHTTPMiddleware

class CustomHeaderMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        start_time = time.time()
        response = await call_next(request)
        process_time = time.time() - start_time
        response.headers["X-Process-Time"] = str(process_time)
        return response
    
middleware = [
    Middleware(CustomHeaderMiddleware)
]

app = FastAPI(middleware=middleware)

@app.get("/hello")
def read_main():
    return {"message": "Hello World from main app"}
```

## CORS
Midlleware 除了上面提到的用途，還有一個常見的用途，那就是為了解決 CORS 所引發的錯誤。
在開始之前，我們先快速看一下發生錯誤的樣子。

首先，在前端向 `http://localhost:8000/hello` 發送請求，並把結果印出來。
> 這邊我是用 Vue 的開發模式，以下只擷取部份程式碼
```javascript
<script setup lang="ts">
import axios from 'axios';

const request = async () => {
  const res = await axios.get("http://127.0.0.1:8000/hello")
  console.log(res)
}

request()
</script>
```

此時，打開開發者工具，就會看到這個錯誤訊息
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F11_cors_error.PNG?alt=media&token=c947df0e-1fa3-4509-bbd5-5f3f9fec5cf8)

但是看後端的 terminal 會發現，其實是有正常送出回應的 (用 Postman 測試也沒問題)
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F11_cors_error_terminal.PNG?alt=media&token=00f541b4-7a4e-4e29-9e5c-36efdb8cb0e2)

顯然，問題就出在前端上。事實上，這是瀏覽器的保護機制，它故意屏蔽了這個請求的回應，目的就是為了遵守[同源政策 (same-origin policy)](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)，因此禁止 CORS (Cross-Origin Resource Sharing)。

## 什麼是同源政策？什麼是 CORS？
簡單來說，就是希望網站的互動都是同一個來源 (網域)，如果不同，那麼可以進行的互動是有限的，例如：網站不能讀取另一個來源的 XMLHttpRequest。藉由同源政策，來減少惡意程式在我們不知情的情況下，偷偷拿到我們資料的機會

> 當然，光靠一個同源政策是不夠的，駭客還是可以透過 XSS (Cross-site scripting) 進行攻擊，但這又是另一個故事了...

> 可以參考這篇[文章](https://medium.com/starbugs/%E5%BC%84%E6%87%82%E5%90%8C%E6%BA%90%E6%94%BF%E7%AD%96-same-origin-policy-%E8%88%87%E8%B7%A8%E7%B6%B2%E5%9F%9F-cors-e2e5c1a53a19)，我覺得它介紹得很好 XD


## 該怎麼解決 CORS 的問題？

我們可以直接使用 FastAPI 的 `CORSMiddleware`，具體的實作方法可以看這個範例
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://127.0.0.1:5173",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def main():
    return {"message": "Hello World"}
```

我們要先知道前端網站的位置 (包含：Host、Port)，並且以 list 的方式放入 `allow_origins` 中，再用 `CORSMiddleware` 幫我們將相關設定給處理好。

> 偷懶的話，也可以直接把 `allow_origins` 設成 `["*"]`，但這樣做其實仍會有一些限制，因此還是建議好好設定網址，詳細可以看[官網說明](https://fastapi.tiangolo.com/tutorial/cors/#wildcards)

接下來我們再次從前端發送請求
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F11_cors_success.PNG?alt=media&token=ec9f367d-f6f7-4532-b9ae-6a9daf77afdc)

這次就順利拿到回應了~

## 重點回顧

今天我們介紹了 
1. Middleware
2. Middleware 的使用方法
3. CORS
4. 如何用 Middleware 解決 CORS 的問題

> CORS 則通常是在前端串接 API 時才會遇到，但因為要解決問題還是要靠後端設定，因此就還是納入這次的介紹範圍了。

在後面的文章，我們會再介紹 Middleware 的應用，相信到那時候就會對 middleware 有更深的了解，並且了解到它的價值。

明天開始，我們會介紹比較應用的主題，預計包含：
1. 登入驗證
2. 資料庫
3. 錯誤處理
4. 日誌系統
5. 測試、部屬、CI/CD

敬請期待~
