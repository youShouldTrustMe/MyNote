# 数据通信基础

## 串行/并行通信

![image-20240709114945281](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_49_45_202407091149342.png)

## 单工/半双工/全双工通信

![image-20240709115029566](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_50_29_202407091150652.png)

## 同步/异步通信

![image-20240709115047069](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_50_47_202407091150120.png)

## 波特率

![image-20240709115101824](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_51_1_202407091151890.png)

## 常见通信接口

| **名称** | **引脚**             | **双工** | **时钟** | **电平** | **设备** |
| -------- | -------------------- | -------- | -------- | -------- | -------- |
| USART    | TX、RX               | 全双工   | 异步     | 单端     | 点对点   |
| I2C      | SCL、SDA             | 半双工   | 同步     | 单端     | 多设备   |
| SPI      | SCLK、MOSI、MISO、CS | 全双工   | 同步     | 单端     | 多设备   |
| CAN      | CAN_H、CAN_L         | 半双工   | 异步     | 差分     | 多设备   |
| USB      | DP、DM               | 半双工   | 异步     | 差分     | 点对点   |

# 串口

## 概念

![image-20240709115152751](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_51_52_202407091151823.png)

## RS232 VS CMOS/TTL

![image-20240709115220783](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_52_20_202407091152830.png)

## 串口参数及时序

![image-20240709115313290](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_53_13_202407091153363.png)

## RS232通信

![image-20240709115335144](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_53_35_202407091153213.png)

![image-20240709115341646](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_53_41_202407091153702.png)

## USB通信

![image-20240709115405408](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_54_5_202407091154473.png)

## 电平的区别

![image-20240709115420564](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_54_20_202407091154628.png)

# USART

## 简介

![image-20240709115448902](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_54_48_202407091154957.png)

## 主要特征

![image-20240709115506529](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_55_6_202407091155579.png)

## 框图

串口数据低位先行

![image-20240709115802260](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_58_2_202407091158342.png)

## 框图简化版

![image-20240709115831073](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_58_31_202407091158129.png)

![image-20240709115838130](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_58_38_202407091158201.png)

## 数据帧

这里的字长就是数据位长度，空闲帧和断开帧是局域网协议用的，我们的串口用不到

![image-20240709115857318](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_58_57_202407091158383.png)

停止位分别为1、1.5、2、0.5，意思就是0.5个时长

![image-20240709115911366](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_59_11_202407091159454.png)

要保证在采样的时候，数据在时钟边沿的中间，这样可以保证数据的可靠性

- 当输入电路侦测到一个数据帧的起始位后，就会以波特率的频率，连续采样一帧数据，同时，从起始位置开始，采样位置就要对齐到位的正中间

- 首先数据的部分电路对采样的时钟进行了细分，他会以波特率的16倍进行采样，也就是说，在一位的时间内，可以进行16次采样，采样策略为
  - 最开始，空闲状态高电平，采样一直为1，在某个位置突然采样采到一个0，就说明出现了下降沿，这个时候在理想情况下应该就要开始采样了，在起始位，会进行16次连续采样，没有噪声的话，那么这16次采样应该都为0，因为串口协议的起始就是检测到下降沿，实际电路还是会出现一点噪声的，所以即使出现下降沿，后续也要再采样几次，以防万一，

![image-20240709115949779](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_11_59_49_202407091159855.png)

## 设置波特率

波特率的产生实际上就是一个预分频的过程，这里的16就是上面的内部还有一个16倍的采样时钟

![image-20240709120020816](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_12_0_20_202407091200878.png)

![image-20240709120027194](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_12_0_27_202407091200261.png)

![image-20240709120040728](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_12_0_40_202407091200771.png)

![](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_12_0_40_202407091200771.png)

## 数据模式

![image-20240709132602954](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_13_26_3_202407091326050.png)

# 实验

## 简单的发送数据

```c

void serial_init()
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_USART2,ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Pin = SERIAL_TX_PIN;
	GPIO_InitStruct.GPIO_Speed =GPIO_Speed_50MHz;
	GPIO_Init(SERIAL_PORT,&GPIO_InitStruct);
	
	USART_InitTypeDef USART_InitStruct;
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStruct.USART_Mode = USART_Mode_Tx;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	USART_Init(USART2,&USART_InitStruct);
	
	USART_Cmd(USART2,ENABLE);
}

void serial_sendByte(u8 byte)
{
	USART_SendData(USART2,byte);
	while(USART_GetFlagStatus(USART2,USART_FLAG_TXE) == RESET);
}

void serial_sendArray(u8 *array,u16 length)
{
	for(u16 i = 0; i < length; i++){
		serial_sendByte(array[i]);
	}
}

void serial_sendString(char *str)
{
	
	for(u8 i = 0;str[i] != '\0';i++){
		serial_sendByte(str[i]);
	}
}

u32 serial_pow(u32 x, u32 y)
{
	u32 result = 1;
	while(y--){
		result *= x;
	}
	return result;
}

//将数字以字符的样式发送
void serial_sendNum(u32 number, u8 length)
{
	for(u8 i = 0; i < length; i++){
		serial_sendByte(number / serial_pow(10,length - i -1) % 10 + '0');
	}
}

int main(void){
	
	char str[50];
	u8 test[] = {0x42,0x43,0x44,0x45};

	serial_init(); 
	serial_sendArray(test,4);
	serial_sendNum(12345,5);

	while(1){
//                sprintf(str,"%x: %hu %hu\n",(u32)adValue,adValue[0],adValue[1]);
//                send_string(str);
	}
}
	

```

