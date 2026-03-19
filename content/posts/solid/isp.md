+++
title = 'SOLID：介面隔離原則 (ISP)'
date = 2026-03-05T00:34:04+08:00
draft = false
tags = [
    "basic",
]
description = "探討 SOLID 中的介面隔離原則 (ISP)，以及它如何與「組合取代繼承」的設計思維相輔相成，打造具備高靈活性的系統架構。"
+++

## 介紹

SOLID 是由 [Robert Cecil Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) 等人倡議的五項核心原則。

它們並非刻板的硬性規定，而更像是一種**精神指引**，幫助開發者建立易於維護、具備彈性的系統：

1. **SRP：單一職責原則 (Single Responsibility Principle)**
2. **OCP：開放封閉原則 (Open-Closed Principle)**
3. **LSP：里氏替換原則 (Liskov Substitution Principle)**
4. **ISP：介面隔離原則 (Interface Segregation Principle)**
5. **DIP：依賴反轉原則 (Dependency Inversion Principle)**

這篇將聚焦於第四個原則：**介面隔離原則 (ISP)**。

這項原則的存在感比較小，用來對「介面」的設計進行規範，

在物件導向設計中，遵循 ISP 往往能自然引導我們走向另一項重要的設計準則：「組合取代繼承」

## ISP：介面隔離原則 (Interface Segregation Principle)

>**不應強迫客戶端依賴它們不使用的方法。**
>(No client should be forced to depend on methods it does not use.
)

這項原則強調應該將大型、臃腫的介面（Fat Interface）拆分成更小、更專注的介面。

在系統設計上，ISP 對於應對未來的潛在改變非常重要；

若能妥善規劃介面，未來的重構與擴展將會更加順利。

### 程式舉例

假設今天我們需要設計一套多功能印表機的系統：

```java
    public interface IMachine {
        void print();
        void scan();
        void fax();
    }

    public class SmartPrinter implements IMachine {
        public void print() { /* do something */ }
        public void scan() { /* do something */ }
        public void fax() { /* do something */ }
    }

    public class BasicPrinter implements IMachine {
        public void print() { /* do something */ }
        public void scan() { throw new UnsupportedOperationException(); }
        public void fax() { throw new UnsupportedOperationException(); }
    }
```

上述例子中，**BasicPrinter**（基本印表機）被迫實作了它根本不需要的 **scan** 與 **fax** 方法。
這會導致以下問題：

1. 程式碼冗餘：出現許多拋出例外的無效實作。
2. 不同介面耦合過高：如果 scan 方法的介面改變，連帶不需要該方法的 BasicPrinter 也可能需要重新編譯或修改。

根據 ISP，我們可以將介面拆分如下：

```java
public interface IPrinter {
        void print();
    }

    public interface IScanner {
        void scan();
    }

    public interface IFax {
        void fax();
    }

    // 智慧型印表機實作所需的多個介面
    public class SmartPrinter implements IPrinter, IScanner, IFax {
        public void print() { /* do something */ }
        public void scan() { /* do something */ }
        public void fax() { /* do something */ }
    }

    // 基本印表機僅實作自己需要的介面
    public class BasicPrinter implements IPrinter {
        public void print() { /* do something */ }
    }
```

## 組合取代繼承(Composition over Inheritance)

ISP 將臃腫的介面拆分為多個專一的小介面，
這種「模組化」的思維與《設計模式》中的核心準則高度契合:

>「優先使用物件組合，而不是類別繼承」
>(Favor object composition over class inheritance)

### 什麼是組合

組合是一種 **"has-a"** 的關係，

將多個物件 (通常宣告 interface) 組合在一起，來實現複雜功能。

被稱為**黑箱復用(Black-box reuse)**，

因為物件之間只透過介面互動，無須知道彼此的內部細節。

透過組合來構建類別，可以帶來以下優勢：

1. 低耦合：物件之間維持獨立，修改一個類別對其他類別的影響降到最低。
2. 動態性：可以在執行時期更換組合的物件，靈活改變行為。
3. 職責單純：各介面與類別的職責簡潔化，避免了繼承體系過深或類別爆炸的問題。

## 結論

ISP 表面上關注的是消除冗餘的程式碼與方法，但其最深層的目的是：

> 確保介面設計具備高度的靈活性與內聚力

當介面被正確隔離，系統的組件就能像樂高積木一樣，透過「組合」靈活應對各種需求變化。

## 參考

- Clean Architecture - Robert C. Martin
- Design Patterns: Elements of Reusable Object-Oriented Software - Gang of Four, GoF
