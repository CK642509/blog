---
title: "[Day 08] API 的 Response"
date: 2023-09-21 23:47:53
categories: 鐵人賽
tags: ithome Python FastAPI
---
今天讓我們來聊聊 API 的 Response。

前面我們都是簡單地回傳一個 dictionary 或是一個字串，但其實也可以傳 list，或甚至是 pydantic model 這類的物件，使用起來非常方便。畢竟，在大部分框架，是沒辦法直接回傳一個物件的。

因此，今天我們會介紹
1. 如何回傳 pydantic model
2. Response 種類
3. 案例分享
<!-- more -->

## 回傳 pydantic model

這邊直接看範例
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    age: int


@app.post("/user")
async def create_user(user: User) -> User:
    return user

@app.get("/users")
async def read_users() -> list[User]:
    return [
        User(name="John Doe", age=18),
        User(name="Ithome Ironman", age=22),
    ]
```

這邊我們在 return 的地方直接寫 `User` 物件，就可以回傳了。

接著再用 Postman 測試一下
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F8_postman_1.PNG?alt=media&token=4c41f525-5731-4579-b51a-e14ad6fdaab0)

可以發現 Pydantic model 順利被轉成 JSON 回傳了

### 過濾資訊
在 API 的 function 加註回傳的 pydantic model，還有一個重要功能，那就是「過濾資訊」

讓我們來看看下方這個例子
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    age: int

class UserDetail(BaseModel):
    name: str
    age: int
    email: str
    phone: str

fake_db = [UserDetail(name="John Doe", age=18, email="johndoe@gmail.com", phone="0911555666"),
           UserDetail(name="Ithome Ironman", age=22, email="ithome@gmail.com", phone="0911777888")]

@app.get("/users")
async def read_users() -> list[User]:
    return fake_db

@app.get("/admin/users")
async def read_users() -> list[UserDetail]:
    return fake_db
```

這邊我們建立了兩個 model，分別是部份的 user 資訊，以及完整的 user 資訊。兩個 API 除了路由有差異之外，只剩下加註的回傳 model 有差別，讓我們來看看 Postman 測試結果。

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F8_postman_2.PNG?alt=media&token=e45ec6aa-10e4-4971-81d6-625acc71b536)
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F8_postman_3.PNG?alt=media&token=3fe83d4a-ba9d-4278-bc01-795200750d0d)

可以發現，我們只要調整 model，就可以輕鬆過濾資訊，這對於同樣一筆資料庫的資料對不同層級的使用者要回傳不同的資訊時，是十分有用的。

### 另一種寫法
這邊額外提一下，程式碼還有另一種寫法

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    age: int

class UserDetail(User):
    email: str
    phone: str

fake_db = [UserDetail(name="John Doe", age=18, email="johndoe@gmail.com", phone="0911555666"),
           UserDetail(name="Ithome Ironman", age=22, email="ithome@gmail.com", phone="0911777888")]

@app.get("/users", response_model=list[User])
async def read_users():
    return fake_db

@app.get("/admin/users", response_model=list[UserDetail])
async def read_users():
    return fake_db
```

主要想提的地方有兩個：
1. pydantic model 那邊可以用繼承的(就像 class 一樣)，寫起來比較簡單
2. 回傳的 model 也可以改成寫在 `response_model` 內

## Response 種類
除了之前和上面介紹的這些 response 種類，FastAPI 也有支援其他類型的 response，以下將逐一介紹 FastAPI 有支援的 Response 種類。

### `PlainTextResponse`
沒什麼好介紹的，就是單純字串 XD

```python
from fastapi import FastAPI
from fastapi.responses import PlainTextResponse

app = FastAPI()

@app.get("/", response_class=PlainTextResponse)
async def main():
    return "Hello World"
```

### `JSONResponse` (以及 `ORJSONResponse`、`UJSONResponse`)
預設的格式，也可以用 [orjson](https://github.com/ijl/orjson) 和 [ultrajson](https://github.com/ultrajson/ultrajson)，但後面者兩個我沒使用過。

### `FileResponse`
當要傳遞檔案到前端時 (例如：下載)，可以考慮使用 `FileResponse`

```python
from fastapi import FastAPI
from fastapi.responses import FileResponse

some_file_path = "large-video-file.mp4"
app = FastAPI()

@app.get("/")
async def main():
    return FileResponse(some_file_path)
```

需要注意的是，這邊的 `FileResponse` 參數只能是「檔案路徑」，不能是 file-like object (例如：用 `open` 得到的物件)，如過要回傳 file-like object，則需要使用 `StreamingResponse`

### `StreamingResponse`
如果要使用 `StreamingResponse`，需要使用 iterator (例如下方範例的有使用到 `yield` 的函數)

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()


async def fake_video_streamer():
    for i in range(10):
        yield b"some fake video bytes"


@app.get("/")
async def main():
    return StreamingResponse(fake_video_streamer())
```

### `RedirectResponse`
某些情況下，或許需要直接幫使用者跳轉到其他網址，此時就可以用 `RedirectResponse`

```python
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/typer")
async def redirect_typer():
    return RedirectResponse("https://ithelp.ithome.com.tw/2023ironman/event")
```

### `HTMLResponse`
FastaPI 也可以回傳一個網頁

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/hello")
async def hello():
    data = "<h1>Hello</h1>"
    return HTMLResponse(content=data)
```

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F8_web_1.PNG?alt=media&token=2b9a0a2f-4b94-4629-b545-0d63d07672d0)

接下來，只要再進一步添加 CSS、JavaScript，就可以有一個漂亮的前端畫面了！
> 這部份讓我們留到明天繼續介紹...

## 案例分享
前端送 csv，後端處理好之後回傳 csv，並自動下載
用 FileResponse 比較好串接，(也可能是我前端功力不夠QQ)，但這樣一定要存檔
因此改用 StreamingResponse

這是一開始寫的版本
```python
from fastapi import FastAPI, Form 
from fastapi.responses import StreamingResponse
import pandas as pd
import io

app = FastAPI()

@app.post("/csv")
async def read_users(file=Form()):
    file_buffer = await file.read()
    file_object = io.BytesIO(file_buffer)
    df = pd.read_csv(file_object)
    
    # do something
    
    new_file_buffer = df.to_csv(index=False)

    return new_file_buffer
```

前端得到的回應會是 `name,age\r\nJohn,18\r\nIthome,22\r\n`
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F8_postman_4.PNG?alt=media&token=17c46242-c676-4277-ac77-8bc62a4c2d92)

前端要用也是可以用，但不是很方便。因此後來把程式碼下半部改成這樣
```python
@app.post("/csv")
async def read_users(file=Form()):
    file_buffer = await file.read()
    file_object = io.BytesIO(file_buffer)
    df = pd.read_csv(file_object)
    
    # do something
    
    new_file_buffer = df.to_csv(index=False)

    def iterator(buffer):
        yield buffer

    return StreamingResponse(iterator(new_file_buffer))
```

此時前端得到的回應就變成這樣
```shell
name,age
John,18
Ithome,22
```

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F8_postman_5.PNG?alt=media&token=5faab164-3540-4815-9ae0-6b32e60230a9)

讓我在前端做資料呈現時方便很多~

## 重點回顧
今天時間有點緊迫，能討論的不多，但還是努力地介紹了
1. 如何回傳 pydantic model
2. Response 種類
3. 案例分享

明天會繼續延伸介紹 `HTMLResponse`，讓 FastAPI 回傳一個漂亮網頁~
