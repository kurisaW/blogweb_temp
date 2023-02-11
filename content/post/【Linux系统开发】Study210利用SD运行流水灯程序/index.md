---
title: Study210利用SD运行流水灯程序
description: Study210利用SD运行流水灯程序
slug: 【Linux系统开发】Study210利用SD运行流水灯程序
date: 2022-07-24 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---



## 1.安装ecilpse
#### （1）确认自己的PC机开发环境。开发板光盘中有如下四个eclipse包：
```
eclipse-kepler-for-arm-windows-x86_32.7z 
eclipse-kepler-for-arm-windows-x86_64.7z 
eclipse-kepler-for-arm-gtk-linux-x86_64.7z 
eclipse-kepler-for-arm-gtk-linux-x86_32.7z
```
选择自己需求对应的安装包下载解压即可（[此处可点击下载](https://download.csdn.net/download/qq_56914146/85162554)）
#### （2）配置好eclipse的环境变量
借鉴[Eclipse环境变量配置-超详细](https://blog.csdn.net/m0_46165586/article/details/107296429?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165018384616781685349830%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165018384616781685349830&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-107296429.142^v9^pc_search_result_control_group,157^v4^control&utm_term=eclipse%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E9%85%8D%E7%BD%AE&spm=1018.2226.3001.4187)
## 2.开始工程的创建
#### （1）首先双击eclipse.exe文件进入，初次进入需要选择一个存储位置作为工程存放处（workplace）

#### （2）建一个流水灯工程
首先在Project Explorer的空白栏右键单击->New->C Project
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b37eba37269446faf5de58793963da2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
项目名称填写LED_test
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b7cce5b3fa341f98792a316a1937054.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
点击next，finish

找到我们的项目工程示例，将全部文件复制到剪贴板
![在这里插入图片描述](https://img-blog.csdnimg.cn/2bbe14f0a69a46fe8328ce92f1492421.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
工程右键选择paste，选择粘贴全部
![在这里插入图片描述](https://img-blog.csdnimg.cn/842db416ea2c4929ac655d29bc4bfb07.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
这是粘贴好的文件项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/d1dae4dddaf346b1be377c674fefdd35.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
工程右键Build Project或直接CTRL+B编译
![在这里插入图片描述](https://img-blog.csdnimg.cn/9389ca0a332143fdbfb650e41fb84f2b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
此时回到我们存放工程的workplace文件目录下，可以发现生成了output文件目录
![在这里插入图片描述](https://img-blog.csdnimg.cn/776361da8d1045c3a39af79804bf0320.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
进入该目录下，可以发现生成了led.bin映像文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/e497619b0ab74765aaffd33dc55be299.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
## 3.下载源码到SD卡
打开SD卡烧写工具，将上面生成的映像文件下载到SD卡
![在这里插入图片描述](https://img-blog.csdnimg.cn/07008c13a27e41bebc960a47747c2283.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
## 4.实例演示
#### （1）清除开发板中的bootloader
由于S5PV210芯片无法直接从SD2通道启动，首先会从SD0通道启动，而SD0通道接了emmc芯片，因此我们务必将emmc中已存在的bootloader破坏掉！(关于Windows下破坏板载BootLoader方法可借鉴[【Linux系统开发】Study210开发板刷安卓系统](https://blog.csdn.net/qq_56914146/article/details/124204098?spm=1001.2014.3001.5501))

#### （2）通过SD卡运行裸机程序
将烧有裸机程序的SD卡插到Study210开发板上，长按POWER键，约3秒后即可松手，这时可以发现，四盏LED灯已经在来回闪烁了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ddcba636b89841629cc4eba87b6a90b2.gif)
