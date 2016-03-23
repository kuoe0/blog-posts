---
layout:  post
title:   "替 SBCL 增加 autocomplete 功能 - Linedit"
date:    2013-06-16
tags:    ["Common Lisp", "SBCL", "autocomplete | 自動補齊"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

在 CLI 環境下，很多指令都只能熟記，不過指令真的太多了，要通通記得實在有點麻煩，所以 autocomplete 就顯得非常重要啦！在 bash、zsh 等 shell 中，都有著強大 autocomplete 功能，以方便我們找到要使用的指令名稱。而 vim 中，許多人也會安裝各式各樣的套件來幫助自己更快速也更正確地（電腦不會手殘）編寫程式。另外，在 python 中，也有個 [bpython](http://bpython-interpreter.org/) 的項目，讓工程師們可以更快速的使用 python 來工作！因此，我覺得所有的 interactive environment 應該都要有 autocomplete 的功能才是啊！！

於是我就 Google 了一下，看看 SBCL 是否也有相關的功能！沒想到還真的給我找到了，就是 Linedit！其放置於 github 上專案：[nikodemus / linedit](https://github.com/nikodemus/linedit)。看起來也一陣子沒更新了，不過還是可以正常使用！

來看看他的簡介：

> Linedit is a readline-style library written in Common Lisp that provides customizable line-editing for Common Lisp programs.

先前的文章提過 Common LISP 的套件管理員－[Quicklsip](http://blog.kuoe0.tw/posts/2013/05/30/package-manager-for-common-lisp-quicklisp)，Linedit 一樣可以透過 Quicklisp 安裝！打開 SBCL 後，輸入 `(ql:quickload "linedit")` 就會把 Linedit 的套件下載下來！

為了上我們每次打開 SBCL 都會自動載入 Linedit，需要在 ~/.sbclrc 中加入以下內容：

```common-lisp
;;; Check for --no-linedit command-line option.
(if (member "--no-linedit" sb-ext:*posix-argv* :test 'equal)
	(setf sb-ext:*posix-argv* 
		(remove "--no-linedit" sb-ext:*posix-argv* :test 'equal))
	(when (interactive-stream-p *terminal-io*)
		(require :sb-aclrepl)
		(require :linedit)
		(funcall (intern "INSTALL-REPL" :linedit) :wrap-current t)))
```

如此以來，我們每次打該 SBCL 的 REPL 就會自動載入 Linedit 了！載入後會發現，SBCL 的 prompt 改變了，變成了 `CL-USER(1):`，並且該數字會隨著輸入的敘述增加而遞增！接著我們來試試 autocomplete 的功能，輸入 `(ex` 接著按下 tab 鍵。這時候 SBCL 就會跳出以下畫面：

```common-lisp
CL-USER(1): (ex
exit    extern-alien    extended-char    exp    expt    export
CL-USER(1): (ex
```

代表 ex 開頭的函式有這些，我們再進一步的輸入一個 `i` 並按下 tab 鍵，就會發現 Linedit 自動幫我們補齊了 `t`，這表示 exi 開頭的函式就只有 exit 了，所以就會自動補齊了！

不過要使用 autocomplete 也是要付出些代價的，就是每次 SBCL 開啓的時間將會增加，因為 Linedit 需要每次都重新載入所有函式庫，所以需要花點時間！如果不希望開啟 SBCL 時載入 Linedit 的話，只要在啟動 SBCL 時加入一項參數 `--no-linedit` 即可！但如果事後又想要手動載入的話，就依序執行以下敘述：

```common-lisp
(require :sb-aclrepl)
(require :linedit)
(funcall (intern "INSTALL-REPL" :linedit))
```
