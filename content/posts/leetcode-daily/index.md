+++
title = 'Leetcode Daily'
date = 2025-11-25T18:24:35+08:00
draft = true
categories = [
    "leetcode",
]
tags = [
    "key point",
]
description = "每天的 leetcode 解題暫時紀錄"
+++

## 主要內容

### 1015. Smallest Integer Divisible by K

- 目標 : 找出最小的正整數 n，使得 n 能被 k 整除，且 n 只包含數字 1

- 關鍵 1 : 2 和 5 無法整除只包含數字 1 的數字

- 關鍵 2 : 使用餘數循環的概念來找出答案，公式 : remainder = (remainder * 10 + 1) % k

- 關鍵 3 : 最多 k 次迴圈可以找到答案，若超過則無解

- 概念 : 鴿籠原理: 若有 k + 1 個物品放入 k 個盒子，至少有一個盒子會超過一個物品

## 免責聲明

以上內容算是個人的筆記，可能有誤或較多引用，期望能拋磚引玉

歡迎大家在底下補充與討論