---
title: 时钟管理（原理+实战）
description: 操作系统需要通过时间来规范其任务的执行，操作系统中最小的时间单位是时钟节拍 (OS Tick)。
slug: 玩转RT-Thread
date: 2022-07-13 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---



## 一、时钟节拍

任何操作系统都需要提供一个时钟节拍， 以供系统处理所有和时间有关的事件，如线程的延时、线程的时间片轮转调度以及定时器超时等。

RT-Thread 中，时钟节拍的长度可以根据 `RT_TICK_PER_SECOND` 的定义来调整，等于 1/RT_TICK_PER_SECOND 秒。也就是说，在RT-Thread中，`系统的时钟节拍频率是由RT_TICK_PER_SECOND决定的！`

rtconfig.h配置文件中定义:

> * 频率是1000HZ周期是1/1000 s
> * 所以节拍是1ms
>
> * #define RT_ _TiCK_ PER_ SECOND 1000

![](https://img-blog.csdnimg.cn/d00c117cba6245909ad5fdab19c9bf74.png)
	

#### 1、void SysTick_Handler()

在RT-Thread中，当系统滴答定时器时间到了的时候，就会执行`void SysTick_Handler`（系统滴答定时器中断处理函数）这个回调函数（中断处理函数）

![](https://img-blog.csdnimg.cn/c97f508be8c844a7bfa08cff57233dc1.png)


> 可以发现在`void SysTick_Handler()`这个函数中，首先会执行中断入口函数，然后`void rt_tick_increase`对`rt_tick`(系统滴答时钟,初值为0，静态`全局变量`)进行自加操作,会记录从启动到现在的时钟节拍数

#### 2、void rt_tick_increase()

![](https://img-blog.csdnimg.cn/b7122a14e60444e29f36e106bfb5166d.png)


`也就是说，系统滴答定时器中断处理函数会每1ms触发一次systick定时器中断 `


#### 3、rt_tick_get(void)；

名称：获取系统统计函数

功能：返回当前操作系统的时钟数

返回值：返回当前时钟数

![](https://img-blog.csdnimg.cn/dd9e18ad7b4942ce9af13f1870fcc232.png)

## 二、定时器管理

#### 1、概念

定时器，是指从指定的时刻开始，经过一定的指定时间后触发一个事件，例如定个时间提醒第二天能够按时起床。定时器有`硬件定时器`和`软件定时器`之分：

1）**硬件定时器**是芯片本身提供的定时功能。`一般是由外部晶振(HSE)提供给芯片输入时钟`，芯片向软件模块提供一组配置寄存器，接受控制输入，到达设定时间值后芯片中断控制器产生时钟中断。硬件定时器的精度一般很高，可以达到纳秒级别，并且是`中断触发方式`。

2）**软件定时器**是由`操作系统提供的一类系统接口`，它构建在硬件定时器基础之上，使系统能够提供不受数目限制的定时器服务。

RT-Thread 操作系统提供软件实现的定时器，以时钟节拍（OS Tick）的时间长度为单位，即`定时数值必须是 OS Tick 的整数倍`，例如一个 OS Tick 是 10ms，那么上层软件定时器只能是 10ms，20ms，100ms 等，而不能定时为 15ms。RT-Thread 的定时器也基于系统的节拍，提供了基于节拍整数倍的定时能力。

#### 2、RT-Thread定时器介绍

RT-Thread 的定时器提供两类定时器机制：

第一类是`单次触发`定时器，这类定时器在启动后只会触发一次定时器事件，然后定时器自动停止。
第二类是`周期触发`定时器，这类定时器会周期性的触发定时器事件，直到`用户手动的停止`，否则将永远持续执行下去。

另外，根据超时函数执行时所处的上下文环境，RT-Thread 的定时器可以分为 `HARD_TIMER `模式（硬件定时器模式）与` SOFT_TIMER` 模式（软件定时器模式），如下图。

![](https://img-blog.csdnimg.cn/36887975b53c4dc684343c70a2f8db09.png)


1）HARD_TIMER 模式：中断上下文

HARD_TIMER 模式的定时器超时函数在中断上下文环境中执行，可以在初始化 / 创建定时器时使用参数` RT_TIMER_FLAG_HARD_TIMER `来指定。

在中断上下文环境中执行时，对于超时函数的要求与中断服务例程的要求相同：`执行时间应该尽量短，执行时不应导致当前上下文挂起、等待`。例如在中断上下文中执行的超时函数它不应该试图去申请动态内存、释放动态内存等。

2）SOFT_TIMER 模式：线程上下文

SOFT_TIMER 模式可配置，通过宏定义 RT_USING_TIMER_SOFT 来决定是否启用该模式。

该模式被启用后，系统会在`初始化时创建一个 timer 线程`，然后 `SOFT_TIMER 模式的定时器超时函数在都会在 timer 线程的上下文环境中执行`。可以在初始化 / 创建定时器时使用参数 `RT_TIMER_FLAG_SOFT_TIMER `来指定设置 `SOFT_TIMER` 模式。

#### 3、定时器源码分析

1）RT-Thread OS 启动阶段，执行rtthread_startup函数，在该函数中调用了定时器初始化函数

![](https://img-blog.csdnimg.cn/0ee79523a62540a7a18155a0d9010365.png)


2）rt_system_timer_init（硬件定时器初始化）

```c
void rt_system_timer_init(void)
{
    int i;// 结构体数组，在初始化的时候只有一个元素，就是链表头，后期添加定时器，按定时器定时时间顺序进行顺序插入
    
    for (i = 0; i < sizeof(rt_timer_list) / sizeof(rt_timer_list[0]); i++)
    {
        rt_list_init(rt_timer_list + i);
    }
}
```

3）rt_system_timer_thread_init（软件定时器初始化）

![](https://img-blog.csdnimg.cn/68e5d39a7cf94149a474f05b0f370717.png)


#### 4、定时器工作机制

下面以一个例子来说明 RT-Thread 定时器的工作机制。在 RT-Thread 定时器模块中维护着两个重要的`全局变量`：

（1）当前系统经过的 tick 时间 rt_tick（当硬件定时器中断来临时，它将加 1）；

（2）定时器链表 rt_timer_list。系统新创建并激活的定时器都会按照`以超时时间排序`的方式`插入到 rt_timer_list 链表`中。

如下图所示，系统当前 tick 值为 20，在当前系统中已经创建并启动了三个定时器，分别是定时时间为 50 个 tick 的 Timer1、100 个 tick 的 Timer2 和 500 个 tick 的 Timer3，这三个定时器分别加上系统当前时间 rt_tick=20，从小到大排序链接在 rt_timer_list 链表中，形成如图所示的定时器链表结构。

![](https://img-blog.csdnimg.cn/bb6521cccf884694867fd32a5c76da27.png)


而 rt_tick 随着硬件定时器的触发一直在增长（每一次硬件定时器中断来临，rt_tick 变量会加 1），`50 个 tick 以后，rt_tick 从 20 增长到 70`，与 `Timer1 的 timeout 值相等`，这时会`触发与 Timer1 定时器相关联的超时函数`，同时将 `Timer1 从 rt_timer_list 链表上删除`。

同理，100 个 tick 和 500 个 tick 过去后，与 Timer2 和 Timer3 定时器相关联的超时函数会被触发，接着将 Timer2 和 Timer3 定时器从 rt_timer_list 链表中删除。

如果系统当前定时器状态在 10 个 tick 以后（rt_tick=30）有一个任务新创建了一个 tick 值为 300 的 Timer4 定时器，由于 Timer4 定时器的 `timeout=rt_tick+300=330`, 因此它将被插入到 Timer2 和 Timer3 定时器中间，形成如下图所示链表结构：

![](https://img-blog.csdnimg.cn/515005f47d5148848a7ea4fe883b6f39.png)


#### 5、定时器相关接口

1）**动态创建定时器**

动态创建声明：

```c
rt_timer_t rt_timer_create(const char *name,
                           void (*timeout)(void *parameter),
                           void       *parameter,
                           rt_tick_t   time,
                           rt_uint8_t  flag);
```

详细函数定义：

![](https://img-blog.csdnimg.cn/87411b4f65564541a22792b8fc676ec1.png)


查看flag定义：

```c
#define RT_TIMER_FLAG_ONE_SHOT          0x0             // 单次触发
#define RT_TIMER_FLAG_PERIODIC          0x2             // 周期性触发

#define RT_TIMER_FLAG_HARD_TIMER        0x0             // 硬件定时器模式
#define RT_TIMER_FLAG_SOFT_TIMER        0x4				// 软件定时器模式
```



> 同时这里我们注意到`rt_timer_create`这个函数的返回值是`rt_timer_t`，通过查找定义可以发现该类型是通过typedef重命名的
>
> 也就是说`struct rt_timer` <=>`*rt_timer_t`

```c
typedef struct rt_timer *rt_timer_t;
```

下面我们也可以详细看到rt_time这个结构体对定时器的一个详细描述

```c
struct rt_timer
{
    struct rt_object parent;                            /**< inherit from rt_object */

    rt_list_t        row[RT_TIMER_SKIP_LIST_LEVEL];

    void (*timeout_func)(void *parameter);              /**< timeout function */
    void            *parameter;                         /**< timeout function's parameter */

    rt_tick_t        init_tick;                         /**< timer timeout tick */
    rt_tick_t        timeout_tick;                      /**< timeout tick */
};
```

2）**删除定时器**

函数声明：

```
rt_err_t rt_timer_delete(rt_timer_t timer);
```

函数返回值：返回操作系统的状态，成功返回0，失败返回1

![](https://img-blog.csdnimg.cn/71ccf0b21d70422fa817b7f9ae7d6843.png)


3）**动态创建定时器演示**

```c
// 主函数

int main(void)
{
    tm = rt_timer_create("tm_demo",tm_callback,NULL,3000, RT_TIMER_FLAG_PERIODIC | RT_TIMER_FLAG_SOFT_TIMER);
    if(tm == RT_NULL)
    {
        LOG_E("rt_timer_create faile...\n");
        return -ENOMEM;
    }

    LOG_D("rt_timer_create successed...\n");
    return 0;
}
```

在这里也可以看到，我们设置了一个名为tm_demo的定时器，设置超时时间为3s，同时flag我们是设置为周期定时和软件定时（flag设置详见上文flag定义	）。

```c
// 编写中断回调函数（超时函数）

void tm_callback(void *parameter)
{

}
```

```c
// 返回值结构图定义

rt_timer_t tm ; 
```

4）**开启定时器**

函数声明：

```c
rt_err_t rt_timer_start(rt_timer_t timer);
```

函数返回值：成功返回0，失败返回1

![](https://img-blog.csdnimg.cn/3542f82f0cc84f4f9c74b871c17753fe.png)


5）实例：

```c
rt_timer_start(tm);
```

此时我们在超时函数中编写代码：

```c
void tm_callback(void *parameter)
{
    rt_kprintf("tm_callback is running...\n");
}
```

此时回到串口查看，就可以发现tm_demo这个定时器已经被激活了，并且定时器的周期和超时时间也都发生改变，由于我们在上面设置的超时时间为3S，所以在串口显示会三秒打印一次信息 

6）**静态创建定时器**

函数定义：

```c
void rt_timer_init(rt_timer_t  timer,
                   const char *name,
                   void (*timeout)(void *parameter),
                   void       *parameter,
                   rt_tick_t   time,
                   rt_uint8_t  flag);
```

这里我们看下`rt_timer_init`这个函数的返回值和参数

返回值：void

参数：

|   参数    |           描述           |
| :-------: | :----------------------: |
|   timer   |      结构体指针类型      |
|   name    |           名字           |
|  timeout  |     超时回调函数指针     |
| parameter | 传递给超时回调函数的参数 |
|   time    |        定时器时间        |
|   flag    |        定时器标志        |



7）**脱离函数**(静态创建时使用)

描述：当`静态创建`的定时器不需要在使用时，我们调用下面这个函数接口

函数声明：

```c
rt_err_t rt_timer_detach(rt_timer_t timer);
```

返回值：成功返回0，失败返回1

8）**定时器控制**

函数声明：

```c
rt_err_t rt_timer_control(rt_timer_t timer, int cmd, void *arg);
```

cmd命令定义查看

```c
#define RT_TIMER_CTRL_SET_TIME          0x0             /**< set timer control command */
#define RT_TIMER_CTRL_GET_TIME          0x1             /**< get timer control command */
#define RT_TIMER_CTRL_SET_ONESHOT       0x2             /**< change timer to one shot */
#define RT_TIMER_CTRL_SET_PERIODIC      0x3             /**< change timer to periodic */
#define RT_TIMER_CTRL_GET_STATE         0x4             /**< get timer run state active or deactive*/
```



![](https://img-blog.csdnimg.cn/cb30a12c60e24f51a5ef081c296347bd.png)


实例：

>查看终端数据，可以发现终端执行顺序为：打印一次tm的中断回调函数信息，然后打印三次tm2的信息。
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/67007c956ebd48478e44b9233abd8efe.png)


```c
#include <rtthread.h>
#include <rtdevice.h>
#include <board.h>
#include <rtdbg.h>

rt_timer_t tm ;
struct rt_timer tm2 ;
int flags = 0;

void tm_callback(void *parameter)
{
    rt_kprintf("tm_callback is running...\n");
}

void tm2_callback(void *parameter)
{
    flags++;
    if(flags == 10)// 当flags标志位位10时，设置单次触发
    {
        rt_timer_control(&tm2, RT_TIMER_CTRL_SET_ONESHOT, NULL);
        flags = 0;
    }
    rt_tick_t timeout = 1000;
    rt_timer_control(&tm2, RT_TIMER_CTRL_SET_TIME, (void *)&timeout);
    rt_kprintf("[%u]tm2_callback is running...\n",rt_tick_get());
}

int main(void)
{
    // 动态创建定时器
    tm = rt_timer_create("tm_demo",tm_callback,NULL,3000, RT_TIMER_FLAG_PERIODIC | RT_TIMER_FLAG_SOFT_TIMER);
    if(tm == RT_NULL)
    {
        LOG_E("rt_timer_create faile...\n");
        return -ENOMEM;
    }

    LOG_D("rt_timer_create successed...\n");
    rt_timer_start(tm);

    // 静态创建定时器
    rt_timer_init(&tm2,"tm2_demo",tm2_callback,NULL,3000, RT_TIMER_FLAG_PERIODIC | RT_TIMER_FLAG_SOFT_TIMER);
    rt_timer_start(&tm2);

    return 0;
}
```

## 三、高精度延时

> `注意:这个函数只支持低于1个OS Tick的延时，否则 SysTick会出现溢出而不能够获
> 得指定的延时时间`

* 函数声明:`void rt_hw_us_delay(rt_uint32_t us);`

![](https://img-blog.csdnimg.cn/43a92b7bdd884f18bf6ce522214cb520.png)


* 应用场景：应用于某些场景下对高精度延时有要求的情况下





