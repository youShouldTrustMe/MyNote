# 1 概述

CAPL全称为Communication Access Programming Language，即通信访问编程语言。它是Vector公司专门为CANoe开发环境设计的编程语言，在语法和概念上与C语言类似。借助CAPL，用户可以编写程序并应用到网络的各个节点上，也可以利用CAPL编程加强测量分析功能，以及搭建高效的自动化测试模块。  

# 2 CAPL开发环境

![image-20240708153534486](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_15_35_34_202407081535638.png)

# 3 数据类型

## 3.1 全局变量和局部变量

在CAPL中，全局变量需要被声明在variables部分，同时可以使用直接赋值方法进行初始化。如果没有初始化，编译器会执行自动初始化，默认值为0。全局变量的作用域包括整个CAPL文件以及与此文件有链接的其他CAPL文件。

与C语言不同，局部变量在CAPL中总是被静态地创建。这意味着初始化只在程序体启动时执行，当再次进入程序时，局部变量被假定是上一次跳出程序时的值。局部变量的作用域，仅限于当前函数体范围内，即该函数的大括号范围内。

## 3.2 简单变量

### 3.2.1 整型

整数就是没有小数部分的数字，如3、201、-3412和0。根据数值的大小不同，CAPL提供了以下几种整型。

![image-20240708154706863](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_15_47_6_202407081547911.png)

### 3.2.2 字符

区别于C语言，CAPL未将char类型（长度1B）归类至整型中，这是因为在CANoe中提供了byte类型。如果数据是具体数值则应使用byte，而对于字符，则应用char（字符串使用char数组）。char类型和byte类型之间可以直接转换。

``` c
byte data1 = 100;
char ch1 = 'd';
ch1 = 0x62;
data1 = 's';
```

### 3.2.3 浮点型

CAPL提供两种浮点型变量：float和double。

![image-20240708155019947](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_15_50_19_202407081550984.png)

## 3.3 复合类型

### 3.3.1 结构体

CAPL中可以简单地按照C语言的方法来声明结构（struct），但结构名在程序中必须是唯一的。简单类型、枚举类型或者其他的结构都可以作为结构的成员。  

```c
struct b{
    struct a a1;
    enum clolrs c1;
    int p;
    long l;
}
```

结构b中包含：一个结构成员a，变量名为a1；一个枚举成员colors，变量名为c1；一个整型变量p；最后还有一个长整型变量l。
用户可以在类型定义时直接声明结构类型的变量，在这种情况下，类型的名称可以省略，也可以直接使用结构的名字来引用类型。

```c
struct cost mycost;
struct {
    int chinese;
    int math;
    int english;
}scores;
```

关于结构初始化，可以在变量声明期间直接初始化结构成员，不需要单独命名单个成员，编译器将按照结构定义的顺序默认初始化它们。例如：  

```c
struct Scores myScores = {
    Chinese = 79,
    Math = 99,
    English = 88
};
struct Scores myScores = {79,99,88};
```

使 用 “.” 操 作 符 可 以 访 问 结 构 中 的 成 员 ， 例 如 ， myScores.Chinese ＝ 100。另外，结构体可以作为参数传给函数，但不能作为函数的返回值。  

### 3.3.2 枚举

枚举（enum）类型的声明也与C语言中的语法完全一致，但需要注意的是，枚举的成员名必须唯一，否则将有可能代替隐藏数据库中同名的报文和信号。如果没有在声明枚举的同时对成员进行赋值，编译器将按照成员声明的顺序对成员进行初始化。即第一个成员被初始化为0，往后依次加1。例如：

```c
enum {Apple, Pear, Banana} Fruit = Apple;
enum Colors {Red = 1, Green = 3, Blue = 9};
```

### 3.3.3 数组

数组（Array）作为一种基本的数据结构，也同样被CAPL支持，就像在C语言中一样。但为了方便使用，CAPL支持直接用字符串初始化字符数组的行为。

```c
int a[3] = {1,2,3};
char b[13] = "Hello World!";
```

同 样 ， CAPL 也 支 持 多 维 数 组 ， 并 且 可 以 通 过 内 建 函 数elCount（数组名）来获得数组成员的个数。如果数组的索引超出范围，即小于零或大于等于数组长度，CAPL将会在数组下标前提示错误。例如：  

```c
int v[3][3] = {{1,2,3},{4,5,6},{7,8,9}};
int a[3] = {1,2,3};
int b = elCount(a);
```

### 3.3.4 特殊类型

#### 3.3.4.1 报文

