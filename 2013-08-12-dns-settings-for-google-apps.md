---
layout: post
title:  "DNS 設定 for Google Apps"
date:   2013-08-12
tags:   ["DNS", "network management | 網路管理", "Google Apps"]
feature:
    photo: false
---

由於懶得自行管理 mail server，想起之前實習的公司曾用過 [Google Apps](http://www.google.com/apps) 代管 mail server，而且其他實驗室也有使用 Google Apps  作為代管的，所以決定也幫實驗室申請一個！

不過，去年十二月 Google 宣布了不再提供免費的 Google Apps ([link](http://www.inside.com.tw/2012/12/07/no-free-google-apps-for-small-business-anymore))。不禁懷疑其他實驗室真的也都付費使用嗎？對於我們這新創的實驗室來說似乎太奢侈了，索性問了一下其他實驗室的學長，才知道原來有「[教育版](www.google.com/enterprise/apps/education/‎)」這東西，而且是**免費**的，但要提交申請就是了！

不過這篇重點不是怎麼申請 Google Apps，是怎麼設定 DNS！申請 Google Apps 的郵件代管後，是還無法使用的！首先要將自己所擁有的 domain name 的 MX record 指向 Google 提供的 mail server。設定如下：

```
@   3600    IN  MX  1   aspmx.l.google.com.
@   3600    IN  MX  5   alt1.aspmx.l.google.com.
@   3600    IN  MX  5   alt2.aspmx.l.google.com.
@   3600    IN  MX  10  aspmx2.googlemail.com.
@   3600    IN  MX  10  aspmx3.googlemail.com.
@   3600    IN  MX  10  aspmx4.googlemail.com.
@   3600    IN  MX  10  aspmx5.googlemail.com.
```

以上的設定中，可以發現第 4 欄有一些數字，這些數字表示 mail server 的優先層級，數字越低表示越優先。所以如果提供服務的 DNS server 只能設定一個 MX record 的話，就設定 **aspmx.l.google.com.** 這個就好。

要確認設定的話，可以到 [Google Apps Tools](https://toolbox.googleapps.com/apps/checkmx/) 進行檢查！另外設定好後要等待個 1~48 小時才會生效，根據 DNS 的設定而異！另外也有其他線上提供的檢測工具：[MXToolBox](http://mxtoolbox.com/)。

另外也可以客制 Google Apps 提供的其他服務的網址，進入管理面板 → 公司設定檔 → 管理自訂網址。以 mail 來說，預設的網址為 **mail.google.com/a/xxxlab.csie.ncku.edu.tw**，下面有個選項可以選擇使用自定的網址，可以改成 **mail.xxxlab.csie.ncku.edu.tw**。而 DNS 設定如下：

```
mail       3600 IN CNAME ghs.googlehosted.com.
calendar   3600 IN CNAME ghs.googlehosted.com.
drive      3600 IN CNAME ghs.googlehosted.com.
site       3600 IN CNAME ghs.googlehosted.com.
video      3600 IN CNAME ghs.googlehosted.com.
group      3600 IN CNAME ghs.googlehosted.com.
```

由於我也啟用了其它的服務，所以不只有 mail 要設定而已！

另外，可以新增 [SPF](http://www.openspf.org/) 設定，以防止同網域的郵件被送進垃圾郵箱中。只要增加一筆 TXT record 並指向 `v=spf1 include:_spf.google.com ~all` 這個內容即可！內容如下：

```
@   3600 IN TXT "v=spf1 include:_spf.google.com ~all"
```

如果是自行編寫 zone file 的話，引號可別忘了！如果是使用其他代管的 DNS，就不必增加引號了。

**參考資料：**

- [Configure MX records - Google Apps Help](https://support.google.com/a/bin/answer.py?hl=en&answer=140034)
- [MX record values - Google Apps Help](https://support.google.com/a/bin/answer.py?hl=en&answer=174125&topic=2683820&ctx=topic)
- [Create a custom web address - Google Apps Help](http://support.google.com/a/bin/answer.py?hl=en&answer=53340)
- [Create SPF records - Google Apps Help](http://support.google.com/a/bin/answer.py?hl=en&answer=178723&topic=2759192&ctx=topic)
