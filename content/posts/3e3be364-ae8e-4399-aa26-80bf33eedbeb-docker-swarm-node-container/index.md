---
title: "Docker Swarm 跨 Node 的 Container 彼此無法通信"
date: "2023-09-22T06:35:00Z"
tags: [docker]
---

# Docker Swarm 跨 Node 的 Container 彼此無法通信

## 起因

幫客戶升級 OS (Oracle 8) 後發現 docker swarm mode 下，
redis slave 連線不到另外一台的 redis master，導致無法建成 cluster。

錯誤訊息如下：

```PHP
redis 02:36:11.14
redis 02:36:11.15 Welcome to the Bitnami redis container
redis 02:36:11.15 Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-redis
redis 02:36:11.24 Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-redis/issues
redis 02:36:11.24
redis 02:36:11.24 INFO  ==> ** Starting Redis setup **
redis 02:36:11.36 WARN  ==> You set the environment variable ALLOW_EMPTY_PASSWORD=yes. For safety reasons, do not use this flag in a production environment.
redis 02:36:11.37 INFO  ==> Initializing Redis
redis 02:36:11.56 INFO  ==> Setting Redis config file
redis 02:36:11.86 INFO  ==> Configuring replication mode
timeout reached before the port went into state "inuse"
```

{{< br >}}

## 原因

VMWare + RedHat 相關 OS + `vmxnet3` 會 drops packets

{{< br >}}

## 解法

### **解法一 (Application 解法)：**

`ens160` 是我這台的網卡，請依照自己的情況換掉

```Bash
ethtool -K ens160 tx-checksum-ip-generic off
```

官方說是 workaround，而且重開機就失效，需要設定 `rc.local`。

我認為把功能關掉很怪，但跟下面兩個方法比起來好像比較好一點。

{{< br >}}

### **解法二 (Kernel 解法)：**

有的版本 RedHat 宣稱 ok 了但是還是不行。

缺點是更新就有可能遭殃，這樣每次更新都要測試記錄這個版本行不行。

不確定是不是只有 Kernel 版本有影響，有可能其他模組也有可能影響？

{{< br >}}

### **解法三 (VMWare 解法)：**

把網卡設定為 e1000 / e1000e

![](esxi-5132c5ce-4aea-4277-9ba0-68ccf589b24f.png)

但是這樣網卡的速度就不夠快了(VMXNET 3 是 10Gbit)。

{{< br >}}

## References

### **RHEL 官方 Issue**

[RHEL8: TCP encapsulated in UDP tunnel is dropped on vmxnet3 adapters](https://access.redhat.com/solutions/5881451)

### **Docker 官方 Issue**

[Upgrade to 20.10 breaks swarm network #41775](https://github.com/moby/moby/issues/41775)