报文（CAN/LIN messages）是车载网络最基本的构成部分， CAPL提供了各种网络相对应的报文类。这里主要介绍CAN报文和LIN报文。

使用关键字message来声明一个报文变量，当使用message声明报文变量时，默认变量为CAN报文变量。当有数据库支撑的时候，一个 完 整 的 声 明 应 该 包 括 message ID 或 者 message name 。 结 合database的例子，使用ID 0xA或者报文名来声明一条数据库中的EngineData报文。例如

```c
message 0xA ml;
message 100 m2;
message EngineData m3;
```

以标识符“x”结尾的ID表示这是一个扩展帧ID，例如，100x。而“*”则表明这条报文在声明时还不含有CAN ID。例如：  

```c
on message CAN1.*
{
    message *msg;
    if(this.dir!= rx)
        return;
    msg = this;
}
```

切记，使用这种方式声明报文时，一定要指定ID后才能将msg发送出去。

CAPL提供了一系列的选择器（Selector）以保证用户能够按照自己的意图去修改CAN message的属性。

![image-20240708170256710](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_17_2_56_202407081702770.png)

例如，如果需要在CAN1网络上发送一条指定的报文，报文名： magicMessage；报文ID：0x252；包含8个字节0x03 3B 40 00 00 00 00 00；可以定义如下。  

```c
magicMessage.CAN = 1;
magicMessage.ID = 0x252;
magicMessage.DLC = 8;
magicMessage.Byte(0)=0x03;
magicMessage.Byte(1)=0x3B;
magicMessage.Byte(2)=0x40:
magicMessage.Byte(3)=0x00:
magicMessage.Byte(4)=0x00;
magicMessage.Byte(5)=0x00;
magicMessage.Byte(6)=0x00;
magicMessage.Byte(7)=0x00;
output (magicMessage);
```

#### 3.3.4.2 诊断报文

CAPL 通 过 诊 断 请 求 （ DiagRequest ） 和 诊 断 响 应（DiagResponse）这两种对象来实现跟ECU之间的诊断服务交互。通常情况下，诊断服务需要首先对Diagnostic对象声明和初始化。

```c
DiagRequest ServiceQualifier request;
DiagResponse ServiceQualifier response;
```

上述声明语句分别声明了诊断请求对象“request”和诊断响应对象“response”；并通过给出的诊断服务“ServiceQualifier”进行初始化。这种初始化将在节点仿真开始时被执行一次，并在每次诊断目标（DiagTarget）改变时被执行一次。
如果使用*来代替“ServiceQualifier”，诊断对象将被初始化为未添加诊断描述的空对象，但对象的数据必须在发送之前完成设置。

#### 3.3.4.3 系统变量

系统变量是一种特殊的变量，用来描述某种特殊状态（例如某种事件触发）或者记录测量数据。一般有系统定义和用户自定义两种，它们的作用域都是在各自的命令空间内。  

![image-20240708171438086](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_17_14_38_202407081714142.png)

#### 3.3.4.4 定时器

CAPL 提 供 了 两 种 定 时 器 变 量 ： timer 基 于 秒 的 时 间 单 位 ； msTimer基于毫秒的时间单位，例如：  

```c
msTimer myTimer;//声明一个毫秒定时器
Timer myTimer1;//声明一个秒定时器myTimer1
```

# 4 常见运算

与C语言一样，CAPL也提供了算术、逻辑和位运算的运算符，其用法也与C语言保持一致。

![image-20240708171805511](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/8_17_18_5_202407081718577.png)

# 5 流程控制

## 5.1 if条件语句

```c
if(表达式)
    语句;
```

```c
if(表达式)
    语句1;
else
    语句2;
```

## 5.2 switch句

```c
switch(表达式){
    case常量表达式1：
        语句1;
    case常量表达式2：
        语句2;
    case常量表达式n:
        语句n;
    default:
        语句n+1;
}
```

## 5.3 while循环语句

```c
whi1e(表达式)
    语句;
```

## 5.4 do-while循环

```c
do{
    循环体语句;
}while(表达式);
```

# 6 程序结构

## 6.1 头文件

CAPL提供了*.cin文件（callback interface file），用户可以通过该文件搭建自定义的测试框架。比如，将基本的函数接口按照不同类型分别定义在各自的*.cin文件中，然后再在不同的*.can文件中包含所需要的*.cin文件，从而形成二层引用结构。同时也可以在*.cin中包含其他的*.cin文件，然后在*.can文件中包含上层*.cin文件，进而形成多层的引用结构，从而达到提高代码复用效率的目的。  

