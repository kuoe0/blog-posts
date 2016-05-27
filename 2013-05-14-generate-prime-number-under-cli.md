---
layout: post
title:  "在 CLI 下產生質數表"
date:   2013-05-14
tags:   ["CLI | 命令列介面", "Unix", "prime | 質數"]
---

最近為了出作業需要 1~1000 中的所有質數，雖然用 C/C++ 寫個 [Sieve of Eratosthenes](http://blog.kuoe0.tw/posts/2009/10/23/prime-table-sieve-of-eratosthenes) 也是很快，不過還是覺得有點長，還要編譯很麻煩。（謎：那就用 python 啊！）

若是可以直接在 command line 下用 shell script 找出質數該有多好，於是 Google 了一下，找到了些資料，並修改了一下變成以下的程式碼：

```bash
#!/usr/bin/env bash
for i in `seq $1 $2`; do
	result=`factor $i | wc -w`;
	if [ $result == 2 ]; then
		echo $i;
	fi
done
```

找出 1~1000 中的質數的使用方式就是：

```bash
$ ./prime_gen 1 1000
```

`seq $1 $2` 就是列舉出 $1~$2 的數字，在以上的例子 $1 是 1 且 $2 是 1000。進去迴圈之後利用 `factor` 與 `wc` 指令，來解釋一下這兩個指令。首先是 `factor` 指令，該指令用來做質因數分解。以下是使用的範例：

```bash
$ factor 1000
1000: 2 2 2 5 5 5
$ factor 31211
31211: 23 23 59
$ factor 19
19: 19
$ factor 131
131: 131
```

看了看輸出，很明顯的質數一定只會有自己跟自己，並不會有其他數字。所以我們就可以利用 `wc` 這個指令，該指令用來計算文本的資訊，例如字數、byte 數以及行數等。根據 `factor` 輸出的資料可以發現質數一定只有兩個 word，所以可以把該結果利用 `wc -w` 來計算有幾個 word，`-w` 就是顯示有幾個 word。

所以最後在判斷 `wc -w` 的結果是不是 2 個 word 即可，如果結果是 2 個 word 就表示該數字是質數並輸出。
