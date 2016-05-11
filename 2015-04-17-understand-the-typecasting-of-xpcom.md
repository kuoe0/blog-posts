---
layout: post
title:  "看懂 XPCOM 的轉型函式"
date:   2015-04-17
tags:   ["Mozilla", "Firefox", "XPCOM", "typecasting | 轉型"]
feature:
    photo:       true
    photo_url:   "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2015-04-17-understand-the-typecasting-of-xpcom.jpg"
    creator:     "Alan Levine"
    url:         "https://www.flickr.com/photos/cogdog/8662368656"
    license:     "CC 2.0"
    license_url: "https://creativecommons.org/licenses/by/2.0/"
---

本文同步刊載於[謀智台客](http://tech.mozilla.com.tw/posts/6245/%E7%9C%8B%E6%87%82-xpcom-%E7%9A%84%E8%BD%89%E5%9E%8B%E5%87%BD%E5%BC%8F)。

在研讀 Gecko 的程式碼時，常常會看到 `do_QueryInterface` 與 `do_GetInterface` 的用法出現，另外還有一個比較少見的 `do_QueryObject`。這三個函式的主要用途都是對於一個物件請求另一個介面的物件。在介紹這三個函式之前，我們先介紹 `QueryInterface` 與 `GetInterface` 這兩個函式的用途。

## `QueryInterface`

讓我們先從 `QueryInterface` 講起。在 Gecko 中，所有的 XPCOM 元件都是繼承自 **nsISupports** 這個類別。在 **nsISupports** 中，定義了三個成員函式，分別是：

- `AddRef`
- `Release`
- **`QueryInterface`**

這邊要探討的就是其中的 `QueryInterface` 這個成員函式。一般來說，XPCOM 元件都會繼承不同的 XPIDL interface，要查詢一個物件是否「**具備**」某個 XPIDL interface 就可以使用 `QueryInterface` 來進行。若該物件具備該 XPIDL interface 可以使用的話，`QueryInterface` 就會以該 XPIDL interface 來回傳該物件的指標。也就是說，該物件的類別本身有繼承該 XPIDL interface，所以才能用 `QueryInterface` 來取得。因此，經過 `QueryInterface` 取得的物件與原本的物件其實還是同一個物件，所以同樣能夠用 `QueryInterface` 來返回原本的物件。也就是說，`QueryInterface` 是個 interface 間的雙向通道。

> P.S. 當類別被定義為可以被 cycle collector 偵測時，便可以向該類別請求 **nsCycleCollectionISupports** 與 **nsXPCOMCycleCollectionParticipant** 這兩個 interface。而這兩個 interface 本身並不是 XPCOM，因此並無法再透過 `QueryInterface` 將已經被轉型為這兩個 interface 的物件轉型回原本的 interface。

![QueryInterface](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2015-04-17-understand-the-typecasting-of-xpcom-1.png)

而 `QueryInterface` 的函式定義不需要自己編寫，編譯器可以幫我們產生。在編寫 XPCOM 元件時，只要加入 `NS_DECL_ISUPPORTS` 這個敘述在類別的定義中即可使得該類別具有 **nsISupports** 的介面。而在成員函式的實作中，只要加上 `NS_IMPL_ISUPPORTS(class name, interface1, interface2, ...)`，編譯器就會產生相對應的 **`QueryInterface`**、`AddRef` 與 `Release` 這三個成員函式的定義。如果是使用 JavaScript 所寫的 XPCOM 元件的話，只要在 prototype 中加入 `QueryInterface: XPCOMUtils.generateQI([Ci.interface1, Ci.interface2, ...])` 這個項目，即可自動產生相對應的 `QueryInterface` 函式。

## `GetInterface`

`GetInterface` 來自 **nsIInterfaceRequestor** 這個 XPIDL interface。前面提到的 `QueryInterface` 是用來查詢物件是否「具備」某個 XPIDL interface，所以該物件的類別必須有繼承要查詢的 XPIDL interface 才能返回結果。而 `GetInterface` 則是用來向物件「**請求**」某個 XPIDL interface 的物件，而這個物件可以不必是該物件的類別所繼承的。所以回傳的結果可能是該類別的成員物件、另外創建的物件或是物件本身。因此，經由 `GetInterface` 取得的物件，大部分都不是同一個物件。這使得該物件並不一定能夠用 `GetInterface` 來返回原本的物件。

![GetInterface](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2015-04-17-understand-the-typecasting-of-xpcom-2.png)

若要令一個類別支援 `GetInterface` 必須要讓該類別繼承 **nsIInterfaceRequestor** 這個 interface。並且，**需要自行編寫 `GetInterface` 的函式定義**。因為每個類別針對不同 XPIDL interface 的請求會有不同的需求，所以需要自己定義。

以 **nsDocShell** 的 `GetInterface` 片段程式碼為例：

```cpp
NS_IMETHODIMP
nsDocShell::GetInterface(const nsIID& aIID, void** aSink)
{
  NS_PRECONDITION(aSink, "null out param");

  *aSink = nullptr;

  if (aIID.Equals(NS_GET_IID(nsICommandManager))) {
	NS_ENSURE_SUCCESS(EnsureCommandHandler(), NS_ERROR_FAILURE);
	*aSink = mCommandManager;
  } else if (aIID.Equals(NS_GET_IID(nsIURIContentListener))) {
	*aSink = mContentListener;
  } else if ((aIID.Equals(NS_GET_IID(nsIScriptGlobalObject)) ||
			  aIID.Equals(NS_GET_IID(nsIGlobalObject)) ||
			  aIID.Equals(NS_GET_IID(nsPIDOMWindow)) ||
			  aIID.Equals(NS_GET_IID(nsIDOMWindow)) ||
			  aIID.Equals(NS_GET_IID(nsIDOMWindowInternal))) &&
			 NS_SUCCEEDED(EnsureScriptEnvironment())) {
	return mScriptGlobal->QueryInterface(aIID, aSink);
  } else if (aIID.Equals(NS_GET_IID(nsIDOMDocument)) &&
			 NS_SUCCEEDED(EnsureContentViewer())) {
	mContentViewer->GetDOMDocument((nsIDOMDocument**)aSink);
	return *aSink ? NS_OK : NS_NOINTERFACE;
  } else if (aIID.Equals(NS_GET_IID(nsIDocument)) &&
			 NS_SUCCEEDED(EnsureContentViewer())) {
	nsCOMPtr<nsIDocument> doc = mContentViewer->GetDocument();
	doc.forget(aSink);
	return *aSink ? NS_OK : NS_NOINTERFACE;
  } else if (aIID.Equals(NS_GET_IID(nsIApplicationCacheContainer))) {
	*aSink = nullptr;

	// Return application cache associated with this docshell, if any
	...
  }
  ...
  NS_IF_ADDREF(((nsISupports*)*aSink));
  return *aSink ? NS_OK : NS_NOINTERFACE;
}
```

我們可以發現 `GetInterface` 其實就是一堆 `if-else` 的判斷敘述，並根據不同的 XPIDL interface 的 IID 來返回不同的對象。

## `do_QueryInterface` 與 `do_GetInterface`

讓我們回到 `do_QueryInterface`、`do_GetInterface` 以及 `do_QueryObject` 這三個函式。不過這邊我們先介紹 `do_QueryInterface` 與 `do_GetInterface`，稍後在介紹 `do_QueryObject`。先以 `NewChannelFromURIWithProxyFlagsInternal` 這個函式內的片段程式碼為例：

```cpp
nsCOMPtr<nsIProtocolHandler> handler;
rv = GetProtocolHandler(scheme.get(), getter_AddRefs(handler));

// do something

nsCOMPtr<nsIProxiedProtocolHandler> pph = do_QueryInterface(handler);
if (pph) {
  // do something
}
else {
  // do something
}
```

在這段程式碼中，一開始先拿到了一個指向 **nsIProtocolHandler** 的 **nsCOMPtr** 物件，接著便呼叫 `do_QueryInterface` 來嘗試取得指向 **nsIProxiedProtocolHandler** 的 **nsCOMPtr** 物件。這邊很神奇的是，為什麼 `do_QueryInterface` 並不需要給予 **nsIProxiedProtocolHandler** 的 IID 來進行查詢就可以返回相對應的物件呢？

這邊我們可以看到 `do_QueryInterface` 的原始碼：

```cpp
inline nsQueryInterface
do_QueryInterface(nsISupports* aRawPtr)
{
  return nsQueryInterface(aRawPtr);
}
```

這裏很單純的返回一個用原始物件初始化的 **nsQueryInterface** 物件。而這個物件將被回傳給身為 **nsCOMPtr\<nsIProxiedProtocolHandler\>** 型別的變數 `pph`。此時，C++ 會默默的呼叫 **nsCOMPtr\<nsIProxiedProtocolHandler\>** 的建構子。其定義如下：

```cpp
MOZ_IMPLICIT nsCOMPtr(const nsQueryInterface aQI)
  : NSCAP_CTOR_BASE(0)
{
  NSCAP_LOG_ASSIGNMENT(this, 0);
	assign_from_qi(aQI, NS_GET_TEMPLATE_IID(T));
}
```

這裡可以發現終於有 IID 的資訊了。只要呼叫 `NS_GET_TEMPLATE_IID(T)` 即可得到 **nsIProxiedProtocolHandler** 的 IID 了。再將 `do_QueryInterface` 所回傳的 **nsQueryInterface** 物件與 XPIDL interface 的 IID 作為參數來呼叫 `assign_from_qi`，其定義如下：

```cpp
template<class T>
void
nsCOMPtr<T>::assign_from_qi(const nsQueryInterface aQI, const nsIID& aIID)
{
  void* newRawPtr;
  if (NS_FAILED(aQI(aIID, &newRawPtr))) {
	newRawPtr = 0;
  }
  assign_assuming_AddRef(static_cast<T*>(newRawPtr));
}
```

在 `assign_from_qi` 裡面，**nsQueryInterface** 竟然被作為函式呼叫。讓我們來看到 **nsQueryInterface** 的定義：

```cpp
class MOZ_STACK_CLASS nsQueryInterface MOZ_FINAL
{
public:
  explicit
  nsQueryInterface(nsISupports* aRawPtr) : mRawPtr(aRawPtr) {}

  nsresult NS_FASTCALL operator()(const nsIID& aIID, void**) const;

private:
  nsISupports* MOZ_OWNING_REF mRawPtr;
};
```

這裡定義了 **nsQueryInterface** 的 functor (`operator()`)，所以 **nsQueryInterface** 該類別的物件可以被作為函式進行呼叫。而其定義如下：

```cpp
nsresult
nsQueryInterface::operator()(const nsIID& aIID, void** aAnswer) const
{
  nsresult status;
  if (mRawPtr) {
	status = mRawPtr->QueryInterface(aIID, aAnswer);
#ifdef NSCAP_FEATURE_TEST_NONNULL_QUERY_SUCCEEDS
	NS_ASSERTION(NS_SUCCEEDED(status),
				 "interface not found---were you expecting that?");
#endif
  } else {
	status = NS_ERROR_NULL_POINTER;
  }

  return status;
}
```

就在這個 functor 的定義中，終於讓發現 `QueryInterface` 被呼叫的蹤跡了。如此一來，就可以拿到指向 **nsIProxiedProtocolHandler** 介面的 **nsCOMPtr** 物件了。

讓我們整理一下從 `do_QueryInterface` 進行 interface 的請求的流程：

1. `do_QueryInterface` 返回 **nsQueryInterface** 物件。
2. 將回傳的 **nsQueryInterface** 物件作為參數來呼叫 **nsCOMPtr\<T\>** 的建構子。
3. 取得 **nsCOMPtr\<T\>** 中 T 的 IID。
4. 將 IID 作為參數來呼叫 **nsQueryInterface** 的 functor。
5. **nsQueryInterface** 的 functor 裡在實際去呼叫原本物件的 `QueryInterface` 函式來進行 XPIDL interface 的請求。

至於 `do_GetInterface` 的做法也是類似的。比較不一樣的地方是 `do_GetInterface` 回傳的是一個 **nsGetInterface** 的物件。但是我們無法找到適用於 **nsGetInterface** 的 **nsCOMPtr\<T\>** 建構子。但可以發現 **nsGetInterface** 繼承自 **nsCOMPtr_helper**，並且 **nsCOMPtr\<T\>** 有相對應的建構子：

```cpp
MOZ_IMPLICIT nsCOMPtr(const nsCOMPtr_helper& aHelper)
  : NSCAP_CTOR_BASE(0)
{
  NSCAP_LOG_ASSIGNMENT(this, 0);
  assign_from_helper(aHelper, NS_GET_TEMPLATE_IID(T));
  NSCAP_ASSERT_NO_QUERY_NEEDED();
}
```

接下來就類似 `do_QueryInterface` 的流程進行下去，最後一樣會把 **nsGetInterface** 物件作為函式呼叫，來實際呼叫 `GetInterface`。

## `do_QueryObject`

除了前面提到的 `do_QueryInterface` 與 `do_GetInterface` 之外，Gecko 的程式碼中也很常看到使用 `do_QueryObject` 來進行轉型。先來看到 `do_QeuryObject` 的定義：

```cpp
template<class T>
inline nsQueryObject<T>
do_QueryObject(T* aRawPtr)
{
  return nsQueryObject<T>(aRawPtr);
}
```

與其他兩者相似，都是產生一個物件回傳。該物件為 **nsQueryObject**，其定義如下：

```cpp
template<class T>
class MOZ_STACK_CLASS nsQueryObject : public nsCOMPtr_helper
{
public:
  explicit nsQueryObject(T* aRawPtr)
    : mRawPtr(aRawPtr)
  {
  }

  virtual nsresult NS_FASTCALL operator()(const nsIID& aIID,
                                          void** aResult) const MOZ_OVERRIDE
  {
    nsresult status = mRawPtr ? mRawPtr->QueryInterface(aIID, aResult)
                              : NS_ERROR_NULL_POINTER;
    return status;
  }
private:
  T* MOZ_NON_OWNING_REF mRawPtr;
};
```

單從這個物件的 functor 來看，似乎與 nsQueryInterface 的 functor 無異。因此，`do_QueryObject` 的實際行為與 `do_QueryInterface` 完全相同。不一樣的地方在於 `do_QueryObject` 是一個函式樣板 (function template)，其參數不像 `do_QueryInterface` 一樣都是 **nsISupports\*** 而是根據傳入的參數型別所生成的 **T\***。因此使用 `do_QueryObject` 的話，物件並不會被轉型為 **nsISupports\***，而是保持原本的型別來呼叫 `QueryInterface`。

而之所以會有需要這個函式原因在於**大部份的實體類別 (concrete class) 存在 [diamond inheritance pattern](http://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) 的現象**，因此使得該物件在進行轉型到 **nsISupports** 出現的混淆 (ambiguous) 的情況。

以 **nsDocShell** 這個類別來說，其繼承的 XPIDL interface 如下：

```cpp

class nsDocShell MOZ_FINAL
  : public nsDocLoader
  , public nsIDocShell
  , public nsIWebNavigation
  , public nsIBaseWindow
  , public nsIScrollable
  , public nsITextScroll
  , public nsIDocCharset
  , public nsIContentViewerContainer
  , public nsIRefreshURI
  , public nsIWebProgressListener
  , public nsIWebPageDescriptor
  , public nsIAuthPromptProvider
  , public nsILoadContext
  , public nsIWebShellServices
  , public nsILinkHandler
  , public nsIClipboardCommands
  , public nsIDOMStorageManager
  , public nsINetworkInterceptController
  , public mozilla::SupportsWeakPtr<nsDocShell>
{
  // something
};
```

**nsDocShell** 繼承的所有 XPIDL interface 本身也都繼承自 **nsISupports**，diamond inheritance pattern 的現象就這麼發生了。在 Gecko 中，有非常多的實體類別與 **nsDocShell** 一樣都同時繼承了兩個以上的 XPIDL interface，所以 diamond inheritance pattern 的現象其實在 Gecko 裡面很常見。因此，當我們要對 **nsDocShell\*** 請求他所支援的 XPIDL interface 時，就需要使用 `do_QueryObject` 來進行，才不會產生編譯錯誤。

不過，使用 `do_QueryObject` 也是需要付出代價的。前面有提到，`do_QueryObject` 本身是一個函式樣板，樣板的運作原理是透過編譯器來產生一份針對該型別的程式碼。在 Gecko 中有非常多實體類別，如果這些類別在進行請求 interface 時都透過 `do_QueryObject` 的話，除了編譯時間會增加外，編譯出來的程式大小也將變得更加肥大。畢竟這將使得每個類別都有一份屬於自己的 `do_QueryObject` 函式以及一份屬於自己的 `nsQueryObject` 類別。

此外，我們也可以透過 `static_cast` 來解決同樣的問題。在開發時，通常我們已知該實體類別所支援的 XPIDL interface。因此先透過 `static_cast` 轉型為其中一個 XPIDL interface 就不會有 diamond inheritance pattern 的現象發生了。例如以下的程式碼：

```cpp
nsCOMPtr<nsIDocShellTreeItem> docShellItem(do_QueryInterface(static_cast<nsIDocShell*>(mDocShell)));
```

> P.S. XPIDL interface 不允許多重繼承，因此不會有 diamond inheritance pattern 的現象。

然而，`static_cast` 不會檢查執行期的物件狀態，單純檢查類別之間是否存在繼承關係。因此，在執行期物件可能會無法轉型成功而造成程式執行錯誤。理論上，不建議使用 `static_cast` 這樣危險的轉型方法。

## 總結

以上我們介紹了 `QueryInterface` 與 `GetInterface` 的相異之處。主要的差別在於轉型後的結果是否可以透過同樣的方式轉型回去。

另外，我們介紹了這兩個轉型函式的輔助函式 `do_QueryInterface`、`do_GetInterface` 與 `do_QueryObject`。`do_QueryInterface` 與 `do_GetInterface` 差別在於 `QueryInterface` 與 `GetInterface` 的功能上差異。而 `do_QueryInterface` 與 `do_QueryObject` 其功能上完全相同，差別在於使用的對象。`do_QueryInterface` 用來對一個 **XPIDL interface 的指標**請求某個 XPIDL interface，`do_QueryObject` 則用來對一個**實體類別的指標**請求某個 XPIDL interface。
