+++
title = '關注點分離 (Separation of Concerns, SoC)'
date = 2026-03-13T16:02:06+08:00
draft = false
tags = [
    "code principle",
]
description = "關注點分離 (Separation of Concerns, SoC) 是一個軟體設計原則，強調在軟體開發過程中，應該將不同的關注點分離開來，以便更有效地組織思維和解決問題。這種方法可以幫助我們更專注於當前的問題，並且在不同的時間點專注於不同的方面，以達到更好的效果。"
+++

## 介紹

關注點分離（Separation of Concerns, SoC）是一個重要的軟體設計原則，由 Edsger W. Dijkstra 在 1974 年提出。

在他的文章 **EWD447 – On the role of scientific thought** 中，Dijkstra 提出了「separation of concerns」這個概念，用來描述一種有效的思考方式：  
在研究複雜問題時，一次只專注於其中的一個切面（aspect）。

原文：

```text
"Let me try to explain to you, what to my taste is characteristic for all intelligent thinking. It is, that one is willing to study in depth an aspect of one's subject matter in isolation for the sake of its own consistency, all the time knowing that one is occupying oneself only with one of the aspects. We know that a program must be correct and we can study it from that viewpoint only; we also know that it should be efficient and we can study its efficiency on another day, so to speak. In another mood we may ask ourselves whether, and if so: why, the program is desirable. But nothing is gained —on the contrary!— by tackling these various aspects simultaneously. It is what I sometimes have called "the separation of concerns", which, even if not perfectly possible, is yet the only available technique for effective ordering of one's thoughts, that I know of. This is what I mean by "focussing one's attention upon some aspect": it does not mean ignoring the other aspects, it is just doing justice to the fact that from this aspect's point of view, the other is irrelevant. It is being one- and multiple-track minded simultaneously.
```

"The separation of concerns" 在原文當中的關注點分離，在於專注於某個特定的方面，並且將其他方面暫時忽略掉，以便更深入地研究該方面的內容。這種方法可以幫助我們更有效地組織思維，並且在不同的時間點專注於不同的方面，以達到更好的效果。

## 關注點分離(Separation Of Concerns)

關注點分離，在軟體工程領域中帶來了諸多影響：

- 建模：將系統分成不同的模型。

- 模組化：將程式碼分成不同的模組。

- 分層架構：將系統分成不同的層次。

- 微服務架構：將系統拆分成小型的服務。

- 任務拆分：將複雜的任務拆分成更小的子任務。

以上只是部分例子。在軟體開發過程中，許多設計方式其實都體現了關注點分離的思想。

其核心理念可以簡單整理為：

1. 一次只專注於一個切面（aspect）
2. 這並不意味著忽略其他切面，而是暫時將注意力集中在當前問題
3. 這是一種有效整理思考的方式

透過這種方式，我們可以逐步解決不同切面的問題，最終完成整體系統的設計與實作。

## SoC 與 SRP

關注點分離（SoC）與 單一職責原則（Single Responsibility Principle, SRP） 經常一起出現，但兩者其實並不完全相同。

簡單來說：

- SoC 更關注 **「問題切面（concern）」** 的拆分

- SRP 更關注 **「變動原因（reason to change）」** 的拆分

因此：

- SoC 偏向**思考方式與設計理念**

- SRP 偏向**程式設計層級的設計原則**

## 結論

關注點分離不僅是一種軟體設計原則，也是一種思考方式。

在軟體工程中，系統往往非常複雜，很難在同一時間處理所有問題。

SoC 提醒我們，在特定階段應該專注於某一個問題切面，而不是試圖同時解決所有問題。

透過將問題拆分並逐步處理，我們可以更有條理地理解系統，並最終完成整體設計。

### 思考

我們還可以進一步思考一些問題：

1. 如何決定關注的切面範圍?
2. 什麼時候切換到另一個切面?
3. 如何平衡不同切面，確保整體系統的一致性?

## 參考資料

- [EWD 447](https://www.cs.utexas.edu/~EWD/transcriptions/EWD04xx/EWD447.html)