例如，基本诊断服务定义在BaseServices.cin中，基本函数定义在CommonFunctions.cin中，测试用例函数定义在TestFunctions.cin中，将相关的测试用例定义在测试模块文件ECU_01.can中，那么在各个文件中的头文件结构如下。  

在TestFunctions.cin中：

```c
includes
{
    #include "CommonFunctions.cin"
    #include "Baseservices.cin"
}
```

在ECU_01.can中：  

```c
includes
{
    #include "TestFunctions.cin"
}
```

这样，在ECU_01.can中可以调用三个*.cin文件中的所有函数。如果需要编写另一个测试模块ECU_02，只需要在ECU_02.can中包含TestFunctions.cin即可。  

## 6.2 全局变量声明

CAPL跟C语言一样，变量的作用域和生命周期仅限于变量声明的函数体（即大括号范围）内。CAPL在每个程序的开始部分提供了variables区域给用户声明全局变量。  

```c
variables{
    int i 0;
    message 100 msg;
    msTimer myTimer;
    byte ECU_SERIAL_NUMBER[3]={0x31,0x32,0x33}:
}
```

在此部分声明的全局变量的生命周期从仿真开始持续到仿真结束，其作用域为整个CAPL文件。而在*.cin文件中声明的全局变量在包含它的*.can或*.cin中视为可见。  

## 6.3 事件处理

在什么条件下，在什么时间节点，发生了什么样的报文传递，得到了什么样的报文反馈。而这种面向事件的机制是通过event handler来实现的。  

### 6.3.1 事件起始关键字on

```
on *
{
	语句;
}
```

on后面加某种条件，一旦条件满足则执行下面函数体内的语句。函数体内的语句是实现接下来需要完成的操作。  

### 6.3.2 关键字this

在can的报文事件或变量事件中，可以使用关键字this访问数据内容

```
on message 100
{
	byte byte0;
	byte0 = this.byte(0);
}
```

这里的this就代表的是message这个报文

### 6.3.3 系统事件

系统事件主要用于处理CANoe测量系统的控制功能，主要有on start、on preStart、on stopMeasurement、on preStop、on key＜ newKey＞以及on timer。

![image-20240717171822648](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/17_17_18_22_202407171718726.png)

栗子：

```
on prestart{
	write("Measurement started!");
	msg_Count = 0;
}
on start{
	write("start Node A");
	setTimer(cycTimer,20);
	CallAll0nEnwar();
}
on prestop{
	message ShutdownReg m;
	output (m);
	Deferstop(1000);
}
on stopMeasurement{
	write("Message 0xx received:&d",msg.id,msg_Count);
}
```

### 6.3.4 CAN控制器事件

CAN控制器事件是对硬件接口设备中CAN控制器状态变化事件的响应  。

![image-20240717171806097](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/17_17_18_6_202407171718189.png)

栗子：下面的代码在侦测到Bus Off状态时，系统会输出信息到Write窗口，并复位ECU。  

```
on busoff{
	//在Bus off状态下复位CAN控制器
	Write("The CAN controller is in Bus off state");
	resetCanEx(Channel);
}
```

### 6.3.5 CAN报文事件

CAN报文事件在CAN总线上有指定的或任意报文出现时被调用。关键字为：on message xxx。例如，下面列出了不同的on message事件。  

```
on message 123	//对报文123(dec)反应
on message 0x123	//对报文123(hex)反应
on message MotorData	//对报文MotorData反应
on message CAN1.123	//对CAN通道1收到报文123反应
on message *	//对所有报文反应
on message 100-200	//对CAN ID在100~200间报文反应
```

### 6.3.6 CAN信号事件

CAN信号事件是在CAN总线上出现指定的信号时被调用（需要配合DBC文件使用）。关键字为：on signal xxx或on signal_update xxx。需要注意的是，前者只在指定信号的值发生变化时被调用，后者在每次接收到指定信号时均被调用。  

```
on signal Lightswitch:：Onoff{
	STAT1=this;
}
on signal_update Lightswitch::Onoff{
	STAT2=this;
}
```

### 6.3.7 定时事件

定时器变量可以用来创建一个定时事件，SetTimer函数用来设定时间间隔。当定时器运行到达设定的时间间隔时，将触发该事件，这时on timer函数中的程序块将被执行。需要提醒的是，周期性触发需要在每次触发结束后使用SetTimer复位。若在定时器运行中需要停止计时，可以使用cancelTimer函数来取消计时。

定时器事件关键字为on timer xxx。以下代码通过定时器事件实现每100ms发送一次报文0x555。

