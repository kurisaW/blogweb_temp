---
title:  RDC 2022纪念版开发板-D1S在RT-Smart运行
description: 近日在RT-Thread举办的RDC开发者大会上抽奖获得的全志D1S开发板，这段时间也是借此来简单做个小测试。
slug:  RDC 2022纪念版开发板-D1S在RT-Smart运行
date: 2023-1-19 00:00:00+0000
image: cover.jpg
categories:
    - experience_sharing
tags:
    - Risc-v
    - RDC
    - D1s
    - RT-Smart
---

## 开发环境

软件

* ubuntu20.04
* VMware Workstation

硬件

* RDC2022纪念版开发板
* 全志D1s芯片

## 材料下载

首先打开虚拟机，创建一个目录存放本次测试的代码，然后克隆RT-Smart用户态代码。

```makefile
git clone https://github.com/RT-Thread/userapps.git
```

![image-20230119110742488](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191107894.png)

在`userapps`目录下克隆RT-Thread仓库代码

```makefile
git clone https://github.com/RT-Thread/rt-thread.git
```

![image-20230119110934253](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191109402.png)

## Riscv工具链配置

进入`userapps/tools`，运行 get_toolchain.py 的脚本，会下载对应的工具链并展开到` userapps\tools\gun_gcc` 目录。

```makefile
python3 get_toolchain.py riscv64
```

![image-20230119111856993](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191118227.png)

返回上一级，刷新工具链环境，同时记住这里的`EXEC_PATH`工具链路径，后面需要修改为此路径

```makefile
cd ..
source smart-env.sh riscv64
```

![image-20230119111552268](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191115786.png)

## 内核环境编译

#### scons安装

环境编译会用到`scons`，所以我们先下载scons

```makefile
sudo apt install scons
```

查看scons版本信息可判断是否安装成功

![image-20230119112101897](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191121945.png)

#### env工具安装

依次执行以下程序：

```makefile
scons --menuconfig
source ~/.env/env.sh
pkgs --update
```

#### 内核编译

使用 scons 命令进行编译，编译成功后会在 `userapps/rt-thread/bsp/allwinner/d1s` 目录下生成 `sd.bin`，这个文件就是我们需要烧录到开发板中的文件，它包括了 `uboot.dtb，opensbi，rtthread.bin`。

```
scons
```

此时直接编译会报错，因为工具链路径还没有修改

![image-20230119112916923](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191129532.png)

我们复制上面的工具链路径，vi命令修改rtconfig.py，这里的路径依据你自己的工具链路径

![image-20230119113207832](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191132933.png)

再次执行scons命令编译

![image-20230119113353060](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191133159.png)

## 程序烧录

我这里采用的是从TF卡作为启动方式。

1、首先准备一张容量在128G的空白TF卡

2、格式化TF卡，并使用ubuntu的gparted工具重新分区

如果没有下载该工具可使用下面的命令进行下载：

```c
sudo apt install gparted
```

启动该工具

```
sudo gparted
```

这里我使用的是一张64G的TF卡，扇区大小为512字节，同时我们需要预留8M的前空间，并且分区的文件系统格式为fat32

![image-20230119114019113](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191140208.png)

3、接下来进行程序的烧录

首先进入`userapps/rt-thread/bsp/allwinner/d1s/tools`，执行命令：

```makefile
sudo dd if=boot0_sdcard_sun20iw1p1_d1s.bin of=/dev/sdb bs=1024 seek=8
```

![image-20230119114457823](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191144935.png)

返回上一级，再次执行命令：

```makefile
sudo dd if=sd.bin of=/dev/sdb bs=1024 seek=56
```

![image-20230119114605503](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191146686.png)

到此烧录工作已完成。

## 启动RT-Smart

我们将刚刚烧录好程序的TF卡直接插入到开发板卡槽，并连接开发板UART端口进行串口查看验证。

此处注意串口波特率为`500000`

![image-20230119115334091](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191153203.png)

简单测试下MSH命令：

![image-20230119115950076](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301191159278.png)

到此就测试结束啦，欢迎大家讨论交流。