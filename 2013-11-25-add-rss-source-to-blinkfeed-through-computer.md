---
layout: post
title:  "透過電腦在 BlinkFeed 中增加 RSS 來源"
date:   2013-11-25
tags:   ["HTC", "BlinkFeed", "RSS"]
---

這幾天 HTC one 終於有機會升到 Sense 5.5 版本了！在這個版本中，一個我最喜歡的變更是－BlinkFeed 可以自行增加 RSS 來源！

原本的 BlinkFeed 只能訂閱 HTC 願意給你的內容，雖然剛開始內容很少，不過 HTC 也一直在增加內容來源，例如：愛評網、Mobile01 等等。但是，每個人閱讀的習慣與喜好都不相同，如今可以自定才是真正把 BlinkFeed 大解放啊！只要在瀏覽器中點擊 RSS 的連結就會跳出個選擇開啟程式的選單，然後選擇「BlinkFeed RSS」這個選項來開啟，就可以加入 BlinkFeed 的訂閱了。

但是...我都沒有成功過。上網搜尋了一下發現 XDA 上有人發現可以使用電腦來進行增加 BlinkFeed 的來源。透過 Android SDK 中的 adb 工具，即可輕鬆完成！在此之前記得先安裝 Android SDK 以及 platform-tool，這邊就自行上網搜尋該如何安裝吧！

## 操作方式

假設現在要加入的訂閱來源是 `http://blog.kuoe0.tw/posts.atom`，也就是我的部落格。

接著只要在終端機（或是命名提示字元）輸入以下指令：

```
$ adb shell am start -n com.htc.socialnetwork.rss/.octopus.HeadlineListActivity -a com.htc.socialnetwork.rss.ADD_FEED -eu Uri http://blog.kuoe0.tw/posts.atom
```

並返回以下的結果，

```
Starting: Intent { act=com.htc.socialnetwork.rss.ADD_FEED dat=http: cmp=com.htc.socialnetwork.rss/.octopus.HeadlineListActivity (has extras) }
```

接著就會在手機上看到以下畫面：

![add rss source](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-11-25-add-rss-source-to-blinkfeed-through-computer-1.png)

點擊右上角的「＋」符號，就會開始訂閱啦！自左向右滑開，就會發現自訂的 RSS 來源也在清單中了！

![added it to BlinkFeed](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-11-25-add-rss-source-to-blinkfeed-through-computer-2.png)

如果還要加入其他 RSS 來源的話，只要把我的部落格訂閱連結換成想要訂閱的連結即可！

**參考資料：**

- [http://forum.xda-developers.com/showthread.php?t=2489219](http://forum.xda-developers.com/showthread.php?t=2489219)
