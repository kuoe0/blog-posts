---
layout: post
title:  "[C/C++] 計時"
date:   2011-10-03
tags:   ["C/C++", "time | 計時"]
feature:
    photo:       true
    photo_url:   "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2011-10-03-c-cplusplus-timing.jpg"
    creator:     "Jinx!"
    url:         "https://www.flickr.com/photos/7567658@N04/3462680490/"
    license:     "CC-BY-SA 2.0"
    license_url: "https://creativecommons.org/licenses/by-sa/2.0/"
---

C/C++ 中的使用 clock() 函數來計時，回傳的資料型別為 clock_t。
使用該函式與該資料型別時，必須要引入標頭檔 time.h（C++可引入 ctime 此標頭檔）。

**Definition**

函式定義為：`clock_t clock(void);`，其回傳系統當前的 CPU time （tick 數）。

`clock_t` 為保留時間用的資料型別，其定義為：

```c++
#ifndef     _CLOCK_T_DEFINED
typedef     long  clock_t;
#define     _CLOCK_T_DEFINED
#endif
```

由以上程式碼可得知，`clock_t` 為一個整數型別。

另外，在標頭檔中定義了一個常數 `CLOCKS_PER_SEC` 表示每秒會有幾次的 CPU 單位時間（tick）。因此可以使用 `clock() / CLOCKS_PER_SEC` 得到該程式目前執行的時間。

**Example**

```c++
#include <time.h>
#include <stdio.h>

int main() {
    clock_t begin, end;
    double timeCnt;
    
    begin = clock();
    /*
    ANYTING_YOU_WANT_TO_COUNT
    */
    end = clock();
    timeCnt = (double)( end - begin ) / CLOCKS_PER_SEC;
    printf( "Time: %lf sec\n", timeCnt );
    
    return 0;
}
```

上述範例，即可用來計算想要計時的程式片段所執行的時間長度。修改 `ANYTING_YOU_WANT_TO_COUNT` 部份為想要計時的程式碼。
