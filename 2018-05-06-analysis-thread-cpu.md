---
title: Linux分析Java进程中哪个线程占用了大量的CPU
name: analysis-thread-cpu
date: 2018-05-06
tags: [性能,cpu,线程分析,java进程]
categories: [Java,Linux]
---

* 目录
{:toc}

当运行在服务器中的 Java 程序越来越慢，`top` 查看 CPU 的使用率非常高，确定是某个 Java 进程在占用过多的 CPU，此时定位是哪部分代码或者业务导致。

主要分为以下几个步骤实现：

**1. 使用 `top -H -p [pid]` 命令列出进程的所有线程**

![cpu_1](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/cpu_1.png)

通过上述命令，这里假定发现ID为"**5171**"的线程占用了 75% 的CPU

**2. 使用计算器将十进制的线程ID"5171"转换成16进制**

这里计算后的结果是 "**0x1433**"

 ![cpu_2](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/cpu_2.png)

 **3. 使用命令 `jstack [pid] > jstack.txt` 导出进程中所有线程上下文**

 ![cpu_3](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/cpu_3.png)

 **4. 使用命令 `cat jstack.txt | grep -i 1433` 查找匹配的线程ID**

 ![cpu_4](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/cpu_4.png)

 从上述输出的信息可以直接得出执行的堆栈，从而确定哪部分代码出现了问题




