# WDG

## 简介

![image-20240709151957478](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_19_57_202407091519564.png)

## IWDG VS WWDG

|            | **IWDG独立看门狗**           | **WWDG窗口看门狗**                  |
| ---------- | ---------------------------- | ----------------------------------- |
| 复位       | 计数器减到0后                | 计数器T[5:0]减到0后、过早重装计数器 |
| 中断       | 无                           | 早期唤醒中断                        |
| 时钟源     | LSI（40KHz）                 | PCLK1（36MHz）                      |
| 预分频系数 | 4、8、32、64、128、256       | 1、2、4、8                          |
| 计数器     | 12位                         | 6位（有效计数）                     |
| 超时时间   | 0.1ms~26214.4ms              | 113us~58.25ms                       |
| 喂狗方式   | 写入键寄存器，重装固定值RLR  | 直接写入计数器，写多少重装多少      |
| 防误操作   | 键寄存器和写保护             | 无                                  |
| 用途       | 独立工作，对时间精度要求较低 | 要求看门狗在精确计时窗口起作用      |

## 溢出时间计算原理

计算步骤如下

1. 首先需要知道看门狗的时钟频率，假设是60kHz
2. 设置分频系数，现在设置为8分频，那么分完频之后的时钟就是60_000Hz / 8=75_00Hz,也就是1s跳动7500次
3. 如果想要设置溢出时间为0.5s的话，那么就是用0.5*7500=3750，就是设置为3750

# IWDG

## 概述

![image-20240709152229375](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_22_29_202407091522425.png)

## 作用

![image-20240709152240863](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_22_40_202407091522912.png)

## 工作原理

![](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_22_40_202407091522912.png)

## 框图

实际上就是一个递减计数器，在程序运行期间，适当的提高计数值就可以避免出现复位，手动重装重装载寄存器的操作就是喂狗,递减计数器是12位的，所以最大的计数是 $2 ^ {12}-1 = 4095$。

- 当递减计数器自减到0时，会产生一个IWDG的复位信号

- 当我们在重装载寄存器中写好值之后，在键寄存器里写一个特定的数据，控制电路进行喂狗，这个时候重装值就会将数值赋给当前计数器


下图分为上下两部分，上面部分工作在1.8v电压下，下面部分工作在VDD电压下

![image-20240709152336754](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_23_36_202407091523816.png)

## 键寄存器

由于IWDG_SR是写保护的，所以不用担心该寄存器被干扰，为了防止另外两个寄存器受到干扰，使用键寄存器对寄存器进行写保护，一旦两个寄存器写入之后会被再次保护起来

![image-20240709152423740](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_24_23_202407091524804.png)

## 配置寄存器

![image-20240709152437935](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_24_37_202407091524984.png)

## 计算溢出时间

![image-20240709152504849](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_25_4_202407091525907.png)

![image-20240709152510298](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_25_10_202407091525352.png)

# WWDG

## 概述

![image-20240709152537701](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_25_37_202407091525758.png)

## 作用

![image-20240709152548330](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_25_48_202407091525369.png)

## 工作原理

中断产生在复位的前一刻，复位产生时间0x3f，中断产生时刻0x40

![image-20240709152600278](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_26_0_202407091526336.png)

## 框图

这里的WDGTB和上面的PSC都是一个东西，都是预分频器

从T6-T0一共七个位，但是却是6位递减计数器，实际上只有T5-T0是计数器，T6用作溢出标志位，T6位为1时，表示计数器未溢出，T6位为0时，表示计数器溢出

假设我们初始时写入111_1111,依次递减111_1110->111_1101……直到减到100_0000（0x40）时，如果再向下减，T6位将会变为0，T6将会产生一个信号去往或门

WDGA是激活位，也就是使能位，WDGA写入1，启用窗口看门狗

计算最早界限计数值

W6-W0中是最早界限计数值，这些值写入之后是固定不变的，一旦执行写入CR操作时，与门打开，写入CR其实就是写入计数器，也就是喂狗

喂狗时，比较器开始工作，当前计数器T6-0 > 窗口值W6-0，比较结果为1，通过或门，产生复位信号

也就是说喂狗的时候，把当前计数值和预设的窗口计数值进行比较，如果发现狗的余粮还比较充足，但是喂的很频繁，那必定是存在问题的，所以会产生一个复位信号

![image-20240709152649212](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_26_49_202407091526265.png)

## 工作特性

![image-20240709152703906](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_27_3_202407091527986.png)

## 计算溢出时间

乘以4096的原因是在LSI时钟信号后面有一个固定的4096分频

![image-20240709152723314](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_27_23_202407091527371.png)

![image-20240709152729060](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_15_27_29_202407091527109.png)

# 实验

## IWDG

```c
int main(void){
	serial_init();
	key_init();
	
	if(RCC_GetFlagStatus(RCC_FLAG_IWDGRST) == SET){
		serial_printf("IWDG RST!\n");
		RCC_ClearFlag();
	}else{
		serial_printf("RST!\n");
	}
	
	IWDG_WriteAccessCmd(IWDG_WriteAccess_Enable);        //解除写使能，可以往键寄存器中写入数据
	IWDG_SetPrescaler(IWDG_Prescaler_16);        //设置预分频，现设置超时时间为1s，那就要求喂狗时间不能超过1s，需要查看手册，选择合适的分频系数
	IWDG_SetReload(2499);        //设置重装值，通过公式计算，重装值计算：rlr = 1 * 40000 / 16 = 2500
	IWDG_ReloadCounter();        //先喂一次狗
	IWDG_Enable();        //启动看门狗
	while(1){
		getKey();
		IWDG_ReloadCounter();        //喂狗
		serial_printf("FEED!\n");
		Delay_ms(300);
		serial_printf("\n");
		Delay_ms(300);
		
	}

}
```

## WWDG

```c


int main(void){
	serial_init();
	key_init();
	
	if(RCC_GetFlagStatus(RCC_FLAG_WWDGRST) == SET){
		serial_printf("WWDG RST!\n");
		RCC_ClearFlag();
	}else{
		serial_printf("RST!\n");
	}
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_WWDG,ENABLE);        //wwdg和iwdg不同，wwdg的时钟来源是APB1
	
	WWDG_SetPrescaler(WWDG_Prescaler_8);        //设置预分频,我们想要设置超时时间为50ms，T = 0.05 * 36000000 /4096/ 2^3 = 54.93164 = 55,所以T[5:0] = 54
	WWDG_SetWindowValue(21 | 0x40);        //设置窗口值，我们窗口时间设置为30ms，窗口时间W = 54 - 0.03 * 36000000 / 4096/2^3  = 54 - 33 = 21 
	WWDG_Enable(54 | 0x40);        //这里使用|0x40的目的是为了让T[6]=1
	
	
	while(1){
		getKey();
	
		serial_printf("FEED!\n");
		Delay_ms(10);
		serial_printf("\n");
		Delay_ms(20);
		WWDG_SetCounter(54 | 0x40);        //喂狗，这里不能再WWDG_ENABLE后面直接喂狗，因为执行时间太短，在非窗口期喂狗是会复位的
		
	}

}


```

