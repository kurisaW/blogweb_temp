---
title: ART-Pi 网络时钟
description: 自制网络时钟，实现云端天气及实时时间获取
slug: 玩转RT-Thread
date: 2022-07-22 00:00:00+0000
image: cover.jpg
categories:
    - RT-Thread
tags:
    - RT-Thread
---

## 《玩转RT-Thread》自制网络时钟

---
@[toc]
 ## 一、准备工作

* 开发平台：RT-Thread Studio

* 开发板：ART-PI
* 主控芯片：STM32H750
* 温湿度传感器：SHT30
* 显示模组：0.96’OLED（SSD1306）
* 串口调试助手：SecureCRT

注意：这里由于ART-PI开发板自带WiFi模组，可直接使能。如果使用其他开发板，可考虑使用ESP8266通信模块。

## 二、新建RT-Thread 项目

![在这里插入图片描述](https://img-blog.csdnimg.cn/dfeff108ee0241919514065992e79ef8.png)




![在这里插入图片描述](https://img-blog.csdnimg.cn/9f49c13343914adf8d92f12a1ebf832e.png)


## 三、获取温湿度数据

#### 1、双击打开左边导航栏的RT-Thread Setting

![在这里插入图片描述](https://img-blog.csdnimg.cn/096e053c2ec545a9950c86dcb1e12d9e.png)


#### 2、使能软件模拟i2c（单击点亮即可）

![](https://img-blog.csdnimg.cn/6bb6a362155641c0b0b6fa0953c60e45.png)


#### 3、配置i2c及相关引脚

`这里的i2c引脚配置依自己开发板而定，配置完成后CTRL+S保存配置`

![](https://img-blog.csdnimg.cn/ae8aaaa39cf04296809e01ccef73d980.png)


#### 4、添加SHT3X软件包

![](https://img-blog.csdnimg.cn/d480f380b622466c9c38ae5129550067.png)


`CTRL+S保存配置，点击编译并下载`

具体RT-Thread Studio的一般使用可参照[【玩转RT-Thread】 RT-Thread Studio使用（1）（按键控制电机正反转、蜂鸣器）](https://blog.csdn.net/qq_56914146/article/details/124079730?spm=1001.2014.3001.5502)

`此时打开串口工具，可以看到前面配置的i2c1和i2c3已经注册成功`

![](https://img-blog.csdnimg.cn/04803141590e474bbe767242a8258ed5.png)


此时在串口输入help，可以看出有一个sht3x配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb9d35af5d19463282a1bd8400e90364.png)


```
输入：
sht3x probe i2c3 pd
sht3x read（读取温湿度信息）
```

## 四、获取NTP时间

#### 1、使能选择WiFi框架

![](https://img-blog.csdnimg.cn/fac0022dff324acc9aa4a94e85407e69.png)


#### 2、使能AP6212库

![](https://img-blog.csdnimg.cn/97afeb11678140b2a5acbea44bc8937f.png)


#### 3、添加easyflash和netutils软件包

![在这里插入图片描述](https://img-blog.csdnimg.cn/08cda56d446541358d2b5038545ca284.png)


`鼠标右键netutils打开配置项`

![在这里插入图片描述](https://img-blog.csdnimg.cn/cd98d8f9bb0f4f15941c4617c32b7aa3.png)


`使能NTP (网络时间协议)客户端	`

![](https://img-blog.csdnimg.cn/7fe0c1a627d94d1dba87dbcf4918d127.png)


`使能软件模拟RTC`

![](https://img-blog.csdnimg.cn/f2ac26826c0a463bb1124804fcc7c563.png)


`CTRL+S保存配置`

`修改配置`

```cassandra
(1)打开电脑中项目所在的路径-workpace-项目名称-packages-EasyFlash-v4.1.0-port，将port目录下的ef_fal_port.c文件复制到workpace-项目名称-board-port中

(2)修改port中宏定义FAL_EF_PART_NAME 中的名字
#define FAL_EF_PART_NAME "easyflash" //修改后的宏定义
```

`此时再编译并下载到开发板中`

#### 4、连接WiFi

```c
wifi scan     //搜索wifi
wifi join [SSID] [PASSWORD]     //连接WiFi
    
SSID:WiFi名称
PASSWORD：WiFi密码
```

#### 5、设置开机自连接WiFi

`（1）在board/port 目录下创建wifi_config.c文件来实现wifi上电自动连接
代码如下:`

```c
/*
 * Copyright (c) 2006-2021, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2022-06-09     ASUS       the first version
 */
#include <rtthread.h>

#ifdef BSP_USING_WIFI

#include <wlan_mgnt.h>
#include <wlan_cfg.h>
#include <wlan_prot.h>

#include <easyflash.h>
#include <fal.h>

#include <stdio.h>
#include <stdlib.h>

#if (EF_SW_VERSION_NUM < 0x40000)

static char *str_base64_encode_len(const void *src, char *out, int input_length);
static int   str_base64_decode(const char *data, int input_length, char *decoded_data);

static const unsigned char base64_table[65] =
    "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

static const char base64_decode_table[256] =
{
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x3E, 0x00, 0x00, 0x00, 0x3F,
    0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x3A, 0x3B, 0x3C, 0x3D, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E,
    0x0F, 0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 0x18, 0x19, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x1A, 0x1B, 0x1C, 0x1D, 0x1E, 0x1F, 0x20, 0x21, 0x22, 0x23, 0x24, 0x25, 0x26, 0x27, 0x28,
    0x29, 0x2A, 0x2B, 0x2C, 0x2D, 0x2E, 0x2F, 0x30, 0x31, 0x32, 0x33, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
};

static char *str_base64_encode_len(const void *src, char *out, int len)
{
    unsigned char *pos;
    const unsigned char *end, *in;
    size_t olen;

    olen = len * 4 / 3 + 4; /* 3-byte blocks to 4-byte */
    olen += olen / 72; /* line feeds */
    olen++; /* nul termination */

    end = (const unsigned char *)src + len;
    in = (const unsigned char *)src;
    pos = (unsigned char *)out;
    while (end - in >= 3)
    {
        *pos++ = base64_table[in[0] >> 2];
        *pos++ = base64_table[((in[0] & 0x03) << 4) | (in[1] >> 4)];
        *pos++ = base64_table[((in[1] & 0x0f) << 2) | (in[2] >> 6)];
        *pos++ = base64_table[in[2] & 0x3f];
        in += 3;
    }

    if (end - in)
    {
        *pos++ = base64_table[in[0] >> 2];

        if (end - in == 1)
        {
            *pos++ = base64_table[(in[0] & 0x03) << 4];
            *pos++ = '=';
        }
        else
        {
            *pos++ = base64_table[((in[0] & 0x03) << 4) |
                                                  (in[1] >> 4)];
            *pos++ = base64_table[(in[1] & 0x0f) << 2];
        }
        *pos++ = '=';
    }

    *pos = '\0';
    return (char *)out;
}

/*
 * return: length, 0 is error.
 */
static int str_base64_decode(const char *data, int input_length, char *decoded_data)
{
    int out_len;
    int i, j;

    if (input_length % 4 != 0) return 0;

    out_len = input_length / 4 * 3;

    if (data[input_length - 1] == '=') out_len--;
    if (data[input_length - 2] == '=') out_len--;

    for (i = 0, j = 0; i < input_length;)
    {
        uint32_t sextet_a = data[i] == '=' ? 0 & i++ : base64_decode_table[data[i++]];
        uint32_t sextet_b = data[i] == '=' ? 0 & i++ : base64_decode_table[data[i++]];
        uint32_t sextet_c = data[i] == '=' ? 0 & i++ : base64_decode_table[data[i++]];
        uint32_t sextet_d = data[i] == '=' ? 0 & i++ : base64_decode_table[data[i++]];

        uint32_t triple = (sextet_a << 3 * 6)
                          + (sextet_b << 2 * 6)
                          + (sextet_c << 1 * 6)
                          + (sextet_d << 0 * 6);

        if (j < out_len) decoded_data[j++] = (triple >> 2 * 8) & 0xFF;
        if (j < out_len) decoded_data[j++] = (triple >> 1 * 8) & 0xFF;
        if (j < out_len) decoded_data[j++] = (triple >> 0 * 8) & 0xFF;
    }

    return out_len;
}

static int read_cfg(void *buff, int len)
{
    char *wlan_cfg_info = RT_NULL;

    wlan_cfg_info = ef_get_env("wlan_cfg_info");
    if (wlan_cfg_info != RT_NULL)
    {
        str_base64_decode(wlan_cfg_info, rt_strlen(wlan_cfg_info), buff);
        return len;
    }
    else
    {
        return 0;
    }
}

static int get_len(void)
{
    int len;
    char *wlan_cfg_len = RT_NULL;

    wlan_cfg_len = ef_get_env("wlan_cfg_len");
    if (wlan_cfg_len == RT_NULL)
    {
        len = 0;
    }
    else
    {
        len = atoi(wlan_cfg_len);
    }

    return len;
}

static int write_cfg(void *buff, int len)
{
    char wlan_cfg_len[12] = {0};
    char *base64_buf = RT_NULL;

    base64_buf = rt_malloc(len * 4 / 3 + 4); /* 3-byte blocks to 4-byte, and the end. */
    if (base64_buf == RT_NULL)
    {
        return -RT_ENOMEM;
    }
    rt_memset(base64_buf, 0, len);

    /* interger to string */
    sprintf(wlan_cfg_len, "%d", len);
    /* set and store the wlan config lengths to Env */
    ef_set_env("wlan_cfg_len", wlan_cfg_len);
    str_base64_encode_len(buff, base64_buf, len);
    /* set and store the wlan config information to Env */
    ef_set_env("wlan_cfg_info", base64_buf);
    ef_save_env();
    rt_free(base64_buf);

    return len;
}

#else

static int read_cfg(void *buff, int len)
{
    size_t saved_len;

    ef_get_env_blob("wlan_cfg_info", buff, len, &saved_len);
    if (saved_len == 0)
    {
        return 0;
    }

    return len;
}

static int get_len(void)
{
    int len;
    size_t saved_len;

    ef_get_env_blob("wlan_cfg_len", &len, sizeof(len), &saved_len);
    if (saved_len == 0)
    {
        return 0;
    }

    return len;
}

static int write_cfg(void *buff, int len)
{
    /* set and store the wlan config lengths to Env */
    ef_set_env_blob("wlan_cfg_len", &len, sizeof(len));

    /* set and store the wlan config information to Env */
    ef_set_env_blob("wlan_cfg_info", buff, len);

    return len;
}

#endif /* (EF_SW_VERSION_NUM < 0x40000) */

static const struct rt_wlan_cfg_ops ops =
{
    read_cfg,
    get_len,
    write_cfg
};

void wlan_autoconnect_init(void)
{
    fal_init();
    easyflash_init();

    rt_wlan_cfg_set_ops(&ops);
    rt_wlan_cfg_cache_refresh();
}

#endif
```

`（2）在main.c中添加自动连接函数`

```c
#include <rtthread.h>
#include <rtdevice.h>
#include "drv_common.h"

#define LED_PIN GET_PIN(I, 8)
extern void wlan_autoconnect_init(void);

int main(void)
{
    rt_uint32_t count = 1;

    rt_pin_mode(LED_PIN, PIN_MODE_OUTPUT);
    /* init Wi-Fi auto connect feature */
    wlan_autoconnect_init();
    /* enable auto reconnect on WLAN device */
    rt_wlan_config_autoreconnect(RT_TRUE);

    return RT_EOK;
}

#include "stm32h7xx.h"
static int vtor_config(void)
{
    /* Vector Table Relocation in Internal QSPI_FLASH */
    SCB->VTOR = QSPI_BASE;
    return 0;
}
INIT_BOARD_EXPORT(vtor_config);
```

`编译并下载，此时开发板就能够从flash中自动读取上次连接数据并自动连接WiFi了。`

## 五、OLED屏显示温湿度和实时时间信息

#### 1、添加u8g2软件包

![](https://img-blog.csdnimg.cn/9aa5ecc3b3484b14b1c57ef4c75aae73.png)


#### 2、编写oled_display显示线程

`（1）在application分组下创建一个用户文件oled_display.cpp文件，存放本项目中的OLED显示代码。`

`代码如下：`

```cpp
#include <rthw.h>
#include <rtthread.h>
#include <rtdevice.h>
#include <U8g2lib.h>
#include <stdio.h>
#include <board.h>	
#include "drv_common.h"

#include <drv_soft_i2c.h>

extern "C"
{
#include <sht3x.h>
}
extern "C"
{
sht3x_device_t sht3x_init(const char *i2c_bus_name, rt_uint8_t sht3x_addr);
rt_err_t sht3x_read_singleshot(sht3x_device_t dev);
}

#define OLED_I2C_PIN_SCL                    24 //
#define OLED_I2C_PIN_SDA                    25  //

static U8G2_SSD1306_128X64_NONAME_F_SW_I2C u8g2(U8G2_R0,\
                                         /* clock=*/ OLED_I2C_PIN_SCL,\
                                         /* data=*/ OLED_I2C_PIN_SDA,\
                                         /* reset=*/ U8X8_PIN_NONE);

#define SUN 0
#define SUN_CLOUD  1
#define CLOUD 2
#define RAIN 3
#define THUNDER 4

static void drawWeatherSymbol(u8g2_uint_t x, u8g2_uint_t y, uint8_t symbol)
{
  // fonts used:
  // u8g2_font_open_iconic_embedded_6x_t
  // u8g2_font_open_iconic_weather_6x_t
  // encoding values, see: https://github.com/olikraus/u8g2/wiki/fntgrpiconic

  switch(symbol)
  {
    case SUN:
      u8g2.setFont(u8g2_font_open_iconic_weather_6x_t);
      u8g2.drawGlyph(x, y, 69);
      break;
    case SUN_CLOUD:
      u8g2.setFont(u8g2_font_open_iconic_weather_6x_t);
      u8g2.drawGlyph(x, y, 65);
      break;
    case CLOUD:
      u8g2.setFont(u8g2_font_open_iconic_weather_6x_t);
      u8g2.drawGlyph(x, y, 64);
      break;
    case RAIN:
      u8g2.setFont(u8g2_font_open_iconic_weather_6x_t);
      u8g2.drawGlyph(x, y, 67);
      break;
    case THUNDER:
      u8g2.setFont(u8g2_font_open_iconic_embedded_6x_t);
      u8g2.drawGlyph(x, y, 67);
      break;
  }
}

static void drawWeather(uint8_t symbol, int degree)
{
  drawWeatherSymbol(0, 63, symbol);
  u8g2.setFont(u8g2_font_logisoso32_tf);
  u8g2.setCursor(55, 63);
  u8g2.print(degree);
  u8g2.print("C");
}
static void drawHumidity(uint8_t symbol, int humidity)
{
  drawWeatherSymbol(0, 63, symbol);
  u8g2.setFont(u8g2_font_logisoso32_tf);
  u8g2.setCursor(55, 63);
  u8g2.print(humidity);
  u8g2.print("%");
}

#ifdef __cplusplus
extern "C"
{
#endif

void oled_display()
{
    u8g2.begin();
    u8g2.clearBuffer();

    u8g2.setFont(u8g2_font_logisoso32_tf);
    u8g2.setCursor(48+3, 42);
    u8g2.print("Hi~");     // requires enableUTF8Print()

    u8g2.setFont(u8g2_font_6x13_tr);            // choose a suitable font
    u8g2.drawStr(30, 60, "By Mculover666");   // write something to the internal memory
    u8g2.sendBuffer();

    sht3x_device_t  sht3x_device;
    sht3x_device = sht3x_init("i2c3", 0x44);

    rt_thread_mdelay(2000);

    int status = 0;
    char mstr[3];
    char hstr[3];
    time_t now;
    struct tm *p;
    int min = 0, hour = 0;
    int temperature = 0,humidity = 0;

    while(1)
    {
        switch(status)
        {
            case 0:
                now = time(RT_NULL);
                p=gmtime((const time_t*) &now);
                hour = p->tm_hour;
                min = p->tm_min;
                sprintf(mstr, "%02d", min);
                sprintf(hstr, "%02d", hour);


                u8g2.firstPage();
                do {
                     u8g2.setFont(u8g2_font_logisoso42_tn);
                     u8g2.drawStr(0,63,hstr);
                     u8g2.drawStr(50,63,":");
                     u8g2.drawStr(67,63,mstr);
                   } while ( u8g2.nextPage() );


                rt_thread_mdelay(5000);
                status = 1;
                break;
           case 1:
               if(RT_EOK == sht3x_read_singleshot(sht3x_device))
               {
                   temperature = (int)sht3x_device->temperature;
               }
               else
               {
                   temperature = 0;
               }
               u8g2.clearBuffer();
               drawWeather(SUN, temperature);
               u8g2.sendBuffer();
               rt_thread_mdelay(5000);
               status = 2;
               break;
           case 2:
               if(RT_EOK == sht3x_read_singleshot(sht3x_device))
              {
                   humidity = (int)sht3x_device->humidity;
              }
              else
              {
                  humidity = 0;
              }
              u8g2.clearBuffer();
              drawHumidity(RAIN, humidity);
              u8g2.sendBuffer();
              rt_thread_mdelay(5000);
              status = 0;
              break;
        }
    }
}
MSH_CMD_EXPORT(oled_display, oled start);

#ifdef __cplusplus
}
#endif
```

`（2）在 applications 文件夹下创建oled_display.h`

```c
/*
 * Copyright (c) 2006-2021, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2022-06-09     ASUS       the first version
 */
#ifndef APPLICATIONS_OLED_DISPLAY_H_
#define APPLICATIONS_OLED_DISPLAY_H_
#ifdef __cplusplus
extern "C"
{
#endif

void oled_display(void);

#ifdef __cplusplus
}
#endif

#endif /* APPLICATIONS_OLED_DISPLAY_H_ */
```

`（3）最终的主函数`

```c
/*
 * Copyright (c) 2006-2020, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2020-09-02     RT-Thread    first version
 */

#include <rtthread.h>
#include <rtdevice.h>
#include "drv_common.h"

extern void wlan_autoconnect_init(void);

int main(void)
{
    rt_uint32_t count = 1;

    rt_pin_mode(LED_PIN, PIN_MODE_OUTPUT);

    wlan_autoconnect_init();

    rt_wlan_config_autoreconnect(RT_TRUE);

    oled_display();

    return RT_EOK;
}

#include "stm32h7xx.h"
static int vtor_config(void)
{
    /* Vector Table Relocation in Internal QSPI_FLASH */
    SCB->VTOR = QSPI_BASE;
    return 0;
}
INIT_BOARD_EXPORT(vtor_config);
```

`（4）参考board.h关于i2c的引脚配置，同款开发板的作者可参照，当然此处的i2c1也可以直接在oled_display.cpp中直接定义，因为前面在RT-Thread Setting中就已经配置好了，可以直接定义，此处只作为一个重定义。`

```c
/*-------------------------- I2C CONFIG BEGIN --------------------------*/

#ifdef BSP_USING_I2C1
#define BSP_I2C1_SCL_PIN    24 // p2 10 PB8
#define BSP_I2C1_SDA_PIN    25 // p2 7 PB9
#endif

#ifdef BSP_USING_I2C3
#define BSP_I2C3_SCL_PIN    119 //p1 29 PH7
#define BSP_I2C3_SDA_PIN    120 //p1 28 PH8
#endif

/*-------------------------- UART CONFIG END --------------------------*/
```

## 六、实验展示

![在这里插入图片描述](https://img-blog.csdnimg.cn/c0cbf5357ce14d7d963684d4338eee9e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/d329c18ee11d45a59252f42bb043663b.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/99f00b0cf72b4e20a0697914e7048f97.png)



## 七、问题总结

`注意：由于我们是在C主程序下调用c++代码，但是RT-Thread对于C++不太友好，需要我们将CPP程序封装成C。同样的在cpp程序中调用C也需要封装`

```c
//如何在封装CPP代码为C：需要我们在.h和.cpp代码中分别对被调用的C++代码都进行封装，具体可参照上文中oled_display.cpp代码

#ifdef __cplusplus
extern "C"
{
#endif

// cpp函数

#ifdef __cplusplus
}
#endif
```

`在使用开发板的过程中，一定需要频繁的去翻阅数据手册和引脚图，有时候开发工具也会莫名的出故障，一般可以尝试下重新构建思路和原理，重启以及寻求大佬帮助。`

---

这次的分享就到这里，有相关问题的欢迎留言私信！

