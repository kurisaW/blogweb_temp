---
title: C素养提升-指针专题
description: 在c语言中，内存单元的地址成为指针，专门用来存放地址的变量，称为指针变量。在不影响理解的情况中，有时对地址、指针和指针变量不区分，统称为指针。
slug: C素养提升-指针专题
date: 2021-06-29 00:00:00+0000
image: cover.jpg
categories:
    - C素养提升
tags:
    - C_pointer
    - C语言
    - 指针

---



## 指针

在c语言中，内存单元的地址成为指针，专门用来存放地址的变量，称为指针变量。

在不影响理解的情况中，有时对地址、指针和指针变量不区分，统称为指针。

#### 地址和变量

`在计算机内存中，每一个字节单元（Byte），都有一个编号，称为地址`。

编译或函数调用时为其分配内存单元。

变量是对程序中数据存储空间的抽象。

#### 指针变量的说明

一般形式如下：

```
<存储类型> <数据类型> * <指针变量名>；

例如,char *pName;
```

指针的存储类型是指针变量本身的存储类型。

指针说明时指定的数据类型不是指针变量本身的数据类型，而是指针目标的数据类型。简称为指针的数据类型。

指针在说明的同时，也可以被赋值初值，成为指针的初始化

一般形式如下：

```
<存储类型> <数据类型> * <指针变量名> = <地址量>；

例如：int a, *pa = &a;
```

在上面的语句中，把变量a的地址作为初值赋了刚说明的int型指针pa。

```
int a = 3;
int *pa = &a;	//相当于:int * pa; pa = &a;
```

下面是一个程序示例：

```c
#include <stdio.h>
int main(int argc, char *argv[])
{
        int a = 10;
        int * p;
        p = &a;
        printf("p:%p a:%p\n",p,&a);
        return 0;
}
```

***可以看到由于整型变量a取地址给指针变量p，最后打印可以发现这两个变量分配的地址都是`0x7fff64003e1c`***

![image-20230112173909015](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301121739386.png)

下面为了更清楚指针变量赋值与指针变量的地址，我们修改代码：

```c
#include <stdio.h>
int main(int argc, char *argv[])
{
        int a = 10;
        int * p;
        p = &a;
        printf("&p:%p sizeof(p):%d\n",&p,sizeof(p));
        printf("p:%p a:%p\n",p,&a);
        return 0;
}
```

![image-20230112175033147](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301121750199.png)

***编译查看结果，可以发现上述的`p =  &a`是作为一个赋值操作，将a的地址赋值给了指针变量p，而指针变量本身还会分配一个地址单元，也就是上面显示的`0x7ffc915b44e0`***

一般我们清楚，在指针中`*p`是作为取值，而`&p`则是取地址，我们再次对程序作出修改：

```c
#include <stdio.h>
int main(int argc, char *argv[])
{
        int a = 10;
        int * p;
        p = &a;
        printf("&p:%p sizeof(p):%d\n",&p,sizeof(p));
        printf("p:%p a:%p\n",p,&a);
        printf("%d %p %d \n",*p,*(&p),*(*(&p)));
        return 0;
}
```

![image-20230112182106265](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301121821672.png)

***那么我们可以看到`a = *p = *(*(&p)) = 10`，仔细理解`*(*(&p))`，也就是对p这个指针变量取地址之后再取值，此时所表示的意思其实一个地址量，也就是`p = *(&p)`,此时对其取地址，可以发现和p所对应的地址相同，此时再对`*(*(&p))`取值，那么也就是对应的一个数据，同理，`&p = &(*(&p))`也就是指针变量p所占用存储区域的地址，作为一个系统随机默认分配的常量，这也是成立的。***

#### 指针的目标

指针指向的内存区域中的数据成为指针的目标。

如果它指向的区域是程序中的一个变量的内存空间，则这个变量成为指针的目标变量。简称指针的目标。

在上述程序中，整型指针变量p所指向的就是整型变量a的内存空间，那么也可以称变量a是指针p的目标变量。

#### 引入指针

引入指针要注意程序中的px, *px和&px三种表示方法的不同意义。设px为一个指针，则：

>px --- 指针变量，它的内容是地址量

>*px --- 指针所指向的对象，它的内容是数据

