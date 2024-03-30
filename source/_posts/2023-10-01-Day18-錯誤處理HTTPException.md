---
title: "[Day 18] 錯誤處理 (一)：HTTPException"
date: 2023-10-01 23:56:20
categories: 鐵人賽
tags: ithome Python FastAPI
---
接下來這幾天來聊聊錯誤處理

在我們開發程式的時候，難免會遇到各種千奇百怪的狀況，很難保證程式一定可以照我們所預期地被執行，有時候是我們開發的問題，有時候則是使用者的神奇輸入導致，因此，如何做好錯誤處理一直以來都是一個重要的項目。
<!-- more -->

## 可預期 和 不可預期
發生的錯誤，可以簡單分成兩種，一種是可預期的錯誤，另一種是不可預期的錯誤。

可預期的錯誤十分常見，例如：註冊時要求密碼長度 8 位以上，但剛好前端沒有正確阻擋到，就送到後端了，此時就應該要回傳錯誤訊息給前端，請使用者重新提供符合要求的密碼。

不可預期的錯誤則往往是因為開發時的程式邏輯不完整 (或不正確)，導致最終程式執行出錯，最後只能先簡單回傳前端說發生錯誤 (但不清楚確切原因)，之後開發者再根據錯誤訊息去修正程式的錯誤。

## 在 FastAPI 怎麼處理錯誤？

讓我們先來看看最基本的錯誤，這邊先簡單的寫一支 API

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def hello():
    print(a)
    return {"message": "hello"}
```

接著執行主程式

```shell
uvicorn main:app
```

你會發現，儘管我們在 API 中要求印出未定義的變數 `a`，但因為啟動 FastAPI 後端時不會用到這段程式碼，所以後端是可以順利執行起來的。

接下來用 Postman 進行測試，果然發生錯誤

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F18_postman_1.PNG?alt=media&token=fe463ff3-82fe-4a3b-99d4-80621439d4b8)

緊接著，我們看一下 Terminal，就會看到一串長長的錯誤訊息。

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F18_vscode_1.PNG?alt=media&token=5a7b06d7-4945-4275-9682-63849f516b12)

相信有一定程式基礎的朋友，很快就可以找到關鍵的訊息在最後面幾行

```shell
File "C:\Users\CK642509\Documents\GitHub\ithome_ironman_2023\Day18\main.py", line 7, in hello
    print(a)
NameError: name 'a' is not defined
```

上面寫說我們在 `main.py` 的第 7 行的 `print(a)` 發生了 `NameError`，相關說明則是 `name 'a' is not defined`。

到目前為止，這一切都很好理解，我們故意添加了錯誤的程式碼，最終也導致呼叫 API 時發生程式的錯誤，而這個錯誤也可以在 Terminal 看見。

這邊就牽涉到兩個重點
1. 當發生不可預期錯誤時，FastAPI 預設會回傳 500 的錯誤
2. Terminal 可以看到的日誌 (log) 內容有兩條，分別是
   ```shell
   INFO:     127.0.0.1:5701 - "GET / HTTP/1.1" 500 Internal Server Error
   ```
   以及上面圖片中的那一長串錯誤訊息，日誌等級分別是 Info 和 Error

> 更精確一點的說法是，Response 的 HTTP status code 是 500，代表的是 Internal server error

回傳 500 錯誤屬於這次我們要討論的範圍，日誌系統則會留到下一個大主題。

## 預設的錯誤處理
在上面的例子中，我們沒有添加任何的錯誤處理，後端會回傳 500 錯誤完全是 FastAPI 自動處理的。而這個自動處理的背後原因，是 Starlette 預設會有兩個 middleware，包含：
1. `ServerErrorMiddleware`：讓我們在發生錯誤時回傳 500，同時也永遠是最外層的 middleware 
2. `ExceptionMiddleware`：負責針對特定錯誤做對應處理，也就是 exception handler

> 詳細可以去看 [Starlette 文件](https://www.starlette.io/middleware/)

> Middleware 在 [[Day 11]](https://ithelp.ithome.com.tw/articles/10326779) 有介紹過

## 自己進行錯誤處理
雖然有預設的錯誤處理，但所有錯誤都會被導到 500，因此理想上錯誤處理這件事還是要由我們自己掌控比較好，讓我們可以在發生錯誤時更快找到問題所在。

最簡單的做法就是，針對可能發生的錯誤，回傳對應的錯誤訊息，同時，也要去調整 Response 的 HTTP status code，方便讓前端更容易知道有錯誤。

> 正常的 Response 的 HTTP status code 是 200

FastAPI 有一個好用的東西 ── `HttpException`，在前面的文章中也有出現幾次，但沒有機會好好介紹，它可以讓我們很容易地去調整 status code，並且有一個 `detail` 欄位 (可以不填) 讓我們可以放錯誤訊息，之後就會以一個預設模板將錯誤訊息送到前端。

讓我們稍微修改一下上面的程式
```python
# main.py
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

@app.get("/")
async def hello():
    try:
        print(a)
    except:
        raise HTTPException(status.HTTP_501_NOT_IMPLEMENTED, detail="Test exception")
    return {"message": "hello"}
```

這邊我們把 status code 設成 501 進行測試，實際開發時則要去查一下應該使用哪個 status code 比較合適，FastAPI 官網上也有簡單的[介紹](https://fastapi.tiangolo.com/tutorial/response-status-code/?h=status#about-http-status-codes)。

如果熟悉 status code 的話，也可以直接輸入數字，也就是直接這樣寫
```python
raise HTTPException(501, detail="Test exception")
```

用 status 的好處是可以讓開發工具 (例如：VS Code) 去補全程式碼時，順便查看各個 status code 對應的說明，這部份就看大家習慣了。

設定好 `main.py` 並重啟程式後，接著一樣用 Postman 進行測試
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F18_postman_2.PNG?alt=media&token=bf30cb87-3941-4a8d-8006-7163280ee2a1)

可以發現，status code 被順利改成 501，錯誤訊息也改成我們自訂的內容了。而且 terminal 的 log 也乾淨許多，只有一筆訊息，如下圖。

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F18_vscode_2.PNG?alt=media&token=662807e4-7912-42d1-bf99-3bb1c9179809)


## 那非預期錯誤呢？
上面提到的做法比較適用於可預期的錯誤，如果要處理非預期的錯誤，但又不想用預設的樣子 (回傳 500，沒有錯誤訊息)，粗暴的一點的做法就是再多包一層 `try` `except`，但如果要每個 API 都這樣做其實也是很麻煩，因此 middleware 就再度登場了。

```python
# main.py
from fastapi import FastAPI, HTTPException, status, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    try:
        response = await call_next(request)
        return response
    except:
        return JSONResponse(
            status_code=501,
            content={"detail": "Test exception"},
        )

@app.get("/")
async def hello():
    print(a)
    return {"message": "hello"}
```

一樣用 Postman 測試
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F18_postman_3.PNG?alt=media&token=b385ebbb-4632-4a73-ac44-d6fdc7e4a693)

可以發現效果跟上面是一樣的。

再來看 terminal
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F18_vscode_3.PNG?alt=media&token=c476c482-f4d7-4fb4-89b9-a44b5c479224)

效果一樣跟上面相同。

> 需要注意的是，這時候的 log 就不會記錄到我們自定義的 `detail`，這背後原因牽涉到 log 產生的時機點，就留到討論 log 的時候再仔細說明。

## 重點回顧

今天我們介紹了錯誤處理的方法了，明天會繼續介紹如何客製化我們的 `HTTPException`
