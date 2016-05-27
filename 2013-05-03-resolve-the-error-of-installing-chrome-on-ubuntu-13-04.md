---
layout: post
title:  "解決 Ubuntu 13.04 上無法安裝 Google Chrome"
date:   2013-05-03
tags:   ["Ubuntu", "Chrome", "Linux"]
---

由於習慣了 Dock 的操作方式，最近決定從 Linux Mint 跳回 Ubuntu，剛好最近也發佈了 13.04 版，於是就安裝了這熱騰騰的版本！本身使用的瀏覽器是 Chrome，不過 Ubuntu 內建的是 Firefox，想當然爾，勢必要立馬安裝 Chorom！

首先必須要到 Chrome 的網頁上下載 Chrome 的 deb 安裝檔，由於我使用的是 64 位元版本的 Ubuntu，於是我下載的是 google-chrome-stable_current_amd64.deb。

點擊開啓後，就會發現 Ubuntu 的軟體中心告訴你相依的套件 libudev0 沒有正確安裝。我個人習慣在 CLI 下進行安裝，輸入以下指令即可安裝 deb 安裝檔：

```bash
$ sudo dpkg -i google-chrome-stable_current_amd64.deb
```

然後，就會看到錯誤訊息如下（僅截取部分）：

```bash
dpkg: dependency problems prevent configuration of google-chrome-stable:
 google-chrome-stable depends on libgconf2-4 (>= 2.27.0); however:
  Package libgconf2-4 is not installed.
 google-chrome-stable depends on libudev0 (>= 147); however:
  Package libudev0 is not installed.
…
Errors were encountered while processing:
 google-chrome-stable
```

這告訴了我們，Chrome 的相依的套件 libudev0 沒有正確安裝。解決方法如下：

首先，將 libudev0 的來源加入 APT 的 source list 中。編輯 `/etc/apt/sources.list` 檔案，加入 `deb http://ubuntu.mirror.cambrium.nl/ubuntu/ quantal main` 這行。

接下來執行 `apt-get update` 來更新套件庫資訊，更新後再執行 `apt-get -f install` 來修復套間的相依關係。再來就可以安裝 libudev0 套件，輸入 `apt-get install libudev0`。

這時，libudev0 就已經安裝成功了，再一次安裝 Chrome 的 deb 檔案即可！

完整流程如下：

```bash
$ echo "deb http://ubuntu.mirror.cambrium.nl/ubuntu/ quantal main" | sudo tee -a /etc/apt/sources.list
$ sudo apt-get update
$ sudo apt-get -f install
$ sudo apt-get install libudev0
$ sudo dpkg -i google-chrome-stable_current_amd64.deb
```
