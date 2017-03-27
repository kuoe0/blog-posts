---
layout: post
title:  "我的 Gecko 開發環境"
date:   2017-03-23
tags:   ["Firefox"]
image:  "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2017-03-23-my-gecko-development-environment.jpg"
image_info:
    creator:     "wetwebwork"
    url:         "https://www.flickr.com/photos/wetwebwork/265231749/in/album-72157594313059704/"
    license:     "CC-BY-SA 2.0"
    license_url: "https://creativecommons.org/licenses/by-sa/2.0/"
---

掐指一算，竟然已經在 Mozilla 待超過兩年了。不過覺得自己跟 Gecko 其實還沒很熟，到底是筆者的問題還是 Gecko 的問題？！

這次決定分享一下開發 Gecko 時，所使用的一些小玩意兒。這篇文章會針對以下幾個項目來介紹：

- 使用 Git 來開發 Gecko
- 不同 config 的切換
- 在 prompt 顯示當前的使用的 mozconfig
- 上傳 patch 到 bugzilla 進行 code review
- 自動設定 Gecko-specific 的環境
- 初始化完整的 Gecko 開發環境


## 使用 Git 來開發 Gecko

Gecko 採用的版本控制系統是 Mercurial，但當今世上還是 Git 比較熱門吧。對於已經習慣 Git 的人來說，要再去熟練另一套版本控制系統是需要一些時間成本的。而且 Gecko 其實也有 Git 版本的 [mirror](https://github.com/mozilla/gecko-dev)，所以用 Git 也是可以進行開發。如此一來，特地學習 Mercurial 的效益也不大。

但使用 Git 開發，還是存在一些缺點：

- 追蹤的不是最新的 Gecko codebase
- 上傳 patch 到 try server (test server) 不是很方便

於是，國外的同事就開發了一個 Git 套件叫做 [git-cinnabar](https://github.com/glandium/git-cinnabar)，讓我們可以使用 Git 的指令直接與 Mercurial repo 互動。這個套件可以讓我們使用以下的指令來追蹤 mozilla-central 這個 Mercurial repo，看起來就跟操作 Git repo 的感覺差不多。

```
$ git remote add central hg://hg.mozilla.org/mozilla-central
```

另外，要上傳 patch 到 try server 也很容易，也是使用以下指令就可以達成：

```
$ git push try
```

至於如何設置 git-cinnabar 來開發 Gecko，可以參考這位同事寫的這篇文章：[Mozilla: A git workflow for Gecko development](https://github.com/glandium/git-cinnabar/wiki/Mozilla:-A-git-workflow-for-Gecko-development)。

> 自從用了 cinnabar 後，我再也沒有打過 hg 指令了！

## 不同 config 的切換

Gecko 的 build config 都放在 mozconfig 這個檔案中，不同的項目會使用的 config 也不太一樣。除了 debug / optimized build 之外，Firefox Desktop / Firefox for Android 也都是透過 mozconfig 來設定。如果每次要用不同 config 時，都打開編輯器去修改 mozconfig 實在很沒效率。因此，有個同事寫了 [mozconfigwrapper](https://github.com/ahal/mozconfigwrapper) 這個工具，可以快速地切換 mozconfig。

只要把不同的 config 都放在同一個資料夾中，並且設定環境變數 `BUILDWITH_HOME` 為該資料夾即可。每次要切換 config 時，只要輸入以下指令即可。

```
$ buildwith config-xxx
```

由於很多 config 可能會共用一些設定。可以考慮建立一個 common 的 mozconfig，而其他的 mozconfig 再去 `source common`。這樣一來，一些共用的東西就不必每個 config 檔案都手動新增，只要新增到 common 這個檔案即可。可以參考筆者的 mozconfig 資料夾：[gecko-utils/mozconfigs](https://github.com/KuoE0/gecko-utils/tree/master/mozconfigs)。

> 自從用了 mozconfigwrapper，切換 mozconfig 的速度比翻臉還快！

## 在 prompt 顯示當前的使用的 mozconfig

使用 mozconfigwrapper 雖然方便，但是也可能造成不知道當前使用的 mozconfig 是哪一個的問題。一個最直接的方法是輸入以下指令來查詢：

```
$ echo $MOZCONFIG
```

但每次都要查詢實在太麻煩，現在很流行在 prompt 上加入各式各樣的資訊來提示開發者當前的環境。因此，筆者決定效仿一樣的方式把 mozconfig 的資訊加到 prompt 中。把下面這行加到你的 profile 中，即可在 prompt 中顯示當前使用的 mozconfig。

```
PS1+=$(basename $(mozconfig))
```

然而，在筆者的環境下，這個做法會失效。筆者使用的是 Z shell，套用的 prompt 主題是 [pure](https://github.com/sindresorhus/pure)。這個主題的 prompt 要設置在 `$PROMPT` 這個環境變數中，這個環境變數沒有被更新，實際顯示的 prompt 就不會更新。

因此必須要每次按下 enter 後，讓 `$PROMPT` 變數被更新。研究了一下 Z shell 的 manpage 發現有個指令 `add-zsh-hook` 可以在特定時機點觸發指定的函式。如果要在每次 prompt 出現之前，就要掛在 `precmd` 上。只要把以下內容寫在 profile 內，就可以動態地把當前使用的 mozconfig 更新到 prompt 上了。

```sh
update_prompt() {
    CONFIG=$(basename $MOZCONFIG)
    PROMPT="%F{242}${CONFIG} ${ORIGIN_PROMPT}"
}
add-zsh-hook precmd update_prompt
```

## 上傳 patch 到 bugzilla 進行 code review

過去在進行 code review 時，都要手動上傳 patch 到 bugzilla。而且如果使用 Git 開發的話，還要先透過 [`git-patch-to-hg-patch`](https://github.com/mozilla/moz-git-tools/blob/master/git-patch-to-hg-patch) 這個指令來轉換成 hg patch。

最近公司的某個部門就開發了 mozreview，讓開發者可以直接透過指令上傳 patch 到 bugzilla。下載 [mozilla/version-control-tools](https://github.com/mozilla/version-control-tools) 這個 repo，並且將 `version-control-tools/git/commands` 加入到環境變數 `PATH` 中即可使用。只要記得在 commit message 中放入 bug ID，就可以透過以下指令上傳 patch 到指定的 bug ID 的 bugzilla 頁面來進行 code review：

```
$ git mozreview push
```

這樣一來，就會自動把 mozilla-central 之後到 HEAD 的 commit 上傳到 bugzilla 上進行 review。不過要注意的是，要上傳的 patch 們都要是屬於同一個 bug ID 的 patch。如果針對某些 commit 來上傳 patch 的話，可以透過下面的方式。 

上傳單一 patch 的方式如下：

```
$ git mozreview push <commit>
```

上傳多個 patch 的方式如下，同樣的上傳多個 patch 時，這些 patch 都要是同一個 bug ID 的 patch：

```
$ git mozreview push <commit>..<commit>
```

此外，version-control-tools 也可以透過 Gecko 的 `mach bootstrap` 指令來進行安裝，在提示問你要不要設定 mozreview 時，記得選擇 yes。這麼一來，version-control-tools 會被安裝在 `$HOME/.mozbuild` 底下。

> 自從用了 mozreview 後，再也沒有手動上傳 patch 了！

## 自動設定 Gecko-specific 的環境

在看完前面幾個項目，會發現需要在 profile 中加入很多設定，而且都是 Gecko-specific。這時候，可以透過 autoenv 這類的工具來自動啟動或關閉這些設定。autoenv 的實作滿多的，在 GitHub 上可以找到很多不一樣的實作，選擇自己用起來順手的就好。

前面提過，筆者使用的 shell 為 Z shell，因此選擇了 [Tarrasch/zsh-autoenv](https://github.com/Tarrasch/zsh-autoenv) 這款。安裝這個 autoenv 的 plugin 後，只要在 Gecko 的資料夾下加入 `.autoevn.zsh` 與 `.autoenv_leave.zsh` 這兩個檔案即可。每當進入 Gecko 資料夾時，就會觸發 `.autoenv.zsh`。而離開 Gecko 資料夾實，會觸發 `.autoenv_leave.zsh`。所以，只要將前面針對 Gecko 的設定放到 `.autoenv.zsh` 中，而要恢復的設定可以寫在 `.autoenv_leave.zsh`。在寫這個 script 時，有幾個技巧可以使用，接下來會繼續介紹。

### autostash

`autostash` 這個指令，可以把原本環境變數的值記下，並且在離開該資料夾時自動恢復。

假設修改了 `PATH` 變數，如下：

```
autostash PATH="$TOOL_PATH:$PATH"
```

如此一來，離開 Gecko 資料夾時，`PATH` 就會被還原為原本的值了。


### auto completion for mach

`mach` 是 Mozilla 版本的 `make` 指令。這個指令可以做的事情非常多，像是 build Gecko、跑 test case、執行 firefox 或是 Google（別懷疑）。

大部分常操作 terminal 的人，想必都很喜歡 auto completion 的功能吧！畢竟人性就是惰性，可以按 <key>Tab</key> 就完成的指令，誰要一個字一個字慢慢輸入。

把以下的內容放到 `.autoenv.zsh` 中，就可以利用 <key>Tab</key> 來補全 `mach` 的 sub command 了。

```
autoload bashcompinit
bashcompinit
source "$GECKO/python/mach/bash-completion.sh"
```

### no more ./mach

筆者很討厭輸入 `./`，所以就加入了以下的 alias 來解決。

```
alias mach="$GECKO/mach"
```

因為 Gecko 資料夾底下還有其他執行檔，沒有需要執行其他執行檔的需求。所以才不考慮將整個 Gecko 資料夾放到 `PATH` 變數中。

> 自從用了 autoenv 後，覺得 zshrc 變乾淨了！

## 初始化完整的 Gecko 開發環境

人性就是惰性，因為筆者真的很懶惰，所以前面的設定真的很懶得每次都要重做一次。尤其是設定 git-cinnabar 那段實在有點累，於是寫了一個 [script](https://github.com/KuoE0/gecko-utils/blob/master/setup-gecko.sh) 把以下這些事情處理好：

- 下載並設定 git-cinnabar
- 下載並設定 mozconfigwrapper
- 更新 Gecko codebase
- 設定 autoenv

> 自從用了這個 script，設定好 Gecko 開發環境就是這麼簡單！
