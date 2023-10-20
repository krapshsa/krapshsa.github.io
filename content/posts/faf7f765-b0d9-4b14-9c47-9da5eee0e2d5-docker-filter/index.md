---
title: "煩人的 docker 輸出與 filter"
date: "2023-10-13T03:44:39Z"
tags: [docker]
---

# 煩人的 docker 輸出與 filter

## 起因

要列出 Docker Container 中用到的 Mount，只能自己想辦法把 `Mounts` 區段擷取出來。

`docker inspect <container>` 預設輸出是 `JSON` 格式，所以要用 `jq` 進行處理，例如：

```Bash
docker inspect 444938fa3467 | jq .[0].Mounts
```

當然你也可以用 go template 格式輸出，只是很醜最後還是要轉成 json 再 `jq` 整理：

```Bash
docker inspect --format '{{json .Mounts}}' 444938fa3467 | jq .
```

{{< br >}}

{{< br >}}

而列出 images 的時候又有 filter 可以用，但是 format 不是 JSON 例如：

```Bash
docker images --filter "dangling=true"
```

想要串接 shell 指令刪除的時候可以：

```Bash
docker images --filter "dangling=true" --format "{{.Repository}}:{{.Tag}}" | xargs docker rmi
```

當然土炮一點也是可以，就是要把顯示欄位的 header 去掉：

```Bash
docker images | tail -n +2 | awk '{print $1":"$2}'  | xargs docker rmi
```

{{< br >}}

所以 Go Template 與 `jq` 兩種處理輸出的方法都要會，在此記錄一下現在有用到的處理，之後可能慢慢增加。

我個人是覺得 json 格式好用一點，因為我輸出一次之後就可以自己拆解內容，而 Go Template 則是要參考官方文件有那些 Key 可以用。

{{< br >}}

## Go Template

### 格式化列出 images

```Bash
docker images --format "{{.Repository}}:{{.Tag}}"
```

```Bash
docker images --filter "dangling=true" --format "{{.Repository}}:{{.Tag}}"
```

```Bash
docker images --filter "label=version=latest"  --format "{{.Repository}}:{{.Tag}}"
```

{{< br >}}

## `jq`

### 輸出全部容器的 Mount (格式化成名稱 + Mount)

```Bash
docker ps -a | xargs docker inspect | jq 'map({Name: .Name, Mounts: .Mounts})'
```

{{< br >}}

### 輸出容器的網路資訊

我發現會有好幾個地方有 IP 資訊，沒辦法再更近一步 filter

```Bash
docker ps -a | xargs docker inspect | jq 'map({Name: .Name, Networks: .NetworkSettings.Networks})'
```