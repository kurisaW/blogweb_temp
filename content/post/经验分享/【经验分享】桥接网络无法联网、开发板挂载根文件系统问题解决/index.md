---
title: 总结：开发板挂载根文件系统遇到的一些问题
description: 桥接网络、开发板端测
slug: 经验分享
date: 2022-03-30 00:00:00+0000
image: cover.jpg
categories:
    - experience sharing
tags:
    - experience
---



## 一、桥接网络

#### 1、简介

是指需手动配置虚拟机的IP地址（IP地址可自定义，但要和主机在同一个网段下）子网掩码，网关，此时虚拟机相当于局域网的另一台电脑，占用一个IP地址

#### 注意避坑：

如果你的虚拟机选择了桥接模式，那么建议最好是不要使用校园网，因为一般校园网会需要验证登录，但是在虚拟机中好像并不会弹出登录界面（个人理解），因此你的网络在虚拟机中是无法运行的。



#### 2、解决办法：

<1>选择直接使用网线连接到电脑，然后在虚拟机中桥接选择自己对应的网卡即可，博主自己是没有连接网线的，所以我自己是没有采取这个办法的。

![](https://img-blog.csdnimg.cn/7167af1edaa64e2da1d357d70c1f2b75.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


![](https://img-blog.csdnimg.cn/2464ef833a5f44cc857003eea990b847.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


<2>无线网卡连接

考虑到生活的便捷性，大多数人一般都是使用的无线网卡上网，所以这里我们采用连接自己的个人热点进行网络桥接(当然也可以选择WiFi热点，此处为个人热点指南，WiFi连接可同样参考)

如下配置：

* 主机配置

首先电脑win+R，输入`cmd`进入终端，然后输入命令：`ipconfig`，找到自己的热点网络信息

![](https://img-blog.csdnimg.cn/e8696623357746ff88e5c21a6a2f2a38.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


![](https://img-blog.csdnimg.cn/5574394acfd04750ab469dde635794ff.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


![](https://img-blog.csdnimg.cn/838f45e5c87a4d388f7988c0fbfac8f9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


* 虚拟机配置

![](https://img-blog.csdnimg.cn/1ef17e5bffa644d9901a99cddd829509.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


![](https://img-blog.csdnimg.cn/f913337e07084f8ca7e4b45ef6c0e3c1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


```
ctrl+alt+T打开终端，输入命令:vi /etc/network/interfaces
```

![](https://img-blog.csdnimg.cn/7ecb691179304cb5b85f0a906248de63.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


```
保存退出后，再次输入命令：
首先将网卡关闭：ifdown eth0(一般桥接默认为eth0网卡)
然后启用网卡：ifup eth0
```

## 二、开发板端测试：

`以下内容为开发板挂载根文件系统，感兴趣的可以动手实践一下借鉴下面这篇博客`

[【Linux系统开发】x210开发板根目录文件系统构建](https://blog.csdn.net/qq_56914146/article/details/124407302?spm=1001.2014.3001.5502)

我们打开secureCRT：

开机先ping下虚拟机网络：`ping '虚拟机IP'`

> 注意：此处如果无法ping通虚拟机，一般是自己的虚拟机网络有问题，可以尝试输入以下命令解决

```
方法一：打开命令：sudo gedit /etc/NetworkManager/nm-system-settings.conf

出现文件内容：

# This file is installed into /etc/NetworkManager, and is loaded by
# NetworkManager by default.  To override, specify: '--config file'
# during NM startup.  This can be done by appending to DAEMON_OPTS in
# the file:
#
# /etc/default/NetworkManager
#

[main]
plugins=ifupdown,keyfile

[ifupdown]
managed=true

（这里false改成true）
```

```
方法二：虚拟机重置网卡

sudo /etc/init.d/networking restart
sudo /etc/init.d/networking start

ifdown eth0
ifup eth0
```

当开发板ping通虚拟机后，我们在secureCRT控制台输入`reset`命令重启开发板

![](https://img-blog.csdnimg.cn/44ed871d254047618d4b116007408f0e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)




这里的内核加载过程中再次出现了问题，显示我nfs服务端无回应

![](https://img-blog.csdnimg.cn/c6e42fcd827b48f89c479f34e45f6767.png)




解决：

```
mount -t nfs -o nolock '开发板ipaddr ip':/root/rootfs/x210_rootfs   //再次重新挂载根文件系统

//NFC网络重启
/etc/init.d/nfs-kernel-server restart 
sudo /etc/init.d/networking start
```

![](https://img-blog.csdnimg.cn/350f40f5e7134ccca7657925ed590139.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


问题解决！
