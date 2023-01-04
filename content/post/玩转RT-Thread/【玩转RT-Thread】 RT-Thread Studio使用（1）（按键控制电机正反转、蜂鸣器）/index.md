---
title: RT-Thread Studio使用（1）（按键控制电机正反转、蜂鸣器）
description: RT-Thread，全称是 Real Time-Thread，顾名思义，它是一个嵌入式实时多线程操作系统，基本属性之一是支持多任务，允许多个任务同时运行并不意味着处理器在同一时刻真地执行了多个任务。
slug: 玩转RT-Thread
date: 2022-07-15 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---



## 一、初识RT-Thread

`做世界级的 OS，让万物互联，信息畅通无阻。`
`成为未来 AIoT 领域最为主流的操作系统平台。`
#### 1.简介
> RT-Thread 是一个集`实时操作系统（RTOS）内核、中间件组件和开发者社区于一体`的技术平台，由`熊谱翔先生`带领并集合开源社区力量开发而成，RT-Thread 也是一个`组件完整丰富、高度可伸缩、简易开发、超低功耗、高安全性`的`物联网操作系统`。

#### 2.前景
> RT-Thread 具备一个 IoT OS 平台所需的所有关键组件，例如GUI、网络协议栈、安全传输、低功耗组件等等。经过11年的累积发展，RT-Thread 已经拥有一个`国内最大的嵌入式开源社区`，同时被广泛应用于能源、车载、医疗、消费电子等多个行业，累积装机量超过 14亿 台，成为国人`自主开发`、国内最成熟稳定和装机量最大的`开源 RTOS`。
#### 3.软件生态
> RT-Thread 拥有`良好的软件生态`，支持市面上所有主流的编译工具如 GCC、Keil、IAR 等，工具链完善、友好，支持各类标准接口，如 POSIX、CMSIS、C++应用环境、Javascript 执行环境等，方便开发者移植各类应用程序。商用支持所有主流MCU架构，如 ARM Cortex-M/R/A, MIPS, X86, Xtensa, C-Sky, RISC-V，几乎支持市场上所有主流的 MCU 和 Wi-Fi 芯片。
## 二、实验准备
* 编程工具：`RT-Thread studio`
* 开发板：`潘多拉STM32L475`

---
## 三、实验需求
* 1.使用按键控制蜂鸣器和电机，当按下KEY0 后电机左转，当按下KEY1 后电机
右转，当按下KEY2 后电机停止，当按住WK_UP 时蜂鸣器鸣叫，松开WK_UP 后蜂鸣器关闭。
* 2.其中KEY0 KEY1 KEY2 三个按键会触发中断，通过pin 设备的中断回调函数控制电机，WK_UP 按键通过轮询的方式控制蜂鸣器鸣叫。

