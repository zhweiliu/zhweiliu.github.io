---
title: "Design Patterns - Abstract Factory Pattern"
date: 2023-03-11T21:27:46+08:00
categories: [""]
keywords: [""]
summary: |
  簡而言之， Factory Method Pattern 描述的是定義製作者(Creator)和產品(Product)間的關係 ， Abstract Factory Pattern 則是一種將多個製作者(Creator) 群組化的關係；Abstract Factory Pattern是基於Factory Method Pattern建構而成的一種設計模式，因此需要先理解 Factory Method Pattern 的核心精神與設計方式。
---

利用 [Abstract Factory Pattern - Design Patterns](https://www.youtube.com/watch?v=v-GiuMmsXj4&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=5) 學習設計模式 **Abstract Factory Pattern**，並利用 Python 撰寫 sample 

## 摘要

簡而言之， Factory Method Pattern 描述的是定義製作者(Creator)和產品(Product)間的關係 ， Abstract Factory Pattern 則是一種將多個製作者(Creator) 群組化的關係；

**Abstract Factory Pattern** 是基於 **Factory Method Pattern** 建構而成的一種設計模式，因此需要先理解 Factory Method Pattern 的核心精神與設計方式

{{<innerlink src="posts/design-pattern/factory-method-pattern.md">}}

在[影片](https://www.youtube.com/watch?v=v-GiuMmsXj4&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=5)中提供的案例是應用程式的 UI 設計 :

當一組應用程式的 UI 控制項固定後，每個控制項功能背後要達成的目的是固定的，但依據作業系統的不同，如: Windows、Mac OS 、 Linux，控制項功能需要有不同的實作方式，如: 呼叫不同的 driver 或 kernel APIs，以達成目的，而 Factory Method Pattern 恰好適合解決這個問題；

因此，透過宣告 Abstract Factory Pattern，將每個控制項功能當作一種產品(Product)，透過不同的製作者(Creator)來產生控制項，並將適用於同一種作業系統的製作者(Creator)整合成一個群組，當 **需要渲染出符合當前作業系統的UI 時，只要採納對應的群組生成控制項功能就能達成目的**，這也表達了 Abstract Factory Pattern 的設計理念

![](/images/design-pattern/abstract-factory-pattern/01-definition.png)

---
## 舉例

假設有一速食快餐店要推出漢堡系列的套餐，套餐的組合中有主餐漢堡、副餐炸薯條和一杯飲料；其中炸薯條和飲料可以點選不同的尺寸選擇，並且預設為中份薯條和大杯飲料。

在這個場景中，使用 Abstract Factory Pattern 來實現套餐組合的過程。

### 產品 (Product)

整個套餐是透過不同的產品組合而成，假設當前已知的產品如下表

| 產品類型 | 產品名稱 | 單價 | 單位 |
| :---- | :---- | ----: | :---- |
| 肉 (Meat) | 牛肉 (Beef) | 30.0 | 片 |
| 肉 (Meat) | 豬肉 (Pork) | 25.0 | 片 |
| 肉 (Meat) | 魚肉 (Fish) | 20.0 | 片 |
|  |  |  |  |
| 飲料 (Drink) | 可樂 (Coke) | 依據尺寸 | 杯 |
| 飲料 (Drink) | 蘇打 (Soda) | 依據尺寸 | 杯 |
| 飲料 (Drink) | 綠茶 (GreenTea) | 依據尺寸 | 杯 |
|  |  |  |  |
| 薯條 (Fries) | 薯條 (Fries) | 依據尺寸 | 份 |
|  |  |  |  |
| 漢堡 (Burger) | 漢堡 (Burger) | 依據肉量 + 20.0 | 份 |

從產品表中可以整理出下列資訊 : 
- 漢堡 : 漢堡由麵包和肉類組成。依據加入的肉類與肉片量，有不同的售價。 
- 飲料 : 使用不同尺寸的杯子裝飲料。不同尺寸有不同售價。
- 炸薯條 : 薯條有不同尺寸份量。不同尺寸有不同售價。

進一步對飲料尺寸、薯條尺寸，與漢堡組成的方式做不同定義

**飲料尺寸**

利用**枚舉 (enumeration)** 定義飲料尺寸與售價，並**建立飲料尺寸的資料結構**來儲存尺寸和售價的資訊。

| 飲料尺寸 | 售價 |
| :---- | ----: |
| 特大(venti) | 10.0 |
| 大(grande) | 7.5 |
| 中(tall) | 5.0 |
| 小大(short) | 2.5 |

**薯條尺寸**

利用**枚舉 (enumeration)** 定義薯條尺寸與售價，並**建立薯條尺寸的資料結構**來儲存尺寸和售價的資訊。

| 薯條尺寸 | 售價 |
| :---- | ----: |
| 特大(supersize) | 10.0 |
| 大(large) | 7.5 |
| 中(medium) | 5.0 |


**漢堡組成**

將漢堡組成定義成一種產品，指定漢堡建構子需要傳入**肉類(meat)**和**肉片數量(piece)** 以產生不同類型的漢堡。


### 製作者 (Creator)

透過枚舉建立尺寸資訊的資料結構，當製作者需要生產不同尺寸的飲料/薯條時，只需要將資料結構傳遞給飲料/薯條的建構子，便能簡化產生具體尺寸的飲料/薯條的產品。

另外，將漢堡組成定義成產品的好處，在於製作者要生產漢堡時，也只需要傳遞對應的肉類和肉片數量，便可以生產不同具體類型的漢堡，如：牛肉漢堡、豬肉漢堡或魚肉漢堡。

為了滿足套餐組合的需求，分別需要定義**漢堡製作者(BurgerCreator)**、**飲料製作者(DrinkCreator)** 和**薯條製作者(FriedCreator)**。

同時，為了讓套餐組合時可直接選用，還需要分別建立不同的 漢堡/飲料和尺寸/薯條尺寸 的製作者，並讓其繼承該項產品類型的製作者。

| 製作者類型 | 製作者名稱 | 建構子需求 |
| :---- | :---- |  :---- |
| 漢堡製作者 | 牛肉漢堡製作者 | 肉類: 牛肉, 肉片量: 1 片 (預設) |
| 漢堡製作者 | 豬肉漢堡製作者 | 肉類: 豬肉, 肉片量: 1 片 (預設) |
| 漢堡製作者 | 魚肉漢堡製作者 | 肉類: 魚肉, 肉片量: 1 片 (預設) |
| | | |
| 飲料製作者 | 可樂製作者 | 飲料尺寸: [ 特大、大、中、小 ] |
| 飲料製作者 | 蘇打製作者 | 飲料尺寸: [ 特大、大、中、小 ] |
| 飲料製作者 | 綠茶製作者 | 飲料尺寸: [ 特大、大、中、小 ] |
| | | |
| 薯條製作者 | 薯條製作者 | 薯條尺寸: [ 特大、大、中 ] |


### 套餐組合 (combo)

套餐組合直接從已定義好的各類產品製作者進行挑選，以組成漢堡、飲料和薯條的套餐內容；透過選擇不同的產品製作者，便能夠快速地建立起不同的套餐內容

| 套餐名稱 | 套餐內容 |
| :---- | :---- |
| 牛肉漢堡套餐 | [牛肉漢堡、特大杯可樂、特大份薯條] |
| 豬肉漢堡套餐 | [豬肉漢堡、大杯可樂、中份薯條] |
| 魚肉漢堡套餐 | [魚肉漢堡、大杯綠茶、中份薯條] |


---
## Demo

用 Python 撰寫 Abstract Factory Pattern 的示例。  
[Source code](https://github.com/zhweiliu/design-pattern-study/blob/master/05_AbstractFactoryPattern/Demo.py)


### 示例 : 顯示套餐內容

不同的具體套餐組合都繼承了**套餐(Combo)** 類別，若想顯示不同具體的套餐的內容時，便可調用同一組程式碼來達成目的

**order information**
```python
# 傳入 Combo 類，即具體的套餐
def order_information(combo: Combo):
    # 獲取套餐內的漢堡
    burger = combo.get_burger()

    # 獲取套餐內的飲料
    drink = combo.get_drink()

    # 獲取套餐內的薯條
    fries = combo.get_fries()

    # 整合套餐內容的產品資訊，並計算整份套餐的售價
    description = {
        'burger': burger.get_description(),
        'drink': drink.get_description(),
        'fries': fries.get_description(),
        'total_price': burger.price + drink.price + fries.price
    }

    # 打印套餐類型的售價
    print(f'Combo: {combo.__class__.__name__}')
    print(description)
    print('-' * 30)
```

**主程式**
```python
if __name__ == '__main__':
    # 生產豬肉漢堡套餐
    pork_combo = PorkBurgerCombo()
    # 打印套餐內容
    order_information(pork_combo)

    # 生產魚肉漢堡套餐
    fish_combo = FishBurgerCombo() 
    # 打印套餐內容
    order_information(fish_combo)

    # 生產牛肉漢堡套餐
    hamburger_combo = SupersizeHamburgerCombo()
    # 打印套餐內容
    order_information(hamburger_combo)
```

**執行結果**
![](/images/design-pattern/abstract-factory-pattern/02-executed-result.png)

**小結**

在 Factory Method Pattern 和本篇 Abstract Factory Pattern 中，我沒有使用工廠這一翻譯措辭的原因: 
```text
我認為工廠的定義是可變的，依據不同的場景會對工廠一詞有不同的定義；
但工廠的目的是不變的，即生產產品，並且不需要關注由誰生產產品。
```

因此，我才使用了 產品、製作者、製作者群組 等詞來表達我認知的 Factory Method Pattern 和 Abstract Factory Pattern。

在本文的示例中所採用的編成架構，將各項具體單品(產品)、具體生產者，以及套餐組合進行分層，並且讓每一層只專注自己的變動，如 :
- 產品層只需要關注單品的參數，如名稱、售價，且單品參數的改變並不影響生產者層和套餐組合層的處理方式，以此為產品層異動保留了極大的彈性

- 生產者層同樣只考慮該如何生產出具體的產品，圍繞著建構子傳遞的肉類或尺寸的既有資訊進行拓展，而不是去改變肉類或尺寸的資訊

- 套餐組合層也僅考慮選用哪些具體生產者，來定義套餐組合的內容

未來無論是要增加新的套餐組合或是新的產品，不會(或非常小)對於現有的程式碼與軟體架構造成改動。 

我覺得這是一個，很好的闡述了鬆散耦合軟體框架的範例