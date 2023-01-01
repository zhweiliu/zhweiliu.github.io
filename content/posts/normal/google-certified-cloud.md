---
title: Google Certified 與 Cloud
date: '2021-09-22T14:14:13.175Z'
categories: ['Google Cloud Platform', 'Certified']
keywords: ['Google Cloud Platform', 'GCP', 'Certified']
---

轉換到雲端領域工作也過了大半年，這段不算長且還在進行中旅程中也獲取了三張 Google Cloud Platform ( GCP ) 的 Certified : [Associate Cloud Engineer](https://www.credential.net/2060f4eb-0afb-4628-9237-f5c8eae687e9) | [Professional Cloud Architect](https://www.credential.net/37dfa404-45f2-4896-bca4-6cea88bc4c5d) | [Professional Cloud Network Engineer](https://www.credential.net/36f2cddd-6111-450b-8ac0-5470a37ee034)

每每在考取認證的當下，也試著將這份喜悅分享給社群好友，也因此成為了開啟與好友交流雲端使用經驗的契機。

最近，和 Enzo 聊到在工作領域深耕的話題。Enzo 對資料科學的領域具有高度熱忱，也希望朝著 Senior Data Engineer 的角色發展；目前對於 Senior Data Engineer 的專業需求中，經常看到需要具備雲端平台的服務或工具等使用經驗；Enzo 除了使用中的 Google Compute Engine ( GCE ) Virtual Machine 服務之外，也希望進一步了解自學 GCP 的必要性與可能性，同時透過考取認證的方式確認自己學習的成果，以及希望將其作為對外證明的一舉兩得好方式。

和 Enzo 交流討論的過程中，我也從中發現一些值得紀錄的觀點。無論未來的我對這個觀點是抱持著贊同的態度，也或者大相徑庭，都是一種值得回味的思考。

以下透過幾個問題的交流過程，記錄我對使用雲端平台以及上雲這件事情的想法

> `拿認證對工作實戰的幫助以及對職涯的幫助，還是說有使用經驗其實不一定要拿認證，以實用性來說是不是熟悉其中幾項服務就足夠了 ?`

當初考慮轉換工作領域時，我也曾思考過這個問題；再陸續考取認證的過程中，也找到了一個自己認為合適的答案。

> `_考取認證僅證明你確實理解官方在這張認證領域上所提出的 Best Practice，並且具備將其轉換應用到實務上的基礎能力_`

`換句話說 : 認證是一個敲門磚。`

對外來說確實也是一個不錯的證明，面對非相同專業領域的人而言，這也代表了官方的背書。

> `推薦的學習路徑和學習資源 ：會建議先去拿助理認證，還是可以直衝專家認證 ?`

因為長期使用 GCE 的經驗，促使 Enzo 希望從 GCP 的認證作為起步。

對於希望學習 GCP 的人而言， Associate Cloud Engineer ( ACE ) 確實是幫助入門 Cloud 的好途徑：由於 Cloud 的資源眾多，若希望執行搬遷上雲的計畫， ACE 會以較為具體的執行方案帶你領略各個 GCP 資源的使用藍圖以及限制。

舉 Data 相關的工作為例，個人淺見認為不管從事的工作內容是 資料科學家 | 資料分析師 | 資料工程師 ，都無可避免的需要明確的知道資料從哪裡來、資料格式長相如何，以及處理過的資料最終需要往哪邊去。

因此，可以得到一個較為直覺的論點

> `資料流 ( Dataflow ) 與 資料處理 ( Process ) 可以單獨存在與單獨討論`

在過往的開發工作進行時，通常所使用的設備資源都是完整可見的硬體設施，而我們在這些設施上完成相關的開發作業；後來，當發現各家雲平台都有提供 Virtual Machine 的資源時，`最直覺的上雲`就是將工作搬往雲平台的 Virtual Machine ，藉此省下的硬體設施的費用。

至此，如果你問我使用雲平台的 Virtual Machine 算不算上雲了，我的回答是`看狀況`

> `雲端與地端的區別在哪裡 ?`

若我們以最廣為人知的定義來討論雲端與地端，則地端屬於公司內部設備，使用內部網路來隔離與外部網路的接觸，並透過層層保護設施來保護內部服務與資料不會輕易的被未授權的使用者存取；

而雲端則是使用雲平台提供同等硬體設施的虛擬資源，如： GCE、GKE，或者是雲平台提供的虛擬網路定義，如：VPC ( Virtual Private Cloud ) ，同時雲平台也提供底層的封包加解密、密鑰管理以及 Security Command Center 等等服務。

在使用雲平台的經驗中，可以`省下大量需要人力進行底層設施 ( Infrastructure ) 更換以及佈署的時間`。

然而，如果我們用雲平台的資源，做著與地端時期相同的工作，真的就是上雲了嗎 ?

> `硬體上雲與服務上雲`

在智慧手機不如現在功能強大的十幾年前，我們大多在個人電腦上使用社群媒體來與朋友進行交流；而現在，我們利用智慧手機來刷社群媒體的時間可能比坐在電腦前的時間還多。

誠然，我們將地端時期的工作搬遷到雲平台上，可以節省硬體設施等相關費用，但工作上的思考仍然停留在地端時期，我們只是將工作搬遷到另外一個環境上而已。

> `同樣是以 Dataflow 與 Process 的角度來看`

`地端時期`，資料流中的每個端點，如： NAS、SFTP 或是其他伺服器服務，都是明確的執行在具體一台伺服器設備，或是特定的叢集伺服器機群；這樣`明確的設備端到設備端的資料流向`也是地端時期的明顯特徵之一。

雲平台提供了 Virtual Machine 的資源，提供了從地端上雲的可能性；然而，我們會發現在這個模式的運作下， Dataflow 與 Process 仍然依賴著設備端到設備端的模式。

還記得前面提到的 `資料流 ( Dataflow ) 與 資料處理 ( Process ) 可以單獨存在與單獨討論` 嗎 ? 這表示它們其實並不需要依賴特定的設備端，明確地說

> 並不僅僅只能在 Virtual Machine 上實現

> `眾多服務為何只取 Virtual Machine ?`

雲平台提供了眾多的資源，幾乎可以滿足絕大部分的資料流設計與脫離 Virtual Machine 執行 Process 的方式，即 serverless，以下列舉幾個雲平台中與 Data 相關的資源

### Data Storage

*   Google Cloud Storage 提供 Bucket 資源可以儲存大量文件資料
*   Google Cloud Pub/Sub 提供 MQTT 服務，作為資料流渠道角色，同時也具備儲存批次資料的能力
*   Firebase 提供了最適合 Web 與 Mobile App 需要的儲存空間

### `Serverless`

*   Cloud Functions 提供了 serverless 的服務，適用於執行微服務 function、 stateless function 或是單純的 API endpoints

### Data Process

*   Dataprep 提供了視覺化與簡易編輯的 Transform 功能，也就是 ETL | ELT 中經常被提到的 “ T ”
*   Dataflow 提供了整個資料流的編排功能，可以透過視覺化工具來編排資料流的來源、處理以及目的地；同時也支援將雲平台中與資料相關的資源作為來源端或目的地端，也支援從其他雲平台引入資料

> `雲端一角`

至此便能總結出幾項要點，用以描述我對`雲端`的觀點

1.  將 workload 抽離對於資源的依賴性；workload 應可被單獨設計與討論
2.  雲端資源的提供者並非僅有一家。雲端與地端都能作為資料流來源，同時也能作為資料流去處
3.  節省硬體與底層資源的佈建佈署等成本

> `資料的尾巴`

為什麼通篇都以資料流作為舉例，並不斷地提起呢 ?

[Every company is a data company](https://www.coursera.org/lecture/gcp-fundamentals/every-company-is-a-data-company-2qy5f) 中給出了很好的解釋，有空的話可以看看