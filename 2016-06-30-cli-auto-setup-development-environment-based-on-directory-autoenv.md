---
layout: post
title:  "[CLI] 基於資料夾來自動設定開發環境 - autoenv"
date:   2016-06-30
tags:   ["CLI", "ZSH", "Firefox"]
feature:
    photo: false
---

相信許多人在開發不同專案時都會需要不同的開發環境，也因為這個原因才會有 Python 的 [virtualenv](https://virtualenv.pypa.io/en/stable/) 與 Ruby 的 [RVM](http://rvm.io/) 或是 [rbenv](http://rbenv.org/) 出現。

筆者在進行 Firefox 開發時，會需要用到一些只有開發 Firefox 才會用到的工具或是指令。然而筆者不希望把這些工具的路徑或是指令加入 profile（像是 `~/.zshrc`、`~/.bashrc` 等等）中而污染了整個系統的設定，這時候就需要 autoenv 啦！

autoenv 是基於資料夾來自動設定環境。其概念其實就是當進入某個資料夾後，查看該資料夾（或其上層上上層...）是否存在某個設定檔，並根據該設定檔來設定環境。目前 autoenv 的實作滿多種的：

- [Tarrasch/zsh-autoenv](https://github.com/Tarrasch/zsh-autoenv)
- [horosgrisa/autoenv](https://github.com/horosgrisa/autoenv)
- [kennethreitz/autoenv](https://github.com/kennethreitz/autoenv)

筆者的選擇是 [Tarrasch/zsh-autoenv](https://github.com/Tarrasch/zsh-autoenv)，主因是可以直接使用 ZSH 的 plugin manager 來安裝以及 varstash 的功能。雖然 [horosgrisa/autoenv](https://github.com/horosgrisa/autoenv) 也可以使用 plugin manager 進行安裝，但至少從文件上的介紹是沒看到 varstash 的功能。所以以下的介紹都是基於 [Tarrasch/zsh-autoenv](https://github.com/Tarrasch/zsh-autoenv)。

一旦安裝了 autoenv，每次在終端機中進行資料夾切換時，都會依序發生以下兩件事情：

1. 載入正要離開的資料夾中的 **.autoenv_leave.zsh**
2. 載入正要進入的資料夾中的 **.autoenv.zsh**

而這兩個檔案其實就只是一般的 shell script，autoenv 做的就只是幫你自動執行這兩個 shell script 而已。

## Varstash

Varstash 的功能是我選擇 [Tarrasch/zsh-autoenv](https://github.com/Tarrasch/zsh-autoenv) 的最主要原因。在不同的專案中，最常發生的狀況之一就是需要將某些工具的路徑放到 `PATH` 變數中。然而，如果我們在 **.autoenv.zsh** 中修改了 `PATH` 變數，那麼也需要在 **.autoenv_leave.zsh** 中進行復原才不會污染開發環境。

Varstash 提供以下三個指令：

- `stash`：將該變數原本的值存起來，並設定為新給定的值。
- `unstash`：將該變數存起來的值取出，並設定回該變數。
- `autostash`：執行 `stash` 來設定該變數，該變數會在離開該資料夾時自動觸發 `unstash`。

就如同前面所說的，在某些專案中，可能會需要將該專用才用得到的工具加入 `PATH` 變數中。這時候 autoenv 的設定就可以寫的像下面的範例一般：

**.autoenv.zsh**

```zsh
stash PATH="path/to/tools:$PATH"
```
**.autoenv_leave.zsh**

```zsh
unstash PATH="path/to/tools:$PATH"
```

這樣當離開該資料夾時，`PATH` 變數就會恢復回原本的值了。然而，如果不想要每次都在 `.autoenv_leave.zsh` 中自行 `unstash` 的話，可以選擇使用 `autostash`。範例如下：

**.autoenv.zsh**

```zsh
autostash PATH="path/to/tools:$PATH"
```

如此一來，儘管沒有 `.autoenv_leave.zsh` 這個設定檔，也能夠讓環境變數回復成原本的值。

## Firefox 的 autoenv 設定

**.autoenv.zsh**

```sh
OS=$(uname)

autostash GECKO=$(git rev-parse --show-toplevel)
alias mach="$GECKO/mach"

# Load completion for mach
autoload bashcompinit
bashcompinit
source "$GECKO/python/mach/bash-completion.sh"

# Load git-cinnabar
if echo "$PATH" | grep -q "cinnabar"; then
	echo "git-cinnabar is already ready!"
else
	LOCATE="locate"
	[[ "$OS" = "Darwin" ]] && LOCATE="mdfind" # use `mdfind` in OS X as locate

	if [[ -z ${CINNABAR_PATH+x} ]]; then
		# cache CINNABAR_PATH
		export CINNABAR_PATH=$($LOCATE git-cinnabar | grep cinnabar | sort | head -n 1)
	fi

	autostash PATH="$CINNABAR_PATH:$PATH"
	echo "git-cinnabar is ready!"
fi
```

- 設定 mach command 的 alias [註 1]
- 設定 mach command 的 complete function
- 將 git-cinnabar 加入 `PATH` 環境變數中 [註 2]

> 註 1：mach command 是 Gecko 專案專用的開發工具，提供非常多項開發所需要的任務。
>
> 註 2：git-cinnabar 是一個 git plugin，讓開發者可以使用 git 來開發使用 mercurial 做為 VCS 的專案，像是 Gecko。未來有機會會在另闢專文介紹 git-cinnabar 。

**.autoenv_leave.zsh**

```sh
unalias mach
complete -r mach
```

- 撤銷 mach command 的 alias
- 撤銷 mach command 的 complete function

由於在設定 `PATH` 變數時，有加上 `autostash`，因此不需要自行回復 `PATH` 變數的值。
