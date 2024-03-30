---
title: "[Day 27] 背景任務 Background Task"
date: 2023-10-10 23:43:46
categories: 鐵人賽
tags: ithome Python FastAPI
---
剩下最後幾天了，接下來應該都會挑一些小主題來介紹，今天先從背景任務開始吧~

> 也有人翻作「後台任務」
<!-- more -->

## 什麼是背景任務？

這邊指的背景任務，簡單來說就是在 FastAPI 後端回傳 Response 之後，才進行的程式。

> 但硬要說的話，我覺得用分頭進行比較貼切。這邊附上 FastAPI 官網的[原文](https://fastapi.tiangolo.com/tutorial/background-tasks/#background-tasks)給大家參考：You can define background tasks to be run after returning a response.

通常用於某個需要花較多時間的流程上，例如：分析數據、機器學習的訓練等，當使用者向後端發送請求後，後端可以先回傳一個 [202](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/202) 表示接收到請求了，再開始執行後續的任務。

## 實作 Background Task

讓我們直接來看一個範例
```python
# main.py
from fastapi import BackgroundTasks, FastAPI
import time

app = FastAPI()

def print_msg():
    time.sleep(5)
    print("Hello")

@app.get("/")
async def read_main(background_tasks: BackgroundTasks):
    print("Hi")
    background_tasks.add_task(print_msg)
    return {"msg": "Hello World"}
```

在後端收到請求後，`background_tasks` 就添加了一個新任務去執行 `print_msg()`，這邊為了更明顯感受到兩者是分頭進行的 (API 不會等到 `print_msg()` 跑完才回傳回應)，我故意在裡面添加了 `time.sleep(5)` 讓程式晚 5 秒才印出訊息。測試的結果如下

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F27_terminal_1.PNG?alt=media&token=79966890-bf08-48e0-8e2f-865a167bbf4d&_gl=1*1639glg*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5Njk1MTE5Mi4zMy4xLjE2OTY5NTEzMTQuNTAuMC4w)

可以看到 `Hello` 是在 API 回傳回應 (也就是 log 紀錄 200 OK) 之後才印出來的。

如果要帶參數到 background task 的函數中，則可以加在後面。讓我們稍微修改一下上面的範例

```diff
  from fastapi import BackgroundTasks, FastAPI
  import time
  
  app = FastAPI()

- def print_msg():  
+ def print_msg(name: str):
    time.sleep(5)
-   print("Hello")
+   print(f"Hello {name}~")

  @app.get("/")
  async def read_main(background_tasks: BackgroundTasks):
      print("Hi")
-     background_tasks.add_task(print_msg)
+     background_tasks.add_task(print_msg, "Ithome")
      return {"msg": "Hello World"}
```

測試結果如下
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F27_terminal_2.PNG?alt=media&token=436fe7ed-1675-41c9-9884-071942cc2079&_gl=1*1etfg84*_ga*MTcwNTU5Njc2Ny4xNjk0Njk5NzY3*_ga_CW55HF8NVT*MTY5Njk1MTE5Mi4zMy4xLjE2OTY5NTE2OTIuNTIuMC4w)


## 前端如何知道背景任務的狀態？
或許此時大家會有新的疑問：那前端要怎麼知道後端的背景任務的狀態？或者前端能否知道背景任務已經執行完了？

我想，這部份的做法有不只一種，最基本的做法就是在背景任務執行完畢後，去修改資料庫的資料或某個全域變數，之後如果前端想要知道，就再向另一支 API 發送請求確認資料或變數即可。這對於沒有需要那麼即時呈現的使用情境來說是比較適合的，只要前端設計一個重新整理的按鈕去觸發這個 API 就好。

但上面的做法對於需要即時呈現的情境 (例如：查看任務處理進度的進度條或進度百分比)，就沒那麼適合。

此時的問題變成：後端如何主動發送通知到前端？這部份的答案或許也不只一種，但最常見的做法就是使用 WebSocket，這是一個不同於 HTTP 的網路傳輸協定，在 FastAPI 也可以做到，但這就留到明天再繼續介紹了~

## 小結
今天我們介紹了如何設定背景任務，明天會繼續介紹 WebSocket，讓前端可以即時知道背景任務執行完畢了。

