---
title: 【NXP】LPC55S69-Micropython移植日志
description: 本篇文章用于LPC55S69使用Micropython软件包教程
slug: 【NXP】LPC55S69-Micropython移植日志
date: 2023-02-06 00:00:00+0000
image: cover.jpg
categories:
    - NXP学习
tags:
    - NXP
    - LPC55s69
    - LPC
    - MicroPython
    - RT-Thread

---



## 简单了解Micropython

* MicroPython 是 Python 3 编程语言的一种精简而高效的实现，它包含 Python 标准库的一个子集，并被优化为在微控制器和受限环境中运行。

* RT-Thread MicroPython 可以运行在任何搭载了 RT-Thread 操作系统并且有一定资源的嵌入式平台上。

* MicroPython 可以运行在有一定资源的开发板上，给你一个低层次的 Python 操作系统，可以用来控制各种电子系统。

* MicroPython 富有各种高级特性，比如交互式提示、任意精度整数、闭包函数、列表解析、生成器、异常处理等等。

* MicroPython 的目标是尽可能与普通 Python 兼容，使开发者能够轻松地将代码从桌面端转移到微控制器或嵌入式系统。程序可移植性很强，因为不需要考虑底层驱动，所以程序移植变得轻松和容易。

## 开发环境

* VScode
* Keil（v5.38.0.0）
* RT-Thread MicroPython IDE（VScode插件搜索）
* [ENV v1.4.0（可点击链接下载）](https://github.com/RT-Thread/env-windows/tree/v1.3.5)

## 初步移植

首先从RT-Thread官方仓库克隆master分支的仓库到本地

![image-20230206105228123](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061052497.png)

来到该目录：`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk`，鼠标右键打开ENV工具，首先打开命令行菜单

```
menuconfig
```

使能添加`Micropython软件包`：`RT-Thread Online Packages--->launage packages--->Micropython`

![image-20230206110054882](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061100977.png)

`Heap size`修改为`20480`（初次分配20K，后续用户可根据需求修改），同时版本选择最新版(这里由于我选择版本时没有注意到最下方的latest版本，但是经测试并于多出的报错问题，相关的报错也可参考该文章)

![image-20230206110338978](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061103056.png)

进入`Hardware Module`，使能`machine uart`

![image-20230206110701904](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061107994.png)

同时我们回到主菜单界面，进入`Hardware Drives config--->on-chip Peripheral Drivers`，使能UART0和UART2

![image-20230206110948958](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061109036.png)

由于后续需要在main线程中启动Micropython运行时环境，需要增大main线程的栈大小，这里我们选择栈大小修改为8k：回到主界面`RT-Thread Components--->set main thread stack size`修改为8192

![image-20230206115128667](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061151008.png)

保存退出，并使用命令下载软件包：

```
pkgs --update
```

![image-20230206115308233](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061153317.png)

使用ENV生成MDK工程：

```
scons --target=mdk5
```

![image-20230206115527689](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061155767.png)

## BUG修复

双击打开`project.uvprojx`，进行编译

![image-20230206115702684](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061157814.png)

这里由于我们的keil工程为AC6版本（如果您的编译器版本为AC5，应该不需要修改，仅猜测），需要将软件包进行修改：`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk\packages\micropython-v1.13.0\SConscript`

![image-20230206120429651](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061204757.png)

切记此时需要回到bsp目录下，重新使用ENV工具生成MDK文件，然后再回到keil重新编译工程：

```
scons --target=mdk5
```

此时编译错误大大减少，只剩下三个错误：

![image-20230206120743700](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061212323.png)

第一个错误需要在菜单中使能`Support legacy version for compatibility`（目前该问题以推送至官方仓库，已被修复此问题），并重新使用ENV生成MDK工程文件

![image-20230206111143483](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061111567.png)

重新编译继续有报错，这里我们找不到该函数的定义，先在头文件中进行外部声明

![image-20230206121231129](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061212175.png)

找到头文件所在位置：`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk\packages\micropython-v1.13.0\port\mpgetcharport.h`

![image-20230206121521727](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061215795.png)

此时就剩下最后一个错误啦，这里报错是说这个宏没有定义，通过翻阅RT-Thread库函数，确定该宏是文件系统的一个宏，且定义为整型3，具体作用可查看此[PR](https://github.com/RT-Thread/rt-thread/pull/2100)，所以解决该问题就是重新定义一下`DFS_FD_OFFSET`

![image-20230206121656320](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061216368.png)

![image-20230206122027240](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061220316.png)

想不到编译之后居然还有一个错误，这里参考这位开发者的[issue](https://github.com/RT-Thread/rt-thread/issues/6657)，将`list_mem();`注释（此处可能是个官方BUG，后续尝试修复）

![image-20230206122146590](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061221642.png)

![image-20230206122748054](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061227108.png)

最后发现，终于没有错误啦！！！

![image-20230206122817350](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061228418.png)

## RT-Thread Micropython环境搭建

VScode扩展搜索下载RT-Thread Micropython

![image-20230206123632247](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061236343.png)

#### 创建工程

vscode下方导航栏点击`创建Micropython工程`，创建一个新的MicroPython工程，并选择工程存放路径

![image-20230206151916502](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061519616.png)

![image-20230206152143031](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061521140.png)

#### 上电测试Micropython

点击下方工具栏连接开发板，打开串口设备后点击复位，此时出现RT-Thread官方LOGO

![image-20230206152315131](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061523214.png)

![](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061524180.png)

#### 测试示例

LPC55S69也成功移植了RT-Thread的FINSH组件，点击TAB键可查看Finsh控制台命令，我们可以看到有一个python命令行

![image-20230206154101713](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061541861.png)

## Micropython测试

Finsh控制台输入python，转到python控制台，同时还支持`quit()`、`exit()`命令退回Finsh控制台

![image-20230206154310678](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061543769.png)

简单测试下micropython，下面使用python命令运行脚本时给了一个提示说未使能uos module

![image-20230206160622977](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061606460.png)

打开图形化菜单进入该路径下：`RT-Thread online packages-->launage packages--->system module`，使能`uos:basic 'operating system' services `

同时更新软件包，并使用env工具重新生成MDK，再进行编译下载，成功解决问题！

![image-20230206162718225](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061627396.png)

## 结语

搭建好Micropython后，那么就可以自由发挥才能去创作自己的作品啦！

## 联系

* [Email :yifang.wangyq@foxmail.com](mailto:yifang.wangyq@foxmail.com)
* [Github Address :https://github.com/kurisaW](https://github.com/kurisaW)
* [My Website :https://kurisaw.github.io](https://kurisaw.github.io/)
