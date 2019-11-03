---
title: 旅行必备 Wireguard 的搭建与使用
name: wireguard-build
date: 2019-06-07
tags: [翻墙,ssr,wireguard,出国]
categories: [Other]
---

* 目录
{:toc}

![wireguard](//via.placeholder.com/594x311?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20190610/og-logo.png"}

## 1. 介绍

[WireGuard](https://www.wireguard.com/) 是 Jason A. Donenfeld 开发的开源VPN协议。目前已经支持Linux, macOS, Android, iOS以及OpenWrt平台。WireGuard使用UDP协议传输数据，在不使用的情况下默认不会传输任何 UDP 数据包，所以比常规VPN省电很多，可以像SS一样一直挂着使用。WireGuard协议的速度瞬秒其它VPN协议。

## 2. 安装和使用

### 2.1 创建实例

参考 [旅行必备 ShadowsocketR 搭建与使用](//wangjun.dev/2019/06/ssr-build/) 中的创建实例

> 唯一区别就是系统选择Centos7

### 2.2 执行安装

打开 vm 实例的 ssh 终端，输入以下命令执行安装：

```shell
# 切换成root用户
$ sudo -i

# 安装wget工具
$ yum install -y wget

# 下载一键安装脚本
$ wget https://raw.githubusercontent.com/yobabyshark/wireguard/master/wireguard_install.sh

# 授权并执行
$ chmod +x wireguard_install.sh && ./wireguard_install.sh
```

先升级系统内核，选择1，等待升级完成后，执行`reboot`重启系统

```
1. 升级系统内核
2. 安装wireguard
3. 升级wireguard
4. 卸载wireguard
5. 显示客户端二维码
6. 增加用户
0. 退出脚
```

系统重启完成后，再次执行一键安装脚本`./wireguard_install.sh`，这次选择**安装wireguard**，等待执行完成后，会显示配置二维码和配置信息，如下：

![wireguard-result](//via.placeholder.com/487x713?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20190610/wireguard-result.png"}

> 二维码内容就是 /etc/wireguard/client.conf 文件内容

### 2.3 防火墙规则

wireguard安装完后，查看随机分配的端口号，如上面截图中的是38597，确定在防火墙规则中开放了对此端口号的出入站。

### 2.4 客户端使用

IOS与Mac客户端可以在appstore里直接下载wireguard，但需要使用美区账号。

Window端下载 [https://tunsafe.com/download](https://tunsafe.com/download)