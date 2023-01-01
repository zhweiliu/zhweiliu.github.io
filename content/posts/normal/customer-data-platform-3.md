---
title: Customer Data Platform 是如何煉成的 (三)
date: '2022-04-04T16:27:27.434Z'
categories: ['Customer Data Platform']
keywords: ['Customer Data Platform', 'CDP']
showToc: true
TocOpen: true
summary: |
  在最後給出的結論如下
  *   在前 6% 的首次訪問者中，超過 6% 的人會在後續訪問時產生購買行為
  *   整體而言，只有 0.7% 的首次訪問者，會在後續訪問時產生購買行為
  *   瞄準前 6% 的第一次訪問者名單，會使營銷投資回報率提高 9 倍
  因此，若是能夠在得到模型預測的結果後，依據營銷策略進行即時的投放處理，包含但不限於: EDM廣告、Coupon折價券或是限定綑綁折扣等等，建構起一個自動化營銷的方法；這也是一種 **Data Driven** 的方法。
---

在 [**Customer Data Platform 是如何煉成的 (二)**]中提到了 User Behavior，但 User Behavior 的資料從哪裡來，又該如何定義呢 ?
{{< innerlink src="posts/normal/customer-data-platform-2.md" >}} 

以 GA ( Google Analytics ) 為例，在 Web 或 Mobile APP 中進行 GA 埋 code，這些 code 可以是 GA 預設的事件，如 : Page View 、 Session Engagement 、 Activity User，或者是自定義的 event 等等。 GCP BigQuery 提供的 [Public datasets](https://cloud.google.com/bigquery/public-data) 中也提供已將 GA 資料轉化為 [ecommerce](https://console.cloud.google.com/bigquery?project=data-to-insights&page=ecommerce) 的 dataset ，來源為 [Qwiklabs: Predict Visitor Purchases with a Classification Model in BQML](https://www.qwiklabs.com/focuses/1794?catalog_rank=%7B%22rank%22%3A1%2C%22num_filters%22%3A3%2C%22has_search%22%3Atrue%7D&parent=catalog&search_id=16035547)

本篇文章也利用這份公開資料集進行說明 :

*   **What Goal We Need**
*   **Feature and Label**
*   **Improve and Tune**

![](/images/normal/customer-data-platform-3/image_0.png)

### User Behavior

比較直觀的是對 **behavior** 的理解，可以想像當消費者在不同的 **E-Commerce website** 進行瀏覽商品、獲得推薦或是購物車結帳等操作時，由於 website 的設計不同，消費者可能需要跨越不同的頁面、點擊不同的連結，或是輸入不同的資料等等。

因此，消費者在 website A 與 website B 的 「**behavior**」也會不同；從另外一個角度來看 : 同一個 website 中，消費者要達成相同目的的操作，必定會在有限個數的操作**途徑**中完成。而這些**途徑**也就構成了 **User Behavior** 基本單位，而有針對性、目的性的對途徑資料進行挑選，也就構成了一個 **specific behavior** 的定義。

### What Goal We Need ?

在一開始會想知道 website 訪問人數、購買次數以及轉化率各是多少；如下圖所示，分別是

*   訪問人數 : 約 74 萬
*   購買次數 : 約 2 萬
*   轉化率 : 約 2.7%

然而，僅從結果層面獲得的資料，無法描述每項商品的銷售情況，因此對每項商品進行排比

這可能就是常見的報表內容，即各項商品的銷售情況與 website 的成效指標；但是若更進一步的思考，這份報表所表達的是對資料進行統計後的資訊，或是常見的用詞 **Data Informed** ；那麼，分析的目標只是產生報表數據嗎 ?

或者，是希望能從資料中協助識別「**哪些訪問者更可能會促成購買商品的事件**」呢 ?

### Feature and Label

首先考慮有多少訪問人數進行了購買，包含了**第一次瀏覽就購買**以及**再次訪問後進行購買**

共有 (11,873 / 741,721) = **1.6%** 的人是在第二次訪問商品頁面時，才進行購買；雖然沒有一個正確答案，但普遍的原因可能是消費者在購買前會進行商品的比價。

以此為例，可以從原始資料中列舉一些因素，作為判斷訪問者是否會進行購買的依據 ( **feature** )，並將再次回訪是否產生購買行為作為答案 ( **label** ) 對其訓練一個模型 ( **model** )；期望在下次收集到相關資料時，模型可以識別並告知**訪問者是否會產生購買行為**。

第一次挑選 feature 時，對以下兩個因素進行分析

*   **bounces** : 訪問者是否立即離開 website
*   **time\_on\_site** : 訪問者在 website 停留的時間

通常在訓練和評估模型之前，直接判斷 feature 的選擇是好或不好都為時過早，但在 time\_on\_site 排比前 10 的結果中，只有 1 個客戶返回購買；而模型測試的準確率也確實不好。

### **Improve and Tune**

在原始數據中，可能有更多的 feature 可以幫助 model 進行識別購買行為起到作用；而找出 feature 的方法除了對所有排列組合逐一進行嘗試外，也可以透過與相關人員，如 : 營銷人員、UX 設計師、統計專家或資料科學家等等，進行討論並達成共識後得出。

在這個案例中，除了 **bounces** 與 **time\_on\_site** 之外，還可以加入以下的 feature

*   訪問者第一次訪問時，在結帳過程中經歷了多少次的操作 (距離)
*   流量的來源 : 透過搜索或是 referring site 等等
*   設備的類別 : 手機 、 平板 或是 PC
*   地理資訊 : 來自哪個國家

重新訓練模型後測試的準確率也有所提高，同時模型也能提供一個預測結果，告知該訪問者是否會進行購買行為

### Summary

[Qwiklabs: Predict Visitor Purchases with a Classification Model in BQML](https://www.qwiklabs.com/focuses/1794?catalog_rank=%7B%22rank%22%3A1%2C%22num_filters%22%3A3%2C%22has_search%22%3Atrue%7D&parent=catalog&search_id=16035547) 的案例在最後給出的結論如下

*   在前 6% 的首次訪問者中，超過 6% 的人會在後續訪問時產生購買行為
*   整體而言，只有 0.7% 的首次訪問者，會在後續訪問時產生購買行為
*   瞄準前 6% 的第一次訪問者名單，會使營銷投資回報率提高 9 倍

因此，若是能夠在得到模型預測的結果後，依據營銷策略進行即時的投放處理，包含但不限於: EDM廣告、Coupon折價券或是限定綑綁折扣等等，建構起一個自動化營銷的方法；這也是一種 **Data Driven** 的方法。

整篇文章寫到這裡，從 「What Goal We Need」 中辨別更具價值的目標、「Feature and Label」中定義需產出的貢獻與評斷方式，到最後「Improve and Tune」透過討論達成共識，並進行相對應的調整，讓整體結果能夠產出的更好貢獻，我認為這是一種不斷優化與建立 CDP 的好方法。