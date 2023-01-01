---
title: ETL | ELT 與 IoT Device Alive Check
date: '2021-09-27T10:47:11.023Z'
categories: ['Data Engineering']
keywords: ['ETL', 'Data Pipeline']
---

既上次發布 [Google Certified 與 Cloud](https://medium.com/@zhweiliu/google-certified-%E8%88%87-cloud-34bfa7b2067e?source=your_stories_page-------------------------------------) 後，和 Ryan 討論人流偵測系統中的資料流，以及感測設備是否存活的議題； Ryan 的工作背景是 Compute Vision 相關，相對於 ETL 資料處理流程中屬於提供 E ( extract ) 端服務的角色，也特別重視 extract 的功能是否都能如期發揮作用。

> `ETL | ELT 是流程還是系統 ?`

ETL ( Extract-Transform-Load ) 與 ELT ( Extract-Load-Transform ) 是資料處理中常見的處理流程代名詞；個人認為 ETL ≠ ELT ， L | T 的先後順序除了影響處理流程的腳本之外，其實也需要搭配 scenario 來一起討論，同時也可能需要`依賴應用系統的受眾群體特徵`，搭建出對應的處理框架，以期在合理的效能下達成提供資料的目的。

在上述過程中可以看出，ETL | ELT 會依據實際狀況而對於框架設計有所改變， `ETL | ELT 應屬於流程`，在實作完成後才會變成具體的系統；而流程則可以被獨立提出進行討論。

> `Extract 是否有在好好運作 ? 資料遺失是否可以避免 ?`

在 Ryan 提出的議題中，extract 的服務由具體的感應偵測設備產生 log 資料，並不斷的往後段進行傳送，以便進行分析或儲存；當 extract device 離線或者是發生故障，若沒有在第一時間進行確認與通知相關人員，往往要等到進行資料統計時才會發現資料遺失。

為此，主動進行 Check Sensor Is Alive 的機制看起來不可避免，或是有其他的途徑可以達成相同的目的呢 ?

> `Extract Device 的資訊屬於已知或者未知 ?`

回到人流偵測的場景中，我本身對於這個應用場景較為陌生，因此參考了《基於影像處理及深度學習的兩階段人流偵測系統》[[1]](https://www.artc.org.tw/upfiles/ADUpload/knowledge/tw_knowledge_666538533.pdf)

> 傳統計算人流的方法可能會在入口處利用計數 器手動計算，或者透過閘門式機械設備逐一計算，常 見的方法為紅外線感應或旋轉門計數，但是對於公車 候車亭等開放式、半開放式的場域而言，缺少固定入 口來協助逐一計算，因此本研究利用影像處理方法達 到智慧監控的目的。

透過影像處理方法，對 Camera 蒐集到的畫面特徵擷取並將 ROI ( Region of interest ) 作為機器學習的 input data 之一；從這段描述中可以分析出兩個先決條件

`1\. Camera 作為蒐集影像的設備，對應了 extract device 的角色`

`2\. Camera 的部署地點是已知條件，換句話說 : extract device 已紀錄在案`

已知條件的 extract device 在處理上會相對便利：由於已經確認該 device 有週期性或者需要不間斷的蒐集資料並回傳，透過設計 Slots 的方式來定期收取資料，並給予一些容許值；對於超出容許值的 device 或許就能夠先進行 Is Alive 的判斷機制，並盡早的通知相關人員或進行對應的處理。

> `為什麼需要容許值 ?`

為了方便設計 ETL | ELT ，在設計的最初通常會假設 extract device 蒐集並提供的資料量是 100% ，即沒有任何資料遺失。然而 100% 的資料傳遞表示實作好的系統中，各個環節都不會有任何意外狀況發生，如：extract device 永遠不故障或者不需要汰舊換新、偵測區域的地點永遠不會有施工與環境物變更、永遠都有人流經過 ROI或是 ROI 永遠不變更...等，往往在實際操作的經驗上，都會面臨因各種調整事件而造成的 incident 。

因此，透過討論達成共識並給定一個容許值，可以讓整個系統在達成原先設立目的的同時具備一些彈性，並可以在後續的處理 ( Process ) 上考慮加入容許值條件以進行調整。

> `Cloud IoT Core & Monitoring`

GCP ( Google Cloud Platform ) 上提供了 [Cloud IoT Core](https://cloud.google.com/iot-core) 的服務，讓 Device 可以直接或者間接將資料回傳至 GCP resources ，並且也提供了 [monitoring](https://cloud.google.com/iot/docs/how-tos/monitoring) 的功能來檢視 resource ，包含已記錄在案的 device ， 並提供視覺化的圖形報表以方便檢視功能運作的狀況，如：[uptime check](https://cloud.google.com/monitoring/api/resources)。

![](/images/normal/etl-elt-iot-device-alive-check/image_0.png)
AWS 、Azure 或是其他雲端平台可能也有提供相對應的功能組合或者服務，若沒有頭緒的話可以參考 GCP 給出的範例，並嘗試在不同的雲平台中討論合適的解決方案與架構。

## Reference

[1] [_基於影像處理及深度學習的兩階段人流偵測系統, 林泓邦 1 *、林仁信 2 、廖伯翔 3, 中華民國自動機工程學會第二十五屆車輛工程學術研討會論文集, 中華民國一百零九年十月三十日_](https://www.artc.org.tw/upfiles/ADUpload/knowledge/tw_knowledge_666538533.pdf)