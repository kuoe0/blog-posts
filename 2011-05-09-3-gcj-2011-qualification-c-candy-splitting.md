---
layout: post
title:  "[GCJ] 2011 Qualification - C - Candy Splitting"
date:   2011-05-09
tags:   ["Google Code Jam", "bitwise operation | 位元運算"]
---

題目網址：[C - Candy Splitting](http://code.google.com/codejam/contest/dashboard?c=975485#s=p2)

## 題目概述

Sean 與 Patrick 要分糖果，糖果上都有一個整數。現在要將糖果分為兩堆，而且每一堆數字的加總後和要相同。若不相同，Patrick 會哭哭，所以 Patrick 一定會去檢查和有沒有相同。

但 Patrick 的加法不好，而且他不會十進位加法，只會二進為加法，重點是還不會進位。而 Sean 的算術很強，所以他可以利用 Patrick 的弱勢，偷偷讓自己這堆糖果的數字和大於 Patrick 的，而且讓 Patrick 還傻傻地以為兩堆的數字和一樣。

Patrick 加法範例：

	  1100 (12)
	+ 0101 ( 5)
	-----------
	  1001 ( 9)

判斷是否可能存在使 Patrick 不會哭的分法，若存在，Sean 最多可以拿到多少。

### Technique Detail

**small case**

- 測試資料的數量 **T**, 1 ≤ T ≤ 100
- 糖果編號 **C<sub>i</sub>**, 1 ≤ C<sub>i</sub> ≤ 1,000,000
- 糖果數量 **C**, 2 ≤ N ≤ 15

**large case**

- 測試資料的數量 **T**, 1 ≤ T ≤ 100
- 糖果編號 **C<sub>i</sub>**, 1 ≤ C<sub>i</sub> ≤ 1,000,000
- 糖果數量 **C**, 2 ≤ N ≤ 1,000

### 輸入格式

每一筆測試資料由一個整數 T 開始，表示接下來有 T 組測試資料。

每一組測試資料由一個整數 N 開始，表示有 N 個糖果。接下來有 N 個整數，這些整數代表糖果上的數字。

### 輸出格式

對於每一筆測試資料，輸出只有一行，輸出測試資料編號與一個整數 `Case #X: Y`。X 為測試資料編號，由 1 開始；Y 為 Sean 能取得的最大值，若不存在解則為 `NO`。

---

## 解題思路

剛看題目的加法還以為很難，再仔細想一下發現根本就是xor (exclusive-or) 的運算方式。接著很明顯的，若存在某個 bit 的數量是奇數，那麼答案肯定是 `NO`。

既然知道題目算術方法是 xor，又知道有奇數個 bit 無法分堆，那麼很明顯的只要把所有糖果上的數字 xor 起來，若是不為 0，表示不可分堆，就直接輸出 `NO` 吧。

接著，既然答案不是 `NO`，那就表示可以分割了。接下來也很簡單，就盡可能地找出一個最小（最大）的堆。因此只要對數字排序後，慢慢地將最小的數字從整個堆中分開，直到找到一個分堆方法可使兩個堆 xor 後的值都相同。答案即為較大的那一堆的數字和。

> Time Complexity: O(N)

### Source Code

<script src="https://gist.github.com/KuoE0/1619671.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/1619671).
