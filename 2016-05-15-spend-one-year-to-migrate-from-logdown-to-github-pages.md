---
layout: post
title:  "花了一年從 Logdown 到 GitHub Pages"
date:   2016-05-15
tags:   ["GitHub Pages", "Jekyll", "SSL", "HTTPS"]
feature:
    photo: false 
---

沒想到距離上一篇文章還是超過了一年...。胡適先生說過「**發表是最好的記憶**」，我一直很相信這句話，畢竟寫下來的東西都是經過消化才排泄出來的（好像怪怪的？！）。而且在寫作時，為了避免誤導讀者，這個過程中都會反覆地查詢資料確認自己所寫的內容是正確的，這個過程中通常可以讓我對我寫作的內容有更進一步的認識。不過也無可避免自己還是常常寫出錯誤的內容，或是忘記自己曾經寫過的東西。但總之，過去這一年多來一直沒有好好整理自己的筆記與學習的事物，真的覺得自己腦袋變鈍了，決定還是應該要盡快讓部落格恢復運作！

起初暫停寫作的原因是要跳槽到其他部落格平台，因為遲遲還沒搬遷完成就一直很懶得寫新文章。而跳槽的理由是 Logdwon 付費會員到期，沒有 custom domain 可以使用，所以決定跳槽到 GitHub Pages。為了 custom domain 這小功能要花新台幣 1500，我實在花不下去...orz。而且我自己是習慣用 off-line 編輯器來寫文章，所以 Logdown 那潮潮的 on-line 編輯器對我來說也沒什麼吸引力。加上，身為一個 geek，把部落格架在 GitHub 上才潮呀！

[GitHub Pages](https://pages.github.com/) 是 GitHub 提供的網站服務器，可以架設自己的網站在 Github 上，不過只能使用靜態網頁。許多放在 GitHub 上的專案，都可以到設定的頁面中起啟動一個自動產生的專案網站，或是自行創建一個 `gh-pages` 的 branch 並將自己的內容推到 Github 上即可。所以如果要用來作為部落格，就可以選擇使用後者，將部落格文章放在 `gh-pages` 的 branch 後即可瀏覽了！

不過，如果每一篇文章都要自己從頭到尾去寫 HTML 實在太累了，而且一旦發了新的文章，相信會有很多頁面需要跟著修改。為了避免這麻煩，GitHub Pages 也支援使用 [Jekyll](https://jekyllrb.com/)。Jekyll 是 Ruby 寫的靜態網頁產生器，可以利用你寫好的一些 template 與 Markdown 檔案來產生真正的 HTML 網頁。

此外，GitHub Pages 也支援 custom domain 的功能。只要在 repo 裡面增加一個檔案，其檔名叫做 `CNAME`，內容就放要給這個網站的 domain name。接著再到你的 DNS provider 去把該 domain name 的 CNAME record 指向 `<username>.github.io.` 即可生效。詳細可參考 GitHub 上的說明「[Using a custom domain with GitHub Pages](https://help.github.com/articles/using-a-custom-domain-with-github-pages/)」。

對我來說，一個部落格「至少」需要具備以下功能：

- Custom Domain Name
- Markdown Support
- Source Code Syntax Highlight

使用 GitHub Pages 通通可以滿足，而且還免費，真沒理由不跳槽啊！網路上可以找到非常多 Jekyll 的 template 來使用，以下列出幾個蒐集滿多 template 的網站：

- [Jekyll Theme](http://jekyllthemes.org/)
- [JekyllThemes.io](http://jekyllthemes.io/)
- [Jekyll Themes](http://themes.jekyllrc.org/)
- [Jekyll Themes and Templates - Jekyll Tips](http://jekyll.tips/templates/)
- [Dr. Jekyll's Themes](https://drjekyllthemes.github.io/)

上面的網站中可以找到很多不錯的 template，免費的付費的都有。不過我個人不知道是在挑惕什麼，覺得功能齊全的外觀不太喜歡，不然就是外觀喜歡的功能不齊。最後只好挑了一個外觀喜歡的，然後 fork 出來進行修改了！

挑了很久看上的是 [Harmony](https://github.com/gayanvirajith/harmony) 這個 template，不過有一些功能上的缺失：

- 程式碼區塊沒有行號
- 文章不能設定特色圖片
- 不支援 Disqus
- 不支援 MathJax
- 不支援 OpenGraph

所以只好自己緩慢地加入這些功能了...我修改後的版本也放在 GitHub 上，有興趣可以前往參考：[Harmono](https://github.com/KuoE0/harmono)。不過還有很多東西沒改好，程式碼也有點亂就是了。有很多 Harmony 留下的程式碼還沒清乾淨，網頁架構上應該還可以做些重構。不過，覺得不能再繼續拖下去了，就決定先讓部落格復活吧！

而在發布前，還有最後一哩路要走。Mozilla 一直以來致力於提升網路使用的安全，既然身為一個 Mozilla 的軟體工程師，要求自己的部落格走 HTTPS 也是應該的！如果使用的是 `<username>.github.io` 這樣的 domain name 的話，本身就是支援 HTTPS 的，這類的 domain name 本身就已經擁有 GitHub 的 SSL 憑證了，比較麻煩的是 custom domain name。

目前 GitHub 官方似乎沒有辦法支援 Let's Encrypt，本來以為無望了不過後來查到可以使用 CloudFlare 來簽發 SSL 憑證，詳細可以看這篇「[Set Up SSL on Github Pages With Custom Domains for Free](https://sheharyar.me/blog/free-ssl-for-github-pages-with-custom-domains/)」。CloudFlare 可以提供免費的 SSL 憑證給你的網站，只要你讓 CloudFlare 做為你的 DNS 託管服務器即可。也就是在你原本的 DNS provider 中的 NS record 設定為以下兩個位址：

- `chloe.ns.cloudflare.com`
- `mario.ns.cloudflare.com`

除了 CloudFlare 外，Kloudsec 也有提供相同的服務，有興趣嘗試可以到以下連結註冊使用：[Kloudsec for GitHub Pages](https://kloudsec.com/github-pages/new)。在找資料時，發現還滿多人對於 HTTPS for custom domain 有需求，這個討論串討論的還挺熱烈的：[Github Pages: Let's Encrypt!](https://gist.github.com/coolaj86/e07d42f5961c68fc1fc8)。

總之，部落格終於恢復運作了，希望接下來可以維持一個月至少一篇部落格的習慣！
