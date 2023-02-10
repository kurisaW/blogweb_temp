---
title:  ubuntu安装交叉编译工具链
description: ubuntu安装交叉编译工具链
slug: 【Linux系统开发】ubuntu下安装交叉编译链
date: 2022-07-14 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---

# ubuntu安装交叉编译工具链（附避坑指南）

> 1.打开Ubuntu，在终端进入/usr/local/目录下

	cd /usr/local/

> 2.在local/目录下创建一个名为arm的文件夹

	mkdir arm

>3.在自己的共享文件夹下找到[arm-2009q3.tar.bz2](https://download.csdn.net/download/qq_56914146/85094381),并复制到之前创建的arm目录下

	cp /mnt/hgfs/Myshare/arm-2009q3.tar.bz2 /usr/local/arm/

> 4.进入到arm目录下，解压该其中文件

	cd /usr/local/arm
	tar -jxvf arm-2009q3.tar.bz2	
>5.然后执行：

	cd arm-2009q3/bin
	./arm-none-linux-gnueabi-gcc -v

`注意：`<font color=deepskyblue>这里如果输入`./arm-none-linux-gnueabi-gcc -v`终端显示 ‘没有这样的文件存在’ ，这是因为在64位的系统下安装32位交叉编译工具链，会无法使用，所以我们需要安装32位库的支持

	sudo apt-get install libc6:i386
<font color=deepskyblue>安装好了之后重新输入`./arm-none-linux-gnueabi-gcc -v`
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0660902aed64a88a257ed92b892b8f7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
<font color=deepskyblue>操作成功！

>6.为了能让它其他目录中也可以这么操作，我们把它导出到环境变量中
打开配置文件

	sudo vim /etc/profile

>7.在vi界面末尾处加入

	export PATH=$PATH:/usr/local/arm/arm-2009q3/bin

>8.回到主目录，查看交叉编译工具是否可用

	cd ~
	source /etc/profile

`注` <font color=broen>这里如果没有出现相关信息，切换root用户再次输入命令

使用	`echo $PATH`查看交叉编译链的安装路径是否加入了环境变量。
使用`arm-linux-gnueabihf-gcc -v`测试交叉编译链是否好使


>9.建立一个符号链接，进入到/usr/local/arm/arm-2009q3/bin#目录下，vi新建一个[mk-arm-linux-.sh]脚本（文章最后可复制粘贴该脚本），然后输入命令：

	chmod 777 mk-arm-linux-.sh
	./mk-arm-linux-.sh
`这里由于运行时报错，原因详见`[解决linux的-bash: ./xx.sh: Permission denied](https://blog.csdn.net/LWJdear/article/details/79868551?ops_request_misc=&request_id=&biz_id=102&utm_term=bash:%20./mk-arm-linux-.sh:%20Perm&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-79868551.142^v7^pc_search_result_control_group,157^v4^control&spm=1018.2226.3001.4187)
	
>ls查看，可以发现符号链接出现,到此，交叉编译链配置成功！

![在这里插入图片描述](https://img-blog.csdnimg.cn/6dc86a581621467d8639643cc154877a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)

---
`附件`:
* [arm-2009q3.tar.bz2](https://download.csdn.net/download/qq_56914146/85094381)

* `mk-arm-linux-.sh脚本文件`
```
ln arm-none-linux-gnueabi-addr2line -s arm-linux-addr2line
ln arm-none-linux-gnueabi-ar -s arm-linux-ar
ln arm-none-linux-gnueabi-as -s arm-linux-as
ln arm-none-linux-gnueabi-c++ -s arm-linux-c++
ln arm-none-linux-gnueabi-c++filt -s arm-linux-c++filt
ln arm-none-linux-gnueabi-cpp -s arm-linux-cpp
ln arm-none-linux-gnueabi-g++ -s arm-linux-g++
ln arm-none-linux-gnueabi-gcc -s arm-linux-gcc
ln arm-none-linux-gnueabi-gcc-4.4.1 -s arm-linux-gcc-4.4.1
ln arm-none-linux-gnueabi-gcov -s arm-linux-gcov
ln arm-none-linux-gnueabi-gdb -s arm-linux-gdb
ln arm-none-linux-gnueabi-gdbtui -s arm-linux-gdbtui
ln arm-none-linux-gnueabi-gprof -s arm-linux-gprof
ln arm-none-linux-gnueabi-ld -s arm-linux-ld
ln arm-none-linux-gnueabi-nm -s arm-linux-nm
ln arm-none-linux-gnueabi-objcopy -s arm-linux-objcopy
ln arm-none-linux-gnueabi-objdump -s arm-linux-objdump
ln arm-none-linux-gnueabi-ranlib -s arm-linux-ranlib
ln arm-none-linux-gnueabi-readelf -s arm-linux-readelf
ln arm-none-linux-gnueabi-size -s arm-linux-size
ln arm-none-linux-gnueabi-sprite -s arm-linux-sprite
ln arm-none-linux-gnueabi-strings -s arm-linux-strings
ln arm-none-linux-gnueabi-strip -s arm-linux-strip
```

---
`有问题欢迎评论留言致信:`[blogs](https://blog.csdn.net/qq_56914146?type=blog)