>&px --- 指针变量所占用的存储区域的地址，是个常量

#### 指针的赋值

指针的赋值运算指的是通过赋值运算符指向指针变量送一个地址值。

向一个指针变量赋值时，送的值必须时地址常量或指针变量，不能时普通的整数（除了赋零）

指针赋值运算常见的有以下几种形式：

```c
// 1、把一个普通变量的地址赋给一个具有相同数据类型的指针：
double x = 15, *px;
px = &x;

// 2、把一个已有地址值的指针变量赋给具有相同数据类型的另一个指针变量：
float a, *px, *py;
px = &a;
py = px;

// 3、把一个数据的地址赋给具有相同数据类型的指针：
int a[20], *pa;
pa = a;	//等价 pa = &a[0]
```

下面是一个程序案例：

```c
#include <stdio.h>
int main(int argc, char *argv[])
{
        int a = 10;
        int * p;
        int * q;

        p = &a;
        q = &a;

        printf("&p:%p %d\n",&p,sizeof(p));
        printf("p:%p a:%p\n",p,&a);
        printf("%d %d\n",a,*p);

        printf("\n\n&q:%p %d\n",&q,sizeof(q));
        printf("%p %d\n",q,*q);

        return 0;
}
```

![image-20230112194158128](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301121941566.png)

***在上述程序中，我们将a的地址量分别传给指针p和指针q，然后打印这两个指针对应的地址，可以发现两者间相差8位`（一个指针在32位的计算机上，占4个字节；一个指针在64位的计算机上，占8个字节。此处由于我是64位系统，所以一个指针对应的就是8位，）`，也就是说指针p和指针q都是指向目标变量a。***

#### 指针运算

指针运算是以`指针变量所存放的地址量作为运算量而进行的运算`。

指针运算的`实质就是地址的计算`。

指针运算的种类是有限的，它只能进行赋值运算、算术运算和关系运算。

| 运算符 | 计算形式 |             意 义              |
| :----: | :------: | :----------------------------: |
|   +    |   px+n   | 指针向地址大的方向移动n个数据  |
|   -    |   px-n   | 指针向地址小的方向移动n个数据  |
|   ++   |   px++   | 指针向地址小的方向移动1个数据  |
|   --   |   px--   | 指针向地址小的方向移动1个数据  |
|   -    |  px-py   | 两个指针之间相隔数据元素的个数 |



* 不同数据类型的两个指针实行加减整数运算是无意义的。

* px+n表示的实际位置的地址量是：(px) + sizeof(px的类型)*n
* px-n表示的实际位置的地址量是：(px) - sizeof(px的类型)*n 
* px-py运算的结果是两指针指向的地址位置之间相隔数据的个数，因此两指针相减不是两指针持有的地址量相减的结果，而是一个整数值，表示两指针之间相隔数据的个数。
* 两指针之间的关系运算表示它们指向的地址位置之间的关系。指向地址大的指针大于指向地址小的指针。
* 指针与一般整型变量之间的关系运算没有意义。但可以和零进行等于或不等于的关系运算，判断指针是否为空。

注意：

`两个指针之间的运算需要有连续的内存地址，否则会发生预想不到的错误`，示例如下：

![image-20230112210030039](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301122100423.png)

正确的运行示例：

![image-20230112210312170](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301122103226.png)

`这里也可以与上面的知识点相对应：px-py运算的结果是两指针指向的地址位置之间相隔数据的个数`

下面是一些指针运算的示例：

![image-20230112212116348](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301122121416.png)

上述程序重要的就是理顺指针的关系以及运算符优先级问题。

---

知识扩展：

**在32位系统与64位系统下，不同数据类型所对应的字节数--->**

|  数据类型   | 32位  | 64位  |                           备注                           |
| :---------: | :---: | :---: | :------------------------------------------------------: |
|    char     |   1   |   1   |                                                          |
|    short    |   2   |   2   |                                                          |
|     int     |   4   |   4   |                                                          |
|    long     |   4   |   8   |                      32位与64位不同                      |
|    float    |   4   |   4   |                                                          |
|   char *    |   4   |   8   |           其他指针类型如long *，int *也是如此            |
|  long long  |   8   |   8   |                                                          |
|   double    |   8   |   8   |                                                          |
| long double | 10/12 | 10/16 | 有效位10字节。32位为了对其实际分配12字节；64位分配16字节 |



