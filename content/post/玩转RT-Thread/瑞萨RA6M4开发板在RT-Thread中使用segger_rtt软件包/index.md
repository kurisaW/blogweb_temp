---
title: 瑞萨RA6M4开发板在RT-Thread中使用segger_rtt软件包
description: 瑞萨RA6M4开发板使用示例<RT-Thread的版本为v4.1.0及以上>
slug: 玩转RT-Thread
date: 2022-08-22 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---



#### 一、创建工程，选择SEGGER_RTT软件包

![image-20221003133030692](https://img-blog.csdnimg.cn/img_convert/015bb29dd26648570d03e65cd419f972.png)

![image-20221003133219108](https://img-blog.csdnimg.cn/img_convert/fb34abd8ec95bf35f09fb3bcde1f5d1d.png)

#### 2、添加jlinkRtt初始化函数[ 路径：/rt-thread/src/kservice.c ]

在`rt_console_set_device`前调用`rt_hw_jlink_rtt_init`初始化函数

![image-20221003133721333](https://img-blog.csdnimg.cn/img_convert/492ff3b5ab1bf24e62a4380f3d47bf29.png)

#### 3、控制台对接上jlinkRtt

```
rtconfg.h

// 修改RT_CONSOLE_DEVICE_NAME为空
```

![image-20221003134935152](https://img-blog.csdnimg.cn/img_convert/a4101391c61fad4add9376b4ebcd71e9.png)

```c
shell.c [ 路径:D:\rt-thread\components\finsh\shell.c]

/* 1、首先添加以下头文件 */
#include "SEGGER_RTT.h"
#include "SEGGER_RTT_Conf.h"

/* 2、修改finsh_getchar */
int finsh_getchar(void)
{
#ifdef RT_USING_DEVICE
    char ch = 0;
#ifdef RT_USING_POSIX_STDIO
    if(read(STDIN_FILENO, &ch, 1) > 0)
    {
        return ch;
    }
    else
    {
        return -1; /* EOF */
    }
#else
    rt_device_t device;

    RT_ASSERT(shell != RT_NULL);

    device = shell->device;
    if (device == RT_NULL)
    {
        extern char rt_hw_console_getchar(void);
        return rt_hw_console_getchar();
    }

    while (rt_device_read(device, -1, &ch, 1) != 1)
    {
        rt_sem_take(&shell->rx_sem, RT_WAITING_FOREVER);
        if (shell->device != device)
        {
            device = shell->device;
            if (device == RT_NULL)
            {
                return -1;
            }
        }
    }

    return ch;
#endif /* RT_USING_POSIX_STDIO */
#else
    extern char rt_hw_console_getchar(void);
    return rt_hw_console_getchar();
#endif /* RT_USING_DEVICE */
}
```

```c
kservice.c [ 路径:\rt-thread\src\kservice.c ]
// 另外我们还需要完成对控制台字符读取的对接，修改rt_hw_console_output 

RT_WEAK void rt_hw_console_output(const char *str)
{
    /* empty console output */
    rt_size_t i = 0, size = 0;

    size = rt_strlen(str);
    for (i = 0; i < size; i++)
    {
        if (*(str + i) == '\n')
        {
           break;
        }
    }
    SEGGER_RTT_printf(0,"%s",str);
}
RTM_EXPORT(rt_hw_console_output);
```

#### 4、实验效果

首先确保已经下载好`J-Link RTT Viewer`，直接去[官网](https://www.segger.com/products/debug-probes/j-link/tools/rtt-viewer/)下载最新版本即可

然后编译和下载工程，注意下载方式为`J-Link`

双击打开rtthread.map[ 路径: /Debug/rtthread.map ]文件，查看`_SEGGER_RTT`变量地址(全局搜索即可，找到.bss._SEGGER_RTT)

![image-20221003140449806](https://img-blog.csdnimg.cn/img_convert/608cb3d791c683d7886a92eef5ae848f.png)

打开`J-Link RTT Viewer`

![image-20221003140736161](https://img-blog.csdnimg.cn/img_convert/5e3d4a57e4a6b55dd62f61b5d6577105.png)

此时就可以正常使用segger_rtt了

![image-20221003140911791](https://img-blog.csdnimg.cn/img_convert/d09d8a0f28e2c45542199eb982dfed6e.png)

