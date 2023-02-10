---
title: I2C(内核学习)
description: I2C（Inter Integrated Circuit）总线是 PHILIPS 公司开发的一种半双工、双向二线制同步串行总线。I2C 总线传输数据时只需两根信号线，一根是双向数据线 SDA（serial data），另一根是双向时钟线 SCL（serial clock）。SPI 总线有两根线分别用于主从设备之间接收数据和发送数据，而 I2C 总线只使用一根线进行数据收发。
slug: 【玩转RT-Thread】I2C(内核学习)
date: 2022-07-15 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---

## 一、i2c协议
由飞利浦公司开发，支持设备间的短距离通信。i2c通信需要的引脚少，硬件实现简单、可扩展性强，被广泛应用在系统内多个集成电路（IC）间的通信。

## 二、i2c物理层
* i2c通信总线可连接多个i2c通信设备，支持多个通信主机和多个通信从机。i2c通信只需要两条双向总线——SDA（串行数据线）和SCL（串行时钟线）。
`SDA`：用于传输数据
`SCL`：用于同步数据收发
* 每个连接到总线的设备都有一个独立地址，共7bit，主机正是利用该地址对设备进行访问
* i2c支持多主控，任何时间点都只能有一个主控。
![在这里插入图片描述](https://img-blog.csdnimg.cn/01cc1805f0db4842a836a7dae9b11978.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)

* i2c器件的SDA引脚和SCL引脚是开漏电路[（参照资料）](https://blog.csdn.net/ngulb/article/details/81174233?ops_request_misc=&request_id=&biz_id=102&utm_term=%E5%BC%80%E6%BC%8F%E7%94%B5%E8%B7%AF%E4%BB%80%E4%B9%88%E6%84%8F%E6%80%9D&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-81174233.142^v7^pc_search_result_control_group,157^v4^control&spm=1018.2226.3001.4187)形式，因此，SDA和SCL总线都需要连接上拉电阻[（参照资料）](https://blog.csdn.net/fymx203/article/details/89426403?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164973690016782092947037%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164973690016782092947037&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-89426403.142^v7^pc_search_result_control_group,157^v4^control&utm_term=%E4%B8%8A%E6%8B%89%E7%94%B5%E9%98%BB&spm=1018.2226.3001.4187)，当总线空闲时，两条总线均为高电平。
* 各器件的SDA和SCL信号线在总线上都是`线与`关系。（即连接到总线上的任意器件输出低电平都会将总线信号拉低）
## 三、i2c协议层
协议层定义了i2c的通信协议。一个完整的i2c数据传输包含开始信号，器件地址，读写控制，器件内访问地址，有效数据，应答信号和结束信号。
#### 1.i2c总线的位传输
数据传输：当SCL位高电平时，SDA必须保持稳定，SDA上传1位数据。
数据改变：当SCL为低电平时，SDA才可以改变电平
`i2c位传输时序图`
![在这里插入图片描述](https://img-blog.csdnimg.cn/3bcc9522f82841b5a9808703e4c29fa9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
#### 2.i2c总线的开始和结束信号
`开始信号`：SCL 为高电平时，主机将SDA 拉低，表示数据传输即将开始。
`结束信号`：在SDA 为低电平时，主机将SCL 拉高并保持高电平，然后在将SDA 拉高，表示传输结束。
#### 3.i2c应答信号
* 在`主机`发送完每一个字节数据后，释放SDA（保持高电平），被寻址的接收器在成功接收到每一个字节后，必须产生一个应答`ACK`（从机将SDA拉低，使它在这个时钟脉冲的高电平期间保持稳定的低电平）
* 当`从机`接收不到数据或通信故障时，`从机`必须使SDA保持高电平，`主机`产生一个结束信号终止传输或者产生新的传输。
#### 4.i2c总线的仲裁机制
* SDA的仲裁也是建立在总线具有`线与`逻辑功能的原理上的。
* 节点在发送1位数据后，比较总线上所呈现的数据与自己发送的是否一致。是，继续发送；否则，退出竞争。
* SDA的仲裁可以保证i2c总线系统在多个主节点上同时企图控制总线时通信正常进行而且数据不丢失（总线系统通过仲裁只允许一个主节点可以继续占据总线）
* 当SCL为高电平时，仲裁在SDA上发生。在其他主机发送低电平时，发送高电平的主机将会断开它的数据传输级，因为总线上的电平是`线与`连接。
## 四、访问i2c总线设备
一般情况下MCU 的I2C 器件都是作为主机和从机通讯，在RT-Thread 中将I2C 主机虚拟为I2C 总线设备，I2C 从机通过I2C 设备接口和I2C 总线通讯，相关接口如下所示：
|函数|描述|
|-|-|
|rt_device_find()|根据I2C 总线设备名称查找设备获取设备[句柄](https://blog.csdn.net/wyx0224/article/details/83385168?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164976053816780265492902%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164976053816780265492902&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-83385168.142^v7^pc_search_result_control_group,157^v4^control&utm_term=%E5%8F%A5%E6%9F%84&spm=1018.2226.3001.4187)|
|rt_i2c_transfer() |传输数据|
## 五、查找i2c总线设备
在使用I2C 总线设备前需要根据I2C 总线设备名称获取设备句柄，进而才可以操作I2C 总线设备，查找设备函数如下所示，

	rt_device_t rt_device_find(const char* name);

|参数|描述|
|-|-|
|name|i2c总线设备名称|
|<strong>返回|--|
|设备句柄|查找到对应设备将返回相应的设备句柄|
|RT-NULL|没有找到相应的设备对象|

一般情况下，注册到系统的I2C 设备名称为i2c0 ，i2c1 等，使用示例如下所示：
```
#define AHT10_I2C_BUS_NAME "i2c1" /* 传感器连接的I2C总线设备名称*/
struct rt_i2c_bus_device *i2c_bus; /* I2C总线设备句柄*/
/* 查找I2C总线设备， 获取I2C总线设备句柄*/
i2c_bus = (struct rt_i2c_bus_device *)rt_device_find(name);
```
## 六、数据传输
获取到I2C 总线设备句柄就可以使用rt_i2c_transfer() 进行数据传输。函数原型如下所示：
```
rt_size_t rt_i2c_transfer(struct rt_i2c_bus_device *bus,
										struct rt_i2c_msg msgs[],
										rt_uint32_t num);
```
|参数|描述|
|-|-|
|bus|i2c总线设备句柄|
|msgs[]|待传输的消息数组指针|
|num|消息数组的元素个数|
|<strong>返回|-|
|-|-|
|消息数组的元素个数|成功|
|错误码|失败|

* 和SPI 总线的自定义传输接口一样，I2C 总线的自定义传输接口传输的数据也是以一个消息为单位。
* 参数msgs[] 指向待传输的消息数组，用户可以自定义每条消息的内容，实现I2C 总线所支持的2 种不同的数据传输模式。如果主设备需要发送重复开始条件，则需要发送2 个消息。
`!!! note “注意事项” 此函数会调用rt_mutex_take(), 不能在中断服务程序里面调用，会导致assertion报错。`
>I2C 消息数据结构原型如下：
```
struct rt_i2c_msg
{
rt_uint16_t addr; /* 从机地址*/
rt_uint16_t flags; /* 读、写标志等*/
rt_uint16_t len; /* 读写数据字节数*/
rt_uint8_t *buf; /* 读写数据缓冲区指针　*/
}
```
* 从机地址addr：支持7 位和10 位二进制地址，需查看不同设备的数据手册。
* 标志flags 可取值为以下宏定义，根据需要可以与其他宏使用位运算“|” 组合起来使用。
`!!! note “注意事项” RT-Thread I2C 设备接口使用的从机地址均不包含读写位，读写位控制需修改标志flags。`
```
#define RT_I2C_WR 0x0000 /* 写标志*/
#define RT_I2C_RD (1u << 0) /* 读标志*/
#define RT_I2C_ADDR_10BIT (1u << 2) /* 10 位地址模式*/
#define RT_I2C_NO_START (1u << 4) /* 无开始条件*/
#define RT_I2C_IGNORE_NACK (1u << 5) /* 忽视NACK */
#define RT_I2C_NO_READ_ACK (1u << 6) /* 读的时候不发送ACK */
```
> 使用示例如下所示：
```
#define AHT10_I2C_BUS_NAME "i2c1" /* 传感器连接的I2C总线设备名称*/
#define AHT10_ADDR 0x38 /* 从机地址*/

struct rt_i2c_bus_device *i2c_bus; /* I2C总线设备句柄*/

/* 查找I2C总线设备， 获取I2C总线设备句柄*/
i2c_bus = (struct rt_i2c_bus_device *)rt_device_find(name);

/* 读传感器寄存器数据*/
static rt_err_t read_regs(struct rt_i2c_bus_device *bus, rt_uint8_t len, rt_uint8_t
*buf)
{
	struct rt_i2c_msg msgs;
	msgs.addr = AHT10_ADDR; /* 从机地址*/
	msgs.flags = RT_I2C_RD; /* 读标志*/
	msgs.buf = buf; /* 读写数据缓冲区指针　*/
	msgs.len = len; /* 读写数据字节数*/
	/* 调用I2C设备接口传输数据*/
	if (rt_i2c_transfer(bus, &msgs, 1) == 1)
	{
		return RT_EOK;
	}
	else
	{
		return -RT_ERROR;
	}
}
```
## 七、I2C 总线设备使用示例
I2C 设备的具体使用方式可以参考如下示例代码，示例代码的主要步骤如下：
1. 首先根据I2C 设备名称查找I2C 名称，获取设备句柄，然后初始化aht10 传感器。
2. 控制传感器的2 的函数为写传感器寄存器write_reg() 和读传感器寄存器read_regs()
这两个函数分别调用了rt_i2c_transfer() 传输数据。读取温湿度信息的函数read_temp_humi() 则是调用这两个函数完成功能。
```
/*
* 程序清单： 这是一个I2C 设备使用例程
* 例程导出了i2c_aht10_sample 命令到控制终端
* 命令调用格式： i2c_aht10_sample i2c1
* 命令解释： 命令第二个参数是要使用的I2C总线设备名称， 为空则使用默认的I2C总线设备
* 程序功能： 通过I2C 设备读取温湿度传感器aht10 的温湿度数据并打印
*/
#include <rtthread.h>
#include <rtdevice.h>

#define AHT10_I2C_BUS_NAME "i2c1" /* 传感器连接的I2C总线设备名称*/
#define AHT10_ADDR 0x38 /* 从机地址*/
#define AHT10_CALIBRATION_CMD 0xE1 /* 校准命令*/
#define AHT10_NORMAL_CMD 0xA8 /* 一般命令*/
#define AHT10_GET_DATA 0xAC /* 获取数据命令*/

static struct rt_i2c_bus_device *i2c_bus = RT_NULL; /* I2C总线设备句柄*/
static rt_bool_t initialized = RT_FALSE; /* 传感器初始化状态*/

/* 写传感器寄存器*/
static rt_err_t write_reg(struct rt_i2c_bus_device *bus, rt_uint8_t reg, rt_uint8_t*data)
{
	rt_uint8_t buf[3];
	struct rt_i2c_msg msgs;
	buf[0] = reg; //cmd
	buf[1] = data[0];
	buf[2] = data[1];
	msgs.addr = AHT10_ADDR;
	msgs.flags = RT_I2C_WR;
	msgs.buf = buf;
	msgs.len = 3;
	
	/* 调用I2C设备接口传输数据*/
	if (rt_i2c_transfer(bus, &msgs, 1) == 1)
	{
		return RT_EOK;
	}
	else
	{
		return -RT_ERROR;
	}
}

/* 读传感器寄存器数据*/
static rt_err_t read_regs(struct rt_i2c_bus_device *bus, rt_uint8_t len, rt_uint8_t*buf)
{
	struct rt_i2c_msg msgs;
	msgs.addr = AHT10_ADDR;
	msgs.flags = RT_I2C_RD;
	msgs.buf = buf;
	msgs.len = len;

	/* 调用I2C设备接口传输数据*/
	if (rt_i2c_transfer(bus, &msgs, 1) == 1)
	{
		return RT_EOK;
	}
	else
	{
		return -RT_ERROR;
	}
}

static void read_temp_humi(float *cur_temp, float *cur_humi)
{
	rt_uint8_t temp[6];
	write_reg(i2c_bus, AHT10_GET_DATA, 0); /* 发送命令*/
	rt_thread_mdelay(400);
	read_regs(i2c_bus, 6, temp); /* 获取传感器数据*/
	/* 湿度数据转换*/
	*cur_humi = (temp[1] << 12 | temp[2] << 4 | (temp[3] & 0xf0) >> 4) * 100.0 / (1<< 20);
	/* 温度数据转换*/
	*cur_temp = ((temp[3] & 0xf) << 16 | temp[4] << 8 | temp[5]) * 200.0 / (1 << 20)- 50;
}

static void aht10_init(const char *name)
{
	rt_uint8_t temp[2] = {0, 0};
	/* 查找I2C总线设备， 获取I2C总线设备句柄*/
	i2c_bus = (struct rt_i2c_bus_device *)rt_device_find(name);
	if (i2c_bus == RT_NULL)
	{
		rt_kprintf("can't find %s device!\n", name);
	}
	else
	{
		write_reg(i2c_bus, AHT10_NORMAL_CMD, temp);
		rt_thread_mdelay(400);
		temp[0] = 0x08;
		temp[1] = 0x00;
		write_reg(i2c_bus, AHT10_CALIBRATION_CMD, temp);
		rt_thread_mdelay(400);
		initialized = RT_TRUE;
	}
}

static void i2c_aht10_sample(int argc, char *argv[])
{
	float humidity, temperature;
	char name[RT_NAME_MAX];
	humidity = 0.0;
	temperature = 0.0;
	if (argc == 2)
	{
		rt_strncpy(name, argv[1], RT_NAME_MAX);
	}
	else
	{
		rt_strncpy(name, AHT10_I2C_BUS_NAME, RT_NAME_MAX);
	}
	if (!initialized)
	{
		/* 传感器初始化*/
		aht10_init(name);
	}
	if (initialized)
	{
		/* 读取温湿度数据*/
		read_temp_humi(&temperature, &humidity);
		rt_kprintf("read aht10 sensor humidity : %d.%d %%\n", (int)humidity, (int)
		(humidity * 10) % 10);
		if( temperature >= 0 )
		{
			rt_kprintf("read aht10 sensor temperature: %d.%d°C\n", (int)temperature,
			(int)(temperature * 10) % 10);
		}
		else
		{
			rt_kprintf("read aht10 sensor temperature: %d.%d°C\n", (int)temperature,
			(int)(-temperature * 10) % 10);
		}
	}
	else
	{
		rt_kprintf("initialize sensor failed!\n");
	}
}
/* 导出到msh 命令列表中*/
MSH_CMD_EXPORT(i2c_aht10_sample, i2c aht10 sample);
```

学习资料参考：[《嵌入式系统设计》](https://item.jd.com/10022312146340.html)、[RT-Thread](https://club.rt-thread.org/index.html)

