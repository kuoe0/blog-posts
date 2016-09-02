---
layout: post
title:  "如何從 Firefox 中把 compositor 產生的畫面擷取出來"
date:   2016-09-02
tags:   ["Firefox", "compositor", "debug"]
feature:
    photo: false 
---

最近在 Firefox for Android 上替 Presentation API 在加入 Chromecast 的支援時，遇到一個問題。問題敘述如下：

> 當成功在 Chromecast 上投影畫面後，手機上的 Firefox 就再也無法更新畫面了。

由於不知道問題的癥結點在哪，就嘗試了各種方式來 debug。其中一種方式就是檢查 compositor 是否有產生正確的畫面，因此需要將 compositor 每一次產生的結果給輸出。由於 graphics 這邊筆者很不熟，主要是請教前輩該怎麼 debug，怕下次還會有需要因此打算記錄下來！不過 Gecko 天天都在變化，感覺很快這招就沒用了，但概念上應該類似。

由於 Android 使用 OpenGL 來作為繪圖的函式庫，因此會運行到的程式碼會在這裡 `gfx/layers/opengl/CompositorOGL.cpp`。根據前輩的說法，在 `CompositorOGL::EndFrame()` 被呼叫時就表示 compositor 已經把要繪製的畫面產生好了！因此，可以在這個函式中把該結果給讀出來。只要在 `CompositorOGL::EndFrame()` 中加入以下的程式碼即可：

```c++
// count
static int dumpCnt = 0;
++dumpCnt;

// filename
std::stringstream sstream;
sstream << "/sdcard/Android/data/org.mozilla.fennec_kuoe0/files/dump/"
        << dumpCnt << "_dump_"
        << (void*) this // which compositor
        << "_" << mSurfaceSize.width << "x" << mSurfaceSize.height // size
        << ".rgb";

// read pixel data from GL context
char *imageData = new char[mSurfaceSize.width * mSurfaceSize.height * 4];
mGLContext->fReadPixels(0, 0, mSurfaceSize.width, mSurfaceSize.height, LOCAL_GL_RGBA, LOCAL_GL_UNSIGNED_BYTE, imageData);

// write pixel data into the file
FILE *pWritingFile = fopen(sstream.str().c_str(), "wb+");
if (pWritingFile) {
  printf_stderr("Write fbo(%d, %d) to file!", mSurfaceSize.width, mSurfaceSize.height);
  fwrite(imageData, mSurfaceSize.width * mSurfaceSize.height * 4, 1, pWritingFile);
  fclose(pWritingFile);
} else {
  printf_stderr("Cannot dump to file!");
}

delete imageData;
```

輸出的檔案會是 RGBA 的 raw 檔，建議在檔名上加上寬高比較方便後期進行 raw 檔的還原。畢竟 raw 檔裡頭只有 RGBA 的資訊，並沒有其他 metadata，像是寬高之類的。然後，可以透過 ImageMagick 可以輕易把 RGBA 檔轉成 PNG 的格式，注意副檔名一定要是 `.rgba`，否則 ImageMagick 不會知道他是 RGBA 檔。

```
$ convert -size "1280x720" -flip -depth 8 15_dump_0x879e1ec0_1280x720.rgba 15_dump_0x879e1ec0_1280x720.png
```

或是直接複製以下的 script 存檔為 `rgba2png.sh`：

```sh
SOURCE_FOLDER="$1"
TARGET_FOLDER="$2"

for RAW_FILE in $(find $SOURCE_FOLDER -name "*.rgba"); do
	SIZE=$(echo $RAW_FILE | cut -d'_' -f4 | cut -d'.' -f1)
	BASENAME=$(basename $RAW_FILE | cut -d'.' -f1)
	PNG_FILE="${TARGET_FOLDER}/${BASENAME}.png"
	convert -size "$SIZE" -flip -depth 8 "$RAW_FILE" "$PNG_FILE" 2>/dev/null
done
```

接著只要夠過以下的方式執行該 script，就可以對整個資料夾內的 raw 進行轉檔：

```
$ ./rgba2png.sh [source] [target]
```

