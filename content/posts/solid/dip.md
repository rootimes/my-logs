+++
title = 'SOLID：依賴反轉原則 (DIP)'
date = 2026-03-05T11:07:20+08:00
draft = false
tags = [
    "basic",
]
description = "本篇文章將探討 SOLID 原則中的依賴反轉原則 (Dependency Inversion Principle)，並透過程式碼範例解析物件間的依賴關係與抽象的重要性。"
+++

## 介紹

SOLID 是由 [Robert Cecil Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) 等人倡議的五項核心原則。

它們並非刻板的硬性規定，而更像是一種**精神指引**，幫助開發者建立易於維護、具備彈性的系統：

1. **SRP：單一職責原則 (Single Responsibility Principle)**
2. **OCP：開放封閉原則 (Open-Closed Principle)**
3. **LSP：里氏替換原則 (Liskov Substitution Principle)**
4. **ISP：介面隔離原則 (Interface Segregation Principle)**
5. **DIP：依賴反轉原則 (Dependency Inversion Principle)**

本篇將聚焦於最後一個原則：**依賴反轉原則 (DIP)**。

## Dependency Inversion Principle

> 1. **高階模組（核心邏輯）不應依賴低階模組（細節實作），兩者都應依賴抽象。**
> (High-level modules should not depend on low-level modules. Both should depend on abstractions.)
> 2. **抽象不應依賴細節。細節應依賴抽象。**
> (Abstractions should not depend on details. Details should depend on abstractions.)

我們似乎又換了一個說法，講著同樣的事

> **Programming to an abstraction, not an implementation**

### 程式舉例：依賴、持有、組合?

在探討反轉之前，我們需要先釐清什麼是「依賴」。在物件導向設計中，物件之間的關係強度各有不同。

#### **依賴 (Dependency - "uses-a")**

```java
// 依賴的範例
public class A {
    public void call() {/*do something*/};
}

public class B {
    public void call(A inj) {
        A.call();
    };
}
```

當一個類別的方法使用了另一個類別，但沒有將其儲存為狀態，這是一種較弱的關聯。

#### **聚合 / 持有 (Aggregation - "has-a")**

```java
// 持有的範例
public class B {
    private A myA;

    public B (A inj) {
        this.myA = inj;
    }

    public void call() {
        this.myA.call();
    }
}
```

這時候是依賴嗎?還是?

> Ans:
> 這是 "has-a"
> 持有(has-a)是更強的依賴

持有，多數情況下也被稱作，聚合(aggregation)

#### **組合 (Composition - "contains-a")**

```java
// 組合的範例
public class B {
    final private myA = new A();

    public void call() {
        this.myA.call();
    }
}
```

這是 "composition"
> 組合的依賴，影響生命週期
> B 不僅擁有 A，還負責 A 的創建與銷毀。

#### 試問：這是什麼關聯?

刀在人在，刀不在人不在

### 程式範例：依賴反轉

什麼才是依賴反轉?

我們將原本的程式調整一下，來作說明

```java
public interface Contract {
    public void call();
}

// A 相對不穩(實作)
public class A implements Contract {
    @Override
    public void call() { /* do something */ }
}

// B 相對穩定(控制)
public class B {
    private Contract myContract;

    public B (Contract inj) {
        this.myContract = inj;
    }

    public void call() {
        this.myContract.call();
    }
}
```

在這個結構中，A 類別與 B 類別同時依賴 **Contract** 這個抽象介面。

Class B 不再關心 A 的具體實作，

只要 Contract 這個「抽象」保持不變，B 就能穩定運作。

除非有跳脫抽象意義的實作(違反 LSP)，

這就是依賴反轉。

## 結論

DIP 關注不僅只有依賴強度，更重要的是：

> 軟體是否依賴於不穩定的細節？還是依賴於穩定的抽象？

### 思考

#### 抽象是什麼?

- 對我而言，「抽象」是軟體工程裡「說好的規範與合約」。

#### 維持軟體穩定的「抽象」，只有在只有在程式碼層級中嗎?

## 參考資料

- Clean Architecture - Robert C. Martin
