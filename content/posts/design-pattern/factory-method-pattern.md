---
title: "Design Patterns - Factory Method Pattern"
date: 2023-03-08T21:15:20+08:00
categories: ["Design Patterns"]
keywords: ["Design Patterns", "Factory Method Pattern", "設計模式"]
summary: |
  
---

利用 [Factory Method Pattern - Design Patterns](https://www.youtube.com/watch?v=EcFVTgRHJLM&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=4) 學習設計模式 **Factory Method Pattern**，並利用 Python 撰寫 sample code.

## 摘要
Factory Method Pattern 簡明扼要的說，就是定義 **製作者(Creator)** 和 **產品(Product)** 間的關係

- 定義製作者可生產的產品，需要產品的時候才讓製作者生產出產品
- 不同的製作者可以有不同的製作方式，但統一使用**生產 (produce)** 來明確要求進行產品產出的行為

依據上述兩點可以得知 :
- 為產品定義一個基礎類別，並將同類型的 **具體產品(Concrete Product)** 繼承該基礎類別
- 為製作者定義一個基礎類別，並將產出同類型產品的 **具體製作者(Concrete Creator)** 繼承該基礎類別
- 不同類型的具體產品，應有不同的基礎產品類別
- 產出不同類型產品的具體製作者，應有不同的基礎製作者類類別

![](/images/design-pattern/factory-method-pattern/01-definition.png)

當需要某一特定產品時，委託可生產該產品的製作者進行生產並交付；

```
我們並不需要在意具體是哪個製作者生產產品，
也不需要在意製作者用何種方式生產特定產品，
因為我們關注的部分為，是否可拿到特定產品。
```

*註:在 《Design Pattern》書中有給出明確的定義和說明。我覺得太過繞口，所以用自己的想法來表達*

---

## 舉例

假設有一個工業物流的場景，這個場景中將原礦定義為一級產品，並包含了二級產品與三級產品，每一層級的產品都由具體的工廠提供。考量日後存在統計庫存的需求，對於庫存與生產管理的要求如下 :

- 需要對原料產品規範對應生產的具體工廠
- 工廠需檢視當前原料庫存量，以判斷是否可生產產品
- 若一個工廠具備生產多種具體產品的能力，則假設庫存原料原料充足
- 若一個工廠具備生產多種具體產品的能力，則該工廠每次生產一樣隨機產品


### 產品

假設已知的所有產品如下表所示

| 產品名稱 | 層級 | 需求原料 [品名, 數量] | 產出數量 |
| :---- | :----: | :---- | ----: |
| 鐵礦石 | 一級產品 | | 1 |
| 銅礦石 | 一級產品 | | 1 |
| 鐵板 | 二級產品 | [鐵礦石, 1] | 1 |
| 銅線 | 二級產品 | [銅礦石, 1] | 3 |
| 電路板 | 三級產品 | [鐵板, 1], [銅線, 2] | 1 |

一級產品作為二級產品的原料、二級產品作為三級產品的原料，可得知
 - 產品可能作為下一層級的生產原料
 - 產品需要標註原料資訊、產出資訊，以及產品名稱

為此，將需求原料定義成一個資料組結構作為生產過程的依據，這會帶來以下好處
- 當製作者在生產過程需要參考需求原料時，可從當前已知的資料組結構中，提取需要的參數資料，而不需要考慮整體資料組結構包含什麼
- 保留需求原料資訊的擴充彈性，即便是刪除資料組結構中既定的參數，也只需要修改使用了刪除參數的生產過程，而不需要檢視所有的製作者

將需求原料資訊打包成一個整體，並傳遞給接收者使用，這是 **依賴注入(Dependency Injection)** ， 一種簡單有效的降低**軟體耦合**的方法。

### 製作者

在定義具體的製作者時，需要考量的是該製作者可以提供什麼產品；
製作者可以具備生產同類型中多種具體產品的能力，但每次只能生產出一個特定的產品。

為此，先定義出一個符合要求的製作者 **DefaultCreator** 的生產過程
```python
def produce(self) -> Union[Product, None]:
    # 取得產品資訊
    product_description = self.product.get_description()

    # 取得產品的原料需求資訊
    require_materials: list[Material] = product_description.materials

    material_to_product = {}

    # 檢核庫存
    for m in require_materials:
        # 判斷目前庫存中是否具備需求的原料
        if not self.check_material(m.require.name):
            # 若庫存中不具備需求原料，則無法生產產品
            return None

        # 計算該原料最多可製作的產品數量
        material_to_product[m.require.name] = self.materials[m.require.name] // m.amount

    # 從每項原料可製作的產品數量中取最小值。 若最小值不大於 0 ， 則表示無法生產產品
    product = self.product if min(material_to_product.values()) > 0 else None

    return product
```

若有不同於 **DefaultCreator** 的生產方式，則每種生產方式都可以定義一個對應的製作者。
假設希望使用隨機挑選產品來進行生產，定義一個 **RandomCreator** 的生產方式如下

```python
def produce(self) -> Union[Product, None]:
    # 定義隨機選取的產品清單
    product_list = [
        IronOre, CopperOre, IronPlate, CopperCable, CircuitBoard
    ]

    # 使用隨機方法圖選產品，並直接提供該產品
    self.product = product_list[random.randrange(len(product_list))]()

    return self.product
```
---

## Demo

用 Python 撰寫 Factory Method Pattern 的示例。  
[Source code](https://github.com/zhweiliu/design-pattern-study/blob/master/04_FactoryMethodPattern/Demo.py)

### 示例 : 指定生產具體產品

這個示例指定了鐵板作為 DefaultCreator 生產的具體產品，同時要為檢視庫存與投料提供對應的方法

```python
# 指定鐵板為 DefaultCreator 生產的具體產品
factory = DefaultCreator(product=IronPlate())

# 檢視當前庫存
factory.list_materials()
print(f'DefaultCreator produce : {factory.produce()}')

# 投料
factory.update_material(material=Material(require=IronOre(), amount=10))

# 再次檢視當前庫存
factory.list_materials()

# 生產具體產品
print(f'DefaultCreator produce after update materials : {factory.produce()}')
```

執行結果
![](/images/design-pattern/factory-method-pattern/02_demo_1.png)

### 示例 : 隨機生產具體產品

這個示例中， **RandomCreator** 會從所有產品中，隨機挑選一種產品進行生產

```python
# RandomCreator 隨機挑選產品進行生產
random_factory = RandomCreator()
# 第一次隨機生產
print(f'RandomCreator produce : {random_factory.produce()}')

# 第二次隨機生產
print(f'RandomCreator produce : {random_factory.produce()}')

# 第三次隨機生產
print(f'RandomCreator produce : {random_factory.produce()}')
```

執行結果
![](/images/design-pattern/factory-method-pattern/02_demo_2.png)


### 示例 : 隨機生產一級產品

因一級產品和二級產品會作為下一層級的原料，這邊也選擇用隨機挑選生產產品，並用一級產品製造者 **OreCreator** 進行演示。和 

與 RandomCreator 不同的部分，則是在 **生產(produce)** 方法提供不同的產品列表。

```python
# OreCreator 生產方法
def produce(self) -> Union[Product, None]:
    # 僅選擇一級產品作為隨機挑選的產品清單
    ore_list = [IronOre, CopperOre]

    self.product = ore_list[random.randrange(len(ore_list))]()

    return self.product
```
執行結果
![](/images/design-pattern/factory-method-pattern/02_demo_3.png)