# 时间戳

## 概述

Unix时间戳(Unix Timestamp)定义为从UTC/GMT的1970年1月1日0时0分0秒开始所经过的秒数，不考虑闰秒

时间戳存储在一个秒计数器中，秒计数器为32位/64位的整型变量

世界上所有时区的秒计数器相同，不同时区通过添加偏移来得到当地时间

![时间线](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/7_17_9_13_202409071709780.png)

## GMT和UTC

GMT(Greenwich Mean Time)格林尼治标准时间是一种以地球自转为基础的时间计量系统。它将地球自转一周的时间间隔等分为24小时，以此确定计时标准

UTC(Universal Time Coordinated)协调世界时是一种以原子钟为基础的时间计量系统。它规定铯133原子基态的两个超精细能级间在零磁场下跃迁辐射9,192,631,770周所持续的时间为1秒。当原子钟计时一天的时间与地球自转一周的时间相差超过0.9秒时，UTC会执行闰秒来保证其计时与地球自转的协调一致

## 时间戳转换

| **函数**                                                     | **作用**                               |
| ------------------------------------------------------------ | -------------------------------------- |
| time_t time(time_t*);                                        | 获取系统时钟                           |
| struct tm* gmtime(const time_t*);                            | 秒计数器转换为日期时间（格林尼治时间） |
| struct tm* localtime(const time_t*);                         | 秒计数器转换为日期时间（当地时间）     |
| time_t mktime(struct tm*);                                   | 日期时间转换为秒计数器（当地时间）     |
| char* ctime(const time_t*);                                  | 秒计数器转换为字符串（默认格式）       |
| char* asctime(const struct tm*);                             | 日期时间转换为字符串（默认格式）       |
| size_t strftime(char*, size_t, const char*, const struct tm*); | 日期时间转换为字符串（自定义格式）     |

> [!TIP]
>
> 需要引入time.h才可以使用上述函数

![时间戳转换](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/7_17_13_42_202409071713702.png)

实验：时间戳转换

```c
#include<time.h>

time_t time_cnt;        //秒计数器
struct tm time_date;        //日期时间数据类型
char* time_str;        //字符串数据类型

int main(void){

    time_cnt = time(NULL);        //这里的time_cnt实际上就是一个长整秒，int64类型的
    time(&time_cnt);        //和上面的代码作用一样

    time_date = *gmtime(&time_cnt);        //加上*就可以取结构体的内容，将长整秒转换为伦敦时间，也就是格林尼治时间
    time_date.tm_year += 1900;        //由于时间的基准是1900年，所以需要加上1900
    time_date.tm_mon += 1;

    time_date = *localtime(&time_cnt);        //转换长整秒为当地时间，该函数会将时间转换为当地时间

    time_cnt = mktime(&time_date);        //是转换的逆过程，也就是将日期时间转换为长整秒

    time_str = ctime(&time_cnt);        //将长整秒转换为字符串

    time_str = asctime(&time_date);        //将数据类型转换为字符串类型

    char t[50];
    strftime(t,50,"%H-%M-%S",&time_date);        //类似于printf函数，将数据类型字符串化，可以自定义输出
}
```

# RTC

实时时钟(Real Time Clock,RTC),本质是一个计数器，计数频率常为秒，专门用来记录时间。

![RTC时钟源](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/7_17_20_1_202409071720245.png)

> [!NOTE]
>
> 普通定时器拿来作时钟可行吗？
>
> > 普通定时器无法掉电运行！

RTC特性

1. 能提供时间（秒钟数）
2. 能在MCU掉电后运行
3. 低功耗

RTC有以下特点及功能

- RTC是一个独立的定时器，可为系统提供时钟和日历的功能
- RTC和时钟配置系统处于后备区域，系统复位时数据不清零，VDD(2.0 ~ 3.6V)断电后可借助VBAT（1.8~3.6V)供电继续走时
- 32位的可编程计数器，可对应Unix时间戳的秒计数器
- 20位的可编程预分频器，可适配不同频率的输入时钟
- 可选择三种RTC时钟源：
  1. HSE时钟除以128（通常为8MHz/128)
  2. LSE振荡器时钟（通常为32.768KHz)
  3. LSI振荡器时钟(40KHz)

## 常见的解决方案

![RTC一般方案](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/7_17_32_36_202409071732282.png)

| 对比因素 | 内部RTC          | 外置RTC          |
| -------- | ---------------- | ---------------- |
| 信息差异 | 提供秒/亚秒信号  | 提供秒信号和日历 |
| 功耗     | 功耗高           | 功耗低           |
| 体积     | 不用占用额外体积 | 体积大           |
| 成本     | 成本低           | 成本高           |

