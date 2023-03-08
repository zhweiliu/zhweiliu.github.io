---
title: "Design Patterns - Observer Pattern"
date: 2023-02-22T20:41:43+08:00
categories: ["Design Patterns"]
keywords: ["Design Patterns", "Observer Pattern", "設計模式"]
summary: |
  在 Observer Pattern 中，將會明確定義出兩種角色 : 1. IObservable : 被觀察者，如上述的 Server (A類), 2. IObserver : 觀察者，如上述的 Client (B類) 。讓 Server 主動推送(Push)狀態變更的信號給 Client，可以有效的改善輪詢帶來的缺點。
---

利用 [Observer Pattern - Design Patterns](https://www.youtube.com/watch?v=_BpmfnqjgzQ&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=2) 學習設計模式 **Observer Pattern**，並利用 Python 撰寫 sample code.

## 摘要
當有 A 、 B 兩類實體，其中實體 B 類實體為了某些原因，需要得知 A 類的某些狀態是否有改變；
若在 Server-Client 的架構中，假設 Server 為 A 類、Client 為 B 類 ，最簡單的想法是讓 Client 定期去問 Server 狀態是否已經改變，這樣的方式稱為**輪詢(Polling)**
![](/images/design-pattern/observer-pattern/01_polling.png)

然而，輪詢有些顯而易見的缺點：
- 大多時候輪詢的答案都是狀態未改變，屬於**無效的輪詢**
- 若同時有多個 Client 實體對 Server 進行輪詢，**會降低 Server 的處理效能**

讓 Server 主動**推送(Push)**狀態變更的信號給 Client，可以有效的改善上述的缺點
![](/images/design-pattern/observer-pattern/02_push.png)

因此在 Observer Pattern 中，將會明確定義出兩種角色 :
- **IObservable** : 被觀察者，如上述的 Server (A類)
- **IObserver** : 觀察者，如上述的 Client (B類)

## Pattern

```text
[0..*] 表示 IObservable 和 IObserver 的關係為 1-to-many，  
即 1 個 IObservable 可以同時註冊多個 IObserver 。
```

![](/images/design-pattern/observer-pattern/03_pattern.png)

### IObservable

如上圖所述，在 IObservable 介面定義 **register(註冊)** 、 **unregister(註銷)** 和 **notify(通知)** 三種基礎方法。
- `register` : 註冊 IObserver 。 使得 `notify()` 方法可以通知到已註冊的 Observer。
- `unregister` : 註銷 IObserver 。 已註銷的 IObserver 不會收到 notify 方法的通知
- `notify` : 促使 IObservable 通知目前已註冊且未註銷的 IObserver ， 使得符合條件的 IObserver 可以執行 `update()`

### IObserver

IObserver 中定義僅定義了 `update()` 方法，問題是 IObserver 需要和**誰**取得更新的狀態或訊息 ？

在 IObservable 的描述中，已知 IObservable has-a (or has-many) IObserver ， 因此 IObservable 執行 `notify()` 方法時會有具體的對象可以呼叫， 但 IObserver 卻沒有定義一個可以 update() 的對象。

為此，仍需實做**輪詢**中的一個重點步驟 : **詢問(Ask)** ，同時為了避免 **Circular Imports**的情形，**詢問(Ask)** 的實作會放在繼承了 IObserver 的子類建構子中，使得 IObserver 的子類能夠獲取具體向哪一個 IObservable 取得更新的狀態與訊息；

實際上 **詢問(Ask)** 是一個雙向的溝通，即：**有問有答**，因此在繼承了 IObservable 的子類中，也會定義對應的方法讓 IObserver 的子類可以獲取更新的狀態和訊息。

## Demo

整併上述提及 **詢問(Ask)** 的實作 ，並定義出 IObservable 和 IObserver 的子類，如下圖所示
![](/images/design-pattern/observer-pattern/04_demo_pattern.png)

用 Python 撰寫 Observer Pattern 的示例。  
[Source code](https://github.com/zhweiliu/design-pattern-study/blob/master/02_ObserverPattern/Demo.py)

```python
    # initial publisher and subscribers
    topic = Publisher()
    sub_1 = Subscriber(publisher=topic)
    sub_2 = Subscriber(publisher=topic)
    sub_3 = Subscriber(publisher=topic)

    # register subscribers to publisher
    topic.register(subscriber=sub_1)
    topic.register(subscriber=sub_2)
    topic.register(subscriber=sub_3)

    # set message for publisher
    topic.message = 'Demo Observer Pattern'
    print(f"{'-'*20}\n")

    # change message to verify notify and update
    topic.message = 'Change message 2nd times'
    print(f"{'-' * 20}\n")

    # unregister sub_1
    topic.unregister(subscriber=sub_2)

    # change message to verify notify and update
    topic.message = f'Unregister sub 2 (id: {sub_2.id})'
```

執行結果
![](/images/design-pattern/observer-pattern/05_print_result.png)

撰文的當下是 2023 年，而 YT 影片大約是 2017 年的產物，以 2023 年應用發展來看，我覺得 Observer Pattern 最常見的一種應用(或者說演化)，就是 [PubSub Pattern](https://notfalse.net/11/pub-sub-pattern)；

因此，我也直接以 Simple Pub/Sub 的概念來實做這個 Demo。