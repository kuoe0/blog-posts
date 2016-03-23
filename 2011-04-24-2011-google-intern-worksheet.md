---
layout:  post
title:   "2011 Google Intern Worksheet"
date:    2011-04-24
tags:    ["Google", "intern | 實習"]
feature:
    photo:       true
    creator:     "Carlos Luna"
    url:         "https://www.flickr.com/photos/carlosluna/2856173673"
    license:     "CC 2.0"
    license_url: "https://creativecommons.org/licenses/by/2.0/"
---

今天早上10點要寫google intern的worksheet，平常都睡到12點的我，整個很擔心會睡過頭。昨天特地打給我媽，請他今天早上一定要叫醒我。早上還沒九點我媽媽就打來叫醒我了，起來就先吃個麵包喝個伴點當早餐，然後開始很緊張地等待10點到來。10點一到，Gmail馬上收到信，裡面有8個題目。

由於只有兩個小時，感覺時間緊迫，沒有看說明就直接去寫題目了。直到我每題都看完，能想出來能掰出來的都寫了，11 點多快結束時，我才去看前面的說明，發現他有說，四題 General Tech Question 只要挑三題寫，四題 Product Related Question 只要挑一題寫。害我作答過程一直很趕，一直擔心寫不完，現在覺得我的回答都好爛，方法也好爛，我要跟 google intern 說掰掰了嗎 T口T。

## General Tech Question

**第一題**

> Implement toExcelColumnName(n) that returns "A" for n=1, "B" for n=2, "Z" for n=26, "AA" for n=27, “AB” for n=28, and so on.

這題應該是最簡單的題目，就他輸入一個數字，要輸出像在 Excel 上，column 的名稱，第 1 行是 A，第 2 行是 B，第 27 行是 AA，第 52 行是 AZ，以此類推。

我的想法就是先對 n 取26的模數，模數為 1，表示最尾端的字母為 A，模數為 25 就為 Y，模數為 0 就是 Z。取完模數後，再對 n 除以 26，但若之前取得到的模數為 0，要先對 n 減去 26 再做除以 26 的步驟。如此重複直到 n 被除到剩 0 結束，就可以得到 column 的名稱了。

<script src="https://gist.github.com/1595344.js?file=toExcelColumnName.cpp"></script>


**第二題**

> Given a class that reads from a file in 4K chunks, implement a class (in C++ or Java) that can read arbitrary amounts of data.

其實不是很懂這題要幹嘛，感覺就是寫一個可以讀取任意指定長度的資料到 buffer 裡面，但他又給了個可以讀 4K 的 class，不曉得有沒有用到。我也不知道確切一幹嘛，就直接寫了個 class，直接使用 fread 進行指定長度的讀取。結果不能用 fread 我就爆掉了，這題真的讓我覺得很擔心  。

**第三題**

> You are given a stream of quotes for a stock from the last trading day. Assume its already time sorted. Find the maximum amount of money you could have made on this stock by making 1 or 2 transactions, respectively. A buy and a sell is counted as one transaction. You can only buy or hold one share at a time. Example: Time/price pairs are (1, 10) (2, 11) (3, 7) (4, 15) (5, 8) (6, 17) (7, 16). For N=1 answer is 10: buy at 3 and sell at 6. For N=2 answer is 8 + 9: buy at 3, sell at 4, buy at 5, sell at 6.

給你股票的交易資訊，買進賣出算一次交易。要求交易一次跟交一兩次的最大收益。其實這題剛開始沒看到交易一次跟兩次，所以直接用很笨的方法去寫，我竟然直接用 backtracking 去暴力硬搜，我覺得我這題也爆掉了。交易一次感覺 O(n) 就可以了，交易兩次感覺也 O(n<sup>2</sup>) 就可以了。這題應該會害我進不去 intern 了。

**第四題**

> Suppose a simple network topology with N source nodes that send data to M sink nodes across one switch/router node, where all links are fixed bandwidth and fixed latency. Suppose futher that all nodes and the switch/router have perfectly synchronized clocks, and that nodes can cooperatively plan the precise timing of their workloads in advance. What is the optimal buffering architecture for the router? Explain your reasoning. Explain the relationship between latency and utilization, where latency is the time taken to send data from sources to sinks and utilization is the ratio of total workload quantity to router/switch capacity.

這題完全看不懂，感覺是專業領域的問題，但可惜我應該不是在這領域的，所以我就完全跳過了。測驗後問電子哥，他說這是 network-on-chip 的資源分配問題。說這個現在很熱門，超級熱門，但他也不會就是了，感覺考這個太難了吧。

---

## Product Related Question

**第一題**

> Suppose you can modify the software running on your 3G cellular phone. How would you design a tethering system, that is, a system allowing your laptop or computer to connect to the Internet through your 3G phone? List the software components you need and briefly describe how it works.

這題要求設計一個系統，讓電腦可以透過 3G 手機上網。我最後選擇答這題，但也全都亂掰的，我根本完全不會。我就答什麼我覺得要用USB介面，因為 USB 介面容易取得，加上現在 USB 3.0 問世，擁有極快的傳輸速度之類的。

**第二題**

> How to speed up iGoogle and make it fast?

就如前面所說，我以為每題都要作答，所以這題我也有寫答案，但最後submit 前把它砍了，這題答案我寫的很愚蠢：

- 少用點Widgets
- 用 Google Chrome Browser，最快的瀏覽器
- 增加你的網路頻寬
- 第二點根本是在拍馬屁

**第三題**

> Some mobile phones have one back camera and one front camera. Think of a cool or useful application that use two cameras at the same time. Describe how it works in detail and its algorithm if there is any.

現在許多手機都有前後鏡頭，要求設計一個應用程式可以同時使用這兩個鏡頭。這題我也放棄，但還好只要挑一題寫就好。這種要無中生有的創意問題，對我來說如同登天一般困難，所以就放棄了。

**第四題**

> Propose an architectural improvement (hardware, software, or both) that would reduce boot time for ChromeOS devices. More specifically, contrast your proposal with what you think are the primary boot time bottlenecks in the existing CR-48 devices.

設計硬體或軟體加速 Chrome OS 的開機速度。對硬體不熟，對 OS 也不熟，根本無從下手。但也有作答，我寫用 SSD 儲存 boot loader，其他儲存設備都可以捨棄，全部改裝 ram，畢竟 Chrome OS 是雲端作業系統，不需要儲存資料在本地，所以也不太需要靜態儲存裝置，所以放個小小的 SSD 存 boot loader 就夠了吧  。但 SSD 可以加快開機，這不是大家都知道的事嗎  ，所以我覺得這題的回答很爛，也決定不選擇他做為 submit 的題目。

---

submit 後，會收到以下的訊息。

> Thank you for completing the worksheet! As I mentioned in my previous email, if I don’t contact you in the next few weeks, it might be that we filled the internship role. However, it doesn’t mean that we won’t find a role that fits you in the future. Please keep an ear out for possible internship program for next year. Of course, when you are graduating and ready to search for a permanent role, please don’t hesitate to follow-up with me too for full-time positions. Thanks again and enjoy the rest of the weekend! Jane

真希望他在這幾個禮拜可以聯絡我。

**Please contact me in the next few weeks.**


