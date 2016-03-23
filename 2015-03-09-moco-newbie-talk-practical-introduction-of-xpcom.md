---
layout:  post
title:   "謀智菜逼八談 XPCOM 實務入門"
date:    2015-03-09
tags:    ["Mozilla", "Firefox", "XPCOM"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

本文同步刊載於[謀智台客](http://tech.mozilla.com.tw/posts/6010/%E8%AC%80%E6%99%BA%E8%8F%9C%E9%80%BC%E5%85%AB%E8%AB%87-xpcom-%E5%AF%A6%E5%8B%99%E5%85%A5%E9%96%80)。

## 前言

關於 XPCOM，請參考 MDN 上 [XPCOM 的簡介](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XPCOM/Guide/Creating_components/An_Overview_of_XPCOM)。MDN 上關於 XPCOM 的教學，有部分實作上的細節已經過時了。此外，MDN 上的教學偏向使用 XPCOM 編寫 gecko 的擴充元件，身為 Mozilla 菜逼八軟體工程師要學的當然是給 gecko 內部使用的 XPCOM！因此在編寫上會有些微不同的地方。這篇文章將修改 gecko 附帶的 XPCOM 範例，並將其加入 necko 模組中。

### 下載 Gecko 程式碼

**使用 Git**

```
$ git clone https://github.com/mozilla/gecko-dev firefox-dev
```

**使用 Mercurial**

```
$ hg clone https://hg.mozilla.org/mozilla-central firefox-dev
```

- Necko 模組目錄：[firefox-dev/netwerk](https://dxr.mozilla.org/mozilla-central/source/netwerk)
	- 這個 netwerk 絕對不是拼錯，只是個荷蘭文的網路。
- XPCOM 範例目錄：[firefox-dev/xpcom/sample](https://dxr.mozilla.org/mozilla-central/source/xpcom/sample)

## Module 進入點

在 MDN 的 [XPCOM 介紹](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XPCOM/Guide/Creating_components/Creating_the_Component_Code#Coding_for_the_Registration_Process)中，描述了所有 module 都需要提供一個名為 `NSGetModule` 的函式作為進入點。但自 Gecko 2.0 後，將不再使用這樣的界面。改為提供一個型別為 `mozilla::Module` 且命名為 `NSModule` 的外部變數作為 module 的進入點。Necko 模組的進入點寫在 [firefox-dev/netwerk/build/nsNetModule.cpp](https://dxr.mozilla.org/mozilla-central/source/netwerk/build/nsNetModule.cpp)。可以在[最後一行](https://dxr.mozilla.org/mozilla-central/source/netwerk/build/nsNetModule.cpp#1104)發現以下程式碼：

```cpp
NSMODULE_DEFN(necko) = &kNeckoModule;
```

其中 `NSMODULE_DEFN` 為一個 macro，其定義為：

```cpp
#define NSMODULE_DEFN(_name) extern NSMODULE_SECTION mozilla::Module const *const NSMODULE_NAME(_name)
```

將原本的程式碼展開後如下：

```cpp
extern NSMODULE_SECTION mozilla::Module const *const NSMODULE_NAME(necko) = &kNeckoModule;
```

然而，該 macro 返回後的結果仍然包含一個名為 NSMODULE_NAME 的 marco，因此會再進行一次展開。

```cpp
#define NSMODULE_NAME(_name) _name##_NSModule
```
	
最後展開結果為如下：

```cpp
extern NSMODULE_SECTION mozilla::Module const *const _necko_NSModule = &kNeckoModule;
```

以上宣告表示為一個名為 `_necko_NSModule ` 的變數其型別為一個指向常數 `mozilla::Module` 的常數指標。如何判讀以上宣告可以參考 StackOverflow 上這篇 [what is the difference between const int*, const int * const, int const * t](http://stackoverflow.com/questions/1143262/what-is-the-difference-between-const-int-const-int-const-int-const)。

備註：若為非定義 `MOZILLA_INTERNAL_API` 的 module，其 module 的外部變數的名稱皆為 `NSModule`。詳細請參考 [firefox-dev/source/xpcom/components/Module.h](https://dxr.mozilla.org/mozilla-central/source/xpcom/components/Module.h#119) 。

## XPCOM 目錄結構與編譯設定

以 [firefox-dev/netwerk/base](https://dxr.mozilla.org/mozilla-central/source/netwerk/base) 為例，[firefox-dev/netwerk/base](https://dxr.mozilla.org/mozilla-central/source/netwerk/base) 下會包含了非常多個 XPCOM，可以在 base 目錄下找到 public 與 src 這兩個目錄。其中，XPCOM 界面的部分都將被放在 public 目錄，而實作的部分則會放在 src 目錄。介面的檔案主要包含一些 [XPIDL](https://developer.mozilla.org/en-US/docs/Mozilla/XPIDL) (.idl) 以及標頭檔 (.h) 檔案，實作的檔案包含一些 C++ (.cpp)、標頭檔 (.h)、JavaScript (.js) 以及 JavaScript Module (.jsm) 檔案。

因此只需要將寫好的 XPIDL 檔案放置於 public 目錄中，並將實作的 C++/JavaScript 程式碼放置於 src 目錄中。接下來，只需要將新增的 XPCOM 寫入 moz.build 中，使得 Mozilla 的 build system 知道要編譯該 XPCOM。

備註：不是所有 XPCOM 都與 [firefox-dev/netwerk/base](https://dxr.mozilla.org/mozilla-central/source/netwerk/base) 一樣，有些 XPCOM 可能會把 public 目錄取名為 interfaces 之類的。甚至可能並沒有分類，直接將介面與實作的程式碼都混在一起。

### moz.build

在 [firefox-dev/netwerk/base](https://dxr.mozilla.org/mozilla-central/source/netwerk/base) 中，會有三份 moz.build 存在：

- [firefox-dev/netwerk/base/moz.build](https://dxr.mozilla.org/mozilla-central/source/netwerk/base/moz.build)
- [firefox-dev/netwerk/base/public/moz.build](https://dxr.mozilla.org/mozilla-central/source/netwerk/base/public/moz.build)
- [firefox-dev/netwerk/base/src/moz.build](https://dxr.mozilla.org/mozilla-central/source/netwerk/base/src/moz.build)

**firefox-dev/network/base/moz.build**

```
DIRS += ['public', 'src']
```

`DIRS` 變數會儲存該目錄下，希望能夠被 build system 掃描的目錄名稱。

**firefox-dev/netwerk/base/public/moz.build**

```python
XPIDL_SOURCES += [
	...
]

if CONFIG['MOZ_TOOLKIT_SEARCH']:
	XPIDL_SOURCES += [
		...
	]

XPIDL_MODULE = 'necko'

EXPORTS += [
	...
]

EXPORTS.mozilla.net += [
	...
]
```

`XPIDL_SOURCES` 儲存所有需要被編譯的 XPIDL 檔案列表。要注意的是，該檔案列表須按照字典序排序，否則會報錯導致編譯失敗。

`XPIDL_MODULE` 定義該目錄下的 XPCOM 編譯後所屬的 module，以 netwerk 為例，所有的 XPCOM 都會被編譯到 necko 這個 module 中。

`EXPORTS` 儲存可以被外部程式引入的 header 列表。

`EXPORTS.mozilla.net` 儲存可以被外部程式引入的結構化 header 列表。與 EXPORTS 不同的是 EXPORTS 預設將 header 放置在最頂層的目錄。而 EXPORTS.mozilla.net 則會將 header 放置於 mozilla/net/ 目錄下。舉個例子：

```python
EXPORTS += [ 'foo.h' ]
EXPORTS.mozilla.net += [ 'bar.h' ]
```

若要引入上述兩個 header 的話，需要依照以下的方式引入：

```cpp
#include <foo.h>
#include <mozilla/net/bar.h>
```

其中，mozilla.net 也可以用其他名稱替代。每多一個小數點，就代表一個子目錄。

`if CONFIG['MOZ_TOOLKIT_SEARCH']:` 可以用來進行一些條件判斷來使用不同的設定。基本上 moz.build 本身是 Python code，因此其條件判斷式使用方式與 Python 相同。

**firefox-dev/netwerk/base/src/moz.build**

```python
UNIFIED_SOURCES += [
	...
]

if CONFIG['MOZ_WIDGET_TOOLKIT'] == 'windows':
	SOURCES += [
		...
	]

EXTRA_JS_MODULES += [
...
]
```

`UNIFIED_SOURCES` 儲存可以被一起編譯的 C/C++ 程式碼。UNIFIED 的機制是把所有 C++ 程式碼串接起來，並對串接後的檔案進行一次編譯，這麼做的理由是為了減少編譯時間。新加入的 XPCOM 都要盡可能可以加入 UNIFIED_SOURCES。

> 為什麼 UNIFIED 的機制可以減少編譯時間：
> 
> - 避免一直引入相同的標頭檔
> - 避免一直實例化相同的 template classes/functions (C++ only)
> - 避免一直擴展相同的 inline functions (C++ only)

`SOURCES` 儲存要被單獨編譯的 C/C++ 程式碼。這些程式碼如果跟其他程式碼串接起來一起編譯的話可能會有錯誤出現，因此需要單獨編譯。

`EXTRA_JS_MODULES` 必要的 JavaScript 檔案。

更多關於 Mozilla build system 的參考資料：

- [Mozilla Build System: past, present and future](http://glandium.org/blog/?p=3318)
- [UNIDIED_SOURCES 原始討論串](https://groups.google.com/forum/#!searchin/mozilla.dev.platform/unified$20source/mozilla.dev.platform/WjcCfckml4A/y27nUc8k8gcJ)
- [moz.build](https://developer.mozilla.org/en-US/docs/How_Mozilla's_build_system_works#moz.build_Files。)

## 註冊 XPCOM

完成一個 XPCOM 後，需要將其註冊到 gecko 中，這樣 gecko 才知道有這個 XPCOM 的存在。以 necko 來說，其註冊資訊都寫在 [firefox-dev/netwerk/build/nsNetModule.cpp](https://dxr.mozilla.org/mozilla-central/source/netwerk/build/nsNetModule.cpp) 這個檔案中。

### Prerequisite

- 給定 XPCOM 一個唯一的 CID
	- 利用 `NS_DEFINE_NAMED_CID` 來宣告 XPCOM 的 CID 變數
	- [程式碼參考](https://dxr.mozilla.org/mozilla-central/source/xpcom/glue/nsID.h?from=NS_DEFINE_NAMED_CID&case=true#94)
- 創建符合 XPCOM 規格的 factory constructor
	- 使用 `NS_GENERIC_FACTORY_CONSTRUCTOR` 來替 XPCOM 建立 factory constructor
	- 如果特殊需求可以選擇使用 `NS_GENERIC_FACTORY_CONSTRUCTOR_INIT` 或是 `NS_GENERIC_FACTORY_SINGLETON_CONSTRUCTOR`
	- [程式碼參考](https://dxr.mozilla.org/mozilla-central/source/xpcom/components/ModuleUtils.h?from=NS_GENERIC_FACTORY_CONSTRUCTOR#12)

### Registration

- CID 對應到  Factory Constructor
	- 添加在 `static const mozilla::Module::CIDEntry kNeckoCIDs[]` 陣列中
- Contract ID  對應到 CID
	- 添加在 `static const mozilla::Module::ContractIDEntry kNeckoContracts[]` 陣列中

在 gecko 啟動後，XPCOM service 將會把 Contract ID、CID 以及 Factory Constructor 對應的關係儲存到註冊的表格中。因此其他程式其可以夠透過 Contract ID 來使用該 XPCOM。

## 測試 XPCOM

在 [firefox-dev/netwerk/test](https://dxr.mozilla.org/mozilla-central/source/netwerk/test) 目錄下，可以找到許多測試程式。在編譯完 firefox-dev 後，可以在 firefox-dev/obj-x86_64-apple-darwin13.4.0/dist/bin 該目錄下找到這些測試程式的執行檔。假設我們要執行 TestBind 該測試程式，輸入以下指令指令即可：

	$ ./mach cppunittest obj-x86_64-apple-darwin13.4.0/dist/bin/TestBind

第一次執行測試程式都請務必使用 mach 來進行，否則可能會發生找不到動態函式庫的錯誤發生。有興趣可以到 [firefox-dev/testing/runcppunittests.py](https://dxr.mozilla.org/mozilla-central/source/testing/runcppunittests.py#19) 了解 `mach cppunittest` 如何設定環境，使得動態函式庫可以被正確載入。

備註：不同平台的目標目錄名稱不完全相同，請找 obj-YOUR-PLATFORM 這個目錄。

## 移植 XPCOM 範例

### nsSample2 元件

首先，將位於 [firefox-dev/xpcom/sample](https://dxr.mozilla.org/mozilla-central/source/xpcom/sample) 底下的 XPCOM 範例移植到 necko 中，我們將把該 XPCOM Component 放置於 [firefox-dev/netwerk/base](https://dxr.mozilla.org/mozilla-central/source/netwerk/base) 中。先將必要的檔案複製到指定目錄，由於這個範例 nsSample 本身也會被編譯出來，所以該範例都將改名為 Sample2：

```
$ cp firefox-dev/xpcom/sample/nsISample.idl firefox-dev/netwerk/base/public/nsISample2.idl
$ cp firefox-dev/xpcom/sample/nsSample.cpp firefox-dev/netwerk/base/src/nsSample2.h
$ cp firefox-dev/xpcom/sample/nsSample.cpp firefox-dev/netwerk/base/src/nsSample2.cpp
```

接下來，將開始修改這些檔案令 nsSample2 可以成功編譯。

**nsISample2.idl**

為了避免跟 nsISample 衝突，因此需要給 nsISample2 另一個 uuid 作為 IID，可以使用系統內建的 `uuidgen` 這個指令來產生。不過建議使用位於 firefox-dev 目錄下的 mach 程式來產生，使用方式如下：

```
$ ./mach uuid
d1a85c09-f610-4a53-ba7b-ea7698e9de17

{ 0xd1a85c09, 0xf610, 0x4a53, \
{ 0xba, 0x7b, 0xea, 0x76, 0x98, 0xe9, 0xde, 0x17 } }
```

假設給定的新的 IID 為 `d1a85c09-f610-4a53-ba7b-ea7698e9de17`。此外，所有介面名稱也需要改成 nsISample2。因此 nsISample2.idl 的部分程式碼會改為以下所示：

```idl
[scriptable, uuid(d1a85c09-f610-4a53-ba7b-ea7698e9de17)]
interface nsISample2 : nsISupports
{
	attribute string value;
	void writeValue(in string aPrefix);
	void poke(in string aValue);
};
```

**nsSample2.h**

同樣的，為了避免與 nsSample 衝突，需要給 nsSample2 一個新的 CID 以及 Contract ID。CID 的部分一樣用 `./mach uuid` 產生，輸出的第二種格式就是 CID 所需要的格式。而 Contract ID 則給定為 `@mozilla.org/sample2;1`，所以部分程式碼將改成以下所示：

```cpp
#define NS_SAMPLE2_CID		{ 0x8ab22c97, 0x9322, 0x49ec, \
							{ 0x96, 0xf2, 0xe4, 0x34, 0x1a, 0xfc, 0x2d, 0x1b } }

#define NS_SAMPLE2_CONTRACTID "@mozilla.org/sample2;1"
```

除此之外，所有 nsISample 都需要替換成 nsISample2 以及 nsSampleImpl 都需要替換成 nsSample2Impl。

**nsSample2.cpp**

同 nsSample2.h 一樣，需要把所有包含 nsISample 及 nsSampleImpl 都進行替換。另外，把所有使用到 nsEmbedString 的部分都去除，連同引入標頭檔的部分。由於這個名為 nsSample2 的 XPCOM 是要用來作為 gecko 內部使用的，並不是一個外部擴充元件。nsEmbedString 這個函式庫僅是個提供外部 XPCOM 使用的 API。

### 修改 moz.build

要令 Mozilla build system 可以編譯該 XPCOM，需要修改兩個 moz.build 檔案。分別是：

- [firefox-dev/netwerk/base/public/moz.build](https://dxr.mozilla.org/mozilla-central/source/netwerk/base/public/moz.build)
	- 將 nsISample2.idl 加入 XPIDL_SOURCES 這個列表中。
- [firefox-dev/netwerk/base/src/moz.build](https://dxr.mozilla.org/mozilla-central/source/netwerk/base/src/moz.build)
	- 將 nsSample2.cpp 加入 UNIFIED_SOURCES 這個列表中。

***請注意在將檔案加入列表中時，是否有按照字典序排列。***

### 註冊 nsSample2 元件

在使用 nsSample2 的 XPCOM 前，需要先進行註冊。註冊資訊都將寫在 [firefox-dev/netwerk/build/nsNetModule.cpp](https://dxr.mozilla.org/mozilla-central/source/netwerk/build/nsNetModule.cpp)。首先，在 nsNetModule.cpp 中引入 nsSample2.h 標頭檔。再替 nsSample2 宣告其 CID 以及創建其 factory constructor。

**宣告 CID**

```cpp
NS_DEFINE_NAMED_CID(NS_SAMPLE2_CID);
```

**創建 factory constructor**

```cpp
NS_GENERIC_FACTORY_CONSTRUCTOR(nsSample2Impl)
```

在 nsSample2 有 CID 以及 factory constructor 後，在把其對應關係加到註冊的表格中即可，部分程式碼如下：

```
static const mozilla::Module::CIDEntry kNeckoCIDs[] = {
  ...
  { &kNS_SAMPLE2_CID, false, nullptr, nsSample2ImplConstructor },
  ...
};

static const mozilla::Module::ContractIDEntry kNeckoContracts[] = {
  ...
  { NS_SAMPLE2_CONTRACTID, &kNS_SAMPLE2_CID },
  ...
};
```

重新編譯後，nsSample2 該 XPCOM 就成功註冊了！

### 測試 nsSample2

這邊就不參考範例測試程式了。除了程式碼過時外，也因為我們要寫的是用於 gecko 內部的 XPCOM，所以編寫方式會有些微不同。程式碼如下：

```cpp
#include "TestHarness.h"
#include "nsISample2.h"
#include "prerror.h"

using namespace mozilla;

int
main()
{
  ScopedXPCOM xpcom("Sample2");

  if (xpcom.failed()) {
	fail("Unable to initialize XPCOM");
	return -1;
  }

  nsresult rv;

  nsCOMPtr<nsISample2> sample = do_CreateInstance("@mozilla.org/sample2;1", &rv);
  if (NS_FAILED(rv)) {
	fail("Failed to create sample2.");
	  printf("Error code: 0x%X\n", rv);
	return -1;
  }


  rv = sample->WriteValue("Initial print:");
  if (NS_FAILED(rv)) {
	printf("Error code: 0x%X\n", rv);
	return -3;
  }

  const char* testValue = "XPCOM defies gravity";
  rv = sample->SetValue(testValue);
  if (NS_FAILED(rv)) {
	printf("Error code: 0x%X\n", rv);
	return -3;
  }
  printf("Set value to: %s\n", testValue);
  char* str;
  rv = sample->GetValue(&str);

  if (NS_FAILED(rv)) {
	printf("Error code: 0x%X\n", rv);
	return -3;
  }
  if (strcmp(str, testValue)) {
	printf("Test FAILED.\n");
	return -4;
  }

  NS_Free(str);

  rv = sample->WriteValue("Final print :");
  printf("Test passed.\n");

  passed("Sample2 Works");

  return 0;
}
```

在測試的部分，使用 ScopedXPCOM 物件來啟動 XPCOM 服務。沒有啟動 XPCOM 服務的話，將會導致 XPCOM 物件無法被初始化。使用 ScopedXPCOM 物件必須要引入 TestHarness.h 標頭檔才能夠使用，並且 ScopedXPCOM 物件也僅能被用來撰寫測試。而給與 ScopedXPCOM 物件的字串參數純粹只是該次測試的名稱，真的想要隨便給也是可以。啟動 XPCOM 服務後，即可透過 `do_CreateInstance` 來創建 nsSample2 的物件，程式碼如下：

```cpp
nsCOMPtr<nsISample2> sample = do_CreateInstance("@mozilla.org/sample2;1", &rv);
```

接下來就可以透過操作 nsSample2 的物件來進行測試了！

## 疑難排解

**error: nsStringAPI.h is only usable from non-MOZILLA_INTERNAL_API code!**

因為我們修改的範例程式碼使用了 nsEmbedString.h 這個標頭檔，在 nsEmbedString.h 中引入了 nsStringAPI.h，而 nsStringAPI.h 是僅提供給外部使用的函式庫。Gecko 內部若要使用字串相關函式庫，請使用 nsAString.h 或是 nsACString.h 這兩個函式庫。關於內部使用的字串函式庫可以參考 MDN 上的 [Mozilla internal string guide](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XPCOM/Guide/Internal_strings)。

**Error Code: 0xC1F30001**

`0xC1F30001` 這個錯誤碼表示 `NS_ERROR_NOT_INITIALIZED`。可能造成此錯誤的原因如下：

- XPCOM 服務沒啟動導致 XPCOM 物件無法初始化。
- 沒有註冊 XPCOM 元件。

**其他錯誤碼**

如出現其他錯誤碼，可以參考 MDN 上的 [Error codes returned by Mozilla APIs](https://developer.mozilla.org/en-US/docs/Mozilla/Errors)，來查找錯誤的發生原因。

