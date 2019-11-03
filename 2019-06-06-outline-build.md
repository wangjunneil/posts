---
title: 旅行必备 Outline 搭建与使用
name: outline-build
date: 2019-06-06
tags: [outline,ssr,出国,翻墙]
categories: [Other]
---

* 目录
{:toc}

## 1. 简介

Outline 基于ss的开源软件，出自于Jigsaw公司，Jigsaw公司前身是Google Ideas，是Google旗下的技术孵化器公司，目标是以技术来克服全球的网络安全难题，
包括地址网络审查制度、降低网络攻击的威胁，以及防止大众收到网络骚扰等。Outline即为Jigsaw的成果之一。


## 2. 创建实例

参考 [旅行必备 ShadowsocketR 搭建与使用](//wangjun.dev/2019/06/ssr-build/) 中的创建实例。

## 3. 一键安装

```shell
$ sudo -i
$ curl -sS https://get.docker.com/ | sh
$ systemctl start docker
$ systemctl enable docker
$ systemctl status docker
$ wget -qO- https://raw.githubusercontent.com/Jigsaw-Code/outline-server/master/src/server_manager/install_scripts/install_server.sh | bash
```

从上述命令可见 outline 的server环境是基于 docker 来做的，首先安装 docker 容器，启动、设置开机自启动、查看启动状态，执行一键脚本安装。
执行完后会输出一段 json 格式的内容，其实就是一个连接服务端的加密串，将它复制保存到记事本中，后续需要使用。

![outline-result](//via.placeholder.com/599x255?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20190605/outline-result.png"}

| 这里需要注意的是端口号是否在防火墙规则中

## 4. outline manager

![outline-manager](//via.placeholder.com/599x740?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20190605/outline-manager.png"}

点击下载 [outline manager](https://github.com/Jigsaw-Code/outline-server/releases) 并安装，将上面输出的 json 加密串配置在 outline manager 中，
启动并点击钥匙按钮获取连接key，类似如下：

```
ss://Y2hhY2hhMjAtaWV0Zi1wb2x5MTMwNTpETVdTQUlSbFN6dWc=@34.92.208.222:54320/?outline=1
```

## 5. outline client

点击下载 [outline client](https://github.com/Jigsaw-Code/outline-client/releases)，根据自己系统自行按照，使用上一步的key添加连接，保存并启动即可。

![outline-client](//via.placeholder.com/527x762?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20190605/outline-client.png"}


