---
title: x11vnc安装与配置
description: 最近学校在做ros使用树莓派做一个路径规划，但是一般的vncserver无法使用rviz可视化工具，所以选择了x11vnc这款工具。
slug: Linux系统开发
date: 2022-07-14 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---




# 1. 安装x11vnc

```bash
sudo apt-get install x11vnc -y
```

直接安装成功。

# 2. 设置vnc密码

密码存储在/etc/目录里面

```bash
sudo x11vnc -storepasswd /etc/x11vnc.pass
```

放在这个位置，需要设置文件读取权限  
否则会提示密码校验失败

```bash
sudo chmod 777 /etc/x11vnc.pass
```

# 3.创建vnc配置文件

在/etc/init 下创建一个x11vnc.conf的文件

```bash
 cd /etc/init 
 sudo gedit x11vnc.conf
```

文件内容如下：

```bash
#description "xiaoqiang vnc server"
#start on runlevel [2345]
#stop on runlevel [06]
#script
    exec /usr/bin/x11vnc -auth guess -capslock -forever -loop -noxdamage -repeat -rfbauth /etc/x11vnc.pass -rfbport 5900 -shared
#end script
```

我的密码创建在/etc目录下，可以直接复制这段，不需要按照别人博客的修改成自己的，这里用的5900端口，也可以自己换成其他的。

# 4.启动vnc服务

```bash
source /etc/init/x11vnc.conf
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/12b8d37ebfbb4160a7d50196cf3bd2b7.png)  
启动了VNC和X11服务，端口号为5902，我这里用的5902，5900和5901被我分给其他的了

# 5.设置自启动

我直接添加开机启动项没有成功，又写了一个脚本，将脚本添加到开机启动项才成功了。

## (1)首先编写一个脚本

```bash
gedit x11vnc.sh
```

添加以下内容  
第一行是要添加的解释器，后面是要执行的指令内容

```bash
#!/bin/bash
source /etc/init/x11vnc.conf
```

防止误删，从home移动到/etc/init.d/文件夹中  
并添加权限

```bash
sudo mv x11vnc.sh /etc/init.d/
sudo chmod 777 /etc/init.d/x11vnc.sh
```

## (2)添加启动项

点开ubuntu的显示所有应用程序，左下角9个点，找到启动应用程序打开,图中第二行第5个。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/b1f7234307b64d9ca6844704a100a243.png)  
点击右侧添加，添加自动启动项。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/fd2f7303980f4e7598e7f4ce6950ef20.png)  
添加内容如下；重要的是第二行，

```bash
bash /etc/init.d/x11vnc.sh
```

用bash启动才能成功，保存之后重启，确实可以开机自启了。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/caa1005977844d8baffbba60e7d6ea93.png)

# 6.x11vnc配置（安装虚拟显卡驱动）

如果你没有实时使用显示器而又想通过vnc远程查看桌面的话，可以考虑安装虚拟显卡驱动，唯一的缺点就是配置好后显示器那边可能无法正常显示

## (1)首先还是安装命令

```
sudo apt-get install xserver-xorg-video-dummy
```
## (2)接下来就是创建配置文件 `/etc/X11/xorg.conf`

```
Section "Device"
    Identifier  "Dummy"
    Driver      "dummy"
    VideoRam    64000
    Option      "IgnoreEDID"    "true"
    Option      "NoDDC" "true"
EndSection

Section "Monitor"
    Identifier  "Monitor"
    HorizSync   15.0-100.0
    VertRefresh 15.0-200.0
EndSection

Section "Screen"
    Identifier  "Screen"
    Monitor     "Monitor"
    Device      "Dummy"
    DefaultDepth    24
    SubSection  "Display"
        Depth   24
        Modes   "1280x720"
    EndSubSection
EndSection
```

## (3)再修改个文件加点配置

```
vi /boot/firmware/usercfg.txt
```

```
framebuffer_width=1280
framebuffer_height=720
```

**如果想要恢复显示器的连接，可以先使用ssh访问终端并将`/etc/X11/xorg.conf`这个文件删除，再次重启即可**
