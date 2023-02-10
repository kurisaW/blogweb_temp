---
title: 线程管理
description: 在 RT-Thread 中，线程是实现任务的载体，它是 RT-Thread 中最基本的调度单位，它描述了一个任务执行的运行环境，也描述了这个任务所处的优先等级，重要的任务可设置相对较高的优先级，非重要的任务可以设置较低的优先级，不同的任务还可以设置相同的优先级，轮流运行。
slug: 【玩转RT-Thread】线程管理
date: 2022-07-16 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---

原理实战请查看[【玩转RT-Thread】 RT-Thread Studio使用（2） 内核实战篇（线程）](https://blog.csdn.net/qq_56914146/article/details/124539095)


# 一、序言
在日常生活中，我们通常会将一个大的问题拆分细化，拆开成若干个小问题，通过逐个解决小问题，大问题也就解决了。
同样的在RT-Thread多线程操作系统中，开发人员基于这种分而治之的思想，将一个复杂的应用问题抽象成若干个小的、可调度的、可序列化的程序单元。当合理地划分任务并正确地执行时，这种设计能够让系统满足实时系统的性能及时间的要求。

下面看一个例子：我们的任务是读取传感器上的数据，并将相关数据显示出来。通过拆分结构，我们可以发现主要有两个任务：
> 1.读取数据
2.显示数据

简单来说，就是一个子任务不间断地读取传感器数据，并将数据写到共享内存中，另外一个子任务周期性的从共享内存中读取数据，并将传感器数据输出到显示屏上。
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d57696f2ffc4e859bf8c8c1ffc0789e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
在RT-Thread 中，与上述子任务对应的程序实体就是线程，`线程是实现任务的载体`。
它是RT-Thread中`最基本的调度单位`，它描述了一个任务执行的运行环境，也描述了这个任务所处的优先等级，重要的任务可设置相对较高的`优先级`，非重要的任务可以设置较低的优先级，不同的任务还可以设置相同的优先级，轮流运行。
`上下文：`当线程运行时，它会认为自己是以独占CPU 的方式在运行，线程执行时的运行环境称为上下文，具体来说就是各个变量和数据，包括所有的寄存器变量、堆栈、内存信息等。
# 二、线程管理的功能特点
RT-Thread 线程管理的主要功能是`对线程进行管理和调度`，系统中总共存在两类线程，分别是`系统线程`和`用户线程`。系统线程是由RT-Thread 内核创建的线程，用户线程是由应用程序创建的线程，这两类线程都会从内核对象容器中分配线程对象，当线程被删除时，也会被从对象容器中删除。

如图所示，每个线程都有重要的属性，如线程控制块、线程栈、入口函数等。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5584a47897de430597897d3a6bddd710.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* RT-Thread 的线程调度器是`抢占式`的，主要的工作就是从就绪线程列表中查找最高优先级线程，保证最高优先级的线程能够被运行，最高优先级的任务一旦就绪，总能得到CPU 的使用权。
* 当一个运行着的线程使一个比它优先级高的线程满足运行条件，当前线程的CPU 使用权就被剥夺了，或者说被让出了，高优先级的线程立刻得到了CPU 的使用权。
如果是中断服务程序使一个高优先级的线程满足运行条件，中断完成时，被中断的线程挂起，优先级高的线程开始运行。
* 当调度器调度线程切换时，先将当前线程上下文保存起来，当再切回到这个线程时，线程调度器将该线程上下文（`详细内容可参考`[【操作系统】进程上下文和线程上下文](https://blog.csdn.net/qq_56914146/article/details/124145153)）信息恢复。

# 三、线程的工作机制
## 1.线程控制块
在RT-Thread 中，线程控制块由结构体struct rt_thread 表示，线程控制块是操作系统用于管理线程的一个数据结构，它会存放线程的一些信息，例如优先级、线程名称、线程状态等，也包含线程与线程之间连接用的链表结构，线程等待事件集合等，详细定义如下：
```
/* 线程控制块*/
struct rt_thread
{
	/* rt 对象*/
	char name[RT_NAME_MAX]; /* 线程名称*/
	rt_uint8_t type; /* 对象类型*/
	rt_uint8_t flags; /* 标志位*/
	rt_list_t list; /* 对象列表*/
	rt_list_t tlist; /* 线程列表*/

	/* 栈指针与入口指针*/
	void *sp; /* 栈指针*/
	void *entry; /* 入口函数指针*/
	void *parameter; /* 参数*/
	void *stack_addr; /* 栈地址指针*/
	rt_uint32_t stack_size; /* 栈大小*/

	/* 错误代码*/
	rt_err_t error; /* 线程错误代码*/
	rt_uint8_t stat; /* 线程状态*/

	/* 优先级*/
	rt_uint8_t current_priority; /* 当前优先级*/
	rt_uint8_t init_priority; /* 初始优先级*/
	rt_uint32_t number_mask;
	......
	rt_ubase_t init_tick; /* 线程初始化计数值*/
	rt_ubase_t remaining_tick; /* 线程剩余计数值*/

	struct rt_timer thread_timer; /* 内置线程定时器*/

	void (*cleanup)(struct rt_thread *tid); /* 线程退出清除函数*/
	rt_uint32_t user_data; /* 用户数据*/
};
```
* ` 其中init_priority 是线程创建时指定的线程优先级，在线程运行过程当中是不会被改变的（除非用户
执行线程控制函数进行手动调整线程优先级）。`
* `cleanup 会在线程退出时，被空闲线程回调一次以执行用户设置的清理现场等工作。`
* `最后的一个成员user_data 可由用户挂接一些数据信息到线程控制块中，以提供类似线程私有数据的实现。`
## 2.线程的重要属性
#### （1） 线程栈
* RT-Thread 线程具有独立的栈，当进行线程切换时，会将当前线程的上下文存在栈中，当线程要恢复运行时，再从栈中读取上下文信息，进行恢复。
* 线程栈还用来存放函数中的局部变量：函数中的局部变量从线程栈空间中申请；函数中局部变量初始时从寄存器中分配（ARM 架构），当这个函数再调用另一个函数时，这些局部变量将放入栈中。
* 对于线程第一次运行，可以以手工的方式构造这个上下文来设置一些初始的环境：入口函数（PC 寄存器）、入口参数（R0 寄存器）、返回位置（LR 寄存器）、当前机器运行状态（CPSR 寄存器）。
* 线程栈的增长方向是芯片构架密切相关的，RT-Thread 3.1.0 以前的版本，均只支持栈由高地址向低地址增长的方式，对于ARM Cortex-M 架构，线程栈可构造如下图所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/041732b5e1fd43d5a62382931ad50360.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
#### （2） 线程状态
线程运行的过程中，同一时间内只允许一个线程在处理器中运行，从运行的过程上划分，线程有多种不同的运行状态，如初始状态、挂起状态、就绪状态等。
在RT-Thread 中，线程包含五种状态，操作系统会自动根据它运行的情况来动态调整它的状态。如下表所示：
|状态|描述|
|-|-|
|初始态|当线程刚开始创建还没开始运行时就处于初始状态；在初始状态下，线程不参与调度。此状态在RT-Thread 中的宏定义为RT_THREAD_INIT|
|就绪态|在就绪状态下，线程按照优先级排队，等待被执行；一旦当前线程运行完毕让出处理器，操作系统会马上寻找最高优先级的就绪态线程运行。此状态在RT-Thread 中的宏定义为RT_THREAD_READY|
|运行态|线程当前正在运行。在单核系统中，只有rt_thread_self() 函数返回的线程处于运行状态；在多核系统中，可能就不止这一个线程处于运行状态。此状态在RT-Thread 中的宏定义为RT_THREAD_RUNNING|
|挂起态|也称阻塞态。它可能因为资源不可用而挂起等待，或线程主动延时一段时间而挂起。在挂起状态下，线程不参与调度。此状态在RT-Thread 中的宏定义为RT_THREAD_SUSPEND|
|关闭态|当线程运行结束时将处于关闭状态。关闭状态的线程不参与线程的调度。此状态在RT-Thread 中的宏定义为RT_THREAD_CLOSE|
#### （3） 线程优先级
* RT-Thread 线程的优先级是表示线程被调度的优先程度。每个线程都具有优先级，线程越重要，赋予的优先级就应越高，线程被调度的可能才会越大。

* RT-Thread 最大支持256 个线程优先级(0~255)，数值越小的优先级越高，0 为最高优先级。在一些资源比较紧张的系统中，可以根据实际情况选择只支持8 个或32 个优先级的系统配置；对于ARM Cortex-M系列，普遍采用32 个优先级。最低优先级默认分配给空闲线程使用，用户一般不使用。在系统中，当有比当前线程优先级更高的线程就绪时，当前线程将立刻被换出，高优先级线程抢占处理器运行。
#### （4） 时间片
>每个线程都有时间片这个参数，但时间片仅对优先级相同的就绪态线程有效。系统对优先级相同的就绪态线程采用时间片轮转的调度方式进行调度时，时间片起到约束线程单次运行时长的作用，其单位是一个系统节拍（OS Tick）。

假设有2 个`优先级相同的就绪态线程A 与B`，A 线程的时间片设置为10，B 线程的时间片设置为5，那么当系统中不存在比A 优先级高的就绪态线程时，系统会在A、B 线程间来回切换执行，并且每次对A 线程执行10 个节拍的时长，对B 线程执行5 个节拍的时长，如下图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/31c4fb41bf8947c1a47864b12b9e602e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
#### （5） 线程的入口函数
线程控制块中的`entry`是线程的入口函数，它是线程实现预期功能的函数。

线程的入口函数由用户设计实现，一般有以下两种代码形式：
1.`无限循环模式`
> 在实时系统中，线程通常是被动式的：这个是由实时系统的特性所决定的，实时系统通常总是等待外
界事件的发生，而后进行相应的服务：
```
void thread_entry(void* paramenter)
{
while (1)
	{
		/* 等待事件的发生*/
		/* 对事件进行服务、进行处理*/
	}
}
```
> 作为一个实时系统，一个优先级明确的实时系统，如果一个线程中的程序陷入了死循环操作，那么比它优先级低的线程都将不能够得到执行。
> 所以在实时操作系统中必须注意的一点就是：<strong>线程中不能陷入死循环操作，必须要有让出CPU使用权的动作，如循环中调用延时函数或者主动挂起。用户设计这种无线循环的线程的目的，就是为了让这个线程一直被系统循环调度运行，永不删除。</strong>

2.`顺序执行或有限次循环模式`
> 如简单的顺序语句、do whlie() 或for() 循环等，此类线程不会循环或不会永久循环，可谓是“一次性”线程，一定会被执行完毕。在执行完毕后，线程将被系统自动删除。
```
static void thread_entry(void* parameter)
{
	/* 处理事务#1 */
	…
	/* 处理事务#2 */
	…
	/* 处理事务#3 */
}
```
#### （6） 常见的线程错误码
```
#define RT_EOK 0 /* 无错误*/
#define RT_ERROR 1 /* 普通错误*/
#define RT_ETIMEOUT 2 /* 超时错误*/
#define RT_EFULL 3 /* 资源已满*/
#define RT_EEMPTY 4 /* 无资源*/
#define RT_ENOMEM 5 /* 无内存*/
#define RT_ENOSYS 6 /* 系统不支持*/
#define RT_EBUSY 7 /* 系统忙*/
#define RT_EIO 8 /* IO 错误*/
#define RT_EINTR 9 /* 中断系统调用*/
#define RT_EINVAL 10 /* 非法参数*/
```
## 3.线程状态切换
RT-Thread 提供一系列的操作系统调用接口，使得线程的状态在这五个状态之间来回切换。几种状态间的转换关系如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/3c79cc6198144f02b4830d4521384c30.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
> * 线程通过调用函数`rt_thread_create/init()` 进入到初始状态`（RT_THREAD_INIT）`；
> * 初始状态的线程通过调用函数`rt_thread_startup()` 进入到就绪状态`（RT_THREAD_READY）`；
>* 就绪状态的线程被调度器调度后进入运行状态`（RT_THREAD_RUNNING）`；
>* 当处于运行状态的线程调用rt_thread_delay()，rt_sem_take()，rt_mutex_take()，rt_mb_recv() 等函数或者获取不到资源时， 将进入到挂起状态`（RT_THREAD_SUSPEND）`；

>* 处于挂起状态的线程，如果等待超时依然未能获得资源或由于其他线程释放了资源，那么它将返回到就绪状态。
>* 挂起状态的线程，如果调用`rt_thread_delete/detach() `函数，将更改为关闭状态`（RT_THREAD_CLOSE）`；
> * 而运行状态的线程，如果运行结束，就会在线程的最后部分执行`rt_thread_exit() `函数，将状态更改为关闭状态。

<strong>`!!! note “注意事项” RT-Thread 中，实际上线程并不存在运行状态，就绪状态和运行状态是等同的。`
## 4.系统线程
系统线程是指由系统创建的线程，用户线程是由用户程序调用线程管理接口创建的线程，在RT-Thread 内核中的系统线程有空闲线程和主线程。
#### （1）空闲线程
`空闲线程`是系统创建的最低优先级的线程，线程状态`永远为就绪态`。当系统中无其他就绪线程存在时，调度器将调度到空闲线程，它通常是一个死循环，且永远不能被挂起。

另外，空闲线程在RT-Thread 也有着它的特殊用途：
* 若某线程运行完毕，系统将自动删除线程：自动执行rt_thread_exit() 函数，先将该线程从系统就绪队列中删除，再将该线程的状态更改为关闭状态，不再参与系统调度，然后挂入rt_thread_defunct 僵尸队列（资源未回收、处于关闭状态的线程队列）中，最后空闲线程会回收被删除线程的资源。
* 空闲线程也提供了接口来运行用户设置的钩子函数，在空闲线程运行时会调用该钩子函数，适合钩入功耗管理、看门狗喂狗等工作。（关于[钩子函数](https://blog.csdn.net/u010132177/article/details/110704721?ops_request_misc=&request_id=&biz_id=102&utm_term=%E9%92%A9%E5%AD%90%E5%87%BD%E6%95%B0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-110704721.nonecase&spm=1018.2226.3001.4187)和[看门狗](https://blog.csdn.net/as480133937/article/details/99121645?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164983700616780269879215%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164983700616780269879215&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-99121645.142^v7^article_score_rank,157^v4^control&utm_term=%E7%9C%8B%E9%97%A8%E7%8B%97&spm=1018.2226.3001.4187)不懂的可以看这里）
#### （2） 主线程
在系统启动时，系统会创建main 线程，它的入口函数为main_thread_entry()，用户的应用入口函数main() 就是从这里真正开始的，系统调度器启动后，main 线程就开始运行。

过程如下图，用户可以在main() 函数里添加自己的应用程序初始化代码。
![在这里插入图片描述](https://img-blog.csdnimg.cn/b08911ca57334476945d473ab814f006.png)
# 四、线程的管理方式
可以使用rt_thread_create() 创建一个动态线程，使用rt_thread_init() 初始化一个静态线程。

动态线程与静态线程的区别是：动态线程是系统自动从动态内存堆上分配栈空间与线程句柄（初始化heap 之后才能使用create 创建动态线程），静态线程是由用户分配栈空间与线程句柄。

下图描述了线程的相关操作，包含：创建/ 初始化线程、启动线程、运行线程、删除/ 脱离线程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/fea91f8134b648ccaefeb5cf201de15f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
## 1.创建和删除线程
#### （1）创建线程
一个线程要成为可执行的对象，就必须由操作系统的内核来为它创建一个线程。可以通过如下的接口创建一个动态线程：
```
rt_thread_t rt_thread_create(const char* name,
							 void (*entry)(void* parameter),
							 void* parameter,
							 rt_uint32_t stack_size,
							 rt_uint8_t priority,
							 rt_uint32_t tick);
```
>调用这个函数时，系统会从动态堆内存中分配一个线程句柄以及按照参数中指定的栈大小从动态堆内存中分配相应的空间。分配出来的栈空间是按照rtconfig.h 中配置的RT_ALIGN_SIZE 方式对齐。

线程创建rt_thread_create() 的参数和返回值见下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f1ba4c76de384fcc91060025bff4bef7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
#### （2）删除线程
对于一些使用rt_thread_create() 创建出来的线程，当不需要使用，或者运行出错时，我们可以使用下面的函数接口来从系统中把线程完全删除掉：
	
	rt_err_t rt_thread_delete(rt_thread_t thread);
调用该函数后，线程对象将会被移出线程队列并且从内核对象管理器中删除，线程占用的堆栈空间也会被释放，收回的空间将重新用于其他的内存分配。实际上，用rt_thread_delete() 函数删除线程接口，仅仅是把相应的线程状态更改为RT_THREAD_CLOSE 状态，然后放入到rt_thread_defunct 队列中；而真正的删除动作（释放线程控制块和释放线程栈）需要到下一次执行空闲线程时，由空闲线程完成最后的线程删除动作。

线程删除rt_thread_delete() 接口的参数和返回值见下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/eb9a07efdd95454c8873506f5178d1ff.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
`这个函数仅在使能了系统动态堆时才有效（即RT_USING_HEAP 宏定义已经定义了）。`
## 2.初始化和脱离线程
#### （1）初始化线程
`线程的初始化`可以使用下面的函数接口完成，来初始化静态线程对象：
```
rt_err_t rt_thread_init(struct rt_thread* thread,
					    const char* name,
						void (*entry)(void* parameter), void* parameter,
						void* stack_start, rt_uint32_t stack_size,
						rt_uint8_t priority, rt_uint32_t tick);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c1e6ca6ef6b84ebe93d0888ff71332af.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
#### （2）脱离线程
对于用rt_thread_init() 初始化的线程，使用rt_thread_detach() 将使线程对象在线程队列和内核对象管理器中被脱离。线程脱离函数如下：
	
	rt_err_t rt_thread_detach (rt_thread_t thread);

|参数|描述|
|-|-|
thread|线程句柄，它应该是由rt_thread_init 进行初始化的线程句柄。
|返回|---|
RT_EOK|线程脱离成功
-RT_ERROR|线程脱离失败
## 3.启动线程
创建（初始化）的线程状态处于初始状态，并未进入就绪线程的调度队列，我们可以在线程初始化/创建成功后调用下面的函数接口让该线程进入就绪态：

	rt_err_t rt_thread_startup(rt_thread_t thread);

![在这里插入图片描述](https://img-blog.csdnimg.cn/431f38ae0feb46dc833f9835ec57577e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)

> 当调用这个函数时，将把线程的状态更改为就绪状态，并放到相应优先级队列中等待调度。如果新启
动的线程优先级比当前线程优先级高，将立刻切换到这个线程。

## 4.获得当前线程
在程序的运行过程中，相同的一段代码可能会被多个线程执行，在执行的时候可以通过下面的函数接口获得当前执行的线程句柄：
	
	rt_thread_t rt_thread_self(void);
![在这里插入图片描述](https://img-blog.csdnimg.cn/33c1e79ddc8b42c0a28237a91e67b92c.png)
## 5.使线程出让处理器资源
> 当前线程的时间片用完或者该线程主动要求让出处理器资源时，它将不再占有处理器，调度器会选择相同优先级的下一个线程执行。线程调用这个接口后，这个线程仍然在就绪队列中。

线程让出处理器使用下面的函数接口：

	rt_err_t rt_thread_yield(void);
> 调用该函数后，当前线程首先把自己从它所在的就绪优先级线程队列中删除，然后把自己挂到这个优先级队列链表的尾部，然后激活调度器进行线程上下文切换（如果当前优先级只有这一个线程，则这个线程继续执行，不进行上下文切换动作）。
## 6.使线程睡眠
> 在实际应用中，我们有时需要让运行的当前线程延迟一段时间，在指定的时间到达后重新运行，这就叫做“线程睡眠”。

线程睡眠可使用以下三个函数接口：
```
rt_err_t rt_thread_sleep(rt_tick_t tick);
rt_err_t rt_thread_delay(rt_tick_t tick);
rt_err_t rt_thread_mdelay(rt_int32_t ms);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6ebefaeea61f48a7b26d43e0d6589055.png)
## 7.挂起和恢复线程
#### （1）线程挂起
> * 当线程调用rt_thread_delay() 时，线程将主动挂起；当调用rt_sem_take()，rt_mb_recv() 等函数时，资源不可使用也将导致线程挂起。
>* 处于挂起状态的线程，如果其等待的资源超时（超过其设定的等待时间），那么该线程将不再等待这些资源，并返回到就绪状态；或者，当其他线程释放掉该线程所等待的资源时，该线程也会返回到就绪状态。

`线程挂起`使用下面的函数接口：

	rt_err_t rt_thread_suspend (rt_thread_t thread);
![在这里插入图片描述](https://img-blog.csdnimg.cn/eed2dcfd794040798637c029b253fb87.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
`!!! note “注意事项” 通常不应该使用这个函数来挂起线程本身， 如果确实需要采用rt_thread_suspend() 函数挂起当前任务， 需要在调用rt_thread_suspend() 函数后立刻调用rt_schedule() 函数进行手动的线程上下文切换。`

#### （2）恢复线程
> 恢复线程就是让挂起的线程重新进入就绪状态，并将线程放入系统的就绪队列中；如果被恢复线程在
所有就绪态线程中，位于最高优先级链表的第一位，那么系统将进行线程上下文的切换。

`线程恢复`使用下面的函数接口：

	rt_err_t rt_thread_resume (rt_thread_t thread);

![在这里插入图片描述](https://img-blog.csdnimg.cn/0640fc789a1645d3b424f605d3aca937.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
## 8.控制线程
当需要对线程进行一些其他控制时，例如动态更改线程的优先级，可以调用如下函数接口：

	rt_err_t rt_thread_control(rt_thread_t thread, rt_uint8_t cmd, void* arg);

![在这里插入图片描述](https://img-blog.csdnimg.cn/348eb561864549528d9695d8f504af92.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)

	指示控制命令cmd 当前支持的命令包括：
	•RT_THREAD_CTRL_CHANGE_PRIORITY：动态更改线程的优先级；
	•RT_THREAD_CTRL_STARTUP：开始运行一个线程，等同于rt_thread_startup() 函数调用；
	•RT_THREAD_CTRL_CLOSE：关闭一个线程，等同于rt_thread_delete() 函数调用。
## 设置和删除空闲钩子
> 空闲钩子函数是空闲线程的钩子函数，如果设置了空闲钩子函数，就可以在系统执行空闲线程时，自动执行空闲钩子函数来做一些其他事情，比如系统指示灯。

设置/ 删除空闲钩子的接口如下：	

	rt_err_t rt_thread_idle_sethook(void (*hook)(void));
	rt_err_t rt_thread_idle_delhook(void (*hook)(void));
|参数|描述|
|-|-|
|hook|设置/删除的钩子函数|
|返回|---|
|RT-EOK|设置/删除成功|
-RT_EFULL|设置失败
-RT_ENOSYS|删除失败

`!!! note “注意事项” 空闲线程是一个线程状态永远为就绪态的线程，因此设置的钩子函数必须保证空闲线程在任何时刻都不会处于挂起状态，例如rt_thread_delay()，rt_sem_take() 等可能会导致线程挂起的函数都不能使用。`

## 10.设置调度器钩子
`在整个系统的运行时，系统都处于线程运行、中断触发- 响应中断、切换到其他线程，甚至是线程间的切换过程中，或者说系统的上下文切换是系统中最普遍的事件。有时用户可能会想知道在一个时刻发生了什么样的线程切换，可以通过调用下面的函数接口设置一个相应的钩子函数。`

在系统线程切换时，这个钩子函数将被调用：

	void rt_scheduler_sethook(void (*hook)(struct rt_thread* from, struct rt_thread* to));

`设置调度器钩子函数的输入参数`如下表所示：
|参数|描述|
|-|-|
hook|表示用户定义的钩子函数指针

`钩子函数hook() 的声明`如下：
 	 		
 	void hook(struct rt_thread* from, struct rt_thread* to);

|参数|描述|
|-|-|
from|表示系统所要切换出的线程控制块指针
to|表示系统所要切换到的线程控制块指针

`!!! note “注意事项” 请仔细编写你的钩子函数，稍有不慎将很可能导致整个系统运行不正常（在这个
钩子函数中，基本上不允许调用系统API，更不应该导致当前运行的上下文挂起）。`

---
资料参考：
（1）[【STM32】HAL库 STM32CubeMX教程五----看门狗(独立看门狗,窗口看门狗)](https://blog.csdn.net/as480133937/article/details/99121645?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164983700616780269879215%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164983700616780269879215&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-99121645.142%5Ev7%5Earticle_score_rank,157%5Ev4%5Econtrol&utm_term=%E7%9C%8B%E9%97%A8%E7%8B%97&spm=1018.2226.3001.4187)
（2）[什么是钩子函数](https://blog.csdn.net/u010132177/article/details/110704721?ops_request_misc=&request_id=&biz_id=102&utm_term=%E9%92%A9%E5%AD%90%E5%87%BD%E6%95%B0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-110704721.nonecase&spm=1018.2226.3001.4187)
（3）[RT-Thread文档中心](https://www.rt-thread.org/document/site/#/)
