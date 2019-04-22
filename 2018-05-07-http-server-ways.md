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
python2 -m SimpleHTTPServer 8000
python3 -m http.server 8000
```



## 2. Php 方式

```shell
php -S 0.0.0.0:80000
```



## 3. Ruby 方式

```shell
ruby -rwebrick -e \ 
"WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => Dir.pwd).start"
```



## 4. Npm 方式

可以全局安装`npm install http-server -g `，也可以使用`npm install`方式安装

```json
"scripts": {
	"start": "http-server -c-1 -p 8081"
},
"devDependencies": {
	"http-server": "^0.11.1"
}
```

