---
title: "2024 DevOpsDays 心得"
date: 2024-07-11 23:02:11
tags: 研討會
---

7/10、7/11 這兩天是 [DevOpsDays Taipei](https://devopsdays.tw/2024/)，稍微記錄一下這兩天的心得與聽到的關鍵字們。
<!-- more -->


![](https://firebasestorage.googleapis.com/v0/b/images-7e754.appspot.com/o/blog%2Fdevopsdays_2024.jpg?alt=media&token=fc2442a6-7517-4ee5-a47f-8d806a7a0d2b)

## 共筆
[共筆](https://hackmd.io/@DevOpsDay/2024/%2F%40DevOpsDay%2FS1-lbMuLC)



## 7/10

### 【KeyNote】 Trunk-Based Development makes DevOps easy
#### 重點
- 把公司分成 4 種規模：
  - 發布週期：100、10、1、1/10 天
  - 發布緊急修復版本的次數：2.5、0.5、<0.1、0 (一個發布週期內)
- Release 的頻率越高，hotfix 的需求越低，緊急修復的情狀就越少
- 理想上，只需要持續地進行計畫性的 Release，沒有任何緊急修復

- 中心思想：
> 專家會練到不犯錯 (Amateurs practice until they get it right, but
professionals practice until they do not get it wrong.)

#### 心得
這邊提倡的是 Trunk Based Development，與比較常見的 Feature branch 的思路不同。Trunk Based Development 的概念是只要程式可以跑 (沒有 bug) 就可以 merge 到 main branch，不一定要等到功能完成才 merge。如此一來，就可以大幅增加 merge 的頻率，並減少 merge 時的痛苦 (因為可能需要解衝突)。

然而，這樣做最大的問題是，代表未開發完全的功能就會出現在正式環境中，因此，需要把這個功能給藏起來，讓使用者不會有機會操作到。常見的做法有兩個：最簡單的做法就是先不要做介面，正常執行時完全不可能觸碰到未開發完的程式碼；或是使用 feature toggle，用設定的方式讓這個功能先被關閉，之後甚至可以做到只開放給部分使用者使用，供早期測試用。

另外，為了確保 main branch 是正常的 (或是有信心認為它是正常的)，所以測試也很重要。

> 我覺得我們公司還是先繼續用 Feature branch 的做法就好，共同開發的人少，merge 通常不會有問題

### 【KeyNote】 從 API First 到 AI First

#### 重點
- 讓 AI 去產生合適的 API 的參數
  - AI 可以擷取出訊息重點，讓之後打 API 時，可以自動帶入合適的 API 參數
  - API 要寫的好，AI 才好懂，才容易提供合適的 API 參數
- AI 適合模糊計算，不適合精準計算
  > 預算適合由 AI 處理，但結帳不適合
- 有時為了方便，會在 API 內留一些捷徑，讓功能開發上比較方便，但如果要搭配 AI，這樣反而有害。要讓 AI 看得懂 API，就要讓 API 盡量精確，沒有不確定性
- [簡報連結](https://docs.google.com/presentation/d/10o1VN0Q-97eTwYN_N-UP8pzLlxrxSBJbfXCcq0mTEDk/edit#slide=id.p)
  
#### 心得
講者的 GPT 的思路跟前幾天 [GenAI Stars](https://genaistars.org.tw/)我們這組的思路很接近，只是他是用在電商，我們是用在醫病溝通平台。


### 【DevOps入門班】DevOps 實務中不可或缺的產品思維

#### 重點
- 產品四大風險
  - Value 價值風險：做出來沒人要
  - Usability 使用風險：沒人知道怎麼用
  - Feasibility 建構風險：團隊做不出來
  - Viability 市場風險：沒人願意做
- 產出 Output -> 成果 Outcome -> 影響 Impact
- 做出產品 -> 使用者滿意 (使用者) -> 公司賺錢，股票上漲 (市場)
- 中心思想：
> 應該要最大化成果和影響，而非產出


- 機會與解決方案樹
- 北極星指標 (共同目標)
  - 直接用 Netflix 舉例比較好懂
    - 目標：提高串流總觀看時間
    - 團隊 A：提升使用者數
    - 團隊 B：提升每次觀看的時數
    - 團隊 C：提升每月觀看次數

#### 心得
這場對我來說重點有兩個：
1. 之後不論是績效考核，或是撰寫履歷時，都要思考到底是在寫產出、成果，還是影響?
2. 今天收到一個新需求時，一定要思考：
   - 預期效果是什麼?
   - 這樣做，真的可以達到預期效果嗎?
   - 要怎麼知道有沒有達成預期效果?

### 【DevOps入門班】超越直覺：產品交付階段的數據驅動科學
沒想法

### 【DevOps入門班】建立 Agile / DevOps 團隊的經驗甘苦談
沒想法

### 【DevOps入門班】何謂 GitOps? 為什麼愈來愈多團隊開始導入?
#### 重點
- 很清楚地解釋 Gitops，推薦看 PPT
- [簡報連結](https://speakerdeck.com/hwchiu/introduction-to-gitops)

#### 心得
很清楚的描述傳統的作法 (也就是我們公司現在的做法) 在產品變多時會遇到的問題，所以才提出 GitOps 這種解決辦法。不過，我們短期內繼續用傳統作法就好，等產品穩定再考慮其他做法。

## 7/11

### 【KeyNote】 從系統思考角度談 DevOps 三步工作法
前面有一小段沒聽到，內容也沒完全聽懂，之後再研究

### 導入DataOps沒想像中這麼難：大型媒體公司經驗談
介紹了 Data Pipeline 的演進 (工具、架構)
ETL/ELT
各種工具 

### 前端可觀測性 - Grafana 在下的一盤大棋
看來真的該去研究一下 Grafana 了

### Observability with Grafana
沒想法

### 衝出新手村，開發與維運的體驗進化之旅
一丁點想法都沒有

### 在 DevOps 中加入 AI 賦能：從自動化 Code Review 開始…
#### 重點
- [簡報連結](https://arock.blob.core.windows.net/blogdata202407/2024%20DevOpsDays%20TPE-p.pdf)

#### 心得
講者有料
看完之後，開始對於公司的開發流程有一些優化上的美好想像