---
title: Linux个人常用命令汇总收集（不断更新）
name: linux-skill-gather
date: 2017-07-23
tags: [linux,command]
categories: [Linux]
---

* 目录
{:toc}

## 显示当前目录下文件大小

```shell
du -sh * | sort -nr 
```

## 列出子目录及大小

```shell
du -h -d 1
```

## 码表

```shell
time read (ctrl-d to stop)
```

## ascii词典

```shell
man ascii
```

## 在linux上关闭windows系统

```shell
net rpc shutdown -I ipAddressOfWindowsPC -U username%password
```

## 显示前10内存占用的进程

```shell
ps aux | sort -nk +4 | tail
```

## base64编解码

```shell
echo 'HelloWorld' | base64
bash64 -d <<< 'SGVsbG9Xb3JsZAo='
```

## 查看端口被谁占用

```shell
lsof -i:8080
```

## 递归删除指定文件

```shell
find . -name "*.bak" -exec rm {} \;
find . -name "*.c" | xargs rm -rf;
```

## 内存cache占用大

使用`free -m`时会发现内存的**cache**段缓存的过多，导致内存不够用，可以使用以下命令进行清空。

```shell
shell> free -m
             total       used       free     shared    buffers     cached
Mem:         15888      15226        661        248        624       5123
-/+ buffers/cache:       9479       6408
Swap:          499          0        499
```

```shell
sync;
sync;
sync;
echo 3 > /proc/sys/vm/drop_caches
```

> 此命令有风险，会破坏内存中存储的数据，尤其不要在Oracle服务器中使用

## 批量查找文件并替换内容

```shell
find -name '要查找的文件名' | xargs perl -pi -e 's|被替换的字符串|替换后的字符串|g'
```

