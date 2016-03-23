---
layout:  post
title:   "Arduino 使用 Ino 進行編譯時出現 Robot_Control 的錯誤訊息"
date:    2013-11-24
tags:    ["Arduino", "OS X", "CLI | 命令列介面", "Ubuntu"]
feature:
    photo:       true
    creator:     
    url:         
    license:     
    license_url: 
---

發現當使用 Ino 進行編譯時都會出現以下的編譯錯誤訊息：

```
$ ion build
...
Scanning dependencies of Robot_Control
src/sketch.cpp
Robot_Control/glcdfont.c
Robot_Control/utility/twi.c
Robot_Control/Adafruit_GFX.cpp
Robot_Control/Arduino_LCD.cpp
Robot_Control/ArduinoRobot.cpp
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp: In constructor 'RobotControl::RobotControl()':
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:8: error: 'LCD_CS' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:8: error: 'DC_LCD' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:8: error: 'RST_LCD' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp: In member function 'void RobotControl::begin()':
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:18: error: 'MUXA' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:18: error: 'MUXB' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:18: error: 'MUXC' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:18: error: 'MUXD' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:19: error: 'MUX_IN' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:22: error: 'BUZZ' was not declared in this scope
/Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control/ArduinoRobot.cpp:25: error: 'Serial1' was not declared in this scope
make: *** [.build/nano328/Robot_Control/ArduinoRobot.o] Error 1
Make failed with code 2
```

其實不太確定是什麼問題，目前也沒使用到該函式庫，我猜 Ino 可能會莫名跑去編譯沒用到的函式庫，然後 Arduino 官方提供的 Robot_Control 這個函式庫可能也有問題。

一個暫時的解決方法是，直接把該函式庫刪除即可！

```
$ rm -rf /Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control
```

不過，因為其實還不太熟 Arduino，所以我決定暫時把它隱藏起來就是。

```
$ mv /Applications/Arduino.app/Contents/Resources/Java/libraries/Robot_Control /Applications/Arduino.app/Contents/Resources/Java/libraries/.Robot_Control
```

查了些資料，好像說其中有些程式碼有問題的樣子，所以如果有需要該函式庫的話，可以到以下的連結下載。

[https://github.com/adafruit/Adafruit-GFX-Library](https://github.com/adafruit/Adafruit-GFX-Library)

並將該函式庫放在 Arduino 預設工作目錄中的 `libraries` 目錄下，並取名為 `Adafruit_GFX`。
