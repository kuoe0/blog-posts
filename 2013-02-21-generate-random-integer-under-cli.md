---
layout: post
title:  "在 CLI 下產生隨機整數"
date:   2013-02-21
tags:   ["CLI | 命令列介面", "Unix"]
feature:
    photo:       true
    photo_url:   "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2013-02-21-generate-random-integer-under-cli.jpg"
    creator:     "James Bowe"
    url:         "https://www.flickr.com/photos/jamesrbowe/4001776922"
    license:     "CC 2.0"
    license_url: "https://creativecommons.org/licenses/by/2.0/"
---

有時候在產生測試資料時，會有需要產生隨機整數的檔案，以前往往都自己寫一份像下面的程式碼來產生隨機數。

```c++
#include <iostream>
#include <cstdlib>
using namespace std;

int main() {
    int n;
    srand( time( 0 ) );
    scanf( "%d", &n );
    for ( int i = 0; i < n; ++i )
        printf( "%d\n", rand() );
    return 0;
}
```
    
這一年來也漸漸習慣 CLI (Command Line Interface) 的環境了，於是想說產生亂數好像也是滿常見的功能，在 CLI 下應該也有辦法用一兩行解決吧？！於是 Google 了一下，發現竟然有個環境變數叫做 `$RANDOM`！

發現在 bash 的 man page 裡可以找到他的說明：

> Each time this is referenced, a random integer between 0 and 32767 is generated. The sequence of random numbers may be initialized by assigning a value to RANDOM. If RANDOM is unset, it loses its special properties, even if it is subsequently reset.

不過不太確定他是 bash 專有的還是系統內的，我只知道我用 zsh 也是有的。既然是個環境變數，如此一來要在 CLI 下輸出一個隨機整數就簡單了，僅僅需要一行而且沒幾個字：

```
$ echo $RANDOM
```

如果要輸出多個亂數，僅需要利用 for 迴圈即可！以下是輸出 10 個隨機數的方法：

```bash
for i in {1..10}; do 
    echo $RANDOM; 
done
```
    
如果要在特定範圍間的隨機數，可以利用模運算與加法運算！以下是取 [100,200) 範圍內的隨機數：

```bash
echo $(( $RANDOM % 100 + 100 ))
```
    
另外發現系統中還存在兩個特殊的裝置，`/dev/random` 與 `/dev/urandom`，這兩個裝置都會一直輸出隨機的資料，如果有需要讀取隨機整數或資料時都可以使用。利用以下指令即可取得一個隨機整數：

```
$ od -An -N2 -i /dev/random
```

而 `/dev/random` 與 `/dev/urandom` 的差異在那個 **u**（廢話），這個 u 表示的是 **unlocked**。`/dev/random` 是取得 entropy pool 隨機值，因此當 entropy pool 空了後，將導致 `/dev/random` 被暫停，直到搜集了足夠的 entorpy。而 `/dev/urandom` 則不會有這樣的問題出現，但在 entropy pool 內資訊不足時，可能導致取出的隨機值質量較低。如果是 OSX 或 FreeBSD 的使用者，其 `/dev/random` 與 `/dev/urandom` 是相同的，可以在 man page 中看到相關訊息。
