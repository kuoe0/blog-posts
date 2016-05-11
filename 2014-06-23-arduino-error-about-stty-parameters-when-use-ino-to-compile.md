---
layout: post
title:  "Arduino 使用 Ino 進行編譯時出現 stty 參數錯誤"
date:   2014-06-23
tags:   ["Arduino", "OS X", "CLI | 命令列介面"]
feature:
    photo: false
---

> 作業系統：OS X 10.9 Mavericks

相信應該有些人在 Mac 上用 Ino 時也出現過以下訊息：

```
ino upload -m uno
/usr/local/opt/coreutils/libexec/gnubin/stty: invalid argument ‘-f’
Try '/usr/local/opt/coreutils/libexec/gnubin/stty --help' for more information.
Guessing serial port ... /dev/tty.usbmodem1421
stty failed
```

不過，應該有些人不會有這樣的問題。這全根據系統中是否安裝了 GNU coreutils，並且在環境變數 `$PATH` 中將 GNU coreutils 的路徑放置於 `/bin` 之前。假使沒有將 GNU coreutils 的路徑放置於 `/bin` 之前，那麼也不會產稱該問題。

而這個問題產生的原因主要是 OS X 的 `stty` 與 GNU coreutils 中的 `stty` 參數並不相容。在 OS X 版本的 `stty` 要指定裝置需要的是 `-f` 參數；然而在 GNU coreutils 中的 `stty` 則是用 `-F` 或是 `--file` 來指定裝置。

因為還是比較習慣 GNU coreutils 內的工具，不打算重新讓 `/bin` 的位置跑到 GNU coreutils 的路徑之前。索性只好直接去看這使用到 `stty` 的程式碼。

我是用 `pip install ino` 進行安裝的，預設的安裝位置如下：

```
/usr/local/Cellar/python/2.7.7_1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages/ino
```

用 grep 搜尋了一下發現用到 stty 的檔案就是 commands/upload.py。進去找後一下 `-f` 參數到底在哪裡設置的，直接把它改成 `-F` 就可以了！
