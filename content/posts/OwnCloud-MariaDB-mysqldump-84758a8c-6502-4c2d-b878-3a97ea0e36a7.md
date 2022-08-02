---
title: "OwnCloud MariaDB mysqldump 失敗"
date: "2022-07-27T14:58:00+08:00"
draft: false
toc: true
autoCollapseToc: false
comment: true
categories: []
tags: [mariadb]
---

### 徵狀

近期遭遇 `mysqldump` 備份 OwnCloud 的 DB 結果 crash 的問題

google 了一下發現 NextCloud 也有人發生類似的問題

[Table `oc_filecache_extended` corrupt?](https://help.nextcloud.com/t/table-oc-filecache-extended-corrupt/123149)

摘錄連結內的錯誤訊息：

```
Thread pointer: 0x8548068d8
Attempting backtrace. You can use the following information to find out
where mysqld died. If you see no messages after this, something went
terribly wrong...
stack_bottom = 0x7fffdf76cf38 thread_stack 0x3c000
0x12fcbfc <my_print_stacktrace+0x3c> at /usr/local/libexec/mariadbd
0xc6330f <handle_fatal_signal+0x28f> at /usr/local/libexec/mariadbd
0x8018f7de0 <pthread_sigmask+0x530> at /lib/libthr.so.3

Trying to get some variables.
Some pointers may be invalid and cause the dump to abort.
Query (0x85484dab0): show fields from `oc_cards`

Connection ID (thread ID): 4
Status: NOT_KILLED
```

有問題的語句就是這條：

```
show fields from <TABLE>
```

執行檢查，但是結果也全部都是 OK

```
mysqlcheck --all-databases
```

### 急救

由於這台機器還有人在用，要想辦法趕快讓服務正常。

經過多方嘗試，目前有效的做法如下：

1. 開啟一台相同設定的新 DB，為了讓他產生好 DB 基本資料
```
docker run -v ${PWD}/data:/bitnami/mariadb/data -p 3306:3306 -e MARIADB_ROOT_PASSWORD=mypassword -d bitnami/mariadb:10.5.15
```
2. 關閉新 DB
```
docker stop <db 的 id>
```
3. 複製 DB 資料夾下， `ibdata1` 與 `owncloud` 的資料夾到新 DB 資料夾下
4. 再次啟動新 DB

之後也試著去找出原因，但是連 core file 都沒辦法產生所以沒辦法 GDB。

### 參考資料

[MariaDB Error：1932 Table doesn't exist in engine 的解决方法_u014461454的博客-CSDN博客](https://blog.csdn.net/hawht/article/details/84246261)

[MySQL/MariaDB已被锁表运行中热复制为副本/innodb表错误Table xx doesn't exist in engine处理](https://blog.path8.net/archives/7608.html)

[How does InnoDB store tables in ibdata file?](https://dba.stackexchange.com/questions/62989/how-does-innodb-store-tables-in-ibdata-file)