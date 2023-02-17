---
title: "借助 ChatGPT 修改履歷"
date: 2023-02-17T23:29:09+08:00
categories: ["Resume", "ChatGPT"]
keywords: ["Resume", "ChatGPT"]
summary: |
  透過 ChatGPT 輔助修改履歷的過程，我覺得在和 ChatGPT 溝通的過程其實挺順暢的，簡單扼要並清楚的告訴 ChatGPT 當前遭遇的問題，以及相關的背景設定與假設條件，通常都能獲得不錯的結果。

  即使第一次輸出的結果並不符合我的期待，在明確告知 ChatGPT 後也能得到較為滿意的結果。

  而唯一需要注意的一點，是 ChatGPT 告訴我他會執行下載檔案的行為，以便讀取檔案內容；未來若有相似需要 ChatGPT 的協助，則應該更加注意是否已避開提供機敏性的資料。
---
*這篇文章主要記錄借助 ChatGPT 來修改履歷的過程*

由於正在尋找新的工作機會，但覺得自己實在是不擅長撰寫履歷，便想嘗試讓 ChatGPT 對於已寫好的初版履歷提供修改建議；
但轉念一想，何不乾脆請 ChatGPT 直接輸出修改好的內容，於是便有了這次嘗試。


## ChatGPT
[ChatGPT](https://openai.com/blog/chatgpt/) 是由 OpenAI release 的一套對談式 AI Model (撰文當下已被 Microsoft 收購)；由於對談溝通的特性，目前也有許多利用 ChatGPT 進行二次開發的服務與產品，多數是在各行業領域以擔任助手的角色，提供使用者諸多便利性。

在實際使用 ChatGPT 的過程，我感覺到 ChatGPT 應該是具備短期記憶的特性，而短期記憶又能夠讓我們建立起和 ChatGPT 的共識，令 ChatGPT 能夠明白當前問題的背景環境與假設條件，促使 ChatGPT 盡可能的回答出符合環境設定的回答。

### 與 ChatGPT 建構共識
我覺得在需要溝通以合作解決問題的情境下，我必須先知道 ChatGPT 是否已具備了某些基礎能力，以夠應付接下來合作過程中我所提出的需求；
對此，我對 ChatGPT 進行了以下測試

- 是否具備上傳檔案到 Google Driver 的能力
- 是否具備訪問 Public Files URL 的能力

#### 檔案上傳測試
我在自己的 Google Driver 建立一個共用文件夾，並設定共用權限讓知道連結者可進行編輯，再對 ChatGPT 提出問題
![](/images/chatgpt/resume/upload_test.png)

**ChatGPT 目前尚未具備上傳檔案的能力**

#### 訪問 Public Files URL 測試
我在前一個測試中建立的公用資料夾中上傳一個簡單的純文字檔案，檔案內容為 `Hello from ChatGPT!` ，並對 ChatGPT 提問
![](/images/chatgpt/resume/access_public_file_test_1.png)

ChatGPT 的輸出並非我想要的，於是繼續和 ChatGPT 溝通
![](/images/chatgpt/resume/access_public_file_test_2.png)

ChatGPT 正確訪問了檔案連結並讀出內容；

同時， ChatGPT 告訴我**他透過執行了 Python script 來下載檔案以及輸出內容**

### 讀取履歷
我事先準備好一份撰寫完成的履歷，並將履歷設定為可被公開訪問，並請 ChatGPT 讀取履歷後產生一份適合我的自傳；
同時，也告訴 ChatGPT 目前我想尋找的職缺優先順序為何
![](/images/chatgpt/resume/chatgpt_biography.png)

**在 ChatGPT 產生的自傳中，有 80% 描述符合我的過往經歷。**

### 追加條件以修改履歷
我進一步的追加一個條件
```text
假設每位 HR 只有 30秒的時間來查看一份履歷
```
並請 ChatGPT 依據條件，提出修改履歷的建議，以利增加履歷的獨特性與曝光度

簡略列出 ChatGPT 提供吸引 HR 目光的建議
```
1. 突顯關鍵字
2. 結構簡潔
3. 重點突出
4. 特色突顯
5. 具體數據
6. 簡明扼要
```

以及對履歷的修改建議
```
1. 強調成果
2. 更好的排版
3. 添加關鍵字
4. 突出自己的技能
5. 更新簡歷內容
6. 自我介紹
```

![](/images/chatgpt/resume/recommend_1.png)
![](/images/chatgpt/resume/recommend_2.png)


### 調整自我介紹和自傳
依據 ChatGPT 的建議，我分別提供簡單的自我介紹、自傳以及工作經歷，並請 ChatGPT 協助修改，以便更加符合 HR 查看履歷時著重的重點

**自我介紹**
![](/images/chatgpt/resume/self_introduce.png)


**自傳**
![](/images/chatgpt/resume/biography.png)


**工作經歷**
![](/images/chatgpt/resume/work_experience.png)


我覺得在 ChatGPT 修改過後，讀起來確實通暢許多。

### 結語
透過 ChatGPT 輔助修改履歷的過程，我覺得在和 ChatGPT 溝通的過程其實挺順暢的，簡單扼要並清楚的告訴 ChatGPT 當前遭遇的問題，以及相關的背景設定與假設條件，通常都能獲得不錯的結果。

即使第一次輸出的結果並不符合我的期待，在明確告知 ChatGPT 後也能得到較為滿意的結果。

而唯一需要注意的一點，是 ChatGPT 告訴我他會執行下載檔案的行為，以便讀取檔案內容；未來若有相似需要 ChatGPT 的協助，則應該更加注意是否已避開提供機敏性的資料。

