---
title: 【NXP】LPC55S69开发环境搭建
description: 本篇文章用于LPC55S69开发环境及配置工具的使用教学
slug: 【NXP】LPC55S69开发环境搭建
date: 2023-02-04 00:00:00+0000
image: cover.jpg
categories:
    - NXP学习
tags:
    - NXP
    - LPC55s69
    - LPC

---



## 前期准备

资料：

* [MCUXpresso_IDE_User_Guide.pdf](https://www.nxp.com.cn/docs/zh/user-guide/MCUXpresso_IDE_User_Guide.pdf)

开发环境(官方直链)

* [**MCUXpresso Config Tools**](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-config-tools-pins-clocks-peripherals:MCUXpresso-Config-Tools)
* [**MCUXpresso IDE**](https://nxp.flexnetoperations.com/control/frse/download?agree=Accept&element=13944367)
* [**SDK代码包**](https://mcuxpresso.nxp.com/zh/welcome)

`MCUXpresso Config Tools`和`MCUXpresso IDE`的安装不再赘述，下面是`SDK代码包`的安装教学

1.选择开发板-->

<img src="https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041205187.jpg" alt="image-20230203110821236" style="zoom:50%;" />

2.这里我们选择处理器为LPC55S69（选择自己所需的处理器型号），点击构建MCUXpresso SDK v2.13.0（默认最新即可）

<img src="https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302031110101.png" alt="image-20230203111016013" style="zoom:50%;" />

3.根据自己的开发需求进行组件及中间件等，同时选择需要的工具链，这里我们全选，包括工具链和IDE，并点击`下载SDK`

<img src="https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302031122007.png" alt="image-20230203112253916" style="zoom: 50%;" />

4.等待构建完成，这里我们选择我们刚刚生成的档案，点击下载软件包

![image-20230204102200685](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041022034.png)

5.直接选择点击下载SDK档案，包括文档。当然这里也提供了单独的示例工程和API参考手册，需要的朋友也可根据需求下载

![](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041028831.png)

## IDE配置

完成IDE软件、配置工具的安装还有SDK代码包的下载后，我们打开`MCUXpresso IDE`，在主界面的下方栏可以看到有一个`Installed SDKs`，准备好刚刚下载的SDK代码包，导入其中

![image-20230204105301726](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041103892.png)

 之后我们就可以使用这个SDK代码包去创建一个新的工程了。

## 工程导入

这里我们简单做个示范，选择导入示例工程

![image-20230204105601052](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041103834.png)

选择指定的开发板后点击下一步

![image-20230204110547819](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041106647.png)

![image-20230204110900541](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041109611.png)

在下一步这里，就主要是一些Memory的分散加载问题，还有就是编译器语言的标准问题，一般来讲我们默认不做更改，点击完成即可

![image-20230204111200845](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041114568.png)

工程的用户代码是存放在source目录下的，我们这时候就可以给开发板上电，然后点击编译

![image-20230204111519951](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041115091.png)

![image-20230204111803236](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041120134.png)

`MCUXpresso IDE`有两个地方都可以启动调试，选择一个习惯的即可

![image-20230204112026675](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041120777.png)

#### 调试与下载

········

#### 配置工具使用

和`MCUXpresso IDE`配套的还有`MCUXpresso Config Tools`，打开`MCUXpresso IDE`，找到配置工具按钮打开

![image-20230204113350126](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041133492.png)

![image-20230204113817301](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302041138433.png)

自此就可以自由发挥啦！
