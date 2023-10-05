---
title: "PHP 升級至 8，OwnCloud 加密變慢"
date: "2023-09-21T09:29:31Z"
tags: [Nextcloud]
---

# PHP 升級至 8，OwnCloud 加密變慢

## 起因

升級了 PHP 8 後 `stream_write` 不再只會拿到 8192 bytes，

設定 chunk size, buffer 都沒有用：

```PHP
stream_set_chunk_size($f, 8192);
stream_set_read_buffer($f, 8192);
stream_set_write_buffer($f, 8192);
```

我是舊版的 OwnCloud 自己維護，對應到新版的 code 是這裡：

`encryption/lib/Crypto/Encryption.php`

```PHP
$data = \substr($data, $this->getUnencryptedBlockSize(true));
```

因為這個 `$data` 現在不只 8192 bytes 了，最大可能是 chunk size，所以這行很慢。

{{< br >}}

至於怎麼找到是這個問題的，參見 profiling。

一點小心得是：

觀察 CPU 的 loading 是滿載的，以為是加密慢，但是試著 `strace` ，加總秒數卻是很少的。

看了半天用直覺猜可能是字串處理的問題後猜對了，但其實不應該用矇的。

下次遇到應該先 profiling ，實驗環境的話成本很低。

{{< br >}}

## Profiling

1. 設定環境變數

    ```Bash
    export XDEBUG_MODE="profile"
    export XDEBUG_CONFIG="profiler_output_name=cachegrind.out.%r.%p.%R"
    ```
2. 上傳檔案觸發加密後收集 cachegrind 檔

    ```Bash
    cachegrind.out.0ccd29.7._remote_php_webdav__anydesk_dmg-chunking-2132-1-0.gz
    ```
3. PHPStorm > Analyze Xdebug Profiler Snapshot … > 選擇上述的 `.gz`

    ![](Screenshot_2023-09-21_at_6-857907ae-9dc0-416a-a405-75b2da1bf3c2.36.37_PM.png)

    可以發現是 `substr` 耗費最多時間，上傳一個 10 MB 檔案加密就花掉 10 秒。

    {{< br >}}

## 解法

改指定 offset 來做 `substr`。

```PHP
-        // While there still remains some data to be processed & written
-        while (strlen($data) > 0) {
-            // Remaining length for this iteration, not of the
-            // entire file (may be greater than 8192 bytes)
-            $remainingLength = strlen($data);
+        $position        = 0;
+        $remainingLength = strlen($data);

+        // While there still remains some data to be processed & written
+        while ($remainingLength > 0) {
             // If data remaining to be written is less than the
             // size of 1 6126 byte block
             if ($remainingLength < 6126) {
@@ -457,13 +456,11 @@ class Stream
                 // data in writeCache after the writing round
                 // has finished, then the data will be written
                 // to disk by $this->flush().
-                $this->writeCache = $data;
-
-                // Clear $data ready for next round
-                $data = '';
+                $this->writeCache = substr($data, $position);
+                break;
             } else {
                 // Read the chunk from the start of $data
-                $chunk = substr($data, 0, 6126);
+                $chunk = substr($data, $position, 6126);

                 $encrypted = $this->preWriteEncrypt($chunk, $this->plainKey);

@@ -472,10 +469,8 @@ class Stream
                 // being handled totals more than 6126 bytes
                 fwrite($this->handle, $encrypted);

-                // Remove the chunk we just processed from
-                // $data, leaving only unprocessed data in $data
-                // var, for handling on the next round
-                $data = substr($data, 6126);
+                $position        += 6126;
+                $remainingLength -= 6126;
             }
         }
```

{{< br >}}

## 補充資訊 - 測試 `streamWrapper`

1. 先用 `dd` 開出一個測試用的檔案

    ```Bash
    dd if=/dev/urandom of=random_data_10.bin bs=1M count=10
    ```
2. 實作一個 stream wrapper，我在 `stream_write` 有寫 Log 看看讀到多少資料

    ```Bash
    <?php
    
    class DebugStreamWrapper {
        private $realStream;
    
        public function stream_open($path, $mode, $options, &$opened_path) {
            $path = substr($path, strlen('debug//'));
            $this->realStream = fopen($path, $mode);
            if ($this->realStream) {
                return true;
            }
    
            $this->stream_set_option(STREAM_OPTION_WRITE_BUFFER, STREAM_BUFFER_FULL, 8192);
    
            return false;
        }
    
        public function stream_read($count) {
            return fread($this->realStream, $count);
        }
    
        public function stream_write($data) {
            echo sprintf("length: %d\n", strlen($data));
            return fwrite($this->realStream, $data);
        }
    
        public function stream_tell() {
            return ftell($this->realStream);
        }
    
        public function stream_eof() {
            return feof($this->realStream);
        }
    
        public function stream_seek($offset, $whence) {
            return fseek($this->realStream, $offset, $whence) === 0;
        }
    
        public function stream_close() {
            return fclose($this->realStream);
        }
    
        public function stream_set_option($option, $arg1, $arg2) {
            echo sprintf("option:%s\n", $option);
    
            switch ($option) {
                case STREAM_OPTION_BLOCKING:
                    return stream_set_blocking($this->realStream, $arg1);
                case STREAM_OPTION_READ_TIMEOUT:
                    return stream_set_timeout($this->realStream, $arg1, $arg2);
                case STREAM_OPTION_WRITE_BUFFER:
                    return stream_set_write_buffer($this->realStream, $arg1);
                default:
                    return false;
            }
        }
    }
    
    stream_wrapper_register("debug", "DebugStreamWrapper");
    
    $data = file_get_contents('./random_data_10.bin');
    
    $file = fopen("debug://var/www/html/random_data.bin_10.enc", "w+");
    
    var_dump(stream_get_meta_data($file));
    
    fwrite($file, $data);
    
    fclose($file);
    ```
3. 測試，這樣寫就可以很快在不同環境切換

    ```Bash
    docker run -v .:/var/www/html -it php:8.2.10-fpm-bullseye php test.php
    ```

    ```Bash
    docker run -v .:/var/www/html -it php:7.4.33-fpm-bullseye php test.php
    ```

{{< br >}}