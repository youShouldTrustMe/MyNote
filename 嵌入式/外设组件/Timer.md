# 定时器概述

![image-20240705172623623](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_26_23_202407051726672.png)

## 定时器分类

![image-20240705172654037](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_26_54_202407051726085.png)

| 定时器类型 | 主要功能                                                     |
| ---------- | ------------------------------------------------------------ |
| 基本定时器 | 没有输入输出通道，常用作时基，即定时功能                     |
| 通用定时器 | 具有多路独立通道，可用于输入捕获/输出比较，也可用作时基      |
| 高级定时器 | 除具备通用定时器所有功能外，还具备带死区控制的互补信号输出、刹车输入等功能（可用于电机控制、数字电源设计等） |

## 特性表

![image-20240705172805200](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_28_5_202407051728257.png)

# 滴答定时器

`SystemCoreClock / 1000` 表示配置 SysTick 定时器以 1ms 的间隔触发中断。具体来说，SysTick 定时器是一个递减计数器，当计数器递减到零时会触发中断，然后自动重新加载初始值继续计数。通过设置初始值为 `SystemCoreClock / 1000`，我们可以使 SysTick 定时器每经过 1ms 触发一次中断。

### 解释

1. **SystemCoreClock** 是系统核心时钟频率，单位是 Hz。它表示每秒钟内系统时钟的震荡次数。例如，如果 `SystemCoreClock` 是 84MHz（84,000,000 Hz），则表示每秒钟系统时钟震荡 84,000,000 次。

2. **SysTick_Config(SystemCoreClock / 1000)** 将 SysTick 定时器的重装载值（reload value）设置为 `SystemCoreClock / 1000`。重装载值是 SysTick 计数器从这个值开始递减的初始值。

3. **为什么是 1ms**：
    - 假设 `SystemCoreClock` 是 84MHz。
    - `SystemCoreClock / 1000` 等于 84,000,000 / 1000 = 84,000。
    - 这意味着 SysTick 计数器从 84,000 开始递减，每个时钟周期递减一次。
    - 因为系统时钟频率是 84MHz，即每秒钟有 84,000,000 个时钟周期，所以每 1,000 个时钟周期表示 1ms（1,000,000 / 1,000 = 1ms）。
    - 当 SysTick 计数器从 84,000 递减到 0 时，刚好经过了 1ms，然后触发中断，并重新加载为 84,000 继续递减。

### SysTick_Config 函数

`SysTick_Config` 是一个方便的函数，用于配置 SysTick 定时器并启用中断。其原型通常如下：

```c
uint32_t SysTick_Config(uint32_t ticks);
```

参数 `ticks` 是重装载值，即 SysTick 计数器的初始值。函数返回 0 表示配置成功，返回 1 表示配置失败。

### 示例

假设 `SystemCoreClock` 是 84MHz，下面的代码设置 SysTick 定时器每 1ms 触发一次中断：

```c
#include "stm32f4xx.h"

volatile uint32_t tick_count = 0;

void SysTick_Handler(void) {
    tick_count++;
    if (tick_count % 1000 == 0) {
        GPIO_ToggleBits(GPIOD, GPIO_Pin_12);  // 每秒切换一次 LED 状态
    }
}

void SysTick_Init(void) {
    // 配置 SysTick 为 1ms 中断
    if (SysTick_Config(SystemCoreClock / 1000)) {
        // 配置失败，进入无限循环
        while (1);
    }
}

int main(void) {
    SystemInit();  // 初始化系统时钟
    SysTick_Init();  // 初始化 SysTick

    RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOD, ENABLE);

    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_12;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_OUT;
    GPIO_InitStructure.GPIO_OType = GPIO_OType_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_100MHz;
    GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
    GPIO_Init(GPIOD, &GPIO_InitStructure);

    while (1) {
        // 主循环
    }
}
```

# 基本定时器

## 概述

![image-20240705172841947](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_28_42_202407051728000.png)

## 基本定时器框架

![image-20240705172907269](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_29_7_202407051729327.png)

## 影子寄存器

STM32微控制器中的"影子寄存器"通常指的是定时器（TIM）模块中的影子寄存器或ARR（自动重载寄存器）。这些影子寄存器在定时器的工作中起到重要作用，主要用于以下目的：

1. **实现无缝的定时周期更改**：定时器的自动重载寄存器（ARR）定义了计数器溢出后定时器将重新加载的值，从而决定了定时器的定时周期。通过在ARR寄存器中预先加载新的周期值，您可以实现在定时器运行时更改定时周期而无需停止定时器。这有助于实现无缝的定时控制。

2. **防止比较匹配中断的误触发**：当定时器的计数器与比较寄存器（例如CCR1、CCR2等）进行比较时，如果比较匹配，可能会触发中断或产生输出。使用影子寄存器，可以避免在计数器计数期间对比较寄存器进行更改导致误触发。影子寄存器会在计数器溢出时从ARR复制新的值，然后进行比较，从而确保比较匹配的稳定性。

3. **支持PWM生成**：在PWM生成模式下，ARR寄存器的值用于确定PWM的周期。通过定期更新ARR寄存器的值，您可以实现PWM信号的周期更改，从而控制PWM输出的频率。

