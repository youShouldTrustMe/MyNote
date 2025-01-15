# 参考链接

[STM32 IIC通讯协议详解—小白入门_stm32 iic速率修改-CSDN博客](https://blog.csdn.net/qq_54747686/article/details/119202856)

# 概述

IIC:Inter Integrated Circuit,集成电路总线，是一种同步串行半双工通信总线。

总线：传输数据的通道

协议：传输数据的规则

# 物理层

![ICC电路结构](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_13_59_26_202409131359673.png)

物理层描述：

1. 它是一个支持设备的总线。“总线”指多个设备共用的信号线。在一个IIC通信总线中，可连接==多个IIC通信设备==，支持多个通信主机及多个通信从机。
2. 一个IIC总线只使用两条总线线路。
   1. 一条双向串行数据线（SDA），数据线用来表示数据
   2. 一条串行时钟线（SCL），时钟线用于同步数据的收发。
3. 每个连接到总线的设备都有一个==独立==的地址，主机可以利用这个地址进行不同设备之间的访问。
4. 总线通过上拉电阻接到电源。
   1. 当IIC设备空闲时，会输出高阻态，而当所有设备都空闲，都输出高阻态，由上拉电阻把总线拉成高电平。
   2. 当总线上开始发送数据，会将总线拉低，所以同一时刻，总线上只能有一条数据
5. 多个主机同时使用总线时，为了防止数据冲突，会利用==仲裁==的方式决定由哪个设备占用总线。
6. 具有三种传输模式：
   1. 标准模式传输速率为100kbps
   2. 快速模式为400kbps
   3. 高速模式可达3.4Mbps，但目前大多IIC设备尚不支持高速模式。
7. 连接到相同总线的IIC设备数量收到总线的最大电容400pF限制。

在一主多从的情况下

- 主机拥有SCL的绝对控制权，所以主机的SCL可以配置成推挽输出，所有从机的SCL都配置成浮空输入或者上拉输入，主机发送，从机接收

- 由于是半双工的协议，所以主机的SDA在发送的时侯是输出，在接收的时候是输入，同样的，从机的SDA也会在输入和输出之间反复切换，为了协调这一点，==禁止所有设备输出强上拉的高电平，采用外置弱上拉电阻加开漏输出的电路结构==。

  ![内部结构](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_14_13_11_202409131413040.png)

- 图中的SCLK就是连接的结构图的SCL，SDA同理，所以才规定SCL和SDA都必须配置成==开漏输出==模式

  - 如果使用的是推挽输出，那么当SDA释放时，输出为强。
  - 上图就是主机内部结构，左边为SCL，右侧为SDA，SCL经过一个施密特触发器对数据进行缓冲，任何设备在任何时候都是可以输入的。
  - 输出低电平时，下管导通，强下拉，输出高电平时，下管断开，但是没有上管了，此时引脚属于浮空的状态，这样的话所有的设备只能输出低电平，而不能输出高电平，为了避免输出出现浮空状态，在外部添加一个上拉电阻（弱上拉），这样做的好处
    - 完全杜绝了电源短路的现象，保证电路的安全
    - 避免了引脚模式的频繁切换，开漏加弱上拉的模式，同时兼具了输入和输出的功能，如果想输出，就去拉杆子或者放手，操作杆子的变化即可，如果想输入，只需要直接放手，然后观察杆子的高低即可
    - 存在线与的现象，只要有一个或者多个设备输出了低电平，总线就处于低电平，只有所有的设备都输出高电平，总线才处于高电平

# 协议层

## 基本时序单元

*`开始`*

SCL初始保持高电平也是通过上拉电阻实现的，默认输出高电平

如果想要开始通讯，就先将SDA拉下来，当从机捕获到SCL高电平且SDA==下降沿==信号时，就会进行自身的复位，等待主机的召唤，主机将SCL拉下来，将SCL拉下来一方面是为了占用这个总线，另外一方面也方便基本单元的拼接

> SCL高电平期间，SDA从高电平切换到低电平

![开始信号](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_14_26_31_202409131426942.png)

*`结束`*

和起始相反，结束信号是SCL先放手，SDA再放手，产生一个上升沿，这个上升沿触发终止条件

> SCL高电平期间，SDA从低电平切换到高电平

![结束信号](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_14_27_45_202409131427923.png)

*`发送`*

发送一个字节：SCL低电平期间，主机将数据位依次放到SDA线上(高位先行)，然后释放SCL,从机将在SCL高电平期间读取数据位所以SCL高电平期间SDA不允许有数据变化，依次循环上述过程8次，即可发送一个字节

![发送过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_14_32_52_202409131432013.png)

> [!NOTE]
>
> I2C是高位先行，也就是先发送最高位B7，串口是低位先行。SCL低电平期间，允许改变SDA电平，也就是说在低电平期间，SDA可以切换1或0。当SCL变成高电平时，从机读取SDA，所以高电平期间，SDA不允许变化，一般都是在上升沿这个时刻，从机就已经完成了读取。从图中可以看出，主机基本上不用着急数据的存放读取，因为主机有时钟的主导权，但是从机就必须要尽快的读取和存放数据.可以理解为所有设备和从机都始终处于输入模式。当主机需要发送的时候，就可以主动去拉低SDA，而主机在被动接收的时候，就必须先释放SDA

发送应答：主机在接收完一个字节之后，在下一个时钟发送一位数据，数据0表示应答，数据1表示非应答

![发送应答](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_14_49_38_202409131449061.png)

*`接收`*

接收一个字节：SCL低电平期间，从机将数据位依次放到SDA线上(高位先行)，然后释放SCL,主机将在SCL高电平期间读取数据位所以SCL高电平期间SDA不允许有数据变化，依次循环上述过程8次，即可接收一个字节（主机在接收之前，需要释放SDA)

