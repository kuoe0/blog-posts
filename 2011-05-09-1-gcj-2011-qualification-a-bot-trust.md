---
layout:  post
title:   "[GCJ] 2011 Qualification - A - Bot Trust"
date:    2011-05-09
tags:    ["Google Code Jam", "simulation | 模擬"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

題目網址：[A - Bot Trust](ttp://code.google.com/codejam/contest/dashboard?c=975485#s=p0)

## 題目概述

有兩個機器人 Orange 跟 Blue，他們各自被困在一個長長的走廊中。走廊的牆上有 100 個按鈕，按鈕上分別標著 1 ~ 100，按鈕 k 總是距離走廊的起點 k 公尺。每一秒，機器人可以往某方向前進 1 公尺，或是留在原地。留在原地時，可以選擇按下該位置之按鈕或是不做任何動作。

現在給定這兩機器人需要按下按鈕的時間點清單，兩個機器人最初都位在按鈕 1 的位置。求出最少需要多少時間可以完成此清單之要求。

### Technique Detail

**small case**

- 測試資料的數量 **T**, 1 ≤ T ≤ 20
- 需要被按下的按鈕數量 **N**, 1 ≤ N ≤ 10

**large case**

- 測試資料的數量 **T**, 1 ≤ T ≤ 20
- 需要被按下的按鈕數量 **N**, 1 ≤ N ≤ 10

### 輸入格式

每一筆測試資料由一個整數 T 開始，表示接下來有 T 組測試資料。

每一組測試資料由一個整數 N 開始，即為需要被按下的按鈕數量。接下來會有 N 組含兩個整數的按鈕資料 R<sub>i</sub>, P<sub>i</sub>。R<sub>i</sub> 表機器人的顏色，以 O 表示 Orange；B 表示 Blue。而 P<sub>i</sub> 為按鈕編號。

### 輸出格式

對於每一筆測試資料，輸出只有一行，輸出測試資料編號與一個整數 `Case #X: Y`。X 為測試資料編號，由 1 開始；Y 為最少所需要的時間。

---

## 解題思路

在給定清單時，依照 O 與 B 存在不同的 queue 上，儲存時也記錄此對資料的時間點。接著開始模擬，看 O 與 B 誰的時間點較早，就先處理。

假設 O 按下按鈕的時間較早，我們就先處理 O，也就是直接把 O 移動到他所要按的按鈕位置，並按下按鈕（時間 + 1），再將花去的時間加上。此時，B 也須做處理，B 要盡可能的移動到 B 下一個所要按的按鈕的位置，但不可按下按鈕，反之亦然。

另一種情況是，O 或 B 已沒有需要按的按鈕了，此時就處理另一個還有按鈕清單的機器人即可。直到兩方都沒有按鈕需要按時，程式終止，輸出答案即可。

時間複雜度為 O(N)，對每次按下按鈕處理一次即可。

> Time Complexity: O( N )

### Source Code

<script src="https://gist.github.com/KuoE0/1619523.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/1619523).
