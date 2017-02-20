---
layout: post
title:  "[LeetCode] 64 - Minimum Path Sum"
date:   2017-02-20
tags:   ["problem solution | 題解"]
---

題目連結：[https://leetcode.com/problems/minimum-path-sum/?tab=Description](https://leetcode.com/problems/minimum-path-sum/?tab=Description)

## 題目概述

給訂一個 N × M 的網格，每個格子內都會有一個非負整數。找一條路線從最左上角的格子到最右下角的格子，使這條路線經過的格子內的數字和最小。輸出這個最小的數字和是多少？

### Clarification

- 格子內的數字皆為非負整數
- 路徑的行徑方向只有向右與向下

---

## 解題思路

先從 1 × 1 的網格來想，答案勢必就是處於 (0, 0) 的數字。那如果是 1 × 2 的網格，那答案就是 (0, 0) 的數字加上 (0, 1) 的數字。而如果是 2 × 1 的網格，答案就是 (0, 0) 的數字加上 (1, 0) 的數字。那如果是 2 × 2 的網格，勢必要先走到 (1, 0) 或是 (0, 1) 才能走到 (1, 1) 對吧（只能往下走或往右走）！所以只要挑走到 (0, 1) 跟走到 (1, 0) 其中最小的路徑即可！

所以可以歸納出到每個位置的答案都會是從上方或是左方的最小和加上本身的數字，這麼一來我們就很容易可以找出到每個位置的答案，而題目的答案也跟著出來了！

### Complexity

- 時間複雜度：O(N × M)
- 空間複雜度：O(N × M)

### Source Code

```c++
class Solution {
public:
	int minPathSum(vector<vector<int>>& grid) {
		size_t N = grid.size(), M = grid.back().size();
		vector<vector<int>> minPathGrid(N, vector<int>(M, 0));
		minPathGrid[0][0] = grid[0][0];
		for (size_t i = 1; i < M; ++i) {
			minPathGrid[0][i] = minPathGrid[0][i - 1] + grid[0][i];
		}
		for (size_t i = 1; i < N; ++i) {
			minPathGrid[i][0] = minPathGrid[i - 1][0] + grid[i][0];
		}

		for (size_t i = 1; i < N; ++i) {
			for (size_t j = 1; j < M; ++j) {
				minPathGrid[i][j] = min(minPathGrid[i - 1][j], minPathGrid[i][j - 1]) + grid[i][j];
			}
		}
		return minPathGrid.back().back();
	}
};
```
