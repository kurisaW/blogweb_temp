---
title: RT-Thread Studio使用 2.内核实战篇（线程）
description: 在 RT-Thread 中，线程是实现任务的载体，它是 RT-Thread 中最基本的调度单位，它描述了一个任务执行的运行环境，也描述了这个任务所处的优先等级，重要的任务可设置相对较高的优先级，非重要的任务可以设置较低的优先级，不同的任务还可以设置相同的优先级，轮流运行。
slug: 【玩转RT-Thread】 RT-Thread Studio使用 2.内核实战篇（线程）
date: 2022-07-14 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---

详细原理参考：[【玩转RT-Thread】线程管理（详细原理）](https://blog.csdn.net/qq_56914146/article/details/124141250)


## 一、线程创建

#### 1、函数原型

```c
// 线程创建
rt_thread_t rt_thread_create(const char* name,
							 void (*entry)(void* parameter),
							 void* parameter,
							 rt_uint32_t stack_size,
							 rt_uint8_t priority,
							 rt_uint32_t tick);
```

> 首先我们来看看线程创建函数返回值类型：

![](https://img-blog.csdnimg.cn/de83fe0a4aad4ffe9989eacdf86e96df.png)


> 可以看到线程创建函数的返回值类型为：`rt_thread_t`，找到定义处（如下图），可以看到它的返回值类型是一个结构体指针变量。

![](https://img-blog.csdnimg.cn/1dfc5d7964484cae9cc44d8067ffcdd0.png)


#### 2、线程定义

那么我们先定义一个结构体指针的线程th1_ptr，这样通过rt_thread_create函数创建的进程控制块的地址就能直接赋值给th1_ptr变量：

```c
rt_thread_t th1_ptr = NULL
```

> 接下来就是我们给进程控制块传参了

![](https://img-blog.csdnimg.cn/90d481586b964d01958c9a14a2bd4695.png)


![](https://img-blog.csdnimg.cn/7f39dba639bb4c5faeed06524f57a60d.png)


#### 3、线程创建判断

由于线程创建有返回值，所以我们此处再加入一个判断函数去判断线程是否创建成功

我们先来看下线程返回值（如下图）

> 如果`成功创建`的话，返回值是会返回我们所创建的线程对象的

![](https://img-blog.csdnimg.cn/b19a07990b2240728d423f2c7064d47c.png)


> 如果创建失败的话，可以看到是会返回一个RT_NULL，也就是0

![](https://img-blog.csdnimg.cn/faf3ad30a6f046638db1c77a0c8275a4.png)



```c
// 判断	
if(th1_ptr == RT_NULL)
    {
        //错误信息打印
        LOG_E("rt_thread_create create failed...\n");
    	return -RT_ENOMEM; // 设定当线程th1_ptr创建失败后，返回一个空间不足的标志
    }
    
    //打印debug调试信息
    LOG_D("rt_thread_create create successed ...\n");
```



#### 4、线程入口函数

我们在线程的入口处理函数写一个循环函数：

```c
void th_entry(void* parameter)
{
    while(1)
    {
        rt_kprintf("th_entry running ...\n");
        rt_thread_mdelay(1000);
    }
}
```

`注意：我们在使用线程的处理函数的循环函数的时候，一定要记得及时释放资源，也就是出让CPU资源，不然这个线程会一直执行并占用系统资源`

* 编译，串口观察

![](https://img-blog.csdnimg.cn/4c5556830c644e48bfa76008218fb680.png)


由于RTT studio有内置的串口终端，我们直接打开

![](https://img-blog.csdnimg.cn/cd4fd4b573c0421a88a73d9f8e7160dd.png)


终端输入list_thread可以查看所有的线程

![](https://img-blog.csdnimg.cn/edae1f6480c54759915145477406f17d.png)




#### 5、总结

这里也许就有疑问了，为什么线程入口函数的打印命令没有被执行？

其实我们再看th_demo线程的状态可以看到是`init`，参考[【玩转RT-Thread】线程管理（详细原理）](https://blog.csdn.net/qq_56914146/article/details/124141250)

`当线程刚开始创建还没开始运行时就处于初始状态；在初始状态下，线程不参与调度。此状态在RT-Thread 中的宏定义为RT_THREAD_INIT`

其实这句话就表明当`线程处于初始化状态下是不参与系统调度`的！

#### 6、补充

线程错误码：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f32f5440cb604b2d8eea4a4546d977b0.png)



---

## 二、线程启动

函数原型

在主函数中加入命令，使线程进入就绪态：

```
rt_thread_startup(th1_ptr); 
```

但是我们此时打开终端可以发现：线程入口函数虽然被执行，但线程状态为`挂起态`

![](https://img-blog.csdnimg.cn/f5f40f386046488f89e56ab8ee8db6d4.png)


`解释:`虽然我们调用`rt_thread_startup`函数使线程进入就绪态，但是回到入口函数我们可以看到，我们调用了`rt_thread_mdelay`函数使其有一定时间的休眠，从而进入了挂起态`

## 三、初始化线程
`rt_thread_init`

#### 1、函数声明

```c
// 模板函数
rt_err_t rt_thread_init(struct rt_thread* thread,
					    const char* name,
						void (*entry)(void* parameter), void* parameter,
						void* stack_start, rt_uint32_t stack_size,
						rt_uint8_t priority, rt_uint32_t tick);
```

#### 2、函数定义

```c
ret = rt_thread_init(&th2,"th2_demo", th2_entry, NULL, th2_stack, sizeof(th2_stack), 19, 5);
```

> 此处我们需要定义一个ret整型变量用于`rt_thread_init`的返回值传参，然后定义一个线程结构体，用于静态线程传参。同时需要为线程栈分配内存，所以我们创建一个栈数组，注意这里的线程栈大小我们设定512，而线程的优先级设为19，比线程th1_demo要高一个优先级，后续观察现象。

![](https://img-blog.csdnimg.cn/b7ce209e742948a398d235d4fd799279.png)


![](https://img-blog.csdnimg.cn/06a57b94d88c4cdf97f9a431d1578862.png)


#### 3、线程入口函数

代码如下：

```c
void th2_entry(void* parameter)
{
    for(i=0;i<10;i++)
    {
        rt_kprintf("th2_entry running ...\n");
        rt_thread_mdelay(1000);
    }
}
```

#### 4、判断创建状态

静态线程创建成功的话会返回0，失败的话会返回一个负值，若成功创建线程，我们调用`rt_thread_startup`函数使线程2进入就绪态，并执行线程处理函数。

```
if(ret < 0)
    {
        LOG_E("rt2_thread_create create failed ...\n"); // 错误信息打印
        return ret;
    }
    
    LOG_D("rt_thread2_create create successes ...\n"); 
    rt_thread_startup(&th2); // 创建成功后，我们开启线程，使其进入就绪态
```

> 这里注意：由于我们线程2定义是一个数组，所以需要取地址进行线程开启

#### 5、实验结果

> 分析：首先我们把线程1和线程2的启动函数都开启，可以看到线程1和线程2都处于挂起态，线程2的命令先于线程1执行，这是由于前面我们设定优先级给线程2（优先级19）比线程1（优先级20）高，所以在命令执行是先线程2，再线程1。

![](https://img-blog.csdnimg.cn/861754f210a74896a6d63d20fbb629f0.png)


> 线程2在执行完10次循环后就结束进程了，此时在终端再次输入list_thread可以发现线程2已经退出，只剩下线程1还在循环执行

![](https://img-blog.csdnimg.cn/85333e4d1b1942d6a59b10e528797ecc.png)



