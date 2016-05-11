---
layout: post
title:  "使用網頁作為訊號繪製界面 (with Python)"
date:   2013-12-30
tags:   ["Python", "signal plotting | 訊號繪製", "Tornado", "jQuery", "Flot", "web dev | 網頁開發", "MPU6050"]
feature:
    photo: false
---

之前使用過 Python 的 matplotlib 套件來繪製，可惜僅能事先錄製好訊號，再將存放訊號的檔案透過 matplotlib 來繪製訊號圖。這樣的方式除了愚蠢外，也不便於硬體開發，因此打算寫一個 GUI 程式並可以即時 (real-time) 的繪製出訊號。

但一直以來覺得自己實在不會寫 GUI 程式，雖然大二寫過幾次 Java，不過實在不想寫 Java 所以放棄。也寫過幾次 Qt，但太久以前都還不知道是 4.x 版本的時期了，現在 5.x 都出來了！身為一個走在科技尖端的宅宅一定要用一下 5.x 版本啊！不過，最重要的原因是...現在常常很懶得寫 C/C++，除了很多雜事要自己做外，還要編譯。所以現在能用 Python 解決就會想用 Python 解決，於是我只好去看了 PyQt5...。但文件幾乎不齊全，網路上資料似乎也還不夠多，有點不知道怎麼下手就放棄了。（完全都是自己的問題）

