---
title: "瀏覽器下載續傳"
date: "2023-10-05T08:32:28Z"
tags: [PHP]
---

# 瀏覽器下載續傳

## 起因

原本我以為只要有支援 `range` 就可以讓瀏覽器續傳，後來發現這樣條件還不夠。

其他下載軟體的行為我沒測就不知道了。

{{< br >}}

## 解方

1. Server 需要送出 `etag` 來做後續資源是否變更的判段
2. Browser 接到有 `etag` 的下載，如果斷線續傳會送出：
    1. `IF_RANGE` 值是之前的 `etag` ，Server 要針對這個值來判斷檔案是否變了，變了的話就不能走續傳這條路了。
    2. `RANGE` 從哪裡開始下載

{{< br >}}

如果一開始就沒有送 `etag` 給瀏覽器，那他連 `RANGE` 都不會送。

{{< br >}}

## 小問題

chrome 我用 fast 3G 再切到 offline 讓他終止下載。

回到 fast 3G 後選擇 `Resume` 續傳他會無視 fast 3G 的速度限制下載。

{{< br >}}

## 簡易實做

`download.php`

```PHP
<?php

const READ_CHUNK_SIZE = 1024;

$file = 'random_data.bin';

if (!file_exists($file)) {
    header($_SERVER['SERVER_PROTOCOL'] . ' 404 Not Found');
    exit;
}

$fileSize = filesize($file);
$fileName = basename($file);
$etag     = md5_file($file); // 計算文件的md5以作為ETag

header("Content-Type: application/octet-stream");
header("Content-Disposition: attachment; filename=$fileName");
header("Accept-Ranges: bytes");
header("Content-Length: $fileSize");
header("ETag: $etag"); // 發送ETag header

$start = 0;
$end = $fileSize - 1;

// 如果 If-Range 頭部的值不等於我們的 ETag，那麼忽略 Range 頭部並發送整個文件
if (
    isset($_SERVER['HTTP_IF_RANGE']) &&
    isset($_SERVER['HTTP_RANGE']) &&
    $_SERVER['HTTP_IF_RANGE'] == $etag
) {
    list(, $range) = explode('=', $_SERVER['HTTP_RANGE'], 2);
    if (strpos($range, ',') !== false) {
        header('HTTP/1.1 416 Requested Range Not Satisfiable');
        header("Content-Range: bytes $start-$end/$fileSize");
        exit;
    }

    if ($range == '-') {
        $start = $fileSize - substr($range, 1);
    } else {
        $range = explode('-', $range);
        $start = $range[0];
        $end   = (isset($range[1]) && is_numeric($range[1])) ? $range[1] : $fileSize;
    }

    if ($start > $end || $start > $fileSize - 1 || $end >= $fileSize) {
        header('HTTP/1.1 416 Requested Range Not Satisfiable');
        header("Content-Range: bytes $start-$end/$fileSize");
        exit;
    }

    header('HTTP/1.1 206 Partial Content');
    header("Content-Range: bytes $start-$end/$fileSize");
}

$stream = fopen($file, 'rb');
fseek($stream, $start);

while (!feof($stream) && ($pos = ftell($stream)) <= $end) {
    if ($pos + READ_CHUNK_SIZE > $end) {
        echo fread($stream, $end + 1 - $pos);
    } else {
        echo fread($stream, READ_CHUNK_SIZE);
    }
    flush();
}
fclose($stream);
```

{{< br >}}

`index.html`

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Download Resume Example</title>
</head>
<body>
    <a href="download.php">點擊此處下載檔案</a>
</body>
</html>
```

{{< br >}}