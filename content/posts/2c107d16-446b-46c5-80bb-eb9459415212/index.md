---
title: "下載續傳"
date: "2023-10-05T08:32:28Z"
tags: [PHP]
---

# 下載續傳

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