```
variables{
	message 0x555 msgl {dlc=1};
	msTimer myTimer;	//将myTimer声明ms为单位的定时器变量
}

on start{
	setTimer (myTimer,100);	//将定时值设定为100ms,并启动
}

on timer myTimer
{
	setTimer (myTimer,100);	//不能遗漏，复位定时器
	msg1.byte(0) = msg1.byte(0) + 1;	//更新报文byte(0)数据
	output (msg1);	//输出报文
}
```

### 6.3.8 键盘事件

在测量的过程中，通常需要由用户来触发某些事件来模拟实际测试环境的人工操作，例如，开始记录log、改变信号或变量的值、停止测量等。利用CAPL提供的键盘事件可以方便地完成这些操作。键盘事件的关键字为on key xxx。

```c
on key 'a'	//按a键反应
on key ' '	//按空格键反应
on key 0x20	//按空格键反应
on key F1	//按F1键反应
on key Ctrl-F12	//按Ctr1+F12组合键反应
on key PageUp	//按Page Up键反应
on key Home	//按Home键反应
on key	*	//按所有键反应
on key 's'
{
	Write("Logging starts");
}
```

### 6.3.9 错误帧事件

当总线上出现错误帧或者过载帧时，错误帧处理机制将被调用。下面的代码将输出总线错误码，同时将错误帧的信息输出到Write窗口中。  

```
on errorFrame
{
	const int buffersize 256;
	char buffer[buffersize];
	char cdirection[2][3]{"RX","TX"};
	int ndir;
	word ecc;
	word extInfo;
	int isProtocolException;
	
	ecc = (this.ErrorCode >> 6)&0x3f;
	extInfo = (this.ErrorCode >>12)&0x3;
	isProtocolException (this.ErrorCode &(1 <15))!=0;
	ndir extInfo ==0 extInfo ==2 0:1;
	//根据extInfo来判断错误帧传输方向：接收或发送
}

```



### 6.3.10 环境变量事件

环境变量事件是对环境变量发生变化的响应，关键字为on enVar xxx

```
on envVar Switch
{
	//声明一个CAN报文变量，用于传输
	message Controller msg;
	//读取环境变量Switch的数值，并赋值给信号Stop
	msg.Stop = getvalue(this);
	//发送报文到总线上
	output(msg);
}
```

可以使用getValue（）和putValue（）读写环境变量的值

```
//读取环境变量Switch的数值，并赋值给变量val
val = getvalue (Switch);
//将数值0赋值给环境变量Switch
putvalue(Switch,0);
```

#### 6.3.10.1 环境变量事件

与环境变量事件类似，系统变量事件是对系统变量发生变化的响应，关键字为==on sysVar xxx或on sysVar_update xxx==  

```
on sysvar IO::DI_0
{
	$Gateway:IOValuel = @this;
}
on sysvar update IO::DI_0
{
	SGateway::IOValue2 = @this;
}
```

#### 6.3.10.2 诊断事件

诊断事件是在诊断请求或诊断响应发生时产生  

![image-20240731172706361](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/31_17_27_6_202407311727576.png)

```
on diagRequest ECU.DefaultSession_start
{
	//发生诊断请求事件
	Write("Default Session Switch request received")
}
on diagRequestSent ECU.HardReset
{
	//诊断请求发送完成
	Write("HardReset service sent completely,ECU should reset")
}
on diagResponse ECU.VehicleIdentification_Number_Read
{
	//收到相应的诊断请求的响应
	Write("VehicleIdentification Number Read response received successfully")
}
```



#### 6.3.10.3 自定义函数



1. 当返回值类型省略时，被默认解释为void类型。
2. 正如C＋＋一样，允许函数包含一个空的形参列表。
3. 允许重载函数（即同一个函数名，但每个函数的形参列表必须不同，例如，不同的形参类型或者在形参列表中的不同次序）。
4. 函数会对实参进行类型检查，如果类型不同则检查是否能够通过隐式类型转换，如不能，则无法通过编译。
5. 任意维度或大小的数组都可被作为函数参数传递。  
6. 大部分CAPL支持的数据类型都可以直接声明为函数参数，例如，整型、浮点型、枚举、结构、定时器以及它们的引用。 但是有一些类型不能被直接声明，而需要加上*号（注意该符号并不是C语言中指针的意思）。例 如 ， signal * s 、 envvarInt * ev 、 sysvarFloat * sv 、diagRequest * dr、diagResponse * dr、int matrix［］［］，以及所有来自于database的变量，均需要加上 * 号声明。需要注意的是message类型比较特殊，如果该变量是用户自定义的，那么在函数参数声明时，message和message*均可以，但如果该变 量 来 自 database ， 那 么 只 有 message * 可 用 。

