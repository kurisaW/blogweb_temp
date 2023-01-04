---
title:  x210开发板根目录文件系统构建
description: x210开发板根目录文件系统构建
slug: Linux系统开发
date: 2022-07-28 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---

## 一、开发板配置

（使用secureCRT）
首先确保开发板完成以下配置：

主机IP：
`set ipaddr192.168.1.10`
服务器IP：
`set serverip 192.168.1.141`
网关：
`set gatewayip 192.168.1.1`
子网掩码:
`set netmask 255.255.255.0`
内核驱动设置：
`set bootcmd 'tftp 30008000 zImage; bootm 30008000'`
bootargs配置：
`set bootargs root=/dev/nfs nfsroot=192.168.1.141:/root/rootfs/x210_bsp ip=192.168.1.10:192.168.1.141:192.168.1.1:255.255.255.0::eth0:off init=/linuxrc console=ttySAC2,115200`

最后输入save保存一下，这样开发板的网络和内核配置就设置好了

## 二、了解rootfs

rootfs的两种表现形式：
1、nfs方式启动的文件夹形式的rootfs（主机）

2、用来烧录的镜像形式rootfs（开发板）

## 三、虚拟机文件配置

#### 1.目录配置

首先我们需要root进入超级用户模式，在虚拟机的root目录下再次创建以下两个目录：
`rootfs  x210_bsp`

这时候我们需要知道这两个文件夹下有什么：

> * x210_bsp：用于uboot烧录和配置
> * rootfs：用于挂载开发板根文件系统

#### 2.x210_bsp配置

首先进入到该目录下，并将文件qt_x210v3s_160307.tar.bz2复制到该目录下解压

