---
title:  ubuntu彻底删除通过apt方式安装的程序
description: ubuntu彻底删除通过apt方式安装的程序
slug: ubuntu彻底删除通过apt方式安装的程序
date: 2022-07-16 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---



以删除apache2为例，其它程序也都是这么删...  
1.先通过apt删除程序和相关配置文件

```
sudo apt-get --purge remove apache2
```

2.自动删除不使用的软件包

```
sudo apt-get autoremove
```

3.找出与apache2相关的程序

```
dpkg --get-selections|grep apache2
```

没有就不显示，如果有就删除这些相关的程序

```
sudo apt-get --purge remove xxx
```

4.查看apache2是否还有进程存在

```
ps -ef |grep apache2
```

如果有就杀掉

```
sudo kill -9 8888 //后面接pid号码，用空格隔开
```

5.全局查找和apache2相关的文件，需要一定时间，稍等

```
sudo find / -name apache2*
```

将找到的文件逐个删掉

```
sudo rm -rf /usr/share/bash-completion/completions/apache2ctl
```

这样就彻底删除掉apache2了
