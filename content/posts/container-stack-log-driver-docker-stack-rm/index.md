---
title: "使用 container 在同一個 stack 部屬 log driver 時，docker stack rm 經常性失敗 "
date: "2022-07-28T08:19:00Z"
tags: [docker]
---

# 使用 container 在同一個 stack 部屬 log driver 時，docker stack rm 經常性失敗 

## 起因

最近把 `fluentd` 加入到 docker swarm 中當作 Log Driver 的時候，

移除 docker stack 時，關閉得比較慢的 service 有機會沒辦法完整移除。

此時 `docker inspect` ， `docker rm` 等指令都會卡住。

唯一的解法就是重啟 docker daemon。

猜測因為沒辦法保證 `fluentd` 在最後一個關閉導致出錯了。

{{< br >}}

## 重製步驟

1. 移除 docker stack

        docker stack rm mystack
2. 檢查 service 都移除掉了

        docker service ls
    ID        NAME      MODE      REPLICAS   IMAGE     PORTS
3. 發現還有 container 沒有正常關閉，且永遠卡在這個狀態

        docker stack ps mystack --no-trunc
    ID                          NAME                          IMAGE                        NODE                    DESIRED STATE   CURRENT STATE                ERROR     PORTS
    5fllkwthpur60gzgeqq4cow48   i1i65yn2b35jqsfqk7gb9y7l8.1   my-clamav:2022-0406-145048   localhost.localdomain   Remove          Running about a minute ago

{{< br >}}

檢查系統的 Log，有出現如下訊息 (我是 CentOS)：

    sudo cat /var/log/messages | grep 3c902d2962f7
    Jul 28 10:48:49 localhost dockerd[1265564]: time="2022-07-28T10:48:49.917640954+08:00" level=info msg="Configured log driver does not support reads, enabling local file cache for container logs" container=3c902d2962f7fb6af8ce5a16a965bd2707383dbb18fed9b6942e1687f6419c4b driver=fluentd
    Jul 28 10:48:49 localhost containerd[1161]: time="2022-07-28T10:48:49.936470840+08:00" level=info msg="starting signal loop" namespace=moby path=/run/containerd/io.containerd.runtime.v2.task/moby/3c902d2962f7fb6af8ce5a16a965bd2707383dbb18fed9b6942e1687f6419c4b pid=1269465 runtime=io.containerd.runc.v2
    Jul 28 10:53:00 localhost dockerd[1265564]: time="2022-07-28T10:53:00.719182463+08:00" level=info msg="ignoring event" container=3c902d2962f7fb6af8ce5a16a965bd2707383dbb18fed9b6942e1687f6419c4b module=libcontainerd namespace=moby topic=/tasks/delete type="*events.TaskDelete"
    Jul 28 10:53:00 localhost containerd[1161]: time="2022-07-28T10:53:00.719579189+08:00" level=info msg="shim disconnected" id=3c902d2962f7fb6af8ce5a16a965bd2707383dbb18fed9b6942e1687f6419c4b
    Jul 28 10:53:00 localhost containerd[1161]: time="2022-07-28T10:53:00.719667935+08:00" level=warning msg="cleaning up after shim disconnected" id=3c902d2962f7fb6af8ce5a16a965bd2707383dbb18fed9b6942e1687f6419c4b namespace=moby
    Jul 28 10:53:07 localhost dockerd[1265564]: time="2022-07-28T10:53:07.730207608+08:00" level=info msg="Container failed to exit within 10s of signal 15 - using the force" container=3c902d2962f7fb6af8ce5a16a965bd2707383dbb18fed9b6942e1687f6419c4b
    Jul 28 10:57:07 localhost dockerd[1271313]: time="2022-07-28T10:57:07.971173522+08:00" level=info msg="Removing stale sandbox c540d42e6cba844695f8cb780f5c28a82a59500207ace2b6526d11eca5b54600 (3c902d2962f7fb6af8ce5a16a965bd2707383dbb18fed9b6942e1687f6419c4b)"
    Jul 28 11:12:19 localhost dockerd[1271313]: time="2022-07-28T11:12:19.902248293+08:00" level=error msg="Error getting service 3c902d2962f7: service 3c902d2962f7 not found"
    Jul 28 11:12:19 localhost dockerd[1271313]: time="2022-07-28T11:12:19.905022331+08:00" level=error msg="Error getting task 3c902d2962f7: task 3c902d2962f7 not found"
    Jul 28 11:12:19 localhost dockerd[1271313]: time="2022-07-28T11:12:19.906474336+08:00" level=error msg="Error getting node 3c902d2962f7: node 3c902d2962f7 not found"

{{< br >}}

## 猜測

一開始懷疑是沒有辦法關掉 process，可能在卡在寫 Log 了。
照著這個思路，先去查詢查詢 container 相對應的 pid：

    docker inspect -f '{{.State.Pid}}' 3c902d2962f7
    1269485

    sudo pstree -a -p
    
    ...
    
    |-containerd-shim,1269465 -namespace moby -id 3c902d2962f7fb6af8ce5a16a965bd2707383dbb18fed9b6942e1687f6419c4b -address/run/
    |   |-docker-init,1269485 -- /bootstrap.sh
    |   |   |-clamd,1269536 -c /etc/clamav/clam.conf
    |   |   |   `-{clamd},1269731
    |   |   |-freshclam,1269560 -d -c 6
    |   |   |   `-{freshclam},1269732
    |   |   `-gunicorn,1269558 /usr/bin/gunicorn --workers=2 --threads=2 --bind 0.0.0.0:8000 --daemon wsgi:app
    |   |       |-gunicorn,1269561 /usr/bin/gunicorn --workers=2 --threads=2 --bind 0.0.0.0:8000 --daemon wsgi:app
    |   |       `-gunicorn,1269562 /usr/bin/gunicorn --workers=2 --threads=2 --bind 0.0.0.0:8000 --daemon wsgi:app
    
    ...

接著移除掉整個 stack。

但是發現儘管 `docker ps` 看到 container 還在，但 pid 已經找不到了。

{{< br >}}

## Workaround

看起來這個 container 沒有施力點，只好試試看手動調整關閉 Service 的順序

最後寫成腳本：

    function get_services() {
      services=$(docker service ls --format "{{.Name}}" | sed -E 's/mystack_//')
      echo "$services"
    }
    
    function down() {
      stack_prefix="mystack_"
      last_service="fluentd"
      services=$(get_services)
      services=${services[*]/$last_service}
    
      for service in $services
      do
        docker service rm "$stack_prefix$service" > /dev/null
      done
    
      # wait for all containers removed.
      for i in {1..60}
      do
        docker ps -a | grep -v "$stack_prefix$last_service" | grep -q "$stack_prefix"
        if [ $? -eq 1 ]; then
          docker service rm "$stack_prefix$last_service" > /dev/null
          break
        fi
        sleep 1
      done
    
      if [[ "60" -eq "$i" ]]; then
        exit 1
      fi
    
      for i in {1..60}
      do
        docker ps -a | grep -q "$stack_prefix"
        if [ $? -eq 1 ]; then
          break
        fi
        sleep 1
      done
    
      if [[ "60" -eq "$i" ]]; then
        exit 1
      fi
    }
    
    down

{{< br >}}