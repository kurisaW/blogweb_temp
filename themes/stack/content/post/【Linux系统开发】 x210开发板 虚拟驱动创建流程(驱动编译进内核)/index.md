---
title: x210开发板 虚拟驱动创建流程
description: x210编写驱动编译进内核
slug: 【Linux系统开发】 x210开发板 虚拟驱动创建流程(驱动编译进内核)
date: 2022-07-22 00:00:00+0000
image: cover.jpg
categories:
    - Linux
tags:
    - Linux
---

## 内核编译常用命令

安装模块
`lsmod module_test.ko`
创建设备文件
`mknod /dev/test c 250 0`
查看设备状态
`lsmod module_test.ko`
查看设备注册信息(分为字符设备和块设备)
`cat /proc/devices`



## 知识补充:

```c
#include<stdio.h>
int main(void)
{
	int i;
	static int j;
	printf(i);
	printf(j);
}
// 注意：这里如果没有指定i值，则打印出来的是随机值
// 如果定义一个静态变量而没有赋值，则打印默认为0

```

## 虚拟驱动创建流程

首先进入x210_bsp/kernel 

make menuconfig

![](https://img-blog.csdnimg.cn/783c09aa06cb4a83a4df08d76d31c447.png)


![](https://img-blog.csdnimg.cn/dc736c9d0d2b4ceaaaffcd6f90a1a16a.png)


![](https://img-blog.csdnimg.cn/1193fb6e02be4c52a1d5951b0236ff38.png)


make -j4 

 cp arch/arm/boot/zImage /tftpboot/ -f

重启开发板查看开发板设备

ls /sys/devices/platform/

![](https://img-blog.csdnimg.cn/f57e4a58658e4380967bbfa1c0813864.png)


![](https://img-blog.csdnimg.cn/2f3b1acd43e740e68774ecbe2824ea2f.png)


cd sys/class/leds

led_test_4编写完成后

编译不报错即可

cd /root/x210_bsp/kernel/drivers/leds/

cp /mnt/hgfs/Myshare/driver/led_test_4/leds-s5pv210.c ./



vi Makefile->

`obj-$(CONFIG_LEDS_S5PV210)              += leds-s5pv210.o`

![](https://img-blog.csdnimg.cn/b30ce84418064d24a4e01c96218834f2.png)




vi Kconfig更改依赖（添加以下文件）

`config LEDS_S5PV210
        tristate "LED Support for S5PV210"
        help
          This option enables support for on-chip LED drivers found on Marvell
          Semiconductor 88PM8606 PMIC.`



![](https://img-blog.csdnimg.cn/43715e94bbcb4e8ab850492c919afc33.png)






进入到x210_bsp/kernel

执行make menuconfig

可以发现生成了新的配置（Device Drivers-> LED_Support），使能这个

![](https://img-blog.csdnimg.cn/31b19678d8204209b05d62de54afd1d9.png)


执行make编译

` cp arch/arm/boot/zImage /tftpboot/ -f`



secureCRT：

cd sys/class/leds

进入LED1,执行

echo 1 > brightness   // 灯亮

echo 0 > brightness //灯灭

---
最后附上源代码：

`leds-s5pv210.c`
```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <asm/uaccess.h>
#include <mach/gpio-bank.h>
#include <mach/regs-gpio.h>
#include <linux/ioport.h>
#include <asm/io.h>
#include <linux/cdev.h>
#include <linux/device.h>
#include <mach/gpio.h>
#include <linux/leds.h>

#define GPIO_LED1	S5PV210_GPJ0(3)
#define GPIO_LED2	S5PV210_GPJ0(4)
#define GPIO_LED3	S5PV210_GPJ0(5)

#define X210_LED_OFF	1
#define X210_LED_ON		0

struct led_classdev mydev1;
struct led_classdev mydev2;
struct led_classdev mydev3;

void s5pv210_led1_set(struct led_classdev *led_cdev,enum led_brightness value)
{
	printk(KERN_INFO "s5pv210_led1_set\n");
	
	if(value == LED_OFF)
	{
		gpio_set_value(GPIO_LED1,X210_LED_OFF);
	}
	else
	{
		gpio_set_value(GPIO_LED1,X210_LED_ON);
	}
}
void s5pv210_led2_set(struct led_classdev *led_cdev,enum led_brightness value)
{
	printk(KERN_INFO "s5pv210_led2_set\n");
	
	if(value == LED_OFF)
	{
		gpio_set_value(GPIO_LED2,X210_LED_OFF);
	}
	else
	{
		gpio_set_value(GPIO_LED2,X210_LED_ON);
	}
}
void s5pv210_led3_set(struct led_classdev *led_cdev,enum led_brightness value)
{
	printk(KERN_INFO "s5pv210_led3_set\n");
	
	if(value == LED_OFF)
	{
		gpio_set_value(GPIO_LED3,X210_LED_OFF);
	}
	else
	{
		gpio_set_value(GPIO_LED3,X210_LED_ON);
	}
}



static  int __init s5pv210_led_init(void)
{
	int ret = -1;
	// 申请GPIO
	if(gpio_request(GPIO_LED1,"led1_gpj0.3"))
	{
		printk(KERN_ERR "gpio_request failed.\n");
	}
	else
	{
		gpio_direction_output(GPIO_LED1,1);
	}
	
	mydev1.name = "led1";
	mydev1.brightness = 0;
	mydev1.brightness_set = s5pv210_led1_set;
	
	
	ret = led_classdev_register(NULL,&mydev1);
	if(ret < 0)
	{
		printk(KERN_ERR "led_classdev_register failed.\n");
		return ret;
	}
	
	mydev2.name = "led2";
	mydev2.brightness = 0;
	mydev2.brightness_set = s5pv210_led2_set;
	
	
	ret = led_classdev_register(NULL,&mydev2);
	if(ret < 0)
	{
		printk(KERN_ERR "led_classdev_register failed.\n");
		return ret;
	}
	
	mydev3.name = "led3";
	mydev3.brightness = 0;
	mydev3.brightness_set = s5pv210_led3_set;
	
	
	ret = led_classdev_register(NULL,&mydev3);
	if(ret < 0)
	{
		printk(KERN_ERR "led_classdev_register failed.\n");
		return ret;
	}
	
	return 0;
}

static void __exit s5pv210_led_exit(void)
{
	led_classdev_unregister(&mydev1);
	led_classdev_unregister(&mydev2);
	led_classdev_unregister(&mydev3);
	
	gpio_free(GPIO_LED1);
	gpio_free(GPIO_LED2);
	gpio_free(GPIO_LED3);
}

module_init(s5pv210_led_init);
module_exit(s5pv210_led_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("WYQ");
MODULE_DESCRIPTION("module_test");
```


`Makefile`
```c
#KERN_VER = $(shell uname -r)
#KERN_DIR = /lib/modules/$(KERN_VER)/build

KERN_DIR = /root/x210_bsp/kernel

obj-m += leds-s5pv210.o

all:
	make -C $(KERN_DIR) M=`pwd` modules

.PHONY:clean
clean:
	make -C $(KERN_DIR) M=`pwd` modules clean

```