4. **提供定时器的基准值**：ARR寄存器中的值通常用作定时器的基准值，用于计算各种时间延迟、周期和脉冲宽度。

我们不能直接访问影子寄存器，比如我们修改ARR寄存器中的值，那么修改之后，ARR寄存器中的值将会转移进入影子寄存器，而影子寄存器才是实际上起作用的，那么ARR寄存器实际上只是起到了一个缓冲作用

## 工作过程

### 时钟源

首先有一个内部时钟信号，这里的内部时钟信号就是来源于APB上的时钟

### 控制器

控制器将会产生TRGO主模式输出和复位、使能、计数信号

### 时基单元

通过控制器来到预分频器，通过预分频器产生一个CK_CNT的一个时钟，CNT计数器开始计时，当CNT计数器的值等于ARR的影子寄存器中的值，那么将会产生一个U（update）事件（该事件是默认产生的）或者UI（update interrupt）中断和DMA输出（我们可以配置其产生或者不产生），其中的U事件也可通过UC位可产生软件更新事件然而当有其中一个触发事件（U、UI）产生，之后将会将信号传输到NVIC中，预装载寄存器将会将它其中的值转移到影子寄存器中，这里可以设置ARPE位来管理ARR寄存器不需要有U事件发生就会将ARR中的值转移到影子寄存器中

![image-20240705173116546](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_31_16_202407051731604.png)

#### 递增模式

![image-20240705173214895](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_32_14_202407051732947.png)

#### 递减模式

![image-20240705173229128](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_32_29_202407051732176.png)

#### 中心对齐模式

![image-20240705173250423](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_32_50_202407051732489.png)

#### 定时器溢出时间

溢出的时间算出来是秒为单位

![image-20240705173317224](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_33_17_202407051733271.png)

## 主模式触发DAC功能

STM32定时器的一个特色就是主从模式触发，它能让内部硬件在不受程序的控制下实现自动运行

在触发器缠身TRGO信号后，会使用DAC输出一段波形，需要每隔一段时间来触发一次DAC，让他输出下一个电压点

## 相关寄存器

![image-20240705173407306](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_34_7_202407051734355.png)

![image-20240705173412639](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_34_12_202407051734779.png)

![image-20240705173503846](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_35_3_202407051735906.png)

![image-20240705173513769](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_35_13_202407051735819.png)

![image-20240705173522256](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_17_35_22_202407051735311.png)

![image-20240705173528538](C:\Users\ryf\AppData\Roaming\Typora\typora-user-images\image-20240705173528538.png)

# 通用定时器

## 概述

通用用定时器包含一个 16 位或 32 位自动重载计数器，该计数器由可编程预分频器驱动。它们可用于多种用途，包括测量输入信号的脉冲宽度（ 输入捕获 ）或生成输出波形（ 输出比较和 PWM）。使用定时器预分频器和 RCC 时钟控制器预分频器，可将脉冲宽度和波形周期从几微秒调制到几毫秒。这些定时器彼此完全独立，不共享任何资源

基本定时器只会在更新事件产生的时候才会产生中断或者DMA请求，而通用寄存器可以在多种情况下产生中断

![image-20240708110246135](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_2_46_202407081102202.png)

### 系统框图

![image-20240708110259998](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_3_0_202407081103070.png)

## 功能说明

### 时钟源

![image-20240708110357348](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_3_57_202407081103414.png)

#### *TRGI* 

主要用于触发输入使用，触发输入可以触发定时器的从模式

#### *ETR（External Trigger）*

我们可以配置ETR方波作为时钟源，外部时钟源可能会不太稳定及可能有毛刺，所以需要输入滤波器，ETR引脚主要用于外部触发定时器的启动、停止或复位操作。它允许外部信号来控制定时器的启动和停止，通常用于同步多个定时器或在特定事件触发时启动定时器。

控制器产生的TRGO信号会连接到别的定时器的ITR上去，实现级联功能

#### *计数器时钟源寄存器设置方法*

| 计数器时钟选择类型               | 设置方法                                                   |
| -------------------------------- | ---------------------------------------------------------- |
| 内部时钟(CK_INT)                 | 设置TIMx_SMCR的SMS=000                                     |
| 外部时钟模式1：外部输入引脚(TIx) | 设置TIMx_SMCR的SMS=111                                     |
| 外部时钟模式2：外部触发输入(ETR) | 设置TIMx_SMCR的ECE=1                                       |
| 内部触发输入(ITRx)               | 设置可参考STM32F10xxx参考手册_V10（中文版）.pdf，14.3.15节 |

#### 外部时钟模式

##### 外部时钟模式1

TL1_ED是双边沿的，TL1FP1、TL2FP2来源于两个不同的通道

从通道进来的信号可能不是稳定的信号，使用输入捕获滤波对信号进行处理变成稳定的高低电平

![image-20240708110717818](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_7_17_202407081107875.png)

这里的信号来源于输入通道1和输入通道2的TL1FP1和TL2FP2，通道1是双边沿检测，也就是上升沿和下降沿都检测

