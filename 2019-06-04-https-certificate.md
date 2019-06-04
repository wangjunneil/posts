---
title: 开源免费的 SSL 证书汇总
name: https-certificate
date: 2019-06-04
tags: [ssl,https]
categories: [Linux, Other]
---

* 目录
{:toc}

## 1. Google Mkcert

Go语言编写，用于生成本地HTTPS证书的工具，官网[地址](//github.com/FiloSottile/mkcert)。

### 1.1. 安装

```shell
# Mac方式安装，其他平台见官网
$ brew install mkcert

# 如果用的是firefox还需安装nss
$ brew install nss
```

### 1.2. 生成

根证书生成后会在Mac系统钥匙串应用下的证书项中，存在 **mkcert** 的已安装的证书

```shell
# 生成根证书（只需生成一次）
$ mkcert -install

# 生成对应域名证书，可生成多个，也支持泛域名
$ mkcert localhost 127.0.0.1

Created a new certificate valid for the following names
 - "localhost"
 - "127.0.0.1"

The certificate is at "./localhost+2.pem" and the key at "./localhost+2-key.pem"
```

### 1.3. 使用

```
# nginx
ssl_certificate     /etc/nginx/ssl/localhost+2.pem;
ssl_certificate_key /etc/nginx/ssl/localhost+2-key.pem;

# jekyll
$ sudo jekyll serve --ssl-cert ./localhost+2.pem --ssl-key ./localhost+2-key.pem --port 443

# http-server
$ http-server -p 443 --ssl --cert ./localhost+2.pem --key ./localhost+2-key.pem
```

移动端使用，IOS使用safari下载 **rootCA.pem** 并安装，在设置里确认并信任此证书；Android安装比较特殊，可[参考](//stackoverflow.com/questions/4461360/how-to-install-trusted-ca-certificate-on-android-device/22040887#22040887)。

## 2. Let’s Encrypt

Let’s Encrypt 是由 Mozilla 发起，诸多 CDN 厂商，浏览器厂商共同支持的免费 SSL 项目。Let’s Encrypt 使用一个命令行工具
[certbot](https://certbot.eff.org/) 来自动完成 CSR 生成，发起申请，DCV (Domain Control Validation) 验证，获取证
书的工作，基本上一行命令就搞定。

### 2.1. 安装

这里以 Debain9 和 nginx 为例子

```shell
$ sudo apt-get install certbot python-certbot-nginx -t stretch-backports
```

### 2.1. 使用

```shell
# 证书全自动安装
$ sudo certbot --nginx

# 需要有所定制则使用下面命令安装
$ sudo certbot --nginx certonly
```

### 2.2. 续订

Let的加密证书持续90天，可以使用计划任务续订证书有效期

```shell
$ sudo certbot renew --dry-run
```
