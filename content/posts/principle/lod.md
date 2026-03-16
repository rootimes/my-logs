+++
title = '迪米特法則(Law of Demeter, LoD)'
date = 2026-03-16T15:25:10+08:00
draft = true
tags = [
    "code principle",
]
description = "迪米特法則(Law of Demeter, LoD)是一項物件導向設計原則，強調物件應只與其直接相關的「鄰近」物件互動，以減少物件之間的耦合並提升系統的可維護性。"
+++

## 介紹

迪米特法則(Law of Demeter, LoD)是一項物件導向設計原則，強調物件應只與其直接相關的「鄰近」物件互動，以減少物件之間的耦合並提升系統的可維護性。

此概念由 Karl Lieberherr 提出，並源自其研究專案「Demeter Project」因此得名。

在後續研究中，Law of Demeter 被進一步拓展為「Law of Demeter for Concerns」並提出兩項實作的優點：

1. 資訊隱藏(Information Hiding)，可透過結構隱匿(Structure-shyness)或更一般化的關注點隱匿(Concern-shyness)等技術達成。

2. 降低軟體開發者需要處理的資訊量，減少資訊過載(Information Overload)。

因此最少知識原則(Principle of Least Knowledge)也常被用來描述這項原則。

## 迪米特法則(Law Of Demeter)

### Law Of Demeter

格言

> Only talk to your friends
>
> 只與你的朋友交談

在此原則中，物件的「朋友」通常指的是：

- 物件自身(self / this)
- 物件的成員變數(attributes)
- 物件的方法參數(parameters)
- 物件所創建的其他物件(objects created by the object)

也就是說，一個物件的方法應只與上述物件互動，而不應透過物件鏈結存取更深層的物件。

### Law Of Demeter For Concerns

格言

> Only talk to your friends who share your concerns
>
> 只與具有相同關注點(concerns)的朋友交談

此概念是延伸版本，強調物件之間的互動應限制在具有相同關注點(concern)的模組或元件之間，以進一步降低耦合並提升系統的模組化程度。

### 程式舉例：火車鏈式(Train Wreck / Message Chain)

```java

// 違反 LoD 的例子：物件直接存取其他物件的內部結構

public class A {
    private B b;

    public void exec() {
        b.getC().getD().exec();
    }
}
```

上述程式碼違反了 Law of Demeter。
物件 **A** 透過 **B** 取得 **C**，再透過 **C** 取得 **D** 並呼叫其方法，形成了多層的鏈式呼叫(message chain)。

這代表 **A** 不僅知道 **B** 的存在，也知道 **B** 內部包含 **C**，以及 **C** 內部包含 **D**，使得物件之間的耦合度提高。

可以透過 **委派 delegation** 的方式進行修改：

```java
public class A {
    private B b;

    public void exec() {
        b.exec();
    }
}

public class B {
    private C c;

    public void exec() {
        c.exec();
    }
}

public class C {
    private D d;

    public void exec() {
        d.exec();
    }
}

public class D {
    public void exec() {
    }
}
```

在這個版本中，每個物件只與自己的成員物件互動，並將行為逐層委派下去。

如此一來，每個類別只需要了解其直接關聯的物件，而不需要知道更深層的物件結構，這樣就符合了 Law of Demeter 的原則。

### 此鏈非彼鏈

在上述的例子中，**A** 直接存取 **B** 的內部結構，形成了「火車鏈式(Train Wreck / Message Chain)」的反模式。

然而，並非所有「鏈式呼叫」都違反 LoD。以下幾種常見設計雖然也使用鏈式呼叫，但概念不同。

- Fluent Interface 透過方法鏈結（method chaining）設計 API，使程式碼具有良好的可讀性。
- 每個方法通常回傳物件本身，使呼叫可以連續進行，而呼叫者只需要與該物件互動，不需要了解其內部結構。

```java
public class Main {
    public static void main(String[] args) {
        User user = new User();

        user.setName("Rootimes")
            .setEmail("rootimes8596@gmail.com")
            .setMotto("知道的越多，不知道的越多")
            .output();
    }
}
```

- Builder Pattern（建造者模式）用於封裝複雜物件的建構過程，並透過鏈式方法逐步設定物件屬。
- 呼叫者只與 UserBuilder 互動，而不需要了解 User 的內部建構細節。

```java
public class Main {
    public static void main(String[] args) {
        User user = new UserBuilder()
            .setName("Rootimes")
            .setEmail("rootimes8596@gmail.com")
            .setMotto("知道的越多，不知道的越多")
            .build();
    }
}
```

- Lambda Expression 或 Stream API：在某些語言中，使用 lambda 表達式或匿名函數來處理集合或流式操作。
- 呼叫者只與 Stream 介面互動，而不是存取多層物件結構。

```java
List<String> names = Arrays.asList("Lucas", "Rootimes", "遇如");

public class Main {
    public static void main(String[] args) {
        names.stream()
            .filter(name -> name.startsWith("R"))
            .map(String::toUpperCase)
            .forEach(System.out::println);
    }
}
```

## 結論

迪米特法則，讓每個物件的知識範圍保持在最小，關注的是

> 物件之間互動的直接性和局部性。

## 思考

我們還可以進一步思考一些問題：

1. 貧血模型（Anemic Domain Model）是不是違反了 LoD？
2. 嚴格遵守 LoD 是否會導致什麼問題?如果物件關聯結構與方法穩定呢?

## 參考資料

[Law of Demeter: Principle of Least Knowledge](https://www.ccs.neu.edu/home/lieber/LoD.html)
