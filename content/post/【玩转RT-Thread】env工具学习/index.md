---
title: env工具学习
description: Env 是 RT-Thread 推出的开发辅助工具，包括配置器和包管理器。开发者可以使用 Env 工具对 RT-Thread 内核和组件的功能进行配置，对组件进行自由裁剪，对线上软件包进行管理，使得系统以搭积木的方式进行构建，简单方便。
slug: 【玩转RT-Thread】env工具学习
date: 2022-05-12 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
    - env
---

#### 一、基础配置

1.首先需要下载git并配置好相应的环境变量

2.双击env，在setting中设置

![在这里插入图片描述](https://img-blog.csdnimg.cn/709a490d50c24945a2b06409d51b3209.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


这样就可以指定文件夹打开env工具了

#### 二、基本命令学习

1.scons：编译

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-fPYlwcMS-1649693722218)(C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20220411234217601.png)\]](https://img-blog.csdnimg.cn/976aba29a258481d9128f4b984f78139.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)

`（1）scons:`编译并打印相关内部信息
`（2）scons -c:`清除编译目标。这个命令会清除执行 scons 时生成的临时文件和目标文件。
`（3）scons -s:`编译而不打印具体的内部命令
`（4）scons --target=XXX:`使用以下命令中的其中一种重新生成对应的定制化的工程，然后在 mdk/iar 进行编译下载

```
scons --target=iar
scons --target=mdk4
scons --target=mdk5
```

`（5）scons -jN:`多线程编译目标，在多核计算机上可以使用此命令加快编译速度

```
scons -j4 //双核编译工程
```

<font color=red>注意：一般不建议使用，容易将编译信息和错误混杂</font>
`（6）scons --dist:`搭建项目框架，使用此命令会在 BSP 目录下生成 dist 目录
2.指定编译器安装路径

````
set RTT_CC=keil
set RTT_EXEC_PATH=C:/Keilv5
````

3.menuconfig 
打开菜单配置界面，可用户自定义模块

4.scons进阶学习
scons内置函数

* GetCurrentDir()：
  获取当前路径。

* Glob('*.c')：
  获取当前目录下的所有 C 文件。修改参数的值为其他后缀就可以匹配当前目录下的所有某类型的文件。

* GetDepend(macro)：
  该函数定义在 tools 目录下的脚本文件中，它会从 rtconfig.h 文件读取配置信息，其参数为 rtconfig.h 中的宏名。如果 rtconfig.h 打开了某个宏，则这个方法（函数）返回真，否则返回假。

* Split(str)：
  将字符串 str 分割成一个列表 list。

* DefineGroup(name， src， depend，**parameters)：
  这是 RT-Thread 基于 SCons 扩展的一个方法（函数）。DefineGroup 用于定义一个组件。组件可以是一个目录（下的文件或子目录），也是后续一些 IDE 工程文件中的一个 Group 或文件夹。
  `DefineGroup()`  函数的参数描述：

| <strong>参数</strong> |                    <strong>描述</strong>                     |
| --------------------- | :----------------------------------------------------------: |
| name                  |                         Group 的名字                         |
| src                   | Group 中包含的文件，一般指的是 C/C++ 源文件。方便起见，也能够通过 Glob 函数采用通配符的方式列出 SConscript 文件所在目录中匹配的文件 |
| depend                | Group 编译时所依赖的选项（例如 FinSH 组件依赖于 RT_USING_FINSH 宏定义）。编译选项一般指 rtconfig.h 中定义的 RT_USING_xxx 宏。当在 rtconfig.h 配置文件中定义了相应宏时，那么这个 Group 才会被加入到编译环境中进行编译。如果依赖的宏并没在 rtconfig.h 中被定义，那么这个 Group 将不会被加入编译。相类似的，在使用 scons 生成为 IDE 工程文件时，如果依赖的宏未被定义，相应的 Group 也不会在工程文件中出现 |
| parameters            |   配置其他参数，可取值见下表，实际使用时不需要配置所有参数   |

parameters可加入的参数：

| <strong>参数</strong> | <strong>描述</strong>                  |
| --------------------- | -------------------------------------- |
| dirs                  | SConscript 文件路径                    |
| variant_dir           | 指定生成的目标文件的存放路径           |
| duiplicate            | 设定是否拷贝或链接源文件到 variant_dir |

