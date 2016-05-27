---
layout: post
title:  "徹底清除磁碟資料（填零）"
date:   2013-07-17
tags:   ["Unix", "disk | 磁碟", "security | 安全性"]
image:  "https://raw.githubusercontent.com/KuoE0/blog-assets/master/feature-photos/2013-07-17-wipe-data-zero-filling.jpg"
image_info:
    creator:     "The Preiser Project"
    url:         "https://www.flickr.com/photos/thepreiserproject/12118842256"
    license:     "CC 2.0"
    license_url: "https://creativecommons.org/licenses/by/2.0/"
---

之前聽過學長說他把電腦還實驗室時，由於一般格式化通常透過一些復原軟體就可以把資料復原，所以他都會把硬碟填零，就是把硬碟中每一個 bit 設為 0。一直以來都不知道到底該怎麼把硬碟填零，最近突然想到之前在製作 Linux 的 live usb 時，用過 `dd` 這個指令，該指令可以複製指定的內容到指定的位置。

於是我突然突發奇想想說是不是可以把 `/dev/null` 用 `dd` 寫入到某個分割區來達到清空硬碟的效果，不過我失敗了…。`/dev/null` 不提供任何資料，任何讀取 `/dev/null` 者，都會馬上收到 `EOF`，所以試圖利用 `/dev/null` 是不可行的。

上網查了一下發現原來有另一個裝置可以一直提供 null 字元，就是 `/dev/zero`。所以只要使用下列指令即可達到對硬碟填零的效果：

```bash
dd if=/dev/zero of=/dev/<destination partition>
```

另外，也可以填入亂數來混淆資料，應該可以達到更進一步的安全，只要將資料來源改成 `/dev/urandom`：

```bash
dd if=/dev/urandom of=/dev/<destination partition>
```

在利用 `xxd` 來查看分割區內的資料，指令如下：

```bash
$ sudo xxd /dev/disk2s2 | less
0000000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000030: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000040: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000050: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000060: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000070: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000080: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000090: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

可以發現該分割區的所有資料都被填為 0 了！如果填入的是亂數，則可以看到雜亂無章的內容：

```bash
$ sudo xxd /dev/disk2s2 | less
0000000: f155 d23b fa14 e7fa 2574 ba8d 3ee1 64c0  .U.;....%t..>.d.
0000010: 367a bb71 fb22 0a3d 3a83 bc42 897b 26b2  6z.q.".=:..B.{&.
0000020: b0ad fd62 f04a a6c6 3fee 7f4f cf14 1a35  ...b.J..?..O...5
0000030: 4ec0 6500 221a e977 7a5c e771 e9c2 e3c4  N.e."..wz\.q....
0000040: 79bc 92b5 7b82 75b5 8418 27bc 122e 5742  y...{.u...'...WB
0000050: 70f1 0bd6 7ea2 21a0 7534 4876 aacf 4ccb  p...~.!.u4Hv..L.
0000060: 18c6 66d1 8f34 7e1e 08e0 f162 dbf1 be2c  ..f..4~....b...,
0000070: 5c4e e58c 7605 35e1 7fc1 001d 71d6 5ad6  \N..v.5.....q.Z.
0000080: 3a3e cc19 6820 5517 0c78 df8e 95c8 af99  :>..h U..x......
0000090: 8075 0095 aca8 ac91 e343 4e1f 017a 46a9  .u.......CN..zF.
```

在填零之前需要得知分割區的檔案位置，使用以下指令即可：

**df**

```bash
$ df
Filesystem     1K-blocks      Used Available Use% Mounted on
/dev/disk0s2   244277768 154191532  89830236  64% /
/dev/disk2s2    30405312      3168  30402144   1% /Volumes/UNTITLED
```

**mount**

```bash
$ mount
/dev/disk0s2 on / (hfs, local, journaled)
devfs on /dev (devfs, local, nobrowse)
map -hosts on /net (autofs, nosuid, automounted, nobrowse)
map auto_home on /home (autofs, automounted, nobrowse)
/dev/disk2s2 on /Volumes/UNTITLED (msdos, local, nodev, nosuid, noowners)
```

**diskutil (OS X only)**

假設已知該分割去掛在 /Volumes/UNTILED 這個位置。

```bash
$ diskutil info /Volumes/UNTITLED
   Device Identifier:        disk2s2
   Device Node:              /dev/disk2s2
   Part of Whole:            disk2
   Device / Media Name:      DOS_FAT_32_Untitled_2

   Volume Name:              UNTITLED
   Escaped with Unicode:     UNTITLED

   Mounted:                  Yes
   Mount Point:              /Volumes/UNTITLED
   Escaped with Unicode:     /Volumes/UNTITLED

   File System Personality:  MS-DOS FAT32
   Type (Bundle):            msdos
   Name (User Visible):      MS-DOS (FAT32)

   Partition Type:           Microsoft Basic Data
   OS Can Be Installed:      No
   Media Type:               Generic
   Protocol:                 USB
   SMART Status:             Not Supported

   Total Size:               31.1 GB (31142666240 Bytes) (exactly 60825520 512-Byte-Blocks)
   Volume Free Space:        31.1 GB (31131795456 Bytes) (exactly 60804288 512-Byte-Blocks)
   Device Block Size:        512 Bytes

   Read-Only Media:          No
   Read-Only Volume:         No
   Ejectable:                Yes

   Whole:                    No
   Internal:                 Yes
```

並且，在填零之前務必要把分割區解除掛載（unmount 非 eject），利用以下指令或是圖形化界面即可：

**umount**

```bash
$ umount -f /Volumes/UNTITLED
```

**diskutil (OS X only)**

```bash
$ diskutil umount /Volumes/UNTITLED
Volume UNTITLED on disk2s2 unmounted
```

是說，後來發現一個指令叫做 `shred`，似乎也可以達到一樣的效果，而且更加方便！
