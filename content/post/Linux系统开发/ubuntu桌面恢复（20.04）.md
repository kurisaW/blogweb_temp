---
title: ubuntu桌面恢复（20.04）
description: 恢复ubuntu20.04默认桌面管理器
slug: Linux系统开发
date: 2022-07-14 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---

### 恢复ubuntu20.04默认桌面管理器

- - - [一、GDM, KDM, LightDM, SDDM的区别和安装配置](#GDM_KDM_LightDM_SDDM_5)
    - - [1、GDM，gnome系列的图形管理器](#1GDMgnome_8)
      - [2、KDM,SDDM是KDE系列的图形管理器](#2KDMSDDMKDE_17)
      - [3、LightDM](#3LightDM_27)
    - [二、配置和切换](#_37)
    - [三、恢复ubuntu20.04默认桌面管理器](#ubuntu2004_62)
    - - [1、打开终端，用管理员口令下载相关资源](#1_65)
      - [2、安装gnome-shell](#2gnomeshell_71)
      - [3、安装ubuntu-gnome-desktop](#3ubuntugnomedesktop_80)
      - [4、安装unity-tweak-tool和gnome-tweak-tool](#4unitytweaktoolgnometweaktool_85)
      - [5、安装完成后重启](#5_94)

起因：我是一个windows重度用户，实验室配置了Ubuntu服务器，我试图用远程桌面控制控制服务器的桌面。由于对Linux一窍不通，一顿乱改。结果虽然能[远程控制桌面](https://blog.csdn.net/irober/article/details/112608610)了，可是原有的显示管理器被我更改了。原先跑的好好的深度学习代码也不能跑了，原先的桌面风格（**gnome图形管理器**）也变成了我不喜欢的风格\(**轻量级的LightDM**\)了，大家以后要慎重。  
注意：我是个半吊子，仅供参考。

### 一、GDM, KDM, LightDM, SDDM的区别和安装配置

[GDM, KDM, LightDM, SDDM的区别和安装配置](https://blog.csdn.net/u014466109/article/details/105572470)  
**gdm3，kdm 和 lightdm** 都是显示管理器。 它们提供图形化登录并处理用户身份验证。

#### 1、GDM，gnome系列的图形管理器

```python
sudo apt-get install gdm3
```

```python
sudo apt-get remove gdm3
```

#### 2、KDM,SDDM是KDE系列的图形管理器

kdm 是kde管理器的显示。 但在KDE5中，它被否决为 SDDM，它更适合作为显示管理器，因此在默认情况下，它是在屏幕。

```python
sudo apt-get install sddm 
```

```python
sudo apt-get remove sddm
```

#### 3、LightDM

LightDM用于显示管理器的规范解决方案。 它应该是轻量级的，默认情况下是 Ubuntu。Xubuntu和 Lubuntu。 它是可以配置的，有多种欢迎主题可用。

```python
sudo apt-get install lightdm
```

```python
sudo apt-get remove lightdm
```

### 二、配置和切换

```python
sudo dpkg-reconfigure gdm3
```

你可以在上述命令中使用管理器的名字代替 gdm3，可在它们之间进行选择。 必须重新启动才生效。

要检查当前正在使用的显示管理器，请运行以下命令：

```python
cat /etc/X11/default-display-manager
```

Lightdm，gdm3和KDM都是针对linux的图形化登录。 Lightdm是Ubuntu的默认版本。 要在显示管理器之间进行 switch，请使用以下命令：

```python
sudo dpkg-reconfigure lightdm
```

Lightdm，gdm3和KDM都是针对linux的图形化登录。 Lightdm是Ubuntu的默认版本。 要在显示管理器之间进行 switch，请使用以下命令：

```python
sudo dpkg-reconfigure lightdm
```

GDM\(GNOME Display Manager\)，LightDM\(Light Display Manager\) 和 KDM\(KDE Display Manager\) 是为不同版本的Ubuntu配置的管理器。 他们帮助启动X 服务器。用户会话和欢迎\( 登录屏幕\)。 你可以运行 sudo dpkg-reconfigure gdm 以在 lightdm。gdm和KDM之间进行更改。 安装它们就像 sudo apt-get install \( 显示manger将被 kdm，gdm 和 lightdm 替换。

### 三、恢复ubuntu20.04默认桌面管理器

[恢复ubuntu20.04默认桌面管理器](https://www.zhihu.com/tardis/sogou/art/27659651)  
目前Ubuntu的主流桌面**GNOME**， Ubntu的内置桌面是**Untiy**

#### 1、打开终端，用管理员口令下载相关资源

```python
Ctrl+Alt+T
```

打开终端，用管理员口令下载相关资源

#### 2、安装gnome-shell

```python
sudo apt-get install gnome-shell
```

**提示：**

管理员权限需要输入密码，但是系统不会显示你输入的密码  
输入完成后，直接回车即可

#### 3、安装ubuntu-gnome-desktop

```python
sudo apt-get install ubuntu-gnome-desktop
```

#### 4、安装unity-tweak-tool和gnome-tweak-tool

```python
sudo apt-get install unity-tweak-tool
```

```python
sudo apt-get install gnome-tweak-tool
```

#### 5、安装完成后重启

然后一切恢复如初，仿佛没发生过。  
