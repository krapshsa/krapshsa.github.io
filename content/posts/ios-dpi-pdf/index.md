---
title: "iOS 瀏覽器無法瀏覽高 DPI 的 pdf"
date: "2022-08-15T11:26:00Z"
tags: [javascript]
---

# iOS 瀏覽器無法瀏覽高 DPI 的 pdf

## 起因

NextCloud 有一個功能 (PDF Viewer)，

可以讓我在手機瀏覽器上直接瀏覽 PDF。

但是今天有一個用掃描器掃上來的 PDF 卻無法正常在手機端瀏覽

{{< br >}}

## 症狀

瀏覽 pdf 的時候會一片空白，並且產生錯誤訊息

```Makefile
Canvas area exceeds the maximum limit (width * height > 16777216).
```

{{< br >}}

經查詢原來是 iOS 的 `canvas` element 有大小限制：

[Canvas Area Exceeds The Maximum Limit](https://pqina.nl/blog/canvas-area-exceeds-the-maximum-limit/)

{{< br >}}

跟 PDF.js 無關，所以這條 issue 至今也是無解：

{{< br >}}

## 解決方法

搬運 `askubuntu` 的解法，先在 Server Side 用 ghost script 把 DPI 改小就好了：

```Makefile
gs -sDEVICE=pdfwrite \
-dCompatibilityLevel=1.4 \
-dPDFSETTINGS=/ebook \
-dNOPAUSE \
-dQUIET \
-dBATCH \
-sOutputFile=output.pdf input.pdf
```

{{< br >}}

- `-dCompatibilityLevel` (不知道有什麼用)

    [Understanding PDF compatibility levels in Acrobat 9](https://acrobatusers.com/tutorials/understanding-pdf-compatibility-levels/)
- `-dPDFSETTINGS` 用來調整 DPI
- `-dNOPAUSE` 停用每頁結束時的提示和暫停
- `-dQUIET` 安靜執行，儘量不輸出日誌
- `-dBATCH` 執行到最後一頁退出

{{< br >}}

## 參考資料

[PDF viewer](https://apps.nextcloud.com/apps/files_pdfviewer)

[Canvas Area Exceeds The Maximum Limit](https://pqina.nl/blog/canvas-area-exceeds-the-maximum-limit/)

[How can I reduce the file size of a scanned PDF file?](https://askubuntu.com/questions/113544/how-can-i-reduce-the-file-size-of-a-scanned-pdf-file)

[High Level Output Devices](https://www.ghostscript.com/doc/current/VectorDevices.htm#COMMON)