---
title: "PHP 升級至 8，OwnCloud 加密變慢"
date: "2023-09-21T09:29:31Z"
tags: []
---

# PHP 升級至 8，OwnCloud 加密變慢

## 起因

升級了 PHP 8 後 `stream_write` 不再只會拿到 8192 bytes，

設定 chunk size, buffer 都沒有用：

    stream_set_chunk_size($f, 8192);
    stream_set_read_buffer($f, 8192);
    stream_set_write_buffer($f, 8192);

我是舊版的 OwnCloud 自己維護，對應到新版的 code 是這裡：

`encryption/lib/Crypto/Encryption.php`

    $data = \substr($data, $this->getUnencryptedBlockSize(true));

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

        export XDEBUG_MODE="profile"
    export XDEBUG_CONFIG="profiler_output_name=cachegrind.out.%r.%p.%R"
2. 上傳檔案觸發加密後收集 cachegrind 檔

        cachegrind.out.0ccd29.7._remote_php_webdav__anydesk_dmg-chunking-2132-1-0.gz
3. PHPStorm > Analyze Xdebug Profiler Snapshot … > 選擇上述的 `.gz`

    ![](Screenshot_2023-09-21_at_6-857907ae-9dc0-416a-a405-75b2da1bf3c2.36.37_PM.png)

    可以發現是 `substr` 耗費最多時間，上傳一個 10 MB 檔案加密就花掉 10 秒。

    {{< br >}}

## 解法

改指定 offset 來做 `substr`。

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