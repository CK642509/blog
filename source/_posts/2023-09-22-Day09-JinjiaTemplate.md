---
title: "[Day 09] 做出一個網站：Jinja Template"
date: 2023-09-22 23:28:36
categories: 鐵人賽
tags: ithome Python FastAPI
---
在昨天的文章中，我們提到了 FastAPI 的回應可以是 HTML，今天讓我們來看看要怎麼讓 FastAPI 回傳一個靜態網站
<!-- more -->

## 製作網站
當然，這個網站無法像 API 文件一樣自動生成，因此，第一步就是要要自己製作一個網頁

> 想要做出一個靜態網站，基本上都會需要 HTML、CSS、JavaScript，這邊就不多作介紹了

這邊我們先快速建構一個畫面 (這邊我用以前坊間補習班的作業)

https://gist.github.com/CK642509/8986cb0dd9c5835f43a45ccbe41a6571

然後把這段 HTML 貼到下面程式的 `html` 內
(為了不浪費版面，HTML 就不貼上來了)
```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<html lang="en">
</html>
"""

@app.get("/hello")
async def hello():
    return HTMLResponse(content=html)
```

接下來進到 `http://127.0.0.1:8000/hello`，就可以看到畫面了
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F9_web_1.PNG?alt=media&token=0c9892e7-7e48-41bc-af58-c7e43ab7f964)

如果想把網頁內容拆成 `index.html` 和 `main.css` 分開管理也可以，只要 HTML 內有設好路徑，例如：
```html
<link rel="stylesheet" href="/main.css">
```

再把對應的檔案改用 `FileResponse` 就好
```python
from fastapi import FastAPI
from fastapi.responses import FileResponse

app = FastAPI()

@app.get("/hello")
async def hello():
    return FileResponse("index.html")

@app.get("/main.css")
async def css():
    return FileResponse("main.css")
```

但這樣的做法還是有點太粗暴了，每個 HTML 都要土法煉鋼的慢慢製作，這時，樣板引擎 (template engine) 就登場啦~

此時就可以考慮使用如果遇到需要帶參數到 HTML 內，或是想要更有效率的

## 樣板引擎 Jinja 
樣板引擎主要的功能就是透過樣板來產生 HTML，簡單來說就是用程式來寫 HTML。而基本上，所有樣板引擎 FastAPI [都支援](https://fastapi.tiangolo.com/advanced/templates/?h=tem#templates)，其中，最常見的是 [Jinja](https://jinja.palletsprojects.com/en/3.1.x/)

> 需要多安裝 jinja2，安裝指令是 `pip install jinja2`

在 FastAPI，要正確使用 jinja 的關鍵點是
1. 要 mount HTML 內會使用到的檔案
2. 設定 HTML template 的資料夾路徑
3. 回傳 `TemplateResponse` 時，除了指定要使用的樣板 HTML，其他樣板中會使用到的參數也需要一併帶上

來看看官網給的範例
```python
# main.py
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"))

templates = Jinja2Templates(directory="templates")

@app.get("/items/{id}", response_class=HTMLResponse)
async def read_item(request: Request, id: str):
    return templates.TemplateResponse("item.html", {"request": request, "id": id})
```

`item.html`
```html
<html>
<head>
    <title>Item Details</title>
    <link href="{{ url_for('static', path='/styles.css') }}" rel="stylesheet">
</head>
<body>
    <h1>Item ID: {{ id }}</h1>
</body>
</html>
```

`styles.css`
```css
h1 {
    color: green;
}
```

接下來就可以看到我們定義的樣板的結果了
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F9_web_2.PNG?alt=media&token=46423c88-16dd-4d1d-998e-394fa41303f8)

## 其他作法
如果上面的方法都不喜歡，也可以用現在常見的前端框架 (例如：React、Vue、Angular) 做好畫面後，編譯成靜態檔，再用類似上面的方法加在 FastAPI 後端內。

下面這個例子是我把靜態檔都放在 `dist` 資料夾內

```python
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

app.mount("/assets", StaticFiles(directory="dist/assets"))

templates = Jinja2Templates(directory="dist")

@app.get("/", response_class=HTMLResponse)
async def read_item(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})
```

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F9_web_3.PNG?alt=media&token=02db4406-3618-4939-ae51-1ede015c9815)

但這個做法通常只會是 Demo 用而已，雖然小專案可以透過這個方式把前後端放在一起方便管理，但稍微大一點的專案都會前後端分離。如果有多套一層 proxy server，就會讓 request 直接去拿靜態檔，不必多過一層 FastAPI 後端，一來一回也是多浪費了一些時間。

## 重點回顧
今天主要是介紹如何用 FastAPI 呈現網頁，算是比較不重要的功能，但因為工作上真的有遇過這個需求，因此就還是介紹一下~


終於到週末了，希望可以先寫幾篇文章起來，不然每天晚上都好緊張...

