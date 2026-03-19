+++
title = 'SOLID：里氏替換原則 (LSP)'
date = 2026-03-02T23:42:03+08:00
draft = false
tags = [
    "basic",
]
description = "里氏替換原則 (LSP) 強調子類別應該可以替換其父類別而不影響程式的正確性。本文探討 LSP 的核心定義與實作細節"
+++

## 介紹

SOLID 是由 [Robert Cecil Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) 等人倡議的五項核心原則。

它們並非刻板的硬性規定，而更像是一種**精神指引**，幫助開發者建立易於維護、具備彈性的系統：

1. **SRP：單一職責原則 (Single Responsibility Principle)**
2. **OCP：開放封閉原則 (Open-Closed Principle)**
3. **LSP：里氏替換原則 (Liskov Substitution Principle)**
4. **ISP：介面隔離原則 (Interface Segregation Principle)**
5. **DIP：依賴反轉原則 (Dependency Inversion Principle)**

這篇將聚焦於第三個原則：**里氏替換原則 (LSP)**。

這項原則的存在感可能不如 SRP 和 OCP 那麼強烈，甚至在某些情況下會被忽略。

但它卻是物件導向設計中非常重要的一環，因為它直接關係到繼承和多型的正確使用。

## Liskov Substitution Principle (LSP) 里氏替換原則

> **若對型態 S 的每一個物件 o1，都存在一個型態為 T 的物件 o2，使得在所有針對 T 撰寫的程式 P 中，用 o1 替換 o2 後，程式 P 的行為功能不變，則 S 是 T 的子型態。**
> (If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2, then S is a subtype of T.)

LSP 是「針對抽象寫程式」這項概念的具體說明，確保程式在使用繼承和多型時能保持正確性和一致性。

### 程式舉例：違反 LSP 的例子

```java

// 這是一個違反 LSP 的例子

public class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // 正方形強制寬高一致
    }

    @Override
    public void setHeight(int height) {
        this.height = height;
        this.width = height; // 正方形強制寬高一致
    }
}

public void testArea() {
    Rectangle rect = new Square();
    rect.setWidth(5);
    rect.setHeight(10);

    // 呼叫端預期的行為是面積為 50 (5 * 10)
    // 但實際上會得到 100，因為 Square 的 setHeight 覆寫了 Rectangle 的預期行為，連帶改變了 width
    assert rect.getArea() == 50; // 這裡會失敗
}

```

上述是一個經典的 LSP 違反例子。**Square** 類別繼承自 **Rectangle**，

但其 **setWidth** 和 **setHeight** 方法的行為違反了外界對 **Rectangle** 行為的預期。

**語意背叛**
在開發套件（Library）時，LSP 的違反往往更隱晦。例如：

**父類別契約**：定義一個測量方法，預期回傳攝氏溫度。

**子類別實作**：卻因為內部邏輯或硬體差異，回傳了華氏溫度。

雖然型別對了，但抽象的意義改變了。

更好的做法：提取更純粹的抽象（抽離不適合的行為，改為不可變或建立共通介面）：

```java
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private int width;
    private int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public int getArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private int side;

    public Square(int side) {
        this.side = side;
    }

    @Override
    public int getArea() {
        return side * side;
    }
}

```

## 型態(type) 與介面的約定

常有人的誤解是，LSP 只生效在「類別繼承 (Class Inheritance)」的關係中。

但如果我們回頭看 LSP 的基本定義，它強調的是「子型態 (Subtype)」。

這意味著，即使是針對 Interface（介面）的實作，為了維持系統穩定，我們仍然必須遵守 LSP 的要求。

```java
public interface Shape {
    void setWidth(int width);
    void setHeight(int height);
    int getArea();
}

public class Rectangle implements Shape {
    private int width;
    private int height;

    @Override
    public void setWidth(int width) {
        this.width = width;
    }

    @Override
    public void setHeight(int height) {
        this.height = height;
    }

    @Override
    public int getArea(){
        return this.height * this.width;
    }
}

public class Square implements Shape {
    private int side;

    @Override
    public void setWidth(int width) {
        this.side = width;
    }

    @Override
    public void setHeight(int height) {
        this.side = height; // 強制連動
    }

    @Override
    public int getArea() {
        return this.side * this.side;
    }
}

// 測試介面的合約
public void testShapeArea(Shape shape) {
    shape.setWidth(5);
    shape.setHeight(10);
    
    // 根據 Shape 介面的隱含合約，寬與高應該是獨立設置的
    if (shape.getArea() != 50) {
        System.out.println("違反 LSP！預期 50，實際得到: " + shape.getArea());
    } else {
        System.out.println("符合預期。");
    }
}
```

即便只是實作同一個 **Shape** 介面，**Square** 的內部邏輯依然破壞了呼叫端對於「設定寬不影響高」的合理預期。

## 在抽象上的程式

軟體開發中有一項重要的概念：

> 針對**抽象**寫程式，而非針對實作寫程式
> (Programming to an abstraction, not an implementation)

堅守 LSP 能帶來以下好處：

1. 行為預測性：呼叫者不需要查看子類別原始碼，也能確定行為。

2. 多型穩定性：可以放心地更換不同的實作（感測器、資料庫、圖形元件）而不需要修改主邏輯。

3. 解耦：模組之間透過「契約」對話，而非「細節」對話。

## 結論

LSP 關注的重點不在於語法上的繼承與否，而是：

> 我們是否有確實遵守並實現基礎型態的「行為約定 (Design by Contract)」

## 參考

- Clean Architecture - Robert C. Martin
- [Behavioral Subtyping Using Invariants and Constraints](http://reports-archive.adm.cs.cmu.edu/anon/1999/CMU-CS-99-156.pdf)
