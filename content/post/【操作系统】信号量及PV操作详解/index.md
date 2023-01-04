---
title: 信号量及PV操作详解
description: 无论是大部分的教材上的信号量，还是博客中的信号量，基本上解说都是类似下面这种，给出几个不同的信号量种类然后加一点说明，完全不能理解信号量的PV操作。本文基于此对信号量进行详细叙述，希望能对你有所帮助！
slug: 【操作系统】信号量及PV操作详解
date: 2022-03-10 00:00:00+0000
image: cover.jpg
categories:
    - operating system
tags:
    - operating system
---

## 信号量

>* <font color=deepskyblue>一个特殊变量
>* <font color=deepskyblue>用于进程间传递信息的一个整数值

定义如下：
```
struct semaphore
{
	int count;
	quenue Type quenue;
}
```
> * <font color=indianred>信号量说明:semaphore s;
> * <font color=indianred>对信号量可以实施的操作：初始化、P和V（P、V分别是荷兰语的test（proberen）和increment（verhogen））

## P、V操作定义

<font color=red>P(s)</font>
```
{
	s.count --; //信号量值减一
	if(s.count<0)
	{
		该进程状态置为阻塞态；
		将该进程插入相应的等待队列s.quenue末尾；
		重新调度
	}
}
```
`down`,`semwait`:也代表P操作

<font color=red>V(s)</font>
```
{
	s.ount++;
	if(s.count<=0)
	{
	唤醒相应等待队列s.queue中等待的一个进程；
	改变其状态为就绪态，并将其插入就绪队列；
	}
}
```
`up`,`semsignal`:也代表V操作

>相关说明
> * <font color=Darkblue>P，V操作为原语操作
> * <font color=Darkblue>在信号量上定义了三个操作
<font color=red>初始化（非负数）、P操作、V操作</font>
> * <font color=Darkblue>最初提出的是二元信号量（解决互斥）
> 之后，推广到一般信号量（多值）或技术信号量（解决同步）

## 用PV操作解决进程间互斥问题
>* <font color=coral>分析并发进程的关键活动，划定临界区
>* <font color=coral>设置信号量mutux，初值为1
>* <font color=coral>在临界区前实施P(mutux)
>* <font color=coral>在临界区之后实施V(mutux)

![图片演示](https://img-blog.csdnimg.cn/9f7d7a26cf6a41048d7205423383fa64.png)
>相关解释：

* `临界区` : 我们把并发进程中与共享变量有关的程序段称为临界区。
* `信号量` : 信号量的值与相应资源的使用情况有关。当它的值大于0时，表示当前可用资源的数量；当它的值小于0时，其绝对值表示等待使用该资源的进程个数。
* `进程的互斥`:是指当有若干个进程都要使用某一共享资源时，任何时刻最多只允许一个进程去使用该资源，其他要使用它的进程必须等待，直到该资源的占用着释放了该资源。
* `进程的同步`:是指在并发进程之间存在这一种制约关系，一个进程依赖另一个进程的消息，当一个进程没有得到另一个进程的消息时应等待，直到消息到达才被唤醒。

* `pv操作又称wait,signal原语。`
主要是操作进程中对进程控制的信息量的加减控制。

> <strong>`注意：`在霍尔管程中,`wait操作`和`signal操作`用于被设计为两个可以中断的过程，而非`原语。`
> <strong>在管程中，引入一种数据结构—条件变量（仅在管程中可以被访问）。
> 条件变量的两种操作：
> * wait()操作<font color=red>[阻塞调用进程]</font>
> * signal()操作<font color=red>[释放/唤醒在条件变量上阻塞的进程]</font>

* wait用法：
wait(num),num是目标参数，wait的作用是使其（信息量）减一。
如果信息量>=0，则该进程继续执行；否则该进程置为等待状态，排入等待队列。
signal用法：
signal(num),num是目标参数，signal的作用是使其（信息量）加一。
如果信息量>0，则该进程继续执行；否则释放队列中第一个等待信号量的进程。

>本文资源来自[Operating Systems](https://www.coursera.org/learn/os-pku)
>参考：[操作系统P,V(wait,signal原语)操作讲解](https://blog.csdn.net/thebestway/article/details/105034840?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164992015416780255296134%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=164992015416780255296134&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-105034840.142^v8^pc_search_result_cache,157^v4^control&utm_term=wait%E5%92%8Csignal%E5%8E%9F%E8%AF%AD&spm=1018.2226.3001.4187)
