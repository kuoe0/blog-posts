---
layout:  post
title:   "使用 Prolog 求解數獨"
date:    2013-07-22
tags:    ["Prolog", "sudoku | 數獨"]
feature:
    photo:       true
    creator:     "Tim Psych"
    url:         "https://www.flickr.com/photos/01-17-05_t-m-b/2156513671/"
    license:     "CC 2.0"
    license_url: "https://creativecommons.org/licenses/by/2.0/"
---

這學期第二個程式作業是 Prolog，前面已經寫過簡單的教學 ([連結](http://blog.kuoe0.tw/posts/2013/06/19/prolog-tutorial)) 了，看完那份簡略的教學後，我想應該有機會可以寫出這個作業！不過，我個人的解法上，還是有部分語法是在該教學沒提到的就是。

由於不希望學生們都到網路上抄襲程式碼，因此定下了兩個額外的規定，第一是不準額外引入任何 library，不過如果是 SWI-Prolog 開啓就會自動引入的則例外。主要是我上網查了一下，發現有許多版本都有引入 CLPFD 這個 library，可以讓解數讀的程式變得非常簡潔，有興趣可以參考以下鏈結：[連結 1](http://programmablelife.blogspot.tw/2012/07/prolog-sudoku-solver-explained.html)、[連結 2](http://www.swi-prolog.org/pldoc/doc_for?object=transpose/2)。可以發現有趣的是，SWI-Prolog 的官方手冊竟然就有 sudoku solver 作為範例，可見我這個作業實在出的不好，根本意圖使學生上網抄襲。第二的規定是，不用實作數獨的區塊限制，本來是想說題目做些修改應該可以更有效地防止上網抄襲這件事，不過當我宣布後我就後悔了，畢竟抄襲了一份有做區塊限制的程式碼也是符合作業要求。在批改作業時，發現有些許同學都是參考這份程式碼 ([連結](http://www.karthiknadig.com/2012/03/01/sudoku-in-prolog/))，這份程式碼大量的變數因此令我印象深刻，不過我個人認為這份並不是個好做法就是。

接下來回歸正題，該怎麼利用 Prolog 實作 sudoku solver？！我個人是採用動態增加刪除 fact 的方式，以下是我的解題流程：

1. 讀取題目，並將已填入數字增加到 fact 中，行列區塊各項分開
2. 再讀取一次題目，開始枚舉未填入元素的數字，利用資料庫中已存在的 fact，來枚舉出還未出現的數字
3. 每次枚舉一個數字就要將其增加到 fact 中
4. 若找不到答案而回溯時，需要將該數字的 fact 給刪除
5. 最後即可得出答案

以下是我的程式碼：

<script src="https://gist.github.com/KuoE0/6049175.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/6049175).

首先從第 76 行來看，是程式的進入點，並指定從第 71 行的 `sudoku_solver` 開始。第 71 行的 `sudoku_solver` 才是程式的主體，其內容為從 standard input 讀入題目，再進行求解，最後將答案輸出至 standard output。

求解的過程從第 51 行的 `solve_sudoku` 開始，裡頭只做兩件事情：第一件事情是初始化 Prolog 的知識庫，利用題目來建立已存在 constraint，也就是動態的增加 fact。初始化知識庫後，才真正進行求解。

初始知識庫的過程在第 31 行，該敘述會依序讀入所有元素，若該元素為空的則跳過（第 42 行），否則就將替該元素增加 constraint 的 fact（第 33 行）。增加 fact 的方式是根據 index 找出該元素屬於哪一行、哪一列以及哪一個區塊，再替該行該列該區塊增加一個 fact。

例如：在 index 為 17 的元素為 2，計算後得知該 index 在第 1 列第 8 行的位置，且該位置屬於第 2 個區塊（P.S. 從 0 開始）。因此就要新增以下三個 fact：`row_used(1, 2)`、`col_used(8, 2)`、`block_used(2, 2)`。表示第 1 列、第 8 行以及第 2 個區塊都已經存在 2 這個數字了。

完成知識庫的初始化後，開此進行求解，可以在第 55 行找到求解的程序。求解的過程也是讀入所有元素，若該元素為已填入數字，則不需要進行枚舉，往下一個數字前進。遇到空元素則開始進行枚舉，枚舉方式為列舉出 1~9 的數字，根據該位置的 index 來檢查是否已經存在該數字在該列、該行或該區塊出現過了，可以查看第 24 行的 `available` 程序。找到可放置的數字後，就要將該數字動態的新增到知識庫中，參見第 66 到 68 行。放置後，再繼續往下一個數字前進，直到沒有元素為止。

但是，求解過程中不可能都一次就能枚舉到正確的數字，勢必會有回溯的發生。一旦發生回溯，代表某個枚舉的數字是錯誤的，既然是錯誤的我們就應該把它從知識庫中刪除。所以可以看到我第 66 到 68 行並非直接使用 `asserta` 來進行新增，採用自己定義的程序。

我們再來看到第 47 行，由於已經存在的 fact 是不能再進行一次新增的。若發生這樣的狀況，新增敘述就會回傳 `false`，轉而進行第 48 行的程序。第 48 行是動態地刪除知識庫中的 fact，不過正確的刪除會回傳 `true` 回來，這麼一來仍然會繼續往下一個元素前進。而我們應該要做的是重新枚舉數字，因此在第 48 行的程序結尾加上了 `fail`，以強迫其回傳 `false` 來保證回溯的發生，才能回溯到枚舉數字的程序。


![hardest sudoku](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-07-22-use-prolog-to-solve-sudoku-1.jpg)

測試了一下該程式，我個人認為還滿快的，連號稱[最難的數獨](http://www.telegraph.co.uk/science/science-news/9359579/Worlds-hardest-sudoku-can-you-crack-it.html)也只要花 0.23 秒就可以解決了。程式執行方式如下：

```bash
$ swipl -qs sudoku_solver.pl < sudoku.in > sudoku.out
```

**sudoku.in**

	0 0 5 3 0 0 0 0 0 8 0 0 0 0 0 0 2 0 0 7 0 0 1 0 5 0 0 4 0 0 0 0 5 3 0 0 0 1 0 0 7 0 0 0 6 0 0 3 2 0 0 0 8 0 0 6 0 5 0 0 0 0 9 0 0 4 0 0 0 0 3 0 0 0 0 0 0 9 7 0 0

**sudoku.out**

	1 4 5 3 2 7 6 9 8 8 3 9 6 5 4 1 2 7 6 7 2 9 1 8 5 4 3 4 9 6 1 8 5 3 7 2 2 1 8 4 7 3 9 5 6 7 5 3 2 9 6 4 8 1 3 6 7 5 4 2 8 1 9 9 8 4 7 6 1 2 3 5 5 2 1 8 3 9 7 6 4 
