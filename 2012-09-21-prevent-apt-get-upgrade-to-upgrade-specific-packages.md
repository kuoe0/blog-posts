---
layout:  post
title:   "防止 apt-get upgrade 升級特定套件"
date:    2012-09-21
tags:    ["Ubuntu", "Linux"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

APT 是 Debian 下的套件管理器，Ubuntu 衍生自 Debian，因此自然而然的也繼承了 APT 這個套件管理器。APT 可以幫助我們下載、安裝或升級套件等等，這些就不在此討論了。

在 Ubuntu 中，當我們要更新所有套件時，我們會下 `apt-get upgrade` 的指令。但有時候某些套件我們會有自己的編譯需求或是不想更新至新版本，例如下不同參數、更改些內容或是正值開發中套件不能隨意更新等等。此時該套件如果有在 APT 的套件庫中，每當下達更新的指令時，就會造成我們自行編譯或是不願更新的套件被套件庫中的最新版本覆蓋。因此我們就需要告訴 APT 哪些套件不要更新。

## 方法一

利用 aptitude，輸入指令 `aptitude` 可進入 aptitude 的 UI，`ctrl+T` 可操作選單，並使用搜尋功能找出要保留的套件名稱。

並選擇至該套件上，輸入 `=` ，用以表示將此套件 hold 住（保留）！如此一來，該套件在未來使用 `apt-get upgrade` 時將不再被更新了！

可以直接輸入 `/` 來快速啟動搜尋功能！可在選單中，找到更多功能的快捷鍵，上述用來設定 hold 的 `=` 也是快捷鍵之一！

另外，也可使用 dselect 使用類似的方法！

## 方法二

首先輸入 `dpkg --get-selections > packages`，該指令會將當前套件庫的設定資訊匯出至 packages 這個文字檔中。

接下來在以編輯器打開 packages 進行編輯。將會看到如類似下面的資訊：

```
package1 install
package2 install
…
packageN install
```
	
找到不希望被更新的套件名稱，並將該套件庫後面所接著的 **install** 替換成 **hold** 並存檔離開。

接著再輸入指令 `dpkg --set-selections < packages`。如此一來，對於該套件的設定就完成了！

dpkg 是 APT 的底層工具，因此我們可以直接使用他來進行設定。上列的方法中，會看到 `--get-selections` 這個參數，表示將當前的套件庫狀態輸出。而 `--set-selections` 則是將輸入的設定寫進套件庫。

另外，在網路上有找到更簡潔的寫法：

`echo "<packagename> hold" | dpkg --set-selections`

基本上就是先將套件的新狀態輸出並導向至 dpkg！