## 四、操作流程
#### 1.新建RT-Thread工程
![在这里插入图片描述](https://img-blog.csdnimg.cn/85370c1057554323ba75dd83c3d1844f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
#### 2.RT-Thread Studio界面介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/b24064da660f40b5b00e9e0f03d4f1ff.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
#### 3.代码编写
![在这里插入图片描述](https://img-blog.csdnimg.cn/c556436b0d44443686dafa3a0f389bd5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 4.烧录
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5ea1524e61e4b92af667e17decd12bb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)


#### 5.串口监视
![在这里插入图片描述](https://img-blog.csdnimg.cn/eae3d5a76ae14aa0a7e6a2d00145024d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 五、代码演示
`1.头文件`

	#include <rtthread.h>
	#include <rtdevice.h>
	#include <board.h>
`2.宏定义`
```
//按键初始化
#define PIN_KEY0 GET_PIN(D, 10) // PD10: KEY0 --> KEY
#define PIN_KEY1 GET_PIN(D, 9) // PD9: KEY1 --> KEY
#define PIN_KEY2 GET_PIN(D, 8) // PD8: KEY2 --> KEY
#define PIN_WK_UP GET_PIN(C,13)//PC13：WK_UP

//电机初始化
#define PIN_MOTOR_A GET_PIN(A,1)//PA1：MOTOR_A
#define PIN_MOTOR_B GET_PIN(A,0)//PA0：MOTOR_B

//蜂鸣器初始化
#define PIN_BEEP GET_PIN(B,2)//PB2：BEEP

enum
{
    MOTOR_STOP,
    MOTOR_LEFT,
    MOTOR_RIGHT
};
```
`3.void motor_ctrl(rt_uint8_t turn) //电机控制函数`
```
void motor_ctrl(rt_uint8_t turn)
{
    if (turn == MOTOR_STOP)
    {
        rt_pin_write(PIN_MOTOR_A, PIN_LOW);
        rt_pin_write(PIN_MOTOR_B, PIN_LOW);
    }
    else if (turn == MOTOR_LEFT)
    {
        rt_pin_write(PIN_MOTOR_A, PIN_LOW);
        rt_pin_write(PIN_MOTOR_B, PIN_HIGH);
    }
    else if (turn == MOTOR_RIGHT)
    {
        rt_pin_write(PIN_MOTOR_A, PIN_HIGH);
        rt_pin_write(PIN_MOTOR_B, PIN_LOW);
    }
    else
    {
        rt_kprintf("err parameter ! Please enter 0-2.");
    }
}
```
`4.void beep_ctrl(rt_uint8_t on) //蜂鸣器控制函数`
```
void beep_ctrl(rt_uint8_t on)
{
    if (on)
    {
        rt_pin_write(PIN_BEEP, PIN_HIGH);
    }
    else
    {
        rt_pin_write(PIN_BEEP, PIN_LOW);
    }
}
```
`5.void irq_callback(void *args) // 中断回调函数`
```
void irq_callback(void *args)
{
    rt_uint32_t sign = (rt_uint32_t)args;
    switch (sign)
    {
        case PIN_KEY0:
        motor_ctrl(MOTOR_LEFT);
        rt_kprintf("KEY0 interrupt. motor turn left.");
        break;
        case PIN_KEY1:
        motor_ctrl(MOTOR_RIGHT);
        rt_kprintf("KEY1 interrupt. motor turn right.");
        break;
        case PIN_KEY2:
        motor_ctrl(MOTOR_STOP);
        rt_kprintf("KEY2 interrupt. motor stop.");
        break;
        default:
        rt_kprintf("error sign= %d !", sign);
        break;
    }
}
```
`5.主函数`
```
int main(void)
{
    unsigned int count = 1;

    /* 设置按键引脚为输入模式*/
    rt_pin_mode(PIN_KEY0, PIN_MODE_INPUT_PULLUP);
    rt_pin_mode(PIN_KEY1, PIN_MODE_INPUT_PULLUP);
    rt_pin_mode(PIN_KEY2, PIN_MODE_INPUT_PULLUP);
    rt_pin_mode(PIN_WK_UP, PIN_MODE_INPUT_PULLDOWN);

    /* 设置电机控制引脚为输入模式*/
    rt_pin_mode(PIN_MOTOR_A, PIN_MODE_OUTPUT);
    rt_pin_mode(PIN_MOTOR_B, PIN_MODE_OUTPUT);

    /* 设置蜂鸣器引脚为输出模式*/
    rt_pin_mode(PIN_BEEP, PIN_MODE_OUTPUT);

    /* 设置按键中断模式与中断回调函数*/
    rt_pin_attach_irq(PIN_KEY0, PIN_IRQ_MODE_FALLING , irq_callback , (void *)PIN_KEY0
    );
    rt_pin_attach_irq(PIN_KEY1, PIN_IRQ_MODE_FALLING , irq_callback , (void *)PIN_KEY1
    );
    rt_pin_attach_irq(PIN_KEY2, PIN_IRQ_MODE_FALLING , irq_callback , (void *)PIN_KEY2
    );

    /* 使能中断*/
    rt_pin_irq_enable(PIN_KEY0, PIN_IRQ_ENABLE);
    rt_pin_irq_enable(PIN_KEY1, PIN_IRQ_ENABLE);
    rt_pin_irq_enable(PIN_KEY2, PIN_IRQ_ENABLE);
    while (count > 0)
    {
        if (rt_pin_read(PIN_WK_UP) == PIN_HIGH)
        {
            rt_thread_mdelay(50);
            if (rt_pin_read(PIN_WK_UP) == PIN_HIGH)
            {
                rt_kprintf("WK_UP pressed. beep on.");
                beep_ctrl(1);
            }
        }
        else
        {
            beep_ctrl(0);
        }
        rt_thread_mdelay(10);
        count++;
    }

    return 0;
}
```


## 六、原理讲解
<font color=byellow>通过按键引脚、电机以及蜂鸣器的输入输出模式，并对按键设置中断编写中断回调函数，在使能中断后。
1.电机控制：当有外部事件触发引脚状态（按下按键）时，中断回调函数对特定的触发引脚进行判断，并执行相应的操作
2.蜂鸣器控制：在主函数中循环执行判断是否WK_UP按键是否按下，按下触发蜂鸣器响，松开停止发声。</font>

| 按键 | 功能 |
|--|--|
|KEY0|电机左转|
|KEY1|电机右转|
|KEY2|电机停止|
|WK_UP|蜂鸣器响|





