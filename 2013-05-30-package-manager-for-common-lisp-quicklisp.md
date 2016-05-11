---
layout: post
title:  "Common Lisp 套件管理員 - Quicklisp"
date:   2013-05-30
tags:   ["Common Lisp", "Quicklisp"]
feature:
    photo: false
---

Quicklisp 是 Common Lisp 的第三方套件管理套件，就像 Python 的 pip 或是 Ruby 的 gem 一樣，可以利用它安裝該語言豐富的第三方套件。可以到官方網站 ([連結](http://www.quicklisp.org/beta/releases.html)) 查看目前支援的套件有哪些！

不過目前 Quicklisp 還是 beta 版，不過我目前使用上還沒遇到什麼大問題。不過有點討厭的是在讀取套件時的輸出竟然是輸出到 standard output，導致我寫個程式時，還需要把他的輸出導出到其他地方，例如：/dev/null。

目前可以運行在以下的 Common Lisp 實現：

- ABCL
- Allegro CL
- Clozure CL
- CLISP
- CMUCL
- ECL
- LispWorks
- SBCL
- Scieneer CL

不過我個人是採用 [SBCL](http://www.sbcl.org)，所以接下來的內容還是以 sbcl 為主！

## 安裝 Quicklisp

下載 [http://beta.quicklisp.org/quicklisp.lisp](http://beta.quicklisp.org/quicklisp.lisp) 該檔案，接著再開啓 sbcl 並 load 該檔案。輸入 `sbcl --load quicklisp.lisp` 或是進入 sbcl 的 REPL 後在輸入 `(load "quicklisp.lisp")` 來進行讀取。

如果使用的作業是系統是 Debian 的衍生版的話（例如：Debian、Ubuntu 或 LinuxMint 等等），可以使用 `apt-get install cl-quicklisp` 來進行安裝！安裝後會提示請參考 /usr/share/doc/cl-quicklisp/README.Debian 該檔案來進行安裝，裡面其實就是說明透過 APT 安裝後的 quicklisp.lisp 檔案的位置在 /usr/share/cl-quicklisp/quicklisp.lisp。所以一樣用前面提到的方式來讀入 quicklisp.lisp 檔案就可以了，只需要換一下檔案名稱！

成功讀取 quicklisp.lisp 後，就可以進行安裝了，只要在 sbcl 的 REPL 中輸入 `(quicklisp-quickstart:install)` 就會開始進行下載相關內容並安裝。這時候你會發現你的家目錄出現了一個 quicklisp 的資料夾，所有 Quicklisp 的內容都會放置在該資料夾，包括安裝的套件庫！

如果不想要直接安裝在 ~/quicklisp 這個位置的話，也可以自行指定安裝位置，像我就比較希望他是個隱藏的資料夾，所以我會希望它安裝到 ~/.quicklisp 這個位置。因此我只要把原本的安裝指令中加入 `:path` 這個 keyword，並把欲安裝的位置傳入即可，指令就會變成這樣：`(quicklisp-quickstart:install :path "~/.quicklisp/")`。**注意：路徑的結尾一定要有斜線！否則，quicklisp 會被安裝到家目錄，而且是所有檔案都分散在家目錄！**

安裝後，如果要使用 Quicklisp 的話，每次在執行 sbcl 時，我們都需要載入 `~/quicklisp/setup.lisp` 該檔案，一樣可以直接用 --load 進行讀取，或是進入 sbcl 的 REPL 後再輸入 load 敘述。

不過，每次開啓就要讀一次豈不是很麻煩！！因此我們可以把載入 Quicklisp 的指令加入我們的 user initialization 檔案中，在家目錄中找到 .sbclrc 檔案後，加入一行 `(load "~/quickload/setup.lisp")`，之後每次執行 sbcl 的 REPL 就會自動讀取 Quicklisp 了！另外我們也可以在 sbcl 的 REPL 中輸入 `(ql:add-to-init-file)` 指令，如此以來就會自動幫我們在 .sbclrc 中加入自動讀取 Quicklisp 的指令了！

但要注意的是，如果在執行 sbcl 時，有下 `--no-userinit` 參數的話，Quicklisp 就不會被讀取進來！或是說有些參數也會包括有 `--no-userinit` 的功能的話，Quicklisp 也不會被讀取進來，例如：`--script`。

所以如果要利用 `--script` 來執行程式的話，就要手動載入 Quicklisp 了，一樣可以使用 `--load ~/quicklisp/setup.lisp` 來進行加載，或是直接在 script 中加入 `(load "~/quicklisp/setup.lisp")` 敘述即可。

## 安裝套件

```common-lisp
To load a system, use: (ql:quickload "system-name")
```

以上是讀取套件的指令，用 system 這詞還真讓我有點不懂他的意思…。不過，這裡的 system 就是 package/library 的意思啦！所以我們如果要 load 套件進來，只要輸入 `(ql:quick load "package_name")`，例如我要安裝 drakma 這個套件，就只要在 sbcl 的 REPL 中輸入 `(ql:quickload "drakma")` 即可。

他會出現以下訊息，並在最後回傳包含該套件名稱的 list。（是說這個輸出其實有點討厭！）不過，如果是第一次讀入該套件的話，由於電腦中還沒有該套件存在，所以會先出現下載的訊息。

```common-lisp
To load "drakma":
  Load 1 ASDF system:
    drakma
; Loading "drakma"
.......
("drakma")
```

看他的回傳值感覺應該可以一次讀取多個套件才是，不然似乎設計成回傳 list 沒有多大的意義？！於是查看了一下程式碼，發現有以下幾行：

```common-lisp
(unless (consp systems)
    (setf systems (list systems)))
(dolist (thing systems systems)
    …
```

quickload 的說明中也有寫到：

> SYSTEMS is a designator for a list of things to be loaded.

因此可以確定可以直接傳入一個套件名稱的 list，所以我們可以用 `(ql:quickload '("drakma" "cl-html-parse"))` 來一次讀入兩個套件。不過要注意的是 list 前面的 quote，不然就用 `(list "drakma" "cl-html-parse")` 來代替也可以！另外，除了用字串的方式來傳入套件名稱外，也可以利用 keyword 的方式來傳入，例如：`(ql:quickload :drakma)`。

## 搜尋套件

```common-lisp
To find systems, use: (ql:system-apropos "term")
```

以上指令就是用來搜尋套件庫的，他會搜尋所有套件名稱含有 term 的套件。例如我要搜尋含有 "regex" 的套件，就只要輸入 `(ql:system-apropos "regex")` 就會回傳以下資訊：

```common-lisp
#<SYSTEM com.informatimago.common-lisp.regexp / com.informatimago-20130312-git / quicklisp 2013-04-20>
#<SYSTEM lispbuilder-regex / lispbuilder-20130312-svn / quicklisp 2013-04-20>
#<SYSTEM recursive-regex / recursive-regex-20120407-git / quicklisp 2013-04-20>
#<SYSTEM recursive-regex-test / recursive-regex-20120407-git / quicklisp 2013-04-20>
#<SYSTEM regex / regex-20120909-git / quicklisp 2013-04-20>
```

查出有哪些相關的套件後，就可以依照自己喜好選擇要安裝哪個囉！安裝方式就跟前面提到的一樣！

## 其它

更新所有套件：`(ql:update-all-dists)`

更新 Quicklisp：`(ql:update-client)`

查詢哪些套件相依於某套件：`(ql:who-depends-on "package-name")`

**編修紀錄**

- 2013-05-31: 增加 Debian 衍生版安裝方式
- 2013-05-31: 自訂安裝位置注意結尾的斜線 (slash)