## 指针与数组

#### 指针对数组的访问

在c语言中，数组的指针是指数据在内存中的起始地址，数组元素的地址是指数组元素在内存中的起始地址。

一维数组的数组名为以为数组的指针（起始地址）。

例如：

```c
double x[8];
```

因此，x为x数组的起始地址。

> 设指针变量px的地址值等于数组指针x（即指针变量px指向数组的首元素），则：
>
> **`x[i]、*(px+i)、 *(x+i)和px[i]具有完全相同的功能，也就是说，x[i] = *(px+i) = *(x+i) = px[i]`**：访问数组第i+1个数组元素，下面参照示例：

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int a[] = {4, 6, 2, 8, 9,4, 7, 1, 5};
        int *p, i, n;

        p = a;
        n = sizeof(a) / sizeof(int);

        printf("%d\n",n);
        printf("%p %p %p\n",a, a+1, a+2);
        
        for(i = 0; i < n; i++)
                printf("%d %d %d %d\n",a[i],*(p+i),*(a+i),p[i]);
        puts("");
        return 0;
}
```

![image-20230113163021566](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301131630839.png)

那么参照上述程序，在某种程度上p和a是否是等效的呢？其实这还是有区别的，数组a作为一个整型数组常量，而整型指针p则是一个变量，只能说在他们有相似的使用方法，这种情况还是需要区分的。

`注意:`

* 指针变量和数组在访问数组中元素时，一定条件下其使用方法具有相同形式，因为指针变量和数组名都是地址量

* 但是指针变量和数组的指针（或叫数组名）在本质上不同，指针变量时地址变量，而数组的指针是地址常量

#### 程序案例

程序1：下面编写一个程序，使用指针将整型数组中n个数按反序存放：

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int a[] = {4, 6, 2, 8, 9,4, 7, 1, 5};
        int *p, *q, t;

        int n = sizeof(a) / sizeof(int);
        
        p = a;
        q = &a[n-1];


        while(p < q)
        {
                t =*p;
                *p = *q;
                *q = t;
                p++;
                q--;
        }

        for(t = 0;t < n;t++)
        {
                printf("%d ",a[t]);
        }  
        puts("");
        return 0;
}
```

![image-20230113170338589](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301131703949.png)

程序2

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int a[] = {4, 6, 2, 8, 9,4, 7, 1, 5};
        int *p, i, n;

        p = a;
        n = sizeof(a) / sizeof(int);

        printf("%d\n",n);
        printf("%p %p %p\n",a, a+1, a+2);

        for(i = 0; i < n; i++)
                printf("%d %d %d %d\n",a[i],*(p+i),*(a+i),p[i]);
        puts("");

        p++;
        printf("%d\n",p[1]);
        return 0;
}
```

![image-20230113171028194](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301131710490.png)

这里我们发现，数组下标p[1]的本质，其实就是*(p+1)，前面已经p++了，此时的p[1]其实就相当于 *(p+1+1)，也就是 *p[2] = 2

**知识点：**

**数组p[i]，其实就相当于*(p+i)，也就是：p[i] = *(p+i)**

## 指针与二维数组

#### 二维数组的性质

多维数组就是具有两个或两个以上下标的数组。

在c语言中，二维数组的元素连续存储，按行优先存取。

下面看程序案例：

案例一：

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int a[3][2] = { {1, 6}, {9, 12}, {61, 12}};
        int *p, i, n;

        n = sizeof(a) / sizeof(int);
        printf("%d\n",n);
        p = a[0];
        printf("%d\n",sizeof(a[0]));
        
        printf("%p %p\n", p, p+1);
        printf("%p %p\n", a, a+1);
        
        for(i = 0;i < n; i++)
                printf("%d ", *(p+i));
        puts("");
        
        return 0;
}
```

![image-20230113173618278](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301131739597.png)

上述程序中可以看出：a[0]为8个字节大小，所以可以看出数组名加1，移动的是一行元素。

