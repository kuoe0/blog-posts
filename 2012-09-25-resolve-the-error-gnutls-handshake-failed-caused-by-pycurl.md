---
layout: post
title:  "解決 pycurl 造成的 gnutls_handshake() failed 錯誤"
date:   2012-09-25
tags:   ["Ubuntu", "Linux", "LinuxMint", "GnuTLS", "OpenSSL"]
---

在 Ubuntu 上，常常會使用 APT 來作管理，我個人也比較喜歡有個工具來管理套件，APT 上找不到或是有特殊需求才自己編。但官方不見得會收入所有套件，所以通常會在套件庫中增加他人的 PPA (Personal Package Archives) 以方便安裝、移除或更新他人維護的套件。

因此在每次重灌電腦後，新增 PPA 也變成必要的工作之一。這學期開始研究生的旅程，實驗室也開始有了自己的位置與桌機，灌成 Ubuntu 是首要工作。於是就遇上了新增 PPA 時，得到 `gnutls_handshake() failed: A TLS packet with unexpected length was received.` 錯誤。

```bash
$ sudo apt-add-repository ppa:mitya57
Traceback (most recent call last):
  File "/usr/bin/apt-add-repository", line 125, in <module>
    ppa_info = get_ppa_info_from_lp(user, ppa_name)
  File "/usr/lib/python2.7/dist-packages/softwareproperties/ppa.py", line 80, in get_ppa_info_from_lp
    curl.perform()
pycurl.error: (35, 'gnutls_handshake() failed: A TLS packet with unexpected length was received.')
```

到 Google 上搜尋可以看到滿多人討論該問題，搜尋的結果似乎較多人是在使用 git 時所遭遇到的。所以我想 git 應該也有相同問題，但 git 上的 error 是由 git-core 所引起的，而 apt-add-repository 則是由 pycurl 所造成。

因為 Ubuntu 上預設的 SSL 是使用 GnuTLS，GnuTLS 似乎在某些網路架構下造成 TLS 封包長度異常。難怪我以前在家中（中華電信 ASDL/光世代）都沒遇過該問題，在學校就出現這問題。據說政大也有相同問題，然而交大則沒有該問題，大概是成大跟政大的網路上不知道加了層什麼鬼東西！！

## 解決方法

該問題的解決方式為－令 pycurl 改使用 OpenSSL。因為 Ubuntu 套件庫中使用的都是 GnuTLS，所以只能自己編譯改用 OpenSSL 版本的 pycurl。

首先先來確定目前安裝的 pycurl 版本，按照以下步驟即可，並可以發現當前的版本使用的是 GnuTLS。

```
$ python
Python 2.7.3 (default, Aug  1 2012, 05:14:39)
[GCC 4.6.3] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import pycurl
>>> pycurl.version
'libcurl/7.22.0 GnuTLS/2.12.14 zlib/1.2.3.4 libidn/1.23 librtmp/2.3'
```

**使用 OpenSSL 編譯 pycurl**

以下為我撰寫的 shell script  可以自動進行替換安裝：

<script src="https://gist.github.com/KuoE0/6192061.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/6192061).

上面的步驟中，其中有一步為 `apt-get source python-pycurl`，該步驟為獲取 pycurl 的原始碼。由於我使用的 distribution 是 [LinuxMint 13](http://linuxmint.com/)，預設的 repositories 中並沒有擷取 source 的 URI，於是只能手動增加。開啟 /etc/apt/sources.list 檔案，並加入以下這行。在執行 `apt-get update` 更新 APT 套件庫，就可以順利獲取 pycurl 的原始碼。

```
deb-src http://tw.archive.ubuntu.com/ubuntu/ precise main
```

再檢查一次 pycurl 版本，就可以看到當前 pycurl 已改成使用 OpenSSL。

```
$ python
Python 2.7.3 (default, Aug  1 2012, 05:14:39)
[GCC 4.6.3] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import pycurl
>>> pycurl.version
'libcurl/7.22.0 OpenSSL/1.0.1 zlib/1.2.3.4 libidn/1.23 librtmp/2.3'
```

另外，`apt-get upgrade` 會造成我們自行編譯的 pycurl 重新被 GnuTLS 版本的覆蓋。因此我們需要告知 APT 不要升級當前版本的 pycurl。方法可以參考「[防止 apt-get upgrade 升級特定套件](http://blog.kuoe0.tw/posts/2012/09/21/prevent-apt-get-upgrade-to-upgrade-specific-packages)」該文章。

```bash
echo "python-pycurl hold" | sudo dpkg --set-selections
echo "python-pycurl-dbg hold" | sudo dpkg --set-selections
```

前面提到 git 也可能會遇到該問題，解決的方法也是令 git 使用 OpenSSL 重新編譯 git 即可。
