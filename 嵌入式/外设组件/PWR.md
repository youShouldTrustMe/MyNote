# 概述

PWR(Power Control)电源控制，PWR负责管理STM32内部的电源供电部分，可以实现可编程电压监测器和低功耗模式的功能

可编程电压监测器(PVD)可以监控VDD电源电压，当VDD下降到PVD阀值以下或上升到PVD阀值之上时，PVD会触发中断，用于执行紧急关闭任务

低功耗模式包括：

- 睡眠模式(Sleep)
- 停机模式(Stop)
- 待机模式(Standby)

可在系统空闲时，降低STM32的功耗，延长设备使用时间

## 电源系统结构

![电源的系统结构](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_18_46_48_202410151846192.png)

1. VDDA供电区域，主要负责模拟部分的供电，包括AD转换、温度传感器、复位模块PLL。这些电路供电正极是VDDA，负极是VSSA，其中AD转换器还有两个参考电压引脚，分别为VREF+和VREF-
2. 由两部分组成，左边部分是VDD供电区域，包括IO电路、待机电路、唤醒电路及看门狗，右边部分是VDD通过电压调节器，降压至1.8V，1.8v区域包括CPU核心、存储器和内置数字外设，stm32内部的关键电路，CPU、存储器和外设其实都是以1.8V的低电压运行的。当这些电路想要和外界进行交流时，才会通过IO电路转换到3.3v。==使用低电压运行的主要目的是为了降低功耗，电压越低，内部电路运行的功耗就相对越低==
3. VBAT后备供电区域，其中包括LSE晶体震荡器、后备寄存器、RCC BDCR寄存器和RTC，其中有一个低电压检测开关，VDD有电时，由VDD供电，VDD没电时，由VBAT供电

## 电源监控

电源监控即对某些电源电压(VDD/VDDA/VBAT)进行监控。POR/PDR监控器、PVD监控器、BOR监控器、AVD监控器、VBAT阈值、温度阈值

![电源监控](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_18_51_29_202410151851842.png)

- POR/PDR(power on/down reset):上电/掉电复位
- PVD(programmable voltage detector)):监控V~DD~电压
- BOR(brown out reset):欠压复位
- AVD(analog voltage detector)：监控V~DDA~电压
- VBAT阈值(battery voltage thresholds)：监控V~BAT~电池电压
- 温度阈值(temperature thresholds):监控结温

> 注意：不同芯片包括的电源监控功能不同

### 上电/掉电复位

当VDD或者VDDA电压过低时，内部电路会直接产生复位，让stm32复位，不能乱操作

在POR和PDR之间设置了一个40mV的迟滞电压，大于POR产生复位，小于PDR时保持复位，设置两个阈值的作用就是防止电压在某个阈值附近波动时，造成输出也来回抖动

![上掉电复位](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_18_54_11_202410151854735.png)

### 电压检测器

PVD阈值电压可以使用程序指定，可以自定义调节

PVD的输出信号可以去申请中断，这个中断申请是由外部中断实现的

![电压检测器](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_18_55_13_202410151855604.png)

# 低功耗

STM32具有运行、睡眠、停止和待机四种工作模式。

上电后默认是在运行模式，当内核不需要继续运行时，可以选择后面三种低功耗模式。

![低功耗](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_18_57_38_202410151857005.png)

## 睡眠模式

通过两个库函数进入睡眠模式：WFI和WFE，WFI和WFE是内核的指令

- WFI：wait for interrupt 等待中断，当中断发生，醒来之后的第一件事就是处理中断
- WFE: wait for event 等待事件，这个事件可以事外部中断配置为事件模式，也可以是使能了中断，但是没有配置NVIC，调用的WFE进入的睡眠模式，产生唤醒事件时会立即醒来，醒来之后一般不需要进入中断函数，直接从睡眠的地方继续运行

![睡眠模式](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_18_59_1_202410151859916.png)

## 停止模式

==只有外部中断才能唤醒，其他中断唤醒不了。==

当进入待机模式之后，将会关闭1.8v的时钟，也就是说CPU和外设均不可运行。定时器正在定时的会暂停，串口收发数据也会暂停，但是没有关闭电源，所以CPU和外设的寄存器数据都是维持原状的

![停止模式](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_19_0_11_202410151900282.png)

## 待机模式

只有几个指定的信号才能唤醒：

1. WKUP引脚的上升沿
2. RTC闹钟事件
3. NRST引脚上的外部复位：也就是按下复位键也是可以唤醒的
4. IWDG复位

由于这里的电源关了，所以CPU和外设中的寄存器数据都是不能保持的

![待机模式](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_19_1_37_202410151901492.png)

## 低功耗模式表

从上到下功耗越来越低，且越来越难唤醒

低功耗的启动方法就是关闭CPU时钟和关闭电源（电压调节器）

| 模式                         | 进入                                | 唤醒                                                         | 对内核电路时钟 影响                       | 对VDD区 域时钟的影 响    | 电压调节器                                            |
| ---------------------------- | ----------------------------------- | ------------------------------------------------------------ | ----------------------------------------- | ------------------------ | ----------------------------------------------------- |
| 睡眠 (立即休眠或 退出时休眠) | WFI 、WFE                           | 任意中断 唤醒事件                                            | CPU时钟关，对 其他时钟或模拟 时钟源无影响 | 无                       | 开                                                    |
| 停止                         | PDDS和LPDS位 +SLEEPDEEP位 +WFI或WFE | 任意外部中断（在外 部中断寄存器中设置)                       | 关闭所有内核电 路时钟                     | HS和HSE 定) 的振荡器关闭 | 开启或处于低功 耗模式（依据电 源控制寄存器 PWR_CR的设 |
| 待机                         | PDDS位 +SLEEPDEEP位 +WFI或WFE       | WKUP引脚的上升沿、 RTC闹钟(唤醒/入侵/ 时间戳)事件、NRST 引脚上的外部复位、 IWDG复位 | ：                                        | ：                       | 关                                                    |

