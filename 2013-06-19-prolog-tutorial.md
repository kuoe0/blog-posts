---
layout:  post
title:   "Prolog 入門"
date:    2013-06-19
tags:    ["Prolog"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

本文同步刊載於[程式人雜誌 2013 年 8 月](http://programmermagazine.github.io/201308/htm/article3.html)。

這學期第二個教的程式語言是 Prolog，這個語言更奇妙了！之所以稱作 Prolog 的理由是，他是「**Pro**gramming in **log**ic」的縮寫。被廣泛使用於人工智慧的領域，更常被作為設計專家系統的語言。

老實說，Prolog 個人覺得真的不好學習，對於習慣 imperative programming 的來說，思維模式比 functional programming 更難令人接受！Prolog 是由我們預先設計好一些規則作為其知識庫，接下來我們就可以給予問題詢問 Prolog，Prolog 就會搜尋其知識庫並回答出符合其知識庫中的正確答案。Prolog 也是有許多不同的實現，這邊我選擇了 [SWI Prolog](http://www.swi-prolog.org/)。

## 詞法結構 Syntax

先看看 Prolog 的基本語法，在 Prolog 中有三種主要語法結構，分別是：fact、clause 以及 query，而這三種語法結構都是由 predicate 為基礎，中文稱之為「謂詞」。predicate 有點像是其他語言的函式一般，由一個名稱與一些參數組成，且其回傳值僅為 boolean 值。見下表的詞法結構：

| type | BNF |
| --- | --- |
| predicate | `<predicate> ::= <P>(<ARGS>)` |
| fact | `<fact> ::= <predicate>.` |
| clause | `<clause> ::= <predicate> :- <predicate> {(,｜;) <predicate>}.` |
| query | `<query> ::= ?- <predicate> {(,｜;) <predicate>}.` |

先看到 fact，其語法是 predicate 後，加上一個句點 (.)，表示該 predicate 為真！而 clause 其語法可以是多個 predicate，表示當所有 clause 所包含的 predicate 都為真時，該 clause 為真！另外，在 clause 中，逗號 (,) 表示「且 (and)」，分號 (;) 表示「或 (or)」，所以，clause 就像是一個條件式。用分號分隔的 clause 也可能分開成多條 clause 來寫。例如：

```prolog
clause :- 
    goal1 ;
    goal2.
```
    
可以改寫成：

```prolog
clause :- goal1.
clause :- goal2.
```
    
從上面的例子中也可以發現，Prolog 的語法都可以跨行，並不一定要寫在單一行！在 clause 中，還可以利用括號來做群組。另外，還有一個符號 `->` 用來作條件判斷，當置於 `->` 之前的敘述結果為 true 時，才執行 `->` 之後的敘述，一樣直接看例子：

```prolog
clause :-
    (    cond1 ->
        goal1
    ;    cond2 ->
        goal2
    ;    goal3
    ).
```

利用括號與 `->`，可以很容易的實現 if-else 的條件敘述。

最後，是 query，長個很像 fact，唯一的差別是前面加上了 `?-` 的符號，表示一個查詢，Prolog 會幫我們找出令該 predicate 為真的結果。

另外，其實 fact 也可以看作是一個 clause，以下是其 BNF 語法：

```
<fact> ::= <predicate> :- true.
```

### 範例

我們先假設幾條 predicate：

```prolog
isMom(A, B)            % A 是 B 的媽媽
isDad(A, B)            % A 是 B 的爸爸
isParent(A, B)        % A 是 B 的父母
isGrandParent(A, B)    % A 是 B 的祖父或祖母
```

有了這幾個 predicate 後，我們可以建立這些 predicate 的關係，也就是 clause：

```prolog
isParent(A, B) :- isMom(A, B).    % A 是 B 的媽媽的話，表示 A 是 B 的父母
isParent(A, B) :- isDad(A, B).    % A 是 B 的爸爸的話，表示 A 是 B 的父母
isGrandParent(A, C) :- isParent(A, B), isParent(B, C).    % A 是 B 的父母且 B 是 C 的父母的話，表示 A 是 C 的祖父或祖母
```

再來我們需要針對這些 predicate 建立一些 fact：

```prolog
isMom(毛福梅, 蔣經國).
isMom(蔣方良, 蔣孝文).
isMom(蔣方良, 蔣孝武).
isDad(蔣介石, 蔣經國).
isDad(蔣經國, 蔣孝文).
isDad(蔣經國, 蔣孝武).
```
    
那個該怎麼在 SWI Prolog 中宣告這些規則呢？首先先開啓 SWI Prolog 的 interactive shell，在終端機中輸入 `swipl` 開啟。執行後會輸出以下訊息：

```prolog
    Welcome to SWI-Prolog (Multi-threaded, 64 bits, Version 6.2.6)
    Copyright (c) 1990-2012 University of Amsterdam, VU Amsterdam
    SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software,
    and you are welcome to redistribute it under certain conditions.
    Please visit http://www.swi-prolog.org for details.
    
    For help, use ?- help(Topic). or ?- apropos(Word).
    
    ?- 
```

會發現 SWI Prolog 的 prompt 是 `?-`，還記得前面提過的 Prolog 的詞法結構嗎？可以發現這就是 query 的開頭，所以在 SWI Prolog 的 interactive shell 中，我們能做的就是進行 query！

只能 query 的話，該怎麼使用上述的規則呢？我們建立一個檔案名為 `chiangfamily.pl` （注意檔名不可以使用大寫）內容如下：

```prolog
isParent(A, B) :- isMom(A, B).    % A 是 B 的媽媽的話，表示 A 是 B 的父母
isParent(A, B) :- isDad(A, B).    % A 是 B 的爸爸的話，表示 A 是 B 的父母
isGrandParent(A, C) :- isParent(A, B), isParent(B, C).    % A 是 B 的父母且 B 是 C 的父母的話，表示 A 是 C 的祖父或祖母

isMom(毛福梅, 蔣經國).
isMom(蔣方良, 蔣孝文).
isMom(蔣方良, 蔣孝武).
    
isDad(蔣介石, 蔣經國).
isDad(蔣經國, 蔣孝文).
isDad(蔣經國, 蔣孝武).
```
    
並在這個檔案存在的資料夾中開啟 SWI Prolog，接著輸入 `consult(chiangfamily).` 的 query，這時候就可以將這個檔案中的規則都讀取進來到 Prolog 的知識庫中。

讀取後我們就可以開始查詢蔣家的親屬關係了！例如我們想知道蔣介石是不是蔣經國的爸爸？只要輸入 `isDad(蔣介石, 蔣經國).`，這時候 Prolog 就會回傳一個 `true` 回來，按下 enter 表示確定。不過這個例子只是查詢我們預先定義的事實罷了，好像沒什麼！那我們再來試試，蔣介石是不是蔣孝文的祖父呢？輸入 `isGrandParent(蔣介石, 蔣孝文).`，這時候 Prolog 一樣回傳了 `true`，這次查詢就不是我們給定的事實了，而是 Prolog 自己根據 fact 與 clause 所推論出來的！因為我們存在 `isDad(蔣介石, 蔣經國).` 與 `isDad(蔣經國, 蔣孝文).` 這兩個事實，再根據推論方法 `isParent(A, B) :- isDad(A, B).` 與 `isGrandParent(A, C) :- isParent(A, B), isParent(B, C).` 即可得知，這是利用 Prolog 的*比對能力！

再來，我們在看個 Prolog 更厲害的地方－**列舉**。如果，我們想知道蔣介石的孫子有誰呢？我們只要輸入 `isGrandParent(蔣介石, X).`，Prolog 會回傳 `X = 蔣孝文.`，這時候先不要按下 enter 確定，可以按下 space 取得更多答案，就會再回傳一個 `X = 蔣孝武.` 的結果，前面的 `X = 蔣孝文` 後面也會由句號變成分號，這表示蔣孝文跟蔣孝武都是蔣介石的孫子！一樣的我們可以查詢誰是蔣孝文的祖父母？輸入 `isGrandParent(X, 蔣孝文).`，這時候 Prolog 一樣先回傳了 `X = 毛福梅.`，如果按下 space 可以得到更多答案，也就是 `X = 蔣介石.`。如果我們再進一步的按下 space 想知道更多的答案的話，會得到一個 `false`，這表示已經沒有符合的值了。

欲離開 SWI Prolog 的話，只要輸入 `halt.` 即可。以下是完整的執行流程：

```prolog
$ swipl
Welcome to SWI-Prolog (Multi-threaded, 64 bits, Version 6.2.6)
Copyright (c) 1990-2012 University of Amsterdam, VU Amsterdam
SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software,
and you are welcome to redistribute it under certain conditions.
Please visit http://www.swi-prolog.org for details.

For help, use ?- help(Topic). or ?- apropos(Word).

?- consult(chiangfamily).
% chiangfamily compiled 0.00 sec, 10 clauses
true.

?- isDad(蔣介石, 蔣經國).
true.

?- isGrandParent(蔣介石, 蔣孝文).
true .

?- isGrandParent(蔣介石, X).
X = 蔣孝文 ;
X = 蔣孝武.

?- isGrandParent(X, 蔣孝文).
X = 毛福梅 ;
X = 蔣介石 ;
false.

?- halt
$
```

## 資料結構 Data Type

首先，我們稱任何資料單位都叫做項 (term)。term 可以是原子 (atom)、數字 (number)、變數 (variable) 或是複合項 (compound term)，其中 atom 與數字屬於常數 (constant)。我個人是不太喜歡把 term 跟 atom 還有 compound term 給翻成中文，總覺得怪怪的，接下來這幾個詞還是以英文為主。

### 原子 atom

小寫開頭的文字序列，可以包含空白等特殊符號，但若是包含空白或特殊符號就需要用單引號 (') 包圍。若是要使用大寫開頭的 atom，也必須要使用單引號 (') 包圍，以區別跟變數的不同。另外，空列表 `[]` 也算是一個 atom。

### 數字 number

任何整數或浮點數，在 Prolog 中並沒有限制整數的大小，也就是支援整數的**大數運算**。

### 變數 variable

任何字母或是數字組成且由大寫或是底線開頭的文字串列。變數可以經過 unification 的機制而實例化，實例化表示將該變數用其他 term 替代，其他 term 可以是 atom、數字、變數或是 compound term。另外，在 Prolog 中有個特別的變數為 `_`，就單一一個底線，代表為匿名變數，表示我們對他的內容並不關心，它可以是任何值。

舉個匿名變數的例子：

定義一個 fact 為 `isMom(_, 蔣經國).`，一樣把它放進 chiangfamily.pl 的檔案中，並開啓 SWI Prolog 載入。然後我們來查詢誰是蔣經國的媽媽？輸入 `isMom(X, 蔣經國).` 會得到以下兩個結果：

```prolog
true ;
X = 毛福梅.
```
    
第一個 true 表示，我們根本不關心 `X` 到底是多少，至少第二個參數是`蔣經國`，就可以回傳 true 了。但我們也有定義另一個 fact 是 `isMom(毛福梅, 蔣經國).`，所以也會有第二個答案出現 `X = 毛福梅.`。既然我們不關心 `X` 的值是多少，那麼是不是隨便填都應該回傳 true 呢？那我們來試試 `isMom(宋美齡, 蔣經國).`，得到的結果也是 true，但我們並沒有把 `isMom(宋美齡, 蔣經國).` 寫入 fact 中。為了尊重蔣前總統，所以我就不再拿其他例子試啦！

另外，變數是俱有狀態的，前面提過透過 unification 的機制，變數可以被實例化，而被實例化的變數它就已經被綁定到某個值了，所以會說該變數已被綁定了。

### 複合項 compound term

一個由 functor 與參數表所組成的結構，可以把它想像成是 C/C++ 中的 structure，應該有點 [ADT](http://zh.wikipedia.org/wiki/%E6%8A%BD%E8%B1%A1%E8%B3%87%E6%96%99%E5%9E%8B%E5%88%A5) 的概念。functor 就是一個類似函式的名稱的東西，那後面接著參數表，其實就大概可以預見長什麼樣子了。直接舉幾個例子：`car('Ferrari', red, 'XY-1234')`、`'PersonInfo'('KuoE0', '1990-05-06', 'National Cheng Kung University')`。其實，atom 可以被視為一個沒有參數列表的 compound term。通常我們會用 **f/n** 來表示一個 compound term，其中 f 為 functor，n 為其 arity（arity 為參數數量）。

前面提過，Prolog 的語法基礎是 predicate，再來看一次 predicate 的 BNF 語法：

```
<predicate> ::= <P>(<ARGS>)
```

把 `<P>` 當作 functor，可以發現原來 predicate 本身就是一個 compound term 了，所以其實 Prolog 的所有基礎都建立在 compound term 這個資料結構！

以下是幾個用於 compound term 的操作：

`arg(N, Term, Arg)`：對 compound term 的第 N 個參數進行操作。

```prolog
?- arg(1, car('Ferrari', red, 'XY-1234'), X).
X = 'Ferrari'.
    
?- arg(2, car('Ferrari', X, 'XY-1234'), blue).
X = blue.
```
    
`functor(Term, Functor, NumberOfArgs)`：獲取 compound term 的名稱與參數數量，或是建構一個俱有特定名稱且擁有特定數目個自由變數的 compound term。

```prolog
?- functor(car('Ferrari', X, 'XY-1234'), F, A).
F = car,
A = 3.
    
?- functor(C, car, 3).
C = car(_G891, _G892, _G893).
```
    
`=..`：建構或解構一個 compound term。

```prolog
?- car('Ferrari', X, 'XY-1234') =.. X.
X = [car, 'Ferrari', X, 'XY-1234'].
    
?- X = [car, 'Ferrari', X, 'XY-1234'].
X = car('Ferrari', X, 'XY-1234').
```

### 列表 list

其實 list 在 Prolog 也是一個 compound term，只是 Prolog 賦予了她一個語法糖－利用中括號來使用。前面提過，一個空的 list 是一個 atom，寫作 `[]`，而一個包含 a、b、c、d 四個物件的 list 可以寫作 `[a, b, c, d]`。那為什麼說 list 是 compound 呢？其實中括號是一個 functor 為 `.` 且俱有兩個 arty 的 compound term，用 compound term 地表示方式為 **./2**。 `.` 俱有裡個參數，其中第一個可以是任何物件，但是第二個必須是一個 list。一個包含一個物件的 list，我們寫作 `.(a, [])`。前面包含 a~d 的例子用 `.` 的話就要寫作 `.(a, .(b, .(c, .(d, []))))`。這兩種寫法都是可以混用的，所以包含 a~d 的 list 也可以寫作 `.(a, [b, c, d])`。利用這個 functor，我們很容易可以得到一個 list 的頂端元素與剩餘元素，就像 Lisp 中的 car 與 cdr，以下是其 clause：

```prolog
car(X, L) :- L = .(X, _).
cdr(X, L) :- L = .(_, X).
```

來看看 car，由於 `X` 必須要是 `L` 的開頭，所以他應該也要是 `.` 這個 compound term 的第一個參數。另外，我們不需要知道該 list 的剩餘元素是什麼，所以就直接利用匿名變數 `_` 來表示即可。cdr 的原理也是一樣的。

假如說，現在要實現一個功能，但該功能僅需要 list 中的第一個元素，那麼就可以利用 `.` 這個 compound term。直接看例子，現在我要把第一個元素給加上 1：

```prolog
addOneToHead(.(H, T), X) :- Y is H + 1, X = .(Y, T).
```

執行 `addOneToHead([1, 2, 3], X).` 後，會得到 `X = [2, 2, 3].` 的結果。

但是，`.` 這個 compound term 其實還滿不易讀得我覺得，在 Prolog 中可以利用中括號 `[]` 與豎線 `|` 完全取代 `.` 的功能，用法就像是 `.`，傳入兩個參數，第一個是要放置在 list 前面的元素，第二個則是一個 list，然後用豎線隔開。前面包含 a~d 的例子就可以改寫為 `[a|[b|[c|[d|[]]]]]`，或是 `[a|[b, c, d]]`。利用這樣的方式改寫的 car 與 cdr 如下：

```prolog
car(X, L) :- L = [X|_].
cdr(X, L) :- L = [_|X].
```
    
前面的 `addOneToHead` 的例子也可以改寫為：

```prolog
addOneToHead([H|T], X) :- Y is H + 1, X = [Y|T].
```

如此一來，程式碼的閱讀上也較為容易！

### 字串 string

字串在 Prolog 中，字串是由雙引號 (") 所包圍住的文字串列，例如：`"KuoE0"`。其實字串在 Prolog 終究相當於一個整數的 list，這些整數就是這些字元所對應的 ASCII/UTF-8 code。以 `"KuoE0"` 這個字串來說，就相當於 `[75, 117, 111, 69, 48]` 這個 list。

另外有個用於 list 與字串的操作－`name/2`。其定義為 `name(Text, List)`，可以將 `Text` 轉換為 ASCII/UTF-8 code 的 list 後存給 List，反之亦然。

## 一致化 Unification

Prolog 如此強大的能力是基於其 unification 的機制，我們接下來就來看看什麼是 unification。

當兩個 predicate 的名稱與參數相同時，就可以促成 unification 的機制進行。而 unification 要成功的話，兩個 predicate 需要能夠化做一模一樣的形式。規則如下：

1. 當 term1 與 term2 皆為常數時，可以 unify 的條件為當兩者為相同的 atom 或是數字。
2. 當 term1 是變數而 term2 為任意型別，則 term1 可以被 term2 取代進行 unify，反之亦然。當兩者皆為變數時，兩者都可以互相取代進行 unify，而兩者的值也將會相同。
3. 當 term1 跟 term2 都是 compound term 時，必須要符合下列條件才能進行 unify：
    - 俱有相同的 functor 與 arity
    - 所有參數都可以 unify
    - 相同變數只能 unify 到相同的值

以下是利用 Prolog 實作 unify 的程式碼，如果看得懂的話，應該更能體會 unify 發生的條件與過程：

```prolog
unify(A,B) :-
    atomic(A), atomic(B), A = B.
unify(A,B) :-
    var(A), A = B.
unify(A,B) :-
    nonvar(A), var(B), A = B.
unify(A,B) :-
    compound(A), compound(B),
    A =.. [F|ArgsA], B =.. [F|ArgsB],
    unify_args(ArgsA, ArgsB).
  
unify_args([A|TA], [B|TB]) :-
    unify(A, B),
    unify_args(TA, TB).
unify_args([], []).
```

一樣用前面的例子來看，假設我們要問蔣經國的爸爸是誰？我們會輸入 `isDad(X, 蔣經國).`，前面一直忘記提到，在 Prolog 中，大寫開頭的 token 是變數的意思！看看我們給定的 fact 中，可以找到三個 `isDad` 的 predicate：

    isDad(蔣介石, 蔣經國).
    isDad(蔣經國, 蔣孝文).
    isDad(蔣經國, 蔣孝武).

由於常數不可以被取代，所以 query 中的「蔣經國」不可以被替換掉，那麼這三個 `isDad` 的 fact 就又被過濾掉了一些，並只剩下一個－`isDad(蔣介石, 蔣經國).`。而變數是可以被替換掉的，所以如果我們將變數 `X` 替換成「蔣介石」的話就可以符合我們給定的 fact 了，因此有了 `true` 的情況發生，Prolog 就會回傳一個 `X = 蔣介石.` 的結果給我們！

再來看另一個例子，我們想查詢所有的父子關係呢？輸入 `isDad(X, Y).` 進行查詢，一樣可以發現有三個 `isDad` 的 fact，由於我們查詢中並沒有常數，所以這三個 fact 都可以帶入得到 `true` 的結果，因此該次查詢可以得到三種答案！

那麼如果找不到可以符合的結果的話，就會回傳 `false`！例如我們想知道蔣介石的爸爸是誰？輸入 `isDad(X, 蔣介石).` 就會得到 `flase` 的結果。

透過 clause 推論規則，加上 unification 機制，Prolog 強大的推論便油然而生。來看個簡單的例子，`isParent(A, B) :- isMom(A, B).` 與 `isParent(A, B) :- isDad(A, B).` 這兩條 clause 都是 isParent 的推論規則。所以當我們想知道蔣經國的父母有誰？就可以透過 `isParent(X, 蔣經國).` 來做查詢，然後得到 `X = 毛福梅.` 與 `X = 蔣介石.` 這兩個結果。其推論的流程是先將 clause 替換成我們定義的推論規則，在這個例子中，`isParent(X, 蔣經國).` 會變成 `isMom(X, 蔣經國).` 或是 `isDad(X, 蔣經國).`。再透過 `isMom(X, 蔣經國).` 經由 unification 的機制將 `X` 帶入`毛福梅`後可以得到相對應的 fact，也就是得到 true 的結果，所以會回傳 `X = 毛福梅.` 的結果。然而，`isParent(X, 蔣經國).` 也可以推論為 `isDad(X, 蔣經國).`，將 `X` 帶入`蔣介石`後也可以找到相對應的 fact，所以也會回傳 `X = 蔣經國.`。

再看一個比較複雜一點點的範例，查詢所有祖孫關係。要查詢所有祖孫關係的話，需要輸入 `isGrandParent(X, Y).`。`isGrandParent(X, Y).` 可以推論為 `isParent(X, Z), isParent(Z, Y).` 也就是要找到 `isParent(X, Z).` 的結果是 true 且 `isParent(Z, Y).` 的結果是 true 的情況才會回傳 true。回傳結果如下：

```prolog
X = 毛福梅,
Y = 蔣孝文 ;
X = 毛福梅,
Y = 蔣孝武 ;
X = 蔣介石,
Y = 蔣孝文 ;
X = 蔣介石,
Y = 蔣孝武 ;
false
```
    
最後的 false 表示已經找不到這樣的關係了，所以可以發現四組祖孫關係。要瞭解 Prolog 的詳細推論流程的話，可以在 SWI Prolog 中輸入 `trace.` 打開 trace mode。這樣一來，Prolog 將會將所以推論流程通通顯示在螢幕上！以下是 `isParent(蔣經國, X).` 的推論流程：

```prolog
?- trace.
true.
    
[trace]  ?- isParent(蔣經國, X).
   Call: (6) isParent(蔣經國, _G1558) ? creep
   Call: (7) isMom(蔣經國, _G1558) ? creep
   Fail: (7) isMom(蔣經國, _G1558) ? creep
   Redo: (6) isParent(蔣經國, _G1558) ? creep
   Call: (7) isDad(蔣經國, _G1558) ? creep
   Exit: (7) isDad(蔣經國, 蔣孝文) ? creep
   Exit: (6) isParent(蔣經國, 蔣孝文) ? creep
X = 蔣孝文 ;
   Redo: (7) isDad(蔣經國, _G1558) ? creep
   Exit: (7) isDad(蔣經國, 蔣孝武) ? creep
   Exit: (6) isParent(蔣經國, 蔣孝武) ? creep
X = 蔣孝武.

[trace]  ?-
```


### occurs check

在 unification 中有個 issue 是 occur check，直接看個例子：

```prolog
?- X = f(X)
X = f(X).
```
    
可以看到變數 `X` 被 unify 成一個 compound term，而且這個 compound term 還包含他自己。因此他成了一個可無限遞回的結構，這麼一來我們除了可以把 `X` unify 為 `f(X)` 外，還可以 unify 為 `f(f(X))`、`f(f(f(X)`、`f(f(f(f(X)))` 等等無窮循環下去。

但在 Prolog 預設的 unify 方式就會像是上面的例子，如果不希望有這樣的 unify 情況發生，可以使用 `unify_with_occurs_check` 來進行 unification。看以下例子：

```prolog
?- unify_with_occurs_check(X, f(X)).
false.
```

因為有 occurs check，可以檢測到 `X` 會被 unify 為一個包含自己的 compound term，所以不會發生 unify。

## 編寫獨立的 Prolog 程式

就我個人來說，看了許多教學後，其實我還是不知道該怎麼寫出一個 Prolog 程式，應該說可以直接執行而不是進入 interactive environment 後在自己執行。看了 [Wikipedia](https://zh.wikipedia.org/wiki/Prolog) 後，發現其 quick sort 範例中有個 `:- initialization(q).` 的 clause，發現這個敘述前面竟然沒有東西，直接就接下 `:-` 的符號，我在想應該就是 interactive environment 開始的位置，而 `q` 就是自己編寫的 predicate，並從該處開始。其實有點像 C/C++ 裡的 `main` 一樣，在 Prolog 的程式進入點就是 `initialization`。不過，其實並不是 `initialization` 是 Prolog，只要開頭是 `:-` 這樣的符號，Prolog 都會直接執行該敘述。但為了方便程式有個進入點，就把 `initialization` 當作進入點吧！

以下程式碼引用自 [Wikipedia][3]，另外我加上了 `halt` 來離開程式，並儲存為 `quicksort.pl`：

```prolog
q:- L = [33, 18, 2, 77, 66, 18, 9, 25], last(P,_), (quicksort(L,P,_), write(P), nl), halt.

partition([], _, [], []).
partition([X|Xs], Pivot, Smalls, Bigs) :- 
    (   X @< Pivot ->
        Smalls = [X|Rest],
        partition(Xs, Pivot, Rest, Bigs)
    ;    Bigs = [X|Rest],
        partition(Xs, Pivot, Smalls, Rest)
    ).

quicksort([])     --- [].
quicksort([X|Xs]) ---
    { partition(Xs, X, Smaller, Bigger) },
    quicksort(Smaller), [X], quicksort(Bigger).
                                                              
:- initialization(q).
```

在終端機輸入下列指令即可直接執行該程式碼：

    $ swipl -q -s quicksort.pl
    [2,9,18,18,25,33,66,77]

其中 `-s` 是指將接下來的檔案以 script 的方式執行，而 `-q` 則是叫 SWI Prolog 安靜一點，不要輸出一些有的沒有。

