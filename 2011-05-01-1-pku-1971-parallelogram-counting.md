---
layout: post
title:  "[POJ] 1971 - Parallelogram Counting"
date:   2011-05-01
tags:   ["POJ", "sort | 排序", "computational geometry | 計算幾何"]
---

題目網址：[1971 - Parallelogram Counting](http://poj.org/problem?id=1971)

## 題目概述

給定二維平面上 **N** 個點，求出有幾種組合可以形成平行四邊形。

### Technique Detail

- 點的數量 **N**, 1 ≤ N ≤ 1,000

### 輸入格式

測試資料由一個整數 K 開始，表示接下來有 K 組測試資料。

每一筆測試資料由一個整數 N 開始，表示平面上有 N 個點。接下來有 N 行，每行兩個整數 x<sub>i</sub> y<sub>i</sub> ，表示該點座標。

### 輸出格式

對於每一筆測試資料，輸出只有一行，且只含有一個整數，即為可以形成平行四邊形的組合數量。

---

## 解題思路

枚舉兩個點，並記錄所有中點。若有重複的中點，即可從這些點中，任取兩個出來組成一個平行四邊形。平行四邊形的兩對角線中點會重合，故此方法可行。對所有中點排序後，即可快速找出重複的中點。

中點的數量會有 C(N, 2) 個，大約為 N<sup>2</sup> 個。因此排序的時間為 O( N<sup>2</sup> × log N<sup>2</sup>)

> Time Complexity: O( N<sup>2</sup> × log N<sup>2</sup>)

### Source Code

<script src="https://gist.github.com/KuoE0/1612017.js"></script>

Source code on [gist](https://gist.github.com/1612017).
