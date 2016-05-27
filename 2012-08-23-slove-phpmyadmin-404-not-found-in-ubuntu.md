---
layout: post
title:  "解決 Ubuntu 下 phpMyAdmin 的 404 Not Found 頁面"
date:   2012-08-23
tags:   ["Ubuntu", "Linux", "database | 資料庫", "phpMyAdmin"]
image:  "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2012-08-23-slove-phpmyadmin-404-not-found-in-ubuntu.jpg"
---

AWS 的[一年免費試用](http://aws.amazon.com/free/)到期了，於是決定把主機從 Virginia 遷到 Tokyo，據說會變快？！加上以前都亂裝些有的沒的 plugin，導致資料庫一堆雜七雜八的欄位，就順便重新安裝 wordpress 了。

相信許多人在使用 MySQL 時都會順便安裝 [phpMyAdmin](http://www.phpmyadmin.net/home_page/index.php) 以方便操作資料庫。然而在 ubuntu 下常常安裝後連上 http://yourdomain/phpmyadmin/ (or [http://localhost/phpmyadmin/](http://localhost/phpmyadmin)) 時都會收到 404 Not Found 的錯誤，是由於 apache 那邊並沒有 phpMyAdmin 的設定，因此我們需要將 phpMyAdmin 的設定給 Apache，透過接下來的指令即可完成。並將 apache 重新啟動。

```bash
# link configure file to apache
$ sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf.d/phpmyadmin.conf
# restart apache
$ sudo /etc/init.d/apache2 restart
```