![接收过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_14_53_2_202409131453678.png)

> [!TIP]
>
> 要理解一旦有一个设备为低电平，整个总线都为低电平。当主机将SDA松开时（发送完一个字节的数据之后），如果没有设备下拉为低电平，那么整条总线就为高电平，说明没有设备给予应答，但是一旦有设备下拉SDA，就说明有设备收到数据，且给出应答

接收应答：主机在发送完一个字节之后，在下一个时钟接收一位数据，判断从机是否应答，数据0表示应答，数据1表示非应答（主机在接收之前，需要释放SDA)

![接收应答](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_14_56_11_202409131456092.png)

## 基本读写过程

![基本读写过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_0_21_202409131500703.png)

1.  S 表示由主机的 I2C 接口产生的传输起始信号(S)，这时连接到 I2C 总线上的所有从机都会接收到这个信号。
2. 起始信号产生后，所有从机就开始等待主机紧接下来广播的从机地址信号(SLAVE_ADDRESS)。在 I2C 总线上，每个设备的地址都是唯一的（每个从机地址码不同，如果相同，可以通过调整从机上的引脚改变从机地址）
3. 当主机广播的地址与某个设备地址相同时，这个设备就被选中了，没被选中的设备将会忽略之后的数据信号。根据 I2C 协议，这个从机地址可以是 7 位或 10 位。
   1. 七位地址就是第一个字节就是地址+读写方向
   2. 十位地址就是使用两个字节作为地址，其中前面是标志位，表示使用10位地址，即11110+地址+读写方向。
4. 在地址位之后，是传输方向的选择位，该位为 0 时，表示后面的数据传输方向是由主机传输至从机，即主机向从机写数据。该位为 1 时，则相反，即主机由从机读数据。
5. 从机接收到匹配的地址后，主机或从机会返回一个应答(ACK)或非应(NACK)信号，只有接收到应答信号后，主机才能继续发送或接收数据。

*`写数据`*

