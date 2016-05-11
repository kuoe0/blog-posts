---
layout: post
title:  "[GCJ 2013] Qualification Round - A - Tic-Tac-Toe-Tomek"
date:   2013-04-14
tags:   ["Google Code Jam"]
feature:
    photo: false
---

題目網址：[A - Tic-Tac-Toe-Tomek](https://code.google.com/codejam/contest/2270488/dashboard#s=p0)

## 題目概述

給定一個圈圈叉叉的遊戲狀態，判斷給定的狀態屬於下列哪一種：

- X 獲勝
- O 獲勝
- 平手
- 遊戲尚未結束

題目所給的圈圈叉叉跟我們以往玩的有些許不同，它是由 4 × 4 的方格組成。而勝利條件除了橫向直向對角線都存在相同元素外，也可以由 3 個相同元素加上一個 `T` 來獲得勝利。

### Technique Detail

small case:

- 1 ≤ T ≤ 10

large case:

- 1 ≤ T ≤ 1000

### 輸入格式

測試資料由一個整數 T 開始，表示接下來會有 T 比測試資料。而每筆測試資料都包含四行且每行有四個字元，即為圈圈叉叉的狀態。`X` 表示當前格子放置了 X，`O` 則表示放置了 O，`T` 則表示放置了 T，而 `.` 則表示當前格子為空白。另外，在每一筆測試資料後都會有一個空行。

### 輸出格式

輸出給定的圈圈叉叉的狀態為何，如下：

- X won
- O won
- Draw
- Game has not completed

---

## 解題思路

很明顯的這題就是 Qualification Round 的水題，做法非常容易。先是直接判斷 X 跟 O 有沒有人贏，檢查橫向直向與對角線。若有人勝利則輸出該勝利者，沒有的話在判斷是不是還有空格。如果還存在空格，那麼表示「遊戲尚未結束」，如果不存在空格了，那就輸出「平手」。

> Time Complexity: O(1)

### Source Code

<script src="https://gist.github.com/KuoE0/5381830.js"></script>

Source code on [gist](https://gist.github.com/5381830).
