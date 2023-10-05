---
title: "jQuery File Upload 支援上傳續傳"
date: "2023-10-04T07:27:27Z"
tags: [javascript]
---

# jQuery File Upload 支援上傳續傳

# 起因

客戶希望在網路很差的環境使用 web 上傳，所以要可以支援續傳。

目前我們 web 的上傳是使用 [blueimp/jQuery-File-Upload](https://github.com/blueimp/jQuery-File-Upload) 做的。

{{< br >}}

# 解方

官方文件的註解有如下片段：

```JavaScript
// Callback for failed (abort or error) uploads:
// fail: function (e, data) {}, // .bind('fileuploadfail', func);
```

故使用 `fileuploadfail` 這個事件來實做網路斷線處理。

{{< br >}}

`index.html`

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>File Upload with Chunking</title>
    <link rel="stylesheet" href="https://blueimp.github.io/jQuery-File-Upload/css/jquery.fileupload.css">
</head>
<body>

<input id="fileupload" type="file" name="files[]" multiple>

<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://blueimp.github.io/jQuery-File-Upload/js/vendor/jquery.ui.widget.js"></script>
<script src="https://blueimp.github.io/jQuery-File-Upload/js/jquery.iframe-transport.js"></script>
<script src="https://blueimp.github.io/jQuery-File-Upload/js/jquery.fileupload.js"></script>

<script>
$(function () {
    $('#fileupload').fileupload({
        url: 'upload.php',
        dataType: 'json',
        retryTimeout: 60000, // 1 minute
        maxRetries: 3,
        _retries: 0,
        maxChunkSize: 1000000, // 1MB
        done: function (e, data) {
            console.log('Upload completed:', data.result);
        }
    })
    .on('fileuploadfail', function (e, data) {
      if (data.errorThrown !== 'abort' && data.uploadedBytes < data.files[0].size && data._retries < data.maxRetries) {
        data._retries++;
        setTimeout(function () {
          console.log('Retrying upload...');
          data.submit();
        }, 60000);  // Retry after 1 minute
        return;
      }

      if (data.errorThrown === 'abort') {
        console.log('File upload was aborted by the user.');
      } else {
        console.log('Upload failed.');
      }
    });
});


</script>

</body>
</html>
```

{{< br >}}

`upload.php`

```PHP
<?php

$uploadDir = 'uploads/';

if (!file_exists($uploadDir)) {
    mkdir($uploadDir, 0777, true);
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!isset($_FILES['files']['tmp_name'][0]) || !is_uploaded_file($_FILES['files']['tmp_name'][0])) {
        die(json_encode(['status' => 'error', 'message' => 'No file uploaded.']));
    }

    if (!isset($_FILES['files']['name'][0]) || empty($_FILES['files']['name'][0])) {
        die(json_encode(['status' => 'error', 'message' => 'File name not provided.']));
    }

    $fileName = $_FILES['files']['name'][0];

    $chunk  = isset($_POST['chunk'])  ? intval($_POST['chunk'])  : 0;
    $chunks = isset($_POST['chunks']) ? intval($_POST['chunks']) : 0;

    $filePath = $uploadDir . DIRECTORY_SEPARATOR . $fileName;

    if (!$chunks || $chunk == $chunks - 1) {
        move_uploaded_file($_FILES['files']['tmp_name'][0], $filePath);
    } else {
        $tempDir = $uploadDir . DIRECTORY_SEPARATOR . $fileName . '_temp';
        if (!file_exists($tempDir)) {
            mkdir($tempDir, 0777, true);
        }

        move_uploaded_file($_FILES['files']['tmp_name'][0], $tempDir . DIRECTORY_SEPARATOR . $chunk);

        if ($chunk == $chunks - 1) {
            for ($i = 0; $i < $chunks; $i++) {
                $fileData = file_get_contents($tempDir . DIRECTORY_SEPARATOR . $i);
                file_put_contents($filePath, $fileData, FILE_APPEND);
                unlink($tempDir . DIRECTORY_SEPARATOR . $i);
            }
            rmdir($tempDir);
        }
    }

    echo json_encode(['status' => 'success']);
}

?>
```

{{< br >}}

快速測試一下：

```Bash
php -S localhost:8000
```

{{< br >}}

上傳一個檔案到一半，用 debug tool 切成 offline 再切回來：

![](Screenshot_2023-10-04_at_3-0669506e-2f27-42dc-9f35-a90f9cb28999.34.51_PM.png)

可以看到上傳之敗之後過一分鐘就重試上傳了：

![](Screenshot_2023-10-04_at_3-d6768f97-4576-4359-8c3b-1be4077c22fa.36.27_PM.png)