![image-20240708110742806](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_7_42_202407081107869.png)

##### 外部时钟模式2

![image-20240708110808409](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_8_8_202407081108472.png)

#### 内部触发输入（TRGO）

使用一个定时器产生一个TRGO的信号，这个信号可以连接到ITRx，作为下一个定时器的外部时钟模式2的时钟输入源

![image-20240708110855410](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_8_55_202407081108466.png)

#### 时基单元

可编程定时器的主要模块由一个 16 位/32 位计数器及其相关的自动重装寄存器组成。此计数器可采用递增方式计数。计数器的时钟可通过预分频器进行分频。计数器、自动重载寄存器和预分频器寄存器可通过软件进行读写。即使在计数器运行时也可执行读写操作。

时基单元包括：

● 计数器寄存器 (TIMx_CNT)

● 预分频器寄存器 (TIMx_PSC)

● 自动重载寄存器 (TIMx_ARR)

自动重载寄存器是预装载的。对自动重载寄存器执行写入或读取操作时会访问预装载寄存器。预装载寄存器的内容既可以直接传送到影子寄存器，也可以在每次发生更新事件 (UEV)时传送到影子寄存器，这取决于 TIMx_CR1 寄存器中的自动重载预装载使能位 (ARPE)。当计数器达到上溢值（或者在递减计数时达到下溢值）并且 TIMx_CR1 寄存器中的 UDIS 位为0 时，将发送更新事件。该更新事件也可由软件产生。下文将针对各配置的更新事件的产生进行详细介绍。

计数器由预分频器输出 CK_CNT 提供时钟，仅当 TIMx_CR1 寄存器中的计数器启动位 (CEN)置 1 时，才会启动计数器（有关计数器使能的更多详细信息，另请参见从模式控制器的相关说明）。

请注意，真正的计数器使能信号 CNT_EN 在 CEN 置 1 的一个时钟周期后被置 1。

#### 预分频器

预分频器可对计数器时钟频率进行分频，分频系数介于 1 到 65536 之间。该预分频器基于16 位/32 位寄存器（TIMx_PSC 寄存器）所控 制的 16 位计数器。由于该控制寄存器具有缓冲功能，因此预分频器可实现实时更改。而新的预分频比将在下一更新事件发生时被采用。

### 输入捕获

![image-20240708111013274](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_10_13_202407081110342.png)

- 在输入端有个xor门，如果数据选择器选择上方来源于xor门的信号时，那么输入捕获通道1的输入就是3个引脚的异或值，如果选择另外一个引脚的信号，四个输入通道就各用各的

- 实际上这个xor门就是为三相无刷电机服务，无刷电机有3个霍尔传感器检测转子位置，可以根据转子的位置进行换相

- 设计了两套滤波核边沿检测电路，经过滤波和极性选择，分别得到TI1FP1（TI1     Filter Polarity 1）和TI2FP2的信号，输入给通道（ICx）的后续电路。由图中可以看到，tim1和tim2的FP信号会交叉，交叉的目的：

  - 可以灵活的切换后续捕获电路的输入（切换输入通道1和输入通道2）

  - 可以将一个引脚的输入，同时映射到两个捕获单元

- TRC信号来源于外部时钟模式1产生的信号，这样设计也是为了无刷电机的驱动

![image-20240708111047846](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_10_47_202407081110907.png)

![image-20240708111055097](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_10_55_202407081110186.png)

当信号出现一个上升沿，将会把CNT的值转运到CCR1里面去，之后将CNT清零（从模式{只有从模式有自动清零}自动执行），由于CNT已清零，那么这个时候也就是从第一个上升沿开始从零计时，直到下一次上升沿来临。所以CCR1中每次都会保存最新的一个周期内的计数值（测周法）

![image-20240708111116741](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_11_16_202407081111810.png)

![image-20240708111121824](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_11_21_202407081111900.png)

#### 频率测量

测频法原理：

- 我们可以将闸门时间设置为1s，在这个时间段内，计上升沿的次数，就可以得到频率hz，实际上频率的概念就是在1s内的震荡次数，一般闸门时间越长，越精确，结果更新慢，数值相对稳定，适用于高频

测周法原理：

- 周期的倒数就是频率，只需要测出一个周期的时间，取个倒数就是频率了，实际上我们测时间也是使用定时器计次，我们使用一个标准的频率fc计次时钟来驱动计数器，从一个上升沿开始计时，一直等到下一个上升沿停止，计一个数的时间时1/fc，计n个数就是n/fc.需要信号频率低一点，数据更新快，数据跳变也非常快，结果值有噪声影响，测出来的值不会太稳定，适用于低频

图中左侧为测频法，右侧为测周法

![image-20240708111218894](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_12_18_202407081112967.png)

#### PWMI基本结构

使用两个通道测试一个io引脚，可以同时测量两个周期和占空比

下图中CCR1捕获高电平，CCR2捕获低电平，但是这里CCR2捕获时不会自动清零

从触发CCR1的时候CNT清零，触发CCR2的时候，CCR2中的值就是一个周期内的高电平的时间，所以可以测量PWM波

