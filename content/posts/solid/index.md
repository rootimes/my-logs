+++
title = 'SOLID 介紹'
date = 2024-10-16T23:42:03+08:00
draft = true
categories = [
    "coding-style",
]
tags = [
    "basic",
]
description = "SOLID 介紹"
+++

## 介紹
源自於 [Robert Cecil Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) 等人的倡議，主要有五個核心原則

## 舉例

### Single-responsibility principle (SRP) 單一職責原則

定義：一個模組只有一個理由會使其改變
A class should have only one reason to change

### Open-closed principle (OCP) 開放封閉原則

定義：一個軟體製品在面對擴展時是開放的，且擴充時不應修改到原有的程式
You should be able to extend the behavior of a system without having to modify that system.

### Liskov substitution principle (LSP) 里氏替換原則

定義：若對型態 S 的每一個物件 o1，都存在一個型態為 T 的物件 o2，使得在所有針對 T 編寫的程式 P 中，用 o1 替換 o2後，程式 P 的行為功能不變，則 S 是 T 的子型態。
簡化：繼承項，須按照父項行為目的設計

### Interface segregation principle (ISP) 介面隔離原則

定義：避免強制性的依賴在未使用的方法上
No client should be forced to depend on methods it does not use.

### Dependency inversion principle (DIP) 依賴反向原則

定義：高層模組不應依賴低層模組，它們都應依賴於抽象介面

## 參考資料
[使人瘋狂的 SOLID 原則：單一職責原則 (Single Responsibility Principle)](https://medium.com/程式愛好者/使人瘋狂的-solid-原則-單一職責原則-single-responsibility-principle-c2c4bd9b4e79)
[使人瘋狂的 SOLID 原則：開放封閉原則 (Open-Closed Principle)](https://medium.com/程式愛好者/使人瘋狂的-solid-原則-開放封閉原則-open-closed-principle-f7eaf921eb9c)
[使人瘋狂的 SOLID 原則：里氏替換原則 (Liskov Substitution Principle)](https://medium.com/程式愛好者/使人瘋狂的-solid-原則-里氏替換原則-liskov-substitution-principle-e66659344aed)
[使人瘋狂的 SOLID 原則：介面隔離原則 (Interface Segregation Principle)](https://medium.com/程式愛好者/使人瘋狂的-solid-原則-介面隔離原則-interface-segregation-principle-50f54473c79e)
[使人瘋狂的 SOLID 原則：依賴反向原則 (Dependency Inversion Principle)](https://medium.com/程式愛好者/使人瘋狂的-solid-原則-依賴反向原則-dependency-inversion-principle-a74ca045d776)

## 免責聲明

以上內容算是個人的筆記，可能有誤或較多引用，期望能拋磚引玉

歡迎大家在底下補充與討論