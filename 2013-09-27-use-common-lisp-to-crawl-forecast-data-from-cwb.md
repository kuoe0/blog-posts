---
layout:  post
title:   "使用 Common Lisp 抓取中央氣象局預報資料"
date:    2013-09-27
tags:    ["Common Lisp", "Lisp", "forecast | 氣象預報"]
feature:
    photo:       false
    creator:     
    url:         
    license:     
    license_url: 
---

這次程式語言作業是用 Common Lisp 編寫撈取[中央氣象局](http://www.cwb.gov.tw)下個時段的各縣市天氣預報，並重新排版輸出！輸出內容如下：

```
彰化縣  25~27°C   50%  多雲短暫陣雨或雷雨
嘉義縣  25~27°C   50%  多雲短暫陣雨或雷雨
嘉義市  25~26°C   50%  多雲短暫陣雨或雷雨
新竹縣  25~26°C   50%  多雲短暫陣雨或雷雨
新竹市  25~26°C   50%  多雲短暫陣雨或雷雨
花蓮縣  25~27°C   50%  多雲短暫陣雨
高雄市  27~29°C   50%  多雲短暫陣雨或雷雨
基隆市  24~25°C   30%  多雲短暫陣雨或雷雨
金門縣  24~24°C   50%  多雲時陰短暫陣雨或雷雨
連江縣  20~21°C   80%  多雲時陰短暫陣雨或雷雨
苗栗縣  25~26°C   50%  多雲短暫陣雨或雷雨
南投縣  26~27°C   50%  多雲短暫陣雨或雷雨
澎湖縣  24~25°C   50%  多雲短暫陣雨或雷雨
屏東縣  27~29°C   50%  多雲短暫陣雨或雷雨
臺中市  25~26°C   50%  多雲短暫陣雨或雷雨
臺南市  27~28°C   50%  多雲短暫陣雨或雷雨
新北市  25~26°C   30%  多雲短暫陣雨或雷雨
臺北市  25~26°C   30%  多雲短暫陣雨或雷雨
臺東縣  25~27°C   50%  多雲短暫陣雨或雷雨
桃園縣  25~26°C   30%  多雲短暫陣雨或雷雨
宜蘭縣  23~24°C   30%  多雲短暫陣雨或雷雨
雲林縣  26~27°C   50%  多雲短暫陣雨或雷雨
```

縣市按照各縣市的英文名稱，並按照字典序排列，而縣市的英文名稱採用還只有北高兩個直轄市時的名稱，也就是還沒有五都的時候。所以可以發現，新北市並不是用 New Taipei City！如此規定的原因是，其實網頁中都可以找到各縣市舊有的英文名稱，可以減少程式碼中被寫死的部分。

不過雖然說是要用 Lisp 來撈取氣象局資料，但其實應該說是用 Lisp 處理 xml 的 parser 更為貼切。為了減少學生們的負擔，我已經將發出 http request 與 html 轉 xml list 的套件都找好了，也做好的範例程式給學生們，讓他們在修改我給的範例來完成作業。這樣的好處是，學生們可以學著去瞭解一份現有的程式碼，也可以讓完全不知道怎麼開始的同學比較有點頭緒！

在這次作業中，我要求學生們要安裝 Quicklisp 並利用 Quicklisp 來載入 `drakma` 與 `com.informatimago.common-lisp.html-parser` 這兩個套件，分別是用來作 http request 與 html 轉 xml list 的套件。html 轉 xml list 的方式為：列表第一個元素為 html tag 的名稱，而第二個元素則為該 tag 中所有 attribute 的 list，列表中剩餘的元素則為該 tag 的內容！舉個例子：

**html code**

```html
<div style="color: blue;" class="myClass">
	<p>blog: <a href="http://kuoe0.tw">KuoE0.TW</a></p>
</div>
```

**xml list**

```lisp
(:DIV (:STYLE "color: blue;" :CLASS "myClass")
	(:P NIL "blog: " (:A (:HREF "http://kuoe0.tw") "KuoE0.TW")))
```

可以發現若是該 tag 中沒有任何 attribute 的話，列表中第二個元素將為 `NIL`，若是有任何 attribute 的話，將按照 html code 中的順序放置 attribute 名稱並其下一個元素則為該 attribute 的值！不需要給值的 attribute 則其後不接上其值，直接接上下一個 attribute 的名稱。從第三個元素後（含），就都是該 tag 的內容了！可以看上面例子中的 div 區塊，其第三個元素是字串 `"blog: "`，第四個元素則是 tag a 的列表。

所以我的程式中特別宣告了以下三個 function：

```lisp
(defun html-tag (xml) 
	(first xml))
(defun html-attributes (xml) 
	(second xml))
(defun html-content (xml)
	(cddr xml))
(defun html-attribute (xml key)
	(cadr (member key (second xml))))
```

分別拿來取出一個 xml list 中的 tag 名稱、tag 的 attribute 列表、tag 的內容以及該 tag 各 attribute 所對應的值。不過，其實我是看 `com.informatimago.common-lisp.html-parser` 的 source code 中學來的，只是 `com.informatimago.common-lisp.html-parser` 這個名字真的太長了，所以我決定自己寫一個好了 XD。

氣象局的預報網頁是 [http://www.cwb.gov.tw/V7/forecast/](http://www.cwb.gov.tw/V7/forecast/) 這個鏈結，不過我發現顯示預報的那一區塊是 load 其他頁面的，所以網頁的 source code 中並沒有包含預報的內容。於是查了一下發現，預報的內容是來自這個網頁 [http://www.cwb.gov.tw/V7/forecast/f_index.htm](http://www.cwb.gov.tw/V7/forecast/f_index.htm)。中央氣象局通常都提供接下來三個時段的預報，以下為該時段所對應的網頁：

- 下個時段：[http://www.cwb.gov.tw/V7/forecast/f_index.htm](http://www.cwb.gov.tw/V7/forecast/f_index.htm)
- 下下個時段：[http://www.cwb.gov.tw/V7/forecast/f_index2.htm](http://www.cwb.gov.tw/V7/forecast/f_index2.htm)
- 下下下個時段：[http://www.cwb.gov.tw/V7/forecast/f_index3.htm](http://www.cwb.gov.tw/V7/forecast/f_index3.htm)

不過由於這幾個網址所抓取到的資料並不是完整的網頁，所以會發現他並沒有 `html` tag，所以其產生的 xml list 是一個包含多個 xml list 的列表。如下：

```lisp
((:DIV NIL test1)
(:DIV NIL test2)
(:DIV NIL test3))
```

分別有三個 div 物件，但對於最外層的 list 來說，他並不是一個 xml list，他只是個包含很多 xml list 的列表。因次，若是要尋訪所有預報資料的話，也必須要尋訪該 list 中的所有元素。如果是一個合法的完整網頁的話，照理說轉成 xml list 後，應該是收到一個 xml list 的列表，並且該列表中僅會有一個元素，就是 html 的 xml list。

做資料處理之前，首先要做的是－**觀察資料**，先來看看我們從氣象局那邊拿回來的資料長得什麼樣子。

根據上面的網址所截取到的 html 程式碼並不是一個合法的 html 程式碼，他的結構大概如下：

```html
<script>
	document.title='天氣預報';
</script>

<div class="clear"></div>

<div class="BoxContentTab clearfix">
	<a class="current" href="index.htm">今晚明晨</a>
	<span>│</span>
	<a href="index2.htm">明日白天</a>
	<span>│</span>
	<a href="index3.htm">明日晚上</a>
	<div class="clear"></div>
</div>

<div>
	<div class="modifyedDate modifyedDatetwo">
		06/16 00:00 - 06/16 06:00
	</div>

	<div class="modifyedDatethree">
		發布時間 : 2013/06/15 23:00 溫度單位:℃
	</div>

	<div class="clear"></div>
</div>

<div style="width:100%; overflow:hidden;">
	氣象預報資訊
</div>

地圖操作
```

預報資訊與地圖資訊上面先省略，基本上這次的作業看了看我們拿到的資料，我們只需要氣象預報資訊的部分，地圖資訊時間等等都其實不太需要！

所以我們再來看看氣象預報資訊的 HTML 內容長怎樣，以下不是完整內容：

```html
<div class="big01">
	<div class="NorthArea">
		<table class="N_AreaList" style="background-color: #FFFFFF" width="180">
			<tr>
				<th colspan="4" scope="col">北部</th>
			</tr>

			<tr id="KeelungList" onblur="funcTable('KeelungList','#ffffff')" onfocus="funcTable('KeelungList','#d3e5f7')" onmouseout="funcTable('KeelungList','#ffffff')" onmouseover="funcTable('KeelungList','#d3e5f7')">
				<td width="60%">
					<a href="/V7/forecast/taiwan/Keelung_City.htm">基隆市</a>
				</td>
				
				<td width="50%">
					<a href="/V7/forecast/taiwan/Keelung_City.htm">25~26°</a>
				</td>

				<td width="18%">
					<a href="/V7/forecast/taiwan/Keelung_City.htm">10%</a>
				</td>

				<td width="21%">
					<a href="/V7/forecast/taiwan/Keelung_City.htm">
						<div class="spic">
							<img alt="多雲" border="0" height="45" src="/V7/symbol/weather/gif/night/02.gif" title="多雲" width="45"/>
						</div>
					</a>
				</td>
			</tr>
		</table>
	</div>
</div>
```

我們可以發現 HTML 程式碼中，有著一些規律，所有縣市的預報資訊都被放在表格中，每個縣市放一列 (tr tag)，而各個單項資料則是在表格的每一行 (td tag)。屬於相同區域的縣市會被放入同一個表格，例如：基隆市、臺北市、新北市、桃園縣、新竹市、新竹縣以及苗栗縣都被放置在北部的區塊中。

另外我想是因為 UI 的設計，所以北部與中部又被歸到一個 `class="big01"` 的 `div` 區塊中。而剩餘的東部、南部與外島則都被歸在 `class="big03"` 的 `div` 區塊中。至於 `class="big02"` 的 `div` 區塊忽略就好，只是存放那張台灣地圖。

知道各縣市預報資料位置後，於是我們再來就是觀察單一縣市的資料。可以發現縣市的預報資料在表格中都是各自為一列，也就是用 `tr` 標籤所包住的內容。所以其實我們就只要查找所有的 `tr` 標籤的內容就好，而 `tr` 標籤中有個 `id` 的屬性，其值剛好就是該縣市名稱的英文名字，但後面會加上 List 這幾個字母。這也是我為什麼採用各縣市的就英文名稱來坐排序的原因，因為這個資訊可以直接從網頁的程式碼取得，以減少 hard code 的部分。

接著，每個縣市的預報資料就各自在一行中，這些資料包括了縣市中文名稱、氣溫、降雨機率以及氣候描述。每個資料都被 `td` 的標籤給包住，所以取得 `tr` 程式碼後，就可以進一步找出裡頭的 `td` 標籤所包含的資料了！而且其順序也是固定的，第一欄就是縣市中文名稱，第二欄是溫度，第三欄是降雨機率，第四欄是氣候描述。基本上，所有作業要求的資訊都在程式碼裡面了，縣市名稱也不必自己打了！

這次作業中，我也不要同學們自己去解析 (parse) HTML 文件，直接使用現成的第三方套件，試了很多套最後選定了 `com.informatimago.common-lisp.html-parser` 這個。

把我們拿到的 HTML 資料丟給後，會得到上述的「氣象預報資訊」與「地圖操作」的結果，我也不知道為什麼其他部分消失了…大概是本來抓到的 HTML 程式碼不是合法的吧。不過沒關係，我們需要的資料還在就好。前面已經敘述過經由該解析器所解析後的結果了，所以這邊就不再提了！

根據我們對資料的瞭解，我們知道其實我們只要找出所有的 `tr` 標籤就好了，所以其實只要遞回的進去 list 中判斷第一個元素是不是 `TR` 就可以了，如果是就針對該 list 進行進一步的解析，如果不是就忽略，如果是一個 list，就再進去下一層搜尋。

對於 `TR` 的 list 作進一步解析時，就一樣是搜尋其 list 中的 `TD` 標籤。要注意的是，`TR` 的 list 中，可能會有許多空白字串存在，目前也不確定是什麼原因導致會有如此多的空白字串，所以我取得 `TR` list 後，會先將裡頭所有的空白字串去除。如此以來，就可以很方便的取得預報的各項資料，因為第一欄就會是縣市名稱，第二欄就是溫度，第三欄就是降雨機率，第四欄就是氣候描述了！

以下是這次作業的程式碼參考：

<script src="https://gist.github.com/KuoE0/6725804.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/6725804).

**Line 2 ~ 4**

由於使用 quicklisp 進行 load 套件時會輸出一些 loading 中的訊息，因此將 loading 的訊息都導向 `/dev/null`！在 Windows 中，可以將 `/dev/null` 改成 `nul`。

**Line 6 ~ 17**

前面已解釋過，用來方便獲取 tag 內容。

**Line 19 ~ 21**

進行格式化輸出，這邊輸入的參數為一個 list 的 list (list of list)，資料大概長這樣：

```lisp
((彰化縣  25~27°C   50%  多雲短暫陣雨或雷雨)
(嘉義縣  25~27°C   50%  多雲短暫陣雨或雷雨)
(嘉義市  25~26°C   50%  多雲短暫陣雨或雷雨)
(新竹縣  25~26°C   50%  多雲短暫陣雨或雷雨)
...)
```

這邊利用 `format` 函式來進行格式化的輸出，第一個參數傳入 `t` 表示輸出到當前的使用的輸出端，如果沒特別設定就是「標準輸出」。其中 `~A` 就如同 C 語言的 `printf` 中的 `%d` 之類的功能，會取代為後面給予的參數。不過 `~A` 沒有限定資料型別，任何東西都可以！如果使用 `~@A` 則表示置左靠其，預設為置右靠齊。`~%` 表示換行符號，另外，`~&` 為如果上一個字元不是換行的話，輸出換行符號。

由於給定的資料是 list of list，會了上面的格式輸出方法後，基本上已經可以進行作業要求的格式輸出了，程式碼如下：

```lisp
(dolist (f forecasts)
	(format t "~A  ~7@A  ~4@A  ~A~%" (car f) (car (cdr f)) (car (cdr (cdr f))) (car (cdr (cdr (cdr f))))))
```

不過這樣看起來實在太冗了，`format` 就額外提供了對於 list of list 的輸出方式 `~:{…~}`。只要把中間的 … 部分換成想要的格式，而後面也不再需要一個一個取出元素，只要把 list of list 傳遞給 `format` 就會自動處理了！如下：

```lisp
(format t "~:{~&~A  ~7@A  ~4@A  ~A~}~%" forecasts))
```

更多關於 `format` 的用法請見：

- [FORMATTED OUTPUT (COMMON-LISP-STYLE)](http://www.gnu.org/software/kawa/Format.html)
- [A Few FORMAT Recipes](http://www.gigamonkeys.com/book/a-few-format-recipes.html)


**Line 26 ~ 41**

對資訊作出適當的處理。

去除 `List` 字尾，由於從原始碼得到的英文名稱，其後方都會有額外的 `List` 字尾，故需要去除！轉換百分比符號碼為百分比符號，以及溫度需要加上 C 單位。


**Line 51 ~ 67**

取得縣市名稱、溫度、降雨機率以及氣象慨況資訊，並進行適當的處理，例如：溫度要加上 C 單位、降雨機率需要 html 的百分比符號碼轉換為百分比符號。

**Line 69 ~ 77**

以下是一塊含有預報資料的 `tr` 標籤其內容：

```html
<tr id="KeelungList" onblur="funcTable('KeelungList','#ffffff')" onfocus="funcTable('KeelungList','#d3e5f7')" onmouseout="funcTable('KeelungList','#ffffff')" onmouseover="funcTable('KeelungList','#d3e5f7')">
	<td width="60%">
		<a href="/V7/forecast/taiwan/Keelung_City.htm">基隆市</a>
	</td>
	
	<td width="50%">
		<a href="/V7/forecast/taiwan/Keelung_City.htm">25~26°</a>
	</td>

	<td width="18%">
		<a href="/V7/forecast/taiwan/Keelung_City.htm">10%</a>
	</td>

	<td width="21%">
		<a href="/V7/forecast/taiwan/Keelung_City.htm">
			<div class="spic">
				<img alt="多雲" border="0" height="45" src="/V7/symbol/weather/gif/night/02.gif" title="多雲" width="45"/>
			</div>
		</a>
	</td>
</tr>
```

我們需要的資料有縣市名稱、溫度、降雨機率以及氣象概況，在上面都可以找到！另外，由於要排序，所以需要還要取得縣市的英文名稱。

在上面的資料中，可以發現原始碼上有一些空行，這些空行在 list 中也佔有一個位置，所以我先把資料利用 `filter-out-td` 這個函式進行了過濾。接下來就可以很單純的使用 `get-data` 來取得特定的預報資訊了！

**Line 80 ~ 94**

氣象局的資料本身有按照區域分區塊，每個大區塊內才會有該區域內的縣市預報資料。所以在這個函式內，需要在進行尋訪來找出每個縣市的資料。

由於預報資料都放在表格內，所以我們尋找出每個 `tr` 標籤，而且如果是縣市預報的欄位，其 `tr` 標籤中還會有 `id` 這個屬性，其 `id` 內容即為該縣是英文名稱，並附加上 `List` 這字串，這也是為什麼會需要有去除 `List` 字串的函式。所以如果當前的資料為 `tr` 標籤，並帶有 `id` 屬性的話，截取出這塊 `tr` 區塊後，再將內容傳入 `city-forecast` 該函式來取得各項預報內容，再將資料附加到回傳的變數上。如果不是 `tr` 標籤或是沒有 `id` 屬性，則進入這個資料，並重複上述的動作。

簡單說，這個函式就是遞回的搜尋預報資料，而預報資料的特性就是存放在 `tr` 標籤中，且其帶有 `id` 屬性。

**Line 95**

宣告全域變數 website 存放資料抓取的網址。

**Line 97 ~ 104**

利用 drakma 撈取氣象局網頁原始碼，並透過 UTF-8 進行解碼！接下來再透過 html parser 解析網頁原始碼，並存為 list 形式的 xml 資料，如同前面提過的一樣。

接著依序尋訪這個 list 中的資料，透過 `get-forecast` 這個函式來截取出實際需要的資料，再將預報資料附加在結果上。

**Line 107 ~ 109**

最後，由於我有規定輸出的順序，因此在以縣市英文名稱作為排序依據進行排序。而輸出時不需要縣市的英文名稱，所以再將縣市的英文名稱去除，再進行輸出！

這次會選擇出這個作業的原因主要是：

- 撰寫小工具的能力
- 具備觀察資料並進行自動化處理的能力

如果一個工程師連個自己想用的工具都寫不出來，又如何能寫出別人想用的工具呢？出這個作業主要是想到有時候要查氣象還要打開瀏覽器，並進入中央氣象局網站，再點擊預報頁面實在很麻煩。如果我有個小工具可以直接以指令執行就跟我說預報內容該有多棒啊！

另外，有時候我們會遇到一些重複性高的文件需要處理。不會寫程式的人，或許就自己 copy/paste 來完成工作。但是身為一個會寫一點程式的人，一直做這些重複性的工作不覺得很苦悶嗎？如果可以有能力寫出一個小工具來進行資料的自動化處理豈不是既省時又方便，還可以磨練技術！

我主要是希望學生們能有自己編寫簡單的解析器 (parser) 的能力，雖然這次作業其實只是處理一個已經解析過的 list 而已，但我想多少還是會有些學生對於處理重複性資料比較有些概念了吧！

本來是打算連 HTML → XML list 都讓學生自己利用 regular expression 實作。因為我覺得使用 regex 的能力對於一個資訊學生來說非常重要，要寫出好的 parser 會 regex 才是最重要的，因為你對於資料的觀察更需要透徹。不過，像 HTML 這種常見的格式，用現成的其實就 OK 了！

題外話：

這個小工具其實還是有點弱，用英文名稱排序個人覺得不是個好方法，不過主要是為了改作業方便啦！之後打算就根據氣象局的分類來分成北部、中部、東部、南部與外島，並且希望可以傳入參數來取得特定縣市或區域的預報資料。但是，使用 CLI 就是為了方便（不過是對於習慣 CLI 的使用者），如果要取得特定縣市時還要輸入中文實在不是好方法（個人認為在 CLI 下輸入中文就是個麻煩），可能改用英文縮寫？！而且覺得 drakma 的 http request 好像有點慢…

