+++
title = 'SOLID：開放封閉原則 (OCP)'
date = 2026-02-22T23:42:03+08:00
draft = false
tags = [
    "basic",
]
description = "開放封閉原則 (OCP) 強調軟體實體應該對擴展開放，對修改封閉。本文探討 OCP 的核心定義與實作細節。"
+++

## 介紹

SOLID 是由 [Robert Cecil Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) 等人倡議的五項核心原則。

它們並非刻板的硬性規定，而更像是一種**精神指引**，幫助開發者建立易於維護、具備彈性的系統：

1. **SRP：單一職責原則 (Single Responsibility Principle)**
2. **OCP：開放封閉原則 (Open-Closed Principle)**
3. **LSP：里氏替換原則 (Liskov Substitution Principle)**
4. **ISP：介面隔離原則 (Interface Segregation Principle)**
5. **DIP：依賴反轉原則 (Dependency Inversion Principle)**

本篇將聚焦於第二個原則：**開放封閉原則 (OCP)**。

## Open-Closed Principle (OCP) 開放封閉原則

> **軟體實體應該對擴展開放，對修改封閉。**
> (Software entities should be open for extension, but closed for modification.)

軟體的唯一不變，正是「變」。

OCP 的核心目標是：當需求變更時，透過**增加新程式碼**來解決，而不是**修改舊有且穩定運作的程式碼**。

### 程式舉例：變動的演算法

承接上回 SRP 的例子，如果我們將邏輯寫死在一個方法內

```java

// 違反 OCP 的例子：每增加一種會員類型或折扣邏輯，就必須修改這個方法

public double calcUserOrder(User user, double price, int quantity) {
    if (user.isVIP()) {
      return price * quantity * VIP_DISCOUNT;
    } else {
      return price * quantity;
    }
}
```

如果今天要變動 VIP 的計算行為，必須修改函式內容。

作為既有的程式碼，我們應予以「封閉」，

避免因為修改，導致重新編義與重新測試。

除了按照上回的方式處理職責，我們是否有其他辦法呢?

當然有的，這裡舉其中一個辦法。

假設今天變動的地方是：計算行為。

我們可以這麼做：

```java
public interface OrderStrategy {
    double calc(double price, int quantity);
}

public class NormalStrategy implements OrderStrategy {
    public double calc(double price, int quantity) {
        return price * quantity;
    }
}

public class VIPStrategy implements OrderStrategy {
    private final double vipDiscount;

    public VIPStrategy(double discount) {
        this.vipDiscount = discount;
    }

    public double calc(double price, int quantity) {
        return price * quantity * vipDiscount;
    }
}

public class Order {
    private double price;
    private int quantity;

    public double calculateOrder(OrderStrategy strategy) {
        return strategy.calc(this.price, this.quantity);
    }
}
```

未來如果有「節慶促銷」或「新會員制度」，我們只需要新增一個實作 OrderStrategy 的類別。

**完全不需要更動到 Order 類別的原始碼**，只要擴充 OrderStrategy 種類就可以。

這個實作的解法，也是其中一項設計模式，策略模式 (Strategy Design Pattern)。

## 無止盡的擴充

事實上擴充方向非常多，也因此常常造成過度設計（Over-Engineering）。

「封閉」與「開放」是一個相對的概念。

只有需求真實存在時，「開放」才有意義。

擴充是基於**需求**，而非**預測**。

擴充的邊界，通常以「編譯單元」為界，目標是新增功能時不需要重新編譯或重新發布核心模組。

在函式層面，盡量保持主流程穩定，將變動的部分委派給外部物件。

### 核心目的：擴充

降低風險：不修改既有程式碼，就不會影響原本已經測試過的穩定邏輯。

擴充性高：面對新需求時，只需專注於編寫新的實作類別。

## 結論

實踐 OCP 時，我們最該關注的是：

> 這段程式碼哪裡會改變？

## 參考

- Clean Architecture - Robert C. Martin