若配置的方向传输位为“写数据”方向，广播完地址，接收到应答信号后，一般设备都会向从机发送一个寄存器的地址，用于目标寄存器，当收到应答时，主机开始正式向从机传输数据(DATA)，数据包的大小为 8位，**主机每发送完一个字节数据，都要等待从机的应答信号(ACK)**，重复这个过程，可以向从机传输 N 个数据，这个 N 没有大小限制。当数据传输结束时，主机向从机发送一个停止传输信号§，表示不再传输数据。

*`读数据 `*

若配置的方向传输位为“读数据”方向，广播完地址，接收到应答信号后:

1. 未指定地址读
   1. 主机立马就开始读取数据，但是没有指定寄存器地址，那么从机是读的哪里的数据呢？
   2. 实际上在从机中，所有的寄存器被分配到了一个线性区域中，并且有一个单独的指针变量。指示其中一个寄存器，这个指针上电默认指向0地址，并且没写入或者读取一个字节后，这个指针就会自动自增一次，移动到下一个位置。
   3. 所以当主机要读取数据时，主机没有指定读取哪个地址，从机就会返回当前指针指向的寄存器中的值。
2. 指定地址读
   1. 主机会发送一个写操作，接收到应答之后，主机会再次发送一个字节，用来指定需要读的地址，当从机接收到这个地址之后就会将寄存器指针改写为接收到的地址。这个时候我们就要切换成读数据模式了，所以要重新发送一个起始信号，重新开始读

从机开始向主机返回数据(DATA)，数据包大小也为 8 位，从机每发送完一个数据，都会等待主机的应答信号(ACK)，重复这个过程，可以返回 N 个数据，这个 N 也没有大小限制。当主机希望停止接收数据时，就向从机返回一个非应答信号(NACK)，则从机自动停止数据传输。

*`读和写数据`*

除了基本的读写，I2C 通讯更常用的是复合格式，该传输过程有两次起始信号(S)。一般在第一次传输中，主机通过SLAVE_ADDRESS 寻找到从设备后，发送一段“数据”，这段数据通常用于表示从设备内部的寄存器或存储器地址(注意区分它与 SLAVE_ADDRESS 的区别)；在第二次的传输中，对该地址的内容进行读或写。也就是说，第一次通讯是告诉从机读写地址，第二次则是读写的实际内容。

*`数据的有效性`*

IIC 使用 SDA 信号线来传输数据，使用 SCL 信号线进行数据同步。SDA数据线在 SCL 的每个时钟周期传输一位数据。传输时，SCL 为高电平的时候 SDA 表示的数据有效，即此时的 SDA 为高电平时表示数据“1”，为低电平时表示数据“0”。当 SCL为低电平时，SDA 的数据无效，一般在这个时候 SDA 进行电平切换，为下一次表示数据做好准备。

![数据的有效性](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_18_14_202409131518847.png)

*`地址和数据方向`*

I2C 总线上的每个设备都有自己的独立地址，主机发起通讯时，通过 SDA 信号线发送设备地址(SLAVE_ADDRESS)来查找从机。I2C 协议规定设备地址可以是 7 位或 10 位，实际中 7 位的地址应用比较广泛。紧跟设备地址的一个数据位用来表示数据传输方向，它是数据方向位(R/W)，第 8 位或第 11 位。数据方向位为“1”时表示主机由从机读数据，该位为“0”时表示主机向从机写数据。

![地址和数据方向](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_21_27_202409131521743.png)

*`响应`*

I2C 的数据和地址传输都带响应。响应包括“应答(ACK)”和“非应答(NACK)”两种信号。作为数据接收端时，当设备(无论主从机)接收到 I2C 传输的一个字节数据或地址后，若希望对方继续发送数据，则需要向对方发送“应答(ACK)”信号，发送方会继续发送下一个数据；若接收端希望结束数据传输，则向对方发送“非应答(NACK)”信号，发送方接收到该信号后会产生一个停止信号，结束信号传输。

![响应](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_22_23_202409131522727.png)

## 完整的传输过程

![完整的传输过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_24_11_202409131524450.png)

# IIC架构