![image-20240708111250400](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_12_50_202407081112470.png)

### 输出比较（OC）

- OC（Output     Compare）输出比较
- 输出比较可以通过比较CNT与CCR寄存器值（下图中的捕获比较寄存器）的关系，来对输出电平进行置1、置0或翻转的操作，用于输出一定频率和占空比的PWM波形
- 每个高级定时器和通用定时器都拥有4个输出比较通道
- 高级定时器的前3个通道额外拥有死区生成和互补输出的功能

![image-20240708111319760](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_13_19_202407081113816.png)

阴影部分是输入部分，输入值将会和影子寄存器中的值进行比较，比较出来的值将会去到输出控制中去，当产生一个compare_transfer信号时，预装载寄存器将会将值转移到影子寄存器中去

CC1s：捕获、比较1选择

![image-20240708111345044](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_13_45_202407081113110.png)

将cnt和CCR比较，调节输出比较模式，比较完之后将会输出参考信号（0或1），这里的主模式就是TRGO

![image-20240708111400654](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_14_0_202407081114716.png)

| **模式**         | **描述**                                                     |
| ---------------- | ------------------------------------------------------------ |
| 冻结             | CNT=CCR时，REF保持为原状态                                   |
| 匹配时置有效电平 | CNT=CCR时，REF置有效电平                                     |
| 匹配时置无效电平 | CNT=CCR时，REF置无效电平                                     |
| 匹配时电平翻转   | CNT=CCR时，REF电平翻转                                       |
| 强制为无效电平   | CNT与CCR无效，REF强制为无效电平                              |
| 强制为有效电平   | CNT与CCR无效，REF强制为有效电平                              |
| PWM模式1         | 向上计数：CNT<CCR时，REF置有效电平，CNT≥CCR时，REF置无效电平  向下计数：CNT>CCR时，REF置无效电平，CNT≤CCR时，REF置有效电平 |
| PWM模式2         | 向上计数：CNT<CCR时，REF置无效电平，CNT≥CCR时，REF置有效电平  向下计数：CNT>CCR时，REF置有效电平，CNT≤CCR时，REF置无效电平 |

#### 主从模式触发

- 主模式可以将定时器内部的信号，映射到TRGO引脚上，用于触发别的外设，所以交主模式
- 从模式就是接收其他外设或者自身外设的一些信号，用于控制自身定时器的运行，也就是被别的信号控制

当有触发源来（ITR0….）时，选择指定的一个信号，得到TRGI，TRGI触发从模式，从模式在后面的表格中选一项操作自动执行

![image-20240708111450310](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_14_50_202407081114365.png)

# TIM主要函数

