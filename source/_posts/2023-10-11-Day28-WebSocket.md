---
title: "[Day 28] WebSocket 的實作與測試"
date: 2023-10-11 23:40:54
categories: 鐵人賽
tags: ithome Python FastAPI
---
昨天我們在背景任務提到了「如何讓前端知道背景任務完成」這個議題，有提到可以使用 WebSocket，今天就讓我們來看看該怎麼做。
<!-- more -->

## WebSocket 極簡化懶人包

WebSocket 是一種不同於 HTTP 的通訊協定，在建立 WebSocket 連線後，雙方都可以主動向另一方傳送訊息

> 理論上應該要放更完整的介紹的，但我覺得我怎麼講都還是跟網路上的文章差不多，因此這部份我跳過，請大家自己去查 XD

## 實作 WebSocket

首先，要安裝 `websockets`
```shell
pip install websockets
```

這邊直接來看官網的[範例](https://fastapi.tiangolo.com/advanced/websockets/#websockets-client)

```python
# main.py
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>Chat</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>Send</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
            var ws = new WebSocket("ws://localhost:8000/ws");
            ws.onmessage = function(event) {
                var messages = document.getElementById('messages')
                var message = document.createElement('li')
                var content = document.createTextNode(event.data)
                message.appendChild(content)
                messages.appendChild(message)
            };
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""


@app.get("/")
async def get():
    return HTMLResponse(html)


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

首先，為了要呈現前後端的溝通，這邊建立了一個簡單的前端 (聊天室)，就在 `http://127.0.0.1:8000`
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_browser_1.PNG?alt=media&token=9daf1a3c-a72d-4e66-aea5-2a8e65eb5d29&_gl=1*whqv7z*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzM2MTAuNDYuMC4w)

進到畫面後，前端就會建立 WebSocket 連線
```js
var ws = new WebSocket("ws://localhost:8000/ws");
```

並且如果後端透過 WebSocket 傳訊息的話，就會觸發 `ws.onmessage`，將訊息呈現在輸入框下方。
> 只是目前後端的設定是收到前端訊息才會傳


同時，也可以在 terminal 看到訊息
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_terminal_1.PNG?alt=media&token=e7ce20f0-f13f-405c-9ef1-576555aa5ea8&_gl=1*1bszrbd*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzM3NDUuNjAuMC4w)

接著，我們可以嘗試在輸入框內輸入並送出訊息
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_browser_2.PNG?alt=media&token=2ef5cadf-42a3-450a-9012-1b204e455528&_gl=1*uc6alv*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzM4NzEuNTguMC4w)

送出之後，前端就會收到後端透過 WebSocket 傳來的訊息，並呈現在下方
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_browser_3.PNG?alt=media&token=d785b52c-25eb-42cf-89e7-5c2cdc3f4a08&_gl=1*1irhack*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzM5MDAuMjkuMC4w)

## 能不能用 Postman 測試？
如果對前端或 JavaScript 不太熟，也可以改用強大的 Postman 測試 (在 [[Day 05] 後端開發的得力助手：Postman](https://ithelp.ithome.com.tw/articles/10321833) 有介紹過，但沒有介紹到 WebSocket)。

> 這個功能是 2021 年 5 月才推出的 ([公告](https://blog.postman.com/postman-supports-websocket-apis/))

使用 WebSocket 連線的做法比較不同，首先，點擊左邊的 New 按鈕，並選擇 WebSocket

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_postman_1.PNG?alt=media&token=a7eb68dd-d7bc-4832-88cf-4265a94e2ec6&_gl=1*z7ss8e*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzQ1MzYuNDQuMC4w)

接著在 URL 的地方輸入 `ws://127.0.0.1:8000/ws`，並按下右邊的 Connect 按鈕
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_postman_2.PNG?alt=media&token=5c562ce9-2de4-4509-8bd1-9be4d3792bcb&_gl=1*7w3qsm*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzQ3MTMuNjAuMC4w)

連線成功之後，就會在下方看到提示
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_postman_3.PNG?alt=media&token=d919950a-37c9-4f67-870d-1dd750bcc064&_gl=1*1bjlyhs*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzQ3ODQuNjAuMC4w)

接著在 Message 頁籤內輸入訊息，並按下右邊的 Send 按鈕
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_postman_4.PNG?alt=media&token=ebd5c2e9-d1d0-496b-b7a2-19a267a5f37f&_gl=1*1oyn6wh*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzQ4MTMuMzEuMC4w)

