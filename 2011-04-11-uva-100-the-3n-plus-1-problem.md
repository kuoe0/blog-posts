---
layout: post
title:  "[UVa] 100 - The 3n + 1 problem"
date:   2011-04-11
tags:   ["programming | 程式編寫", "UVa, simulation | 模擬"]
---

題目網址：[100 - The 3n + 1 problem](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=3&page=show_problem&problem=36)

## 題目概述

題目給了一段流程：

1. input n
2. print n
3. if n = 1 then STOP
4. if n is odd then n ← 3n+1
5. else n ← n/2
6. GOTO 2

給定一個數字 **n**，都會按照這流程執行直到結束。每次執行這段流程後，會產生一個新的數字。並最後會回到 1，且演算法結束。這個過程會產生一段數列。

此題就是給定一個範圍，並輸出這範圍內（含兩端）的所有數字經過這段流程後，最長的數列長度。

### Technique Detail

- 0 < n < 1,000,000
- 在產生的數列中，不會有數字超過 32-bit 整數之大小

### 輸入格式

每筆測試資料一行，包含了兩個數字 `A B`，並用空格隔開。

### 輸出格式

對於每一筆測試資料，輸出三個整數 `A B X`。A 與 B 同 input 所給之數字。X 即為 [A,B] 中所有數字經過以上的演算法，所產生的數列最長之長度，一樣以空格隔開。

---

## 解題思路

依照題目之意思直接模擬即可求出正確答案，基本上這個題目大概是所有 ACMer 所接觸的第一題吧！所以不會太刁鑽 :)。

比較值得注意的地方是，其實在走上述之流程時，所產生的數字可能會超過 32-bit 整數可容納之大小。但在 UVa 有特別提到不會超過，所以不需要注意到 overflow 的問題。但在其他 judge 或是比賽時（常常背當做練習題）記得考慮這點，我通常都會直接用 long long int 來寫。

另外還有 input 所給的數字並非一定 A < B。在出現 **B < A 的情況**時，迴圈的部份記得交換，output 的格式也要記得換回來。

### Source Code

<script src="https://gist.github.com/KuoE0/1595194.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/1595194).
