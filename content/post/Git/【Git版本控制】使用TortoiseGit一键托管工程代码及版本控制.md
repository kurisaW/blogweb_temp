---
title: 使用TortoiseGit一键托管工程代码及版本控制
description: TortoiseGit 是 Git 的 Windows Shell 接口，基于 TortoiseSVN。它是开源的，可以完全使用免费提供的软件构建。
slug: Git
date: 2022-07-29 00:00:00+0000
image: cover.jpg
categories:
    - Git
tags:
    - Git
---

#### 一、了解TortoiseGit

TortoiseGit 是 Git 的 Windows Shell 接口，基于 TortoiseSVN。它是开源的，可以完全使用免费提供的软件构建。

由于它不是针对特定 IDE（如 Visual Studio、Eclipse 或其他）的集成，因此您可以将它与您喜欢的任何开发工具以及任何类型的文件一起使用。与 TortoiseGit 的主要交互将使用 Windows 资源管理器的上下文菜单。

TortoiseGit 通过常规任务为您提供支持，例如提交、显示日志、区分两个版本、创建分支和标签、创建补丁等等。

它是在[GPL](https://www.gnu.org/licenses/gpl-2.0)下开发的。这意味着任何人都可以完全免费使用，包括在商业环境中，没有任何限制。源代码也是免费提供的，因此您甚至可以根据需要开发自己的版本。

#### 二、安装GIit及TortoiseGit

* Git下载官网: [https://gitforwindows.org/index.html](https://gitforwindows.org/index.html)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4a21fd0a6bd1453ba31032ce73c67d73.png)


* TortoiseGit下载官网：[https://tortoisegit.org/download/](https://tortoisegit.org/download/)

![image-20220720090147944](https://img-blog.csdnimg.cn/img_convert/ba9793defe4baa02684e53320b79bde4.png)

* 同时下载语言包

![](https://img-blog.csdnimg.cn/img_convert/b9cb668ce4c7a1ba2cda1a6bab5a0a76.png)



当然这里也有百度网盘链接，也可点击下方链接进行下载

链接：[https://pan.baidu.com/s/1eSmu-opC0nzMsL-5GrUHQg?pwd=dzbs](https://pan.baidu.com/s/1eSmu-opC0nzMsL-5GrUHQg?pwd=dzbs)
提取码：dzbs

![image-20220720090721507](https://img-blog.csdnimg.cn/img_convert/db691a46832cc291932ff63fcc3e9a74.png)

#### 三、TortoiseGit配置

完成上述安装后，单击鼠标右键可发现Git及TortoiseGit相关选项

![image-20220720091049882](https://img-blog.csdnimg.cn/img_convert/67fb3421c49f0c5e90e76fa3c04c9b71.png)

这里选择TortoiseGit-Setting(上图已经完成汉化)，选择语言修改为简体中文

![image-20220720091218159](https://img-blog.csdnimg.cn/img_convert/853a58d535b9e2370949ccb7917b80b5.png)

配置用户，用户作为你操作git的个人标识，进入设置，点选左边的Git标签，可以发现,右边可以配置用户的名字与Email信息. 如下图所示：

![image-20220720091439829](https://img-blog.csdnimg.cn/img_convert/e70b76490f6f5cfc699d79df3968e1fe.png)

点击 “编辑全局 .git/config(O)”按钮,会使用记事本打开全局配置文件，在全局配置文件中，在后面加上下面的内容（记住密码）:

```
[credential]
  helper = store
```

完成后保存，关闭记事本，确定即可。

　　则当你使用 HTTPS URL 方式推送项目到GitHub等在线仓库时，海龟git会记住你输入的用户名和密码（这里不是用户的姓名和Email），可以避免每次提交都要输入用户名和密码。

　　如果你编辑的是 本地 .git/config(L)，其实这个翻译为本地有点问题，应该叫局部，也就是在某个项目下面设置，只对此项目有效，配置是一样的。

#### 四、添加GitHub SSH Keys及密钥上传

首先找到想要选择的仓库克隆到本地的一个文件夹，然后找到你们安装TortoiseGit的位置（\TortoiseGit\bin\puttygen.exe），点击Generate生成钥匙，等待进度条结束后，保存公钥和私钥位置（记住位置）

![image-20220720092721281](https://img-blog.csdnimg.cn/img_convert/33d7c521e31040d4921e11ab1b12bd74.png)
![image-20220720093308539](https://img-blog.csdnimg.cn/img_convert/3f27c4682dc82b0bd0f3ef5c89a1df7e.png)



然后复制下方公钥，

![image-20220720093236525](https://img-blog.csdnimg.cn/img_convert/2a5f6daf15cc2e9935764b207b726cf7.png)



打开github，完成下图操作：

![image-20220815182247315](https://img-blog.csdnimg.cn/img_convert/52cf49261a89b3186083dfb7240ea8dd.png)

![image-20220815182404425](https://img-blog.csdnimg.cn/img_convert/1db55857a0118c8cabc02f9a5c4bb19b.png)

![image-20220815182541297](https://img-blog.csdnimg.cn/img_convert/b406ac32656957356de898117c6bb17f.png)

#### 五、使用TortoiseGit提交代码到远端仓库

在Github自建一个仓库（自行选择即可，用于代码托管和版本控制），使用Git clone命令复制到本地文件夹

![image-20220815185026555](https://img-blog.csdnimg.cn/img_convert/846315f99c2e767666f7be83437e4969.png)



鼠标右键可以看到选项`Git在这里创建版本库`，点击创建版本库

![image-20220815185825412](https://img-blog.csdnimg.cn/img_convert/bcb80e2391a690906240fd0d1b197e7a.png)

鼠标右键打开TortoiseGit->设置(Settings)->Git->远端(Remote)，进行如下配置

![image-20220815191405483](https://img-blog.csdnimg.cn/img_convert/a1f9b059fb25b079395ad2588d51c0c5.png)



此时就可以将需要托管的代码放到这个文件夹内，然后进行代码的托管和版本控制了，下面简单做个示范：

![image-20220815190637369](https://img-blog.csdnimg.cn/img_convert/9c4fbf6ee5360e497644e6330255a278.png)

我们创建一个文本文件，可以发现在文件上还有一个附带的图标显示，这分别代表不同的文件状态：

```
正常的：绿色的对号 
被修改过的：红色感叹号 
新添加的：蓝色的加号
未受控的（无版本控制的）：蓝色的问号
忽略不受控的：灰色的减号
删除的：红色的x号 
有冲突的：黄色的感叹号 
```

鼠标右键添加文件

![image-20220815191829438](https://img-blog.csdnimg.cn/img_convert/f146752e79d7bd6a4e9287e2c32ad3c1.png)

![image-20220815191933736](https://img-blog.csdnimg.cn/img_convert/50b75011172011ae724d81bd921f8fdc.png)

![image-20220815192002035](https://img-blog.csdnimg.cn/img_convert/a9acdcdf6d14730de5e91df7564f65f8.png)

![image-20220815192321480](https://img-blog.csdnimg.cn/img_convert/6439911282c3c25bde45fa033f3a58b3.png)



![image-20220815193032859](https://img-blog.csdnimg.cn/img_convert/73d32ab54fc9914ea2d1a967c5b91065.png)



**`注意：由于代理问题，需要开加速器，然后会出现拉取或提交失败，这都是正常现象，多试几次`**



总结：使用TortoiseGit提交代码到远端仓库的步骤（配置完成后）

***`添加->提交->拉取->推送`***



那么以上就是TortoiseGit配置及代码托管的所有教学了，有问题欢迎在评论区或私信提问！
