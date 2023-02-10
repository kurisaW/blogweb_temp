---
title: 线程间同步 信号量
description: 信号量是一种轻型的用于解决线程间同步问题的内核对象，线程可以获取或释放它，从而达到同步或互斥的目的。
slug: 【玩转RT-Thread】线程间同步 信号量
date: 2022-07-17 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---

## 一、概述：

> 多个执行单元(线程、中断)同时执行临界区，操作临界资源，会导致竟态产生，为了解决这种竟态问题，RT-Thread OS提供了如下几种同步互斥机制:

**信号量**（semaphore）、**互斥量**（mutex）、和**事件集**（event）

## 二、信号量

#### 1、简述

信号量是一种轻型的用于解决线程间同步问题的内核对象，线程可以获取或释放它，从而达到`同步`或`互斥`的目的。

信号量工作示意图如下图所示，每个信号量对象都有一个信号量值和一个线程等待队列，信号量的值对应了信号量对象的实例数目、资源数目，假如信号量值为 5，则表示共有 5 个信号量实例（资源）可以被使用，当`信号量实例数目为零时`，再申请该信号量的线程就会被`挂起`在该信号量的等待队列上，等待可用的信号量实例（资源）。

![](https://img-blog.csdnimg.cn/55d38f32e6e84b038d35a27bea1b9074.png)


#### 2、信号量结构体

```c
struct rt_semaphore
{
    struct rt_ipc_object parent;                        /**< 继承自ipc_object类 */

    rt_uint16_t          value;                         /**< value of semaphore. */
    rt_uint16_t          reserved;                      /**< reserved field 预留*/
};
```

当线程对资源进行获取时，value值进行减一操作；直到该信号量被释放，value进行加一操作。

#### 3、信号量使用及管理

> 对一个信号量的操作包含:`创建/初始化信号量、获取信号量、释放信号量、删除/脱离信号量`。

![](https://img-blog.csdnimg.cn/d8bde585ecaa4669817d29f0ffdc34f5.png)


1）动态创建信号量

> 当调用这个函数时，系统将先从对象管理器中分配一个 semaphore 对象，并初始化这个对象，然后初始化父类 IPC 对象以及与 semaphore 相关的部分。在创建信号量指定的参数中，信号量标志参数决定了当信号量不可用时，多个线程等待的排队方式。

> 当选择 `RT_IPC_FLAG_FIFO（先进先出）`方式时，那么等待线程队列将按照先进先出的方式排队，先进入的线程将先获得等待的信号量；
> 当选择 `RT_IPC_FLAG_PRIO（优先级等待）`方式时，等待线程队列将按照优先级进行排队，优先级高的等待线程将先获得等待的信号量。

`函数声明`：

`rt_sem_t rt_sem_create(const char *name, rt_uint32_t value, rt_uint8_t flag);`

`参数介绍`：

![](https://img-blog.csdnimg.cn/f829de152967470393d8b7605e4c2b12.png)


`注意：`

`(1)此处的*name定义最多只能显示八个字符`

`(2)查看rt_sem_create()函数返回值是-->typedef struct rt_semaphore *rt_sem_t;,也就是一个重命名的结构体rt_semaphore `

```c
// flag值	如下

#define RT_IPC_FLAG_FIFO                0x00            /**< FIFOed IPC. @ref IPC.按照先进先出的方式获取信号量资源 */
#define RT_IPC_FLAG_PRIO                0x01            /**< PRIOed IPC. @ref IPC.按线程优先级获取信号量资源 */
```

2）动态创建的信号量删除

> 系统不再使用信号量时，可通过删除信号量以释放系统资源，适用于动态创建的信号量。

> 调用这个函数时，系统将删除这个信号量。`如果删除该信号量时，有线程正在等待该信号量，那么删除操作会先唤醒等待在该信号量上的线程（等待线程的返回值是 - RT_ERROR），然后再释放信号量的内存资源。`

`函数声明`
`rt_err_t rt_sem_delete(rt_sem_t sem);`

`实例`

```c
#include <rtthread.h>
#include <rtdevice.h>
#include <board.h>
#include <rtdbg.h>

rt_sem_t sem1;


int main(void)
{
    sem1 = rt_sem_create("sem_1",1,RT_IPC_FLAG_FIFO);
    if(sem1 == RT_NULL)
    {
        LOG_E("rt_sem_create is failure...\n");
    }

    LOG_D("rt_sem_create is success...\n");
    return 0;
}
```



3）静态创建信号量

`描述`

> 对于静态信号量对象，它的内存空间在编译时期就被编译器分配出来，放在读写数据段或未初始化数据段上，此时使用信号量就不再需要使用 rt_sem_create 接口来创建它，而只需在使用前对它进行初始化即可。

`函数声明`

```c
rt_err_t rt_sem_init(rt_sem_t    sem,
                     const char *name,
                     rt_uint32_t value,
                     rt_uint8_t  flag);
```

`参数描述`

![](https://img-blog.csdnimg.cn/cbe2c704ffc849cf8859a6c46237681a.png)


4）脱离信号量

`描述`

> 脱离信号量就是让信号量对象从内核对象管理器中脱离，适用于`静态初始化的信号量`。

`函数声明`

```c
rt_err_t rt_sem_detach(rt_sem_t sem);
```

5）获取信号量

`描述`

>线程通过获取信号量来获得信号量资源实例，当信号量值大于零时，线程将获得信号量，并且相应的信号量值会减 1。
>
>如果信号量的值等于零，那么说明当前信号量资源实例不可用，申请该信号量的线程将根据 time 参数的情况选择`直接返回、或挂起等待一段时间、或永久等待`，直到其他线程或中断释放该信号量。
>
>如果在参数 time 指定的时间内依然得不到信号量，线程将`超时返回`，返回值是` - RT_ETIMEOUT`。

`函数声明`

```c
rt_err_t rt_sem_take (rt_sem_t sem, rt_int32_t time);
```

`参数描述`

```c
// time参数

#define RT_WAITING_FOREVER              -1              /**< Block forever until get resource. */
#define RT_WAITING_NO                   0               /**< Non-block. */
```

```c
// 扩展：

rt_err_t rt_sem_trytake(rt_sem_t sem); // 无等待获取信号量

// 这个函数与 rt_sem_take(sem, RT_WAITING_NO) 的作用相同，即当线程申请的信号量资源实例不可用的时候，它不会等待在该信号量上，而是直接返回 - RT_ETIMEOUT。
```



6）信号量释放

`函数声明`

```c
rt_err_t rt_sem_release(rt_sem_t sem);
```

`描述`

>例如当信号量的值等于零时，并且有线程等待这个信号量时，释放信号量将唤醒等待在该信号量线程队列中的第一个线程，由它获取信号量；否则将把信号量的值加 1。

#### 4、信号量实例演示
>这里可以看到创建了两个线程，而且线程的优先级都是符合我们定义的20，但是查看线程状态可以发现，线程1和线程2都是阻塞态。这是因为我们在线程的入口函数中使用了mdelay延时函数，执行这个函数，线程会短暂地进入阻塞态
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/632591398002468c9a69cf3e4ac8cfe4.png)

>由于我们在线程2的入口函数中执行了信号量获取函数，但是我们在初始化信号量2的时候设定的初值是0，所以此时线程2由于未获取到信号量而陷入阻塞态

![在这里插入图片描述](https://img-blog.csdnimg.cn/4293c61a10d14a128312fd50d7d2c9c6.png)

>查看信号量设定的标志位是`RT_IPC_FLAG_FIFO`，是按照先进先出的方式进行信号量的获取的，所以在函数的执行顺序中可以发现都是按照线程1->线程2->线程1->线程2...的顺序执行的，这样就实现了线程的并发互斥运行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1000730483a345728f1faa2ba68bde2d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/31c3f295be2e41f9ab8d674ade353007.png)
>最后附上测试源代码

```c
#include <rtthread.h>
#include <rtdevice.h>
#include <board.h>
#include <rtdbg.h>

rt_sem_t sem1;
struct rt_semaphore sem2;

rt_thread_t th1,th2;

int flags = 0;

void th1_entry(void *parameter)
{
    while(1)
    {
        rt_thread_mdelay(8000);
        rt_sem_take(sem1, RT_WAITING_FOREVER);// 获取信号量
        flags++;
        if(flags == 100)
            flags = 0;
        rt_kprintf("th1_entry [%d]\n",flags);
        rt_sem_release(&sem2);// 对获取的信号量进行释放
    }
}

void th2_entry(void *parameter)
{
    while(1)
    {
        rt_sem_take(&sem2, RT_WAITING_FOREVER);// 获取信号量
        if(flags > 0)
            flags--;
        rt_kprintf("th2_entry [%d]\n",flags);
        rt_sem_release(sem1);// 对获取的信号量进行释放
        rt_thread_mdelay(1000);
    }
}

int main(void)
{
    int ret = 0;
    sem1 = rt_sem_create("sem_1",1,RT_IPC_FLAG_FIFO);
    if(sem1 == RT_NULL)
    {
        LOG_E("sem1 rt_sem_create is failure...\n");
    }

    LOG_D("sem1 rt_sem_create is success...\n");

    ret = rt_sem_init(&sem2, "sem2", 0, RT_IPC_FLAG_FIFO);
    if(ret < 0)
    {
        LOG_E("sem2 rt_sem_create is failure...\n");
        return ret;
    }

    LOG_D("sem2 rt_sem_init successed...\n");

    th1 = rt_thread_create("th1", th1_entry, NULL, 512, 20, 5);
    if(th1 == RT_NULL)
    {
        LOG_E("th1 rt_thread_create failed...\n");
        return -ENOMEM;
    }

    LOG_D("th1 rt_thread_create successed...\n");

    th2 = rt_thread_create("th2", th2_entry, NULL, 512, 20, 5);
    if(th2 == RT_NULL)
    {
        LOG_E("th2 rt_thread_create failed...\n");
        return -ENOMEM;
    }

    LOG_D("th2 rt_thread_create successed...\n");

    rt_thread_startup(th1);
    rt_thread_startup(th2);
    return 0;
}
```

