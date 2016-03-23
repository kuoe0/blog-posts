---
layout:  post
title:   "[GCJ 2013] Qualification Round - B - Lawnmower"
date:    2013-04-14
tags:    ["Google Code Jam"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

題目網址：[B - Lawnmower](https://code.google.com/codejam/contest/2270488/dashboard#s=p1)

## 題目概述

有一片 N × M 的草坪要修剪，現在有一台割草機，這台割草機可以設定要將草坪的草修剪到高度為 h，但這台割草機只會從草坪的一端直線走向草坪的另一端。題目會給定一個草坪的最後高度樣式，試問有沒有可能利用這台割草機來達到？

### Technique Detail

- 1 ≤ T ≤ 100

small case:

- 1 ≤ N, M ≤ 10
- 1 ≤ a<sub>i,j</sub> ≤ 2

large case:

- 1 ≤ N, M ≤ 100  
- 1 ≤ a<sub>i,j</sub> ≤ 100

### 輸入格式

測試資料由一個整數 T 開始，表示接下來會有 T 比測試資料。而每筆測試資料由兩個整數 N 與 M 開始，表示草坪的長與寬。接下來會有 N 列，每列會有 M 個數字 a<sub>i,j</sub>，表示草坪樣式的高度。

### 輸出格式

若可以利用該割草機達到該樣式，輸出 YES，否則輸出 NO。


---

## 解題思路

先要有一個概念，割草機的執行順序不重要，而且每一行或每一列都頂多需要割一次，每一格的高度都應該要是當前行與當前列所設定的高度之最小值。我是預設每一行每一列都要進行割草的動作，不過是設定的數字而已，要設定成不割的話就只要將割草機的高度數值設定大一點就好囉！

針對 small case 來說，起初我的想法是枚舉 N 列與 M 行上割草機設定的高度。由於 small case 的高度只有 1 跟 2 兩種，所以時間複雜度為 O( 2<sup>N+M</sup> )。對 small case 來說大概也才 2<sup>20</sup> 而已，大概 1 百萬，感覺四分鐘錯錯有餘了！不過我忘了最後的比較也要花上 O( N × M ) 的時間，時間複雜度應該要是 O( N × M × 2<sup>N+M</sup> )，這樣一來就破億了，四分鐘完全不夠啊！因此獲得一個 wrong try。

於是只好重新思考解法，每一格的高度都會是當前行與當前列所設定的高度之最小值。反過來想，那麼每一列（行）的設定高度應該要是當前列（行）的最大值，否則不可能會有該最大值的存在！所以就針對每一列（行），訪查該列（行）的最大值，就可以得到每一行與每一列的割草機所需要設定的高度了。

最後在針對每一格去做檢查，檢查每一格是否都是當前列與當前行所設定的高度之最小值，若出現不符合條件者，則表示該樣式不可能達到。

> Time Complexity: O(N × M)

### Source Code

<script src="https://gist.github.com/KuoE0/5381892.js"></script>

Source code on [gist](https://gist.github.com/5381892).

