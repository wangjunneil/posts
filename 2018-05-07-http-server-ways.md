---
title: 快速创建简易HTTP服务器的方式
name: http-server-ways
date: 2018-05-07
tags: [http,http-server,http服务器,python,ruby,php,npm]
categories: [Linux]
---

* 目录
{:toc}

## 1. Python 方式

```shell
$ python2 -m SimpleHTTPServer 8000
$ python3 -m http.server 8000
```

## 2. Php 方式

```shell
$ php -S 0.0.0.0:80000
```

## 3. Ruby 方式

```shell
$ ruby -rwebrick -e "WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => Dir.pwd).start"
```

## 4. NPM 方式

```shell
# install
$ npm i -g http-server
$ http-server -p8080 -c-1 dist/

# https
$ http-server -S —p433 C localhost.pem -K localhost-key.pem dist/
```

