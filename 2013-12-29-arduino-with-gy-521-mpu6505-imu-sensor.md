---
layout:  post
title:   "Arduino 與 GY-521 MPU6050 慣性測量感測器"
date:    2013-12-29
tags:    ["Arduino", "OS X", "CLI | 命令列介面", "Ubuntu", "I2Cdevlib", "accelerometer | 加速計", "gyroscope | 陀螺儀", "MPU6050", "IMU | 慣性測量單元"]
feature:
    photo:       true
    creator:     
    url:         
    license:     
    license_url: 
---

目前碩士論文也是打算使用 Arduino 來搭建硬體，其中使用到了慣性測量單元 (IMU) －GY-521，搭載 MPU6050 的加速計與陀螺儀晶片。由於在 I2Cdevlib 中發現有包含 MPU6050 的函式庫，故決定採用該函式庫來進行開發。

I2Cdevlib 是一個開放原始碼的專案，裡頭提供許多微處理裝置的函式庫供使用。要安裝該函式庫只需要到其 Github 頁面將原始碼下載下來，並將其中 `Arduino` 資料夾內的內容通通放到 Arduino 工作目錄中的 `libraries` 資料夾中。

```
$ git clone https://github.com/jrowberg/i2cdevlib /tmp/i2cdevlib
$ mv /tmp/i2cdevlib/Arduino/* $Arduino/libraries/
```

接下來可以在 `libraries/MPU6050/Examples/MPU6050_raw` 中找到範例程式。裡頭有兩個範例，以下僅示範 `MPU6050_raw` 的配置方式。

## Hardware Wiring

![實際接線圖](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-12-29-arduino-with-gy-521-mpu6505-imu-sensor-1.jpg)

這邊僅需要四條杜邦線即可，連線方式如下：

Board | wiring | GY-521
----- | ------ |------
3.3V  |   ↔︎   | VCC
GND   |   ↔︎   | GND
A5    |   ↔︎   | SCL
A4    |   ↔︎   | SDA

這樣接線的原因是 I<sup>2</sup>C 就是透過兩條線來進行通訊，分別是 SCL (clock rate) 與 SDA (data)。而在 Arduino Uno 這塊板子中，A5 剛好就是 SCL 線，A4 則是 SDA 現，所以直接這樣接就完成了。

另外，由於 I<sup>2</sup>C 的通訊是透過固定位址的，預設 MPU6050 透過 I<sup>2</sup>C 通訊時的位址是 `0x68`。如果需要改變這個位址的話，可以透過將 AD0 這接腳接到 3.3V 的電位，如此一來，該 MPU6050 的 I<sup>2</sup>C 的位址將更變為 `0x69`。

## Software Configuration


![Arduino IDE and Serial Output](https://raw.githubusercontent.com/KuoE0/blog-assets/master/content-photos/2013-12-29-arduino-with-gy-521-mpu6505-imu-sensor-2.jpg)

將整個 `MPU6050_raw` 資料夾複製出來到 Arduino 的工作目錄下，並用 Arduino IDE 開啟裡頭的 `MPU6050_raw.ino` 檔案。

選擇硬體規格與串列埠，即可以進行編譯與上傳。上傳完成後，Arduino 就會開始運行了，只要打開串列埠即可看到大量的訊號輸出。這時只要隨意地揮動 GY-521 這塊板子，就能發現輸出的訊號會有較明顯的改變。


## 使用 Ino 命令列工具

如果跟我一樣比較喜歡在 CLI 下工作的話，應該也會選擇使用 Ino 這個工具吧！關於如何設置 Arduino 的 CLI 環境可以參考這篇（[連結](http://blog.kuoe0.tw/posts/2013/11/22/arduino-cli-on-os-x)）。然後，就會發現當輸入 `ino build` 時都會發生錯誤...

```
$ ino build
src/sketch.ino
.build/uno/src/sketch.cpp:2:20: error: I2Cdev.h: No such file or directory
.build/uno/src/sketch.cpp:3:21: error: MPU6050.h: No such file or directory
make: *** [.build/uno/src/sketch.d] Error 1
Make failed with code 2
```

這是因為 Ino 並不會去讀取 Arduino 預設工作目錄下的 `libraries` 資料夾的函式庫，僅會讀取當前目錄下的 `lib` 目錄下的函式庫。以這 `MPU6050_raw` 的範例來說，他額外需要 `I2Cdevlib` 與 `MPU6050` 這兩個函式庫。所以只要把這兩個函式庫的內容抓進 `lib` 目錄下即可成功編譯了！ (P.S. 記得先 `ino clean` 再進行 `ino build`。)

不過我實在不喜歡電腦裡面有太多相同的東西，所以我都直接讓 `lib` 目錄連結到 Arduino 預設工作目錄下的 `libraries` 目錄。
