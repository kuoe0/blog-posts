---
layout:  post
title:   "Arduino CLI on OS X"
date:    2013-11-20
tags:    ["Arduino", "OS X", "CLI | 命令列介面", "Ubuntu"]
feature:
    photo:       true
    creator:     
    url:         
    license:     
    license_url: 
---

由於研究需要自行設計硬體，聽說 Arduino 是個滿容易上手的硬體，所以決定就採用 Arduino 進行開發。要建立開發 Arduino 的環境還滿容易的，只要到[官方網站](http://arduino.cc/en/Main/Software)下載適合自己作業系統的 IDE 就好啦！但我還是習慣在命令列環境下進行開發，以下就是配置 CLI 環境的方法。Arduino 的 CLI 工具叫做 [Ino](http://inotool.org/)。

### Environment

- OS X 10.9 Mavericks
- homebrew 0.9.5
- Xcode 5.0.2

### Requirement

- Python 2.6+
- Arduino IDE
- picocom

首先，Python 2.6+ 在 OS X 本身就內建了，所以不用理會它，而 Arduino IDE 請到 Arduino [官方網站][1]下載並安裝。最後是 picocom，直接 `brew install picocom` 就會安裝好啦！

完成以上了必要條件後，如果你有用 `easy_install` 或是 `pip` 的話，直接輸入 `easy_install ino` 或是 `pip install ino` 即可。

Ino 官方有一份 [Quick Start](http://inotool.org/quickstart) 的文件可以參考，裡頭有基本指令的使用方式，如果想知道更詳盡的內容，可以在終端機中輸入指令，並在其後方加上 `--help` 的參數。

不過，在我安裝完 Ino 的環境後，按照 Quick Start 的步驟去執行時，發現 `ino upload` 一直發生錯誤，錯誤訊息如下：

```
$ ino upload 
Guessing serial port ... /dev/tty.usbserial-A10249RJ
/usr/local/opt/coreutils/libexec/gnubin/stty: invalid argument ‘-f’
Try '/usr/local/opt/coreutils/libexec/gnubin/stty --help' for more information.
stty failed
```

發現是在執行 `stty` 這個指令時，發現不合法的參數 `-f`，`man stty` 後發現的確 `stty` 並沒有 `-f` 這樣的參數，只有 `-F` 可以使用。原來是因為 OS X 內建的是 BSD 版本的 `stty`，該版本中確實是使用 `-f` 而非 `-F`。但是，我有安裝 `coreutils` 來取得 GNU 版本的工具，而且我也將該 GNU 版本的設定為預設使用的版本。而 Ino 有檢查作業系統型別來選擇加上不同的參數，所以當 Ino 發現作業系統是 OS X 時，就會選擇加上 `-f` 的參數。解決方法很簡單，直接去改 Ino 的原始碼就好啦！修改 `/usr/local/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages/ino/commands/upload.py` 這個檔案，並將 `file_switch = '-f' if platform.system() == 'Darwin' else '-F'` 改成 `file_switch = '-F'` 即可。

另外，可能還會發現在 upload 時一直出現以下的錯誤的話：

```
Guessing serial port ... /dev/tty.usbserial-A10249RJ
avrdude: stk500_getsync(): not in sync: resp=0x00
avrdude done.  Thank you.
```

```
Guessing serial port ... /dev/tty.usbserial-A10249RJ
avrdude: stk500_recv(): programmer is not responding
avrdude done.  Thank you.
```

那就是...硬體選錯了！！如果很幸運的剛好是用 Uno 這塊板子的話可能就不會遇到這問題，但像我是用 Nano 這塊，就被這問題卡很久。如果不是用 Uno 的話，在 build 以及 upload 時都要附上 `-m
nano328` 這樣的參數來告知硬體型別。而在 upload 與 serial 時，也要更改 serial port 的位置，加上 `-p /dev/tty.usbserial-A10249RJ` 這樣的參數。

如果懶惰的話，可以寫個 `ino.ini` 的檔案，並在內容寫入：

```
board-model = nano328
serial-port = /dev/tty.usbserial-A10249RJ
```

這樣所有 ino 指令都會預設加上 `-m nano328` 與 `-p /dev/tty.usbserial-A10249RJ` 這兩個參數了！`ino.ini` 檔案還有其他配置方式，在 Ino 的 [Quick Start][3] 可以找到。

最後，在進行 serial 通訊時，也就是 `ino serial` 指令。其實就只是 Ino 會直接呼叫 picocom 來進行連線這樣，所以操作指令都與 picocom 相同。一個比較常用的指令就是「離開」，只要按下「Ctrl+A] 後，再按下「Ctrl+X」即可離開。要注意，千萬不要在斷開 serial 連線之前就把 USB 接頭拔掉，不然會看到很精彩的六國當機畫面。我的電腦超少當機，但每次這樣拔掉就立馬當機...。

以上的安裝方法在 Ubuntu 也適用，而且在 Ubuntu 中不必特別去 Arduino 官網下載 IDE，可以直接透過 `apt-get install arduino` 就可以啦！
