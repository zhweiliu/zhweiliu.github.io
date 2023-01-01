---
title: Customer Data Platform 是如何煉成的
date: '2022-03-22T13:35:08.068Z'
categories: ['Customer Data Platform']
keywords: ['Customer Data Platform', 'CDP']
showToc: true
TocOpen: true
summary: |
  提到 CDP ( Customer Data Platform ) ，可能就會想到利用顧客相關資料，為顧客分群分類貼標籤，透過網站、經營社群或 APP 進行精準投放廣告，達到再行銷的成果；甚至是透過 Machine Learning，或結合 CRM 、 Google Analytics 等資料，達成預估市場規模、優化推薦商品等目的。
---

提到 CDP ( Customer Data Platform ) ，可能就會想到利用顧客相關資料，為顧客分群分類貼標籤，透過網站、經營社群或 APP 進行精準投放廣告，達到再行銷的成果；甚至是透過 Machine Learning，或結合 CRM 、 Google Analytics 等資料，達成預估市場規模、優化推薦商品等目的。

似乎只要有充分的資料，就能開始享受 CDP 為行銷帶來諸多好處。然而，具體上 CDP 是怎麼運作的 ? 又該如何善用 CDP 的功能呢 ? 或許可以從瞭解 CDP 是如何構成的開始。

### Customer and Data Platform

不如簡單粗暴的，試著從名稱上將 Customer 與 Data Platform 分開 ：

對於 Customer ，或者說是與顧客相關的資料，從會員帳號的建立到商品購買紀錄、網頁瀏覽操作紀錄，甚至是會員權益分級等等；因顧客的主動行為產生，無論是否存在誘因，並且蒐集起來寶貴資料。

如同每道料理都是由食材原料組成，僅有食材原料卻沒有辦法變成美味的料理；Data Platform 擔任烹飪者的角色，將這些寶貴資料進一步處理，端出一道又一道的營養又可口的美食。

### Bring Benefits with Data-Platform

回想一下料理的烹飪過程，從準備材料、對食材進行清洗、將食材切割成適當大小或醃製入味，到觀察火候並依序加入對應的食材，最後出鍋上菜，若上菜後發現味道不好，還可以再依據這次的經驗進行調整與修正。

Data Platform 也對資料進行清理、前處理，對處理後的資料進行重組，最終產生出有貢獻的數據成果，並透過轉譯的方式進行交付；整個流程就像下圖所描述：

![](/images/normal/customer-data-platform-1/image_0.pngvm)

洞察與發現 ( Insights Discovery ) > 貢獻與進化 ( Contribute Evolution) > 異常偵測 ( Anomaly Detection) > 洞察與發現 ( Insights Discovery ) > …

而在這一循環的流程中，都緊密的圍繞著一個核心 ( Kernel ) 在進行：如同CDP 是的 Kernel 是 Customer 、 麻婆豆腐的 Kernel 是麻婆一樣 。

Data Platform 中的所有處理、步驟以及流程，都是為了核心在服務。

#### 洞察與發現 ( Insights Discovery )

如果我們產生一個疑問，大多數的情況下在第一時間，我們都會問 :「發生什麼事 ?」 或是 「某個事件是不是造成什麼影響」，而不會是 「某個具體的量(或者數字)是多少」

> 這些問題本身就是 Insight 的催化劑，促使我們想進一步去分析、去理解

#### 貢獻與進化 ( Contribute Evolution)

提出問題並在分析後，若能找到一些可能的答案，便能利用 IFTTT ( If This Then That ) 進行問題簡化，即 :

> 如果發生 A 狀況，那麼會造成什麼結果 / 需要如何應對

這是一種將數據結果進行翻譯的過程，讓數據變成一個應對清單，並能輕鬆的交付給其他利益相關者，以便他們對結果進行下一步的操作。

#### 異常偵測 ( Anomaly Detection)

從上一步驟產生的應對清單，在利益相關者對其進行操作之後，如 : 廣告投放 ， 便能對清單進行驗證、討論；或是對特定名單進行持續的觀測，看看是否仍有與數據結果不符，或者相對異常的行為出現。

當上述的行為出現後，在與相關利益者們進一步討論原因 ( 回到了 Insights Discovery ) ，獲取新的或者增強應對清單 ( 又到了 Contribute Evolution ) ，再次投入進行操作 ( 再持續進行 Anomaly Detection )，以此往復直到這個問題不再需要處理，或者不再具備價值時，就能停止。