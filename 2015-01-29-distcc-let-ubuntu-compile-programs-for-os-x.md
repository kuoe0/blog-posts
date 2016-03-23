---
layout:  post
title:   "distcc - 讓 Ubuntu 幫 OS X 編譯程式"
date:    2015-01-29
tags:    ["distcc", "OS X", "Ubuntu, C/C++"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

前幾天聽到同事說到 distcc 這工具，詢問了一下才知道是這麼好用的工具。其官方網站：[https://code.google.com/p/distcc/](https://code.google.com/p/distcc/)。如同其標題所寫「distcc: a fast, free distributed C/C++ compiler」，就是一個可以進行分散式編譯的概念！C/C++ 程式在建置的過程中，最花時間的就是編譯了。這個工具的概念就是將編譯平行化，讓其他電腦也可以幫忙編譯程式碼，如此一來就可以節省下不少時間！

我本身使用的是 Macbook Air，本身搭載的就是低功耗處理器，對於編譯這種耗 CPU 資源的工作本來就非常不在行。然後看著公司配的桌機幾乎不用好像有點浪費，得知 distcc 後就心想一定要想辦法把這工具裝起來！

而且現在在研究 Firefox，不時就要重新建置 Firefox，一次就要 30 分鐘。使用 distcc 後，建置整個 Firefox 時間縮短為 15 分鐘！可以明顯發現 distcc 真的有效！這些時間都包含前處理、編譯、連結這三個步驟。前處理與連結都只能在本機執行，只有編譯的步驟可以被平行化。也就是前處理跟連結的時間是省不掉的，由此可知 distcc 的效果是如此顯著！

然而，現在要做的是利用 Linux 來編譯 OS X 可以使用的二進位檔，也就是進行 cross-compile。當然，要相反過來也是可行的。所以會在 Linux 上安裝 clang 這個編譯器，之後都會指定 distcc 使用 clang 來進行編譯。我有試過使用 gcc，但 Linux 那端會一直編譯失敗，使得 distcc 變得毫無效果。確切原因我還不太清楚，就先直接使用 clang 吧。


接下來，就來介紹怎麼安裝與使用！

## Enviroment

**Ubuntu server**

- CPU: Intel Core i7-4790 @ 3.60GHz × 8
- Mem: 16GB
- OS: Ubuntu 14.04

**Macbook Air**

- CPU: Intel(R) Core(TM) i7-3667U CPU @ 2.00GHz × 2
- Mem: 8GB
- OS: OS X 10.9

單從規格就可以看出 Macbook Air 與 Ubuntu server 比起來根本就是懶趴比雞腿啊...。更加讓我希望可以用 Ubuntu server 來幫忙編譯了！

## Installation

### Ubuntu Server

安裝 distcc 與 clang：

	$ sudo apt-get install distcc clang


### Macbook Air

安裝 distcc：

	$ brew install distcc


## Setup

### Ubuntu server

在 Ubuntu 這端，首先要編輯 /etc/default/distcc 這個檔案。修改以下三個設定：

- STARTDISTCC
	- 開機使否啟動
- ALLOWEDNET
	- 允許使用本機的 distcc 的網路或是機器
- LISTENER
	- distcc 會聽來自哪些 IP 的連線

設定後的範例結果如下：

	STARTDISTCC="true"
	ALLOWEDNETS="127.0.0.1 10.247.27.0/24"
	LISTENER="0.0.0.0"

這樣表示，開機會自動啟動。並且監聽所有連線，但僅允許本機 (localhost) 與 10.247.27.0/24 這個網域下的機器可以使用本機的 distcc 服務。

設定好後，啟動或是重新啟動 distcc：

	$ sudo service distcc start # or use 'restart' to restart distcc
	 * Restarting Distributed Compiler Daemon: distccd          [OK]

這樣一來，Ubuntu server 上的 distcc 服務就啟動了。可以利用以下指令來查看 distccd 使否真的執行成功：

	$ ps aux | grep distccd
	distccd  16927  0.0  0.0  19784   180 ?        SN   01:36   0:00 /usr/bin/distccd --pid-file=/var/run/distccd.pid --log-file=/var/log/distccd.log --daemon --allow 127.0.0.1 --allow 10.247.27.0/24 --allow 10.247.24.0/24 --listen 0.0.0.0 --jobs 200
	distccd  16928  0.0  0.0  19784   180 ?        SN   01:36   0:00 /usr/bin/distccd --pid-file=/var/run/distccd.pid --log-file=/var/log/distccd.log --daemon --allow 127.0.0.1 --allow 10.247.27.0/24 --allow 10.247.24.0/24 --listen 0.0.0.0 --jobs 200
	distccd  16929  0.0  0.0  19784   180 ?        SN   01:36   0:00 /usr/bin/distccd --pid-file=/var/run/distccd.pid --log-file=/var/log/distccd.log --daemon --allow 127.0.0.1 --allow 10.247.27.0/24 --allow 10.247.24.0/24 --listen 0.0.0.0 --jobs 200
	distccd  16930  0.0  0.0  19784   180 ?        SN   01:36   0:00 /usr/bin/distccd --pid-file=/var/run/distccd.pid --log-file=/var/log/distccd.log --daemon --allow 127.0.0.1 --allow 10.247.27.0/24 --allow 10.247.24.0/24 --listen 0.0.0.0 --jobs 200
	distccd  16931  0.0  0.0  19784   180 ?        SN   01:36   0:00 /usr/bin/distccd --pid-file=/var/run/distccd.pid --log-file=/var/log/distccd.log --daemon --allow 127.0.0.1 --allow 10.247.27.0/24 --allow 10.247.24.0/24 --listen 0.0.0.0 --jobs 200
	distccd  16932  0.0  0.0  19784   180 ?        SN   01:36   0:00 /usr/bin/distccd --pid-file=/var/run/distccd.pid --log-file=/var/log/distccd.log --daemon --allow 127.0.0.1 --allow 10.247.27.0/24 --allow 10.247.24.0/24 --listen 0.0.0.0 --jobs 200
	distccd  16933  0.0  0.0  19784   180 ?        SN   01:36   0:00 /usr/bin/distccd --pid-file=/var/run/distccd.pid --log-file=/var/log/distccd.log --daemon --allow 127.0.0.1 --allow 10.247.27.0/24 --allow 10.247.24.0/24 --listen 0.0.0.0 --jobs 200

如果 grep 出來的結果不是當前指令的話，而是出現很多類似上面的訊息，就表示 distccd 成功啟動了！

### Macbook Air

使用 homebrew 安裝的 distcc 設定檔會在 /usr/local/etc/distcc 目錄底下。這邊需要編輯的檔案只有 /usr/local/etc/distcc/hosts 這個檔案。這個檔案內會寫上有哪些網路上的機器提供 distcc 的服務，以及其相關設定。設定後的檔案內容如下：

	10.247.27.101/200
	localhost/2

以上的內容表示網路上有兩檯機器提供 distcc 的服務，分別是 IP 為 10.247.27.101 與本機 (localhost)。此外，可以發現兩個 IP/hostname 後面都加上了一個斜線及一個數字。這表示該機器最大可以指定的工作數量。如果沒有設定的話，除了本機之外的都是預設為 4，而本機則是預設為 2。這部分還有很多額外的設定可以操作，詳細請見 distcc 的[文件](http://www.manpagez.com/man/1/distcc/)中的 **HOST SPECIFICATIONS** 章節。

而如果在這檔案中沒寫上 localhost 的話，就目前目測觀察來看，似乎就不會把任何編譯工作指派到本機上。所以本機就只要進行前處理以及連結就好，編譯時期就不會 CPU 一直滿載導致電腦使用起來相當不順了。

在 server (Ubuntu server) 跟 client (Macbook Air) 都設定好 distcc 之後，可以在 client 上執行以下指令來查看 distcc 最大可以執行的工作數量：

	$ distcc -j
	202

以我前面設定的為例，可以看到顯示的結果為 202。表示之後在執行 `make` 指令時，最大可以使用 `-j202` 的參數來平行化。

## Usage

在大部分情況下，只需要把環境變數的 CC 與 CXX 修改一下即可。

	$ export CC="distcc clang -target x86_64-apple-darwin"
	$ export CXX="distcc clang++ -target x86_64-apple-darwin"

或是直接去 makefile 中，把原本設定的編譯器加上 `distcc ` 的前綴。這樣一來，執行 `make` 時，就會開始用 distcc 進行編譯了！

然而，如果使用的是 automake 這樣的方式來產生 makefile，則要在 `./configure` 的時候加上 `--cc` 跟 `--cxx` 的參數，如下：

	$ ./configure --cc="distcc clang -target x86_64-apple-darwin" --cxx="distcc clang++ -target x86_64-apple-darwin"

這樣產生的 makefile 就會設定好要使用 distcc 進行編譯了。

如果設定成功的話，執行編譯時，可以查看 Ubuntu server 下的 /var/log/distccd.log 該檔案。即可看到類似以下的輸出，表示 distcc 成功地在幫 OS X 進行編譯。

	distccd[3738] (dcc_job_summary) client: 10.247.24.6:51236 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:271ms clang libavfilter/vsrc_mandelbrot.c
	distccd[3738] (dcc_job_summary) client: 10.247.24.6:51238 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:21ms clang libavfilter/x86/af_volume_init.c
	distccd[3738] (dcc_job_summary) client: 10.247.24.6:51239 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:45ms clang libavfilter/af_astats.c
	distccd[3841] (dcc_job_summary) client: 10.247.24.6:51225 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:1194ms clang libavfilter/af_biquads.c
	distccd[3841] (dcc_job_summary) client: 10.247.24.6:51212 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:123ms clang libavfilter/af_volume.c
	distccd[4052] (dcc_job_summary) client: 10.247.24.6:51237 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:465ms clang libavfilter/vsrc_testsrc.c
	distccd[3952] (dcc_job_summary) client: 10.247.24.6:51230 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:697ms clang libavfilter/avf_showcqt.c
	distccd[3738] (dcc_job_summary) client: 10.247.24.6:51240 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:765ms clang libavfilter/x86/vf_idet_init.c
	distccd[3816] (dcc_job_summary) client: 10.247.24.6:51220 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:1957ms clang libavfilter/vf_shuffleplanes.c
	distccd[4792] (dcc_job_summary) client: 10.247.24.6:51193 COMPILE_OK exit:0 sig:0 core:0 ret:0 time:4771ms clang libavfilter/af_earwax.c


### For Firefox

前面提到過，我是為了加速編譯 Firefox 才安裝 distcc 的。因為 Firefox 本身有自己的 Mozilla build system，所以在設定上會有些不同。不過其實很簡單，除了前面有提到的設定 CC 跟 CXX 變數外，就是把 `-j` 寫進 mozconfig 檔案裡。假如想要加上 `-j100` 的選項，就寫上下面這行在 mozconfig 裡即可。

	mk_add_options MOZ_MAKE_FLAGS="-j100"


下圖有真相，可以看到當 Macbook Air 這端開始進行編譯時，Ubuntu 那端的 CPU 使用率也大幅出現變動。

![Ubuntu Monitor](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2015-01-29-distcc-let-ubuntu-compile-programs-for-os-x-1.png)

另外，附上兩張截圖顯示部屬 distcc 前後的編譯時間差異：

![部屬 distcc 前](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2015-01-29-distcc-let-ubuntu-compile-programs-for-os-x-2.png)
(部屬 distcc 前)

![部屬 distcc 後](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2015-01-29-distcc-let-ubuntu-compile-programs-for-os-x-3.png)
(部屬 distcc 後)

最後，引用自同事的話：

> 用 Air 來 build code 簡直是太閒...
