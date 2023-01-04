---
title: CPK-RA6M4智慧门禁系统教学
description: 本视频秉持着学习开放的思想，在RT-Thread夏令营经历的这几周时间，自己也是结合所学知识开发出一款智慧门禁系统，而为了大家更好的学习交流，本次将以视频录制加源码开源的方式供大家学习参考。
slug: 【玩转RT-Thread】CPK-RA6M4智慧门禁系统教学
date: 2022-07-24 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---

![image-20220804171524901](https://img-blog.csdnimg.cn/img_convert/b98bb037592975e68632b30a8e0845f6.png)

## 1、项目介绍

本次项目主控为CPK-RA6M4开发板，是瑞萨RA6高性能系列的一款基于Arm架构的开发板，而RA产品家族也是提供了一套成熟的工具生态链来帮助开发者更好的进行产品的研发。本次我们使用瑞萨FSP（灵活配置软件包）结合RT-Thread Studio工具进行项目的研发。

下面来说说本次项目的功能：主要就是通过四大模块结合RT-Thread内核机制，开发出一款具有人员签到打卡、温湿度读取，OLED显示以及云端数据上报这四大功能。

## 2、前期准备

开发工具：

* RT-Thread Studio

![在这里插入图片描述](https://img-blog.csdnimg.cn/a95d6ef5c3224144b554bcb416691bcf.png)



RT-Thread Studio是一套一站式的 RT-Thread 开发工具，通过简单易用的图形化配置系统以及丰富的软件包和组件资源，让物联网开发变得简单和高效。

RT-Thread Studio 主要包括工程创建和管理，代码编辑，SDK管理，RT-Thread配置，构建配置，调试配置，程序下载和调试等功能，结合图形化配置系统以及软件包和组件资源，减少重复工作，提高开发效率。

下载链接：[RT-Thread Studio 下载](https://www.rt-thread.org/page/download.html#studio)



* 瑞萨FSP（灵活配置软件包）

![Flexible Software Package (FSP)](https://img-blog.csdnimg.cn/img_convert/fbdf24dd4b66b39b2f108f558ecf4617.png)

瑞萨电子灵活配置软件包 (FSP) 是一款增强型软件包，旨在为使用瑞萨电子 RA 系列 ARM 微控制器的嵌入式系统设计提供简单易用且可扩展的高质量软件。

下载链接：[瑞萨FSP v3.5.0](https://github.com/renesas/fsp/releases/tag/v3.5.0)



模块:

* AHT10
* ESP8266
* RC522及读卡标签
* ssd1306 OLED显示屏

## 3、模块介绍及使用

#### 3.1 AHT10

###### 3.1.1底层I2C通信协议简介

I2C（Inter Integrated Circuit）总线是 PHILIPS 公司开发的一种半双工、双向二线制同步串行总线。I2C 总线传输数据时只需两根信号线，一根是双向数据线 SDA（serial data），另一根是双向时钟线 SCL（serial clock）。SPI 总线有两根线分别用于主从设备之间接收数据和发送数据，而 I2C 总线只使用一根线进行数据收发。

而I2C通信的读写数据是通过等待从机的应答信号（ACK）。

也就是说，当配置方向为“写数据”时，主机每发送完一个字节数据，都要等待从机的应答信号，而当数据传输结束时，主机向从机发送一个停止传输信号，表示不再传输数据；当配置方向为“读数据”时，从机每发送完一个数据，都需要等待主机的应答信号，当主机希望停止接收数据时，会向从机发送一个非应答信号（NACK），从机就不再向主机继续发送数据。

这里需要注意的是，I2C通讯常用的是复合格式，该传输过程中有两次起始信号。在第一次传输中，主机通过slave_address找到从设备后会发送一段数据（通常表示从设备内部的寄存器或存储器系统）；而在第二次的传输中，对该地址的内容进行读写，也就是说，第一次通讯时告诉从机读写地址，第二次通讯才是读写的实际内容。

当 SCL 线是高电平时， SDA 线从高电平向低电平切换，这时候代表通讯的起始；当SCL 是高电平时， SDA线由低电平向高电平切换，这代表通讯的结束。

简单来说，就是I2C 使用 SDA 信号线来传输数据，使用 SCL 信号线进行数据同步。

###### 3.1.2 sensor框架的使用

在RT-Thread中，我们需要了解sensor设备的作用，是为上层提供统一的操作接口，提高上层代码的可重用性。

掌握sensor框架的使用，需要了解一下API的调用：

|          **函数**           |                  **描述**                  |
| :-------------------------: | :----------------------------------------: |
|      rt_device_find()       | 根据传感器设备设备名称查找设备获取设备句柄 |
|      rt_device_open()       |               打开传感器设备               |
|      rt_device_read()       |                  读取数据                  |
|     rt_device_control()     |               控制传感器设备               |
| rt_device_set_rx_indicate() |              设置接收回调函数              |
|      rt_device_close()      |               关闭传感器设备               |



###### 3.1.3 AHT10对接到sensor框架

首先先来介绍下接线：

| 引脚功能 | 引脚接线 |
| :------: | :------: |
|   SCL    |   P512   |
|   SDA    |   P511   |
|   VCC    |   3.3V   |
|   GND    |   GND    |

然后我们打开settings，在硬件部分使能I2C1（芯片设备驱动->Enable I2C BUS->使能I2C1），同时可以检查下组件部分I2C设备驱动程序是否使能

![image-20220805110607476](https://img-blog.csdnimg.cn/img_convert/f301db2fd8e1988f2ca4e311c5d973cf.png)

然后使用下面的程序完成模块初始化工作

```c
#include "sensor_asair_aht10.h"
#define AHT10_I2C_BUS  "i2c1"

/* 模块初始化工作 */
static int rt_hw_aht10_port(void)
{
    struct rt_sensor_config cfg;
    cfg.intf.dev_name  = AHT10_I2C_BUS;
    cfg.intf.user_data = (void *)AHT10_I2C_ADDR;
    rt_hw_aht10_init("aht10", &cfg);
    return RT_EOK;
}
INIT_ENV_EXPORT(rt_hw_aht10_port);
```

AHT10温湿度数据读取

```c
    // AHT10设备读取数值
    float humidity, temperature;

    aht10_device_t dev;
    rt_hw_aht10_port();
    dev = aht10_init(AHT10_I2C_BUS);
    if (dev == RT_NULL)
    {
        rt_kprintf(" The sensor initializes failure");
    }
    else
    {
        rt_kprintf(" The sensor initializes ok!\n");
    }
    
    /* read humidity 采集湿度 */
    humidity = aht10_read_humidity(dev);
    /* read temperature 采集温度 */
    temperature = aht10_read_temperature(dev);
```

#### 3.2 ESP8266

###### 3.2.1 底层uart简介

UART（Universal Asynchronous Receiver/Transmitter）通用异步收发传输器，UART 作为异步串口通信协议的一种，工作原理是将传输数据的每个字符一位接一位地传输。是在应用程序开发过程中使用频率最高的数据总线。

UART作为异步串行通信协议的一种，工作原理是将传输数据的每个二进制位一位接一位地传输。在UART通信协议中信号线上的状态为高电平时代表‘1’，信号线上的状态为低电平时代表‘0’。比如使用UART通信协议进行一个字节数据的传输时就是在信号线上产生八个高低电平的组合。

- 串行通信是指利用一条传输线将数据一位位地顺序传送，也可以用两个信号线组成全双工通信。特点是通信线路简单，利用简单的线缆就可实现通信，降低成本，适用于远距离通信，但传输速度慢的应用场合。
- 异步通信以一个字符为传输单位，通信中两个字符间的时间间隔多少是不固定的，然而在同一个字符中的两个相邻位间的时间间隔是固定的。也就是说两个uart设备之间通信的时候不需要时钟线，但是需要在两个uart设备上指定相同的传输速率，以及空闲位、起始位、校验位、结束位，也就是遵循相同的协议。
- 数据传送速率用波特率来表示，即每秒钟传送的二进制位数。例如数据传送速率为120字符/秒，而每一个字符为10位（1个起始位，7个数据位，1个校验位，1个结束位），则其传送的波特率为10×120＝1200字符/秒＝1200波特。

![查看源图像](https://img-blog.csdnimg.cn/img_convert/5fdffba894c94d48a14c4f64a9992b93.jpeg)

空闲位：UART协议规定，当总线处于空闲状态时信号线的状态为‘1’即高电平，表示当前线路上没有数据传输。

起始位：每开始一次通信时发送方先发出一个逻辑”0”的信号（低电平），表示传输字符的开始。因为总线空闲时为高电平所以开始一次通信时先发送一个明显区别于空闲状态的信号即低电平。

数据位：起始位之后就是我们所要传输的数据，数据位可以是5、6、7、8，9位等，构成一个字符（一般都是8位）。如ASCII码（7位，剩下的1位二进制为0），扩展BCD码（8位）。`先发送最低位，最后发送最高位`，使用低电平表示‘0’高电平表示‘1’完成数据位的传输。

###### 3.2.2 MQTT通讯协议介绍

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于发布/订阅（publish/subscribe）模式的"轻量级"通讯协议，该协议构建于TCP/IP协议上，由IBM在1999年发布。

其优点就是利用极少的代码和有限的带框，为物联网设备远程通讯提供消息传输服务， 相比于HTTP协议在互联网上的客户端请求，服务端应答模式，MQTT的发布订阅模式在物联网设备上更适用。

实现MQTT协议需要客户端和服务器端通讯完成，在通讯过程中，MQTT协议中有三种身份：发布者（Publish）、代理（Broker）（服务器）、订阅者（Subscribe）。其中，消息的发布者和订阅者都是客户端，消息代理是服务器，消息发布者可以同时是订阅者。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a665ee8ebb4d427bbfa4c8a7ec013913.png)


###### 3.2.3 AT组件

AT 命令集是一种应用于 AT 服务器（AT Server）与 AT 客户端（AT Client）间的设备连接与数据通信的方式。 其基本结构如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/1eb3c2cd78584efa85656a2cdd8daa8f.png)


由上图可知，AT的使用需要AT Client和AT Server这两部分共同完成，AT Client通过AT命令向Server发送请求，等待Server的响应，并对响应的数据或主动发送给Client的数据（URC数据）进行解析处理，并获取相关信息。

###### 3.2.4 MQTT协议及AT组件在RT-Thread中的使用

**RT-Thread Settings设置**



添加AT Device及OneNET软件包

AT Device配置：

![image-20220805122341394](https://img-blog.csdnimg.cn/img_convert/bf1c743b728c70c2764a3db1d368039c.png)

OneNET配置:

首先我们需要前往ONENET官网进行产品创建及设备绑定，没有onenet账号的可以去注册一个。

![image-20220805122741348](https://img-blog.csdnimg.cn/img_convert/b40025d68d9bd0f32ff6efa3b74ff12d.png)
![image-20220805122836636](https://img-blog.csdnimg.cn/img_convert/c7c17c8d743dca043a098be627baf93c.png)

然后将创建的信息填写到settings中

![image-20220805123248475](https://img-blog.csdnimg.cn/img_convert/9588bef3c3bf99d27bdc3cbd56394ece.png)

在组件中使能AT命令

![image-20220805123407687](https://img-blog.csdnimg.cn/img_convert/ad6b80e0d34df83a213dcdbf85d4416b.png)


接线示意：

| 引脚功能 | 引脚接线 |
| :------: | :------: |
|    TX    |   P100   |
|    RX    |   P101   |
|   VCC    |    5V    |
|   GND    |   GND    |

**FSP配置**

由于RT-Thread提供了有限的驱动配置，所以需要我们使用瑞萨FSP进行相关的配置

首先点击`RA Smart Configurator`,记住这里使用的FSP版本为`v3.5.0`

![image-20220805152204348](https://img-blog.csdnimg.cn/img_convert/7710c9a62ad3a4486362f9d4bf92869f.png)



![image-20220805153017364](https://img-blog.csdnimg.cn/img_convert/2cb89487678dcf16bf3c784851a42091.png)

![image-20220805153317144](https://img-blog.csdnimg.cn/img_convert/bf0866f3d50eae91f43aee142ef06966.png)

![image-20220805153416015](https://img-blog.csdnimg.cn/img_convert/58fb0f18f54f0eaf49598c6c20f67b62.png)

完成上述操作后保存并编译，注意这里由于RT-Thread版本问题，可能出现`#include <dfs_posix.h>`未参与编译以及还有其他一些问题，可以参考这一issue[[CPK-RA6M4] onenet上云报错<RT-Thread 的版本为 4.1.0 及以上>](https://github.com/RT-Thread/rt-thread/issues/6188)

现在可以下载到开发板了，由于我们使用的AT例程中是默认初始化运行，所以在上电后就会自动连接WIFI了。

然后就是数据上云，代码如下：

```c
            if (onenet_mqtt_upload_digit("temperature",temperature) < 0)
            {
                LOG_E("upload has an error, stop uploading");
            }
            else
            {
                rt_kprintf("humidity : %d.%d\n", (int)temperature, (int)(temperature * 10) % 10);
            }

            rt_thread_delay(5000);


            if (onenet_mqtt_upload_digit("humidity",humidity ) < 0)
            {
                LOG_E("upload has an error, stop uploading");
            }
            else
            {
                rt_kprintf("humidity : %d.%d\n", (int)humidity, (int)(humidity * 10) % 10);
            }
```

这里我们创建了两个数据流，分别是温度以及湿度。在AHT10读取温湿度之后，就可以进行数据的上报了，然后可以在onenet官网不断看到数据的上报了。

![image-20220805145942277](https://img-blog.csdnimg.cn/img_convert/096cd5faec78062ac227adcc12f08f05.png)

![image-20220805145958887](https://img-blog.csdnimg.cn/img_convert/67bd30c613f9a819cc2b7abddb58a5bc.png)



#### 3.3 RC522

###### 3.3.1 底层SPI协议简介

SPI（Serial Peripheral Interface，串行外设接口）是一种高速、全双工、同步通信总线，常用于短距离通讯，主要应用于 EEPROM、FLASH、实时时钟、AD 转换器、还有数字信号处理器和数字信号解码器之间。SPI 一般使用 4 根线通信，如下图所示：

![SPI 主设备和从设备的连接方式](https://img-blog.csdnimg.cn/img_convert/9d4277d0aba7be84c59b68efb9402995.png)

- MOSI –主机输出 / 从机输入数据线（SPI Bus Master Output/Slave Input）。
- MISO –主机输入 / 从机输出数据线（SPI Bus Master Input/Slave Output)。
- SCLK –串行时钟线（Serial Clock），主设备输出时钟信号至从设备。
- CS –从设备选择线 (Chip select)。也叫 SS、CSB、CSN、EN 等，主设备输出片选信号至从设备。



整体的传输大概可以分为以下几个过程：

（1）主机先将`NSS`信号拉低，这样保证开始接收数据；

（2）当**接收端**检测到时钟的边沿信号时，它将立即读取**数据线**上的信号，这样就得到了一位数据（1`bit`;由于时钟是随数据一起发送的，因此指定**数据的传输速度并不重要**，尽管设备将具有可以运行的最高速度。

（3）**主机**发送到**从机**时：主机产生相应的时钟信号，然后数据**一位一位**地将从`MOSI`信号线上进行发送到从机；

（4）**主机**接收**从机**数据：如果从机需要将数据发送回主机，则主机将继续生成预定数量的时钟信号，并且从机会将数据通过`MISO`信号线发送；

![查看源图像](https://img-blog.csdnimg.cn/img_convert/3c32f25bed1d6e44c840608e4e352fc4.png)

###### 3.3.2 RC522读卡机制说明

首先来看下RC522与M1卡的通讯流程：

**寻卡->防止卡片冲撞->选卡->休眠->发送0x40（7bit）->发送0x43->发送0xa0等4字节->发送0x00等18字节**

![image-20220805154320392](https://img-blog.csdnimg.cn/img_convert/4c59b975aa9fdccbb19b38b059b87e1a.png)

* 复位应答（Request）：M1卡的通信协议和通信波特率是定义好的，当有卡片进入读卡器的工作范围时，读卡器要以特定的协议与卡片通信，从而确定卡片的卡型。

* 防冲突机制（Anticollision Loop）：当有多张卡片进入读写器操作范围时，会从中选择一张卡片进行操作，并返回选中卡片的序列号。

* 选择卡片（Select Tag）：选择被选中的卡的序列号，并同时返回卡的容量代码。

* 三次相互确认（3 Pass Authentication）：选定要处理的卡片后，读写器就要确定访问的扇区号，并且对扇区密码进行密码校验。在三次互相认证后就可以通过加密流进行通信。每次在选择扇区的时候都要进行扇区的密码校验。

* 对数据块的操作：
  读（Read）:读一个块的数据；
  写（Write）：在一个块中写数据；
  加（Increment）：对数据块中的数值进行加值；
  减（Decrement）：对数据块中的数值进行减值；
  传输（Transfer）：将数据寄存器中的内容写入数据块中；
  中止（Halt）：暂停卡片的工作；

###### 3.3.3 RC522在RT-Thread的使用

首先打开settings，添加RC522软件包，并在硬件部分使能SPI1

![image-20220805155052783](https://img-blog.csdnimg.cn/img_convert/ef7c08d20f957c70b67fe63943fec730.png)

打开瑞萨FSP，添加一个名为r_spi的新stack，并进行如下配置：

![image-20220805155448374](https://img-blog.csdnimg.cn/img_convert/263c237d561d232466249aacb0dd42a4.png)

引脚接线：

| 引脚功能 | 引脚接线 |
| :------: | :------: |
|   MOSI   |   P411   |
|   MISO   |   P410   |
|   SCL    |   P412   |
|   SDA    |   P311   |
|   RST    |   P312   |
|   VCC    |   3.3V   |
|   GND    |   GND    |
|   IRQ    |   悬空   |

代码部分参考RC522sample

![image-20220805155715593](https://img-blog.csdnimg.cn/img_convert/474416ef76e9523302b9cdd544bbf03b.png)

SPI初始化配置：

```c
#include "mfrc522.h"

static struct rt_spi_device mfrc522_spi_dev;
struct rt_hw_spi_cs
{
    rt_uint32_t pin;
};
static struct rt_hw_spi_cs spi_cs;

static int rt_hw_spi_rc522_init()
{
    rt_err_t res = RT_EOK;

    // Attach Device
    spi_cs.pin = MFRC522_SS_PIN;
    rt_pin_mode(spi_cs.pin, PIN_MODE_OUTPUT);
    res = rt_spi_bus_attach_device(&mfrc522_spi_dev, MFRC522_SPI_DEVICE_NAME, MFRC522_SPI_BUS_NAME, (void*)&spi_cs);
    if (res != RT_EOK)
    {
        rt_kprintf("[RC522] Failed to attach device %s\n", MFRC522_SPI_DEVICE_NAME);
        return res;
    }

    // Set device SPI Mode
    struct rt_spi_configuration cfg = {0};
    cfg.data_width = 8;
    cfg.mode = RT_SPI_MASTER | RT_SPI_MODE_0 | RT_SPI_MSB | RT_SPI_NO_CS;
    cfg.max_hz = MFRC522_SPICLOCK;

    rt_spi_configure(&mfrc522_spi_dev, &cfg);

    return RT_EOK;
}
/* 导出到自动初始化 */
INIT_COMPONENT_EXPORT(rt_hw_spi_rc522_init);
```

另外需要在完成一下配置，`双击打开mfrc522.h，修改MFRC522_SS_PIN为0x3b，MFRC522_RST_PIN为0x3c,分别对应SDA和RST引脚`

![image-20220805155922942](https://img-blog.csdnimg.cn/img_convert/5045a3933b508b5267b5fb9eeec6064b.png)

打开mfrc522.c，修改配置`MFRC522_SS_PIN`及`MFRC522_RST_PIN`

![image-20220805161330936](https://img-blog.csdnimg.cn/img_convert/c61dba211ccb8ece988de9e703dc15cb.png)

打开rtconfig.h，找到以下两个引脚的定义，修改成如下：

注意：**`一旦在RT-Thread settings中做了相关操作并保存设置后，在rtconfig.h中的配置都会以settings中的配置为准而被全部刷新，所以需要保留一个备份，下次保存设置的时候记得重新修改配置`**

```
#define MFRC522_SS_PIN 0x3b
#define MFRC522_RST_PIN 0x3c
```

至此，RC522的相关配置结束

#### 3.4 SSD1306

###### 3.4.1 底层I2C通信协议

（这里参考AHT10关于I2C通信协议的介绍，此处不再赘述）

###### 3.4.2 SSD1306在RT-Thred的使用

接线示意：

| 引脚功能 | 引脚接线 |
| :------: | :------: |
|   SCL    |   P400   |
|   SDA    |   P401   |
|   VCC    |   3.3V   |
|   GND    |   GND    |

RT-Thread Settings配置：

添加ssd1306软件包，然后跳转到配置界面修改i2c address为0x3c，bus name为i2c0

![image-20220805162320450](https://img-blog.csdnimg.cn/img_convert/c6b56810a69b472ea12eaa4e8d60008a.png)

打开rtconfig.h，添加i2c代码，`注意之前在rtconfig.h中进行的配置已经被刷新，需要重新添加配置代码`：

```c
#define BSP_USING_I2C
#define BSP_USING_I2C0
#define BSP_I2C0_SCL_PIN 0x400
#define BSP_I2C0_SDA_PIN 0x401
```

打开drv_soft_i2c.c文件，添加代码：

```c
#ifdef BSP_USING_I2C0
#define I2C0_BUS_CONFIG                                  \
    {                                                    \
        .scl = BSP_I2C0_SCL_PIN,                         \
        .sda = BSP_I2C0_SDA_PIN,                         \
        .bus_name = "i2c0",                              \
    }
#endif
```

打开瑞萨FSP，新建一个r_iic_master的new stack，完成以下配置：

![image-20220805162941069](https://img-blog.csdnimg.cn/img_convert/04a3d96e15bbde106c73e389eca7fe10.png)

生成配置之后添加用户代码：

```c
#include "ssd1306.h"

void oled_init()
{
    ssd1306_Init();

    ssd1306_Fill(Black);
    ssd1306_SetCursor(10, 25);
    ssd1306_WriteString("Hello RT-Thread!", Font_7x10, White);
    ssd1306_UpdateScreen();
}
INIT_APP_EXPORT(oled_init);
```

实时时钟显示代码：

```c
            ssd1306_Fill(White);
            ssd1306_SetCursor(0, 5);
            ssd1306_WriteString("Now Time", Font_16x26, Black);
            ssd1306_SetCursor(40, 40);
            ssd1306_WriteString(mstr, Font_11x18, Black);
            ssd1306_SetCursor(50, 40);
            ssd1306_WriteString(":", Font_11x18, Black);
            ssd1306_SetCursor(60, 40);
            ssd1306_WriteString(hstr, Font_11x18, Black);
            ssd1306_UpdateScreen();
```

温湿度数据显示代码：

```c
            ssd1306_Fill(White);
            ssd1306_SetCursor(4, 2);
            ssd1306_WriteString("Humi_Temp_Detection!", Font_7x10, Black);
            ssd1306_UpdateScreen();

            rt_thread_mdelay(1000);
            char buff[64];

            snprintf(buff, sizeof(buff), "Temperature: %d.%d\n", (int)temperature, (int)(temperature * 10) % 10);
            ssd1306_SetCursor(15, 30);
            ssd1306_WriteString(buff, Font_6x8, Black);
            ssd1306_UpdateScreen();
            rt_kprintf("Temperature_OLED : %d.%d\n", (int)temperature, (int)(temperature * 10) % 10);

            snprintf(buff, sizeof(buff), "Humidity:%d.%d\n", (int)humidity, (int)(humidity * 10) % 10);
            ssd1306_SetCursor(25, 47);
            ssd1306_WriteString(buff, Font_6x8, Black);
            ssd1306_UpdateScreen();
            rt_kprintf("Humidity_OLED : %d.%d\n", (int)humidity, (int)(humidity * 10) % 10);
```

## 4、整体代码框架

#### 4.1 多线程任务分配

本次细分作品功能，共分为四大模块：分别是AHT10温湿度读取、onenet上云、oled显示、rc522读卡。

所以共创建四个线程：

（1）RC522_thread：用于RC522读卡

（2）aht10_read_thread：用于aht10读取温湿度数值

（3）onenet_aht10_thread：云端数据上报

（4）oled_thread：OLED显示

#### 4.2 线程间交互

本次在IPC方面的使用很不成熟，只是在每个线程的入口函数中进行互斥量的保护，并没有将RT-Thread内核机制灵活运用到代码中，是我此次学习的最大不足，其实也做过一些例如邮箱机制的使用，但是由于数据显示异常而没有进行下去，在工程源码的ITNG_Project2中包含了这种机制的使用，也就是说提供了两套方案，但是确实个人效率太低，第二种方案被搁置。

#### 4.3 代码整合

![image-20220802210559475](https://img-blog.csdnimg.cn/img_convert/8aed1d64ef9fd8e219744b794b8dd048.png)

在本次的程序设计中，我使用了一个while循环结合switch选择语句来保证整体代码的运行，在线程的入口程序使用互斥量来完成资源的保护，但是RT-Thread多线程机制的使用也是仍显不足。

都说程序设计也是艺术设计，要学会使用代码抽象人类社会的运行机制，程序设计方面，我设计的不合理，导致整个项目如同流水线般运行，亮点不大，值得反思。

![image-20220805171602257](https://img-blog.csdnimg.cn/img_convert/36369721fa7745022aab2ad2492dc64f.png)

## 5、踩坑指南

其实大部分踩坑说明在上面的教学指南中一般都有说明，这里简单说些：

（1）注意瑞萨FSP目前在RT-Thread中的支持包版本为`v3.5.0`

（2）由于瑞萨有自己完整的生态开发工具，所以RT-Thread与瑞萨合作时对于底层驱动的定义只有部分，还有一些需要在FSP中进行配置并生成配置。同时在HAL库中也需要添加相应的驱动代码，同时记得需要在settings中将相应的外设支持打开。

（3）对于每次的settings设置，其实都会生成相关的宏和定义在rtconfig.h文件中，所以每次更行settings时都会将用户在rtconfig.h中添加的代码删除，这时候需要重新添加，否则会生成一些宏未定义的错误。

