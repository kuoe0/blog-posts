---
layout: post
title:  "質數表 - 埃拉托斯特尼篩法 (sieve)"
date:   2009-10-23
tags:   ["prime | 質數", "number theory | 數論", "algorithm | 演算法"]
feature:
    photo: false
---

Sieve of Eratosthenes ，一般稱作「篩法」，一個用來建立質數表的演算法。由於任何[合數](http://zh.wikipedia.org/wiki/%E5%90%88%E6%95%B0)都可以拆解乘許多的質數相乘，篩法的精髓就在於利用質數將其所有倍數消去。由於篩法用於建立質數表，因此僅能建立有限大小的質數表，對於過大的質數及無計可施。

## Algorithm

1. 列出所有整數（在此假設 1 ~ N），並假設所有的整數都是質數（除了 1）
2. 從 2 開始找尋質數，並將該數除了自己以外，所有小於 N 的倍數標記為合數
3. 重複第 2 步驟的依序找尋質數，並將倍數標記為合數，最後所有合數都將被標記

> 時間複雜度：O(NloglogN)
> 空間複雜度：O(N)

## Improvement

- 先將所有 2 的倍數標記為合數，如此一來，在第 2 步驟中，就可僅收尋奇數
- 利用定理「 **x 不能夠被不大於根號 x 的任何質數整除，則 x 是一個質數**」，在第 2 步驟中可測試到根號 N 即可
- 除了 2 與 3 以外的質數，皆可用 6k±1 的表示，在第 2 步驟中可僅測試 6k±1 的數字（**注意先將 2 與 3 的所有倍數標記為合數**）

**Pseudo Code**

```
initial 1 to N is prime
set 1 is not prime
for i = 2 to sqrt(N)
    if i is prime
        for i * j <= N, j >= 2
            set i * j is not prime
```

**C++ Implementation**

<script src="https://gist.github.com/1594929.js?file=sieve.cpp"></script>

Source code on [gist](https://gist.github.com/KuoE0/1594929#file-sieve-cpp).
