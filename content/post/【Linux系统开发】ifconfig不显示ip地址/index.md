---
title: ifconfig不显示ip地址
description: ubuntu终端下命令ifconfig的问题解决
slug: 【Linux系统开发】ifconfig不显示ip地址
date: 2022-07-22 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---

#### ubuntu终端下命令ifconfig的问题解决

> 问题一. ifconfig之后只显示lo,没有看到eth0
问题二. ifconfig之后显示eth0，但是没有显示静态IP地址，即无inet、地址、广播、掩码。
问题三. ping命令不能使用，因为dns还没设置，编辑/etc/resolv.conf，加上dns服务器地址。  

#### 问题一：ifconfig之后只显示lo,没有看到eth0 ?

> 1.eth0设置不正确，导致无法正常启动，修改eth0配置文件就好 
ubuntu 12.04的网络设置文件是/etc/network/interfaces，打开文件，会看到auto lo iface lo inet loopback 
这边的设置是本地回路。在后面加上 
```auto eth0 
iface eth0 inet static 
address 192.168.1.230 //（ip地址） 
netmask 255.255.255.0 //（子网掩码） 
gateway 192.168.1.1 //（网关） 
```
>其中eth0就是电脑的网卡，如果电脑有多块网卡，比如还会有eth1，都可以在这里进行设置。iface eth0 inet 设置为dhcp是动态获取IP，设置为static则用自定义的IP。这边要自定义IP地址，所以选择static选项。  

> 2.eth0被关了
> 输入命令行：ifconfig eth0 up    #开启eth0
---
#### 问题二：ifconfig之后显示eth0，但是没有显示“inet/地址/广播/掩码/ ”?
> 1.先用sudo dhclient eth0更新IP地址
2.然后运行sudo ifconfig eth0
3.reboot
---
#### 问题三：重启后，ping命令不能使用，因为dns还没设置，编辑/etc/resolv.conf，加上dns服务器地址。
> 设置好后，如果直接ping www.baidu.com会发现ping不通，因为dns还没设置，编辑/etc/resolv.conf，加上dns服务器地址。 

	nameserver 8.8.8.8 
	nameserver 8.8.4.4 

> 这两个是Google提供的免费DNS服务器的IP地址