```c
//恢复缺省配置
void TIM_DeInit(TIM_TypeDef* TIMx);
//时基单元初始化，参数1：选择某个定时器，参数2：配置时基单元的参数
void TIM_TimeBaseInit(TIM_TypeDef* TIMx, TIM_TimeBaseInitTypeDef* TIM_TimeBaseInitStruct);
//配置输出比较通道1
void TIM_OC1Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);
void TIM_OC2Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);
void TIM_OC3Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);
void TIM_OC4Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);

void TIM_ICInit(TIM_TypeDef* TIMx, TIM_ICInitTypeDef* TIM_ICInitStruct);
void TIM_PWMIConfig(TIM_TypeDef* TIMx, TIM_ICInitTypeDef* TIM_ICInitStruct);
void TIM_BDTRConfig(TIM_TypeDef* TIMx, TIM_BDTRInitTypeDef *TIM_BDTRInitStruct);
//给时基单元设置一个默认值
void TIM_TimeBaseStructInit(TIM_TimeBaseInitTypeDef* TIM_TimeBaseInitStruct);
void TIM_OCStructInit(TIM_OCInitTypeDef* TIM_OCInitStruct);
void TIM_ICStructInit(TIM_ICInitTypeDef* TIM_ICInitStruct);
void TIM_BDTRStructInit(TIM_BDTRInitTypeDef* TIM_BDTRInitStruct);
//使能计数器，参数1：选择定时器，参数2：计数器状态，ENABLEN、DISABLEN
void TIM_Cmd(TIM_TypeDef* TIMx, FunctionalState NewState);
void TIM_CtrlPWMOutputs(TIM_TypeDef* TIMx, FunctionalState NewState);
//中断输出控制，参数1：选择定时器，参数2：配置哪个中断输出，参数3：中断输出状态
void TIM_ITConfig(TIM_TypeDef* TIMx, uint16_t TIM_IT, FunctionalState NewState);
void TIM_GenerateEvent(TIM_TypeDef* TIMx, uint16_t TIM_EventSource);
void TIM_DMAConfig(TIM_TypeDef* TIMx, uint16_t TIM_DMABase, uint16_t TIM_DMABurstLength);
void TIM_DMACmd(TIM_TypeDef* TIMx, uint16_t TIM_DMASource, FunctionalState NewState);
//选择内部时钟
void TIM_InternalClockConfig(TIM_TypeDef* TIMx);
//选择ITRx其他定时器的时钟
void TIM_ITRxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_InputTriggerSource);
//选择TLx捕获通道的时钟
void TIM_TIxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_TIxExternalCLKSource,
uint16_t TIM_ICPolarity, uint16_t ICFilter);
//选择ETR通过外部时钟模式1输入的时钟
void TIM_ETRClockMode1Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity,
uint16_t ExtTRGFilter);
//选择ETR通过外部时钟模式2输入的时钟
void TIM_ETRClockMode2Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, 
uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter);
//单独用于配置ETR函数，如时钟源极性
void TIM_ETRConfig(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity,
uint16_t ExtTRGFilter);
//用于写预分频值
void TIM_PrescalerConfig(TIM_TypeDef* TIMx, uint16_t Prescaler, uint16_t TIM_PSCReloadMode);
//选择新的计数器模式
void TIM_CounterModeConfig(TIM_TypeDef* TIMx, uint16_t TIM_CounterMode);
void TIM_SelectInputTrigger(TIM_TypeDef* TIMx, uint16_t TIM_InputTriggerSource);
void TIM_EncoderInterfaceConfig(TIM_TypeDef* TIMx, uint16_t TIM_EncoderMode,
uint16_t TIM_IC1Polarity, uint16_t TIM_IC2Polarity);
//配置强制输出模式，如果在运行中想要暂停输出并且强制输出高或者低电平
void TIM_ForcedOC1Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);
void TIM_ForcedOC2Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);
void TIM_ForcedOC3Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);
void TIM_ForcedOC4Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);
//自动重装器预装功能配置
void TIM_ARRPreloadConfig(TIM_TypeDef* TIMx, FunctionalState NewState);
void TIM_SelectCOM(TIM_TypeDef* TIMx, FunctionalState NewState);
void TIM_SelectCCDMA(TIM_TypeDef* TIMx, FunctionalState NewState);
void TIM_CCPreloadControl(TIM_TypeDef* TIMx, FunctionalState NewState);
//输出比较的预装功能，往影子寄存器中写入数据
void TIM_OC1PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);
void TIM_OC2PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);
void TIM_OC3PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);
void TIM_OC4PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);
//用来配置快速使能
void TIM_OC1FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);
void TIM_OC2FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);
void TIM_OC3FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);
void TIM_OC4FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);
//外部事件时清除REF信号
void TIM_ClearOC1Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);
void TIM_ClearOC2Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);
void TIM_ClearOC3Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);
void TIM_ClearOC4Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);
//用来设置输出比较的极性，带有N的就是高级定时器里互补通道的配置
void TIM_OC1PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);
void TIM_OC1NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);
void TIM_OC2PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);
void TIM_OC2NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);
void TIM_OC3PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);
void TIM_OC3NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);
void TIM_OC4PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);
//用来单独修改输出使能参数
void TIM_CCxCmd(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_CCx);
void TIM_CCxNCmd(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_CCxN);
//选择输出比较模式，用来单独更改输出比较模式的函数
void TIM_SelectOCxM(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_OCMode);
void TIM_UpdateDisableConfig(TIM_TypeDef* TIMx, FunctionalState NewState);
void TIM_UpdateRequestConfig(TIM_TypeDef* TIMx, uint16_t TIM_UpdateSource);
void TIM_SelectHallSensor(TIM_TypeDef* TIMx, FunctionalState NewState);
void TIM_SelectOnePulseMode(TIM_TypeDef* TIMx, uint16_t TIM_OPMode);
void TIM_SelectOutputTrigger(TIM_TypeDef* TIMx, uint16_t TIM_TRGOSource);
void TIM_SelectSlaveMode(TIM_TypeDef* TIMx, uint16_t TIM_SlaveMode);
void TIM_SelectMasterSlaveMode(TIM_TypeDef* TIMx, uint16_t TIM_MasterSlaveMode);
//给计数器写入一个值，可以手动给一个计数值
void TIM_SetCounter(TIM_TypeDef* TIMx, uint16_t Counter);
//给自动重装器写入一个值，可以手动给自动重装值
void TIM_SetAutoreload(TIM_TypeDef* TIMx, uint16_t Autoreload);
//单独更改CCR寄存器值的函数，更改占空比就使用这个函数
void TIM_SetCompare1(TIM_TypeDef* TIMx, uint16_t Compare1);
void TIM_SetCompare2(TIM_TypeDef* TIMx, uint16_t Compare2);
void TIM_SetCompare3(TIM_TypeDef* TIMx, uint16_t Compare3);
void TIM_SetCompare4(TIM_TypeDef* TIMx, uint16_t Compare4);
void TIM_SetIC1Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);
void TIM_SetIC2Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);
void TIM_SetIC3Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);
void TIM_SetIC4Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);
void TIM_SetClockDivision(TIM_TypeDef* TIMx, uint16_t TIM_CKD);
uint16_t TIM_GetCapture1(TIM_TypeDef* TIMx);
uint16_t TIM_GetCapture2(TIM_TypeDef* TIMx);
uint16_t TIM_GetCapture3(TIM_TypeDef* TIMx);
uint16_t TIM_GetCapture4(TIM_TypeDef* TIMx);
//获取当前计数器的计数值
uint16_t TIM_GetCounter(TIM_TypeDef* TIMx);
//获取当前预分频器值
uint16_t TIM_GetPrescaler(TIM_TypeDef* TIMx);
//获取标志位
FlagStatus TIM_GetFlagStatus(TIM_TypeDef* TIMx, uint16_t TIM_FLAG);
//清除标志位
void TIM_ClearFlag(TIM_TypeDef* TIMx, uint16_t TIM_FLAG);
//获取中断状态
ITStatus TIM_GetITStatus(TIM_TypeDef* TIMx, uint16_t TIM_IT);
void TIM_ClearITPendingBit(TIM_TypeDef* TIMx, uint16_t TIM_IT);
```

