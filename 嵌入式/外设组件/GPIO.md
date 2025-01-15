# 是什么

![image-20240705085552462](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_8_55_52_202407050855539.png)

# 特点

1. 不同型号，IO口数量可能不一样，可通过选型手册快速查询

2. 快速翻转，每次翻转最快只需要两个时钟周期（F1最高速度可以到50Mhz）

3. 每个IO口都可以做中断

4. 支持8种工作模式

# 电气特性

![image-20240705085649207](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_8_56_49_202407050856261.png)

# 基本结构

所有的GPIO都是挂载在APB2外设总线上的（每个GPIO包含16个引脚）

- 在GPIO模块中包含寄存器和驱动器
- 寄存器是一段特殊的存储器，内核可以通过APB2总线对寄存器进行读写，这样就可以完成输出电平和读取电平的功能了，寄存器的每一位对应一个引脚，由于每个GPIO模块上面只有16个引脚，所以寄存器只有低16位用到了，高16位保留。驱动器用来增加信号的驱动能力，寄存器只负责存储数据。

![image-20240705104553778](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_45_53_202407051045843.png)

# 端口的基本结构

![image-20240705104714238](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_47_14_202407051047297.png)

## 保护二极管

对输入电压进行限幅，当电压比3.3v高，那么上方的保护二极管将会导通，电流就直接流到VDD，不会进入内部电路

## 施密特触发器

![image-20240705104758837](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_47_58_202407051047900.png)

在施密特触发器的箭头前面是数字量，在施密特触发器的后面的是模拟量

## P-MOS管和N-MOS管

![image-20240705104842694](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_48_42_202407051048750.png)

# 输入输出模式

## 概述

输入输出都相对于ST芯片而言的，比如输入就是从io口进如芯片内部，输出就是信号从芯片到IO引脚

| **GPIO**八种模式   | **特点及应用**                              |
| ------------------ | ------------------------------------------- |
| **输入浮空**       | 输入用，完全浮空，状态不定                  |
| **输入上拉**       | 输入用，用内部上拉，默认是高电平            |
| **输入下拉**       | 输入用，用内部下拉，默认是低电平            |
| **模拟功能**       | ADC、DAC                                    |
| **开漏输出**       | 软件IIC的SDA、SCL等                         |
| **推挽输出**       | 驱动能力强，25mA（max），通用输出           |
| **开漏式复用功能** | 片上外设功能（硬件IIC 的SDA、SCL引脚等）    |
| **推挽式复用功能** | 片上外设功能（SPI 的SCK、MISO、MOSI引脚等） |

| **模式名称** | **性质** | **特征**                                           |
| ------------ | -------- | -------------------------------------------------- |
| 浮空输入     | 数字输入 | 可读取引脚电平，若引脚悬空，则电平不确定           |
| 上拉输入     | 数字输入 | 可读取引脚电平，内部连接上拉电阻，悬空时默认高电平 |
| 下拉输入     | 数字输入 | 可读取引脚电平，内部连接下拉电阻，悬空时默认低电平 |
| 模拟输入     | 模拟输入 | GPIO无效，引脚直接接入内部ADC                      |
| 开漏输出     | 数字输出 | 可输出引脚电平，高电平为高阻态，低电平接VSS        |
| 推挽输出     | 数字输出 | 可输出引脚电平，高电平接VDD，低电平接VSS           |
| 复用开漏输出 | 数字输出 | 由片上外设控制，高电平为高阻态，低电平接VSS        |
| 复用推挽输出 | 数字输出 | 由片上外设控制，高电平接VDD，低电平接VSS           |

## 输入浮空

![image-20240705105400030](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_54_0_202407051054099.png)

IO->触发器->读出

## 模拟输入

![image-20240705105444316](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_54_44_202407051054379.png)

模拟输入可以说是ADC模数转换器的专属配置

Io->模拟输入

## 输入上拉

上下拉可以看作是一个弹簧，输入电压的时候就会操纵弹簧使电压为1或0.

![image-20240705105543905](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_55_43_202407051055962.png)

## 输入下拉

![image-20240705105603416](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_56_3_202407051056484.png)

## 开漏输出

![image-20240705105621383](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_56_21_202407051056449.png)

## 开漏复用输出

![image-20240705105638431](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_56_38_202407051056485.png)

## 推挽输出

![image-20240705105650326](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_56_50_202407051056398.png)

## 推挽复用输出

![image-20240705105839733](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_58_39_202407051058807.png)

# 寄存器介绍

![image-20240705105938650](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_59_38_202407051059707.png)

# 配置步骤

![image-20240705105959628](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_10_59_59_202407051059679.png)

![image-20240705110056030](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_11_0_56_202407051100087.png)

1. 设置需要使用的GPIO外设时钟
2. 设置GPIO模式（八种模式）
3. 初始化GPIO口

```c
int main(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	while (1)
	{
		GPIO_ResetBits(GPIOA, GPIO_Pin_0);
		Delay_ms(500);
		GPIO_SetBits(GPIOA, GPIO_Pin_0);
		Delay_ms(500);
		
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_RESET);
		Delay_ms(500);
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
		Delay_ms(500);
		
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, (BitAction)0);
		Delay_ms(500);
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, (BitAction)1);
		Delay_ms(500);
	}
}

```

