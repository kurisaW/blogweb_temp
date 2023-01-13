---
title: 多线程技术学习（基于Linux）
description: Linux多线程概念、inux线程实现、线程同步的方法、线程的互斥、互斥PK信号量、信号量操作、互斥操作、条件变量
slug: 【Linux系统开发】多线程技术学习
date: 2022-03-22 00:00:00+0000
image: cover.jpg
categories:
    - operating_system
tags:
    - operating system
---



## 1.Linux多线程概念

>**（1）线程：指运行中的程序的调度单位。**  

>**（2）多线程的优点：**
* 运行与一个线程中的多个线程，他们彼此之间使用**相同的地址空间**，**共享大部分数据**，启动一个线程所花费的空间远远小于启动一个进程所花费的空间，并且，线程见彼此切换所需要的时间也远远小于进程间切换所需要的时间。
* 进程间方便的通信机制。对不同的进程来说，它们有独立的数据空间，要进行数据的传递智能通过通信的方式
* 应用程序响应速度提高
* 使多CPU系统更加高效
* 改善程序结构  
> **（3）线程的生命周期**

 就绪->运行->阻塞->终止

---

## 2.linux线程实现

 >**（1）线程创建**
* 头文件包含
 #include <pthread.h>
* 定义函数：

		int pthread_create(pthread_t *restrict tidp,const pthread_attr_t *restrict attr, void *(*start_rtn)(void),void *restrict arg)
 * 函数说明：
 tidp：线程id
    attr：线程属性(通常为空)
    start_rtn：线程要执行的函数    
    arg：  start_rtn的参数  

 >**（2）线程退出**
* 头文件包含：
 #include <pthread.h>
* 定义函数：
void pthread_exit(void * rval_ptr)
* 功能：终止调用线程Rval_ptr:线程退出返回值的指针。

> **（3）线程等待**
* 头文件包含：
 #include <pthread.h>
* 定义函数：
		
	
		int pthread_join(pthread_t tid,void **rval_ptr)
* 功能：阻塞调用线程，直到指定的线程终止。
* 函数说明：
Tid :等待退出的线程id
Rval_ptr：线程退出的返回值的指针

> **（4）线程标识获取**
* 头文件包含：
 #include <pthread.h>
* 定义函数：
pthread_t pthread_self(void)
* 功能：获取调用线程的 thread identifier

> **（5）线程清除**
* 头文件包含：
 #include <pthread.h>
* 定义函数：   

		void pthread_cleanup_push(void (*rtn)(void *),void *arg)
* 功能：将清除函数压入清除栈
* 函数说明：
Rtn:清除函数
Arg:清除函数的参数

---

## 3.线程同步的方法

 进行多线程编程，因为无法知道哪个线程会在哪个时候对共享资源进行操作，因此让如何保护共享资源变得复杂，通过下面这些技术的使用，可以解决线程之间对资源的竞争：
>互斥量（互斥锁）Mutex
>信号灯（信号量）Semaphore
> 条件变量Conditions

---

## 4.线程的互斥

 线程在取出头节点前必须要等待互斥量，如果此时有其他线程已经获得该互斥量，那么该线程将会阻塞在这里。只有等到其他线程释放掉该互斥量后，该线程才有可能得到该互斥量。互斥量从本质上说就是一把锁, 提供对共享资源的保护访问。   

> **（1）创建**

在Linux中, 互斥量使用类型pthread_mutex_t表示。在使用前, 要对它进行初始化: 
* 对于静态分配的互斥量, 可以把它设置为默认属性的mutex对象PTHREAD_MUTEX_INITIALIZER
* 对于动态分配的互斥量, 在申请内存(malloc)之后, 通过pthread_mutex_init进行初始化, 并且在释放内存(free)前需要调用pthread_mutex_destroy。
> 函数使用：
 头文件：
 #include <pthread.h> 

	int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restric attr)
	int pthread_mutex_destroy(pthread_mutex_t *mutex)

> **（2）加锁**

对共享资源的访问, 要使用互斥量进行加锁, 如果互斥量已经上了锁, 调用线程会阻塞, 直到互斥量被解锁。
> 函数使用： 

    int pthread_mutex_lock(pthread_mutex_t *mutex)
    int pthread_mutex_trylock(pthread_mutex_t *mutex)

 返回值: 成功则返回0, 出错则返回错误编号. 
注意：trylock是非阻塞调用模式, 如果互斥量没被锁住, trylock函数将对互斥量加锁, 并获得对共享资源的访问权限; 如果互斥量被锁住了, trylock函数将不会阻塞等待而直接返回EBUSY, 表示共享资源处于忙状态。

> **（3）解锁**  

在操作完成后，必须给互斥量解锁，也就是前面所说的释放。这样其他等待该锁的线程才有机会获得该锁，否则其他线程将会永远阻塞。

    int pthread_mutex_unlock(pthread_mutex_t *mutex)

---
## 5.互斥PK信号量

>Mutex是一把钥匙，一个人拿了就可进入一个房间，出来的时候把钥匙交给队列的第一个。
>Semaphore是一件可以容纳N人的房间，如果人不满就可以进去，如果人满了，就要等待有人出来。对于N=1的情况，称为binary semaphore。
>Binary semaphore与Mutex的差异：
>
>1. mutex要由获得锁的线程来释放（谁获得，谁释放）。而semaphore可以由其它线程释放
>2. 初始状态可能不一样：mutex的初始值是1 ,而semaphore的初始值可能是0（或者为1）。  
---
## 6.信号量操作（代码演示）