# 7 常用库函数

## 7.1 通用函数

![image-20240801165000502](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/1_16_50_1_202408011650093.png)

## 7.2 计算函数

![image-20240801165240597](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/1_16_52_40_202408011652823.png)



## 7.3 字符串函数

![image-20240801165320289](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/1_16_53_20_202408011653611.png)



## 7.4 CAN总线函数

![image-20240801165353798](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/1_16_53_54_202408011653031.png)

## 7.5 LIN总线函数

![](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/1_16_53_54_202408011653031.png)

## 7.6 诊断函数

![image-20240801165632020](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/1_16_56_32_202408011656257.png)

# 8 变量和信号的访问

## 8.1 访问信号

signal在CAPL中代表的是总线信号交互层的表示，它不同于message。message是CAPL的数据类型，而signal不是。因此，不能在CAPL中定义一个类型为signal的变量。

当用户需要访问信号缓冲区并期望读到最后接收到的信号值时，可以使用$符号。

```
value = $EngineSpeed;	//读取信号EngineSpeed的值
value = $EngineSpeed.raw;	//读取信号EngineSpeed的raw数据
$EngineSpeed = 500.0;	//将信号Enginespeed的值设为500
if ($Enginespeed !=500.0)write("Unequal!");
```

一个工程中可能包含多个相同名称的信号名。所以CAPL中需要增加通道（Channel）、网络（Network）、节点（Node）和报文（Message）的信息。完整的语法格式如下。  

```
(channel | network)::
					 [dbNode::]node::
					 					[dbMsg::]message::
					 										[dbsig::]
					 													signal
```

栗子：

```
$Lightswitch::Onoff	//Node + signal
$Lightswitch::Lightstate::Onoff	//Node + Message + Signal
$CAN1:Gateway:Status	//Channel + Node + signal
$PowerTrain::Gateway::Status	//Network + Node + Signal
$CAN1::Status	//Channel + Signal
```

## 8.2 访问系统变量

可以在CAPL中直接访问系统变量而不需要通过函数调用（例如使用SysGetVariableInt 、SysGetVariableFloat等获取变量值，使用SysSetVariableInt 、SysSetVariableFloat等函数修改变量值），以下是需要采取的语法格式：  

```
@Namespace:Variable
```



> [!CAUTION]
>
> 需要注意的是，对于array或struct类的变量，直接访问方式只能访问单个元素，
>
> ```
> intValue = @Namespacel::Parameter2;
> @Debug::MotorValues::EngineSpeed = $Enginespeed;
> intvalue=@Namespacel::ParameterArray[2];	//访问数组变量的单个成员
> @XCP::EcU_2::KL2.Curve2[0]=1.3;	//访问结构体中的数组变量的单个成员
> ```



比 较 通 用 的 访 问 操 作 方 式 是 使 用 以 sysGetVariable 开 头 和sysSetVariable开头的访问函数。  

```
//字符串修改操作
char demo[20] = {'M','u','s','i','c','','t','a','g'}:
sysSetvariablestring(sysvar::IPC::Music_tag,demo);

//long数组类的系统变量读取操作
long 1VarArr[10];
sysGetvariableLongArray(sysvar::MyNamespace::LongArrayvar,lVarArr,elcount(1VarArr));
```

## 8.3 访问环境变量

访问环境变量和系统变量的语法一样，都使用`@`符号进行访问。比较常用的访问函数为

```
//getValue相关操作
int val;
float fval;
char cBuf [25];
byte bBuf[64];
long copiedBytes;

//获取环境变量Switch(整型)的数值并赋值给整数val变量
val = getvalue(Switch);
//获取环境变量Temperature(浮点型)的数值并赋值给浮点数fval变量
fval = getvalue(Temperature);
//读取环境变量NodeName(字符串型)的数值，赋值给字符串cBuf,返回复制的字符长度
copiedBytes = getvalue(NodeName,cBuf);
//读取环境变量DiagData(字符串型)的数值，从32位置赋值给字符串bBuf,返回复制的字符长度
copiedBytes = getvalue(DiagData,bBuf,32);

//putValue相关操作
byte dataBuf [64];

//将0赋值给环境变量Switch
putvalue(Switch,0);
//将22.5赋值给环境变量Temperature
putvalue(Temperature,22.5);
//将"Master"赋值给环境变量NodeName
putvalue (NodeName,"Master");
//复制dataBuf字符串变量中的64个字节给环境变量DiagData
putvalue(DiagData,dataBuf,64);
```

















