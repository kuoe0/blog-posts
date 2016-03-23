---
layout:  post
title:   "[UVa] 705 - Slash Maze"
date:    2011-05-14
tags:    ["UVa", "DFS | 深度優先搜索", "graph theory | 圖論"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

題目網址：[705 - Slash Maze](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=9&page=show_problem&problem=646)

## 題目概述

給定義一張地圖，地圖由 / 與 \\ 組成，求出此地圖有幾個cycle，最大的 cycle 長度為多少。

### Technique Detail

- 地圖寬 **W**, 1 ≤ W ≤ 75
- 地圖高 **H**, 1 ≤ H ≤ 75

### 輸入格式

每筆測試資料由兩個整數開始 W, H，表示地圖的寬 W 與高 H。接下來有 H 列，每列有 W 個字元，字元皆由 / 與 \\ 組成。

### 輸出格式

對於每一筆測試資料，首先輸出該測試資料的編號，編號由 1 開始，格式為 `Maze #t`，t 為編號。若存在 cycle，則輸出`N Cycles; the longest has length L.`，N 為 cycle 數量，L 為最常 cycle 的長度。若不存在任何 cycle，則只要輸出 `There are no cycles.`。

每筆測試資料最後加上一個空行。

---

## 解題思路

對於每一個符號，我們將它轉變為 2 × 2 由 0 與 1 組成的格子。其中 0 表示有路可走，1 表示無路可走。

示意：

	/ => 01
	     10
	-------
	\ => 10
	     01

將地圖轉換完成後，再以DFS搜索即可找出cycle數量。在走斜的方向時，須注意所前進的方向其兩邊的 1 在原地圖中是否為相連的。

> Time Complexity: O(W × H)

### Source Code

<script src="https://gist.github.com/KuoE0/1619732.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/1619732).
