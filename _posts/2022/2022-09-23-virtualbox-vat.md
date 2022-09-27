---
layout: post
title: "VirtualBox VAT模式下使用SSH连接虚拟机"
date: 2022-09-23 15:09:00 +0800
categories: tech
---

## 由来

最近一段时间整虚拟机，使用开源的VirtualBox创建的虚拟机。安装完虚拟机后，发现进入虚拟机后，复制粘贴功能不可用，主机和虚拟机切换还需要输入指令，用起来不是很方便，遂想用ssh工具连接虚拟机。

## 过程

虚拟机网络配置一般有三种模式：VAT模式，桥接模式和Host-Only模式。

- VAT模式，相当于主机作为路由器建立子网，虚拟机接入这个网络，实现网络连接。一般这个网络配置已经可以满足大部分的使用需求。VirtualBox的虚拟机默认使用VAT模式。
- 桥接模式，相当于主机做网桥，虚拟机和主机接入到同一网络，实现网络连接。这个网络配置可以提供更高级的网络使用需求。
- Host-Only模式，这个将主机和虚拟机组建一个独立的网络，不能访问外部连接。
  由于我的办公电脑会经常切换网络，所以使用了VAT模式给虚拟机配置确定的ip访问。但是发现VAT模式下虚拟机可以访问主机，但是主机无法访问到虚拟机。看了一下VirtualBox的配置手册，发现有这样一段话：

![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/20220922150610.png)

简单来讲就是虚拟机接入了Virtual创建的私有网络，这个私有网络是对外部不可访问的。但是可以通过"端口转发"这个配置来让外部可访问。这实际上是Virtual会监听这个端口并且转发所有的数据包到虚拟机上。
关于配置端口转发提供了两种方式，可视化界面配置和基于命令配置。关于命令配置又给了以下的样例：

![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/20220923000342.png)

这是一个ssh转发的配置样例，之后有提到虚拟机ip是内建DHCP服务器自动分配的时候是不需要配置虚拟机ip，但如果是固定分配了ip，虚拟机的ip就一定要配置。

## 最终网络配置

1. 使用可视化配置，在虚拟机网络配置的端口配置创建如下端口转发规则TCP 1022=>22

![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/20220922232423.png)

![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/20220922234129.png)

2. SSH工具添加session连接，配置host填1270.0.0.1，port填1022

![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/20220922234209.png)

然后测试连接，就可以啦！

![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/20220923001104.png)