> [!TIP]
>
> 1. 一般都需要设计RTC外围电路；
> 2. 一般都可以给RTC设置独立的电源；
> 3. 多数RTC的寄存器采用BCD码存储

## 框图

RTC时钟来源：

- 接高速晶振，一般接主晶振（8MHz），通过128分频，可以产生RTCCLK时钟。为了在RTC的时钟为1Hz，所以使用高速时钟的时候需要先进行128倍分频
- 接低速晶振，可以直接给RTCCLK，OSC32的晶振是内部RTC的专用时钟，这个晶振的值不是随便选的，一般和RTC有关的，都是统一的数值，就是32.768KHz（2的次方数15次方），需要注意的是，只有OSC32这个时钟在设备断电后是由VBAT提供电源，其他的LSI和HSE都不接VBAT，所以最好使用OSC32

![RTC系统框图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/9_11_54_0_202409091154706.png)

> [!NOTE]
>
> 框图中的组件：
>
> 1. RTC预分频器
> 2. 32位可编程计数器
> 3. 待机唤醒
> 4. RTC控制寄存器与APB1接口
>
> 3个时钟源：
>
> 1. HSE/128
> 2. LSI 40kHz
> 3. LSE 32.768kHz

### RTC预分频器：

一般来说RTCCLK进来都不是1MHz的，所以需要进行分频，这个分频器由两个寄存器组成重装载寄存器（RTC_PRL）和余数寄存器（RTC_DIV）。

我们在RTC_PRL中写入6，则代表7分频，因为是从零开始的。

RTC_DIV实际上就是一个计数器，当计数记到7时，自然溢出，产生一个时钟，这就是分频的原理，这里的div是一个递减计数器，每来一个输入时钟，DIV的值就自减一次，当自减为0时，再来一个输入时钟，产生一个溢出信号，同时DIV从PRL获取重装值，继续自减。

### 32位可编程计数器

我们可以将这个计数器看作是unix时间戳的秒计数器，所以我们可以使用time.h头文件中的函数读取并且转换时间

此外，这里还有一个RTC_ALR闹钟寄存器，32位寄存器，在ALR中写入一个秒数，设置一个闹钟，当CNT的值和闹钟值相等时，这时就会产生RTC_Alarm闹钟信号，通往右边的中断系统，在中断函数中就可以执行相应的操作了，同时，这个闹钟还可以让STM32退出待机模式

### 待机唤醒

闹钟信号和wake up引脚都可以唤醒设备

### 中断系统

将会产生三个中断：

1. RTC_Second（秒中断）：每秒进入一次中断
2. RTC_Overflow（溢出中断）：32位的计数器记满溢出，将会产生一个中断
3. RTC_Alarm（闹钟中断）：当计数器和闹钟值相等时，将会产生一个中断

### 简易框图

![RTC简易框图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/9_14_24_31_202409091424697.png)

## 注意事项

- 执行以下操作将使能对BKP和RTC的访问：
  1. 设置RCC APB1ENR的PWREN和BKPEN,使能PWR和BKP时钟
  2. 设置PWR CR的DBP,使能对BKP和RTC的访问
- 若在读取RTC寄存器时，RTC的APB1接口曾经处于禁止状态，则软件首先必须等待RTC CRL寄存器中的RSF位（寄存器同步标志）被硬件置1
- 必须设置RTC CRL寄存器中的CNF位，使RTC进入配置模式后，才能写入RTC_PRL、RTC_CNT、RTC_ALR寄存器
- 对RTC任何寄存器的写操作，都必须在前一次写操作结束后进行。可以通过查询RTC_CR寄存器中的RTOFF,状态位，判断RTC寄存器是否处于更新中。仅当RTOFF状态位是1时，才可以写入RTC寄存器

# BKP

BKP(Backup Registers)是备份寄存器

BKP可用于存储用户应用程序数据。当VDD(2.0-3.6V)电源被切断，他们仍然由VBAT(1.8~3.6V)维持供电。当系统在待机模式下被唤醒，或系统复位或电源复位时，他们也不会被复位

TAMPER引脚产生的侵入事件将所有备份寄存器内容清除

RTC引脚输出RTC校准时钟、RTC闹钟脉冲或者秒脉冲

存储RTC时钟校准寄存器

用户数据存储容量：20字节（中容量和小容量)/84字节（大容量和互联型)

