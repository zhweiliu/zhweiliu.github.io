---
title: "Design Patterns - Decorator Pattern"
date: 2023-03-01T22:40:29+08:00
categories: ["Design Patterns"]
keywords: ["Design Patterns", "Decorator Pattern", "設計模式"]
summary: |
  Decorator Pattern 透過修改已定義的行為，以擴展或變更其功能，而不需透過繼承和覆寫。 使用組合 (composition) 替代繼承 (inherit)，可動態地添加或移除行為，且不需要在繼承關係中堆疊子類別。 Decorator模式更具靈活性和可維護性，因此被廣泛地應用於軟體開發領域中。

我覺得這取決於設計模式 (Design Pattern) 的核心精神 : 用組合 (composition) 替代繼承 (inherit) 
---
利用 [Decorator Pattern - Design Patterns](https://www.youtube.com/watch?v=GCraGHx6gso&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=3) 學習設計模式 **Decorator Pattern**，並利用 Python 撰寫 sample code.

## 摘要
Decorator Pattern 透過修改已定義的行為，以擴展或變更其功能，而不需透過繼承和覆寫。

使用組合 (composition) 替代繼承 (inherit)，可動態地添加或移除行為，且不需要在繼承關係中堆疊子類別。

Decorator模式更具靈活性和可維護性，因此被廣泛地應用於軟體開發領域中。

## 繼承 (inherit) 架構面對的問題

假設當前的議題是 : 要在不同的渠道，如: Google Ads、 Meta Ads 或 Instagram，投放廣告 ， 需要建立特定的資料結構來處理廣告文案與投放渠道，並採用繼承的方式來建置資料模型，資料模型間的關係初步設計可能會如下圖所示

![](/images/design-pattern/decorator-pattern/01_inherit.png)

建立一個 AdContent 並定義廣告內容的行為，為文字廣告和圖片廣告分別建立 TextAd 和 ImageAd 並讓兩者繼承 AdContent；

同時，每個投放渠道對於廣告文案的限制可能有所不同，利用繼承建立不同渠道的類別，如: GoogleTextAd 、 LineTextAd、MetaImageAd...等等，透過覆寫來設定不同渠道的限制。

基於上述設計的思考邏輯:
若後續迭代需求要求為 ImageAd 增加一個 split 的屬性，使得圖片廣告可用分割區塊的方式放入更多圖片，對已存在的 GoogleImageAd 、 MetaImageAd 和 InstagramImageAd 實現相關屬性的操作方式。

顯而易見，AdContent 理應為 content 定一個泛用性較高的資料模型。然而，若當前的資料模型無法滿足新需求，則資料模型的調整也會影響整個繼承鏈上的類別。

根本的問題在於採用繼承鏈的架構，當新需求只能適用於特定或部份渠道，卻要對繼承鏈上所有的類別進行調整；當繼承鏈過於龐大或負責的時候，會導致開發迭代版本需要更多的時間成本。

## 用 Composition 替代 Inherit
 
Decorator Pattern 則透過包裹 (wrapper) 方式來達成組合目的。

![](/images/design-pattern/decorator-pattern/02_decorators.png)

因為 AdContent 提供對文案的操作行為，而投放渠道或是廣告類型 ，如:文字或圖片，對文案而言都是附加描述，因此可以將 AdContent 定義為最基礎的元件 (component) ； 

附加描述可以對元件的行為 (behavior) 或執行結果 (result)，做進一步的拓展、擴充或修改，因此將附加描述定義為 decorator 。 因 decorator 是獨立的，因此也可以被其他的 decorator 包覆。

若 decorator 修改執行 behavior 的參數，或是修改了 behavior 的執行結果，則會依據 decorator 的包覆順序做出對應的改變 : 
- 若有多個 decorator 對同一個 behavior 的傳入參數進行修改，則傳入參數的改變會從 outer decorator 影響 inner decorator ，最終影響 component 執性 behavior 時的傳入參數

- 若有多個 decorator 對同一個 behavior 的執行結果進行修改，則從 component 回傳的執行結果開始，由 inner decorator 的修改影響 outer decorator 的修改，直至最外層的 outer decorator 回覆執行結果。

## 設計方法

依據先前的小節說明整理出幾點資訊
- 需要定義出 **元件 (component)** 和 **附加描述 (Decorator)**
- 執行 behavior 是由外層 decorate 到內層 component ， behavior 的 signature 需統一，可判斷**附加描述是元件的一種** (is-a)
- 附加描述可以疊加、 outer decorator 需要呼叫 inner decorator ，可判斷**附加描述應具備 inner decorator 或 component** (has-a)

再依據上述三點描述，以廣告文案投放至不同渠道的議題為例，用 Decorator Pattern 方式設計類別之間的關係，如下圖所示

![](/images/design-pattern/decorator-pattern/03_design.png)

令 Decorator 繼承 Component 類，並在 Decorator 的建構方法傳入 component 實體 ( object | instance ) ， 使 decorator 具備包裹 (wrapper) 其他 decorator 和 component 的能力。

## Demo
用 Python 撰寫 Decorator Pattern 的示例。  
[Source code](https://github.com/zhweiliu/design-pattern-study/blob/master/03_DecoratorPattern/Demo.py)

以下分別提供了 4 種示例

### 示例 1
**Single decorator wrapped component**
```python
print(f'Example 1 - text ad decorator output:')
print(TextAdsDecorator(text_ad).get_content())
print(f'Example 1 - image ad decorator output:')
print(ImageAdsDecorator(image_ad).get_content())
```
![](/images/design-pattern/decorator-pattern/04_demo_example_1.png)

### 示例 2
**Apply multiple decorators**
```python
print(f'Example 2 - Google Ads with text ad decorator output:')
print(AdsChannelDecorator(TextAdsDecorator(text_ad), channel="Google Ads").get_ad_channel())
print(f'Example 2 - Meta Ads with text ad decorator output:')
print(AdsChannelDecorator(TextAdsDecorator(text_ad), channel="Meta Ads").get_ad_channel())
```
![](/images/design-pattern/decorator-pattern/04_demo_example_2.png)

### 示例 3
**Apply multiple decorators with different order**
```python
print(f'Example 3 - image ads decorator output:')
print(AdsChannelDecorator(ImageAdsDecorator(image_ad), channel="Youtube Ads").get_description())
print(f'Example 3 - exchange decorators output:')
print(ImageAdsDecorator(AdsChannelDecorator(image_ad, channel="Instagram Ads")).get_description())
```
![](/images/design-pattern/decorator-pattern/04_demo_example_3.png)

### 示例 4
**Inherit decorator class to setting different behavior**
```python
print(f'Example 4 - inherit image ads decorator output:')
google_image_ads = GoogleImageAdDecorator(AdsChannelDecorator(image_ad, channel='Google Ads'))
# assume google image ads size limitation with: 320 <= width <= 1920, 480 <= height <= 1080
google_image_ads.width = 2560.0
google_image_ads.height = 1440.0
print(f'Over Google Image Ads size limitation')
print(google_image_ads.get_description())
```
![](/images/design-pattern/decorator-pattern/04_demo_example_4.png)
