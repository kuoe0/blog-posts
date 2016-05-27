---
layout: post
title:  "重置被遺忘的 OS X 系統管理員密碼"
date:   2013-04-12
tags:   ["OS X", "security | 安全性"]
image:  "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2013-04-12-reset-the-forgotten-password-of-admin-in-os-x.jpg"
image_info:
    creator:     "Andreas Wieser"
    url:         "https://www.flickr.com/photos/andilicious/4279831984"
    license:     "CC 2.0"
    license_url: "https://creativecommons.org/licenses/by/2.0/"
---

實驗室有台似乎很久沒拿來用的 Macbook Air，而且前陣子才剛維修完拿回來，維修理由很蠢，竟然是乾燥劑塞進了耳機孔…。回到本篇重點，老師今天心血來潮的想拿出來用，但開機後遲遲無法登入，因為老師竟然忘記密碼了…。於是 google 了一下，找到了以下兩種方法：

## 方法一

1. 開機時按住 option 按鍵，此時會跳出開機選單
2. 選擇 recovery 10.8 這個硬碟進入 recovery mode
3. 點擊上方的選單中的工具程式，並選擇終端機
4. 開啓終端機後，輸入 `resetpassword` 會開啓重置密碼的程式
5. 選擇系統所安裝的硬碟，即可選擇要更改密碼的使用者
6. 繼續輸入新的密碼後就大功告成了

這個方法僅適用於 10.7 Lion 與 10.8 Mountain Lion！

## 方法二

1. 開機時按住 command+S
2. 此時會進入 single user mode，並會跳出 #root> 的提示
3. 輸入指令 `fsck -y` 進行硬碟檢測（這一步驟可忽略）
4. 接著輸入 `mount -uaw` 進行掛載硬碟並將掛載模式更變為可讀可寫
5. 接著輸入 `rm /var/db/.AppleSetupDone` 將 .AppleSetupDone 檔案刪除
6. 再輸入 `reboot` 重新開機
7. 會發現電腦似乎回到了第一次開機的狀態，於是即可設定一個新的系統管理員
8. 用新的系統管理員登入後打開 System Preferences，並點選 Users & Groups 的項目
9. 將視窗左下角的鎖頭打開
10. 點選舊的系統管理員賬戶，點擊 Change Password
11. 輸入新的密碼就大功告成了

如果不希望電腦裡還存在著剛剛新增的系統管理員賬戶的話，用舊有的系統管理員登入後，一樣來到 System Preferences 的 Users & Groups 項目，把鎖頭打開後，就可以刪除剛剛新增的系統管理員了！

這個方法似乎對於 10.6 Snow Leopard 以上的版本都適用！

> 突然覺得要更變密碼也太容易，看來電腦只要遺失，如果有機密資料就危險了！
