---
title: 私有知识库 confluence 以 docker 为容器的安装
name: confluence-build
date: 2019-11-20
tags: [wiki,confluence]
categories: [Other,Linux]
---

* 目录
{:toc}


## 1. 背景介绍

没有特殊原因，只想什么都用自己的，数据保存在自己这，做好维护，永不丢失，**代代传承**。几乎用遍所有笔记、wiki、博客，
始终在不断迁移，不停在寻找一个真正合适自己的记录的地方，最终还是决定使用 [confluence](//www.atlassian.com/software/confluence)，尽管相对于个人过于庞大、复杂，
但经过研究后发现可以满足所有需求，本文将介绍如何使用 docker 安装部署一套属于自己的知识库管理系统。

> 这里需要说明一下，为什么使用docker方式，简单、高效、隔离性好，已具备基本环境，所以选择


## 2. 安装步骤

提前安装好 docker 环境，不同系统安装步骤网上搜索，以下步骤是已经安装好的情况下进行

### 2.1 Postgresql镜像拉取与启动

这里使用的是 postgresql 数据库做为数据存储，也可以使用其他，如mariadb等等，麻烦的是需要自行下载驱动文件并上传到confluence目录中，所以这里使用 postgresql

```shell
# 拉取
$ docker pull postgres:latest

# 运行
$ docker run -d \
  --name postgresdb \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=123456 \
  postgres
```

### 2.2 为confluence准备数据库

```shell
# 进入 postgresql 容器
$ docker exec -it postgresdb /bin/bash
# 登录 postgresql 数据库系统
$ psql -U postgres
# 创建一个空的 confluence 数据库
$ CREATE DATABASE confluence WITH OWNER postgres;
# 退出数据库
$ \q
# 退出容器
$ exit
```

### 2.3 Confluence镜像拉取与启动

docker 仓库中有很多 confluence 的镜像，不同镜像之间可能因为版本不同，建议使用如下的镜像以保证后续步骤的正常

```shell
# 拉取
$ docker pull cptactionhank/atlassian-confluence:latest

# 运行
$ docker run -d \
  --name confluence \
  -p 58093:8090 \
  --link postgresdb:db \
  --user root:root \
  cptactionhank/atlassian-confluence:latest
```

> confluence配置了**58093**做为外部映射端口，并连接上面已经启动好的postgresql容器，别名用db，且以root的方式运行

## 3. 界面配置

上述操作完成后，通过浏览器打开 http://192.168.1.100:58093，注意这里的 192.168.1.100 是 docker 容器地址，如果是本地则使用 127.0.0.1，
端口58093是启动 confluence 容器时指定的对外接口，打开后在语言栏切换成中文后，点击下一步

![confluence-1](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-1.webp"}

这两个应用对于个人来说用不上，占硬盘和内存，不选择直接点击下一步

![confluence-2](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-2.webp"}

复制**服务器ID**到记事本上，后续需要使用，不需要点击下一步，**直接关闭浏览器开始破解操作**

![confluence-3](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-3.webp"}

## 4. 破解步骤

### 4.1 获取破解文件

从 confluence 容器中将 *atlassian-extras-decoder-v2-3.4.1.jar* 文件拷贝出来并改名为 *atlassian-extras-2.4.jar*

```shell
# 从容器拷贝出来
$ docker cp confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar ./
# 重命名
$ mv atlassian-extras-decoder-v2-3.4.1.jar atlassian-extras-2.4.jar
```

### 4.2 运行破解程序

先下载破解文件，[点击下载](//cigorsica.com/1oBj)，使用命令`java -jar confluence_keygen.jar`运行，这里需要java运行环境，
输入刚才记录的**服务器ID**，点击**.patch!**按钮，选择之前改名的 *atlassian-extras-2.4.jar* 文件，提示"Jar successfully patched"表示
补丁打成功。

![confluence-4](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-4.webp"}

再点击**.gen!**按钮生成密钥信息并拷贝记录起来，后面要使用

![confluence-5](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-5.webp"}

### 4.3 上传破解补丁

将打好破解补丁的文件 *atlassian-extras-2.4.jar* 重新改名为 *atlassian-extras-decoder-v2-3.4.1.jar* 并上传覆盖到 confluence 容器原有的位置中

```shell
# 重新命名
$ mv atlassian-extras-2.4.jar atlassian-extras-decoder-v2-3.4.1.jar
# 上传覆盖
$ docker cp atlassian-extras-decoder-v2-3.4.1.jar confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/
```

### 4.4 重启confluence容器

重新启动 confluence 容器，以让 confluence 重新加载已经破解的补丁文件，否则内存中使用的还是未破解的

```shell
# 重启容器
$ docker restart confluence
```

## 5. 继续配置

confluence 容器重启好后重新打开界面控制台将会直接跳到输入密钥内容的界面，粘贴上面拷贝的密钥内容后点击下一步开始设置数据库

![confluence-6](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-6.webp"}

![confluence-7](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-7.webp"}

![confluence-8](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-8.webp"}

设置使用通过连接字符串连接数据库，内容填写"jdbc:postgresql://db:5432/confluence"，输入用户名密码，点击测试连接，成功后点击下一步，开始
刷入 confluence 表结构，稍等一会就会出现以下界面表示安装成功

首次使用的话推荐选择"示范站点"，包含了许多demo的文章供编写参考，有洁癖的话使用"空白站点"，这样更纯净些

![confluence-9](//via.placeholder.com/594x485?text=""){: data-src="//vinnycc.oss-cn-shanghai.aliyuncs.com/20191120/confluence-9.webp"}

配置Confluence中的管理用户和组，就是管理员用户名和密码等信息，完成后显示设置成功

## 附录：安装遇到的问题

在设置数据库连接后点击下一步时可能会超时，超时后重新刷新界面出现"Spring Application context has not been set"的错误，
不要怀疑，直接到 confluence 容器中删除文件 confluence.cfg.xml 后，重新启动，将会再次进入第一步的配置界面，密钥可以使用
之前保存的，继续步骤就可以成功

```shell
# 删除 confluence.cfg.xml 文件
$ docker exec confluence rm -rf /var/atlassian/confluence/confluence.cfg.xml
# 重启 confluence 容器
$ docker restart confluence
```
