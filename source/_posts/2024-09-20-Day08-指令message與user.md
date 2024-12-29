---
title: '[Day 08] 觸發條件 (三)：指令 ── message command 與 user command'
date: 2024-09-20 23:29:38
categories: 鐵人賽
tags: ithome Python Discord
---

今天接著繼續介紹指令：訊息指令與用戶指令。

<!-- more -->

## 進度

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F08_roadmap_2.jpg?alt=media&token=a4441b01-d298-44d4-88b7-b8e3b994906b)

訊息指令與用戶指令的概念差不多，就決定合併一起介紹了。

## 什麼是訊息指令 (message command)？
訊息指令，顧名思義，就是作用在訊息的指令。使用方式很簡單，只要在訊息上面按滑鼠右鍵，就會看到「應用程式」的選單，再選擇要使用的指令即可。（沒有預設指令）

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F08_default_message.PNG?alt=media&token=a7f071d4-134f-432c-ba61-31dde1580321)

## 什麼是用戶指令 (user command)？

同理，用戶指令就是作用在用戶上的指令，一樣是對著用戶按下滑鼠右鍵，再進入「應用程式」選單選擇要使用的指令。（同樣沒有預設指令）

對自己：
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F08_default_user_me.PNG?alt=media&token=c491f707-308c-410b-8fda-d582eabd0050)

對其他伺服器成員：
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F08_default_user.PNG?alt=media&token=47a49c17-e5b7-48e5-8231-ce3759125ad5)

## 第一個訊息指令與用戶指令

這兩種指令的撰寫方式非常相似，就一起介紹了。(範例程式簡化自 Github 上的[範例](https://github.com/Rapptz/discord.py/blob/master/examples/app_commands/basic.py))

```python
# day08.py

import discord
from discord import app_commands

GUILD_ID = "your guild ID"
MY_GUILD = discord.Object(id=GUILD_ID)


class MyClient(discord.Client):
    def __init__(self, *, intents: discord.Intents):
        super().__init__(intents=intents)
        self.tree = app_commands.CommandTree(self)

    async def setup_hook(self):
        self.tree.copy_global_to(guild=MY_GUILD)
        await self.tree.sync(guild=MY_GUILD)


intents = discord.Intents.default()
client = MyClient(intents=intents)


@client.event
async def on_ready():
    print(f'Logged in as {client.user} (ID: {client.user.id})')

@client.tree.context_menu(name="Show Join Date")
async def show_join_date(interaction: discord.Interaction, member: discord.Member):
    # The format_dt function formats the date time into a human readable representation in the official client
    await interaction.response.send_message(
        f"{member} joined at {discord.utils.format_dt(member.joined_at)}"
    )

@client.tree.context_menu(name="Report to Moderators")
async def report_message(interaction: discord.Interaction, message: discord.Message):
    await interaction.response.send_message(
        f"Thanks for reporting this message by {message.author.mention} to our moderators.",
    )

client.run('token')
```

與之前的 Quickstart ([Day 04](https://ithelp.ithome.com.tw/articles/10351677)) 一樣，可以大致上把這段程式分成三個階段：
1. 建立 Client
2. 設定 Client
3. 啟用 Client

今天的重點在第二階段的兩個函數：`show_join_date` 與 `report_message`。其中，`show_join_date` 是用戶指令，而 `report_message` 是訊息指令。

這兩個函數的寫法非常接近，有以下幾點注意事項：
1. 都是使用 `context_menu`。其中，`name`不是必要參數，但如果不填的話就會直接使用函數名作為訊息指令或是用戶指令的名稱。
2. 函數需要有「剛好」兩個參數，第一個一定是 `interaction`。
3. 函數的第二個參數一定要加上 type hint (用來指定是訊息指令或是用戶指令)。
   > type hint 只能是以下這幾個其中一個：`discord.Message`, `discord.User`, `discord.Member`, 或是 「`discord.Member` | `discord.User`」

## 測試

### 訊息指令
啟動後，就可以看到原本空的選單已經有剛剛建立的訊息指令了

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F08_demo_msg_01.PNG?alt=media&token=1e465047-88ed-4e7d-ba3f-f4fb5ee4b712)

執行結果：
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F08_demo_msg_02.PNG?alt=media&token=4ed4a5f5-dab1-47e5-9c9c-b8f6bef0363c)

### 用戶指令

用戶指令的選單也有更新

![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F08_demo_user_01.PNG?alt=media&token=879eb6d6-3286-4dc0-871a-65d6c6b62bae)

執行結果：
![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/ithome_2024%2F08_demo_user_02.PNG?alt=media&token=022e74f3-bca8-4be4-9130-5ab71430c476)

## 小結

今天把剩下的兩個指令：訊息指令 (message command) 和用戶指令 (user command) 都介紹完了，明天會介紹指令的其他細節，並補上之前 slash command 沒有介紹完的內容。


明後兩天都在高雄參加 PyCon，好期待啊！
