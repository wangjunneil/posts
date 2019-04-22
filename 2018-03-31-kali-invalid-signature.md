---
title: 使用 apt-get update 签名错误 invalid signature 的处理
name: kali-invalid-signature
date: 2018-03-31
tags: [kali, apt-get, signature]
categories: [Blackat, Linux]
---

* 目录
{:toc}

---

在kali中更新执行"apt-get update"出现"**Invalid signature for Kali Linux repositories**"的解决方法

```shell
$ wget -q -O - https://archive.kali.org/archive-key.asc | apt-key add
```
