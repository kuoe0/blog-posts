---
layout:  post
title:   "在 CLI 下進行網頁截圖"
date:    2013-08-14
tags:    ["screenshot | 截圖", "CLI | 命令列介面", "OS X", "Linux"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

這學期接了一個跟眼動儀相關的工作，使用的軟體為一開源軟體 - [ITU Gaze Tracker](http://www.gazegroup.org/downloads/23-gazetracker)。其功能相當陽春，僅能追蹤眼球並回傳資料，另外比較酷的是可以啓動眼球滑鼠的功能。但國科會計劃怎麼可能這樣就夠用了，少說也要做個實驗。而這部分就跟市面上那幾家眼動儀差的可多了，市面上的軟體都有完善的實驗流程功能，所以我只能自己寫實驗的應用的。再者，因為當初說是要做網頁上的眼動測驗，我就決定直接用 php 來寫實驗應用了！

但是看網路上眼動儀的實驗結果通常都有呈現熱圖或是路徑圖，因此我應該需要實驗過程的截圖！但每一個受測者會有 62 個場景，而且都是隨機生成，手動截圖大概會崩潰。索性跪求 Google 大神，竟然讓我發現的 **webkit2png** 這東西！這玩意兒可以讓我們直接透過 command line 進行截圖，而且是用 Python 寫的。這個小工具有分 Mac 版跟 Linux 版，不過我沒試過能不能在 Windows 上跑。

## Mac Version

Mac 版的下載網址為：[http://www.paulhammond.org/webkit2png/](http://www.paulhammond.org/webkit2png/)，或是透過 homebrew 安裝：`brew install webkit2png`。

使用方式如下：

```bash
$ webkit2png [options] [url]
```

例如現在要截取 Google 首頁，只要輸入：

```bash
$ webkit2png http://google.com
```

接下來就會在當前的資料夾下發現三個檔案：

- googlecom-clipped.png
- googlecom-full.png
- googlecom-thumb.png

其中 googlecom-full.png 就是我們想要的圖片了！另外，這個工具還有許多功能可以作設定，例如想要指定瀏覽器大小，假設需要模擬 1024x768 的螢幕大小，只要加入 `-W` 與 `-H` 這兩個參數，如下：

```bash
$ webkit2png -W 1024 -H 768 http://google.com
```

其實實驗用的螢幕就是這個大小，但現在多數為寬螢幕，而且我想在自己的電腦上寫程式，能修改瀏覽器大小對我來說實在太重要了！想瞭解更多功能，輸入 `webkit2png -h` 即可查看完整的使用方式！

```bash
$ webkit2png -h
Usage: webkit2png [options] [http://example.net/ ...]

Examples:
webkit2png http://google.com/            # screengrab google
webkit2png -W 1000 -H 1000 http://google.com/ # bigger screengrab of google
webkit2png -T http://google.com/         # just the thumbnail screengrab
webkit2png -TF http://google.com/        # just thumbnail and fullsize grab
webkit2png -o foo http://google.com/     # save images as "foo-thumb.png" etc
webkit2png -                             # screengrab urls from stdin
webkit2png -h | less                     # full documentation

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit

  Browser Window Options:
    -W WIDTH, --width=WIDTH
                        initial (and minimum) width of browser (default: 800)
    -H HEIGHT, --height=HEIGHT
                        initial (and minimum) height of browser (default: 600)
    -z ZOOM, --zoom=ZOOM
                        zoom level of browser, equivalent to "Zoom In" and
                        "Zoom Out" in "View" menu (default: 1.0)

  Output size options:
    -F, --fullsize      only create fullsize screenshot
    -T, --thumb         only create thumbnail sreenshot
    -C, --clipped       only create clipped thumbnail screenshot
    --clipwidth=WIDTH   width of clipped thumbnail (default: 200)
    --clipheight=HEIGHT
                        height of clipped thumbnail (default: 150)
    -s SCALE, --scale=SCALE
                        scale factor for thumbnails (default: 0.25)

  Output filename options:
    -D DIR, --dir=DIR   directory to place images into
    -o NAME, --filename=NAME
                        save images as NAME-full.png,NAME-thumb.png etc
    -m, --md5           use md5 hash for filename (like del.icio.us)
    -d, --datestamp     include date in filename

  Web page functionality:
    --delay=DELAY       delay between page load finishing and screenshot
    --js=JS             JavaScript to execute when the window finishes loading
                        (example: --js='document.bgColor="red";')
    --no-images         don't load images
    --no-js             disable JavaScript support
    --transparent       render output on a transparent background (requires a
                        web page with a transparent background)
    --user-agent=USER_AGENT
                        set user agent header
```

## Linux Version

Linux 版本要到 [AdamN/python-webkit2png](https://github.com/AdamN/python-webkit2png/) 這裡下載，雖然網頁中提供的安裝方式有可以利用 pip 安裝，但我下了 `pip install webkit2png` 後，卻死活找不到這個檔案到底去哪裡了…，所以還是推薦直接從 github 下載下來執行就好！以下這幾個相依套件也需要安裝：

- python-qt4
- libqt4-webkit
- xvfb

下載後記得執行 `python setup.py install` 將 webkit2png 安裝到 python 的 library 中。接著賦予 **script/webkit2png** 可執行權限，就可以使用了！

基本的使用方式跟 Mac 版的一樣，使用方式如下：

```bash
$ ./webkit2png [options] [url]
```

例如現在要截取 Google 首頁，只要輸入：

```bash
$ script/webkit2png http://google.com
```

不過，當按下 `Enter` 後，是不會發現任何檔案的，只會得到螢幕出現一堆亂碼的結果…。因為這個版本它是將得到的圖片輸出到 `STDOUT`，所以需要再做一個 redirection 的動作，如下：

```bash
$ script/webkit2png http://google.com > googlecom.png
```

這麼一來，Google 首頁的截圖才會被存放到 googlecom.png 該檔案！另外，Linux 版的參數也跟 Mac 板不相容。像是要指定瀏覽器大小的話，是要下 `-g` 的參數，並直接加上兩個數字代表寬與高。假設我們一樣要 1024x768 大小的 Google 首頁，實際輸入如下：

```bash
$ script/webkit2png -g 1024 768 http://google.com > googlecom.png
```

要看更多說明只要輸入 `script/webkit2png -h` 即可！

```bash
$ script/webkit2png -h
Usage: webkit2png [options] <URL>

Creates a screenshot of a website using QtWebkit.This program comes with
ABSOLUTELY NO WARRANTY. This is free software, and you are welcome to
redistribute it under the terms of the GNU General Public License v2.

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -x WIDTH HEIGHT, --xvfb=WIDTH HEIGHT
                        Start an 'xvfb' instance with the given desktop size.
  -g WIDTH HEIGHT, --geometry=WIDTH HEIGHT
                        Geometry of the virtual browser window (0 means
                        'autodetect') [default: (0, 0)].
  -o FILE, --output=FILE
                        Write output to FILE instead of STDOUT.
  -f FORMAT, --format=FORMAT
                        Output image format [default: png]
  --scale=WIDTH HEIGHT  Scale the image to this size
  --aspect-ratio=RATIO  One of 'ignore', 'keep', 'crop' or 'expand' [default:
                        none]
  -F FEATURE, --feature=FEATURE
                        Enable additional Webkit features ('javascript',
                        'plugins')
  -w SECONDS, --wait=SECONDS
                        Time to wait after loading before the screenshot is
                        taken [default: 0]
  -t SECONDS, --timeout=SECONDS
                        Time before the request will be canceled [default: 0]
  -W, --window          Grab whole window instead of frame (may be required
                        for plugins)
  -T, --transparent     Render output on a transparent background (Be sure to
                        have a transparent background defined in the html)
  --style=STYLE         Change the Qt look and feel to STYLE (e.G. 'windows').
  --encoded-url         Treat URL as url-encoded
  -d DISPLAY, --display=DISPLAY
                        Connect to X server at DISPLAY.
  --debug               Show debugging information.
  --log=LOGFILE         Select the log output file
```

不過，Linux 版的有個缺點，就是執行時會跳出瀏覽器的視窗。向我需要跑大量的資料，要截取非常多張圖，在處理時電腦就幾乎不能使用了。基本上每秒都會跳出新視窗，然後該視窗就會位於頂層，所以做什麼都不行，打字也會中斷…。
