---
title: "[Day 23] 日誌系統 (四)：在 Middleware 紀錄 Log 吧"
date: 2023-10-06 23:46:51
categories: 鐵人賽
tags: ithome Python FastAPI
---
今天來個大整合，把之前的錯誤處理也整合進來。

<!-- more -->

之前在 [[Day 19]](https://ithelp.ithome.com.tw/articles/10333168) 有提到，為了讓程式碼更簡潔，同時又要針對「可預期錯誤」和「非預期錯誤錯誤」進行錯誤處理，我們統一在 middleware 才紀錄 log，同時，為了讓 log 可以記錄到詳細的錯誤訊息，但又不洩漏過多資訊給前端，因此我們使用了自訂的 `HttpException` 來處理「可預期錯誤」。

讓我們再來看一次這段長長的程式碼 (API 有再稍微調整一些)

```python
# main.py
from fastapi import FastAPI, HTTPException, status, Request
from fastapi.responses import JSONResponse
from typing import Any, Dict, Union

app = FastAPI()

class NewHTTPException(HTTPException):
    def __init__(self, status_code: int, detail: Any = None, headers: Union[Dict[str, Any], None] = None, msg: str = None) -> None:
        super().__init__(status_code, detail, headers)
        if msg:
            self.msg = msg
        else:
            self.msg = detail

@app.middleware("http")
async def get_request(request: Request, call_next):
    try:
        response = await call_next(request)
        return response
    except Exception as e:     # 非預期的錯誤
        print("Error:", e)     # 紀錄非預期的錯誤的 log
        return JSONResponse(
            status_code=500,
            content={"detail": "Internal Server Error"},
        )

@app.exception_handler(NewHTTPException)
async def unicorn_exception_handler(request: Request, exc: NewHTTPException):
    print("Error:", exc.detail)   # 紀錄可預期的錯誤的 log
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.detail},
    )

@app.get("/")
async def hello():
    try:
        print(a)
    except NameError as e:   # 可預期的錯誤
        raise NewHTTPException(status.HTTP_501_NOT_IMPLEMENTED, detail="This is Value Error")
    print("Hi")
    return {"message": "Hello World"}
```

之前問了方便介紹，log 的部分就先用 print 代替，而今天，我們要改回用 logger 了。

> 這邊使用 loguru，而不是預設的 logging

## 改用 logger

這邊為了方便，我們使用昨天有提到的 Gist 上的[範例](https://gist.github.com/nkhitrov/a3e31cfcc1b19cba8e1b626276148c49) 的 `logger.py`，並一樣把它放在 `logger.py`，內容上只調整一點點，就是讓結果也存在檔案中。

```diff
logger.configure(
    handlers=[
        {"sink": sys.stdout, "level": logging.DEBUG, "format": format_record},
+        {"sink": "day23.log", "level": logging.INFO, "format": format_record}
    ]
)
```

再來就是改主程式 `main.py`

```diff
# main.py
from fastapi import FastAPI, HTTPException, status, Request
from fastapi.responses import JSONResponse
from typing import Any, Dict, Union
+ from logger import init_logging, logger

app = FastAPI()

+ init_logging()

# class NewHTTPException(HTTPException):
# ...

@app.middleware("http")
async def get_request(request: Request, call_next):
    try:
        response = await call_next(request)
        return response
    except Exception as e:     # 非預期的錯誤
-       print("Error:", e)     # 紀錄非預期的錯誤的 log
+       logger.error(e)
        return JSONResponse(
            status_code=500,
            content={"detail": "Internal Server Error"},
        )

@app.exception_handler(NewHTTPException)
async def unicorn_exception_handler(request: Request, exc: NewHTTPException):
-   print("Error:", exc.detail)   # 紀錄可預期的錯誤的 log
+   logger.error(exc.detail)
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.detail},
    )

@app.get("/")
async def hello():
    try:
        print(a)
    except NameError as e:   # 可預期的錯誤
        raise NewHTTPException(status.HTTP_501_NOT_IMPLEMENTED, detail="This is Value Error")
    print("Hi")
    return {"message": "Hello World"}

```

接著測試一下，就可以在 terminal 看到
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F23_terminal_1.PNG?alt=media&token=8ba1e91c-b549-4bfb-b6d1-ca05916f755b&_gl=1*nkxw7i*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NjYwNDk5My4yOS4xLjE2OTY2MDU0OTIuMzMuMC4w)

以及在 log 看到
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F23_log_1.PNG?alt=media&token=4a4c64d8-c2c8-401f-933a-7022b7c24ec6&_gl=1*1sepu3v*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NjYwNDk5My4yOS4xLjE2OTY2MDU0NzMuNTIuMC4w)

而 Postman 收到的回應則是
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F23_postman_1.PNG?alt=media&token=045e7e3f-0c0b-4038-a1cf-13390dacecb2&_gl=1*1baapef*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NjYwNDk5My4yOS4xLjE2OTY2MDU1MDcuMTguMC4w)

可以發現，對於可預期的錯誤是可以正確記錄錯誤訊息的，而前端只會拿到我們自定義的訊息。

> 不可預期的錯誤也可以正確紀錄，這邊就不放結果了。

## 加上 INFO 的 log

除了錯誤訊息要記錄，假如 API 有被正確呼叫的話，我們希望它也可以被記錄下來，只是 log level 調到 INFO。

讓我們先這樣改

```diff
@app.middleware("http")
async def get_request(request: Request, call_next):
    try:
        response = await call_next(request)
+       logger.info("Info")
        return response
    except Exception as e:     # 非預期的錯誤
        # print("Error:", e)     # 紀錄非預期的錯誤的 log
        logger.error(e)
        return JSONResponse(
            status_code=500,
            content={"detail": "Internal Server Error"},
        )
```

如此一來，當執行 API 時沒有發生錯誤 (或是 raise Exception)，就可以順利的進入 `logger.info("Test Info")`。

但是，這樣寫會有一個問題，那就是處理可預期錯誤時，會進入 `NewHTTPException` 的 exception handler (middleware) 和 http middleware，進而導致一次多了 ERROR 和 INFO 兩筆 log

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F23_terminal_2.PNG?alt=media&token=c8dcdf9d-7692-4aa8-a327-a22177a4a3c8&_gl=1*1ww9v9r*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NjYwNDk5My4yOS4xLjE2OTY2MDYyODkuNTAuMC4w)

因此，就需要再多一層判斷

```diff
@app.middleware("http")
async def get_request(request: Request, call_next):
    try:
        response = await call_next(request)
+       if response.status_code < 400:
+           logger.info("Info")
        return response
    except Exception as e:     # 非預期的錯誤
        # print("Error:", e)     # 紀錄非預期的錯誤的 log
        logger.error(e)
        return JSONResponse(
            status_code=500,
            content={"detail": "Internal Server Error"},
        )
```

## 小結
今天只簡短的介紹在 Middleware 紀錄 log 的做法，原本想繼續介紹目前的做法的下一階段優化，但今天加班到比較晚，沒時間寫了，只好留到明天了...
