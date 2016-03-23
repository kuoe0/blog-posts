---
layout:  post
title:   "[ZJ College] d151 - Break Up is Hard to Do"
date:    2011-10-12
tags:    ["ZeroJudge - College", "BFS | 廣度優先搜索", "DFS | 深度優先搜索", "enumerate | 枚舉"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

題目網址：[d151 - Break Up is Hard to Do](http://140.122.185.166/ZeroJudge/ShowProblem?problemid=d151)

## 題目概述

Alice 跟 Bob 是男女朋友，他們住在不同的城市。有一天他們吵架了，他們不想在見到對方。他們禮拜一到禮拜五都會去拜訪其他城市再回家，Alice 只會拜訪離他家小於等於五英哩的城市，若城市超過五英哩，則會留在家中。而 Bob 則沒有任何限制。

現在我們需要找出是否存在一個路徑規劃，使得 Alice 與 Bob 可以在沒有碰到面的情況下到其他城市。

### Technique Detail

- 城市數量 **N**，1 ≤ N ≤ 99
- 每一個城市都有**恰好兩個**外連的**單向道路**
- 行程的規劃中要**包括居住的城市**

### 輸入格式

每筆測試資料的第一行有三個數字 N, A, B，分別為城市數量 N、Alice 居住的城市 A 與 Bob 居住的城市 B。接下來有 N 行，每行兩個數字 city<sub>1</sub>, city<sub>2</sub>，表示第 i 個城市向外連接的道路所連接到的城市編號 city<sub>1</sub> 與 city<sub>2</sub>。最後一行有十個數字，每兩個一組，共五組，表示 Alice 與 Bob 禮拜一到禮拜五想要到的城市編號。

當測試資料僅包含一個 0 表示城市終止。

### 輸出格式

對於每一筆測試資料輸出一行，每一行由測試資料的編號開始**（由 1 開始）**，並接一個空白。接下來有五個由 Y 或 N 組成的字串。表示禮拜一到禮拜五的每一天是否存在一個可以使雙方見不到面的行程規劃。Y 表示存在該路徑，N 則表示不存在。

---

## 解題思路

這個題目看似複雜，但因為 Alice 不會走超過五英哩的路程，而大大簡化的題目的難度。

由於 Alice 不會走超過五英哩的路程，所以 Alice 可以走的路線最多只有 32 種。因此我們枚舉 Alice 的所有可能路線，並將經過的城市消去，再使用 BFS 來搜索是否存在一個路徑給 Bob，如此即可解決。

我認為比較需要注意的是當某人想去的城市剛好為某人的居住地，此時就直接輸出失敗。還有當沒有任何路徑存在時，記得將 Alice 留在家中，再用 BFS 替 Bob 找尋路徑。

另外，我用 STL 的 vector 在 zerojudge 都會編不過，有點詭異。最後自己手寫 adjacent list 才解決的。

### 小技巧

可以使用 DFS 枚舉 Alice 的路徑，當找到一條路徑時，就使用 BFS 來搜尋是否存在路徑給 Bob。如此一來，就不用將所有的路線儲存起來一一枚舉。

> Time Complexity: O(N)


### Source Code

<script src="https://gist.github.com/KuoE0/1447824.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/1447824).