# 实验

## 基本框架

![image-20240708111921302](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_19_21_202407081119372.png)

## 定时中断的主要代码

在每次初始化的时候，都会进入一次中断函数，是因为在TIM_TimeBaseInit函数中会强制生成一个U或者UI，解决方案就是在开始计时器和NVIC之间将标志位清空

```c
void Timer_Init(void){
	
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	NVIC_InitTypeDef NVIC_InitStruct;
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
	
	//默认使用内部时钟
	TIM_InternalClockConfig(TIM2);
	
	//时基单元的配置
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 10000 - 1;        //决定定时时间，一共有多少个数，以TIM_Prescaler的频率计TIM_Period个数
	TIM_TimeBaseInitStruct.TIM_Prescaler = 7200 - 1;        //决定定时时间，计数频率
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM2,&TIM_TimeBaseInitStruct);
	
	//避免初始化完就进入中断
	TIM_ClearFlag(TIM2,TIM_FLAG_Update);
	//开启中断
	TIM_ITConfig(TIM2,TIM_IT_Update,ENABLE);
	
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_1);
	
	NVIC_InitStruct.NVIC_IRQChannel = TIM2_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 2;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 1;
	NVIC_Init(&NVIC_InitStruct);
	
	//开启定时器
	TIM_Cmd(TIM2,ENABLE);
}

void TIM2_IRQHandler(void){
	if(TIM_GetFlagStatus(TIM2,TIM_IT_Update) == SET){
		TIM_ClearFlag(TIM2,TIM_IT_Update);
	}
}
	
```

## 外部时钟ETR主要代码

```c

void extern_Timer_Init(void)
{

	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	NVIC_InitTypeDef NVIC_InitStruct;
	GPIO_InitTypeDef GPIO_InitStructure;
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	//使用外部时钟2模式
	TIM_ETRClockMode2Config(TIM2,TIM_ExtTRGPSC_OFF,TIM_ExtTRGPolarity_NonInverted,0x0f);
	
	//配置GPIO
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = TIM_CKD_DIV1;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	//时基单元的配置
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 10 - 1;        //决定定时时间
	TIM_TimeBaseInitStruct.TIM_Prescaler = 1 - 1;        //决定定时时间
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM2,&TIM_TimeBaseInitStruct);
	
	TIM_ClearFlag(TIM2,TIM_FLAG_Update);
	//开启中断
	TIM_ITConfig(TIM2,TIM_IT_Update,ENABLE);
	
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_1);
	
	NVIC_InitStruct.NVIC_IRQChannel = TIM2_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 2;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 1;
	NVIC_Init(&NVIC_InitStruct);
	
	//开启定时器
	TIM_Cmd(TIM2,ENABLE);
}

	

```

## PWM输出实验

### 原理

将计数器和捕获/比较寄存器中的值进行比较，当计数器值小于CCR时，为低电平，否则为高电平

![image-20240708112136006](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_21_36_202407081121104.png)

![image-20240708112149572](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_21_49_202407081121673.png)

### 参数的计算

![image-20240708112206009](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_22_6_202407081122074.png)

### 配置

一共两种pwm模式，通过配置CCxP配置有效电平为多少

![image-20240708112229894](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_22_29_202407081122988.png)

### 呼吸灯

```c

void PWM_Init(void)
{
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	GPIO_InitTypeDef GPIO_InitStructure;
	TIM_OCInitTypeDef TIM_OCInitStruct;
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;        //对于，普通的开漏/推挽输出，引脚的控制权来自于输出数据寄存器，
	//如果想要让定时器来控制引脚，就需要使用复用开漏/推挽输出模式，可看开漏\推挽输出图
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	//时基单元的配置
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 100 - 1;        //ARR
	TIM_TimeBaseInitStruct.TIM_Prescaler = 720 - 1;        //PSC
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM2,&TIM_TimeBaseInitStruct);
	
	TIM_ClearFlag(TIM2,TIM_FLAG_Update);
	
	//初始化输出比较单元
	TIM_OCStructInit(&TIM_OCInitStruct);        //给结构体一个默认的值
	TIM_OCInitStruct.TIM_OCMode = TIM_OCMode_PWM1;
	TIM_OCInitStruct.TIM_OCPolarity = TIM_OCPolarity_High;
	TIM_OCInitStruct.TIM_OutputState = TIM_OutputState_Enable;
	TIM_OCInitStruct.TIM_Pulse = 0;        //CCR
	TIM_OC1Init(TIM2,&TIM_OCInitStruct);        //这里的OC1和手册上的引脚是对应的，这里配置的就是TIM的CH1通道，所以需要在手册上找到TIM_CH1通道的引脚，不能使用别的引脚
	
	//开启定时器
	TIM_Cmd(TIM2,ENABLE);
}

void PWM_SetCompare1(u16 compare)
{
	TIM_SetCompare1(TIM2,compare);
}

//通过不断改变ARR的值来改变呼吸灯的频率
int main(void){
	u8 i;;
	PWM_Init();
	while(1){
		for(i = 0;i <= 100; i++){
			PWM_SetCompare1(i);
			Delay_ms(10);
		}
		for(i = 0;i <= 100; i++){
			PWM_SetCompare1(100 - i);
			Delay_ms(10);
		}
	}
}


```