```
#include<stdio.h>
#include<string.h>
#include<pthread.h>
#include<stdlib.h>
#include<semaphore.h>

//子线程处理

char buf[200];
sem_t sem;
int flag;

void *func(void *arg)
{
	sem_wait(&sem); // 接收信号量
	/*
	Sem_wait()递减(锁定)sem指向的信号量。如果信号量的值大于0，则继续递减，函数立即返回。
	如果信号量当前的值为0，那么调用就会阻塞，直到信号量可以递减(即信号量的值高于0)，或者信号处理程序中断调用。
	*/
	//while(strncmp(buf,"end",3) != 0)
	while(flag == 0)
	{
		printf("input %d char.\n",strlen(buf));
		memset(buf,0,sizeof(buf));
	}
	
	pthread_exit(NULL);
}

int main(void)
{
	int ret = -1;
	pthread_t th = -1;
	
	sem_init(&sem,0,0); // 在sem指向的地址处初始化未命名的信号量
	
	ret = pthread_create(&th,NULL,func,NULL); //pthread_create()函数在调用进程中启动一个新线程,创建成功返回0
	if(ret != 0)
	{
		printf("pthread_create error.\n");
		return -1;
	}
	
	printf("please input string,end with Enter.\n");
	while(scanf("%s",buf))
	{
		if(!strncmp(buf,"end",3))
		{
			printf("process end\n");
			flag = 1;
			sem_post(&sem); //增加（解锁）sem指向的信号量
			break;
		}
		
		printf("input %d char .\n",strlen(buf));
		memset(buf,0,sizeof(buf));
	}
	
	printf("wait reclaim child thread.\n");
	ret = pthread_join(th,NULL);
	if(ret != 0)
	{
		printf("pthread_join error.\n");
		exit(-1);
	}
	printf("reclaim child thread successfully.\n");
	
	return 0;
}
```
---
## 7.互斥操作（函数演示）

```
#include<stdio.h>
#include<string.h>
#include<pthread.h>
#include<stdlib.h>

//子线程处理

char buf[200];
pthread_mutex_t mutex;
int flag;

void *func(void *arg)
{
	sleep(1);
	while(flag == 0)
	{ 
		pthread_mutex_lock(&mutex);// 互斥加锁
		printf("input %d char.\n",strlen(buf));
		memset(buf,0,sizeof(buf));
		pthread_mutex_unlock(&mutex); // 解锁
	}
	
	pthread_exit(NULL);
}

int main(void)
{
	int ret = -1;
	pthread_t th = -1;
	
	pthread_mutex_init(&mutex,NULL);
	
	ret = pthread_create(&th,NULL,func,NULL); //pthread_create()函数在调用进程中启动一个新线程,创建成功返回0
	if(ret != 0)
	{
		printf("pthread_create error.\n");
		return -1;
	}
	
	printf("please input string,end with Enter.\n");
	while(1)
	{
		pthread_mutex_lock(&mutex);// 对互斥对象加锁锁定
		scanf("%s",buf);
		pthread_mutex_unlock(&mutex); // 输入后解锁
		if(!strncmp(buf,"end",3))
		{
			printf("process end\n");
			flag = 1;
			break;
		}
		
		printf("input %d char .\n",strlen(buf));
		memset(buf,0,sizeof(buf));
	}
	
	printf("wait reclaim child thread.\n");
	ret = pthread_join(th,NULL); //pthread_join()函数等待由thread指定的线程结束。如果该线程已经终止，则pthread_join()立即返回。
	if(ret != 0)
	{
		printf("pthread_join error.\n");
		exit(-1);
	}
	printf("reclaim child thread successfully.\n");
	pthread_mutex_destroy(&mutex);
	
	
	return 0;
}
```
---
## 8.条件变量（代码演示）
```
#include<stdio.h>
#include<string.h>
#include<pthread.h>
#include<stdlib.h>

//子线程处理

char buf[200];
pthread_mutex_t mutex;
pthread_cond_t cond;
int flag;

void *func(void *arg)
{
	while(flag == 0)
	{ 
		pthread_mutex_lock(&mutex);// 互斥加锁
		pthread_cond_wait(&cond,NULL);// 线程同步等待
		printf("input %d char.\n",strlen(buf));
		memset(buf,0,sizeof(buf));
		pthread_mutex_unlock(&mutex); // 解锁
	}
	
	pthread_exit(NULL);
}

int main(void)
{
	int ret = -1;
	pthread_t th = -1;
	
	pthread_mutex_init(&mutex,NULL);
	pthread_cond_init(&cond,NULL); //初始化条件变量
	
	ret = pthread_create(&th,NULL,func,NULL); //pthread_create()函数在调用进程中启动一个新线程,创建成功返回0
	if(ret != 0)
	{
		printf("pthread_create error.\n");
		return -1;
	}
	
	printf("please input string,end with Enter.\n");
	while(1)
	{
		
		scanf("%s",buf);
		pthread_cond_signal(&cond);// 发送信号
		if(!strncmp(buf,"end",3))
		{
			printf("process end\n");
			flag = 1;
			break;
		}
		
		printf("input %d char .\n",strlen(buf));
		memset(buf,0,sizeof(buf));
	}
	
	printf("wait reclaim child thread.\n");
	ret = pthread_join(th,NULL); //pthread_join()函数等待由thread指定的线程结束。如果该线程已经终止，则pthread_join()立即返回。
	if(ret != 0)
	{
		printf("pthread_join error.\n");
		exit(-1);
	}
	printf("reclaim child thread successfully.\n");
	pthread_mutex_destroy(&mutex);
	pthread_cond_destroy(&cond);// 条件变量销毁
	
	
	return 0;
}
```
