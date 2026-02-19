+++
title = 'SOLID：單一職責原則 (SRP)'
date = 2024-10-16T23:42:03+08:00
draft = false
tags = [
    "basic",
]
description = "單一職責原則 (SRP) 不僅僅是「只做一種事」，更是關於「為了什麼原因而改變」。本文探討 SRP 的核心定義與實作細節"
+++

## 介紹

SOLID 是由 [Robert Cecil Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) 等人倡議的五項核心原則。

它們並非刻板的硬性規定，而更像是一種**精神指引**，幫助開發者建立易於維護、具備彈性的系統：

1. **SRP：單一職責原則 (Single Responsibility Principle)**
2. **OCP：開放封閉原則 (Open-Closed Principle)**
3. **LSP：里氏替換原則 (Liskov Substitution Principle)**
4. **ISP：介面隔離原則 (Interface Segregation Principle)**
5. **DIP：依賴反轉原則 (Dependency Inversion Principle)**

本篇將聚焦於最容易被誤解，卻也最基礎的原則：**單一職責原則 (SRP)**。

## Single-responsibility principle (SRP) 單一職責原則

> **一個類別應僅有一個理由使其改變。**
> (A class should have only one reason to change.)

從微觀的函式、更大的類別，到宏觀的模組或服務，SRP 都適用。

Uncle Bob 後來更精確地定義了所謂的「理由」：

> **一個模組應僅對一個利益關係人 (Actor) 負責。**
> (A module should be responsible to one, and only one, actor.)

### 程式舉例：糾纏的職責

``` java

// 這是一個函式違反的例子

public double calcUserOrder(User user, double price, int quantity) {
    if (user.isVIP()) {
      return price * quantity * VIP_DISCOUNT;
    } else {
      return price * quantity;
    }
}
```

這個例子中我們看見了什麼?

一個函式因為區別了 VIP 身份，因此有兩種的計算方式

當今天我們要修改 VIP 規則時，我們就需要對函式做調整

理想上我們應該這樣寫

``` java
public double calcVipOrder(User user, double price, int quantity) {
  return price * quantity * VIP_DISCOUNT;
}

public double calcUserOrder(User user, double price, int quantity) {
  return price * quantity;
}
```

> 怎麼少了分開 VIP 與一般用戶的 if 函式？

如果可以，這個區別身份的邏輯會被實作在「更上層」的地方。

確保實作層的計算邏輯純粹且單一。

## 說不準的職責

最容易讓人困惑的地方：

> 那件事，到底可以有多大？
>
> 職責可以多廣？

當尺度放大到類別時，又或是套件時，職責往往與「誰在對這段程式碼提要求」有關。

```java
// ❌ 類別違反 SRP 的例子
public class OrderService {
    public double calculateOrder(User user, double price, int quantity) {
        // 邏輯 1：如果是 VIP，給予折扣（這是「銷售部」的規則）
        if (user.isVIP()) {
            return price * quantity * VIP_DISCOUNT;
        } else {
            return price * quantity;
        }
    }

    public void saveToDatabase(Order order) {
        // 邏輯 2：儲存到資料庫（這是「工程部」關注的範圍）
        // ... DB 連線與寫入邏輯 ...
    }
}
```

當銷售部想改折扣，或者工程部想換資料庫時，都會動到 OrderService。

這代表：

> 同一個模組，同時對兩個不同的 Actor 負責。

如果可以，我們應該將這兩個職責分開：

```java
// 專注於定價規則（服務於銷售部門）
public class PricingCalculator {
    public double calculateVipPrice(double price, int quantity) {
        return price * quantity * VIP_DISCOUNT;
    }
    
    public double calculateStandardPrice(double price, int quantity) {
        return price * quantity;
    }
}

// 專注於資料持久化（服務於工程部門）
public class OrderRepository {
    public void save(Order order) {
        // 僅處理 DB 邏輯
    }
}
```

### 依照 Uncle Bob 的說明：

> **只為一個利益關係人（Actor）負責**

因此單一職責，取決於你的**軟體架構**與**組織需求**。

例如：

- 財務部 → 關注計算規則與稅務  
- 行銷部 → 關注報表格式與呈現  
- 設計部 → 關注回傳格式與互動  

**這代表要考量組織結構**

在小團隊中，行銷與財務可能是同一人。
這時候，你的類別可能不需要拆得太細。

但當組織成長、需求來源變多，
原本的「單一職責」可能會變得臃腫。

此時，就必須隨著時間與組織演進，重新調整邊界。

### 核心目的：內聚 (Cohesion)

SRP 的目的不是要把程式拆到支離破碎，而是為了讓程式足夠「**內聚**」。

如果兩段代碼總是因為同一個原因同時修改，那它們應該待在一起。

如果兩段代碼變動的頻率不同、來源不同，那它們就應該被分開。

## 結論

許多人會將 SRP 誤解為「一個類別只做一件事」。

但 SRP 真正關注的不是「做幾件事」，而是：

> 為什麼這段程式碼會改變？

## 參考

[單一職責原則 (SRP) - Uncle Bob](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)

[單一職責定義補充 - Uncle Bob](https://en.wikipedia.org/wiki/Single-responsibility_principle) 再次強調「理由」的定義，以及「利益關係人」的概念，是在 Clean Architecture 這本書中。