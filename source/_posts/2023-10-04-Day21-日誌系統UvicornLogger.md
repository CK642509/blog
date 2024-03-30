---
title: "[Day 21] 日誌系統 (二)：操作預設的 Uvicorn logger"
date: 2023-10-04 23:31:35
categories: 鐵人賽
tags: ithome Python FastAPI
---
延續昨天的內容，我們雖然成功的建立了自己的 log，但是我們自訂的 log 和預設的 log 都會顯示在 terminal，造成畫面很亂

> 謎之音：主辦單位要不要管一下！

因此需要想辦法把預設的關掉，或至少讓它不要在 terminal 印出訊息影響畫面。
<!-- more -->

## 取得預設 `logger` ... 們？

如果想要對 `logger` 進行操作，我們可以使用昨天用到的 `logging.getLogger()` 來取得 `logger`，只要在括弧內放入 `logger` 名稱即可。

> 沒有填的話就是 root logger

但，問題來了，如果我們要操作的是預設的 `logger`，我們就需要先取得預設的 `logger` 的名稱。

好在 `loggging` 有一個方法可以拿到 `logger` 名稱清單，那就是 `logging.root.manager.loggerDict`

> 這個我在文件中時在找不到哪裡有寫...

讓我們測試一下，用 `uvicorn main:app` 啟動 `main.py`

```python
# main.py
from fastapi import FastAPI
import logging

app = FastAPI()

for name in logging.root.manager.loggerDict:
    print(name)
```

在 terminal 可以看到已經有很多 logger 了

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F21_terminal_1.PNG?alt=media&token=f683113b-79be-42c0-8138-04c4c00c52a9&_gl=1*1x76mup*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NjQyNDcxOS4yNS4xLjE2OTY0MjQ3MjguNTEuMC4w)

此外，也可以發現，預設的 log 需要花一點時間才會印出東西 (因為要等到 uvicorn 啟動完成)。

## 尋找目標 `logger`

那在這些 `logger` 中，到底哪些是我們需要去調整的呢？

在公布答案之前，我們可以先來測試一下 (一樣的做法，啟動 `main.py` 後，用 Postman 打 API)

```python
# main.py
from fastapi import FastAPI
import logging

app = FastAPI()

logger = logging.getLogger("uvicorn")
logger.handlers = []

@app.get("/")
async def root():
    logger.info("Logger Info")
    logger.error("Logger Error")
    print("Hi")
    return {"message": "Hello World"}
```

結果如下
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F21_terminal_2.PNG?alt=media&token=281dfe36-c1d2-42db-8ab7-289dcb6038a3&_gl=1*1qfr8et*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NjQyNjc4OC4yNi4xLjE2OTY0MjY5MTcuNTQuMC4w)

這邊我們名稱為 `uvicorn` 的 `logger` 的 `handlers` 清空，並用它來手動紀錄 log。

可以發現，現在啟動再也沒有那四行預設的 log 了，但是打 API 一樣有。此外，Info 等級的 log 並沒有在 terminal 顯示出來，只有 Error 才有。

接著稍微調整一下，再試一次
```diff
# main.py
from fastapi import FastAPI
import logging

app = FastAPI()

logger = logging.getLogger("uvicorn")
logger.handlers = []

+ logger_ac = logging.getLogger("uvicorn.access")
+ logger_ac.handlers = []

@app.get("/")
async def root():
    logger.info("Logger Info")
    logger.error("Logger Error")
    print("Hi")
    return {"message": "Hello World"}
```

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F21_terminal_3.PNG?alt=media&token=c334e21d-e72e-4873-9a24-033a6d00c7f6&_gl=1*1kn3gc1*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NjQyNjc4OC4yNi4xLjE2OTY0MjY4MjcuMjEuMC4w)

可以發現，現在連打 API 都沒有了。

另外，如果用 `uvicorn.access` 這個 logger 去紀錄 log (`logger_ac.info("Logger access")`)，卻會出錯。
> 這部份程式碼沒放上來，發生錯誤的確切原因我也不太清楚...

所以，根據測試結果，需要額外控制的 logger 至少有兩個，分別是 `uvicorn` 和 `uvicorn.access`。

至於 `uvicorn.error`，目前我還不清楚觸發時機，但可以知道的是，它絕對不是設計用來在發生錯誤時觸發的。

> uvicorn 有個 [issue](https://github.com/encode/uvicorn/issues/562) 就是覺得這個 `error` 名稱取的並不恰當 XD

## 讓預設 `logger` 不顯示

回到最一開始的話題，畫面很亂怎麼辦？

我們只要照上面的方法，將 `handlers` 清空，再像昨天一樣自己建立自己的 logger 就好，後續使用也是呼叫自己的 logger。

## 不能修改預設的 `logger` 設定並拿來用就好嗎？

答案是可以的，既然都可以取得 `logger`，那肯定也可以在上面操作，具體的做法可以這樣做
```python
# main.py
from fastapi import FastAPI
from log import init_logging
import logging

app = FastAPI()

logger = init_logging()

logger_ac = logging.getLogger("uvicorn.access")
logger_ac.handlers = []

@app.get("/")
async def root():
    logger.info("Logger Info")
    logger.error("Logger Error")
    print("Hi")
    return {"message": "Hello World"}
```

```python
# log.py
import logging

def init_logging():
    logger = logging.getLogger("uvicorn")
    logger.handlers = []

    logger.setLevel(logging.INFO)

    ch = logging.StreamHandler()
    fh = logging.FileHandler(filename='example.log')
    formatter = logging.Formatter("%(asctime)s %(message)s")

    ch.setFormatter(formatter)
    fh.setFormatter(formatter)
    logger.addHandler(ch)
    logger.addHandler(fh)

    return logger
```

接著再測試一次
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F21_terminal_4.PNG?alt=media&token=5464d3b5-eaf4-4e17-aca4-6add870ce6f8&_gl=1*czsojt*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5NjQzMjU0MS4yNy4xLjE2OTY0MzI1NTIuNDkuMC4w)

可以發現，我們成功修改預設的 `logger` 了~

## 另一個做法

除了上面的做法 (取得 logger 後修改設定)，還有另一個做法，那就是在用 uvicorn 啟動 FastAPI 時，可以額外代入我們是先設定好的設定檔

```shell
uvicorn main:app --log-config=log_conf.yaml
```

至於 `log_conf.yaml` 要怎麼寫，[文件](https://docs.python.org/3/library/logging.config.html#object-connections)中有說明，但實在有夠難看懂，因此建議大家去看這個[範例](https://gist.github.com/liviaerxin/d320e33cbcddcc5df76dd92948e5be3b)，應該會好懂一些。

## 小結

(好像沒有太多重點可以列，只好換個標題)

今天主要介紹了如何取得預設 `logger` 並調整它的設定，算是相對進階的主題，希望大家可以看懂 XD
