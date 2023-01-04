---
title: Ubuntu命令查看手册
description: Ubuntu命令查看手册
slug: Linux系统开发
date: 2022-07-25 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---



## 进程管理类
1.top命令  

> * top命令是一个常用的查看系统资源使用情况和查看占用系统资源最多的进程的命令。
> * top以列形式显示所有的进程，占最多CPU资源的进程会显示在最上面。

2.htop命令

> * htop命令是top的改进版。
> * 默认情况下，大多数Linux发行版本都没有安装htop。
> * htop命令显示的信息与top相同，但它的界面更人性化。

3.pstree

> * pstree命令也可以显示进程信息。
> * 它以树的形式显示进程。

4.kill

> * kill命令可以根据进程ID来杀死进程。  
> * 你可以使用ps -A，top，或者grep命令获取到进程ID。  

    从技术层面来讲，kill命令可以发送任何信号给一个进程。
    你可以使用 kill -KILL [id] 或者 kill -9 [id] 来杀死顽固的进程。  
---
## 文件操作类(基础篇)
>**新建文件:touch**  
    详细文档通过 man [command] 查看  

>**管理文件**
* rm: 删除文件或目录（-r）
* mkdir 新建目录
* cp /home/jack/README.md /home/jack/work/  拷贝文件或目录（-r）
* mv  移动或重命名文件、目录

> **压缩tzip文件**

* zip FileName.zip DirName    # 将DirName本身压缩
* zip -r FileName.zip DirName # 压缩，递归处理，将指定目录下的所有文件和子目录一并压缩

>**解压zip文件**
* unzip filename

> **查找含`spark`的目录、文件**
* find /home/jack -name '*spark*' 

> **更改密码**
* passwd 

> **更改文件名或移动文件位置**
* 语句：mv oldFileName  newFileName  
* 示例：我想把 aaa.txt修改为 bbb.txt示例语句：mv  aaa.txt  bbb.txt

>**删除文件**
* 删除文件: rm test.txt
* 删除空文件夹: rmdir test
* 删除非空文件夹及其目录下的所有文件夹及文件:rm -r test
* 删除 除某个文件或文件夹之外的所有文件以及文件夹:rm -r (文件名称或文件夹名称)括号里可以放多个，用 | 分开，如rm -r (test | test.txt)

## 防火墙状态
首先需要输入安装命令：
`apt install ufw`
> 查看防火墙当前状态
`sudo ufw status`

> 开启防火墙
> `sudo ufw enable`

>关闭防火墙
`sudo ufw disable`

>查看防火墙版本
`sudo ufw version`

>默认允许外部访问本机
`sudo ufw default allow`

>默认拒绝外部访问主机
`sudo ufw default deny`

>允许外部访问443端口
`sudo ufw allow 443`

>拒绝外部访问443端口
`sudo ufw deny 443`

>允许某个IP地址访问本机所有端口
`sudo ufw allow from 192.168.0.1`


## 网络设置
>重置网卡
>sudo /etc/init.d/networking restart
