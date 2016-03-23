---
layout:  post
title:   "在 Ubuntu 上安裝 DNS server"
date:    2013-08-11
tags:	   ["Ubuntu", "Linux", "DNS", "network management | 網路管理"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

目前自認實驗室網管，想幫實驗室假設 DNS server，這樣才不用增加個 subdomain 就要去麻煩網路助教！為了避免他覺得我太煩，我還是自己架比較實在，其實也是自己想要多些網路管理的經驗就是…。這邊我採用最常見的 BIND 這套 DNS server，BIND 全名為 Berkeley Internet Name Daemon，目前版本為 9.8.1。

以下為我當前的環境：

- Ubuntu Server 12.04 LTS
- BIND 9.8.1

以下為假設：

- domain name: 5566.csie.ncku.edu.tw
- IP address: 140.116.55.66
- subdomain: kuoe0.5566.csie.ncku.edu.tw
- subdomain IP address: 140.116.5.6

## 架設 DNS

首先就是先安裝 BIND，直接利用 APT 即可：

```bash
$ apt-get install bind9
```

建議也可以把測試工具與文件也一併安裝！！

**測試工具：**

```bash
$ apt-get install dnsutils
```

**文件：**

```bash
$ apt-get install bind9-doc
```

主要設定檔如下：

- **/etc/bind/named.conf.option**
- **/etc/bind/named.conf**
- **/etc/bind/named.conf.local**

### `/etc/bind/named.conf`

BIND 的主要設定檔為 **/etc/bind/named.conf**，每當 BIND 啓動時，就會去讀取這個檔案！不過其實不用管它，現在 BIND 的設定檔已經結構化了，這個檔案只是負責去引入其他檔案！

### `/etc/bind/named.conf.local`

我們先來設定我們所需要的 zone，首先修改 **/etc/bind/named.conf.local** 這個檔案！

在裡面加入 zone 的 DNS record 存放位置以及 DNS server 的形態。

**正解區**

```
zone "5566.csie.ncku.edu.tw" in {
    type master;
    file "/etc/bind/db.5566.csie.ncku.edu.tw";
};
```

這邊表示該 DNS server 為 master，且 DNS record 存放於 **/etc/bind/db.5566.csie.ncku.edu.tw** 該檔案中！

**反解區：**

```
zone "55.116.140.in-addr.arpa" in {
    type master;
    file "/etc/bind/db.55.116.140";
};
```

這邊表示該 DNS server 為 master，且 DNS record 存放於 **/etc/bind/db.55.116.140** 該檔案中！不過，其實我們沒有反解的權限，這邊純粹設定爽的…。

由於我們跟 BIND 告知了正解區與反解區的設定檔位置，當然我們也要有相對應的檔案！因此需要創建 **/etc/bind/db.5566.csie.ncku.edu.tw** 與 **/etc/bind/db.55.116.140** 這兩個檔案，並寫好 DNS record。

**/etc/bind/db.5566.csie.ncku.edu.tw:**

```
$TTL    604800
@       IN SOA  ns.5566.csie.ncku.edu.tw. admin.5566.csie.ncku.edu.tw. (
                2013080901      ; Serial Number
                604800          ; Refresh
                86400           ; Retry
                2419200         ; Expire
                604800 )        ; Minimum

@       IN NS   ns.5566.csie.ncku.edu.tw.
@       IN A    140.116.55.66

ns      IN A    140.116.55.66
www     IN A    140.116.55.66
kuoe0   IN A    140.116.5.6
```

**/etc/bind/db.55.116.140:**

```
$TTL    604800
@       IN SOA  ns.5566.csie.ncku.edu.tw admin.5566.csie.ncku.edu.tw. (
                20070926        ; Serial Number
                604800          ; Refresh
                86400           ; Retry
                2419200         ; Expire
                86400 )         ; Minimum


@       IN NS   5566.csie.ncku.edu.tw.

66      IN PTR  5566.csie.ncku.edu.tw.
66      IN PTR  www.5566.csie.ncku.edu.tw.
66      IN PTR  dns.5566.csie.ncku.edu.tw.
```

**SOA 格式**

```
@    IN SOA    MNAME  RNAME (
        SERIAL
        REFRESH
        RETRY
        EXPIRE
        MINIMUM )
```

- `MNAME` 為 primary DNS server，輸入主要的 DNS 的 FQDN！
- `RNAME` 為系統聯絡人的 E-mail，記得要把 `@` 符號取代為 `.`，還有因為位址是 FQDN，所以也別忘了最後面的 `.`。
- `SERIAL` 為 DNS record 的流水號，每次更新後記得也要更新這個編號，而且要遞增！這樣一來，其他 DNS server 才知道 DSN record 有更新。
- `REFRESH` 為次要 DNS server 的更新時間，表示次要 DNS server 需要多久來跟主要 DNS server 同步資料。
- `RETRY` 為次要 DNS server 若是更新失敗要重試之前所需要等待的時間。
- `EXPIRE` 為次要 DNS server 的 DNS record 過期時間，也就是如果次要 DNS server 一直無法向主要 DNS server 進行資料更新的話，一旦過了這個時間後，次要 DNS server 將不會再進行解析 DSN record。
- `MINIMUM` 為 DNS record 的最小生存時間。當某個紀錄被查詢到後，DNS server 會將它給暫存起來，而在這個時間內的查詢都將直接利用該暫存的答案進行答覆。過了這個時間後，如果新的查詢才會進行解析。

### `/etc/bind/named.conf.option`

最後在修改 **/etc/bind/named.conf.option** 內的一些設定，我在 options 裡頭加上了：

```
dump-file       "/var/cache/bind/cache_dump.db";
statistics-file "/var/cache/bind/named.stats";
managed-keys-directory "/etc/bind";
```

`dump-file` 用來指定使用 `rndc dumpdb` 指令時輸出檔的位置，而 `statistics-file` 則指定使用 `rndc stats` 指令的輸出檔位置。

另外我在 `forwarders` 中加入了 Google Public DNS 的 IP，用來減輕 DNS 查詢的負擔，將不是給這台 DNS 解析的任務交由 [Google Public DNS](https://developers.google.com/speed/public-dns/?hl=zh-TW)。

以下為完整的 **named.conf.options** 內容：

```
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        dump-file       "/var/cache/bind/cache_dump.db";
        statistics-file "/var/cache/bind/named.stats";
        managed-keys-directory "/etc/bind";

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };

        allow-query { any; };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

設定完成後，只要重新啓動 BIND 即可：

```bash
$ service bind9 restart
```

最後再請你上層的 DNS server 中加入以下兩個 record：

```
5566       IN NS   ns.5566.csie.ncku.edu.tw.
ns.5566    IN A    140.116.55.66
```

有幾個小地方要注意：

- FQDN 最後需要有小數點
- 行末的分號

如果啓動 BIND 出現錯誤的話，可以查看 log 紀錄：

```bash
$ tail /var/log/syslog
     1  Aug  9 22:28:09 5566-Web named[10756]: starting BIND 9.8.1-P1 -u bind
     2  Aug  9 22:28:09 5566-Web named[10756]: built with '--prefix=/usr' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--sysco
nfdir=/etc/bind' '--localstatedir=/var' '--enable-threads' '--enable-largefile' '--with-libtool' '--enable-shared' '--enable-stati
c' '--with-openssl=/usr' '--with-gssapi=/usr' '--with-gnu-ld' '--with-geoip=/usr' '--enable-ipv6' 'CFLAGS=-fno-strict-aliasing -DD
IG_SIGCHASE -O2' 'LDFLAGS=-Wl,-Bsymbolic-functions -Wl,-z,relro' 'CPPFLAGS=-D_FORTIFY_SOURCE=2'
     3  Aug  9 22:28:09 5566-Web named[10756]: adjusted limit on open files from 4096 to 1048576
     4  Aug  9 22:28:09 5566-Web named[10756]: found 4 CPUs, using 4 worker threads
     5  Aug  9 22:28:09 5566-Web named[10756]: using up to 4096 sockets
     6  Aug  9 22:28:09 5566-Web named[10756]: loading configuration from '/etc/bind/named.conf'
     7  Aug  9 22:28:09 5566-Web named[10756]: /etc/bind/named.conf:11: missing ';' before 'include'
     8  Aug  9 22:28:09 5566-Web named[10756]: loading configuration: failure
     9  Aug  9 22:28:09 5566-Web named[10756]: exiting (due to fatal error)
```

## 測試 DNS

由於可能要等待上層 DNS server 更新 NS record 等，在等待的時間我們可以先測是透過本機的 DNS server 查找是否成功。在安裝時，有建議安裝測試工具 dnsutils，他會安裝三個測試用的工具，分別是：host、nslookup 以及 dig。

### host

`host` 的用法為：`host query_domain_name dns_server_ip`，最簡單的方式如下：

```bash
$ host kuoe0.5566.csie.ncku.edu.tw 140.116.55.66
```

就會產生以下的輸出：

```
Using domain server:
Name:    140.116.55.66
Address: 140.116.55.66#53
Aliases:

kuoe0.5566.csie.ncku.edu.tw has address 140.116.5.6
```

就可以看到本機的 DNS server 告訴我們 `kuoe0.5566.csie.ncku.edu.tw` 的 IP 位置為 `140.116.5.6`。如果需要更詳細的資訊可以加入 `-a` 的選項：

```bash
$ host -a kuoe0.5566.csie.ncku.edu.tw 140.116.55.66
Trying "kuoe0.5566.csie.ncku.edu.tw"
Using domain server:
Name: 140.116.55.66
Address: 140.116.55.66#53
Aliases:

;; ->>HEADER<--- opcode: QUERY, status: NOERROR, id: 30030
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; QUESTION SECTION:
;kuoe0.5566.csie.ncku.edu.tw. IN      ANY

;; ANSWER SECTION:
kuoe0.5566.csie.ncku.edu.tw. 604800 IN A      140.116.5.6

;; AUTHORITY SECTION:
5566.csie.ncku.edu.tw. 604800 IN      NS      ns.5566.csie.ncku.edu.tw.

;; ADDITIONAL SECTION:
ns.5566.csie.ncku.edu.tw. 604800 IN   A       140.116.55.66

Received 96 bytes from 140.116.55.66#53 in 0 ms
```

這樣就可以看到 `kuoe0.5566.csie.ncku.edu.tw` 是怎麼從哪裡查找出來的了！

### nslookup

`nslookup` 也很簡單，參數如同 `host`，給出來的資料結構我個人也比較喜歡：

```bash
$ nslookup kuoe0.5566.csie.ncku.edu.tw 140.116.55.66
Server:  140.116.55.66
Address: 140.116.55.66#53

Name:    kuoe0.5566.csie.ncku.edu.tw
Address: 140.116.5.6
```

### dig

`dig` 這個工具據說非常強大，不過我跟網路也不熟，目前對我來說需要的只是查找 domain name 能不能正常查找到 IP 位址而已，所以似乎對我來說也沒那麼強大說。`dig` 的參數跟 `host` 與 `nslookup` 不太相同，其用法為：`dig @dns_server_ip query_domain_name`，廢話不多說，直接看範例：

```bash
$ dig @140.116.55.66 kuoe0.5566.csie.ncku.edu.tw

; <<>> DiG 9.8.1-P1 <<>> @140.116.55.66 kuoe0.5566.csie.ncku.edu.tw
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<--- opcode: QUERY, status: NOERROR, id: 55982
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; QUESTION SECTION:
;kuoe0.5566.csie.ncku.edu.tw. IN      A

;; ANSWER SECTION:
kuoe0.5566.csie.ncku.edu.tw. 604800 IN A      140.116.5.6

;; AUTHORITY SECTION:
5566.csie.ncku.edu.tw. 604800 IN      NS      ns.5566.csie.ncku.edu.tw.

;; ADDITIONAL SECTION:
ns.5566.csie.ncku.edu.tw. 604800 IN   A       140.116.55.66

;; Query time: 0 msec
;; SERVER: 140.116.55.66#53(140.116.55.66)
;; WHEN: Sat Aug 10 02:37:47 2013
;; MSG SIZE  rcvd: 96
```

如果上層的 DNS server 生效後，要進行測試就不必再加上本機的 IP 了，直接查找即可。如果出現問題，那麼就是上層的 DNS server 上的 DSN record 設定有誤囉！如下：

```
$ nslookup kuoe0.5566.csie.ncku.edu.tw
Server:   8.8.8.8
Address:  8.8.8.8#53

Non-authoritative answer:
Name:	   kuoe0.5566.csie.ncku.edu.tw
Address: 140.116.5.6
```

就可以透過本身機器設定的 DNS server 來查詢到該 domain name 對應到的 IP 了！像我這邊就是設定為 Google Public DNS，所以可以透過 8.8.8.8 來解析到正確的 IP。

另外，可以藉由網路上提供的 DNS 檢測工具來做些安全性或性能上的檢測，例如：[DNS check](http://dnscheck.pingdom.com/)、[TWNIC DNS 檢測服務](http://rs.twnic.net.tw/cgi-bin/dns.cgi)、[DNS Sniffer](http://www.dnssniffer.com/)、[DNS Stuff](http://www.dnsstuff.com/)。

## Recursive Query

在利用 [DNS check](http://dnscheck.pingdom.com/) 進行檢測時，發現到一個中度漏洞是 recursive query。以下是該網站告知我的危險：

> The name server answers recursive queries for 3rd parties (such as DNSCheck). By making a recursive query to a name server that provides recursion, an attacker can cause a name server to look up and cache information contained in zones under their control. Thus the victim name server is made to query the attackers malicious name servers, resulting in the victim caching and serving bogus data.

簡單說就是一些 cracker 可以利用 recursive query 來使得使用者訪問到惡意網站。既然有漏洞就要修一下，詢問 Google 大神後發現只要在 **named.conf.option** 中加入 `recursion no` 再重啓 BIND 服務即可。接著在進行一次檢測就可以看到正常的結果了！

> Everything is fine.


## 參考資料

- [DNS Server on Debian 6](http://note.drx.tw/2008/08/serverdns-server-static-adsl.html)
- [Bind9 on Debian](http://neio.pixnet.net/blog/post/15546169-bind9-on-debian)
- [Study DNS-DNS 進階設定、安全、檢測](http://www.weithenn.org/cgi-bin/wiki.pl?Study_DNS-DNS_%E9%80%B2%E9%9A%8E%E8%A8%AD%E5%AE%9A%E3%80%81%E5%AE%89%E5%85%A8%E3%80%81%E6%AA%A2%E6%B8%AC)
- [鳥哥的Linux 私房菜-- DNS Server](http://linux.vbird.org/linux_server/0350dns.php)
- [DNS 線上教學研究計畫](http://dns-learning.twnic.net.tw/)
- [使用 Ubuntu 安裝 Bind9: Domain Name Service (DNS)](http://www.nowtaxes.com.tw/node/1114)
- [Oversimplified DNS](http://rscott.org/dns/soa.html)

**UPDATE**

- 2013/08/25: 加入 SOA 格式，與更正 SOA 中的 primary DNS 位址。
- 2013/08/25: 加入解決 recursive query warning 的方法。

