---
layout: post
title:  "使用 Common Lisp 實現中序運算式轉後序運算式"
date:   2013-05-29
tags:   ["Common Lisp", "postfix expression | 後序運算式"]
feature:
    photo: false
---

這學期擔任程式語言課程助教，該課程會額外介紹三個語言分別是 Lisp、Prolog 與 Python。對於只學過 C/C++/Java 的大二學生來說，這幾種語言的思考邏輯應該比較特別，尤其是 Lisp 的 functional programming 與 Prolog 的 programming in logical。

這次出的作業為利用 Lisp 實現將中序運算式轉換為後序運算式的程式，對於大二學生，他們在上學期的資料結構課程中也已經學習過相同的演算法了，只不過改成 Lisp 來實做而已。不知道轉換的演算法的話，可以參考－[中序式轉後序式（前序式）](http://openhome.cc/Gossip/AlgorithmGossip/InFixPostfix.htm)這篇文章。

作業輸入是由標準輸入來輸入多組中序運算式，每個式子一行，且所有運算子與運算元皆以一個空白分隔。而輸出則是由標準輸出來輸出，對於標準輸入所輸入的每個中序運算式也用一行輸出其後序運算式，一樣用空白分隔所有運算子與運算元。

身為參與程式競賽的選手，雖然曾經認為規範輸入與輸出是一件很討厭的事情。但是，在這個自動化的社會，往往會將 A 程式的輸出拿來作為 B 程式輸入，這麼一來，若格式有誤豈不是會發生問題！所以我認為 I/O 的格式規範非常重要！大概是因為我也想直接寫程式來評斷學生的作業，不過似乎格式的規範也造成了些民怨。

由於我實在不喜歡一問三不知的助教，所以我決定跟著學生們一起學 Lisp，並一起教作業。不過在我寫完後，我發現 moodle 不准助教繳交作業…。

在開始寫 Lisp 時，首先遇到的困難是變數宣告。起初發現了 `setf` 的語法，其 syntax 為 `(setf {pair}*)`，例如：`(setf x 1 y 2)` 會將 x 賦予 1 且 y 賦予 2，並以 y 的值作為回傳值。但是若是該變數並未被宣告的話，會成為 global variable。我個人寫程式是不喜歡宣告太多非必要的 global variable，除了寫 ACM 要求程式執行效率外。

於是，我又發現了 `let` 敘述，他的 syntax 為 `(let (variable-declaration-form*) body-form*)`。但我一直看不懂到底該怎麼用，先試了 `(let (x 1) (print x))`，結果出現錯誤。研究了一下才發現是要 `(let ((x 1)) (print x))`，由於第一個參數他是個變數宣告的 list，可以宣告多組變數，例如：

```common-lisp
(let ((x 1) (y 2))
    (print x)
    (print y))
```
    
所以在宣告一個變數時會發現它被用兩層括號包住。而 `let` 的 body-form* 則是在這個 `let` 敘述內要做的事情，所以也可以有多個敘述在裡面，並且會把最後一個敘述的回傳值作為 `let` 的回傳值。

在知道怎麼宣告變數後，接下來又遇到另一個問題，就是變數的 lifetime。原來用 `let` 宣告的變數只能在 `let` 區塊內使用。這樣一來似乎如果我要宣告一個 function 時，一開始就得用 `let` 敘述來宣告區域變數。所以其實可以這麼想，把 `let` 想成是 C 的大括號區塊，一進入區塊就得宣告好該區塊會用到的 local variable，理解 `let` 的用法後我覺得對於寫 Lisp 程式來說可說是一大進展！

接下來遇到下一個問題－迴圈 (loop)。發現有 `do` 的敘述可以使用，其 syntax 為 `(do ({var | (var [init-form [step-form]])}*) (end-test-form result-form*) declaration* {tag | statement}*)`，例如：

```common-lisp
(do ((i 0 (+ i 1)))
    ((> i 10) i)
    (print i))
```

該敘述會輸出 [0,10] 之間的數字，並回傳 11。對於 `do` 的第一個參數就是變數宣告，可以看作是 C 語言中的 for-loop 其用分號分隔的第一個區塊。而對於每個宣告變數的區塊來說，第一個是變數名稱，第二個是初始值（可不指定），第三個則是每次回圈結束後會進行的動作（可不指定）。所以以上面的例子來說，就是宣告一個變數 `i`，其初始值為 0，且每次迴圈結束就會遞增 1。而 `do` 的第二個參數為終止條件與跳出迴圈後會執行的敘述，並將最後一個敘述作為回傳值。以上面為例就是當 i 大於 10 時會跳出迴圈，並將 i 當前的值作為 `do` 的回傳值！而其餘部分就是迴圈內要執行的敘述，以上為例就是 `(print i)`，輸出每次 `i` 的值。

會宣告變數，又會了迴圈後，還剩下一個控制方法需要會，就是條件分支敘述，就是 if-else 啦！在 Lisp 裡面就是 `if` 敘述，不過很容易就不特別提了！另外若是需要多組條件的話，可以利用 `cond` 敘述，這也很容易理解也不特別說明了。在會了變數宣告、迴圈控制以及條件控制後，我們就可以開始來寫作業啦！

這次作業是中序運算式轉換為後序運算式，有學過的應該都知道我們會需要一個 stack 的資料結構。不過 Lisp 本身就有 list 的資料結構，利用 `car` 與 `cdr` 應該很容易實作 stack！不過，在看一些教學文件有看到 Lisp 本身就有內建 stack 的操作。分別是 `push` 與 `pop` 敘述，其 syntax 為 `(push value stack)` 與 `(pop stack)`，其中 stack 就是一個 list 的變數！請見以下範例：

```common-lisp
(let ((stk ())
    (ret))
    (push 1 stk)
    (push 2 stk)
    (setf ret (pop stk)))
```

該段表示先將 1 放入一個空的 stack 中，再把 2 也放入。接下來在將 stack 頂端的數字拿出來，並賦予給變數 ret，並將 `ret` 作為 `let` 的回傳值。執行後即可發現，`let` 會回傳 2。

以下即為現學現賣的 Lisp 程式碼與測試資料：

<script src="https://gist.github.com/5664977.js"></script>

Source code on [gist](https://gist.github.com/5664977).

上面需要注意一下 expr.lsp 的第一行，為了讓該檔案給予執行權限後可以直接執行，才特別加上去了！若是出現錯誤，請移除該行！

來寫點簡單的學習 Lisp 心得，其實這個語言我想學很久了，當初就是被「[駭客與畫家](http://www.anobii.com/books/%E9%A7%AD%E5%AE%A2%E8%88%87%E7%95%AB%E5%AE%B6/9789867794697/003bca9f19d7b292b4/)」這本書給影響。作者 Paul Graham 是個 Lisp 高手，在這本書的後半段其實也都是在推廣 Lisp，導致我想學 Lisp 想很久了！而大二我也修過相同的課程，不過出的作業都沒幾行就可以完成，根本學不到東西啊！！抱持著「從做中學」的理念，我才派了個較為困難（相較於以往）的作業，希望學生們能從實作中獲得成長，我也藉著這個機會從實作中獲得成長，我想這就是所謂的教學相長吧！

在撰寫的過程中，發現 functional programming 對於習慣 C/C++ 等 imperative programming 來說真的有點難入門。老實說，寫完了一支程式後還是覺得有點難上手，不過我想主要是還沒熟悉一些 builtin function 的使用，但似乎已經可以用 functional programming 的方式來思考程式的撰寫方式，所以覺得其實很多 builtin function 都滿有機會可以自己實現的！而且用 functional programming 的方式思考根本就是在抽象化問題，用簡單地敘述來解決任意規模的問題真的非常優雅！另外，據說 Common Lisp 的 macro 非常強大，目前學的還非常粗淺，希望未來能多用 Lisp 來寫程式！

此外，因緣際會之下發現了 [newLisp](http://www.newLisp.org/) 這個 Lisp 的腳本語言，以後來嘗試多用 newLisp 寫些小東西好了！是說，一直很想學 [Haskell](http://www.haskell.org/haskellwiki/Haskell) 也遲遲沒有開始！！

目前作業還有使用 Lisp 抓取氣象預報資料，以及使用 Prolog 解數獨，等 deadline 過後再來分享！
