# 参考链接

[SPI协议详解（图文并茂+超详细） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/290620901)

# 串口

因为UART没有时钟信号，无法控制何时发送数据，也无法保证双方按照完全相同的速度接收数据。因此，双方以不同的速度进行数据接收和发送，就会出现问题。

如果要解决这个问题，UART为每个字节添加额外的起始位和停止位，以帮助接收器在数据到达时进行同步；

双方还必须事先就传输速度达成共识（设置相同的波特率，例如每秒9600位）。

传输速率如果有微小差异不是问题，因为接收器会在每个字节的开头重新同步。相应的协议如下图所示：

![串口传输](https://pic4.zhimg.com/80/v2-39dd6c1224d0e3fa0144e90519f4745d_720w.webp)

> [!NOTE]
>
> 图中的11001010不等于0x53。这是因为串口协议通常会首先发送最低有效位，因此最小位在最左边LSB。低四位字节实际上是0011 = 0x3，高四位字节是0101 = 0x5。
>
> 串口先发送低位，后发送高位

异步串行工作得很好，但是在每个字节发送的时候都需要额外的起始位和停止位以及在发送和接收数据所需的复杂硬件方面都有很多开销。

不难发现，如果接收端和发送端设置的速度都不一致，那么接收到的数据将是垃圾（乱码）。 

因为UART没有时钟信号，无法控制何时发送数据，也无法保证双方按照完全相同的速度接收数据。因此，双方以不同的速度进行数据接收和发送，就会出现问题。

于是我们想有没有更好一点的串行通讯方式；相比较于UART，SPI的工作方式略有不同。

1. SPI是一个同步的数据总线，也就是说它是用单独的数据线和一个单独的时钟信号来保证发送端和接收端的完美同步。
2. 时钟是一个振荡信号，它告诉接收端在确切的时机对数据线上的信号进行采样。
3. 产生时钟的一侧称为主机，另一侧称为从机。总是只有一个主机（一般来说可以是微控制器/MCU），但是可以有多个从机（后面详细介绍）；
4. 数据的采集时机可能是时钟信号的上升沿（从低到高）或下降沿（从高到低）。

# 介绍

SPI:串行外设设备接口(Serial Peripheral Interface),是一种高速的、全双工、同步的通信总线。

| 功能说明 | SPI总线             | IIC总线                |
| -------- | ------------------- | ---------------------- |
| 通信方式 | 同步串行全双工      | 同步串行半双工         |
| 总线接口 | MOSI、MISO、SCL、CS | SDA、SCL               |
| 拓扑结构 | 一主多从/一主一从   | 多主从                 |
| 从机选择 | 片选引脚选择        | SDA上设备地址片选      |
| 通信速率 | 般50MHz以下         | 100kHz、400kHz、3.4MHz |
| 数据格式 | 8位/16位            | 8位                    |
| 传输顺序 | MSB/LSB             | MSB                    |

SP接口主要应用在存储芯片、AD转换器以及LCD中。

![SPI组成结构](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_13_46_56_202409141346019.png)

> [!IMPORTANT]
>
> 输出引脚配置为推挽输出，输入引脚配置为浮空或者上拉输入，推挽输出高低电平都有着很强的驱动能力，和i2c不同，i2c电平下降快，上升缓慢，而SPI电平不仅上升快，下降也快
>
> 主机一个输入引脚，这个引脚将会接收来自多个从机传输的数据，这势必会产生冲突，所以SPI规定，当从机未被选中时，MISO引脚必须切换为高阻态（相当于引脚断开，不输出任何电平）

## 原理

基本收发电路就是一个==移位模型==

SPI主机中有一个8位移位寄存器，SPI从机中也有一个8位的移位寄存器，移位寄存器有一个时钟输入端，SPI一般都是==高位==先行的，每来一个时钟，移位寄存器都会向左进行移位，从机中的移位寄存器也是同理。移位寄存器的时钟源由主机提供（波特率发生器），它产生的时钟驱动主机的移位寄存器进行移位，同时，这个时钟也通过SCK引脚进行输出给到从机的移位寄存器中，主机移位寄存器左边移出去的数据，通过MOSI输入到从机移位寄存器的右边，从机移位寄存器左边移出去的数据，通过MISO输入到主机移位寄存器的右边

如果是主机只是发送（只是接收），那么还是和上面同时收发一样，只是从机会随便发送一串数据（一般会给0xff或0x00），只要将数据置换出来即可，主机不会关注接收到的数据

![SPI移位的工作原理](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_13_52_58_202409141352788.png)

![SPI数据传输的具体过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_13_56_7_202409141356654.png)

# 特性

## 框架图

![SPI框架图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_13_58_20_202409141358370.png)

![SPI简图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_13_58_34_202409141358800.png)

### 信号线

- MISO：Master input     slave output 主机输入，从机输出（数据来自从机）；
- MOSI：Master     output slave input 主机输出，从机输入（数据来自主机）；
- SCLK ：Serial Clock 串行时钟信号，由主机产生发送给从机；
- SS：Slave Select 片选信号，由主机发送，以控制与哪个从机通信，通常是低电平有效信号。

### 时钟极性

除了配置串行时钟速率（频率）外，SPI主设备还需要配置时钟极性。

根据硬件制造商的命名规则不同，时钟极性通常写为CKP或CPOL。时钟极性和相位共同决定读取数据的方式，比如信号上升沿读取数据还是信号下降沿读取数据；

CKP可以配置为1或0。这意味着您可以根据需要将时钟的默认状态（IDLE）设置为高或低。极性反转可以通过简单的逻辑逆变器实现。您必须参考设备的数据手册才能正确设置CKP和CKE。

- CKP = 0：时钟空闲IDLE为低电平 0；
- CKP = 1：时钟空闲IDLE为高电平1；

### 时钟相位

除配置串行时钟速率和极性外，SPI主设备还应配置时钟相位（或边沿）。根据硬件制造商的不同，时钟相位通常写为CKE或CPHA；

顾名思义，时钟相位/边沿，也就是采集数据时是在时钟信号的具体相位或者边沿；

- CKE = 0：在时钟信号SCK的第一个跳变沿采样；
- CKE = 1：在时钟信号SCK的第二个跳变沿采样；

如何确定数据的有效性：

> 时钟极性(CPOL)：没有数据传输时时钟线的空闲状态电平
>
> 1. 0:SCK在空闲状态保持低电平
> 2. 1：SCK在空闲状态保持高电平

> 时钟相位(CPHA):时钟线在第几个时钟边沿采样数据
>
> 1. 0:SCK的第一（奇数）边沿进行数据位采样，据在第一个时钟边沿被锁存
> 2. 1：SCK的第二（偶数边沿进行数据位采样，数据在第二个时钟边沿被锁存

| SPI工作模式 | CPOL | CPHA | SCL空闲状态 | 采样边沿 | 采样时刻 |
| ----------- | ---- | ---- | ----------- | -------- | -------- |
| 0           | 0    | 0    | 低电平      | 上升沿   | 奇数边沿 |
| 1           | 0    | 1    | 低电平      | 下降沿   | 偶数边沿 |
| 2           | 1    | 0    | 高电平      | 下降沿   | 奇数边沿 |
| 3           | 1    | 1    | 高电平      | 上升沿   | 偶数边沿 |

SPI协议也需要MCU和芯片使用相同的时钟相位和极性来进行数据通信，比如芯片使用0状态进行通信，那么MCU就不能配置1/2/3状态进行通信，所以配置MCU需要看芯片手册支持什么时钟

![SPI时钟相位图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_14_19_47_202409141419622.png)

# 数据传输过程

## 基本时序单元

### 起始条件

SS从高电平切换到低电平

![开始](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_14_25_40_202409141425358.png)

### 终止条件

SS从低电平切换到高电平

![结束](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_14_25_49_202409141425056.png)

### 交换一个字节的数据传输过程

*`模式0`*

CPOL=0:空闲状态时，SCK为低电平

CPHA=0:SCK第一个边沿移入数据，第二个边沿移出数据

![模式0的数据传输过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_14_32_39_202409141432075.png)

*`模式1`*

CPOL=O:空闲状态时，SCK为低电平

CPHA=1:SCK第一个边沿移出数据，第二个边沿移入数据

![模式1的数据传输过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_14_35_57_202409141435281.png)

*`模式2`*

CPOL=1:空闲状态时，SCK为高电平

CPHA=0:SCK第一个边沿移入数据，第二个边沿移出数据

![模式2的数据传输过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_14_38_29_202409141438701.png)

*`模式3`*

CPOL=1:空闲状态时，SCK为高电平

CPHA=1:SCK第一个边沿移出数据，第二个边沿移入数据

![模3的数据传输过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_14_43_7_202409141443984.png)

## 整体数据传输过程

数据的传输分为以下几个步骤：

1. 主机先将NSS信号拉低，这样保证开始接收数据；
2. 当接收端检测到时钟的边沿信号时，它将立即读取数据线上的信号，这样就得到了一位数据（1bit）;
             由于时钟是随数据一起发送的，因此指定数据的传输速度并不重要，尽管设备将具有可以运行的最高速度（稍后我们将讨论选择合适的时钟边沿和速度）。
3. 主机发送到从机时：主机产生相应的时钟信号，然后数据一位一位地将从MOSI信号线上进行发送到从机；
4. 主机接收从机数据：如果从机需要将数据发送回主机，则主机将继续生成预定数量的时钟信号，并且从机会将数据通过MISO信号线发送；

![整体数据传输过程](https://pic2.zhimg.com/80/v2-069df3709fb1a0c5486acbf620890313_720w.webp)

> [!NOTE]
>
> 注意，SPI是“全双工”（具有单独的发送和接收线路），因此可以在同一时间发送和接收数据，另外SPI的接收硬件可以是一个简单的移位寄存器。这比异步串行通信所需的完整UART要简单得多，并且更加便宜；

![数据传输简图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_14_48_41_202409141448390.png)

# 多从机模式

## 多NSS

1. 通常，每个从机都需要一条单独的SS线。

1. 如果要和特定的从机进行通讯，可以将相应的NSS信号线拉低，并保持其他NSS信号线的状态为高电平；如果同时将两个NSS信号线拉低，则可能会出现乱码，因为从机可能都试图在同一条MISO线上传输数据，最终导致接收数据乱码。

![多从机模式的拓扑图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_15_54_18_202409141554985.png)

## 菊花链

在数字通信世界中，在设备信号（总线信号或中断信号）以串行的方式从一 个设备依次传到下一个设备，不断循环直到数据到达目标设备的方式被称为菊花链。

1. 菊花链的最大缺点是因为是信号串行传输，所以一旦数据链路中的某设备发生故障的时候，它下面优先级较低的设备就不可能得到服务了；
2. 另一方面，距离主机越远的从机，获得服务的优先级越低，所以需要安排好从机的优先级，并且设置总线检测器，如果某个从机超时，则对该从机进行短路，防止单个从机损坏造成整个链路崩溃的情况；

具体的连接如下图所示；

![菊花链拓扑图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_15_55_36_202409141555086.png)

![菊花链通信格式](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_15_55_43_202409141555639.png)

# 实验

## Flash

FLASH是常用的用于储存数据的半导体器件，它具有容量大，可重复擦写、按“扇区/块”擦除、掉电后数据可继续保存的特性。

FLASH是有一个物理特性：只能写0，不能写1，写1靠擦除。

FLASH:主要有NOR FIash和NAND FIash两种类型，NOR和NAND是两种数字门电路。

| 类型       | 特点                                                        | 应用举例         |
| ---------- | ----------------------------------------------------------- | ---------------- |
| NOR FLASH  | 基于字节读写，读取速度快，独立地址/数据线，无坏块，支持XIP  | 25Qxx、程序ROM   |
| NAND FLASH | 基于块读写，读取速度稍慢，地址数据线共用，有坏块，不支持XIP | EMMC、SSD、U盘等 |

## 芯片简介

*`NM25Q18`*

NM25Q128,串行闪存器件，属于N0 R FLASH中的一种，容量为128Mb。擦写周期可达10W次，可以将数据保存达20年之久。

1. SPI数据传输时序:支持模式0(CPOL=0,CPHA=0)和模式3(CPOL=1,CPHA=1)
2. 数据格式:数据长度8位大小，先发高位，再发低位
3. 传输速度:支持标准模式104 M bit/s

![NM25Q18芯片原理图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/14_17_19_51_202409141719507.png)

### NM25Q18存储结构

存储器的地址范围为0x000000 ~0xFFFFFF

那么一个模块大存储大小 = 256 * 16 * 16 * 256 = 2^24 = 0xFFFFFF

一般将一个flash分为多个模块来使用？

> 例如：使用0x00 ~ 0x0000ff作为文件系统使用
>
> 使用0x0000ff~0x00ffff作为文件存储使用等等

> [!IMPORTANT]
>
> 当我们使用的时候只需要往分配好的模块写入或者读出数据即可
>
> 擦除只能擦除最小单元，比如页擦除、扇区擦除及整个芯片擦除，擦除的时间比写入的时间要更长
>
> 写入不能跨页，但是读可以跨页

![存储器映射](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_11_38_0_202409231138184.png)

### 常用命令

NOR FLASHE的指令总数比较多，但是如果只需要实现基本操作，还是比较简单的

一般我们只需要5条指令即可完成对NOR FLASH的基本使用

| 指令 (HEX) | 名称     | 作用                                   |
| ---------- | -------- | -------------------------------------- |
| 0X06       | 写使能   | 写入数据/擦除之前，必须先发送该指令    |
| 0X05       | 读SR1    | 判定FLASH是否处于空闲状态，擦除用      |
| 0X03       | 读数据   | 用于读取NOR FLASH数据                  |
| 0X02       | 页写     | 用于写入NOR FLASH数据，最多写256字节   |
| 0X20       | 扇区擦除 | 扇区擦除指令，最小擦除单位 （4096字节) |

*`写使能`06H*

执行Page Program（页写），Sector Erase（扇区擦除），Block Erase（块擦除），Chip Erase（片擦除），Write Status Register（写状态寄存器）等指令前，都需要写使能

![写使能](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_11_42_50_202409231142621.png)

*`读状态寄存器`05H*

![读状态寄存器](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_11_43_50_202409231143265.png)



*`读时序`03H*

![读时序](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_11_44_17_202409231144847.png)

*`页写时序`02H*

页写命令最多可以向FLASH传输256个字节的数据

![页写时序](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_11_46_8_202409231146058.png)

*`扇区擦除时序`20H*

写入数据前，检查内存空间情况是否满足，不满足需擦除

![扇区擦除时序](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_11_47_55_202409231147211.png)

> [!NOTE]
>
> FLASH初始化的时候，数据全为1，当我们想要写入零时，可以直接将1写为0，但是我们将0写为1时，就不能直接写，一般flash都是先擦除后写入，也就是写入0的过程

*`状态寄存器表`*

| 状态寄存器  | Bit7     | Bit6 | Bit5 | Bit4 | Bit3 | Bit2 | Bit1 | Bito |
| ----------- | -------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 状态寄存器1 | SPR      | RV   | TB   | BP2  | BP1  | BPO  | WEL  | BUSY |
| 状态寄存器2 | SUS      | CMP  | LB3  | LB2  | LB1  | (R)  | QE   | SRP1 |
| 状态寄存器3 | HOLD/RST | DRV1 | DRVO | (R)  | (R)  | WPS  | ADP  | ADS  |

SR寄存器：跟踪芯片的状态

BUSY位指示当前状态：

- 0:空闲状态（硬件自动清除）
- 1:当前处于忙碌状态

WEL位：

- 执行Write Enable指令该位为1，可以页写/扇区or块or片擦除/写状态寄存器
- 0:写禁止，不能页编程/扇区or块or片擦除/写状态寄存器

## 软件模拟

```c


void myspi_ss_set(u8 bitValue)
{
    GPIO_WriteBit(SPI_PORT,SPI_SS,(BitAction)bitValue);
}

void myspi_sck_set(u8 bitValue)
{
    GPIO_WriteBit(SPI_PORT,SPI_SCK,(BitAction)bitValue);
}

void myspi_si_set(u8 bitValue)
{
    GPIO_WriteBit(SPI_PORT,SPI_SI,(BitAction)bitValue);
}

u8 myspi_so_get()
{
    return GPIO_ReadInputDataBit(SPI_PORT,SPI_SO);
}

void myspi_init(void)
{
    GPIO_InitTypeDef GPIO_InitStruct;

    RCC_APB2PeriphClockCmd(SPI_CLK,ENABLE);

    GPIO_InitStruct.GPIO_Pin = SPI_SS | SPI_SCK  | SPI_SI;
    GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(SPI_PORT,&GPIO_InitStruct);

    GPIO_InitStruct.GPIO_Pin = SPI_SO;
    GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_Init(SPI_PORT,&GPIO_InitStruct);

    myspi_ss_set(1);
    myspi_sck_set(0);
}

void myspi_start(void)
{
    myspi_ss_set(0);
}

void myspi_stop(void)
{
    myspi_ss_set(1);
}

u8 myspi_swapByte(u8 data)
{
    u8 receiveData = 0x00;
    for(u8 i = 0; i < 8 ; i++){
        myspi_si_set(data & (0x80 >> i));        //模式0，先移除最高位，发送数据
        myspi_sck_set(1);        //第一个上升沿来临
        if(myspi_so_get() == 1){
            receiveData |= (0x80 >> i);        //接收数据
        }
        myspi_sck_set(0);        //下降沿来临
    }
    return receiveData;
}

void w25q_init(void)
{
    myspi_init();
}

void w25q_readId(u8* MID,u16* DID)
{
    myspi_start();
    myspi_swapByte(GET_W25Q_ID);        //发送读取flash的制造商ID和设备ID的指令号9f
    *MID = myspi_swapByte(SPI_DUMMY);        //接收从机返回来的数据，为了交换数据，我们任意给一个数据ff，ff的目的就是将对面有意义的数据置换过来
    *DID = myspi_swapByte(SPI_DUMMY);        //读取高八位
    *DID <<= 8;
    *DID |= myspi_swapByte(SPI_DUMMY);        //读取低八位，注意需要使用|
    myspi_stop();
}

void w25q_writeEnable(void)
{
    myspi_start();
    myspi_swapByte(W25Q_WRITEENABLE);
    myspi_stop();
}

void w25q_waitBusy()
{
    u32 timeOut = 100000;
    myspi_start();
    myspi_swapByte(W25Q_READ_REGISTER_1);
    while((myspi_swapByte(SPI_DUMMY) & 0x01) == 0x01){        //判断芯片是否空闲，使用掩码取出最低位，最低位为busy位，1为繁忙，0为空闲
        timeOut--;
        if(timeOut == 0){
            break;
        }

    }
    myspi_stop();

}

void w25q_pageProgram(u32 address,u8* dataArray,u16 count)//页只能写0-256，如果超过256，则后续的页将会覆盖之前的页
{
    w25q_writeEnable();
    myspi_start();
    myspi_swapByte(W25Q_PAGEPROGRAM);        //发送指令
    myspi_swapByte(address >> 16);        //发送地址，先发送高字节
    myspi_swapByte(address >> 8);
    myspi_swapByte(address);
    for(u16 i = 0; i < count; i++){
        myspi_swapByte(dataArray[i]);
    }
    myspi_stop();
    w25q_waitBusy();
}

//擦除扇区
void w25q_sectorErase(u32 address)
{
    w25q_writeEnable();
    myspi_start();
    myspi_swapByte(W25Q_SECTOR_EARSE_4KB);        //发送指令
    myspi_swapByte(address >> 16);        //发送地址，先发送高字节
    myspi_swapByte(address >> 8);
    myspi_swapByte(address);
    myspi_stop();
    w25q_waitBusy();
}

void w25q_readData(u32 address,u8* dataArray,u16 count)
{
    myspi_start();
    myspi_swapByte(W25Q_READ_DATA);        //发送指令
    myspi_swapByte(address >> 16);        //发送地址，先发送高字节
    myspi_swapByte(address >> 8);
    myspi_swapByte(address);
    for(u32 i = 0; i < count; i++){
        dataArray[i] = myspi_swapByte(SPI_DUMMY);
    }
    myspi_stop();
}


int main(void){

    char str[50];
    char str_tmp[50];
    u8 arrayWrite[] = {0x01,0x02,0x03,0x04};
    u8 arrayRead[4];

    w25q_init();

    serial_init();                                                                                                                                                                                                                                                                                                                                                                       

    w25q_sectorErase(0x000000);
    w25q_pageProgram(0x000000,arrayWrite,4);
    w25q_readData(0x000000,arrayRead,4);


    sprintf(str,"write: %u %u %u %u\n ",arrayWrite[0],arrayWrite[1],arrayWrite[2],arrayWrite[3]);
    send_string(str);

    sprintf(str_tmp,"read: %u %u %u %u\n ",arrayRead[0],arrayRead[1],arrayRead[2],arrayRead[3]);
    send_string(str_tmp);

    while(1){

    }
}

```

## 硬件部分

主模式全双工连续传输

使用模式3：

![使用模式3](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_11_52_40_202409231152615.png)

非连续传输：

![非连续传输](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_11_52_46_202409231152698.png)

代码

```c

void hardware_myspi_init()
{
	GPIO_InitTypeDef GPIO_InitStruct;
	
	RCC_APB2PeriphClockCmd(SPI_CLK,ENABLE);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_SPI2,ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = SPI_SS;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(SPI_PORT,&GPIO_InitStruct);
	
	GPIO_InitStruct.GPIO_Pin = SPI_SCK | SPI_SI;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_Init(SPI_PORT,&GPIO_InitStruct);
	
	GPIO_InitStruct.GPIO_Pin = SPI_SO;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_Init(SPI_PORT,&GPIO_InitStruct);
	
	SPI_InitTypeDef SPI_InitStruct;
	SPI_InitStruct.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_128;
	SPI_InitStruct.SPI_CPHA = SPI_CPHA_1Edge;
	SPI_InitStruct.SPI_CPOL = SPI_CPOL_Low;
	SPI_InitStruct.SPI_CRCPolynomial = 7;
	SPI_InitStruct.SPI_DataSize = SPI_DataSize_8b;        //8bit
	SPI_InitStruct.SPI_Direction = SPI_Direction_2Lines_FullDuplex;
	SPI_InitStruct.SPI_FirstBit = SPI_FirstBit_MSB;        //高位先行
	SPI_InitStruct.SPI_Mode = SPI_Mode_Master;
	SPI_InitStruct.SPI_NSS = SPI_NSS_Soft;
	SPI_Init(SPI2,&SPI_InitStruct);
	
	SPI_Cmd(SPI2,ENABLE);
	myspi_ss_set(1);
}

u8 hardware_myspi_swapData(u8 data)
{
	while(SPI_I2S_GetFlagStatus(SPI2,SPI_I2S_FLAG_TXE) != SET);        //等待TEX变为1，发送寄存器为空，如果发送寄存器不为空，则需要等待
	SPI_I2S_SendData(SPI2,data);        //传入data之后，将会写入到TDR中，之后TDR将会自动转入移位寄存器。由于使用非连续传输，所以在传入数据的这一时间段，下一个数据不会转移到TDR中
	while(SPI_I2S_GetFlagStatus(SPI2,SPI_I2S_FLAG_RXNE) != SET);//在发送的同时，MISO还会接收，发送和接收是同步的，也就是说接收移位完成也代表发送移位完成，接收数据完成后，会置RXNE标志位，所以只需要等待RXNE出现即可
	return SPI_I2S_ReceiveData(SPI2);

}
```































