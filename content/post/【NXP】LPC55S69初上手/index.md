---
title: 【NXP】LPC55S69初上手
description: 前段时间看到恩智浦社区有一个LPC55S69的开发板测评活动，很荣幸能通过报名，第二天也是成功的收到的板子，本次作为开箱测评。
slug: 【NXP】LPC55S69初上手
date: 2023-02-05 00:00:00+0000
image: cover.jpg
categories:
    - NXP学习
tags:
    - NXP
    - LPC55s69
    - LPC

---



## 前言

前段时间看到恩智浦社区有一个LPC55S69的开发板测评活动，很荣幸能通过报名，第二天也是成功的收到的板子，本次作为开箱测评。

<img src="https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302051131611.png" alt="image-20230205113122365" style="zoom: 50%;" />

## 开始测试

首先从RT-Thread仓库的master分支克隆整个仓库，进入目录：`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk`，首先使用RT-Thread的ENV工具生成MDK工程：

```c
scons --target=mdk5
```

![image-20230205113527665](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302051135725.png)

这里建议大家使用最新版ENV工具。然后双击打开`project.uvprojx`工程，点击重新编译。

![image-20230205113758912](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302051137011.png)

但是编译之后发现会有报错，找了很久都没解决，后来经过RTT社区的满老师提示成功解决BUG，下面是解决过程与分析。

![6d869b905839641fa60aadb8d2a6a9d](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302051140994.png)

![dd1f984d7543997e5fa6fa50aee36c7](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302051140331.png)

## BUG分析与解决

首先先看一下我的keil版本为V5.25：

![4b368869fbf8077591b20eccbd05ef8](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302051141382.png)

听满老师讲LPC55S69的工程可能是使用的AC6编译器，但是Keil的V5.25的AC6可能存在问题，所以解决办法就是更新下Keil的版本（建议最新版）

> 此处附上Keil[最新版下载官网](https://www.keil.com/update/check.asp?P=MDK&V=5.38.0.0&S=)

下载好最新版本后，前面的步骤重复，然后重新编译下载即可。

## 项目演示

下面是RT-Thread成功在LPC55S69的示例,可以看到LED灯以500ms进行闪烁：

![video](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302051354900.gif)

## 结语

本博客仅作为开箱测试，后续会继续上传相关测试用例，欢迎讨论交流。

## 联系

* [Email :yifang.wangyq@foxmail.com](mailto:yifang.wangyq@foxmail.com)
* [Github Address :https://github.com/kurisaW](https://github.com/kurisaW)
* [My Website :https://kurisaw.github.io/](https://kurisaw.github.io/)

