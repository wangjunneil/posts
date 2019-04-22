---
title: 怎样快速创建一个chrome扩展并进行发布
name: how-to-create-chrome-extension
date: 2018-03-18
tags: [chrome, extension, manifest]
categories: [Other]
---

* 目录
{:toc}

---

如果你需要编写**chrome extension**，这篇文章将是很好的入门，若需要更加深入的了解和编写扩展程序，请参看[Google官方文档](https://developer.chrome.com/extensions/overview)。本篇文章将会创建一个简单的扩展程序，完成扩展基本编写及发布的过程。

## 1. HelloWorld

### 1.1 程序结构

```
|-- manifest.json
|-- icon.png
|-- hello.html
```

> 其中**icon.png**图标为16x16像素的即可

### 1.2 manifest文件

**manifest.json**文件主要告知chrome浏览器插件的重要信息，例如名称、权限等。下面是最基本的描述信息，更全的**manifes.jsont**配置明细，请参看[Manifest File Format](//developer.chrome.com/extensions/manifest)。

```json
{
    "name": "Hello Extensions",
    "description" : "Base Level Extension",
    "version": "1.0",
    "manifest_version": 2,
    "browser_action": {
        "default_popup": "hello.html",
        "default_icon": "hello_extensions.png"
    }
}
```



### 1.3 hello.html文件

**hello.html**是一个基本的html结构的文件，可以任意添加UI元素或者事件（某些事件需要在manifest.json中声明权限）。

```html
<html>
    <body>
        <h1>Hello Extensions</h1>
    </body>
</html>
```

### 1.4 插件安装调试

+ 打开chrome浏览器，地址栏输入`chrome://extensions`打开扩展列表

![chrome_extension_1](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/chrome_extension_1.png)

+ 勾选右上角的**Developer Mode**

![chrome_extension_1](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/chrome_extension_2.png)

+ 点击**Load Unpacked Extension**按钮加载到程序目录即可

![chrome_extension_1](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/chrome_extension_3.png)

+ 点击Chrome浏览器扩展栏即可实现基本的程序运行

![chrome_extension_1](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/chrome_extension_4.png)
