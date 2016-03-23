---
layout:  post
title:   "於 Ubuntu 12.04 上啟用 WordPress 的 PermaLink"
date:    2012-09-22
tags:    ["PermaLink | 永久鏈結", "web dev | 網頁開發", "SEO | 搜尋引擎最佳化", "Ubuntu", "Linux", "WordPress"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

上次簡單介紹了 [PermaLink](http://blog.kuoe0.tw/posts/2012/09/12/about-permalink)，這次要來在 WordPress 中啟用**自訂 PermaLink** 的功能！為何說是自訂？因為 WordPress 中本身就提供了預設的 PermaLink，格式為 `yourdomainname/?p=post_id`，但這樣的 PermaLink 大概除了**短**之外沒有其他好處了！為了加強 SEO，我們需要讓我們的 PermaLink 更有結構化！

以下是本文章所使用之環境：

- Ubuntu 12.04 Server 64bit
- Apache 2.2.22
- WordPress v3.4.2

按照接下來的步驟即可啟用 WordPress 中的自訂 PermaLink 功能：

### Step 1. 啟用 rewrite module

```bash
$ sudo a2enmod rewrite
$ sudo /etc/init.d/apache2 restart
```
	
我們需要將原 WordPress 提供的 `yourdomainname/?p=post_id` 的動態鏈結，替換為有結構化的靜態鏈結，而 rewrite 模組允許 Apache 根據規則重寫鏈結位置。

### Step 2. 允許 FollowSymLinks

首先開啟 `/etc/apache2/sites-enabled/000-default`（或自己放置設定的檔案）。會有類似以下的內容出現，根據 WordPress 的位置，新增或修改該資料夾的設定。

```
<Directory /var/www/WordPress >
	Options Indexes MultiViews
	AllowOverride None
	Order allow,deny
	Allow from all
</Directory>
```

在 Options 那行新增 `FollowSymLinks` 字串！若本來就有啟用 FollowSymLinks 則不需修改。

### Step 3. 允許 FileInfo 導向

同樣開啟 `/etc/apache2/sites-enabled/000-default`（或自己放置設定的檔案）。會看到有一行 `AllowOverride None`，將這行改成 `AllowOverride FileInfo`，或是改為 `AllowOverride All` 也可。

經過以上兩個步驟後，`/etc/apache2/sites-enabled/000-default`（或自己放置設定的檔案）。將會變為以下內容：

```
<Directory /var/www/WordPress >
	Options Indexes MultiViews FollowSymLinks
	AllowOverride FileInfo
	Order allow,deny
	Allow from all
</Directory>
```
	
設定 FileInfo 後，Apache 才允許 .htaccess 檔案中的規則可覆蓋原規則。

### Step 4. 檢查 .htaccess 檔案

在完成以上的步驟後，即可開啟 WordPress 設定 PermaLink 的頁面中設定鏈結結構。在設定完結構後，選擇確定後，WordPress 將會在 .htaccess 檔案中寫入以下資訊：

```
<IfModule mod_rewrite.c>
	RewriteEngine On
	RewriteBase /
	RewriteRule ^index\.php$ - [L]
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule . /index.php [L]
</IfModule>
```

若是 .htaccess 檔案權限設定為不可寫入，請自行將以上內容複製至 .htaccess 檔案中！若不存在該檔案，請自行創建。

### Step 5. WordPress 的 PermaLink 結構設定

在 WordPress 的 PermaLink 設定頁面中，可以看到幾項寫好的結構供選擇。但在上一篇「[關於 PermaLink][1]」中有提到，PermaLink 的結構需要按照網頁的特性而改變，預先提供的結構可能不敷使用。於是可以使用最下面的自訂來進行設定！

WordPress 提供的參數有：

- %year%

	以四位數顯示年份，例如：2012。

- %monthnum%

	以兩位數顯示月份，例如：06。

- %day%

	以兩位數顯示日期，例如：12。
	
- %hour%

	以兩位數顯示小時（24 小時制），例如：19。
- %minute%

	以兩位數顯示分鐘，例如：56。
	
- %second%

	以兩位數顯示秒數，例如：01。

- %post_id%

	每篇文章的 ID，例如：本篇文章 URL 中的 **1800** 部分。
	
- %postname%

	每篇文章的標題，或自行替文章設定的名稱，例如：本篇文章 URL 中的 **enable-WordPress-permalink-in-ubuntu-12-04** 部分。

- %category%

	文章分類名稱。	

