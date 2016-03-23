---
layout:  post
title:   "Traveling Salesman's Problem - DP solution"
date:    2012-10-11
tags:    ["algorithm | 演算法", "dynamic programming | 動態規劃", "graph theory | 圖論"]
feature:
    photo:       true
    creator:     "Jens Odegaard"
    url:         "https://www.flickr.com/photos/jensodegaard/3317293473"
    license:     "CC-BY-SA 2.0"
    license_url: "https://creativecommons.org/licenses/by-sa/2.0/"
---

NCPC 快到了，快兩年沒練題目，演算法也忘的差不多了。今天系內賽遇到 TSP 的題目一直 Wrong Answer，與同學討論時，因為自己已經忘了演算法細節，實在無法為自己的作法爭辯，決定再重新推敲該演算法。

**Traveling Salesman's Problem**，中文翻譯滿多種了，有**旅行商問題**、**推銷員問題**、**貨郎問題**等等，所以直接採用英文縮寫 **TSP** 稱之比較方便！該問題是圖論上的經典問題，目前已被證明為 [NP-Hard](http://en.wikipedia.org/wiki/NP-hard) 問題，有興趣請自行參閱相關資料。另外，TSP 有一特例為 [Hamiltonian Cycle](http://en.wikipedia.org/wiki/Hamiltonian_path)，當所有路徑長度都為 1 時，即為 [Hamiltonian Cycle](http://en.wikipedia.org/wiki/Hamiltonian_path) 問題。（感謝 [DJWS](http://www.csie.ntnu.edu.tw/~u91029/) 前輩的指正）

## 問題描述

某國家共有 N 個城市，有一位商人從自己居住的城市（Src）出發，想走遍**每個城市**做生意，各城市之間了距離不一，求**遍歷每個城市並回到居住城市的最短路程**。

## 解決方法

最直覺的方法是使用 backtracking 暴力搜索所有可能性，並求出最短的距離。雖然直覺又簡單，但時間複雜度也高達 O(N!)，10 個國家就可以另計算量達百萬等級了。在 ACM-ICPC 程式競賽中，若測試資料小於等於 10 或許還是有機會可以過關的！

不過這邊要介紹的是在 ACM-ICPC 程式競賽中更常用的方法－**[Dynamic Programming](http://www.csie.ntnu.edu.tw/~u91029/DynamicProgramming.html)**。

## Algorithm

動態規劃最重要的就是狀態的移轉！我們問題是要遍歷所有城市，並回到商人居住的城市。每一次的遍歷都會使得遍歷過的城市改變，也會使得當前的所在城市改變，可以觀察到有狀態的移轉！

所以在 TSP 中，所求的即為**遍歷所有城市**，且**當前城市為商人居住城市**，該狀態的結果。因此我們即可得到狀態轉換方程為：**TSP( 已遍歷城市, 當前所在城市 ）→ TSP( 原已遍歷城市＋上一個所待的城市, 下一個未遍歷城市 )**。

狀態的初始為**未遍歷任何城市**，且**當前城市為商人居住城市**，且行經距離為 0。不過這是理論上的作法，在實作上可能會有不同的考量而作不同得處理。在進行狀態的轉換中，根據轉換的城市須加上當前城市與下一個城市的距離，並求最小值即可。

以下為完整的計算與狀態轉換：

$$
TSP(0, Src) = 0
$$
$$
TSP(2^N - 1, Src) = min(TSP((2^N - 1) - (2^Src), X) + length(X, Src))
$$
$$
length(A, B) = the\ distance\ between\ A\ and\ B
$$


Data Structure
--------------

依照上述的轉換方程，可以計算出狀態的數量有 **2<sup>N</sup> × N** 種。城市的遍歷與否很明顯的不是已遍歷就是未遍歷兩種，因此 N 個城市就有 2<sup>N</sup> 種狀態。而當前所在城市就是城市的數量 N。因此可以使用二維作為該演算法的資料結構，城市的遍歷狀態與當前城市作為 index。

而城市的遍歷狀態該如何以數字表示？上面提到城市的遍歷與否僅會存在兩種狀態－已遍歷與未遍歷。因此只要將每個城市的遍歷與否以 bit 表示並串連，就可以得到遍歷狀態的數值。

> 0001011<sub>2</sub>（自右向左），該狀態表示為已遍歷 0、1 與 3 城市，而其餘城市仍未遍歷。

姑且宣告該陣列為 dp[2<sup>N</sup>][N]，該陣列意義即為 **dp[城市遍歷狀態][當前所在城市]**。TSP 所求的解即為 dp[11…111<sub>2</sub>][Src] 其值。

不過，我認為 dp[11…111<sub>2</sub>][X]，不管 X 其值為多少都不影響答案，因為 TSP 所求的為商人周遊各城市並回到居住城市的最短路線，必定是一整圈，既然是一整圈，那麼起點為何似乎不太重要？！

但是 TSP 可以做許多延伸，不同的延伸上，起點位置可能會變得很重要！例如：求 **Hamiltonian Path**，在不需要回到起點的情況下，起點位置就相當重要！

另外，該資料結構的空間複雜度為 O(2<sup>N</sup> × N)，由於是指數成長，N 到 20 就已經是千萬級的記憶體使用量了，因此該算法僅適用於 N 不大的狀況！

此外使用該資料結構時，會有些許的空間是浪費不會使用的。根據我們陣列的意義，可以發現當前所在城市必會出現於已遍歷城市中，因此 dp[X 城市未遍歷][X] 必定不會計算。可以計算出至少會有 N × 2<sup>N - 1</sup> 的空間被浪費。

## 實作

根據上述的轉換方程，可以使用 top-down 的作法來實現。由於使用位元儲存城市的遍歷狀態，每一個位元僅能使用一次。但在 TSP 中，有要求商人從居住城市出發並回到居住城市，按照該要求商人居住城市就遍歷了兩次。為解決該問題，在初始化狀態時，先將 dp[ 2<sup>X</sup> ][ X ] 的值初始為 length(Src → X) 之值。該初始化的意義為令已遍歷 X 城市且處於 X 城市中，其值為自商人居住城市到 X 的城市之距離，由於商人居住城市不能遍歷兩遍，因此直接將商人出發到第一個城市的距離設定好，即可避免需要從 X 城市回到商人居住城市時，發現不能遍歷的問題！

### Pseudo Code

```
init
	for i in n
		dp[ 1 << i ][ i ] = length[ source ][ i ]
 
TSP( visited_status, x ) 
	if dp[ status ][ x ] is found
		return dp[ status ][ x ]
 
	for i in n
		if this city i is visited and city is not city x
			dp[ visited_status ][ x ] = min( TSP( visited_status - current city x, i ) + length( city i to city x )
	return dp[ visited_status ][ x ]
```

### Source Code

<script src="https://gist.github.com/KuoE0/3872341.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/3872341).

有任何問題，歡迎來信或留言與我討論，若有錯誤的地方也歡迎指正！謝謝 :)