最後就會在下方看到提示，顯示我們收到了 `Message text was: Hello Ithome`
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F28_postman_5.PNG?alt=media&token=14076986-36f1-4061-86a7-fa06524be577&_gl=1*x48lrv*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NzAzMzM3Mi4zNC4xLjE2OTcwMzQ4NjUuNjAuMC4w)

## 多個聊天室該怎麼做？
大家如果仔細觀察一下，就會發現，假如同時有多個頁面開啟，大家都會進到同一個 WebSocket，以聊天室的概念來說，就是只有一個聊天室，所有人都在裡面。因此，如果想要有多個聊天室，我們可以加入動態路由的概念，在 WebSocket 的 route 後面加上一個聊天室 ID，接著讓前端每次都隨機產生一個 ID，並在連線時把 URL 加上這個隨機的 ID，就可以錯開了 (就像是有多個聊天室頻道一樣)。

實作的程式碼可以參考這個[範例](https://fastapi.tiangolo.com/advanced/websockets/#handling-disconnections-and-multiple-clients)。
(太長就不貼了)

另外，上面的範例也有使用一個全域的物件，用來管理所有 WebSocket 連線，類似記錄當下到底有哪些聊天室 ID，方便後續管理 (例如：一次性關閉所有頻道或同時對所有頻道推波訊息)。這個做法很實用，建議大家實作時可以使用這個方法。

## 能不能把 WebSocket 連線移到其他檔案？

在 [[Day 06] API 管理與 API 文件](https://ithelp.ithome.com.tw/articles/10322704) 的文章中，我們提到了可以使用 `app.include_router()` 的方式，將各個路由移到其他檔案以方便管理。因此，理想上，這個 WebSocket 連線應該也要比照辦理。

然而，實作之後發現沒那麼簡單，因為 WebSocket 不能用 `include_router()` 的方式，最後查了一下資料，只有看到兩個比較接近的做法。

> 也有[issue](https://github.com/tiangolo/fastapi/issues/98)在討論這個問題

### `.add_websocket_route()`

直接看範例
```python
# main.py
from fastapi import FastAPI
from ws_api import websocket_endpoint

app = FastAPI()

app.add_websocket_route("/ws", chat_socket)
```

```python
# ws_api.py
from fastapi import APIRouter, WebSocket

async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```
這邊的做法是把 function 搬到另一個檔案，而不是用 router，因此如果有多個 WebSocket 的路由，就要用很多個 `.add_websocket_route()`

### Sub Applications

這邊需要使用到 [Sub Application](https://fastapi.tiangolo.com/advanced/sub-applications/) 的概念，就是把一個 FastAPI app 給 mount 在另一個 FastAPI app 上 (在 [[Day 07] API 文件進階設定](https://ithelp.ithome.com.tw/articles/10323496) 後面有提到過)。

```python
# main.py
from fastapi import FastAPI
from ws_api import ws

app = FastAPI()

app.mount("/ws", ws)
```

```python
# ws_api.py
from fastapi import APIRouter, WebSocket

ws = FastAPI()

@ws.websocket("/")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

需要注意的是，`ws_api.py` 內的路由至少要有 `/`，因此連線的 URL 要改用 `ws://127.0.0.1:8000/ws/` (結尾多了一個 `/`)

## 小結

今天我們介紹了 WebSocket 該怎麼實作和測試，之後大家可以依照自己的需求來決定應該使用平常的 HTTP 還是 WebSocket~

剩下最後一個主題可以寫了，但總覺得還有很多可以寫的，還需要再想一下明天要寫什麼主題...
