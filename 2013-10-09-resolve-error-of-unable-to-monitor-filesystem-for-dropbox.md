---
layout:  post
title:   "解決 Dropbox 的錯誤訊息「Unable to monitor filesystem」"
date:    2013-10-09
tags:    ["Dropbox", "Ubuntu", "Linux"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

最近 Ubuntu 上的 Dropbox 都會莫名會跳出一個通知顯示：

> **Unable to monitor filesystem**
>
> Please run "echo 100000|sudo tee /proc/sys/fs/inotify/max_user_watches" and restart Dropbox to correct the problem.

出現這個錯誤表示，Dropbox 無法監控你所有的要同步的檔案，在 Dropbox 的預設設定中，只能監控 8192 個資料夾。檢查一下 `/proc/sys/fs/inotify/max_user_watches` 這個檔案內容：

```
$ cat /proc/sys/fs/inotify/max_user_watches
8192
```

由於很怕資料消失，加上 SSD 哪天會出包實在不敢不做備份，但本身又很懶惰，所以幾乎什麼都放在 Dropbox 了…。利用 `find ~/Dropbox -type d | wc -l` 發現我有 10406 個資料夾，而出現這個錯誤表示 Dropbox 只會選擇 8192 個資料夾進行同步，其餘的將被忽略，因而「**不會進行同步**」。這太可怕了，還是儘快處理得好！

解決方式只要按照提示的步驟，到終端機打上：

```
$ echo 100000 | sudo tee /proc/sys/fs/inotify/max_user_watches
```

並重新啓動 Dropbox，重新啓動的方法只要在狀態列上 Dropbox 的圖示上按右鍵選擇離開，接著在到 dash board 上找到 Dropbox 開啓即可。

不過，這只是暫時的解決方案，只要重新開機的話，又會變成 8192。所以這個方法每次重新開機都要重做一次！

一勞永逸的方法就是寫進系統開機的設定，因此我們需要修改 `/etc/sysctl.conf` 該檔案，並找到 `fs.inotify.max_user_watches` 這行，將後面的數字改成 `100000` 後儲存離開，如果沒找到這行，那麼就在該檔案內加入一行：

```
fs.inotify.max_user_watches = 100000
```

重新開機後就不會再出現錯誤訊息了！不過，如果哪天資料夾超過 100000 可能就在再改一次了！但要超過 100000 也蠻誇張的…。
