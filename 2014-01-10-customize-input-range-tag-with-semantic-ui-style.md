---
layout: post
title:  "打造 Semantic UI 風格的 <input type=\"range\" />"
date:   2014-01-10
tags:   ["web dev | 網頁開發", "Semantic UI", "CSS"]
feature:
    photo: false
---

最近在開發訊號繪製的網頁，詳情可參考這篇－[使用網頁作為訊號繪製界面 (with Python)](http://blog.kuoe0.tw/posts/2013/12/30/use-web-page-to-plot-signals-with-python)，其中套用的 [Semantic UI](semantic-ui.com) 來裝飾頁面。最近想加上一個 low-pass filter 的功能，並且希望可以調整 filter 的強度，打算使用 `<intput type="range />` 來實現調整的功能。找遍了 Semantic UI 的文件，完全找不到有定義 range 型別的樣式（還是我眼殘沒找到？！），就只好自己研究一下該怎麼套出類似的樣式了。

可以發現 W3C 並沒有規範 range 型別中控制點的 CSS selector，因此該功能只有在特定的瀏覽器中能夠作用。由於我主要使用的瀏覽器是 [Google Chrome](http://www.google.com/intl/zh-TW/chrome/)，其核心引擎為 [Blink](http://www.chromium.org/blink)，Blink 本身是 [WebKit](http://www.webkit.org/) 的分支，所以本篇的效果都能在任何基於 WebKit 與 Blink 的瀏覽器上。

## 外框樣式

首先先介紹 CSS selector，要選擇 range 型別的 input 有正規的 selector 是 `input[type="range"]`。但僅能定義整個拉條本身的外觀，並不能定義控制點的樣式。因此在 WebKit 中有規範一些 pseudo element，其中包括一個 `input[type="range"]::-webkit-slider-thumb` 可以用來調整控制點樣式。

由於我們要客制自己的樣式，因此需要先把瀏覽器預設的樣式給去除。只要加上 `-webkit-appearance: none` 的敘述就可以，程式碼如下：

```css
input[type="range"] {
	-webkit-appearance: none;
}
```

效果如下圖：

![step1](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-1.png)

這樣一來，預設的那條拉條就會消失了，只會剩下一個控制點。為了要搭配 Semantic UI 中的 [toggle checkbox](http://semantic-ui.com/modules/checkbox.html)，下一步就是加上一個左右兩邊都是圓弧的淺灰色外框。程式碼如下：

```css
input[type="range"] {
	-webkit-appearance: none;
	border-width: 1px;
	border-style: solid;
	border-radius: 50rem;
	border-color: rgba(0, 0, 0, 0.1);
}
```

效果如下圖：

![step2](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-2.png)

現在看起來很有個樣子了吧！

## 控制點樣式

最後一步就是調整控制點的樣式了。一樣需要把瀏覽器預設的樣式給去除，所以要加入 `-webkit-appearance: none` 的敘述。

```css
input[type="range"]::-webkit-slider-thumb {
	-webkit-appearance: none;
}
```

效果如下圖：

![step3](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-3.png)

然後就會發現整個拉條變扁了，這是因為控制點消失了，因此需要給控制點一個大小。利用試誤法大概猜測出控制點的長寬是 0.7 rem，把這個大小的敘述加到 CSS 裡面，程式碼如下：

```css
input[type="range"]::-webkit-slider-thumb {
	-webkit-appearance: none;
	height: 0.7em;
	width: 0.7em;
}
```

效果如下圖：

![step4](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-4.png)

就可以看到有高度的拉條了，不過控制點還是處於隱形的狀態，所以需要在替控制點加入背景顏色。查了一下 Semantic UI 中 checkbox 的圓點顏色色碼是 <span style="background-color: #58cb73; color: #58cb73">__</span> #58cb73。此外也在樣式中加入圓角的敘述。

```css
input[type="range"]::-webkit-slider-thumb {
	-webkit-appearance: none;
	background: #89B84C;
	border-radius: 50rem;
	height: 0.7em;
	width: 0.7em;
}
```

效果如下圖：

![step5](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-5.png)

這時候樣子大概就完成了！不過怎麼好像還是有些不一樣的地方，就是控制點與外框的距離。只要在替 `input[type="range"]` 加上個 `padding` 的樣式就大功告成了！程式碼如下：

```css
input[type="range"] {
	-webkit-appearance: none;
	border-width: 1px;
	border-style: solid;
	border-radius: 50rem;
	border-color: rgba(0, 0, 0, 0.1);
	padding: 3px 5px;
}
```

效果如下圖：

![step6](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-6.png)

## More Semantic

上面的 CSS 雖然可以打造出外觀與 Semantic UI 風格一致的 range input，但為了與 Semantic UI 一起使用，應該將 selector 與 Semantic UI 更一致一些才對。看了一下 checkbox input 的 selector 是 `.ui.slider.checkbox`，所以就直接在原本的 selector 前面加上 `.ui.slider.range` 即可。程式碼如下：

```css
.ui.slider.range input[type="range"] {
	-webkit-appearance: none;
	border-width: 1px;
	border-style: solid;
	border-radius: 50rem;
	border-color: rgba(0, 0, 0, 0.1);
	padding: 3px 5px;
}

.ui.slider.range input[type="range"]::-webkit-slider-thumb {
	-webkit-appearance: none;
	background-color: #89B84C;
	border-radius: 50rem;
	height: 0.7em;
	width: 0.7em;
}
```

## Work on Firefox

在 Firefox 上，所有 `-webkit` 前綴的敘述都將會失效。如果按照原本的程式碼執行，會看到以下的效果：

![work on firefox](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-7.png)

可以發現外框的效果還是有的，但是莫名中間多出了一條黑線。這是因為 Firefox 有一個 pseudo element 叫做 `-moz-range-track`，用來定義 range input 的拉條樣式。為了讓該拉條消失，需要將其背影與外框給去除。程式碼如下：

```css
.ui.slider.range input[type="range"]::-moz-range-track {
	background: none;
	border: none;
}
```

效果如下：

![step1 on firefox](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-8.png)

由於 `-webkit-slider-thumb` 不會作用，所以控制點將不會有任何更動，持續保持 Firefox 預設的樣式。所以我們需要利用 Firefox 的另一個 pseudo element 叫做 `-moz-range-thumb`，被用來定義控制點樣式。如果依照在 `-webkit-slider-thumb` 相同的敘述的話：

```css
.ui.slider.range input[type="range"]::-moz-range-thumb {
	background: #89B84C;
	border-radius: 50rem;
	height: 0.7em;
	width: 0.7em;
}
```

會發現控制點上仍有一個灰色外框：

![step2 on firefox](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-9.png)

所以做後一步就是把控制點的外框去除或變成跟背景一樣的顏色，加入 `border-color: #89B84C;` 即可，程式碼如下：

```css
.ui.slider.range input[type="range"]::-moz-range-thumb {
	background: #89B84C;
	border-color: #89B84C;
	border-radius: 50rem;
	height: 0.7em;
	width: 0.7em;
}
```

![step3 on firefox](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2014-01-10-customize-input-range-tag-with-semantic-ui-style-10.png)

這樣就大功告成了！但是要注意，千萬不可以因為懶惰而這樣寫：

```css
.ui.slider.range input[type="range"]::-webkit-slider-thumb, 
.ui.slider.range input[type="range"]::-moz-range-thumb {
	/* something */
}
```

這樣的寫法，在兩個瀏覽器上都會被視為 invalid selector，因此會完全沒有效果！

## Source Code

<script src="https://gist.github.com/KuoE0/8346849.js"></script>

Source code on [gist](https://gist.github.com/KuoE0/8346849).

<p data-height="160" data-theme-id="0" data-slug-hash="pInFu" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/KuoE0/pen/pInFu'>pInFu</a> by KuoE0 (<a href='http://codepen.io/KuoE0'>@KuoE0</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//codepen.io/assets/embed/ei.js"></script>
