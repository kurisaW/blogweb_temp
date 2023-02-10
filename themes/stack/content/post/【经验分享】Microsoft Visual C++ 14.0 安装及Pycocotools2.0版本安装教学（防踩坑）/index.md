---
title: Microsoft Visual C++ 14.0 安装及Pycocotools2.0版本安装教学（防踩坑）
description: Microsoft Visual C++ 14.0 安装及Pycocotools2.0版本安装教学
slug: 【经验分享】Microsoft Visual C++ 14.0 安装及Pycocotools2.0版本安装教学（防踩坑）
date: 2022-04-12 00:00:00+0000
image: cover.jpg
categories:
    - experience_sharing
tags:
    - experience sharing
---



## 1、Microsoft Visual C++ 14.0安装

这里附上百度网盘下载链接：
链接: [https://pan.baidu.com/s/1t5GWGymN6mFHDNlgrmD0yw?pwd=ec88](https://pan.baidu.com/s/1t5GWGymN6mFHDNlgrmD0yw?pwd=ec88) 提取码: ec88

下载完成后双击打开
![在这里插入图片描述](https://img-blog.csdnimg.cn/04150c3c158c4baab13f0646ab6bb578.png)


默认下载方式即可 

---

## 2、Pycocotools2.0版本安装

#### （1）准备材料：

* [下载pycocotools安装包](https://github.com/cocodataset/cocoapi)（可直接git拉取到本地文件夹）

#### （2）源码配置

打开下载好的pycocotools，双击打开`setup.py`（文件路径：\cocoapi\PythonAPI\setup.py）

![在这里插入图片描述](https://img-blog.csdnimg.cn/985a37a42b1043bc9dc87d3c3e4e1d0f.png)


这里`将蓝色部分删除，只保留红色部分`(切记需要执行这一步！！！)

开始界面找到所有应用并打开`Anaconda Powershell Prompt`

![](https://img-blog.csdnimg.cn/9ad6e210127c49c9bfb16f9fd9b65968.png)


先打开自己创建的虚拟环境，这里我的虚拟环境为python_env，可供参考。

如上图所示进入到`\cocoapi\PythonAPI`该目录下

分别执行以下两个命令：

```python
python setup.py build_ext --inplace
python setup.py build_ext install
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/02ec23e44b3848609cc74c8c28368a0f.png)


执行pip list查看

![](https://img-blog.csdnimg.cn/642adca979d64daba7f8d2164e88443c.png)
此时回到`\cocoapi\PythonAPI`目录下，可以看到生成了相关文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/bdde5563cb794cf1962800f4656b71f5.png)
将`pycocotools`和`pycocotools.egg-info`文件夹复制到你所创建的虚拟环境中（位置：Anaconda3->envs->python_env->Lib->site-packages）
![在这里插入图片描述](https://img-blog.csdnimg.cn/8dd05ee9c4724de4a2ba536ad84aec81.png)


至此所有问题解决！
