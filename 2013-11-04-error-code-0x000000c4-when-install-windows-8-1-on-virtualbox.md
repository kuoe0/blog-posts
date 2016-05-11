---
layout: post
title:  "在 VirtualBox 上安裝 Windows 8.1 時出現錯誤碼 0x000000C4"
date:   2013-11-04
tags:   ["virtual machine | 虛擬機", "VirtualBox", "Windows"]
feature:
    photo: false
---

最近接了一個別人未完成的案子，由於目標終端是 Windows，因此勢必得安裝個 Windows 了。但又不想直接灌在系統裡，想說裝個虛擬機好了。發現學校有最新的 Windows 8.1 的映像檔，就決定來潮一下安裝最新的 Windows 8.1！結果一開始安裝就出現了以下的錯誤訊息：

```
Your PC needs to restart.
Please hold down the power button.
Error code: 0x000000C4
Parameters:
0×0000000000000091
0x000000000000000F
0xFFFFF801E5962A80
0×0000000000000000
```

以下是我的環境：

- Host OS: Ubuntu 13.04
- VM software: VirtualBox 4.2.10
- Client OS: Windows 8.1

## 解決方法

首先先輸入以下指令，查看虛擬機名稱：

```
$ VBoxManage list vms
"Win8.1" {feb46273-626f-47fa-9594-89a7fbe3a224}
```

接著記下虛擬機名稱後，在套用以下指令：

```
$ VBoxManage setextradata "Win8.1" VBoxInternal/CPUM/CMPXCHG16B 1
```

而這個指令到底在幹嘛呢？其目的是啟動該虛擬機對於 CMPXCHG16B 指令集的支援，在 Windows 8.1 中，微軟除了改進了不少性能外，也增加了 Windows 8.1 的配備需求，其中一點就是需要支援 CMPXCHG16B 指令集！

因此可能會有些老舊機器因為 CPU 本身不支援該指令集而導致無法升級到 Windows 8.1！（[相關鏈結](http://www.neowin.net/news/microsoft-confirms-some-older-amd-processors-do-not-support-windows-81)）
