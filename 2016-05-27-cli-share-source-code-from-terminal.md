---
layout: post
title:  "[CLI] 從 terminal 分享程式碼"
date:   2016-05-27
tags:   ["CLI", "Source Code | 程式碼", "Pastebin", "Gist", "Homebrew"]
image:  "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2016-05-27-cli-share-source-code-from-terminal.jpg"
image_info:
    creator:     "Toomore Chiang"
    url:         "https://www.flickr.com/photos/toomore/23066277453"
    license:     "CC 2.0"
    license_url: "https://creativecommons.org/licenses/by/2.0/"
---

常常在工作時都會有需要貼一些片段程式碼，或是貼一些 log 給同事的情況發生。有些同事會打開瀏覽器並把這些文字貼到 [Gist](https://gist.github.com/) 或是 [Pastebin](http://pastebin.com/) 等服務。但筆者認為這麼單純的一件事情還需要打開瀏覽器實在很沒效率，因此查詢了一下是否有能夠在 terminal 底下做到這些事情的指令。不查還好，一查不得了，也太多類似的東西了吧！一搜尋下去竟然這麼多類似的服務：

- [termbin](http://www.termbin.com/)
- [sprunge](http://sprunge.us/)
- [clbin](https://clbin.com/)
- [ix](http://ix.io/)

這幾個都是把要貼上的內容透過 `nc` 或是 `curl` 來把資料上傳到該網站上並進行貼上。以下是這幾個服務的比較，論功能性最強大的就是 ix 了！

功能                      | termbin | sprunge | clbin | ix
--------------------------|---------|---------|-------|---
純文字貼上                | ✓       | ✓       | ✓     | ✓
syntax highlight 網頁版   |         | ✓       | ✓     | ✓
syntax highlight 終端機版 |         |         |       | ✓
上傳圖片                  |	        |         | ✓     | ✓

不過以上的服務感覺就只能直接寫個 alias 到自己的 bashrc / zshrc，不是很喜歡這麼做。如果能夠直接透過 homebrew 之類的 package manager 來安裝還是比較喜歡些，於是又找到了一個叫做 `pastebinit` 的指令。這個指令可以直接透過 homebrew 安裝：

```
$ brew install pastebinit
```

`pastebinit` 的使用方式也很簡單：

```
$ <command> | pastebinit
```

而且它支援的網站非常多，請見以下清單或是輸入 `$ pastebinit -l` 來查看：

- cxg.de
- dpaste.com
- fpaste.org
- lpaste.net
- p.defau.lt
- paste.debian.net
- paste.drizzle.org
- paste.kde.org
- paste.openstack.org
- paste.pound-python.org
- paste.ubuntu.com
- paste.ubuntu.org.cn
- paste2.org
- pastebin.com
- pastebin.mate-desktop.org
- pb.daviey.com
- slexy.org
- sprunge.us

預設上會採用 Pastebin 作為上傳的網站，如果想要傳到其他網站的話可以透過 `-b` 選項來更改上傳的網站。當筆者決定就使用 `pastebinit` 時，安裝後發現竟然無法使用！正確來說，是不能用 Pastebin 作為上傳的網址，會顯示以下錯誤訊息：

```
Bad API request, invalid api_dev_key
```

查了一下原來是 `pastebinit` 在 v1.4.1 版的 API token 有問題，需要更新到 v1.5 版才有修正。於是就發了人生中第一個 Homebrew 的 PR，請見：[pastebinit 1.5 by KuoE0 · Pull Request #687 · Homebrew/homebrew-core](https://github.com/Homebrew/homebrew-core/pull/687)。不過整個 review 過程好煩，一下不能 replace config 檔案，一下不能跳 prompt 要使用者手動 replace config 檔案。最後 reviewer 的結論是，使用者會自己找到出路 LOL！

最後，如果習慣使用 Gist 的人也可以直接安裝 `gist` ([http://defunkt.io/gist/](http://defunkt.io/gist/)) 這個工具。基本的使用方式如下：

```
$ gist [file]
```

如果想看更詳盡的使用方式，請前往：[https://github.com/defunkt/gist#command](https://github.com/defunkt/gist#command)。