最後決定用網頁來寫好了，用網頁寫的話調整樣式用 CSS 滿方便的。因此決定採用 Python 作為後端來讀串列埠的訊號，並在前端發送請求時將訊號傳送出去。又因為很懶惰懶得再 OS X 上設定 WSGI，所以傾向找本身自帶 web server 的套件來用，也因此讓我發現個好東西－[Tornado](http://www.tornadoweb.org/)。Tornado 本身是一個 web server 也是個輕量級的 web framework，這完完全全就是我想要的東西！我要的就只是個簡單的 framework 讓我可以寫個簡單的 API 將訊號傳送到前端。

而至於前端訊號繪製的部分，上網找到兩個 jQuery 的套件，分別是 [Flot](http://www.flotcharts.org/) 與 [jqPlot](http://www.jqplot.com/)。因為看 Flot 比較順眼的關係，因此我決定採用 Flot 了。

## Environment

- OS X 10.9
- Python 2.7.6
- Tornado 3.1.1
- PySerial 2.7
- Flot 0.8.2

## Tornado Introduction

首先，我們先介紹怎麼讓 tornado 成功運行起來。來看看官方網站的範例：

```python
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, world")

application = tornado.web.Application([
    (r"/", MainHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

這時候只要用瀏覽器打開 [http://localhost:8888/](http://localhost:8888/)，就可以看到以下畫面：

![Tornado Hello World](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-12-30-use-web-page-to-plot-signals-with-python-1.png)

上述的程式碼分為三個部分來講解，第一部分是 `MainHandler` 的部分。主要就是定義一個 request handler 繼承至 `tornado.web.RequestHandler`。並重載一個 `get` 成員函式，當有一個 `GET` 請求發生時，該函式將被呼叫。很直覺的可以想到，如果發生 `POST` 請求呢？那麼就重載一個 `post` 成員函式即可。函式主體就編寫你想要呈現在網頁上的內容，就如同範例中只輸出一個 Hello, world 這樣。若是發生沒有重載的請求，則會回傳 405 錯誤碼。可以將這邊看作是 MVC 中的 M (Model) 部分。

第二部分是 `tornado.web.Application` 這行，`web.Application` 用於宣告一個實際運行的網頁應用實體。可以發現他的初始參數為一個 list，可以包含許多 (url, request handler) 這樣的 tuple。用於定義該應用接收到不同 URL 的請求，所需要作出的回應，因此就稱這部分叫做 URL dispatcher，如同 MVC 中的 C (Controller)。

第三部分則是 `__main__` 這部分，這邊就是開始運行前面設定好的網頁應用。首先是先設定該應用要使用的埠號 (port)。接著 `tornado.ioloop.IOLoop.instance().start()` 這行就會將 tornado 服務啟動，如此該網頁就開始服務了。

~~如果不想利用 `self.write` 的方式來寫 HTML tag 的話，應該可以考慮像 [Jinja 2](http://jinja.pocoo.org/) 之類的樣板引擎來使用，如此就可以擁有完整的 MVC 模型了！~~

既然有了 MVC 的 M 與 C 了，那麼 V (View) 怎麼可以缺少呢！可以引入 `tornado.template` 來增加模板的功能，詳細可以參考官方文件－[tornado.template — Flexible output generation](http://www.tornadoweb.org/en/stable/template.html)。

## Back-end

大概了解 Tornado 的運作方式後，即可開始設計用於接收與傳遞訊號的應用與 API 了。首要需要的接收串列埠所傳送的資料，這邊需要 PySerial 這個套件，使用方式可以參考官方網站的 [short introduction](http://pyserial.sourceforge.net/shortintro.html)。依照範例宣告一個 PySerial 的全域實體，並設定好串列埠與 baud rate，如下行：

```python
ser = serial.Serial('/dev/tty.usbmodem1421', 38400, timeout=1)
```

接著寫一個接收與處理訊號的函式，請編寫運行單次的函式，不要在裡頭加上無窮回圈一直讀取訊號。另外需要一個全域變數來儲存接收到的訊號，如下：

```python
received_data = list()

def receive_signal():
    global received_data
    try:
        data = int(ser.readline())
        received_data.append(data)
    except:
        print 'Error happened...'
```

接下來就開始利用 Tornado 編寫 API 的處理函式，這邊僅針對 `get` 該成員函式來舉例。假設只要回傳 100 個接收到的訊號，並每次都拿掉最舊的訊號（為了達到訊號移動的效果）。由於要跟 Flot 搭配，Flot 需要的資料結構需要為一資料序列與時間序列的組合，並回傳 JSON 格式。

例如：現在有 5 個訊號，分別是 `[1,7,6,0,-1]`，Flot 還需要加上順序資訊，也就是需要的格式為 `[[0,1],[1,7],[2,6],[3,0],[4,-1]]`。

程式碼如下：

```python
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        global received_data
        self.set_header("Content-Type", "text/javascript")
        ret = received_data[:min(100, len(received_data))]
        ret = [list(p) for p in enumerate(ret)]
        self.write(json.dumps({signal: ret}))
        received_data = received_data[1:]
```

最後在程式運行處啟動定時接收訊號與 API 服務，以下程式碼用於週期性的執行訊號接收程式，使得每 50 ms 就會向串列埠接收訊號：

```python
serial_loop = tornado.ioloop.PeriodicCallback(receive_signal, 50)
serial_loop.start()
```

另外要啟動 API 服務就與前面的範例一樣，指定該服務運行的埠號後，並開啟 tornado 服務即可：

```python
application.listen(8888)
tornado.ioloop.IOLoop.instance().start()
```

如此一來我們就完成訊號接收與傳送的 API 了！用瀏覽器開啟 [http://localhost:8888/](http://localhost:8888/) 即可看到以下畫面：

![Tornado Signal Receive API](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-12-30-use-web-page-to-plot-signals-with-python-2.png)

或使用 `curl http://localhost:8888/` 指令，也可以獲得以下資料：

	{"signal": [30800, 27121, 31762, 31465, 32980, 26276, 33979, 30717, 29950, 30823, 28842, 32171, 32062, 29474, 32584, 34640, 32288, 25214, 34088, 27646, 29884, 32428, 29364, 30820, 31717, 29105, 29504, 34367, 25984, 25961, 29318, 29565, 32606, 25335, 34087, 32341, 30709, 31094, 27391, 27324, 26073, 31339, 27464, 32860, 32598, 26386, 25887, 31650, 28522, 27580, 25492, 34003, 25227, 33420, 32395, 26498, 30965, 27332, 30735, 34950, 27014, 31826, 30605, 30111, 26003, 25731, 30463, 26248, 33822, 31699, 28777, 33551, 27322, 31398, 30563, 29374, 27167, 28128, 28870, 25841, 29510, 31338, 31242, 27257, 33175, 27025, 26856, 25098, 31253, 34748, 34644, 34419, 29612, 25621, 31972, 27163, 31274, 25628, 25200, 28675]}

## Front-end

再來是前端的部分，由於需要 real-time 的繪製訊號，因此參考了官方網站的 [real-time 範例](http://www.flotcharts.org/flot/examples/realtime/index.html)。主要需要修改的是資料截取的部分，在官方範例上資料是隨機產生的，需要改成 AJAX 的方式來向我們前面設計的 API 請求資料。因此編些一個定時更新資料的函式，如下：

```javascript
function update() {
	var url = 'http://localhost:8888/';
	$.getJSON(url, function(data) {
		plot.setData([data.signal]);
		plot.draw();
	});
	setTimeout(update, updateInterval);
};
```

不過，會發現這樣根本請求不到任何資料。原因是同源政策 (same-origin policy) 的規範，使得 JavaScript 不被允許進行跨域的請求。因此需要採用 JSONP 的協定來進行資料請求。首先，需要修改請求的位置，本來是 `http://localhost:8888/`，將其修改為 `http://localhost:8888/?callback=?`，jQuery 會自動隨機生成一個值用於 callback 參數，所以 JavaScript 的程式碼會被修改成以下這樣：

```javascript
function update() {
	url = 'http://localhost:8888/?callback=?';
	$.getJSON(url, function (data) {
		plot.setData([data.signal]);
		plot.draw();
	});

	setTimeout(update, updateInterval);
}
```

但這樣還不夠，要使用 JSONP 也需要後端的支援，所以 Python 程式碼也需要做些修正。只要將 callback 參數的值作為函式名稱來包裝 JSON 資料即可，如下：

```python
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        global received_data
        self.set_header("Content-Type", "text/javascript")
        callback = self.get_argument('callback')
        ret = received_data[:min(100, len(received_data))]
        ret = [list(p) for p in enumerate(ret)]
        self.write("{0}({1})".format(callback, json.dumps({'signal': ret})))
        received_data = received_data[1:]
```

完成後打開網頁就會發現有訊號產生了，只是會一直亂跳。因為我們去除了範例的亂數產生部分，而沒有給定初始訊號值。其實就只要一個包含 100 個 0 的陣列給 Flot 即可。其餘關於 Flot 的樣式可以參考 Flot 的官方文件－[Flot Reference](https://github.com/flot/flot/blob/master/API.md)。

用瀏覽器開啟編寫好的 Html 檔案即可看到以下畫面：

![Flot Signal Plotting](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-12-30-use-web-page-to-plot-signals-with-python-3.png)

## Source Code

**後端**

```python
import serial
import tornado.ioloop
import tornado.web
import json

ser = serial.Serial('/dev/ttys009', 38400, timeout=1)

received_data = list()

def receive_signal():
    global received_data
    try:
        data = ser.readline().split(',')[0]
        received_data.append(int(data))
    except:
        print 'Error happened...'

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        global received_data
        self.set_header("Content-Type", "text/javascript")
        callback = self.get_argument('callback')
        ret = received_data[:min(100, len(received_data))]
        ret = [list(p) for p in enumerate(ret)]
        self.write("{0}({1})".format(callback, json.dumps({'signal': ret})))
        received_data = received_data[1:]

application = tornado.web.Application([
    (r"/", MainHandler),
    ])

if __name__ == "__main__":

    serial_loop = tornado.ioloop.PeriodicCallback(receive_signal, 50)
    serial_loop.start()

    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

**前端（修改自 Flot 的 real-time 範例）**

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
	<title>Flot Examples: Real-time updates</title>
	<link href="../examples.css" rel="stylesheet" type="text/css">
	---[if lte IE 8]><script language="javascript" type="text/javascript" src="../../excanvas.min.js"></script><![endif]---
	<script language="javascript" type="text/javascript" src="../../jquery.js"></script>
	<script language="javascript" type="text/javascript" src="../../jquery.flot.js"></script>
	<script type="text/javascript">

	$(function() {

		// We use an inline data source in the example, usually data would
		// be fetched from a server

		var totalPoints = 100;
		var updateInterval = 50;
		var init = [];

		for (var i = 0; i < totalPoints; ++i) {
			init.push([i,0]);
		}

		var plot = $.plot("#placeholder", [init], {
			series: {
				shadowSize: 0	// Drawing is faster without shadows
			},
			yaxis: {
				min: -36000,
				max: 36000
			},
			xaxis: {
				show: false
			}
		});

		function update() {

			url = 'http://localhost:8888/?callback=?';
			$.getJSON(url, function (data) {
				console.log(data.signal);
				plot.setData([data.signal]);
				plot.draw();
			});

			setTimeout(update, updateInterval);
		}

		update();

		// Add the Flot version string to the footer

		$("#footer").prepend("Flot " + $.plot.version + " &ndash; ");
	});

	</script>
</head>
<body>

	<div id="header">
		<h2>Real-time updates</h2>
	</div>

	<div id="content">

		<div class="demo-container">
			<div id="placeholder" class="demo-placeholder"></div>
		</div>

		<p>You can update a chart periodically to get a real-time effect by using a timer to insert the new data in the plot and redraw it.</p>

		<p>Time between updates: <input id="updateInterval" type="text" value="" style="text-align: right; width:5em"> milliseconds</p>

	</div>

	<div id="footer">
		Copyright &copy; 2007 - 2013 IOLA and Ole Laursen
	</div>

</body>
</html>
```

我自己本身是用來量測 MPU6050 該慣性測量感測器，如有需要歡迎到我的 Github 頁面下載，也歡迎 fork 回去自行修改：[MPU6050-WebPlot](https://github.com/KuoE0/MPU6050-WebPlot)。

**修改記錄**

2014/01/04:

- 新增 tornado 中 View 模組－tornado.template
