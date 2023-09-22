---
title: "查詢 Docker Container 與 host 的 pid 對應"
date: "2023-09-21T08:16:13Z"
tags: [docker]
---

# 查詢 Docker Container 與 host 的 pid 對應

## 起因

在除錯的時候，想要用 `gdb -p` 之類的方法去接 pid 除錯，有兩種情境：

1. 已知 Container 內 pid，要用 host 的 debug 工具除錯
2. 已知 Host 的 pid 要用 Container 內的 debug 工具除錯

用了一個土炮方法，記錄起來。

{{< br >}}

## 解方

### 從 host 的 pid 取得 container 內 pid

1. 要知道是哪一個 container，假設 pid 是 `371817`
```Bash
    cat /proc/371817/cgroup
```
```Bash
    12:hugetlb:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    11:rdma:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    10:memory:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    9:cpuset:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    8:devices:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    7:freezer:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    6:blkio:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    5:pids:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    4:net_cls,net_prio:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    3:perf_event:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    2:cpu,cpuacct:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
    1:name=systemd:/docker/5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed
```

    這樣就知道是 `5796a153bb04dc86dec3348772719d3e7bf44bfa9c327e969f6b72ff878498ed` 這個容器

2. 從  `/proc` 取得 container 內的 pid
```Bash
    grep -i nspid /proc/371817/status
```
```Bash
    NSpid:	371817	8
```

    這樣就知道是 `8`

    {{< br >}}

懶惰一點：

1. 取得容器 id
```Bash
    pid=371817; cat /proc/$pid/cgroup | grep "docker" | head -n 1 | awk -F'/' '{print $3}'
```
2. 取得容器內 pid
```Bash
    pid=371817; grep -i nspid /proc/$pid/status | awk '{print $3}'
```

{{< br >}}

### 從 container 內 pid 取得 host 的 pid

可以從 docker top 取得這個容器相關的 process 在 host 上的 pid

假設 id 是 `5796a153bb04`

```Bash
    docker top 5796a153bb04
```
```Bash
    UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
    root                371763              371744              0                   Sep20               ?                   00:00:05            php-fpm: master process (/usr/local/etc/php-fpm.conf)
    webmail             371816              371763              0                   Sep20               ?                   00:00:49            php-fpm: pool www
    webmail             371817              371763              0                   Sep20               ?                   00:00:54            php-fpm: pool www
    webmail             371843              371763              0                   Sep20               ?                   00:00:49            php-fpm: pool www
```

再結合第一點去過濾出真正的 pid 是哪一個

{{< br >}}

假設容器內 pid 是 `8` 

```Bash
    docker top 5796a153bb04 | tail -n +2 | awk '{print $2}' | xargs -I {} grep -i nspid /proc/{}/status
```
```Bash
    NSpid:	371763	1
    NSpid:	371816	7
    NSpid:	371817	8
    NSpid:	371843	9
```

這樣就知道是 `371817`

{{< br >}}

懶惰一點：

```Bash
    pid=8; docker top 5796a153bb04 | tail -n +2 | awk '{print $2}' | xargs -I {} grep -i nspid /proc/{}/status | awk -v pid="$pid" '$3 == pid {print $2}'
```

{{< br >}}