---
layout:  post
title:   "[UVa] 837 - Light and Transparencies"
date:    2011-05-01
tags:    ["UVa", "sweep line | 掃描線", "sort | 排序"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

題目網址：[837 - Light and Transparencies](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=10&page=show_problem&problem=778)

## 題目概述

在一個平面的上方有著許多的投影片，從側面看過去這些投影片為一條線段。光由上往下照射，投影片會希受部分能量，根據該投影片的穿透係數決定。當光線通過多片投影片時，穿透係數的大小為每一片投影片之穿透係數之乘積。求出光線最後抵達平面後，會被切割為幾個區段，以及各區段之能量大小！

在題目中，我們將平面設定為 x 軸。投影片即為這個二維平面上的線段，而光源來自 y = ∞ 的地方。


### 輸入格式

測試資料開頭會有一個整數 K，表示接下來會有 K 筆測試資料。接下來會有一個空行，此外每筆測試資料間也由空行隔開。

每筆測試資料由一整數 NL 開始，表示有 NL 片投影片。接下來有 NL 行，每行有五個浮點數 x<sub>1</sub> y<sub>1</sub> x<sub>2</sub> y<sub>2</sub> r。x<sub>1</sub> y<sub>1</sub> 為線段的一端點，而 x<sub>2</sub> y<sub>2</sub> 為線段的另一端點。最後一個浮點數 r 則為該投影片之穿透係數。

### 輸出格式

對於每筆測試資料，首先由一個整數（NP）開始，表示地面背切割為 NP 個區段。接下來有 NP 行，每一行有三個浮點數（x<sub>1</sub> x<sub>2</sub> r），除了第一個區段與最後一個區段其中會有 ±inf 來取代 x<sub>1</sub> 或 x<sub>2</sub>。其分別表示有一個區間從 x<sub>1</sub> 到 x<sub>2</sub> 且光的能量剩餘 r。

輸出的區段順序，依照座標來做排序。由於第一個區段會往無限小的方向延伸，因此 x<sub>1</sub> 會由 -inf 取代。而最後一個區段會往無限大延伸，因此其 x<sub>2</sub> 會由 +inf 取代。

另外，測試資料之間由一個空行隔開。

---

## 解題思路

由於給定的任兩個 x 座標均不會相同，因此光產生的區段數必定為 2 × NL + 1。至於要算剩下的能量，只需把所有點依照 x 座標排序，並記錄各點其投影片之穿透係數。

接著使用 sweep line 的概念，依排序後的順序走訪各點並計算，即可得到各段剩餘的能量。當掃描到投影片之起始點時，就將當前能量乘上該投影片之穿透係數。若掃描到投影片之終止點時，則將當前能量除以該投影片之穿透係數。能量初始值為 1。

時間複雜度為排序所需的時間 O( NL × log NL )。

> Time Complexity: O( NL × log NL )

### Source Code

<script src="https://gist.github.com/KuoE0/1611184.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/1611184).