### 舵机实验

#### 原理

如果是两条白线一条黑线，那么黑线接地，白线中间的是正极

![image-20240708112343474](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_23_43_202407081123585.png)

当计数器记到500时，也就是0.5ms，舵机输出轴转角就是0度，当计数器记到2500时，舵机的输出轴转角就是180度

#### 硬件电路

![image-20240708112412586](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_24_12_202407081124675.png)

#### 代码

```c

void servos_PWM_Init(void)
{
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	GPIO_InitTypeDef GPIO_InitStructure;
	TIM_OCInitTypeDef TIM_OCInitStruct;
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;        //对于，普通的开漏/推挽输出，引脚的控制权来自于输出数据寄存器，如果想要让定时器来控制引脚，就需要使用复用开漏/推挽输出模式，可看开漏\推挽输出图
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	TIM_InternalClockConfig(TIM2);
	
	//时基单元的配置
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 20000 - 1;        //ARR
	TIM_TimeBaseInitStruct.TIM_Prescaler = 72 - 1;        //PSC
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM2,&TIM_TimeBaseInitStruct);
	
	TIM_ClearFlag(TIM2,TIM_FLAG_Update);
	
	//初始化输出比较单元
	TIM_OCStructInit(&TIM_OCInitStruct);        //给结构体一个默认的值
	TIM_OCInitStruct.TIM_OCMode = TIM_OCMode_PWM1;
	TIM_OCInitStruct.TIM_OCPolarity = TIM_OCPolarity_High;
	TIM_OCInitStruct.TIM_OutputState = TIM_OutputState_Enable;
	TIM_OCInitStruct.TIM_Pulse = 0;        //CCR
	TIM_OC1Init(TIM2,&TIM_OCInitStruct);        //这里的OC1和手册上的引脚是对应的，这里配置的就是TIM的CH1通道，所以需要在手册上找到TIM_CH1通道的引脚，不能使用别的引脚
	
	//开启定时器
	TIM_Cmd(TIM2,ENABLE);
}

void PWM_SetCompare1(u16 compare)
{
	TIM_SetCompare1(TIM2,compare);
}

int main(void){
	
	servos_PWM_Init();

	servou_start();
	while(1){
	}
}


```

### 直流电机实验

![image-20240708112501732](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_25_1_202407081125830.png)

TB6612可以接两个电机

![image-20240708112516614](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_25_16_202407081125730.png)

## 输入捕获测频率

### 测码率

