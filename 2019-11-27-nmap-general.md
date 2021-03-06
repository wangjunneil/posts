---
title: Nmap 常规使用方法
name: nmap-general
date: 2019-11-27
tags: [扫描,漏洞]
categories: [Blackhat]
---

* 目录
{:toc}


## 介绍

[nmap](//nmap.org/) 是一款用于网络扫描的安全审计工具，提供终端和GUI两种方式的工具，使用参数复杂，本文只整理常用的扫描方法。

## 使用

```shell
# ping扫描存活的主机，不做探测（端口或操作系统）
$ nmap -sP 192.168.199.0/24

# 半开放扫描，-sS选项不需要3次握手，使用频率最高的扫描选项：SYN扫描。
# 又称为半开放扫描，它不打开一个完全的TCP连接，执行得很快，效率高
$ nmap -sS 192.168.199.0/24

# 伪装源地址进行扫描，使用-S伪装自己源地址进行扫描的话，必须另外使用-e指定网卡和-Pn参数才能伪装
$ nmap -e eth0 10.0.1.161 -S 10.0.1.167 -Pn

# 综合扫描
$ nmap -O 192.168.1.128
$ nmap -A 192.168.1.128

# 扫描目标主机基本漏洞
$ nmap -script=vuln 192.168.1.128
```