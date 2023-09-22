---
title: "docker swarm 環境設定 firewalld"
date: "2022-06-22T08:47:00Z"
tags: [docker]
---

# docker swarm 環境設定 firewalld

## 起因

{{< br >}}

## 事前準備

```Bash
sudo iptables -t filter -F
sudo iptables -t filter -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo systemctl restart docker
```

{{< br >}}

## 單台

```Bash
sudo firewall-cmd --permanent --direct --remove-chain ipv4 filter DOCKER-USER
sudo firewall-cmd --permanent --direct --remove-rules ipv4 filter DOCKER-USER
sudo firewall-cmd --permanent --direct --add-chain ipv4 filter DOCKER-USER
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -p tcp -m multiport --dports 3306,24224 -s 127.0.0.1/32 -j ACCEPT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -p tcp -m multiport --dports 3306,24224 -j REJECT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 1 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 1 -j RETURN -s 172.16.70.0/24
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 1 -p tcp -m multiport --dports 80,443 -j ACCEPT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 10 -j REJECT
```

`/etc/firewalld/direct.xml`

```XML
<?xml version="1.0" encoding="utf-8"?>
<direct>
  <chain ipv="ipv4" table="filter" chain="DOCKER-USER"/>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="0">-p tcp -m multiport --dports 3306,24224 -s 127.0.0.1/32 -j ACCEPT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="0">-p tcp -m multiport --dports 3306,24224 -j REJECT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="1">-m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="1">-j RETURN -s 172.16.70.0/24</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="1">-p tcp -m multiport --dports 80,443 -j ACCEPT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="10">-j REJECT</rule>
</direct>
```

{{< br >}}

## 多台

```Bash
sudo firewall-cmd --permanent --direct --remove-chain ipv4 filter DOCKER-USER
sudo firewall-cmd --permanent --direct --remove-rules ipv4 filter DOCKER-USER
sudo firewall-cmd --permanent --direct --add-chain ipv4 filter DOCKER-USER
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -p tcp -m multiport --dports 2376,2377,7946,3306,24224 -s 127.0.0.1/32,172.16.21.162/32,172.16.21.163/32,172.16.21.164/32 -j ACCEPT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -p udp -m multiport --dports 4789,7946 -s 127.0.0.1/32,172.16.21.162/32,172.16.21.163/32,172.16.21.164/32 -j ACCEPT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -p tcp -m multiport --dports 3306,24224,2376,2377,7946 -j REJECT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -p udp -m multiport --dports 4789,7946 -j REJECT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 1 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 1 -j RETURN -s 172.16.70.0/24
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 1 -p tcp -m multiport --dports 80,443 -j ACCEPT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 10 -j REJECT
```

`/etc/firewalld/direct.xml`

```XML
<?xml version="1.0" encoding="utf-8"?>
<direct>
  <chain ipv="ipv4" table="filter" chain="DOCKER-USER"/>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="0">-p tcp -m multiport --dports 2376,2377,7946,3306,24224 -s 127.0.0.1/32,172.16.21.162/32,172.16.21.163/32,172.16.21.164/32 -j ACCEPT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="0">-p udp -m multiport --dports 4789,7946 -s 127.0.0.1/32,172.16.21.162/32,172.16.21.163/32,172.16.21.164/32 -j ACCEPT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="0">-p tcp -m multiport --dports 3306,24224,2376,2377,7946 -j REJECT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="0">-p udp -m multiport --dports 4789,7946 -j REJECT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="1">-m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="1">-j RETURN -s 172.16.70.0/24</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="1">-p tcp -m multiport --dports 80,443 -j ACCEPT</rule>
  <rule ipv="ipv4" table="filter" chain="DOCKER-USER" priority="10">-j REJECT</rule>
</direct>
```

{{< br >}}

## 重啟

```Bash
sudo firewall-cmd --reload
```

{{< br >}}

## 持久化

```Bash
sudo cat /etc/firewalld/direct.xml
```

{{< br >}}

## 參考資料

[使用 Firewalld 保护 Docker 端口](https://zhuanlan.zhihu.com/p/371683318)

[如何正確給 Docker 配置 firewalld，以及 Docker 強行覆蓋 iptables 的安全隱患](https://holywhite.com/archives/489)

[iptables multiple source IPs in single rule](https://serverfault.com/questions/6989/iptables-multiple-source-ips-in-single-rule)