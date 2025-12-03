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

### 2435. Paths in Matrix Whose Sum Is Divisible by K

- 目標 : 找出從左上角到右下角的路徑數量，使得路徑上數字總和能被 k 整除

- 關鍵 1 : 使用動態規劃 dp[row][col][mod] 來記錄到達 (row, col) 時，數字總和對 k 取餘數為 mod 的路徑數量

- 關鍵 2 : 初始化起點 dp[0][0][grid[0][0] % k] = 1

- 關鍵 3 : 使用第一個 row 或 col 先算，因為只能從一個方向來

- 關鍵 4 : 狀態轉移方程式 : dp[row][col][(mod + grid[row][col]) % k] += dp[row - 1][col][mod] + dp[row][col - 1][mod]

- 關鍵 5 : 最終答案為 dp[m - 1][n - 1][0]，表示到達右下角且數字總和能被 k 整除的路徑數量

### 3381. Maximum SubArray Sum With Length Divisible by K

- 目標 : 找出子陣列長度能被 k 整除的最大子陣列總合

- 關鍵 1 : 使用前綴和與餘數的概念來解決問題

- 關鍵 2 : 只要 preFixSum[index] 兩個位置的 index 對 k 取模後一樣，它們之間的子陣列長度就是 k 的倍數

### 2872. Maximum Number of K-Divisible Components

- 目標 : 將樹分割成 k 個可連通的元件，找出最大可分割的元件數量

- 關鍵 1 : 使用深度優先搜尋 (DFS) 來遍歷樹

- 關鍵 2 : 在 DFS 過程中，計算每個子樹的節點數量，若子樹節點數量能被 k 整除，則該子樹可以成為一個獨立的元件

### 3512. Minimum Operations to Make Array Sum Divisible by K

- 目標 : 找出最少的操作次數，使得陣列總和能被 k 整除

### 1590. Make Sum Divisible by P

- 目標 : 找出最短的子陣列，使得移除該子陣列後，剩餘陣列總和能被 p 整除

- 關鍵 1 : 計算整個陣列的總和對 p 取餘數，記為 target

- 關鍵 2 : 使用前綴來計算每個位置的前綴和對 p 取餘數，並記錄每個餘數出現的位置

### 2141. Maximum Running Time of N Computers

- 目標 : 找出 n 台電腦能持續運行的最長時間，給定一組電池容量

- 關鍵 1 : 使用二分搜尋來找出最大可行的運行時間

### 3623. Count Number of Trapezoids I

- 目標 : 計算能形成梯形的組合數量

- 關鍵 1 : 統計每個 y 的點數量

- 關鍵 2 : y 上有 n 個點，能形成水平線數量為 n * (n - 1) / 2

- 關鍵 3 : 使用前綴累積法來計算梯形數量

### 3625. Count Number of Trapezoids II

- 目標 : 計算能形成梯形的組合數量

- 關鍵 1 : 先算出所有水平線組合數量

- 關鍵 2 : 減去不符合梯形條件的組合數量

## 免責聲明

以上內容算是個人的筆記，可能有誤或較多引用，期望能拋磚引玉

歡迎大家在底下補充與討論