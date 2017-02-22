---
layout: post
title:  "[LeetCode] 80 Remove Duplicates from Sorted Array II"
date:   2017-02-22
tags:   ["problem solution | 題解"]
---

題目連結：[https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/?tab=Description](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/?tab=Description)

## 題目概述

給定一個已排序的整數序列，其長度為 N，現在允許每個整數在這個數列中最多只能出現兩次。求扣除重複超過兩次的數字後，這個數列的長度 X。另外，原數列的前 X 數字要題目允許的狀態排序後的結果。

例如，給定的數列是 `[1,1,1,2,2,3]`，其輸出的結果為 5。而原數列的前 5 項為 `[1,1,2,2,3]`。

---

## 解題思路

首先提出我的做法，這個做法非常直覺簡單。就是尋訪這個數列，因為是已排序過的所以按照順序尋訪很容易可以判斷重複的數字出現幾次。因此，只要在計數的過程中，計數小於等於 2，就把這個數字放到新的數列中。最後再用這個數列取代原本的數列即可。

### Complexity

- 時間複雜度：O(N)
- 空間複雜度：O(N)

### Source Code

```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
		if (nums.empty()) return 0;
		vector<int> ret;
		int cnt = 0, current = nums[0];
		for (size_t i = 0; i < nums.size(); ++i) {
			if (current == nums[i]) ++cnt;
			else current = nums[i], cnt = 1;
			if (cnt <= 2) ret.push_back(current);
		}
		nums = ret;
		return nums.size();
    }
};
```

但一直覺得這解法太過簡單，應該有更好的做法。一直在想是不是有可以不用使用額外空間的做法？經過一番思考後還是沒想到，只好參考了別人的做法。

這個做法不需要宣告一個新的數列來存放這些允許的數字，直接利用原本的數列空間即可。首先宣告一個計數器 X，用來紀錄目前數列中的前 X 項是允許狀態。該計數器其初始值為 0，然後開始尋訪數列中的每個數字。當 X 小於 2 時，直接把目前尋訪到的數字覆蓋到原數列的 X 位置，並讓 X 計數加上 1。若是 X 開始大於等於 2 時，則判斷當前尋訪的數字是否大於計數器所記錄的允許狀態的倒數第二個數字。若是大於則將該數字覆蓋到原數列的 X 位置，並讓 X 計數加上 1。如果小於等於的話，就不做任何事情。

可以這麼做的理由是如下：

- 當計數器小於 2 時，相同數字最多也只會重複出現兩次。
- 與允許狀態的倒數第二個數字比較，最多只會發生與最後一個數字重複，但不會出現重複三次的狀況。

### Complexity

- 時間複雜度：O(N)
- 空間複雜度：O(1)

### Source Code

```c++
class Solution {
public:
	int removeDuplicates(vector<int>& nums) {
		int len = 0;
		for (size_t i = 0; i < nums.size(); ++i) {
			if (len < 2 || nums[i] > nums[len - 2]) {
				nums[len++] = nums[i];
			}
		}
		return len;
	}
};

```