```c

//PWMI模式
void PWMI_init(void)
{
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	//默认使用内部时钟
	TIM_InternalClockConfig(TIM3);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;        
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	//时基单元的配置
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 65536 - 1;        //最好设置大一点，防止计数溢出
	TIM_TimeBaseInitStruct.TIM_Prescaler = 72 - 1;        //决定频率
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3,&TIM_TimeBaseInitStruct);
	
	TIM_ICInitTypeDef TIM_ICInitStruct;
	TIM_ICInitStruct.TIM_Channel = TIM_Channel_1;
	TIM_ICInitStruct.TIM_ICFilter = 0x0f;
	TIM_ICInitStruct.TIM_ICPolarity = TIM_ICPolarity_Rising;
	TIM_ICInitStruct.TIM_ICPrescaler = TIM_ICPSC_DIV1;
	TIM_ICInitStruct.TIM_ICSelection = TIM_ICSelection_DirectTI;        //用于配置数据选择器，可以选择直连通道，也可以选择交叉通道
	TIM_PWMIConfig(TIM3,&TIM_ICInitStruct);

	//配置时钟源
	TIM_SelectInputTrigger(TIM3,TIM_TS_TI1FP1);
	
	//配置从模式为reset
	TIM_SelectSlaveMode(TIM3,TIM_SlaveMode_Reset);
	
		//开启定时器
	TIM_Cmd(TIM3,ENABLE);
}

//得到占空比
u32 IC_getDuty(void)
{
	u32 a = 0;
	a = TIM_GetCapture2(TIM3) * 100 / TIM_GetCapture1(TIM3);
	return TIM_GetCapture2(TIM3) * 100 / TIM_GetCapture1(TIM3);
}


void IC_init(void)
{
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	//默认使用内部时钟
	TIM_InternalClockConfig(TIM3);
	
	//配置相关GPIO
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;        
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	//时基单元的配置
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 65536 - 1;        //最好设置大一点，防止计数溢出
	TIM_TimeBaseInitStruct.TIM_Prescaler = 72 - 1;        //决定频率
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3,&TIM_TimeBaseInitStruct);
	
	//输入捕获配置
	TIM_ICInitTypeDef TIM_ICInitStruct;
	TIM_ICInitStruct.TIM_Channel = TIM_Channel_1;
	TIM_ICInitStruct.TIM_ICFilter = 0x0f;
	TIM_ICInitStruct.TIM_ICPolarity = TIM_ICPolarity_Rising;
	TIM_ICInitStruct.TIM_ICPrescaler = TIM_ICPSC_DIV1;
	TIM_ICInitStruct.TIM_ICSelection = TIM_ICSelection_DirectTI;        //用于配置数据选择器，可以选择直连通道，也可以选择交叉通道
	TIM_ICInit(TIM3,&TIM_ICInitStruct);
	
	//配置时钟源
	TIM_SelectInputTrigger(TIM3,TIM_TS_TI1FP1);
	
	//配置从模式为reset
	TIM_SelectSlaveMode(TIM3,TIM_SlaveMode_Reset);
	
	//开启定时器
	TIM_Cmd(TIM3,ENABLE);
}

u32 IC_getFreq(void)
{
	//频率由APB2时钟除以预分频系数得到，这里的APB2频率为72MHz，预分频为72，那么得到的就是1Mhz,使用计数器的频率初一CCR中的值就可以得到待测的频率
	return 1000000 / TIM_GetCapture1(TIM3);
}

int main(void){
	
	char str[50];
	
	PWMI_init();
	PWM_Init();
	serial_init();
	
	PWM_setPrescaler(720-1);
	PWM_SetCompare1(50);
	
	while(1){
		sprintf(str,"the freq is %u Hz,the duty is %u \%\n",IC_getFreq(),IC_getDuty());  //使用串口进行测试结果的展示
		send_string(str);
	}
}

	

```

### 编码器测速

![image-20240708112627885](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_26_27_202407081126954.png)

当一个定时器配置了编码器的接口，那么这个定时器基本上干不了其他的活了，且注意，只有通道1和通道2可以接编码器，其他通道不可以接编码器

正交编码器

输出的两个方波信号，相位差90度，超前90度或者滞后90度，分别表示正转和反转

正交信号精度更高，移位AB相都可以计次，相当于计次频率都提高了一倍

正交信号可以抗噪声，如果一个信号不变，另外一个信号连续跳变，也就是产生了噪声，那么这个时候的计次值是不会变化的

![image-20240708112728770](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_27_28_202407081127850.png)

![image-20240708112735044](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_27_35_202407081127128.png)

#### 工作模式

![image-20240708112757078](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_27_57_202407081127155.png)

#### 均不反向

![image-20240708112818233](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_28_18_202407081128308.png)

#### 反向

![image-20240708112836785](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_28_36_202407081128878.png)

## 引脚重映射

### 映射关系

![image-20240708112922248](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_11_29_22_202407081129337.png)

### 代码

```c

void AFIO_PWM_Init(void)
{
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	GPIO_InitTypeDef GPIO_InitStructure;
	TIM_OCInitTypeDef TIM_OCInitStruct;
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO,ENABLE);        //如果想要重映射就需要使用AFIO，注意，重映射的时候要看看引脚是否可以重映射
	
	//将TIM2_CH1的引脚重映射到PA15，这里的重映射是系统自动执行的
	GPIO_PinRemapConfig(GPIO_PartialRemap1_TIM2,ENABLE);
	
	//解除PA15端口使用JTAG调试的作用
	GPIO_PinRemapConfig(GPIO_Remap_SWJ_JTAGDisable,ENABLE);
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;        //对于，普通的开漏/推挽输出，引脚的控制权来自于输出数据寄存器，如果想要让定时器来控制引脚，就需要使用复用开漏/推挽输出模式，可看开漏\推挽输出图
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	//时基单元的配置
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 100 - 1;        //ARR
	TIM_TimeBaseInitStruct.TIM_Prescaler = 720 - 1;        //PSC
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM2,&TIM_TimeBaseInitStruct);
	
	TIM_ClearFlag(TIM2,TIM_FLAG_Update);
	
	//初始化输出比较单元
	TIM_OCStructInit(&TIM_OCInitStruct);        //给结构体一个默认的值
	TIM_OCInitStruct.TIM_OCMode = TIM_OCMode_PWM1;
	TIM_OCInitStruct.TIM_OCPolarity = TIM_OCPolarity_High;
	TIM_OCInitStruct.TIM_OutputState = TIM_OutputState_Enable;
	TIM_OCInitStruct.TIM_Pulse = 0;        //CCR
	TIM_OC1Init(TIM2,&TIM_OCInitStruct);        //这里的OC1和手册上的引脚是对应的，这里配置的就是TIM的CH1通道，所以需要在手册上找到TIM_CH1通道的引脚，不能使用别的引脚
	
	//开启定时器
	TIM_Cmd(TIM2,ENABLE);
}



```