## printf重定向

如果想要使用printf函数，需要先使用microlib库

![image-20240709132746805](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_13_27_46_202407091327860.png)

注意需要引入stdio头文件，还需要重写fputc

```c

//重写这个函数用于重定向printf函数，这个函数是printf的底层函数，printf在打印的时候就是不断调用这个函数来一个一个打印的
int fputc(int ch, FILE *file)
{
	serial_sendByte(ch);
	return ch;
}

```

重写完这个函数就可以正常使用printf函数了

如果想要让printf使用多个串口，那么就需要sprintf函数

```c

sprintf(str,"%x: %hu %hu\n",(u32)adValue,adValue[0],adValue[1]);
serial_sendString(str);

```

为了方便，我们也可以封装sprintf函数，在封装sprintf的时候需要引入头文件stdarg.h头文件。

```c

void serial_printf(char *format, ...)
{
	char string[100];
	va_list arg;
	va_start(arg,format);
	vsprintf(string,format,arg);
	va_end(arg);
	serial_sendString(string);
}

```

解决汉字乱码的情况，如果使用的编码格式为utf-8，则需要先设置编译参数--no-multibyte-chars

![image-20240709133243542](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_13_32_43_202407091332598.png)

## 接收数据

需要修改初始化函数，分为查询式和中断式

```c
//查询式
void serial_init()
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_USART2,ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Pin = SERIAL_TX_PIN;
	GPIO_InitStruct.GPIO_Speed =GPIO_Speed_50MHz;
	GPIO_Init(SERIAL_PORT,&GPIO_InitStruct);
	
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStruct.GPIO_Pin = SERIAL_RX_PIN;
	GPIO_InitStruct.GPIO_Speed =GPIO_Speed_50MHz;
	GPIO_Init(SERIAL_PORT,&GPIO_InitStruct);
	
	USART_InitTypeDef USART_InitStruct;
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStruct.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	USART_Init(USART2,&USART_InitStruct);
	
	USART_Cmd(USART2,ENABLE);
}


int main(void){
	u8 rxData;

	serial_init(); 

	while(1){
		if(USART_GetFlagStatus(USART2,USART_FLAG_RXNE) == SET){
			rxData = USART_ReceiveData(USART2);
			serial_printf("receive: %x\n",rxData);
		}
	}
}


//中断式

void serial_init()
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_USART2,ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Pin = SERIAL_TX_PIN;
	GPIO_InitStruct.GPIO_Speed =GPIO_Speed_50MHz;
	GPIO_Init(SERIAL_PORT,&GPIO_InitStruct);
	
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStruct.GPIO_Pin = SERIAL_RX_PIN;
	GPIO_InitStruct.GPIO_Speed =GPIO_Speed_50MHz;
	GPIO_Init(SERIAL_PORT,&GPIO_InitStruct);
	
	USART_InitTypeDef USART_InitStruct;
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStruct.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	USART_Init(USART2,&USART_InitStruct);
	
	USART_ITConfig(USART2,USART_IT_RXNE,ENABLE);
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	
	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_InitStruct.NVIC_IRQChannel = USART2_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 1;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 1;
	NVIC_Init(&NVIC_InitStruct);
	
	USART_Cmd(USART2,ENABLE);
}


u8 serial_rxData,serial_rxFlag;
void USART2_IRQHandler(void)
{
	if(USART_GetFlagStatus(USART2,USART_IT_RXNE) == SET){
		serial_rxData = USART_ReceiveData(USART2);
		serial_rxFlag = 1;
		USART_ClearITPendingBit(USART2,USART_IT_RXNE);
		}
}

u8 serial_getRxFlag(void)
{
	if(serial_rxFlag == 1){
		serial_rxFlag = 0;
		return 1;
	}
	return 0;
}

u8 serial_getRxData(void)
{
	return serial_rxData;
}

int main(void){
	u8 rxData;

	serial_init(); 

	while(1){
		if(serial_getRxFlag() == 1){
			rxData = serial_getRxData();
			serial_printf("receive %x\n",rxData);
		}
	}
}

```

## 数据包的收发

### HEX数据包

![image-20240709133421260](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_13_34_21_202407091334326.png)

![image-20240709133427176](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_13_34_27_202407091334247.png)

### 文本数据包

![image-20240709133444550](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_13_34_44_202407091334615.png)

![image-20240709133457027](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_13_34_57_202407091334097.png)

### 实验代码

```c

void serial_sendPacket(void)
{
	serial_sendByte(0xFF);
	serial_sendArray(serial_txPacket,4);
		serial_sendByte(0xFE);

}

u8 serial_rxData,serial_rxFlag;
u8 serial_txPacket[4],serial_rxPacket[4];
void USART2_IRQHandler(void)
{
	static u8 rxState = 0;
	static u8 counter = 0;
	if(USART_GetFlagStatus(USART2,USART_IT_RXNE) == SET){
		u8 rxData = USART_ReceiveData(USART2);
		if(rxState == 0){
			if(rxData == 0xff){
				rxState = 1;
				counter = 0;
			}
		}else if(rxState == 1){
			serial_rxPacket[counter++] = rxData;
			if(counter >= 4){
				rxState  = 2;
			}
		}else if(rxState == 2){
			if(rxData == 0xfe){
				rxState = 0;
				serial_rxFlag = 1; 
			}
		}
	}
}

```

