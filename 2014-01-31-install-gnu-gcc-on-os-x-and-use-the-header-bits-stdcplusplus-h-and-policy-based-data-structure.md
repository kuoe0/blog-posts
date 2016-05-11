---
layout: post
title:  "在 OS X 中安裝 GNU GCC 以及使用  bits/stdc++.h 標頭檔與 Policy-Based Data Structure"
date:   2014-01-31
tags:   ["OS X", "GCC", "LLVM", "contest | 競賽"]
feature:
    photo: false
---

系統環境：

- OS X: 10.9
- Homebrew: 0.9.5
- Xcode: 5.0.2

前陣子去參加台大的程式競賽集訓營才知道原來竟然有 `bits/stdc++.h` 這個標頭檔，根本就競賽神器！基本上，只要 `#include <bits/stdc++>` 就相當於引入了**所有標準函式庫**。每次競賽時，總是要先打好以下內容：

```c++
#include <iostream>
#include <string>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <vector>
#include <set>
#include <map>
#include <deque>
#include <stack>
#include <queue>
#include <algorithm>
```

然後看題目有幾題，就複製幾份...突然覺得自己有點愚蠢。只要 `#include <bits/stdc++.h>` 就比我打得那麼大一串還更強大。CodeForces 上有一篇文（[連結](http://codeforces.com/blog/entry/8387)）提供了一份提供競賽快速使用得模板：

```c++
#include <bits/stdc++.h>
#define _ ios_base::sync_with_stdio(0);cin.tie(0);

using namespace std;

int main() { _
	
	return 0;
}
```

另外就是發現有 pb_ds 這 GNU 的擴充，pb_ds 全名是「Policy-Based Data Structure」。裡頭也包含了各式各樣的容器 (container)，例如：tree、hash table、heap 以及 list 等資料結構。以下是一個簡單的 tree 實現 STL map 的範例：

```c++
#include <iostream>
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>

using namespace std;
using namespace __gnu_pbds;

typedef tree<int, int, less<int>, rb_tree_tag, tree_order_statistics_node_update> map_t;

int main() {
	int n;
	while (~scanf("%d", &n)) {
		map_t s;

		for (int i = 0; i < n; ++i) {
			int k, v;
			scanf("%d %d", &k, &v);
			s.insert(pair<int, int>(k, v));
		}
		
		for (int i = 0; i < n; ++i) {
			printf("%d: %d\n", s.find_by_order(i)->first, s.find_by_order(i)->second);
		}
	}
	return 0;
}

```

不過其實我還沒仔細研究，想了解更多資訊可以參考以下網址：

- [Policy-Based Data Structures](http://gcc.gnu.org/onlinedocs/libstdc++/ext/pb_ds/)
- [libstdc++: Policy-Based Data Structures](http://gcc.gnu.org/onlinedocs/gcc-4.8.2/libstdc++/api/a01740.html)
- [Chapter 22. Policy-Based Data Structures](http://gcc.gnu.org/onlinedocs/libstdc++/manual/policy_data_structures.html)

但以上兩個強大的標頭檔的使用限制是得使用 GNU 所提供的 GCC，但 Xcode 的 Command Line Tools 提供的是 clang，一個 LLVM 的 C/C++/Objective-C 的前端編譯器。而 clang 就沒有支援前面提到的 `bit/stdc++.h` 以及 Policy-Based Data Structure 的支援了。但是 Xcode 也有提供 gcc 與 g++，不過應該稱之為 llvm gcc 與 llvm g++。該兩個命令與 clang 以及 clang++ 的差別似乎僅僅在於使用的函式庫不同而已。其採用的是與 GNU GCC 相似的函式庫，不過可惜的是，就是沒有支援 `bits/stdc++.h` 與 Policy-Based Data Structure。但一般競賽中，通常都會使用 Linux 進行（有少數是 Windows），所以大部分應該都會有 GNU GCC，所以應該不需要太擔心，只是要注意一下 GCC 的版本。目前僅測試 GCC 4.8.2 與 GCC 4.9.0 上均可以正常編譯，由於 homebrew 似乎遺失了 GCC 4.7 的連結，也懶得自己去找原始碼編譯了，所以就沒有測試了。

要利用 homebrew 來安裝 GNU GCC 4.8 的話，首先需要增加 package repo，加入 `homebrew/versions` 這個 repo，指令如下：

```bash
$ brew tap homebrew/versions
```

接下來更新一下 homebrew，即可以開始安裝 GNU GCC 4.8，指令如下：

```bash
$ brew update
$ brew install gcc48
```

題外話：

在嘗試用 llvm g++ 編譯引入 `bits/stdc++.h` 標頭檔的原始碼時，一直顯示找不到該標頭檔，所以好奇想去查一下編譯器搜尋的函式庫資料夾位置。相信許多人的電腦裡應該也都有著不少的 `include` 資料夾。查到可以利用以下指令來查詢編譯器會去查找的資料夾位置：

```bash
$ g++ -E -x c++ - -v < /dev/null
```

該指令同樣也適用於 clang 上，所以也可以替換掉 g++ 為 clang++，如下：

```bash
$ clang++ -E -x c++ - -v < /dev/null
```