![BKP系统框图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/7_17_16_23_202409071716671.png)

> [!tip]
>
> 橙色部分就是后备区域，当VDD主电源掉电时，后备区域仍然可以由VBAT的备用电池供电，当VDD主电源上电时，后备区域供电就会由VBAT切换到VDD

# 实验

## 测试备份数据寄存器

```c
int main(void){
	serial_init();
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP,ENABLE);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR,ENABLE);
	PWR_BackupAccessCmd(ENABLE);        //备份访问控制
//        BKP_WriteBackupRegister(BKP_DR1,0x1234);
	serial_printf("the backup value is %x\n", BKP_ReadBackupRegister(BKP_DR1));
}
```

## 实时时钟

```c

u16 myRtc_time[] = {2023,1,1,23,59,59};        //注意在写数据时最好不要在前面随便补零，因为c语言中的0代表了八进制

void myrtc_init(void)
{
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP,ENABLE);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR,ENABLE);
	PWR_BackupAccessCmd(ENABLE);        //备份访问控制
	
	if(BKP_ReadBackupRegister(BKP_DR1) != 0xA5A5){        //当系统完全断电时，BKP_DR1必定为0，所以将会执行以下代码,复位bkp的寄存器也不会为0
//        RCC_LSEConfig(RCC_LSE_ON);        //配置外部时钟
//        while(RCC_GetFlagStatus(RCC_FLAG_LSERDY) != SET);        //LSE并不是说启动就能启动的，所以需要等待标志位变为1
//        RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);        //选择LSE作为时钟源
	
		RCC_LSICmd(ENABLE);        //当LSE起振不了将会一直卡在读取标志位的地方，可以采用LSI
		while(RCC_GetFlagStatus(RCC_FLAG_LSIRDY) != SET);
		RCC_RTCCLKConfig(RCC_RTCCLKSource_LSI);
		
		RCC_RTCCLKCmd(ENABLE);
		
		RTC_WaitForSynchro();        //等待同步
		RTC_WaitForLastTask();        //等待上次写入操作完成
		
//        RTC_SetPrescaler(32768 - 1);        //设置分频系数，我们选择的是LSE，LSE的晶振频率为32.768Khz，也就是32768HZ，如果想要变成1hz，那么就要32768分频
	RTC_SetPrescaler(40000 - 1);
	RTC_WaitForLastTask();        //等待写入完成
	
		myrtc_setTime();
		
		BKP_WriteBackupRegister(BKP_DR1,0xA5A5);        //第一次启动完毕之后，将这个标志位置为0xA5A5
	}else{
		RTC_WaitForSynchro();        //等待同步
		RTC_WaitForLastTask();        //等待上次写入操作完成
	}
	

}        

void myrtc_setTime(void)
{
	time_t time_cnt;
	struct tm time_date;
	
	time_date.tm_year = myRtc_time[0] - 1900;        //需要先设置一个初始化时间
	time_date.tm_mon = myRtc_time[1] - 1;
	time_date.tm_mday = myRtc_time[2];
	time_date.tm_hour = myRtc_time[3];
	time_date.tm_min = myRtc_time[4];
	time_date.tm_sec = myRtc_time[5];
	
	time_cnt = mktime(&time_date);        //将日期数据类型转换为长整秒
//        time_cnt = time_cnt - 8 * 60 * 60;        //适应于北京时间
	RTC_SetCounter(time_cnt);
	RTC_WaitForLastTask();
}

void myrtc_readTime(void)
{
	time_t time_cnt;
	struct tm time_date;
	
	time_cnt = RTC_GetCounter();
//        time_cnt = time_cnt + 8 * 60 * 60;         //这样就可以计算出北京时间了，也就是在小时的位加上8
	
	time_date = *localtime(&time_cnt);
	
	myRtc_time[0] = time_date.tm_year + 1900;        //需要先设置一个初始化时间
	myRtc_time[1] = time_date.tm_mon + 1;
	myRtc_time[2] = time_date.tm_mday;
	myRtc_time[3] = time_date.tm_hour;        //
	myRtc_time[4] = time_date.tm_min;
	myRtc_time[5] = time_date.tm_sec;
	
}


int main(void){
	serial_init();
	myrtc_init();
	
	while(1){
		myrtc_readTime();
		serial_printf("date: %d-%d-%d\n time: %d-%d-%d\n",myRtc_time[0],myRtc_time[1],myRtc_time[2],myRtc_time[3],myRtc_time[4],myRtc_time[5]);
	}

}
```





