案例二：

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int a[3][2] = {{1, 6}, {9, 12}, {61, 12}};
        int *p, i, n;

        n = sizeof(a) / sizeof(int);

        printf("%p %p\n", a, a+1);
        printf("%p %p\n", a[0], a[0]+1);
    	printf("%p %p\n", *a, *a+1);

        return 0;
}
```

![image-20230113195318122](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301131953543.png)

从上述程序中可以看出，a与a+1之间是相隔8个字节，而a[0]与a[0]+1则相隔4个字节，我们发现地址的移动步长发生变化了，原本是按行地址索引，加入指针即*a+1后，则变成了按列索引，更准确的说是原本的一行元素的索引变成了单个元素的索引。

#### 行指针(数组指针)

`二维数组名代表数组的起始地址，数组名加1，是移动一行元素`。因此，**二维数组名常被称为行地址**

**存储行地址的指针变量，叫做`行指针变量`。**形式如下：

> `<存储类型> <数据类型> (*<指针变量名>)[表达式];`
>
> 例如：int a[2] [3];	 int (*p)[3]

**`注意：！！方括号中的常量表达式表示指针加1，移动几个数据。当用行指针操作二维数组时，表达式一般写成1行的元素个数，即列数。`**

我们用一个程序案例来解释：

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int a[3][2] = {{1, 6}, {9, 12}, {61, 12}};
        int (*p)[2];

        p = a;

        printf("%p %p\n",a ,a+1);
        printf("%p %p\n",p ,p+1);

        printf("%d, %d, %d, %d\n",a[1][1], p[1][1], *(*(a+1)), *(*((p+1)+1)));  
        printf("%d %d\n",*(*(a+1)),*(*((a+1)+1)));
   		printf("%p %p\n",&(*(a+1)),&(*((a+1)+1)));
        return 0;
}
```

![image-20230113203626795](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301132036861.png)

根据上述程序，其实就很好理解二维数组与数组指针的关系了，在二维数组中，对于指针的使用，一个取值符号*代表的其实就是行指针的地址量，而两个取值符号**代表的就是对行指针的第一个元素进行取值操作；同理，对一个地址量【 *(a+1)】进行取地址操作&，代表的就是取地址【&( *(a+1))】。

## 字符指针与字符串

#### 字符指针的定义

C语言通过使用字符数组来处理字符串。通常，我们把char数据类型的指针变量称为字符指针变量。字符指针变量与字符数组有着密切关系，它也被用来处理字符串。

#### 字符指针的初始化

**初始化字符指针是把内存中字符串的首地址赋予指针，**并不是把该字符串复制到指针中。

```c
char str[] = "Hell World";
char *p = str;
```

在C编程中，**当一个 字符指针指向一个字符串常量时，不能修改指针指向的对象的值。**

```c
char *p = "Hello World";	//此处直接让一个字符指针等于字符串，其实存取的是这段字符串常量的首地址
*p = 'h';	//错误，字符串常量不能修改
```

#### 程序案例

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        char *p1 = "hello world!";
        char *p2 = "hello world!";

        printf("&p1=%p %p %s\n", &p1, p1, p1);
        printf("&p2=%p %p %s\n", &p2, p2, p2);
        return 0;

}
```

![image-20230114101732695](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301141017174.png)

此处我们可以看到，由于字符指针的内容都是`hello world!`，也就是申请了一段字符串空间存取的内容为`hello world!`，当我们打印字符指针p1和p2指向的地址时可以发现都指向了`0x4006a4`，接着我们打印指针存放的地址，可以发现`&p1=0x7ffc8d801cd8`、`&p2=0x7ffc8d801ce0`，也就是说指针申请的空间都在栈中，而字符串常量空间的申请则是放在静态区**`（放在静态区的有三种情况：全局变量、static修饰的局部变量、常量）`**

## 指针数组

#### 指针数组的定义

**所谓指针数组是指若干个具有相同存储类型和数据类型的`指针变量`构成的集合。**

指针数组的一般说明形式：

> <存储类型> <数据类型> *<指针数组名>[<大小>]；
>
> **指针数组名表示该指针数组的起始地址**

#### 指针数组的声明

声明一个指针数组：

```c
double *pa[2], a[2][3];
```

把一维数组a[0]和a[1]的首地址分别赋予指针数组的数据元素pa[0]和pa[1]：

```c
pa[0] = a[0];	//等价pa[0] = &a[0][0]
pa[1] = a[1];	//等价pa[1] = &a[1][0]

