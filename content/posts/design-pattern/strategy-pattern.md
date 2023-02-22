---
title: "Design Patterns - Strategy Pattern"
date: 2023-02-22T11:32:13+08:00
categories: ["Design Patterns"]
keywords: ["Design Patterns", "Strategy Pattern", "設計模式"]
summary: |
  相對於繼承(inherit)， Strategy Pattern 則是組合優於繼承(composition over inheritance)的精神。假設有一個薪水計算器要給兩個不同的客戶使用 : 速食業客戶以每小時時薪和工時來核算薪水，外送業客戶以每單獎金和總外送單數來核算薪水。薪水計算器需要提供給不同業者不同核算薪水的方法， Strategy Pattern 則提供了一種方式，使得不同業者可以使用同一個計算器，並選擇不同的核算方式，來獲取薪水計算的結果。
---

利用 [Strategy Pattern - Design Patterns](https://www.youtube.com/watch?v=v9ejT8FO-7I&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc) 學習設計模式 **Strategy Pattern**，並利用 Python 撰寫 sample code.

## Strategy Pattern Definition
相對於**繼承(inherit)**， Strategy Pattern 則是**組合優於繼承(composition over inheritance)** 的精神。

假設有一個薪水計算器要給兩個不同的客戶使用 : 速食業客戶以每小時時薪和工時來核算薪水，外送業客戶以每單獎金和總外送單數來核算薪水。

薪水計算器需要提供給不同業者不同核算薪水的方法， Strategy Pattern 則提供了一種方式，使得不同業者可以使用同一個計算器，並選擇不同的核算方式，來獲取薪水計算的結果。

### 繼承(inherit)
繼承的作法可以讓 **子類 (sub classes)** 具有 **父類 (parent classes)** 定義過的**屬性(attributes)** 和 **方法(method)** ，因此可以**重複利用(re-used)** 父類中已經撰寫過的片段程式碼。

當子類需更改父類定義的屬性或方法時，需要透過 **覆寫(overwriting)** 的方式在子類重新定義。

下圖為繼承的示例
![](/images/design-pattern/strategy-pattern/01_duck_inheritance.png)

Wild Duck 、 City Duck 和 Cloud Duck 都是繼承 Duck 的子類，因此也能解釋這些子類**是一個 (is-a)**  Duck 類 ，而 Office Duck 繼承了 City Duck ，也能夠解釋 Office Duck is-a City Duck 。 

City Duck 能夠被解釋為 is-a Duck 類，這個解釋的關係也被 Office Duck 繼承了，因此也能夠說 Office Duck is-a Duck 類。

當子類覆寫父類的屬性或者方法後，繼承**子類的子類** (以下簡稱**孫子類**) 繼承的是覆寫後的結果。然而，若孫子類希望使用的是父類的定義，這時是否又需要在對孫子類進行覆寫呢？

舉個例子：
City Duck 覆寫了 Duck 的 `fly()` 方法，但 Office Duck 卻希望使用 Duck 的 `fly()` 方法； 在遵從繼承原則的情況下， Office Duck 需要在對自己的 `fly()` 方法進行一次覆寫。

在上述的例子中，當 `fly()` 的方法透過覆寫來進行變更，不可避免的會出現 **ˋ相同的片段程式碼** 出現在不同的地方，因此程式碼片段的重複使用性會大大降低，這也容易造成後續維護的困難，或是在需求擴增時對於方法的拓坦變得不那麼彈性。

### 組合(composition)
與繼承不同之處在於，**組合(composition)** 利用 **介面(interfaces)** 拓展可執行的方法 : 透過條件判斷(或預先提供)的方式選擇實際執行片段程式碼，但在介面的定義中僅提供唯一的**署名(signature)**。

實際執行片段程式碼的撰寫方式有很多，以下說明我採取的方式：
```
使用 Abstract Class 定義父類，並透過繼承父類的子類，來實作不同的執行片段程式碼
```

![](/images/design-pattern/strategy-pattern/02_duck_composition.png)

在上圖示例中，將 Duck 的 `fly()` 與 `eat()` 方法分別拓展成兩個介面， `IFlyBehavior` 與 `IEatBehavior` ，並分別定義不同的子類，如: `SimpleEating` `CannotEating` 繼承 `IEatBehavior`， `SimpleFlying` `CannotFlying` 繼承 `IFlyBehavior` ， 以撰寫不同情況的下需要執行的片段程式碼。

將 FlyBehavior 和 EatBehavior 設定為 Duck 類的屬性，並在 Duck 類的 **建構子(Constructor)** ，要求將 FlyBehavior 和 EatBehavior 作為必要傳入參數。 這使得 Duck 類會**有一個(has-a)** FlyBehavior 和 **有一個(has-a)** EatBehavior。

當 Duck 類建構子要求傳入的 Behavior 參數，使得 Duck 需要執行的方法變為`可選擇` ，因此也能夠將原先建構的 **繼承鏈關係** 解耦。

在編寫軟體時，只需要將符合場景需求的對應能力，以組合的方式輸入給 Duck 類建構子，變能預期軟體執行符合期望的行為。


## Demo
用 Python 撰寫 Strategy Pattern 的示例。
![](/images/design-pattern/strategy-pattern/03_demo.png)

`Salary Calculator` 要求傳入 `Paycheck Factor` 作為屬性，並提供 `salary_calculating()` 方法來計算薪資。

[Source code](https://github.com/zhweiliu/design-pattern-study/blob/master/01_StrategyPattern/Demo.py)
```python
from SalaryDef import SalaryCalculator
from PaycheckFactor import SimplePaycheck, TicketPaycheck

if __name__ == '__main__':

    salary_calculator_for_employee_1 = SalaryCalculator(SimplePaycheck(employee_id=1002003))
    salary_calculator_for_employee_2 = SalaryCalculator(TicketPaycheck(employee_id=3002001))

    print(f'employee_1 salary of this month : {salary_calculator_for_employee_1.salary_calculating()}')
    print(f'employee_2 salary of this month : {salary_calculator_for_employee_2.salary_calculating()}')
```

