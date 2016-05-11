---
layout: post
title:  "透過 homebrew 安裝特定版本套件"
date:   2013-04-19
tags:   ["OS X", "homebrew"]
feature:
    photo: false
---

昨天升級了 homebrew 中的不少套件，結果我的 vim 就毀滅了，一直出現 ImportError: cannot import name MAXREPEAT 的錯誤！找了好久才發現原來是 python 在月初發佈了 2.7.4 版本，因此我電腦中的版本也變成了 2.7.4，導致我 vim 中跟 python 相關的 plugin 都掛點了，於是決定來把 python 降到 2.7.3 版！以下是降版的步驟：

首先先來到 homebrew 安裝的資料夾，一般來說應該都在 /usr/local。

```
cd /usr/local
```

如果不知道自己的 homebrew 安裝在哪的話，可以輸入以下指令來進入該資料夾。

```
cd `brew --prefix`
```

接著利用以下指令可以找出該套件各個版本的對應的 homebrew 版本。

```
brew version <formular>
```

輸入該指令後，可以看到類似下方的列表（以 python 為例）：

```
brew versions python                 
2.7.4    git checkout fbf7be9 Library/Formula/python.rb
2.7.3    git checkout 51fb481 Library/Formula/python.rb
2.7.2    git checkout 97c6869 Library/Formula/python.rb
2.7.1    git checkout 83ed494 Library/Formula/python.rb
2.7      git checkout 1bf3552 Library/Formula/python.rb
2.6.5    git checkout acd49f7 Library/Formula/python.rb
2.6.4    git checkout 843bff9 Library/Formula/python.rb
2.6.3    git checkout 5c6cc64 Library/Formula/python.rb
```

像我要把 python 降到 2.7.3，就可以看到該套件在 homebrew 的 git 版本號是 51fb481，然後把 python 的 formula 切換到 2.7.3 版。

```
git checkout 51fb481 Library/Formula/python.rb
```

如果不是第一次安裝該套件的話，要先用以下指令把 homebrew 連結到的版本先取消連結。

```
brew unlink formula
```

然後按照原本安裝套件的方式即可安裝指定的版本了！

```
brew install formula
```

不過，這時候電腦中可能會存在多個版本，為了確定我們現在預設使用的版本是我們剛剛安裝的版本，可以使用以下指令來指定版本！

```
brew switch formula version
```

最後再把 homebrew 的 git repo 恢復到最新的版本，輸入最新的版本的版本編號即可！

```
git checkout fbf7be9 Library/Formula/python.rb
```

然後我無聊寫了個 script 來做這件事情，用法如下：

```
brew_version formula version [param1 param2 …]
```

- formula: 要安裝的套件名稱
- version: 指定的版本號
- param: 附加於 `brew install` 後的其它參數

例如我是要安裝 python，但在安裝時，我會多下一個 `--framework` 的參數。所以在使用這個 script 來安裝 2.7.3 版本的 python 時，就要下以後指令。

```
brew_version python 2.7.3 --framework
```

script 內容如下：

<script src="https://gist.github.com/KuoE0/5421086.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/5421086).
