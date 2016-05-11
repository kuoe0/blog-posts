---
layout: post
title:  "[Data Mining] Apriori Algorithm"
date:   2011-12-01
tags:   ["data mining | 資料探勘", "association rule | 關連性規則"]
feature:
    photo: false
---

**P.S. 以下內容純為小弟個人研究看法，若有錯誤歡迎來信或留言指正，謝謝 <(__)>**

由於這學期有修碩士班的 data mining 課程，這週要 demo 期中 project，加上我們竟然在 ICPC 後才開始寫。時間緊迫，就寫了個最簡單也很經典的 apriori algorihm。可惜執行效率不佳，另外有看到 FP-Tree 的演算法，可惜有點複雜，未來若有接觸到再研究吧。

Apriori 是在資料探勘的領域中，用來找尋關聯式規則 (association rules) 的經典演算法之一。關聯式規則的應用，較常見的例子是在顧客的購物分析上，在其他領域也有相當多的應用。一個比較常被提出的故事為「**尿布與啤酒**」，這故事就是發現許多顧客買了尿布就會買啤酒。經過研究分析後，發現到：「在美國，許多年輕母親都留在家中照顧嬰兒，而父親匯到超市買尿布，但往往就會順便買啤酒」。在資料探勘的領域中，藉由「尿布與啤酒」的故事，我們發現在生活上，可存在許多表面上看似毫無關連的事物，但其實有著密不可分的關係， apriori 就是被提出用來找出這些關連的演算法之一。

## Algorithm

**變數與函式**

- **D**：Transaction Database，儲存我們所要分析資料的資料庫
- **T**：Transaction，資料庫中的一筆資料（交易），T ∈ D
- **Support**：在 D 中，支持該 itemset 的 T 佔有的比例，Support(X) = Occur(X) / Count(D) = P(X)
- **Confidence**：Conf(X → Y) = Support(A ∪ B) / Support(A) = P(Y | X)
- **Candidate Itemset**：通過 apriori 合併操作所得到的 itemset
- **Frequent Itemset**：support 值大於 min support 之 itemset
- **C(k)**：含有 k 個元素之 candidate itemset 之集合
- **L(k)**：含有 k 個元素之 frequent itemset 之集合

**演算法流程**

1. 從 D 中找出 C(1)
2. 再掃描 D 找出所有包含 C(1) 的 T，計算其 support，將 C(1) 中不符合 min support 的 itemset 剔除，結果即為 L(1)
3. 自 L(k) 來產生其他的 L(k+1)，直到找 L(k+1) 為空集合（k 由 1 開始）
4. 產生 L(k+1) 的首要步驟為合併 L(k) 來產生 C(k+1)
5. 在 C(k+1) 中，**若有某 itemset 其所有含 k 項的 subset 不屬於 L(k)，即可將其自 C(k+1) 剔除**
6. 掃描 D 找出所有包含 C(k+1) 的 T，計算其 support，將 C(k+1) 中不符合 min support 的 itemset 剔除
，結果即為 L(k+1)
7. 枚舉此 L(k+1) 中的 itemset，將 itemset 分成兩個部份 {A}{B} 的所有可能（其中 A 含有 p 個項目，而 B 含有 (k+1)-p 個項目，1<=p<=(k+1)-1）
8. 判斷 P(itemset) / P(A) 是否大於等於 min confidence，若符合我們可以判斷存在 A => B 的 association rule

**關於上述第 6 步驟，為什麼可以只判定含有 k 項的 subset？**

由於我們每次產生的 L(k) 也都已經過 k-1 項的 subset 篩選過，在此具有**向下封閉性**。故我們只需要使用其 k 項的 subset 來篩選。

**優點**

- 簡單易懂
- 實現複雜度低

**缺點**

- 產生過多的 candidate itemset
- 需重複掃描資料庫

## Pseudo Code

```
L(1) = Scan database to find {frequent itemset}
	
do
	Gnerate C(k+1) from L(k)
	Remove the itemset which has any subset that is not a frequent itemset (detect from L(k))
	L(k+1) = candicates in C(k+1) with min_support
	
	for all possible of partition L(k+1) into two parts A,B
		if P(L(k+1)) / P(A) >= min_confidence
			A => B is one association rule
	
until L(k+1) is ∅  

```

## C++ Implementation

<script src="https://gist.github.com/KuoE0/1415437.js"></script>

source code on [gist](https://gist.github.com/KuoE0/1415437).

## Reference

- [http://www.cs.sunysb.edu/~cse634/lecture_notes/07apriori.pdf](http://www.cs.sunysb.edu/~cse634/lecture_notes/07apriori.pdf)
- [http://css.engineering.uiowa.edu/~comp/Public/Apriori.pdf](http://css.engineering.uiowa.edu/~comp/Public/Apriori.pdf)
- [http://en.wikipedia.org/wiki/Apriori_algorithm](http://en.wikipedia.org/wiki/Apriori_algorithm)
- [http://www.cnblogs.com/gaizai/archive/2010/03/31/1701573.html](http://www.cnblogs.com/gaizai/archive/2010/03/31/1701573.html)
- [http://www.cnblogs.com/zgw21cn/archive/2009/05/31/1492809.html](http://www.cnblogs.com/zgw21cn/archive/2009/05/31/1492809.html)
