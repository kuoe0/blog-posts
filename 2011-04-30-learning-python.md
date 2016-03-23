---
layout:  post
title:   "Learning Python"
date:    2011-04-30
tags:    ["Python", "Google"]
feature:
    photo:       true
    creator:     
    url:         
    license:     
    license_url: 
---

看到 Python 是 Google 使用的主要語言之一，因為夢想進入Google，因此下定決心學習 Python。不管是否進得去 Google，學的越多，我就比別人擁有越多的優勢。前幾天就開始找 Python 的教學網站，也因此看到了一些其他人對 Python 的看法。大部分對 Python 的評價都是簡單好學，而且可以提升工作效率（非程式執行效率），所以都會選擇 Python 當作撰寫非要求效率的程式
（要求效率還是用C吧），這也更吸引我去學 Python 了！

Python 跟以往我學的程式語言比較不同的是，它屬於[直譯式語言 (interpreted language)](http://en.wikipedia.org/wiki/Interpreted_language)，以往學的 C/C++ 與 Java 都是[編譯式語言 (compiled language)](http://en.wikipedia.org/wiki/Compiled_language)，需要藉由 compiler 將程式碼轉換為可執行檔案。

直譯式語言靠的是直譯器，直譯器可以動態的直譯程式碼，也就是直接執行程式碼。其實我們很常接觸直譯式語言，最常接觸的大概就是網頁語言（包括你現在看得這個部落格），HTML、PHP、JavaScript 都是直譯式語言，其他還有 Lisp（修程式語言好像碰過一點），Ruby（我之前跑資訊營的包裝名），shell script（在 Linux 下 shell 的指令）等都是，說到 shell 讓我想到我一直想把 bash 換成 zsh 看看，反正 bash 也還不熟，直接用 zsh 好了。

要學 Python，就得先安裝 Python，官網可以下載（有各 OS 的安裝檔案）。由於 Python 現在主要分成 2.x 與 3.x 兩種版本，3.x 竟然無法向下相容 2.x 版本，語法不同了，目前知道 print 變成一個 function 的樣子。現今大部分 Python 程式應該都是 2.x 版的吧，但我想漸漸的 3.x 版還是會變主流，所以我就決定下載 3.2 版了。

安裝後，就可以開始寫Python了。但要寫程式，總是需要的IDE（其實也不一定啦，最近有點常用 notepad++ 寫程式），你可以在程式集裡面找到 Python 的資料夾，裡面有個 IDLE 程式，這個就是官方提供的 IDE，這個式圖形化介面的，另外還有 command line 可以執行。這讓你可以直接在輸入 Python 的語法，並直接執行這段語法。或是將程式寫在 .py 副檔名的檔案中，以 python.exe 執行，就可以像用 C 寫出來的 .exe 檔一樣的方式執行了。但要注意的是，.py 檔案只可以在有安裝 Python 的平台上執行，若要在沒有安裝 Python 的狀況下執行，需要另外將它轉換為可執行檔。目前查到 [PyInstaller](http://www.pyinstaller.org/) 跟 [y2exe](http://www.py2exe.org/) 都是不錯的軟體，而且 PyInstaller 似乎不限平台！？有空再來研究。

今天只學的一點點皮毛兒，大概就基本的變數宣告（其實不用宣告）、print 的一點點皮毛、整數的運算（四則運算加上 \*\* 是指數運算，還有可以直接用大數）、字串的運算（+ 跟 * 太好用了吧）以及 for 與 while 迴圈的使用。等熟一點大概會想試試看拿來寫 ACM 吧，SPOJ 可以支援 Python 真是太棒了。

目前學 Python 都是靠這個網站「[什麼是Python? Why Python?](http://ez2learn.com/index.php/python-tutorials)」，雖然他的教學是用2.x版的，但其實也大同小異啦。所以也另外參考了這邊文章「[Python 3 初探，第 1 部分: Python 3 的新特性](http://www.ibm.com/developerworks/cn/linux/l-python3-1/)」，當作發生錯誤時的查詢用的 reference。

另外今天還看到了 [Google Coding Style Guide](http://code.google.com/p/google-styleguide/)，這裡面有 Google 在開發專案時的規範，看來有空應該要看看，這樣才能更接近 Google 一步！裡面發現只有四個語言，分別是 C++、Java、Python 以及 Objective-C，看來這四個語言在Google算是比較常用的語。真是太剛好，因為以後想學 Mac OS 及 iOS 的程式開發，Objective-C 就是其使用的語言，所以他本來就是我想學的語言之一了。
