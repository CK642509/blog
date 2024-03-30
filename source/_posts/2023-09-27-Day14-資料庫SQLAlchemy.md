---
title: "[Day 14] 資料庫 (一)：ORM 與 SQLAlchemy"
date: 2023-09-27 23:45:38
categories: 鐵人賽
tags: ithome Python FastAPI
---
接下來這幾天讓我們來看一下資料庫的部份~

> 儘管這系列主題是 FastAPI，但由於後端通常都會有 DB，而且 SQLAlchemy 套件本身功能也是不少，因此我還是打算多花點篇幅多介紹一下 SQLAlchemy
<!-- more -->

## RDBMS 與 NoSQL
資料庫簡單來說就是儲存資料的地方 (廢話 XD)，但既然稱為資料「庫」，就是預期資料的儲存量不低，或是有能力儲存大量的資料，因此就有人開發出相對好管理的資料庫格式。而資料庫又可以在分成兩大類，「關聯式資料庫」與「非關聯式資料庫」。

> 雖然翻譯上不完全相等，但通常英文會簡寫成 RDBMS (或 SQL) 和 NoSQL，大家看得懂就好

### SQL
先來說一下什麼是 SQL，SQL 全名是 Structured Query Language，中文翻作「結構式查詢語言」，簡單來說就是操作資料庫的語言，基本的操作包含：新增、刪除，查詢、修改。

### RDBMS
RDBMS (Relational Database Management System)，關聯式資料庫管理系統，簡單來說是類似 Excel 的表格們，通常欄位都不太會變動，適合儲存格式相對簡單(而且穩定)的資料。

### NoSQL
NoSQL (non-SQL 或 Not only SQL)，非關聯式資料庫，主要是應對較複雜的資料格式，或是每次格式都不太固定的，雖然使用上較方便，但相對的，查詢速度就比較慢。

> 大家可以去看看這篇之前的[鐵人賽文章](https://ithelp.ithome.com.tw/articles/10187443)，寫得很好懂

## ORM

隨著技術發展，後來 ORM 出現了，它全名是 object-relational mapping，簡單來說就是把我們的想法轉變成 SQL 指令的 library，它的優點主要有這幾點：
1. 通用、廣泛
   由於不同資料庫之間的 SQL 語法還是略有差異，但在 ORM 就沒有這個問題，只要把資料庫連線設定弄好，ORM 就會自動幫我們轉換成對應的 SQL 進行操作
2. 安全
   由於不是直接下 SQL 指令，因此可以避免掉 SQL Injection 攻擊
3. 簡化
   讓整體的程式碼較好理解，也較容易維護

至於缺點，則主要是效能，畢竟多轉換成 SQL 才去操作，因此效能一定有差，但現在已經發展這麼多年，應該優化很多了，通常影響效能的關鍵還是在邏輯設計上 (例如：資料庫格式、查詢方法等)。

> ORM 是用來處理關聯式資料庫的，因此接下來都不會討論 NoSQL

## SQLAlchemy 實作

Python 中最知名的 ORM 就是 SQLAlchemy，同時它也是被 FastAPI 官網作為範例使用的。接下來讓我們簡單地實作一下 SQLAlchemy

### 安裝
首先，要先安裝 SQLAlchemy
```shell
pip install sqlalchemy
```
### 準備資料庫

既然 SQLAlchemy 是用來控制資料庫的，那當然也要有資料庫...吧？

如果手邊沒有現成的資料庫，又懶得準備，或者只是小型專案開發用的，可以考慮使用 [SQLite](https://www.sqlite.org/index.html)，不需要額外安裝，直接進到下一步 xD

接下來我們的 Demo 都會以 SQLite 為主~

### 設定資料庫連線

我們先建立 `database.py`，負責管理資料庫的連線，來看一下官網[範例](https://fastapi.tiangolo.com/tutorial/sql-databases/#create-the-sqlalchemy-parts)
```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
# SQLALCHEMY_DATABASE_URL = "postgresql://user:password@postgresserver/db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()
```

每個資料庫都會它對應的連線 URL (也就是 `SQLALCHEMY_DATABASE_URL`)，包含帳號、密碼等，但 SQLite 沒那麼複雜，只要想好路徑和資料庫名稱就好。這邊我們設定在當前資料夾，名稱為 `test.db`。

### 設定 model
既然是關聯式資料庫，我們就要規劃它要有哪些表格、有哪些欄位、欄位的格式是什麼？

因此，建立 `models.py` 來設定它們的格式，來看一下簡化過後的[範例](https://fastapi.tiangolo.com/tutorial/sql-databases/#create-the-database-models)
 
```python
# models.py
from sqlalchemy import Boolean, Column, ForeignKey, Integer, String

from database import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)


class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String, index=True)
    owner_id = Column(Integer, ForeignKey("users.id"))
```

相信應該不難理解，這邊有兩個表格，名稱分別是 `users` 和 `items`，各有四個欄位並分別記錄在兩個 class 內，並且有設定好它的型別。

### 在主程式使用

接下來就是在主程式 `main.py` 進行資料庫連線。SQLite 有一個特點，只要連線時發現不存在，就會幫我們建立一個新的，因此我們只要直接在主程式啟動後，與資料庫進行連線就好了。

來看一下這個極度簡化過後的官網[範例](https://fastapi.tiangolo.com/tutorial/sql-databases/#main-fastapi-app)

```python
# main.py

from fastapi import FastAPI

import models
from database import SessionLocal, engine

models.Base.metadata.create_all(bind=engine)

app = FastAPI()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

get_db()
```

接著我們只要照平常的方式啟動主程式就好了
```shell
uvicorm main:app
```

接著就會在資料夾看到一個新檔案 `test.db`
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F14_vscode_1.PNG?alt=media&token=f7a8e466-15c8-46f3-b830-013b342c4d16)

### 查看 SQLite

讀取資料庫內容需要另外安裝軟體，因此這邊建議大家去下載 [SQLite browser](https://sqlitebrowser.org/)

> 進到 Download 頁籤就可以找到載點了

> 它會問你要不要一起安裝 SQLCipher，它是有加密保護的 SQLite，這邊我們用不到

安裝後，再次點擊 `test.db`，就會看到提示視窗

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F14_sqlite_3.PNG?alt=media&token=cbb72c1b-7710-4a1c-9626-e88900883513)

接著選擇 DB Browser for SQLite

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F14_sqlite_4.PNG?alt=media&token=b9df5c49-0380-4aa9-bac7-ac66f886dee9)

就可以成功打開來了

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F14_sqlite_1.PNG?alt=media&token=3c026500-4410-4e60-8f53-c96b1d9a6369)

接著點選 Browse Data 頁籤，就可以看到空的 Table 了，並且欄位都已經設定好了

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome%2F14_sqlite_2.PNG?alt=media&token=eeb444a5-2963-4566-b5a8-fb6b01621d6c)

## 重點回顧
今天我們快速簡單的介紹了
1. RDBMS 與 NoSQL
2. ORM
3. 用 SQLAlchemy 建立與連線到 SQLite

明天再繼續把今天省略掉的細節補充回來，包含
1. schema
2. 正確的資料庫連線方式

大家明天見~