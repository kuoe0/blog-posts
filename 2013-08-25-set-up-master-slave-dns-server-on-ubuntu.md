---
layout:  post
title:   "在 Ubuntu 上架設主從式 DNS server"
date:    2013-08-25
tags:	   ["Ubuntu", "Linux", "DNS", "network management | 網路管理"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

繼上次架設了實驗室的 DNS server 後 ([連結](http://blog.kuoe0.tw/posts/2013/08/11/install-dns-server-on-ubuntu
))，透過一些線上的檢測服務發現只有一台 DNS server 是不夠穩定的。少說也要兩台，這樣才能避免那台 server 因故停止服務而導致網址無法解析到正確的 IP，也因此決定架設第二台 DNS server。

不過直接做一次一樣的設定好像有點蠢，而且這樣等於每次更新 DNS 記錄時都得要更新兩台，如果哪天 DNS server 有 N 台，就得登入 N 台伺服器進行修改，這樣太笨了！還有 BIND 有支援主從式 (master-slave) 架構，如此以來我們可以設定一台 master DNS server，而其他的 slave DNS server 會定期的與 master DNS server 進行資料同步！

繼續上一篇的設定，以下為我當前的環境：

- Ubuntu Server 12.04 LTS
- BIND 9.8.1

以下為假設：

- master DNS IP address: 140.116.55.66
- slave DNS IP address: 140.116.22.66

對於 slave 的 **/etc/bind/named.conf.option** 檔案設定，只需要設定的跟 master 的 **/etc/bind/named.conf.option** 一樣就好了！唯一需要修改的只有兩邊的 **/etc/bind/named.conf.local** 檔案。


## [Master] `/etc/bind/named.conf.local`

在需要同步的 zone 區域內加上 `allow-transfer` 與 `also-notify` 以及 `notify yes` 的設定，實際設定如下：

```
zone "5566.csie.ncku.edu.tw" in {
    type master;
    file "/etc/bind/db.5566.csie.ncku.edu.tw";
    allow-transfer {
        140.116.22.66;
    };
    also-notify {
        140.116.22.66;
    };
    notify yes;
};
```

`allow-transfer`：使得 slave 可以主動的向 master 複製 DNS record，並且只有定義中的 IP 列表可以向這台 DNS server 要求同步。

`also-notify`：slave 通常會定時地向 master 進行資料同步，所以有可能發生 master 與 slave 的資料不同步的情況。而 DNS 在進行解析時，並不會知道誰是 master 誰是 slave，所以如果解析從 slave 進行，而資料還未同步的話就會發生解析不正確的狀況。而這個 `also-notify` 就是用來解決這個狀況的選項，當 DNS record 有更新時，該主機就會自動通知列於表中的 IP，使得 slave 被動地同步資料。

`notify yes`：這個是用來設定開啓 notity 的功能，若是設定為 `notify no`，則不會進行通知 slave 的動作！

## [Slave] `/etc/bind/named.conf.local`

再來是 slave 的 **/etc/bind/named.conf.local** 檔案設定。Slave 上的設定也很簡單，只需要將 `type` 設定為 `slave`，並且指定 zone 檔案的存放位置以及 master 的 IP 即可。實際範例如下：

```
zone "5566.csie.ncku.edu.tw" in {
        type slave;
        file "/var/cache/bind/slave/        db.5566.csie.ncku.edu.tw";
        masters {
                140.116.55.66;
        };
};
```

都完成設定後，將兩台機器的 BIND 都重新啓動即可，皆輸入以下指令：

```
$ sudo service bind9 restart
waiting for pid 24891 to die
                                          [ OK ]
 * Starting domain name service... bind9  [ OK ]
```


完成後可以檢查一下 slave 的 syslog 看看有沒有錯誤發生：

```
root@MasterDNS:/etc/bind$ tail /var/log/syslog
Aug 24 02:07:16 SlaveDNS named[18779]: zone 55.116.140.in-addr.arpa/IN: Transfer started.
Aug 24 02:07:16 SlaveDNS named[18779]: transfer of '55.116.140.in-addr.arpa/IN' from 140.116.55.66#53: connected using 140.116.22.66#36959
Aug 24 02:07:16 SlaveDNS named[18779]: zone 55.116.140.in-addr.arpa/IN: transferred serial 2013082401
Aug 24 02:07:16 SlaveDNS named[18779]: transfer of '55.116.140.in-addr.arpa/IN' from 140.116.55.66#53: Transfer completed: 1 messages, 7 records, 251 bytes, 0.001 secs (251000 bytes/sec)
Aug 24 02:07:16 SlaveDNS named[18779]: zone 55.116.140.in-addr.arpa/IN: sending notifies (serial 2013082401)
Aug 24 02:07:17 SlaveDNS named[18779]: zone 5566.csie.ncku.edu.tw/IN: Transfer started.
Aug 24 02:07:17 SlaveDNS named[18779]: transfer of '5566.csie.ncku.edu.tw/IN' from 140.116.55.66#53: connected using 140.116.22.66#58793
Aug 24 02:07:17 SlaveDNS named[18779]: zone 5566.csie.ncku.edu.tw/IN: transferred serial 2013082401
Aug 24 02:07:17 SlaveDNS named[18779]: transfer of '5566.csie.ncku.edu.tw/IN' from 140.116.55.66#53: Transfer completed: 1 messages, 24 records, 633 bytes, 0.001 secs (633000 bytes/sec)
Aug 24 02:07:17 SlaveDNS named[18779]: zone 5566.csie.ncku.edu.tw/IN: sending notifies (serial 2013082401)
```

並檢查一下設定存放的位置是否有出現，如果有正常出現，檔案內容應該會與 master DNS server 上的一模一樣！這樣就完成 master-slave 的 DNS server 架構了！

如果放置 zone 檔案的位置與我的不同，而且發現檔案始終沒有成功同步的話，請檢查一下 syslog 看看！如果出現 `file not found` 的話，表示沒有該資料夾存在，就自己創建一個吧！而如果是出現 `permission denied` 的話，表示 bind 沒有權限可以寫入該檔案！這時去修改一下 `/etc/apparmor.d/usr.sbin.named` 這個檔案。打開該檔案可以看到類似下面的內容：

```
 # /etc/bind should be read-only for bind
 # /var/lib/bind is for dynamically updated zone (and journal) files.
 # /var/cache/bind is for slave/stub data, since we're not the origin of it.
 # See /usr/share/doc/bind9/README.Debian.gz
  /etc/bind/** r,
  /var/lib/bind/** rw,
  /var/lib/bind/ rw,
  /var/cache/bind/** rw,
  /var/cache/bind/ rw,
```

將你要放置的檔案的資料夾加入，並在後面加上 rw 給予讀寫權限即可解決 `permission denied` 的問題！


**參考資料：**

- [DNS BIND Zone Transfers and Updates](http://www.zytrax.com/books/dns/ch7/xfer.html)
- [[DNS] named.conf設定檔:master與slave](http://www.ccy.twbbs.org/blog/?p=349)
- [設定 Master/Slave DNS Server](http://blog.roga.tw/2012/03/%E8%A8%AD%E5%AE%9A-masterslave-dns-server/)
