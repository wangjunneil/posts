---
title: PWA 之拦截 A2HS 实现自主应用添加主屏事件
name: pwa-handle-install-prompt
date: 2019-05-29
tags: [pwa,install,prompt]
categories: [Web, Javascript, Mobile]
---

* 目录
{:toc}

![pwa-a2hs](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190530/1_hWVmsnBY6Fr6OoNvIU5pmg.png)

## 1. 概念

**A2HS** 是（Add to Home screen）的缩写，也是PWA特性之一，它主要为web应用程序提供与本机应用程序相同的用户体验优势，将PWA应用添加至桌面后，用户可以像打开普通应用程序一样使用。

![a2hs-1](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190529/a2hs-1.png)

![a2hs-2](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190529/a2hs-2.png)

## 2. 背景

默认情况下，当编写的web应用满足pwa应用最低要求（manifest.json和service-sworker）时，浏览器会在提示用户可以将当前访问的web安装到桌面上，这个行为是浏览器控制的，用户是否可以不依赖于浏览器，不必每次都弹出 A2HS 的banner，行为由用户控制，如点击按钮提示安装，本文基于此目的编写。

## 3. beforeinstallprompt

要实现上述要求，需要监听 *beforeinstallprompt* 事件并进行处理。

实现代码如下：

```js
// 定义A2HS事件对象
let installPromptEvent;

// 监听beforeinstallprompt事件，浏览器触发A2HS时会执行
window.addEventListener('beforeinstallprompt', (e) => {
  // 阻止自动提示
  e.preventDefault();
  // 储存事件对象，方便在之后的按钮事件中手动触发
  installPromptEvent = e;
});
 
// UI上的按钮点击事件
installBtn.addEventListener('click', (e) => {
	// 弹出A2HS提示
  installPromptEvent.prompt();
  // 等待用户操作结果
  installPromptEvent.userChoice
    .then((choiceResult) => {
      if (choiceResult.outcome === 'accepted') {
        console.log('用户已经接受安装');
      } else {
        console.log('用户已经取消安装');
      }
      
      // 只能用一次
      installPromptEvent = null;
    });
});
```

## 4. demo效果

![demo-a2hs](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190530/2019-05-30 00.20.10.gif)

[完整的demo代码](https://github.com/wangjunneil/exchange-pwa/tree/master/src/views)

## 5. 判断是否安装

监听安装事件

```js
window.addEventListener('appinstalled', (event) => {
	console.log('installed');
});
```

若是从主屏幕启动，则可以通过 javascript 判断

```js
if (window.matchMedia('(display-mode: standalone)').matches) {
  console.log('display-mode is standalone');
}
```

safari浏览器

```js
if (window.navigator.standalone === true) {
  console.log('display-mode is standalone');
}
```

> 这里需要注意的是，上面三个输出只会在 standalone 模式下执行

## 6. 参考链接

+ [Add to Home screen](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Add_to_home_screen)
+ [Before​Install​Prompt​Event](https://developer.mozilla.org/en-US/docs/Web/API/BeforeInstallPromptEvent)
+ [Add to Home Screen your ionic PWA](http://www.jomendez.com/2018/06/05/add-home-screen-pwas/)