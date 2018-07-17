---
layout: post
title:  "該 APT 有著超級牛力"
date:   2018-07-18
tags:   ["Linux"]
image:  "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2018-07-18-this-apt-has-super-cow-powers.jpg"
---

今天無聊執行了 `apt help`，結果如下：

```
apt 1.6.2 (amd64)
Usage: apt [options] command

apt is a commandline package manager and provides commands for
searching and managing as well as querying information about packages.
It provides the same functionality as the specialized APT tools,
like apt-get and apt-cache, but enables options more suitable for
interactive use by default.

Most used commands:
  list - list packages based on package names
  search - search in package descriptions
  show - show package details
  install - install packages
  remove - remove packages
  autoremove - 自動移除所有不再使用的套件
  update - update list of available packages
  upgrade - upgrade the system by installing/upgrading packages
  full-upgrade - upgrade the system by removing/installing/upgrading packages
  edit-sources - edit the source information file

See apt(8) for more information about the available commands.
Configuration options and syntax is detailed in apt.conf(5).
Information about how to configure sources can be found in sources.list(5).
Package and version choices can be expressed via apt_preferences(5).
Security details are available in apt-secure(8).
                                         該 APT 有著超級牛力。
```

疑？怎麼最後一句怪怪的－「**該 APT 有著超級牛力。**」

如果語系設定是英文的話，應該會看到 "This APT has Super Cow Powers."。由於我的 locale 設定是 `zh_TW.UTF-8`，所以會看到翻譯過後的內容。然後我想說的是...這 i18n 怎麼正經的內容不翻譯，偏偏只翻譯最後那句奇怪的東西ＸＤ。

查了一下，似乎還有更多彩蛋。`apt` 有個隱藏的 sub-command 叫做 `moo`。如果執行 `apt moo` 會得到一隻大眼牛：

```
                 (__)
                 (oo)
           /------\/
          / |    ||
         *  /\---/\
            ~~   ~~
..."Have you mooed today?"...
```

更進一步，可以再加一個 `moo` 變成 `apt moo moo` 可以得到另一隻大鼻孔牛：

```
                 (__)
         _______~(..)~
           ,----\(oo)
          /|____|,'
         * /"\ /\
           ~ ~ ~ ~
..."Have you mooed today?"...
```

最後還可以再加一個 `moo` 變成 `apt moo moo moo` 則能獲得一隻正在叫的大小眼牛：


```
                     \_/
   m00h  (__)       -(_)-
      \  ~Oo~___     / \
         (..)  |\
___________|_|_|_____________
..."Have you mooed today?"...
```

嗯...廢文一篇，而且 lag 快十年了才知道。