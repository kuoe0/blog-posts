---
layout:  post
title:   "ANSI/VT100 文字樣式控制碼"
date:    2013-10-30
tags:    ["CLI | 命令列介面", "shell script"]
feature:
    photo:       true
    creator:     
    url:         
    license:     
    license_url: 
---

常常寫些 shell script 時，為了要區別不同訊息，會選擇幫一些訊息加上色彩等樣式以便於快速區別！但每次都要上網重新找一次好像有點麻煩，不如直接記在這當筆記吧！

樣式控制主要分成三類：

- 一般屬性
- 文字色彩（又稱**前景色彩**）
- 背景色彩

控制碼格式如下：

`\x1b[一般屬性;文字色彩;背景色彩m`

其後方直接接上想要套用該樣式的文字即可生效！不過，要注意的是，一旦套用的話，所有接下來的輸出都會被套用該屬性。所以如果希望回復為預設樣式的話，需要使用「重置樣式」的控制碼。所以如果想對特定訊息套用樣式的話，格式為：

`\x1b[一般屬性;文字色彩;背景色彩m欲套用樣式之訊息\x1b[0m`

重格式中可以看到 `\x1b` 的字樣，這表示 1b 為十六進位之數值，也就是 27，在 ASCII code 中表示 ESC (escape)。所以其實也是可以直接輸入 ESC 字元，一般可以看到 `^[` 這樣的表示，在 vim 中可以靠輸入 <key>\<Ctrl\></key>+<key>V</key> 後再按下 <key>ESC</key> 鍵來輸出。如果直接用 `ESC` 來替代的話，格式可能會看起來像這樣：

`^[[一般屬性;文字色彩;背景色彩m`

## 一般屬性

code | effect
---- | ------
0    | 重置樣式
1    | 變亮
2    | 變暗
4    | 加底線
5    | 閃爍
7    | 文字及背景色彩對調
8    | 隱藏文字

不過，並非所有樣式套用後都會有效果，還是要看終端機本身是否有支援！

## 文字色彩

code | color
---- | ------
30   | 黑色 black
31   | 紅色 red
32   | 綠色 green
33   | 黃色 yellow
34   | 藍色 blue
35   | 洋紅 magenta
36   | 青綠 cyan
37   | 白色 white

## 文字色彩

code | color
---- | ------
40   | 黑色 black
41   | 紅色 red
42   | 綠色 green
43   | 黃色 yellow
44   | 藍色 blue
45   | 洋紅 magenta
46   | 青綠 cyan
47   | 白色 white

## 實際效果

以下的 shell script 可以顯示是各種控制碼組合後的結果：

<script src="https://gist.github.com/KuoE0/7228839.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/7228839).

透過以下指令執行後，即可看到結果：

```
$ bash color_code.sh
```

![result](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-10-30-ansi-vt100-text-style-control-code-1.png)

*因為我用的 color scheme 是 solarized-dark，所以可能會跟一般的顏色不太一樣！*
