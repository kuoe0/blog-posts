---
layout: post
title:  "[LeetCode] Reverse Words in a String"
date:   2014-03-16
tags:   ["string | 字串", "problem solution | 題解"]
---

題目連結：[http://oj.leetcode.com/problems/reverse-words-in-a-string/](http://oj.leetcode.com/problems/reverse-words-in-a-string/)

## 題目概述

題目會給定一行字串，這些字串中包含了些空格與詞彙。現在要把詞彙的順序給顛倒過來，例如：把 `the sky is blue` 變成 `blue is sky the`。

### Clarification

- 連續的非空白字元視為一個詞彙
- 需要清除任何前綴或後綴之空白
- 將連許兩個以上（含）的空白字元縮減為一個

---

## 解題思路

其實算還滿簡單的題目，只要將字串依照空白來進行切割，並把切割下的詞彙儲存起來並反向輸出即可。以往在進行字串切割時，都習慣使用 `strtok` 來處理。突然覺得該來找找有沒有更容易實作的方法，因而讓我發現 `istringstream` 與 `ostringstream` 該兩個類別，用於實現將字串作為串流般操作，就像 `cin` 與 `cout` 一樣。程式碼如下：

```c++
class Solution
{
public:
    void reverseWords(string &s)
	{
		istringstream strIn(s);
		ostringstream strOut;
		vector<string> words;
		string tmp;

		while (strIn >> tmp) {
			words.push_back(tmp);
		}

		while (!words.empty()) {
			strOut << words.back();
			words.pop_back();

			if (!words.empty()) {
				strOut << ' ';
			}
		}

		s = strOut.str();
    }
};
```

是說第一次寫 LeetCode 真不習慣，原以為要輸出到 `stdout`，結果拿到 Output Limit Exceeded。後來又想說更改回傳型別好了，就改成回傳 `string`，然後拿到 Wrong Answer。後來才發現原來要直接修改參數。

另外，LeetCode 也可以支援 Python。用 Python 來解更容易，只需要僅僅一行！程式碼如下：

```python
class Solution:
    # @param s, a string
    # @return a string
    def reverseWords(self, s):
        return " ".join(s.split()[::-1])
```
