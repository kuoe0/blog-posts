---
layout:  post
title:   "[POJ] 1094 - Sorting It All Out"
date:    2011-05-04
tags:    ["POJ", "topological sort | 拓樸排序", "graph theory | 圖論"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

題目網址：[1094 - Sorting It All Out](http://poj.org/problem?id=1094)

## 題目概述

給定 **n** 個大寫字母的 **m** 組順序關係（使用 < 符號），並判定是否存在唯一的順序、存在矛盾或是無法判定。

例如：

- A < B，B < C ⇒ 存在唯一的順序ABC
- A < B，B < A ⇒ 存在矛盾
- A < B，B < C，A < D，D < C ⇒ 無法判定

### Technique Detail

- 字母數量 **n**, 1 ≤ n ≤ 26

### 輸入格式

每一筆測試資料由兩個整數 n, m 開始，表示要使用大寫字母的前 n 個，且有 m 組規則。接下來有 m 行，每行都是一組規則 letter<sub>1</sub> < letter<sub>2</sub>，表示 letter<sub>1</sub> 的順序在 letter<sub>2</sub> 前面。

當測試資料的 n 與 m 皆為 0 時，表示程式終止。

### 輸出格式

對於每一筆測試資料，輸出只有一行，根據結果有不同的輸出。

| 結果 | 輸出
| ---- | ---- |
| 存在唯一順序 | Sorted sequence determined after X relations: Y. |
| 無法判定 | Sorted sequence cannot be determined. |
| 存在矛盾 | Inconsistency found after X relations. |

上述之 X 為透過 X 條規則後可確定。而 Y 僅出現在存在唯一順序的部份，Y 即為這唯一的順序，將該順序輸出。

---

## 解題思路

這題很明顯就是拓樸排序，找出所給的關係圖之唯一拓樸排序。由於 n 很小，因此每次加入一個規則就做一次拓樸排序在時間上應相當足夠。

當找到 cycle 時就輸出產生矛盾的解，當找到解時也直接輸出找到的順序。若有上述兩者發生，對於後續的測資將讀取但不處理的動作，**測資一定要讀完**！

若讀完所有測資仍找不到解，在輸出非上述兩者的選項。若在第 x 個規則找到唯一順序，儘管第 x + k 個規則產生矛盾也不予理會，以最先找到的解為主。

### Source Code

<script src="https://gist.github.com/KuoE0/1610955.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/1616119).
