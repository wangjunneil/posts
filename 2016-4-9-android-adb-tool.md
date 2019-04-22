---
title: 在android中adb命令行工具的简单使用和常用命令
name: android-adb-tool
date: 2016-4-9
tags: [android,adb]
categories: [Mobile]
---

* 目录
{:toc}

## 常规命令明细

```shell
# 手机截屏
$ adb shell screencap -p /sdcard/screen.png

# 下载到PC
$ adb pull /sdcard/screen.png
```

## ADB 远程/网络调试

1. 手机和电脑在同一网络环境下，且手机开始"USB Debug"模式

![usb_debug](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190320/abd-usbdebug.png)

2. 手机启用远程调试的两种方法

+ **安装远程adb调试工具**

![network_adb](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190320/adb-remote.png)

+ **在已root的手机上命令行开启**

打开手机终端terminal，执行以下命令：

```shell
$ su
$ setprop service.adb.tcp.port 5555
$ adbd
```

3. 在电脑中执行远程连接

```shell
$ adb connect 192.168.1.10 5555
```
