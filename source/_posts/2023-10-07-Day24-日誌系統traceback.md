---
title: "[Day 24] 日誌系統 (五)：用 traceback 取得更完整訊息"
date: 2023-10-07 23:51:58
categories: 鐵人賽
tags: ithome Python FastAPI
---
昨天礙於時間緊迫，只好把再進一步優化的內容放到今天討論。

<!-- more -->

## 回顧一下主程式

昨天我們為了方便管理 log，我們把錯誤處理和 log 都放到了 middleware，目前不論是
1. 正常的 INFO
2. 可預期的 ERROR
3. 非預期的 ERROR

這三種狀況的 log 都可以正確的紀錄，

在繼續往下討論之前，先複習一下目前的程式碼

```python
# main.py
from fastapi import FastAPI, HTTPException, status, Request
from fastapi.responses import JSONResponse
from typing import Any, Dict, Union
from logger import init_logging, logger

app = FastAPI()

init_logging()

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
        if response.status_code < 400:
            logger.info("Info")
        return response
    except Exception as e:     # 非預期的錯誤   
        logger.error(e)        # 紀錄非預期的錯誤的 log
        return JSONResponse(
            status_code=500,
            content={"detail": "Internal Server Error"},
        )

@app.exception_handler(NewHTTPException)
async def unicorn_exception_handler(request: Request, exc: NewHTTPException):
    logger.error(exc.msg)           # 紀錄可預期的錯誤的 log
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.detail},
    )

@app.get("/")
async def hello():
    # b = 1 / 0
    try:
        print(a)
    except NameError as e:   # 可預期的錯誤
        raise NewHTTPException(status.HTTP_501_NOT_IMPLEMENTED, detail="This is Value Error", msg=str(e))
    print("Hi")
    return {"message": "Hello World"}
```

> 我會依據測試內容決定是否要取消註解 API 內的 `b = 1 / 0`

## 所以問題在哪？

讓我們來看看這兩筆 log

非預期錯誤
```shell
2023-10-07 23:01:28.527 | ERROR    | main:get_request:27 - division by zero
```

可預期錯誤
```shell
2023-10-07 23:02:50.536 | ERROR    | main:unicorn_exception_handler:35 - name 'a' is not defined
```

非預期錯誤中的 `main`、`get_request`、`27` 指的是 middleware 中的 `logger.error(e)`
而可預期錯誤的 `main`、`unicorn_exception_handler`、`35` 則是指 middleware 中的 `logger.error(exc.msg)`

當初這個 log 格式所想要紀錄的內容有包含
1. 檔案名稱
2. function 名稱
3. 程式碼行數

但是，它這邊紀錄到都是 middleware 的資訊，都沒有照我們所預期的記錄下來。說難聽一點，就是這個訊息一點用都沒有。

在此，我們可以下一個簡單結論，我們為了讓程式碼簡潔，多包了一層以上的 `try` `except`，導致對當初這個範例的設計產生了影響。

因此，這邊我們要使用 [traceback](https://docs.python.org/zh-tw/3/library/traceback.html) (Python 內建的模組) 來調整 log 內容。

## 怎麼改？

我們要在 `format_record` 內進行調整 (這邊改比較方便)，讓 loguru 產生 log 之前，去抽換掉特定參數，讓它在產生 log 時代入我們想要的數值。

```diff
# logger.py
def format_record(record: dict) -> str:
+   n1, n2, n3 = sys.exc_info() # 取得Call Stack
+   if len(traceback.extract_tb(n3)) > 0:
+       lastCallStack = traceback.extract_tb(n3)[-1] # 取得Call Stack 最近一筆的內容
+       fn = lastCallStack.filename
+       lineNum = lastCallStack.lineno
+       funcName = lastCallStack.name
+
+       record["name"] = fn.split("\\")[-1].split("/")[-1]
+       record["line"] = lineNum
+       record["function"] = funcName

    format_string = LOGURU_FORMAT
    if record["extra"].get("payload") is not None:
        record["extra"]["payload"] = pformat(
            record["extra"]["payload"], indent=4, compact=True, width=88
        )
        format_string += "\n<level>{extra[payload]}</level>"

    format_string += "{exception}\n"
    return format_string
```

先來看看發生錯誤時 `sys.exc_info()` 的 output，這是非預期錯誤的
```shell
<class 'ZeroDivisionError'>
division by zero
<traceback object at 0x000001F4C6ECC740>
```

這是可預期的錯誤的
```shell
<class 'main.NewHTTPException'>
501
<traceback object at 0x00000187C33BC200>
```

而最後一個的這個 traceback 物件裡面就會包含我們想要的資訊。

為了不影響一般情況下的 INFO，我們這邊多加了一個判斷條件
```python
if len(traceback.extract_tb(n3)) > 0:
```

## 改完之後的結果？

非預期錯誤
```shell
2023-10-07 23:01:04.016 | ERROR    | main.py:hello:43 - division by zero
```
也就是 `b = 1 / 0` 這行

可預期錯誤
```shell
2023-10-07 23:00:25.087 | ERROR    | main.py:hello:47 - name 'a' is not defined
```
也就是 `raise NewHTTPException()` 這行

可以發現，我們成功讓 log 記錄到多往前追蹤一層的錯誤資訊 (不再是 middleware)

## 還差一點點

理想上，可預期錯誤我們希望可以指到 `print(a)` 那行，但目前我還不確定該怎麼做...

不過，換個角度，非預期錯誤才比較需要知道是哪一行程式碼發生錯誤，因此目前的版本還可以接受 XD

## 小結

今天我們用 traceback 來重新拿取錯誤訊息，並透過修改 log 內容的方法，讓我們可以在發生錯誤時獲得更多資訊，尤其是非預期錯誤。

明天應該會換一個主題了，因為我總覺得日誌系統寫到第五篇有點太多了...
(不然其實原本想繼續介紹 log server 的 XD)