![](https://img-blog.csdnimg.cn/b4f56856fa1447b6b49ea43313bc141a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


以上是解压qt_x210v3s_160307.tar.bz2内的文件内容，后面会说到这个目录如何使用

#### 3.rootfs配置

首先我们需要在该目录下继续创建一个名为x210_rootfs的文件夹，并且进入到该文件夹下，将我们上面提到的busybox文件复制到此目录下并解压

![](https://img-blog.csdnimg.cn/87600bf5cbf44ac3a94db3bc7bd01d78.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


以上是解压busybox-1.24.1（这是我选择的busybox版本）的全部文件

#### 4.make menuconfig

进入x210_bsp/kernel 目录下，输入命令：make menuconfig进入图形化菜单

> 这里我们按下面操作完成网络配置

```
[*]Networking support --->
	Networking options --->
```

![](https://img-blog.csdnimg.cn/a33a8e4e25bc4335b263cdbc59bd79a6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)




> 网络文件系统设置

```
File systems --->
	[*]Networking File Systems --->
```

有需要把开发板作为服务器端的也可以选择把`NFS server support`设置打开，这里我们仅实验客户端

![](https://img-blog.csdnimg.cn/ce11d76876fd463db53915dffde15776.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)




以上配置结束后输入命令`make`编译，至此开发板uboot的网络和文件系统部分配置结束。

## 四、busybox的移植实战

#### 1、了解busybox

> busybox是一个集成了一百多个最常用linux命令和工具的软件,他甚至还集成了一个http服务器和一个telnet服务器,而所有这一切功能却只有区区1M左右的大小.我们平时用的那些linux命令就好比是分立式的电子元件,而busybox就好比是一个集成电路,把常用的工具和命令集成压缩在一个可执行文件里,功能基本不变,而大小却小很多倍。

#### 2、busybox源码获取

[busybox官网](http://www.busybox.net/)

`注意：我们在文件系统构建中，内核编译和文件系统的程序编译都必须是使用的统一交叉编译器。（选择将虚拟机中的交叉编译文件复制一份到开发板构建的文件系统下）`


#### 3、busybox配置

（1）修改Makefile

首先进入`~/rootfs/x210_rootfs/busybox-1.24.1`目录下

输入命令`vi Makefile`进入脚本进行以下修改

173行：`CROSS_COMPILE=/usr/local/arm/arm-2009q3/bin/arm-none-linux-gnueabi-`
`注意：此处的交叉编译链需要对照自己电脑的交叉编译链`
191行：`ARCH=arm`

![](https://img-blog.csdnimg.cn/988e15e86dde4807aa26011e6686bc6f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


（2）make menuconfig配置

Tip:此处的图形化菜单需要ncurses库（联网下载）,由于之前博主自己在这里没有很深的基础知识，走了很多弯路。
因为后面的文件系统的挂载需要虚拟机切换网络状态为桥接模式，但是我的虚拟机桥接网络总是会反复重连，所以建议先将该库下载好，方便后续使用。

make menuconfig

```
Busybox Settings--->
	Build Options--->
		[*]Build BusyBox as a static binary(no shared libs)

		
Busybox Library Tuning--->
	[*]vi-style line editing commands
	[*]Fancy shell prompts
	
	
Linux Module Utilities--->
	[ ]Simplified modutils
	[*]insmod
	[*]rmmod
	[*]lsmod
	[*]modprobe
	[*]depmod

	
Linux System Utilities--->[*]mdev
	[*]Support /etc/mdev.conf
	[*]Support subdirs/symlinks
	[*]Support regular expressions substitutions when renaming dev
	[*]Support command execution at device addition/removal
	[*]Support loading of firmwares
```

大家学习使用的时候跟着上面的进行配置即可
配置完成后，输入以下命令：
`make -j4` (4代表我主机的内核数)
无报错继续下一步：
make install

> 解释：在Linux系统中安装软件的一般步骤：下载-配置-编译-安装，所以上面的make -j4就代表编译，make install代表安装

（3）设置busybox安装路径
* `make menuconfig`

```
Busybox Settings --->
	Installation Options ("make install" behavior) --->
		(./)BusyBox installation prefix) //这里设置安装路径
```

![](https://img-blog.csdnimg.cn/cf7de167425b47acaff070d610f78d1f.png)



（4）解决方案
在虚拟机的配置中，由于代码的复杂性时常让我们不能很全面清晰的看到自己所做的改变，有时候就会出现各种各样的状况。

make -j4编译可能遇到的问题：

* `sync.c(text.sync_main+0x78):undefined reference to 'syncfs'`



`分析：`可能是gcc和当前busybox版本不兼容造成的，我们只需要将其禁用即可。

`解决方法：`

`make menuconfig`
点击/进入搜索，输入SYNC，根据提示禁用SYNC
最后再make -j4编译一下即可


![](https://img-blog.csdnimg.cn/7defb1eda80748d29c73a564f2e631be.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


![](https://img-blog.csdnimg.cn/2b4ded5286854f1ca4e7d8fae5e148c4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/932a1580f3b54a5ca685be2b7c5c1dd7.png)


其实还可以选择在源代码中解决这个问题，过程有些繁琐就不赘述，动手能力强的可以一试。

（5）make install简述

* 默认安装位置：./_install
* 文件包含有：bin linuxrc sbin usr

![](https://img-blog.csdnimg.cn/002a97e1448542f784b04d188f6cb7f7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


<strong>ls -l可以看到： linuxrc -> bin/busybox  //这个linuxrc其实就是个符号链接 </strong>

![](https://img-blog.csdnimg.cn/b97a1a96e2a94503832d05c0f8ca072e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


<strong>这里也不难发现，bin下的所有的符号链接都指向了busybox</strong>

（6）make menuconfig更改NFS挂载目录到/root/rootfs/x210_rootfs下

```
make menuconfig 
	Busybox Settings —> 
		Installation Options (“make install” behavior) —> 
			(/root/rootfs/x210_rootfs)BusyBox installation prefix
```

执行`make install`后，回到被挂载的目录下，可以发现这四个文件已经生成。



## 五、NFS挂载根文件系统

#### 1.NFS简述

* NFS 是Network File System的缩写，即网络文件系统。
* 功能：通过网络让不同的机器、不同的操作系统能够彼此分享个别的数据，让应用程序在客户端通过网络访问位于服务器磁盘中的数据。

#### 2.NFS服务器安装

`sudo apt-get install nfs-kernel-server`



#### 3.NFS使用过程

启动NFS服务器->启动NFS客户端->挂载NFS目录

#### 4.NFS配置

* 输入命令`vim /etc/exports`

在最后一行修改

* `"文件挂载目录"  *(rw,sync,no_root_squash,no_subtree_check)`

![](https://img-blog.csdnimg.cn/a355d9b6e233404ea19531fa04947910.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


* 保存退出后，输入`mount -t nfs -o nolock 192.168.240.33:/root/rootfs/x210_rootfs`(根据实际情况修改)

![](https://img-blog.csdnimg.cn/9383c4e9c6e74c7485d07a6aec6f99fd.png)


* 输入命令`/etc/init.d/nfs-kernel-server restart`重启NFS服务

## 六、开发板根目录配置

首先将etc目录放置到挂载根目录下

etc目录下载：

[点击此处](https://download.csdn.net/download/qq_56914146/85219254)

#### 1.inittab文件详解

<1>添加一个典型的inittab文件到etc目录下

[inittab下载](https://download.csdn.net/download/qq_56914146/85219254)

<2>inittab格式解析

`id:runlevels:action:process`

![在这里插入图片描述](https://img-blog.csdnimg.cn/4f374d2fe1ca4f7da6f22403e314525f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


解释：

* id：标识符，即代表记录的名字
* runlevels（可不填）：用于指定该记录在哪些运行级别中运行，runlevel可以设定为单个运行级别，也可以设定多个运行级别

![](https://img-blog.csdnimg.cn/dd8867359c0c44c6945fc29dbf0a90a3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


* action：用于描述该级别该执行什么操作（部分说明）

![](https://img-blog.csdnimg.cn/99fb967e453c456b989bd57ec0e84020.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


* process：具体执行的命令



<3>了解busybox init与inittab之间的关系

* busybox init进程主要完成系统的初始化工作。

busybox init进程的工作流程：


> <strong>为init设置信号处理过程->初始化控制台->剖析/etc/inittab文件->执行系统初始化命令行，缺省（默认）情况下会使用/etc/init.d/rcS->执行所有导致 init 暂停的 inittab 命令(动作类型： wait)->执行所有仅执行一次的 inittab(动作类型： once)</strong>

* 一旦完成以上工作， init 进程便会`循环执行`以下进程：

><1>执行所有终止时必须重新启动的 inittab 命令(动作类型： respawn）
><2>执行所有终止时必须重新启动但启动前必须询问用户的 inittab 命令（动作类型： askfirst)

* 简而言之，就是初始化控制台之后， BusyBox 会检查/etc/inittab 文件是否存在，如果此文件不存在， BusyBox 会使用缺省的inittab 配置，它主要为系统重引导，系统挂起以及 init 重启动设置缺省的动作，此外它还会为四个虚拟控制台（tty1 到 tty4）设置启动 shell 的动作。如果未建立这些设备文件， BusyBox 会报错。

注意：理解inittab的关键就是明白“当满足action的条件时就会执行process这个程序。” 去分析busybox的源代码就会发现，busybox最终会进入一个死循环，在这个死循环中去反复检查是否满足各个action的条件，如果某个action的条件满足就会去执行对应的process。

<4>配置
vi命令打开inittab模板文件
```
#first:run the system script file   注释
::sysinit:/etc/init.d/rcS    //在控制台初始化之前执行rcS

::askfirst:-/bin/sh

::ctrlaltdel:-/sbin/reboot  //执行控制台时的打印信息

#umount all filesystem  //同时按住3键可以重启
::shutdown:/bin/umount -a -r//关机时接触挂载init

#restart init process//重启时启动
::restart:/sbin/init
```

修改脚本：
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ccb3db4d10c417b9eeb5526f87c7793.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


#### 2.rcS文件详解

 <1>添加一个典型的rcS文件到etc目录下

[rcS下载](https://download.csdn.net/download/qq_56914146/85219254)

<2>rcS文件解析
```
#!/bin/sh   需要继续添加环境变量，在后面：/new 即可
PATH=/sbin:/bin:/usr/sbin:/usr/bin 

runlevel=S
prevlevel=N

umask 022

export PATH runlevel prevlevel

mount -a
```

* PATH=xxx

> PATH这个环境变量是linux系统内部定义的一个环境变量，含义是操作系统去执行程序时会默认到PATH指定的各个目录下去寻找。如果找不到就认定这个程序不存在，如果找到了就去执行它。将一个可执行程序的目录导出到PATH，可以让我们不带路径来执行这个程序。

* runlevel=

> linux操作系统自从开始启动至启动完毕需要经历几个不同的阶段，这几个阶段就叫做runlevel。例如init 0就是关机，init 6 就是重启

![](https://img-blog.csdnimg.cn/2c17661c32464ad8a0c96c654e622289.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


* umask=

> umask是linux的一个命令，作用是设置linux系统的umask值,而umask值决定当前用户在创建文件时的默认权限。

* mount -a

> mount -a是挂载所有的应该被挂载的文件系统，在busybox中mount -a时busybox会去查找一个文件/etc/fstab文件，这个文件按照一定的格式列出来所有应该被挂载的文件系统（包括了虚拟文件系统）


#### 3.rcS实战

首先将前面提供的etc压缩包模板下载至共享文件夹

<1>输入命令打开rcS脚本：`vi etc/init.d/rcS`。我们可以发现在每一行代码的后面都有一个^m，将其删除，这样开发板启动的时候就不会报错了

<2>mdev

> udev/mdev的工作就是配合linux驱动生成相应的/dev目录下的设备文件。

rcS文件中没有启动mdev的时候，ls查看/dev目录下启动后是空的；在`rcS`文件中添加以下与mdev有关的2行配置项后：

```
echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d84ea4a6677a4af08741ba6c4b6db580.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)



再次启动系统后发现/dev目录下生成了很多的设备驱动文件

<3>hostname

`我们进入etc目录下创建一个名为sysconfig的文件夹，并在该目录下再次touch创建一个名为HOSTNAME的文件，vi命令进入可修改当前系统主机名`

hostname是linux中的一个shell命令。hostname xxx执行后可以设置当前主机名为xxx ，直接hostname不加参数可以显示当前系统的主机名。

- 添加profile文件(该文件在前面etc提供的模板文件有)后，即可显示用户名和hostname

<4>ifconfig

(1)有时候我们希望开机后进入命令行时ip地址就是一个指定的ip地址（譬如192.168.240.40），这时候就可以在rcS文件中ifconfig eth0 192.168.240.40

![在这里插入图片描述](https://img-blog.csdnimg.cn/5842d7cc0dd94c3fbd348189e908f09b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)



<5>mount挂载测试

这时候我们在secureCRT中启动开发板，可以发现还是存在一些报错，例如

```
mount: mounting tmpfs on /var failed: No such file or directory
mount: mounting tmpfs on /tmp failed: No such file or directory
mount: mounting tmpfs on /dev failed: No such file or directory
......
```

这是由于我们的之前创建的根目录挂载文件中没有创建这些文件，`输入mkdir命令在根目录依次创建即可`。

## 七、动态链接库的拷贝

#### 1.静态编译链接测试

首先我们在开发板根目录下touch a.c文件，然后gcc编译一下它，可以发现在虚拟机中可以成功打印,但是在开发板端执行编译命令却并没有成功，这是因为在开发板中并没有交叉编译的相关文件

```
a.c file->

#include<stdio.h>
int main()
{
	printf("hello world!\n");
	return 0;
}
```

#### 2.解决办法：

`拷贝一份动态链接库文件到开发板根目录下`

```
cp lib/*so* /root/rootfs/x210_rootfs/lib/ -rdf
```

#### 3.解释:

![](https://img-blog.csdnimg.cn/6e39e3914f954ff7967b904a8aa576c5.png)


这时候执行命令./a.out发现可以正常打印

#### 4.strip工具

动态链接库so文件中包含了调试符号信息，这些符号信息在运行时是没用的（调试时用的），这些符号会占用一定空间。在`传统的嵌入式系统中flash空间是有限的`，为了`节省空间`常常把这些符号信息去掉。这样节省空间并且不影响运行。

去掉符号信息的命令：

`arm-linux-strip *so*`

## 八、ext2格式镜像烧录

#### 1.	确定文件夹格式的rootfs可用

前面我们已经提前配置好，此处不再赘述

![](https://img-blog.csdnimg.cn/560f47331e76495994c412e7f9e97425.png)


#### 2.ext2镜像制作

* 首先我们在~/rootfs目录下mkdir ext2_rootfs创建用于我们的挂载目录。

* 然后输入以下命令：

```
dd if=/dev/zero of=rootfs.ext2 bs=1024 count=10240

losetup  /dev/loop1 rootfs.ext2

mke2fs -m 0 /dev/loop1 10240

mount -t ext2 /dev/loop1 ./ext2_rootfs/
```

* 此时我们复制一份开发板根目录到ext2_rootfs下

```
cp rootfs.ext2 /mnt/hgfs/Myshare/ -f
```

* 进入~/rootfs目录，执行清除卸载命令

```
umount /dev/loop1
losetup -d /dev/loop1
```

* 此时在rootfs目录下可以看见生成了一个rootfs.ext2镜像文件，我们将其复制到共享文件夹下，然后再将其复制到电脑[fastboot](https://blog.csdn.net/qq_56914146/article/details/124204098?spm=1001.2014.3001.5501)目录下,执行uboot烧录操作，借鉴该博客[【Linux系统开发】Study210开发板刷安卓系统](https://blog.csdn.net/qq_56914146/article/details/124204098?spm=1001.2014.3001.5501)

至此开发板根目录构建完成，其中也是遇到很多问题，也因此给自己挖了很多坑，然后又给自己填坑，虽然过程不尽人意，但是最后获得的都是自己的，大家在尝试这个实验的时候欢迎博客私信交流！

---

参考资料：

* [Linux开发之根文件系统构建及过程详解](https://blog.csdn.net/wangweijundeqq/article/details/82533485?spm=1001.2014.3001.5502)

* [busybox init进程和/etc/inittab关系](https://blog.csdn.net/u010299133/article/details/93414146)

* [NFS-LINUX挂载实践](https://blog.csdn.net/u010311609/article/details/123137181?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165080397516782184664736%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165080397516782184664736&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-2-123137181.142^v9^pc_search_result_control_group,157^v4^control&utm_term=NFS%E6%8C%82%E8%BD%BDlinux+&spm=1018.2226.3001.4187)

