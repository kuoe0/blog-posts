---
layout:  post
title:   "tee 指令"
date:    2013-04-17
tags:    ["CLI | 命令列介面", "Unix"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

第一次看到 tee 是在下面這個指令中：

```
:w !sudo tee %
```
	
這是一個在 vim 中的指令，使用時機是當你編輯檔案時不俱有 root 權限，但是在寫入時發現該檔案需要有 root 權限才可以寫入時。真的是非常實用的一個指令啊！！

當時一直不能理解這個指令到底為什麼可以運作，昨天看了朋友在寫題目時用的指令：

```
$ ./a.out < input_file | tee output_file
```
	
他說這樣他才能一邊看到 `./a.out < input_file` 的 standard output，也一邊將 `./a.out < input_file` 的 standard output 寫入 output_file 中。所以原來 tee 是可以將他收到的 standard input 以 standard output 輸出外，也將 standard input 輸出到檔案！

看了一下 [Wikipedia](https://zh.wikipedia.org/wiki/Tee) 發現，`tee` 的名字就來自於他的功能，因為他的運作方式畫出來就很像一個 T 字，故以此為名。

所以一開始提到的指令就是把 vim 的 buffer 作為 standard input 傳送給具有 root 權限的 tee，再寫到當前的檔案中，`%` 在 vim 的指令中會被取代為當前檔案名稱。

另外，tee 有兩個參數可以使用，分別是 `-a` 與 `-i`。`-a` 表示使用 append 的模式來寫入檔案，而 `-i` 則是忽略 SIGINT 的中斷。