![IIC架构](https://i-blog.csdnimg.cn/blog_migrate/dc6c8f7b8b34749ed664c1f2bda8d920.png#pic_center)

## 通信引脚

I2C 的所有硬件架构都是根据图中左侧 SCL 线和 SDA 线展开的(其中的SMBA 线用于SMBUS 的警告信号，I2C 通讯没有使用)。STM32 芯片有多个 I2C 外设，它们的 I2C 通讯信号引出到不同的 GPIO 引脚上，使用时必须配置到这些指定的引脚。

## 时钟的控制逻辑

SCL 线的时钟信号，由 I2C 接口根据时钟控制寄存器(CCR)控制，控制的参数主要为时钟频率。配置 I2C 的 CCR 寄存器可修改通讯速率相关的参数：可选择 I2C 通讯的“标准/快速”模式，这两个模式分别 I2C 对应 100/400Kbit/s 的通讯速率。

在快速模式下可选择 SCL 时钟的占空比，可选 Tlow/Thigh=2 或Tlow/Thigh=16/9模式，我们知道 I2C 协议在 SCL 高电平时对 SDA 信号采样，SCL 低电平时 SDA准备下一个数据，修改 SCL 的高低电平比会影响数据采样，但其实这两个模式的比例差别并不大，若不是要求非常严格，这里随便选就可以了。

CCR 寄存器中还有一个 12 位的配置因子 CCR，它与 I2C 外设的输入时钟源共同作用，产生 SCL 时钟，STM32 的 I2C 外设都挂载在 APB1 总线上，使用 APB1 的时钟源 PCLK1，SCL 信号线的输出时钟公式如下：

**标准模式：**
$$
Thigh = CCR*TPCKL1  \\
Tlow = CCR*TPCLK1
$$
**快速模式中 Tlow/Thigh=2 时：**
$$
Thigh = CCR*TPCKL1 \\
Tlow = 2*CCR*TPCKL1
$$
**快速模式中 Tlow/Thigh=16/9 时**：
$$
Thigh = 9*CCR*TPCKL1 \\
Tlow = 16*CCR*TPCKL1
$$
*例如，我们的 PCLK1=36MHz，想要配置 400Kbit/s 的速率，计算方式如下：*

*PCLK 时钟周期： TPCLK1 = 1/36000000*

*目标 SCL 时钟周期： TSCL = 1/400000*

*SCL 时钟周期内的高电平时间： THIGH = TSCL/3*

*SCL 时钟周期内的低电平时间： TLOW = 2*TSCL/3

计算 CCR 的值： CCR = THIGH/TPCLK1 = 30

**计算结果得出 CCR 为 30，向该寄存器位写入此值则可以控制 IIC 的通讯速率为400KHz，其实即使配置出来的 SCL 时钟不完全等于标准的 400KHz，IIC 通讯的正确性也不会受到影响，因为所有数据通讯都是由 SCL 协调的，只要它的时钟频率不远高于标准即可。**

## 数据控制逻辑

I2C 的 SDA 信号主要连接到数据移位寄存器上，数据移位寄存器的数据来源及目标是数据寄存器(DR)、地址寄存器(OAR)、PEC 寄存器以及 SDA 数据线。

当向外发送数据的时候，将一个字节的数据写入到数据寄存器（DR）中，数据移位寄存器以“数据寄存器”为数据源，把数据一位一位地通过 SDA 信号线发送出去。这时，置状态寄存器的TEX位为1，表示发送寄存器为空；

当从外部接收数据的时候，数据移位寄存器把 SDA 信号线采样到的数据一位一位地存储到移位寄存器中，当一个字节的数据收齐之后，数据就整体从移位寄存器转到数据寄存器中，同时置RXNE，表示接收寄存器非空。

若使能了数据校验，接收到的数据会经过 PEC 计算器运算，运算结果存储在“PEC 寄存器”中。

当 STM32 的 I2C 工作在从机模式的时候，接收到设备地址信号时，数据移位寄存器会把接收到的地址与 STM32 的自身“I2C 地址寄存器”的值作比较，以便响应主机的寻址。STM32 的自身 I2C 地址(自定义地址)可通过修改“自身地址寄存器”修改，支持同时使用两个 I2C 设备地址，两个地址分别存储在 OAR1 和 OAR2 中，当有主机广播从机地址时，通过比较器比较，发现相同，则会作为从机响应外部主机的召唤。

## 整体逻辑控制

整体控制逻辑负责协调整个 I2C 外设，控制逻辑的工作模式根据我们配置的“控制寄存器(CR1/CR2)”的参数而改变。在外设工作时，控制逻辑会根据外设的工作状态修改“状态寄存器(SR1 和 SR2)”，我们只要读取这些寄存器相关的寄存器位，就可以了解 I2C的工作状态。除此之外，控制逻辑还根据要求，负责控制产生 I2C 中断信号、DMA 请求及各种 I2C 的通讯信号(起始、停止、响应信号等)。

## 通信过程

使用 I2C 外设通讯时，在通讯的不同阶段它会对“状态寄存器(SR1 及 SR2)”的不同数据位写入参数，我们通过读取这些寄存器标志来了解通讯状态。

### 主发送器

主发送器发送流程及事件说明如下：

**(1)** 控制产生起始信号(S)，当发生起始信号后，它产生事件“EV5”，并会对 SR1 寄存器的“SB”位置 1，表示起始信号已经发送；

**(2)** 紧接着发送设备地址并等待应答信号，若有从机应答，则产生事件“EV6”及“EV8”，这时 SR1 寄存器的“ADDR”位及“TXE”位被置 1，ADDR 为 1 表示地址已经发送，TXE 为 1 表示数据寄存器为空；

**(3)** 以上步骤正常执行并对 ADDR 位清零后，我们往 I2C 的“数据寄存器 DR”写入要发送的数据，这时 TXE 位会被重置 0，表示数据寄存器非空，I2C 外设通过SDA 信号线一位位把数据发送出去后，又会产生“EV8”事件，即 TXE 位被置 1，重复这个过程，就可以发送多个字节数据了；

**(4)** 当我们发送数据完成后，控制 I2C 设备产生一个停止信号§，这个时候会产生EV8_2 事件，SR1 的 TXE 位及 BTF 位都被置 1，表示通讯结束。

![主发送器](https://i-blog.csdnimg.cn/blog_migrate/443ec9fc1f29302281ea371dadbe7fcf.png#pic_center)

### 主接受器

主接收器接收流程及事件说明如下：

**(1)** 同主发送流程，起始信号(S)是由主机端产生的，控制发生起始信号后，它产生事件“EV5”，并会对 SR1 寄存器的“SB”位置 1，表示起始信号已经发送；

**(2)** 紧接着发送设备地址并等待应答信号，若有从机应答，则产生事件“EV6”这时SR1 寄存器的“ADDR”位被置 1，表示地址已经发送。

**(3)** 从机端接收到地址后，开始向主机端发送数据。当主机接收到这些数据后，会产生“EV7”事件，SR1 寄存器的 RXNE 被置 1，表示接收数据寄存器非空，我们读取该寄存器后，可对数据寄存器清空，以便接收下一次数据。此时我们可以控制 I2C 发送应答信号(ACK)或非应答信号(NACK)，若应答，则重复以上步骤接收数据，若非应答，则停止传输；

**(4)** 发送非应答信号后，产生停止信号§，结束传输。

在发送和接收过程中，有的事件不只是标志了我们上面提到的状态位，还可能同时标志主机状态之类的状态位，而且读了之后还需要清除标志位，比较复杂。我们可使用STM32 标准库函数来直接检测这些事件的复合标志，降低编程难度。

![主接受器](https://i-blog.csdnimg.cn/blog_migrate/a85e23dfa0be2a243a8298d34e1ffe2d.png#pic_center)

# 实验

## 软件实现

### 简介

EEPROM是一种掉电后据不丢失的储存器，常用来存储一些配置信息，在系统重新上电时就可以加载。

AT24C02是一个2Kbit的EEPROM存储器，使用IIC通信方式。

![EEPROM](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_41_23_202409131541708.png)

### 详细参数

| **AT24Cxx** | **容量（**bit）    | **页数** | **页内字节数** | **数据地址(占用bit数)** |
| ----------- | ------------------ | -------- | -------------- | ----------------------- |
| AT24C01     | 1K bit (128 B)     | 16       | 8 Byte         | 7bit                    |
| AT24C02     | 2K bit (256 B)     | 32       | 8 Byte         | 8bit                    |
| AT24C04     | 4K bit (512 B)     | 32       | 16 Byte        | 9bit                    |
| AT24C08     | 8K bit (1024 B)    | 64       | 16 Byte        | 10bit                   |
| AT24C16     | 16K bit (2048 B)   | 128      | 16 Byte        | 11bit                   |
| AT24C32     | 32K bit (4096 B)   | 128      | 32 Byte        | 12bit                   |
| AT24C64     | 64K bit (8192 B)   | 256      | 32 Byte        | 13bit                   |
| AT24C128    | 128K bit (16384 B) | 256      | 64 Byte        | 14bit                   |
| AT24C256    | 256K bit (32768 B) | 512      | 64 Byte        | 15bit                   |
| AT24C512    | 512K bit (65535 B) | 512      | 128 Byte       | 16bit                   |

### 通讯地址

通讯地址的高位一般都是由厂家设置好的，用户只可调整低位

![通讯地址](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_43_14_202409131543824.png)

### 读写时序

写操作：

1. AT24C02支持字节写模式和页写模式。
2. 字节写模式就是一个地址一个数据进行写入。
3. 页写模式就是连续写入数据。只需要写一个地址，连续写入据时地址会自增，但存在页的限制，超出一页时，超出数据覆盖原先写入的数据。但读会自动翻页。

![写操作](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_45_15_202409131545076.png)

读操作：

1. AT24C02支持当前地址读模式，随机地址读模式和顺序读模式。
2. 当前读模式是基于上一次读/写操作的最后位置继续读出数据。
3. 随机地址读模式是指定地址读出数据。
4. 顺序读模式是连续读出数据。

![读操作](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_45_24_202409131545361.png)

### 软件和硬件实现的区别

| **IIC**     | **用法**         | **速度** | **稳定性** | **管脚**           |
| ----------- | ---------------- | -------- | ---------- | ------------------ |
| **硬件IIC** | 比较复杂         | 快       | 较稳定     | 需使用特定管脚     |
| **软件IIC** | 操作过程比较清晰 | 较慢     | 稳定       | 任意管脚，比较灵活 |

### 代码

```c

void SCL_set(u8 bitValue)
{
    GPIO_WriteBit(SCL_PORT,SCL_PIN,(BitAction)bitValue);
    Delay_us(10);
}

void SDA_set(u8 bitValue)
{
    GPIO_WriteBit(SDA_PORT,SDA_PIN,(BitAction)bitValue);
    Delay_us(10);
}

u8 SDA_read(void)
{
    u8 bitValue;
    bitValue = GPIO_ReadInputDataBit(SDA_PORT,SDA_PIN);
    Delay_us(10);
    return bitValue;
}

void myi2c_init(void)
{

    GPIO_InitTypeDef GPIO_InitStruct;

    RCC_APB2PeriphClockCmd(IIC_CLK,ENABLE);

    GPIO_InitStruct.GPIO_Pin = SCL_PIN | SDA_PIN;
    GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
    GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;

    GPIO_Init(SCL_PORT,&GPIO_InitStruct);

    GPIO_SetBits(SCL_PORT,SCL_PIN | SDA_PIN);        //将SCL和SDA都释放

}

void myi2c_start(void)
{
    //首先先释放SCL和SDA
    SDA_set(1);        
    SCL_set(1);        
    //起始信号开始,先拉低SDA，之后再拉低SCL
    SDA_set(0);
    SCL_set(0);
}

void myi2c_stop(void)
{
    //为了确保SDA出现上升沿，我们需要先将SDA拉低，然后再释放 
    SDA_set(0);
    SCL_set(1);
    SDA_set(1);
}

void myi2c_sendByte(u8 data)
{
    //保证发送完毕后使得SCL为低
    for(u8 i = 0;i < 8; i++){
        SDA_set(data & (0x80 >> i));
        SCL_set(1);
        SCL_set(0);
    }
}

u8 myi2c_receiveByte()
{
    u8 data = 0x00;
    SDA_set(1);        //主机释放SDA，这个时候从机就可以开始任意写数据了
    for(u8 i = 0; i < 8; i++){
        SCL_set(1);        //主机释放SCL，主机开始从SDA读取数据，此时SCL不能改变
        if(SDA_read() == 1){
            data |= (0x80 >> i); 
        }
        SCL_set(0);
    }

    return data; 

}

void myi2c_sendAck(u8 ACKbit)
{
    SDA_set(ACKbit);
    SCL_set(1);
    SCL_set(0);
}

u8 myi2c_receiveAck()
{
    u8 ACKbit = 0x00;
    SDA_set(1);        //主机释放SDA，这个时候从机就可以开始任意写数据了
    SCL_set(1);        //主机释放SCL，主机开始从SDA读取数据，此时SCL不能改变
    ACKbit = SDA_read();
    SCL_set(0);

    return ACKbit; 

}


void eeprom_writeReg(u8 address,u8 data)
{
    myi2c_start();
    myi2c_sendByte(EEPROM_ADDRESS);
    myi2c_receiveAck();
    myi2c_sendByte(address);
    myi2c_receiveAck();
    myi2c_sendByte(data);
    myi2c_receiveAck();
    myi2c_stop();
    //由于eeprom写入较慢，所以需要延迟10ms才能生效
    Delay_ms(10);
}

u8 eeprom_readReg(u8 address)
{
    u8 data;
    myi2c_start();
    myi2c_sendByte(EEPROM_ADDRESS);
    myi2c_receiveAck();
    myi2c_sendByte(address);
    myi2c_receiveAck();

    myi2c_start();
    myi2c_sendByte(EEPROM_ADDRESS | 0x01);
    myi2c_receiveAck();
    data = myi2c_receiveByte();
    myi2c_sendAck(1);
    myi2c_stop();

    return data;

}

int main(void){

    char str[50];
    u8 data = 0x00;

    myi2c_init();
    serial_init();                                                                                                                                                                                                                                                                                                                                                                       

    eeprom_writeReg(0x19,0x55);
    data = eeprom_readReg(0x19);

    sprintf(str,"get the data: %x\n",data);
    send_string(str);

    while(1){

    }
}
```



## 硬件实现

![硬件实现](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_47_5_202409131547001.png)

主机发送

当起始信号来临时，会产生一个EV5事件，可以将这些EV事件看作是标志位

![主机发送](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_47_39_202409131547273.png)

## 主机接收

![主机接收](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/13_15_47_54_202409131547396.png)

## 代码

```c
void hareWare_myi2c_init()
{
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C2,ENABLE);
	RCC_APB2PeriphClockCmd(HARDWARE_IIC_CLK,ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	GPIO_InitStruct.GPIO_Pin = HARDWARE_IIC_SCL_PIN | HARDWARE_IIC_SDA_PIN;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	
	GPIO_Init(HARDWARE_IIC_PORT,&GPIO_InitStruct);
	
	I2C_InitTypeDef I2C_InitStruct;
	I2C_InitStruct.I2C_Ack = I2C_Ack_Enable;
	I2C_InitStruct.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;
	I2C_InitStruct.I2C_ClockSpeed = 50000;
	I2C_InitStruct.I2C_DutyCycle = I2C_DutyCycle_2;	//只有在时钟频率大于100KHz的情况下才有用，在小于100KHz的情况下，低电平时间比高电平时间约为1：1
	I2C_InitStruct.I2C_Mode = I2C_Mode_I2C;
	I2C_InitStruct.I2C_OwnAddress1 = 0x00;
	I2C_Init(I2C2,&I2C_InitStruct);
	
	I2C_Cmd(I2C2,ENABLE);
	
}

//防止等待不到事件而出现卡死状态
void waitEvent(I2C_TypeDef* I2Cx, uint32_t I2C_EVENT)
{
	u32 timeout = 10000;
	while(I2C_CheckEvent(I2Cx,I2C_EVENT) != SUCCESS){
		timeout--;
		if(timeout == 0) 
			return;
	}
}

void hardware_eeprom_writeReg(u8 address,u8 data)
{
	I2C_GenerateSTART(I2C2,ENABLE);
	waitEvent(I2C2,I2C_EVENT_MASTER_MODE_SELECT);	//相较于软件的阻塞型的通信，硬件I2C需要每次检查事件（EV5）是否发生
	
	I2C_Send7bitAddress(I2C2,EEPROM_ADDRESS,I2C_Direction_Transmitter);
	waitEvent(I2C2,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED);	//这里就不需要等待ACK了，只需要检查事件（EV6）是否发生即可
	
	I2C_SendData(I2C2,address);
	waitEvent(I2C2,I2C_EVENT_MASTER_BYTE_TRANSMITTING);	//等待发送事件（EV8），如果想要连续发送，只需要每次发送完等待EV8即可
	
	I2C_SendData(I2C2,data);
	waitEvent(I2C2,I2C_EVENT_MASTER_BYTE_TRANSMITTED);	//发送结束后，我们需要等待EV8_2事件，标志传输结束
	
	I2C_GenerateSTOP(I2C2,ENABLE);
	
}

u8 hardware_eeprom_readReg(u8 address)
{
	u8 data;
	I2C_GenerateSTART(I2C2,ENABLE);
	waitEvent(I2C2,I2C_EVENT_MASTER_MODE_SELECT);	//相较于软件的阻塞型的通信，硬件I2C需要每次检查事件（EV5）是否发生
	
	I2C_Send7bitAddress(I2C2,EEPROM_ADDRESS,I2C_Direction_Transmitter);
	waitEvent(I2C2,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED);	//这里就不需要等待ACK了，只需要检查事件（EV6）是否发生即可
	
	I2C_SendData(I2C2,address);
	waitEvent(I2C2,I2C_EVENT_MASTER_BYTE_TRANSMITTED);	//这里等待的事件可以是TRANSMITTING，也可以是TRANSMITED
	
	I2C_GenerateSTART(I2C2,ENABLE);
	waitEvent(I2C2,I2C_EVENT_MASTER_MODE_SELECT);	//重复起始条件
	
	I2C_Send7bitAddress(I2C2,EEPROM_ADDRESS,I2C_Direction_Receiver);
	waitEvent(I2C2,I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED);	//设置读取模式
	
	I2C_AcknowledgeConfig(I2C2,DISABLE);	//在接收最后一个字节之前，就需要提前将ack置零，同时设置停止位STOP,对应于EV6_1事件，EV6_1事件仅用于等待接收一个字节的情况
	I2C_GenerateSTOP(I2C2,ENABLE);
	waitEvent(I2C2,I2C_EVENT_MASTER_BYTE_RECEIVED);	//等待事件EV7
	
	data = I2C_ReceiveData(I2C2);	//该函数可以读取DR中的数值
	
	I2C_AcknowledgeConfig(I2C2,ENABLE);	//这里是为了程序收多个字节
	
	return data;
}


int main(void){
	
	char str[50];
	u8 data = 0x00;
	
	hareWare_myi2c_init();
	serial_init();                                                                                                                                                                                                                                                                                                                                                                       
	
	hardware_eeprom_writeReg(0x19,0x66);
	Delay_ms(20);      //注意，写完之后一定要延时20ms，因为eeprom写入较慢，所以如果不延时20ms的话将会卡死
	data = hardware_eeprom_readReg(0x19);

	sprintf(str,"get the data: %x\n",data);
	send_string(str);
	
	while(1){

	}
}
```



















































