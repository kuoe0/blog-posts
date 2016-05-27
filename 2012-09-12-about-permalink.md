---
layout: post
title:  "關於 PermaLink"
date:   2012-09-12
tags:   ["PermaLink | 永久鏈結", "web dev | 網頁開發", "SEO | 搜尋引擎最佳化"]
---

PermaLink 其實是 permanent link 的意思，也就是「永久鏈結」，也有人翻作「靜態鏈結」。用來使網站中的每一頁面都有屬於自己的網址！

## 為什麼要用 PermaLink

假若文章門有自己的 PermaLink，那麼網頁的訪客若想找尋某特定文章的話，僅能自首頁使用滾輪向下拉呀拉，或是一直點選下一頁。

不僅僅是對使用者來說不便，對於對於搜尋引擎也一樣，搜尋引擎將無法取得個篇文章的鏈結，所以搜尋引擎關於你的文章的結果都僅會顯示**首頁**。因為首頁是整個網站中唯一存在的真實鏈結！

在使用 wordpress 時，可以不必擔心這個問題！wordpress 本身就有提供每一篇文章的預設 PermaLink，形式為 `yourdomainname/?p=post_id`。

這是採用 http 中的 get 請求從首頁請求文章內容的作法，而這種形式的 PermaLink 除了很醜外，也沒有其他作用了！

因此接下來所要提的是 PermaLink 的結構！

## PermaLink 結構

一個好的 PermaLink 不僅漂亮，還可以增進 [SEO](http://en.wikipedia.org/wiki/Search_engine_optimization)。通常在設計 PermaLink 時，可以賦予該網址關於該文章的相關資訊，例如：日期、文章標題、分類等等。

在 PermaLink 上賦予相關資訊，對搜尋引擎來說也可以從網址中得到相關資訊，或許該網頁也因此得到較高的權重（當然決定權在搜尋引擎的演算法上）。而且使用具有意義的 PermaLink 不僅可以讓搜尋引擎從網址中得知資訊，人類也能直接從網址中獲得相關資訊！

PermaLink 的結構究竟怎麼設計比較好，網路上也有許多討論，目前似乎也沒有絕對好的結構。我覺得還是要針對文章與部落格的特性去做調整！

若是時效性較強的資訊，建議可以將日期放置於 PermaLink 的結構中！若是有分類導向的，即可將類別放置於 PermaLink 中！我個人也很推薦將標題也放入 PermaLink 中，如此一來，當他人看到網址也可以簡單知道該網頁的標題！

另外，雖然現在許多瀏覽器都支援中文的網址了，但在複製網址時，中文仍然會被轉換為 unicode，此時的網址就像是串亂碼，如此一來，本來用意是讓人類可讀的 PermaLink 也失去了該功能了！

**以 yahoo 的一篇新聞為例：**

我在瀏覽器網址列上看到的是 ↓

`http://tw.news.yahoo.com/嘉市城隍廟-將申請國定古蹟-105609981.html`

而我複製下網址後，貼上於記事本上則看到↓

`http://tw.news.yahoo.com/%E9%99%B8%E8%BB%8D%E6%96%B9%E6%94%BE%E8%A9%B1-%E8%AD%A6%E5%91%8A%E6%97%A5%E6%9C%AC%E5%8B%BF%E7%8E%A9%E7%81%AB-051606525.html`

當然搜尋引擎在檢索時，我想應該還是看到中文的部份，所以在 SEO 上應該影響不大！

另外關於使用中文於 PermaLink 中時，其順序也很重要，因為每個瀏覽器處理 url 編碼的方法不一，小心因此而使得訪客無法瀏覽文章只獲得 404 Error！關於該問題可以參考 xdite 前輩的「[Yahoo News 的 SEO 網址所帶來的問題](http://blog.xdite.net/posts/2011/10/25/yahoo-seo-url/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+xxddite+%28Blog.XDite.net%29)」，裡頭有更詳盡的解說。

我也因為 xdite 前輩的文章而決定將 PermaLink 的結構設定為 `domainname/post_id/post_title`，以避免我有需要使用中文作為網址時，我的文章還可以使用 `domainname/post_id` 連結！