//此时pa[0]指向了一维数组a[0]的第一个元素a[0][0],而pa[1]指向了一维数组a[1]的第一个元素a[1][0]
```

#### 程序案例

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int *p[3];
        int a[] = {3, 6, 1, 7, 10};

        p[0] = a;
        p[1] = a + 1;
        p[2] = a + 3;

        printf("%d %d %d\n",a[0], a[1], a[3]);
        printf("%d %d %d\n",*(p[0]),*(p[1]),*(p[2]));

        printf("%p %p\n",p[0],p[1]);
        printf("%p %p\n",&p[0],&p[1]);
        return 0;
}
```

![image-20230114111849051](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301141118523.png)

> 问：指针数组名相当于什么样的指针？	答：二级指针。

## 多级指针

#### 多级指针的定义

把一个指向指针变量的指针变量，称为多级指针。

对于指向处理数据的指针变量称为一级指针变量，简称一级指针变量，简称一级指针。

对于指向一级指针的指针变量称为二级指针变量，简称一级指针变量，简称二级指针。

二级指针变量的说明形式如下：

`<存储类型> <数据类型> **<指针名>；`

#### 多级指针的运算

**`指针变量加1，是向地址大的方向移动一个目标数据。`**类似的道理，多级指针运算也是以其目标变量为单位进行偏移。

比如：int **p; p+1移动一个int *变量所占的内存空间。

#### 程序案例

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int a[] = {3, 6, 9};
        int *p[2] = {&a[0],&a[1]};

        int **q;

        q = &p[0];
        q = p;

        printf("%d %d\n",a[0],a[1]);
        printf("%d %d\n",*p[0],*p[1]);

        printf("%d %d %d\n",a[0],*p[0],**q);

        return 0;
}
```

![image-20230114171007367](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301141710805.png)

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        char *s[] = {"apple", "pear", "potato"};

        int i, n;
        n = sizeof(s) / sizeof(char *);
        printf("%d %d %d\n",n,sizeof(s),sizeof(char *));

        return 0;
}
```

![image-20230114172259973](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301141723373.png)

## void指针

#### void指针的定义

void指针是一种不能确定数据类型的指针变量，它可以`通过强制类型转换让该变量指向任何数据类型的变量。`

一般形式为:

> void * <指针变量名>

**`对于void指针，在没有强制类型转换前，不能做任何指针的算数运算。`**

#### 程序案例

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int m = 10;
        double n = 3.14;
        void *p, *q;

        p = &m; //(void *) &m
        printf("%d %d\n",m,*(int *)p);

        q = &n; //(void *)&n
        printf("%.2lf %.2lf\n",n, *(double *)q);
        return 0;
}
```

![image-20230114174233538](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301141742757.png)

```c
#include <stdio.h>
int main(int argc,char *argv[])
{
        int a[] = {6, 3, 2, 7, 9, 6};
        int i, n;
        void *p;

        p = a;
        n = sizeof(a) / sizeof(int);
        for(i = 0;i < n;i++)
                printf("%d ",*((int *)p +i));
        puts("");
        return 0;
}
```

![image-20230114175011554](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202301141750626.png)

此处需要注意：对于void指针，在没有强制类型转换前，不能做任何指针的算数运算。所以在上述程序中对void指针的使用首先需要`(int *)p`进行强转，之后对于用户的算数运算就没什么问题了。

## const修饰指针

#### 常量化指针目标表达式

一般说明形式如下：

> const <数据类型> * <指针变量名>[= <指针运算表达式>]

常量化指针目标是限制通过指针改变其目标的数值，`但<指针变量> --->存储的地址值可以修改。`

#### 常量化指针变量

一般说明形式如下：

> <数据类型> * const <指针变量名>[= <指针运算表达式>]

使得<指针变量>存储的地址值不能修改。`但可以通过* <指针变量名>可以修改指针所指向变量的数值。`

#### 常量化指针变量及目标表达式

一般说明形式如下：

> const <数据类型> * const <指针变量名>[= <指针运算表达式>]

`常量化指针变量及目标表达式，使得既不可以修改<指针变量名>的地址，也不可以通过* <指针变量名>修改指针所指向变量的值。`