## 三种模式下的功耗

同等条件下(T=25°C,VDD=3.3V,系统时钟72MHz)

| 模式     | 主要影响                           | 唤醒时间 | 供应电流(典型值) |
| -------- | ---------------------------------- | -------- | ---------------- |
| 正常模式 | 所有外设正常工作                   | 0        | 51mA             |
| 睡眠模式 | CPU时钟关闭                        | 1.8us    | 29.5mA           |
| 停止模式 | 1.8V区域时钟关闭，电压调节器低功耗 | 5.4us    | 35uA             |
| 待机模式 | 1.8V区域时钟关闭，电压调节器关闭   | 50us     | 3.8uA            |

## 模式的选择

WFI和WFE是待机的触发条件，后面所有的寄存器配置都要在这两条指令之后。

执行WFI(Wait For Interrupt)或者WFE(Vait For Event)指令后，STM32进入低功耗模式。

![模式选择](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_19_7_7_202410151907819.png)

### 睡眠模式

- 执行完WFI、WFE指令后，STM32进入睡眠模式，程序暂停运行，唤醒后程序从暂停的地方继续运行
- SLEEPONEXIT位决定STM32执行完WFI或WFE后，是立刻进入睡眠，还是等STM32从最低优先级的中断处理程序中退出时进入睡眠
- 在睡眠模式下，所有的/○引脚都保持它们在运行模式时的状态
- WFI指令进入睡眠模式，可被任意一个NVIC响应的中断唤醒
- WFE指令进入睡眠模式，可被唤醒事件唤醒

### 停止模式

- 执行完WFI/WFE指令后，STM32进入停止模式，程序暂停运行，唤醒后程序从暂停的地方继续运行
- 1.8V供电区域的所有时钟都被停止，PLL、HSI和HSE被禁止，SRAM和寄存器内容被保留下来
- 在停止模式下，所有的/O引脚都保持它们在运行模式时的状态
- 当一个中断或唤醒事件导致退出停止模式时，HS被选为系统时钟
- 当电压调节器处于低功耗模式下，系统从停止模式退出时，会有一段额外的启动延时
- WFI指令进入停止模式，可被任意一个EXTI中断唤醒
- WFE指令进入停止模式，可被任意一个EXT事件唤醒

### 待机模式

- 执行完WFI/WFE指令后，STM32进入待机模式，唤醒后程序从头开始运行
- 整个1.8V供电区域被断电，PLL、HSI和HSE也被断电，SRAM和寄存器内容丢失，只有备份的寄存器和待机电路维持供电
- 在待机模式下，所有的/O引脚变为高阻态（浮空输入）

- WKUP引脚的上升沿、RTC闹钟事件的上升沿、NRST引脚上外部复位、IWDG复位退出待机模式

# 实验

## 修改主频

> [!TIP]
>
> 注意，由于stm32处于低功耗的状态，所以下载的时候，需要先按住复位键，然后点击下载，松开复位键，这时候就能下载成功了

首先需要修改文件夹(包含时钟的c文件)的属性，从只读变为可读可写

![修改主频](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/15_19_13_1_202410151913066.png)

## 睡眠模式

```c

u8 rxData;

int main(void){
	serial_init();
	Led_Init();
	
	serial_printf("SYSCLK: %d\n",SystemCoreClock);
	while(1){
		if(serial_getRxFlag() == 1){
			rxData = serial_getRxData();
			serial_printf("receive the data: %x\n",rxData);
		}
		Led_On();        //如果设备不处于睡眠模式，将会led将会一闪一闪，否则led就灭
		Delay_s(1);
		Led_Off();
		Delay_s(1);
		__WFI();        //实际上是一条汇编指令，睡眠模式只需要使用这一个代码即可
	}

}
```

## 停止模式

```c

int main(void){
	serial_init();
	redRation_Init();
	Led_Init();
	
	Led_Off();
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR,ENABLE);        //开启PWR时钟
	
	while(1){
		Led_On();
		Delay_s(1);
		Led_Off();
		Delay_s(1);
		PWR_EnterSTOPMode(PWR_Regulator_ON,PWR_STOPEntry_WFI);        //使用WFI模式进入停止模式，这个函数将会将所有的寄存器都配置好
		SystemInit();        //由于从停止模式唤醒之后，系统将会启动HSI内部时钟，所以当唤醒时，我们需要调用该函数重新配置系统时钟
	}

}
```

## 待机模式

```c

int main(void){
	serial_init();
	myrtc_init();
	
	u32 alarm = RTC_GetCounter() + 10;
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR,ENABLE);	//开启PWR时钟
	RTC_SetAlarm(alarm);	//设置闹钟值
	serial_printf("the alarm is %d\n",alarm);
	
	while(1){
		serial_printf("the counter is %d\n",RTC_GetCounter());	//获取实时时钟计数值
		serial_printf("the alrf is %d\n",RTC_GetFlagStatus(RTC_FLAG_ALR));	//获取闹钟产生的信号
		PWR_EnterSTANDBYMode();	//进入待机模式
	}

}
```















































