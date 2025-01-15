# 入门

## 裸机与RTOS

![裸机和RTOS](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/15_9_19_10_202408150919859.png)

![任务调度](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/15_9_20_31_202408150920114.png)

![image-20240815092108670](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/15_9_21_8_202408150921723.png)

![image-20240815092116112](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/15_9_21_16_202408150921159.png)

##  裸机和RTOS的特点

![裸机的特点](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/15_9_21_41_202408150921614.png)

![RTOS的特点](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/15_9_21_55_202408150921924.png)

##  基础知识

### 堆

堆就是一块空闲的内存，我们可以管理内存，我们可以从内存中取出一些数据，用完之后再释放（放回）

```c
char heap_buf[1024];        //空闲的内存，只要在上面实现内存的分配和释放，那么他就是一个堆
int pos = 0;        //定义一个位置指针

void *my_malloc(int size)        //分配内存
{
    int old_pos = pos;
    pos += size;
    return &heap_buf[pos];
}

void my_free(void *buf)
{
    //暂时无法实现
}

int main(void)
{
    char ch = 65;
    int i;
    char *buf = my_malloc(100);
    for(i = 0; i < 26; i++){
        buf[i] = 'A' + i;
    }
}
```

###  栈

栈是一块空闲的内存

==一个c函数的开头将会怎么处理栈？==

- 划分栈（用来保存LR等寄存器以及局部变量）
- LR寄存器等存入栈
- 执行代码

```c
int b_func(void)
{

}

int c_func(void)
{

}


int a_func(int val)
{
	int a = 8;
	a += val;
	b_func();
	c_func();
}

int main(void)
{
	a_func(46);        //当该函数执行完，需要返回一个下一个程序的执行地址，也就是下面的return0语句的地址
	return 0;
}
```

==一个函数执行完之后返回的地址保存在哪？==

main如何调用a_func

1. 首先将函数地址保存在某个寄存器中（LR link register），main函数在执行到a_func之前会将return 0 这个语句的地址保存在LR中，之后调用函数a_func
2. a_func掉用b_func前也需要将下一条语句的地址保存到LR中，之后调用b_func
3. 此时LR中的值就会被覆盖，解决方案是
   1. 在a_func内部，将LR中的值压入栈中
   2. 在b_func内部，将LR中的值压入栈中


函数的运行过程：

1. Main->a_func->b_func->c_func
2. 执行main，此时main函数自动划分出一个N字节的栈，将main的返回地址（即下一条语句地址）保存到LR中，此栈中保存main的LR等寄存器及局部变量，之后执行main函数
3. 调用a_func，此时a_func自动划分出一个M字节的栈，LR中保存a_func的返回地址，局部变量a也存放在此栈中
4. 调用b_func，此时b_func也自动划分出一个P字节的栈，LR中保存b_func的返回地址，此栈中保存b_func的LR寄存器及局部变量
5. 调用c_func，此时b_func也自动划分出一个R字节的栈，LR中保存c_func的返回地址，此栈中保存c_func的LR寄存器及局部变量，当执行完函数c的时候，将会取出自己的栈中的LR数据（返回地址），自动跳转到LR指向的地址运行

### 数据结构

####  数据类型

每个移植的版本都含有自己的portmacro.h头文件，里面定义了2个数据类型：

- TickType_t:
  - FreeRTOS配置了一个周期性的时钟中断：Tick Interrupt
  - 每发生一次中断，中断次数累加，这被称为tick count
  - tick count这个变量的类型就是TickType_t
  - TickType_t可以是16位的，也可以是32位的
  - FreeRTOSConfig.h中定义configUSE_16_BIT_TICKS时，TickType_t就是uint16_t
  - 否则TickType_t就是uint32_t
  - 对于32位架构，建议把TickType_t配置为uint32t
- BaseType_t:
  - 这是该架构最高效的数据类型
  - 32位架构中，它就是uint32t
  - 16位架构中，它就是uint16t
  - 8位架构中，它就是uint8t
  - BaseType_t通常用作简单的返回值的类型，还有逻辑值，比如pdTRUE/pdFALSE

#### 变量名

| 变量名前缀 | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| c          | char                                                         |
| s          | int16_t,short                                                |
| l          | int32_t,long                                                 |
| x          | BaseType_t,其他非标准的类型：结构体、task handle、queue handle等 |
| u          | unsigned                                                     |
| p          | 指针                                                         |
| uc         | uint8_t,unsigned char                                        |
| pc         | char指针                                                     |

#### 函数名

函数名的前缀有两个部分`返回类型 + 文件位置`

| 函数名前缀             | 含义                                |
| ----------------- | --------------------------------- |
| VTaskPrioritySet  | 返回值类型：void，在task.c中定义             |
| xQueueReceive     | 返回值类型：BaseType_t，在queue.c中定义      |
| pvTimerGetTimerID | 返回值类型：pointer to void，在timer.c中定义 |
## 任务调度

调度器就是使用相关的调度算法来决定当前需要执行的哪个任务，FreeRTOS一共支持三种任务调度方式：

1. *抢占式`调度`*：主要是针对优先级不同的任务，每个任务都有一个优先级，优先级高的任务可以抢占优先级低的任务。
2. *时间片`调度`*：主要针对优先级相同的任务，当多个任务的优先级相同时，任务调度器会在每一次系统时钟节拍到的时候切换任务。
3. *协程式`调度`*：当前执行任务将会一直运行，同时高优先级的任务不会抢占低优先级任务FreeRTOS:现在虽然还支持，但是官方已经表示不再更新协办程式调度

### 抢占式调度

运行条件：

1. 创建三个任务：Task1、Task2、Task3
2. Task1、Task2、Task3的优先级分别为1、2、3；在FreeRTOS中任务设置的数值越大，优先级越高，所以TASK3的优先级最高。

![抢占式调度](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/15_13_29_59_202408151329026.png)

运行过程如下：

1. 首先Task1在运行中，在这个过程中Task2就结了，在抢占式调度器的作用下Task2会抢占Task1的运行
2. Task2运行过程中，Task3就结了，在抢占式调度器的作用下Task3会抢占Task2的运行
3. Task3运行过程中，Task3阻塞了（系统延时或等待信号量等），此时就绪态中，优先级最高的任务Task2执行
4. Task3阻塞解除了（延时到了或者接收到信号量），此时Task3恢复到结态中，抢占TsK2的运行

> [!IMPORTANT]
>
> 高优先级任务，优先执行

### 时间片调度

运行条件：

1. 创建三个任务：Task1、Task2、Task3
2. Task1、Task2、Task3的优先级均为1；即3个任务同等优先级
   1. 同等优先级任务，轮流执行；时间片流转
   2. 一个时间片大小，取决为商答定时器中断周期
   3. 注意没有用完的时间片不会再使用，下次任务Tsk3得到执行还是按照一个时间片的时钟节拍运行

![时间片调度](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/15_13_34_20_202408151334963.png)

运行过程如下：

1. 首先Task1运行完一个时间片后，切换至Task2运行
2. Task2运行完一个时间片后，切换至Task3运行
3. Task3运行过程中（还不到一个时间片），Task3阻塞了（系统延时或等待信号量等），此时直接切换到下一个任务Tsk1
4. Task1运行完一个时间片后，切换至Task2运行

## 任务状态

RTOS中有四种运行状态：

1. 运行态：正在执行的任务，该任务就处于运行态，注意在STM32中，同一时间仅一个任务处于运行态
2. 就绪态：如果该任务已经能的够被执行，但当前还未被执行，那么该任务处于就绪态
3. 阻塞态：如果一个任务因延时或等待外部事件发生，那么这个任务就处于阻塞态
4. 挂起态:类似暂停，调用函数VTaskSuspend0进入挂起态，需要调用解挂函数VTask Resume()

任务切换基础：==tick中断==

时钟的滴答是周期性的，所以tick中断就是定时器的定时中断，每1ms（可以配置）产生一次中断，这里间隔的1ms就称为tick，同时将会记录发生中断的次数，这个计数值称为tick count，这是RTOS的==时钟基准==。

我们可以在FREERTOSConfig.h文件中配置定时器的中断时间，单位为ms

![配置文件](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/25_14_25_45_202409251425748.png)

四种任务状态之间的转换图:

![任务状态之间的转换图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/25_14_24_26_202409251424761.png)

> [!NOTE]
>
> 仅就绪态可转变成运行态
>
> 其他状态的任务想运行，必须先转变成就绪态

FreeRTOS中无非就四种状态，运行态，就绪态、阻塞态、挂起态

这四种状态中，除了运行态，其他三种任务状态的任务都有其对应如的任务状态列表

![任务状态列表](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/25_14_27_30_202409251427568.png)

32位的变量，当某个位置一时，代表所对应的优先级就绪列表有任务存在

![列表举例](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/25_14_28_29_202409251428020.png)

*==实验==*

```c

void Task1Function( void * param)
{
	TickType_t tickStart = xTaskGetTickCount();        //获得程序运行开始的tick count        
	TickType_t tickNow;        //获得当前的tick count        
	int flag = 0;
	while (1)
	{
		tickNow = xTaskGetTickCount();        //获得当前的tick count
		task1flagrun = 1; 
		task2flagrun = 0; 
		task3flagrun = 0; 
		printf("1");
		if(!flag && (tickNow > tickStart + 10)){
			vTaskSuspend(xHandeTask3);        //将task3变为挂起态，参数为NULL的时候，使自己为挂起态
			flag = 1;
		}
		if(tickNow > tickStart + 20){
			vTaskResume(xHandeTask3);        //将task3从挂起态唤醒，一旦挂起之后，只能通过别的任务进行唤醒
		}
	}
	
}

void Task2Function( void * param)
{
	while (1)
	{
		task1flagrun = 0; 
		task2flagrun = 1; 
		task3flagrun = 0; 
		printf("2");
		vTaskDelay(10);        //进入阻塞状态，这里的单位为tick
	}
	
}

void Task3Function( void * param)
{
	while (1)
	{
		task1flagrun = 0; 
		task2flagrun = 0; 
		task3flagrun = 1; 
		printf("3");
	}
}
```

# 移植系统

首先从[FreeRTOS/FreeRTOS: 'Classic' FreeRTOS distribution. Started as Git clone of FreeRTOS SourceForge SVN repo. Submodules the kernel. (github.com)](https://github.com/FreeRTOS/FreeRTOS)上下载文件。文件树为：

![freeRTOS文件树](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_8_28_26_202412230828226.png)



我们需要使用的源文件位于`FreeRTOS`中。

![FreeRTOS文件树](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_8_30_15_202412230830421.png)

1. Demo文件夹是官方给的例程，看官方给的例程可以看工程配置。

   ![Demo文件夹](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_8_35_10_202412230835029.png)

2. Source文件夹是FreeRTOS的核心源码

   ![source文件夹](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_8_34_12_202412230834860.png)

3. portable文件夹中是和硬件相关的架构代码，一般我们需要保留的是KEIL和MemMang，但是KEIL文件夹中显示和RVDS文件夹一样，所以我们还要保留RVDS文件夹

   ![image-20241223083421261](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_8_34_21_202412230834425.png)

移植步骤：

1. 添加FreeRTOS源码：将FreeRTOS源码添加至基础工程、头文件路径等
2. 添加配置文件：可以在Demo工程中复制一个FreeRTOSConfig.h出来，增加内存管理文件heap.c
3. 修改SYSTEM文件：修改SYSTEM文件中的sys.c、delay.c、usartc
4. 修改中断相关文件：修改Systick中断SVC中断、PendSV中断
5. 添加应用程序：验证移植是否成功



# 任务

## 任务的创建和删除

任务的创建和删除实际上就是调用FreeRTOS的API函数。

| API函数           | 描述             | 细节                                                         |
| ----------------- | ---------------- | ------------------------------------------------------------ |
| xTaskCreate       | 动态方式创建任务 | 任务的任务控制块以及任务的栈空间所需的内存，均由FreeRTOS从FreeRTOS管理的堆中分配 |
| xTaskCreateStatic | 静态方式创建任务 | 任务的任务控制块以及任务的栈空间所需的内存，需用户分配提供   |
| vTaskDelete       | 删除任务         |                                                              |

任务控制块的结构体类型为

```c
typedef struct tskTaskControlBlock
{
    volatile StackType_t 	*pxTopOfStack;							/*任务栈栈顶，必须为TCB的第一个成员*/
    Listltem_t				xStateListItem;							/*任务状态列表项*/
    Listltem_t  			xEventListItem;    						/*任务事件列表项*/
    UBaseType_t 			uxPriority;    							/*任务优先级，数值越大，优先级越大*/
    StackType_t				*pxStack;    							/*任务栈起始地址*/
    char 					pcTaskName[configMAX_TASK_NAME_LEN];	/*任务名字*/
    /*省略很多条件编译的成员*/
}tskTCB;
```

> [!tip]
>
> 任务栈栈顶，在任务切换时的任务上下文保存、任务恢复息息相关。
>
> 注意：每个任务都有属于自己的任务控制块，类似身份证



### 动态创建任务

函数API为:

```c
BaseType_t xTaskCreate
(
    TaskFunction_t					pxTaskCode,		/*指向任务函数的指针*/
    const char *const				pcName,			/*任务名字，最大长度configMAX_TASK_NAME_LEN*/
    const configSTACK_DEPTH_TYPE	usStackDepth,	/*任务堆栈大小，注意字为单位*/
    void* const						pvParameters,	/*传递给任务函数的参数*/
    UBaseType_t 					uxPriority,		/*任务优先级，范围：0~configMAX_PRIORITIES-1*/
    TaskHandle_t* const				pxCreatedTask	/*任务句柄，就是任务的任务控刮块*/
)
```

| 返回值                                | 描述         |
| ------------------------------------- | ------------ |
| pdPASS                                | 任务创建成功 |
| errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY | 任务创建失败 |

创建动态任务流程为：

1. 将宏configSUPPORT_DYNAMIC_ALLOCATION配置为1
2. 定义函数入口参数
3. 编写任务函数

此函数创建的任务会立刻进入就绪态，由任务调度器调度运行，动态创建任务函数内部实现为：

1. 申请堆栈内存&任务控制块内存
2. TCB结构体成员赋值
3. 添加新任务到就绪列表中

```c
void TaskGenericFunction( void * param)
{
    int value = (int)param;

    while (1)
    {
        printf("%d",value);
    }

}

int main(void)
{
    #ifdef DEBUG
    debug();
    #endif

    //初始化硬件
    prvSetupHardware();
    printf("hello word\r\n");

    //创建一个任务,RTOS的核心就是多任务
    xTaskCreate(Task1Function,"Task 1",100,NULL,1,&xHandeTask1);        //参数列表：处理任务的函数名，任务名称，栈深度，参数列表,优先级，句柄
    xTaskCreate(Task2Function,"Task 2",100,NULL,1,NULL);
    xTaskCreateStatic(Task3Function,"Task 3",100,NULL,1,xTask3Stack,&xTask3TCB);
    xTaskCreate(TaskGenericFunction,"Task 4",100,(void *)4,1,NULL);
    xTaskCreate(TaskGenericFunction,"Task 5",100,(void *)5,1,NULL);
    //进行任务调度
    vTaskStartScheduler();

    return 0;
}
```



### 静态创建任务

函数API为：



```c
TaskHandle_t xTaskCreateStatic
(
    TaskFunction_t    			pxTaskCode,		/*指向任务函数的指针*/
    const char 		const  		pcName,			/*任务函数名*/
    const uint32_t    			ulStackDepth,	/*任务堆栈大小注意字为单位*/
    void			*const    	pyParam eters,	/*传递的任务函数参数*/
    UBaseType_t 				uxPriority,		/*任务优先级*/
    StackType_t		*const    	puxStackBuffer,	/*任务堆栈，一般为数组，由用户分配*/
    StaticTask_t	*const    	pxTaskBuffer    /*任务控制块指针，由用户分配*/
)
```

| 返回值 | 描述                                 |
| ------ | ------------------------------------ |
| NUL    | 用户没有提供相应的内存，任务创建失败 |
| 其他值 | 任务句柄，任务创建成功               |

函数的创建流程为：

1. 需将宏configSUPPORT_STATIC_ALLOCATION配置为1

2. 定义空闲任务&定时器任务的任务堆栈及TCB

3. 实现两个接口函数:

   vApplicationGetTimerTaskMemory ()

   vApplicationGetldleTaskMemory()

4. 定义函数入口参数

5. 编写任务函数

此函数创建的任务会立刻进入就绪态，由任务调度器调度运行，内部实现为：

1. TCB结构体成员赋值
2. 添加新任务到惊就绪列表中

```c
//当我们使用静态分配栈和TCB结构时必须要实现该函数
StackType_t xIdleTaskStack[100];
StaticTask_t xIdleTaskTCB;

void vApplicationGetIdleTaskMemory( StaticTask_t ** ppxIdleTaskTCBBuffer,StackType_t** ppxIdleTaskStackBuffer,uint32_t * pulIdleTaskStackSize )
{
    *ppxIdleTaskTCBBuffer = &xIdleTaskTCB;
    *ppxIdleTaskStackBuffer = xIdleTaskStack;
    *pulIdleTaskStackSize = 100;
}

void Task3Function( void * param)
{
    while (1)
    {
        printf("3");
    }

}

int main(void)
{
    StackType_t xTask3Stack[100];
    StaticTask_t xTask3TCB;
    xTaskCreateStatic(Task3Function,"Task 3",100,NULL,1,xTask3Stack,&xTask3TCB);
	return 0;
}
```



### 删除任务

函数API为：

```c
void vTaskDelete (
    TaskHandle_t xTaskToDelete	/*待删除任务的任务句柄*/
);
```

> [!note]
>
> 用于删除已被创建的任务，被删除的任务将从就绪态任务列表、阻塞态任务列表、挂起态任务列表和事件列表中移除
>
> 1. 当传入的参数为NUL,则代表删除任务自身（当前正在运行的任务）
> 2. 空闲任务会负责释放被删除任务中由系统分配的内存，但是由用户在任务删除前申请的内存，则需要由用户在任务被删除前提前释放，否则将导致内存泄露

函数的删除流程为：

1. 使用删除任务函数，需将宏INCLUDE_vTaskDelete配置为1
2. 入口参数输入需要删除的任务句柄(NULL代表删除本身)

内部实现过程为：

1. 获取所要删除任务的控制块：通过传入的任务句柄，判断所需要删除哪个任务，NULL代表删除自身
2. 将被删除任务，移除所在列表：将该任务在所在列表中移除，包括：就绪、阻塞、挂起、事件等列表
3. 判断所需要删除的任务
   1. 删除任务自身，需先添加到等待删除列表，内存释放将在空闲任务执行
   2. 删除其他任务，释放内存，任务数量--
4. 
   更新下个任务的阻塞时间：更新下一个任务的阻塞超时时间，以防被删除的任务就是下一个阻塞超时的任务

```c
TaskHandle_t xHandeTask1;
int task1flagrun  = 0;        //用来表示任务1是否在运行
int task2flagrun  = 0;
int task3flagrun  = 0;

void Task1Function( void * param)
{
    while (1)
    {
        task1flagrun = 1; 
        task2flagrun = 0; 
        task3flagrun = 0; 
        printf("1");
    }

}

void Task2Function( void * param)
{
    int i = 0;
    while (1)
    {
        task1flagrun = 0; 
        task2flagrun = 1; 
        task3flagrun = 0; 
        printf("2");
        if(i++ == 100){
            vTaskDelete(xHandeTask1);        //杀死任务1，只需要操作任务1的handle即可
        }
        if(i == 200){
            vTaskDelete(NULL);        //参数使用NULL即可实现自杀
        }
    }

}

void Task3Function( void * param)
{
    while (1)
    {
        task1flagrun = 0; 
        task2flagrun = 0; 
        task3flagrun = 1; 
        printf("3");
    }

}


StackType_t xIdleTaskStack[100];
StaticTask_t xIdleTaskTCB;
void vApplicationGetIdleTaskMemory( StaticTask_t ** ppxIdleTaskTCBBuffer,StackType_t ** ppxIdleTaskStackBuffer,uint32_t * pulIdleTaskStackSize )
{
    *ppxIdleTaskTCBBuffer = &xIdleTaskTCB;
    *ppxIdleTaskStackBuffer = xIdleTaskStack;
    *pulIdleTaskStackSize = 100;
}

StackType_t xTask3Stack[100];
StaticTask_t xTask3TCB;
int main( void )
{
    #ifdef DEBUG
    debug();
    #endif

    //初始化硬件
    prvSetupHardware();
    printf("hello word\r\n");

    //创建一个任务,RTOS的核心就是多任务
    xTaskCreate(Task1Function,"Task 1",100,NULL,1,&xHandeTask1);        //参数列表：处理任务的函数名，任务名称，栈深度，参数列表,优先级，句柄
    xTaskCreate(Task2Function,"Task 2",100,NULL,1,NULL);

    xTaskCreateStatic(Task3Function,"Task 3",100,NULL,1,xTask3Stack,&xTask3TCB);        

    //进行任务调度
    vTaskStartScheduler();

    return 0;
}
```



## 任务的挂起和恢复

函数API：

| API函数              | 描述                     |
| -------------------- | ------------------------ |
| vTaskSuspend()       | 挂起任务                 |
| vTaskResume()        | 恢复被挂起的任务         |
| xTaskResumeFromISR() | 在中断中恢复被挂起的任务 |

- 挂起:挂起任务类似暂停，可恢复；删除任务，无法恢复，类似“人死两清
- 恢复:恢复被挂起的任务
- “FromlSR":带FromISR后缀是在中断函数中专用的API函数

### 任务的挂起

```c
void vTaskSuspend(
    TaskHandle_t xTaskToSuspend	/*待挂起任务的任务句柄*/
)
```

> [!note]
>
> 此函数用于挂起任务，使用时需将宏INCLUDE_vTaskSuspend配置为1。无论优先级如何，被挂起的任务都将不再被执行，直到任务被恢复。
>
> 当传入的参数为NULL,则代表挂起任务自身（当前正在运行的任务）

### 任务的恢复

#### 任务中恢复

```c
void vTaskSuspend(
    TaskHandle_t xTaskToSuspend	/*待挂起任务的任务句柄*/
)
```

> [!note]
>
> 使用该函数注意宏：INCLUDE_vTaskSuspend必须定义为1
>
> 注意：任务无论被VTaskSuspend()挂起多少次，只需在任务中调用vTaskToResume()恢复一次，就可以继续运行。且被恢复的任务会进入就绪态！

内部实现：

1. 获取所要挂起任务的控制块：通过传入的任务句柄，判断所需要挂起哪个任务，NULL代表挂起自身
2. 移除所在列表：将要挂起的任务从相应的状态列表和事件列表中移除（就绪或阻塞列表）
3. 插入挂起任务列表：将待挂起任务的任务状态列表向插入到挂起态任务列表末尾
4. 判断任务调度器是否运行：在运行，更新下一次阻塞时间，防止被挂起任务为下一次阻塞超时任务
5. 判断待挂起任务是否为当前任务：如果挂起的是任务自身，且调度器正在运行，需要进行一次任务切换，调度器没有运行，判挂起任务数是否等于任务总数，是：当前控利块赋值为NULL，否：寻找下一个最高优先级任务

#### 中断中恢复：

```c
BaseType_t xTaskResumeFromlSR(
    TaskHandle_t xTaskToResume	/*待恢复任务的任务句柄*/
);  
```

返回值为：

| 返回值  | 描述                         |
| ------- | ---------------------------- |
| pdTRUE  | 任务恢复后需要进行任务切换   |
| pdFALSE | 任务恢复后不需要进行任务切换 |



> [!note]
>
> 使用该函数注意宏：INCLUDE_vTaskSuspend和INCLUDE_xTaskResumeFromlSR必须定义为1
>
> 该函数专用于中断服务函数中，用于解挂被挂起任务。中断服务程序中要调用freeRTOS的API函数且中断优先级不能高于FreeRTOS所管理的最高优先级



内部实现：

1. 恢复任务不能是正在运行任务
2. 判断任务是否在挂起列表中是：就会将该任务在挂起列表中移除，将该任务添加到绪列表中
3. 判断恢复任务优先级：判断恢复的任务优先级是否大于当前正在运行的是的话执行任务切换

## 开启任务调度器

`vTaskStartScheduler()`：

作用：用于启动任务调度器，任务调度器启动后，FreeRTOS便会开始进行任务调度

该函数内部实现，如下：

1. 创建空闲任务
2. 如果使能软件定时器，则创建定时器任务
3. 关闭中断，防止调度器开启之前或过程中，受中断干扰，会在运行第一个任务时打开中断
4. 初始化全局变量，并将任务调度器的运行标志设置为已运行
5. 初始化任务运行时间统计功能的时基定时器
6. 调用函数`xPortStartScheduler()`

`xPortStartScheduler()`：

作用：该函数用于完成启动任务调度器中与硬件架构相关的配置部分，以及启动第一个任务

该函数内部实现，如下：

1. 检测用户在FreeRTOSConfig.h文件中对中的相关配置是否有误
2. 配置PendSV和SysTick的中断优先级为最低优先级
3. 调用函数`vPortSetupTimerlnterrupt()`配置SysTick
4. 初始化临界区嵌套计数器为0
5. 调用函数`prvEnableVFP()`使能FPU
6. 调用函数`prvStartFirstTask()`启动第一个任务

想象下应该如何启动第一个任务？

假设我们要启动的第一个任务是任务A,那么就需要将任务A的寄存器值恢复到CPU寄存器，（任务A的寄存器值，在一开始创建任务时就保存在任务堆栈里边！）

> [!note]
> 	
>
> 1. 中断产生时，硬件自动将xPSR,PC(R15),LR(R14),R12,R3-R0出/入栈；而R4~R11需要手动出/入栈
> 2. 进入中断后硬件会强制使用MSP指针，此时LR(R14)的值将会被自动被更新为特殊的EXC_RETURN

`prvStartFirstTask()`:

用于初始化启动第一个任务前的环境，主要是重新设置MSP指针，并使能全局中断

> [!tip] 
>
> 1. 什么是MSP指针？
>    程序在运行过程中需要一定的栈空间来保存局部变量等一些信息。当有信息保存到栈中时，MCU会自动更新SP指针，ARM Cortex-M内核提供了两个栈空间：
>
>    1. 主堆栈指针(MSP):它由OS内核、异常服务例程以及所有需要特权方问的应用程序代码来使用
>
>    2. 进程堆栈指针(PSP):用于常规的应用程序代码（不处于异常服务例程中时）。
>
>       > [!important]
>       >
>       > 在FreeRTOS中，中断使用MSP(主堆栈)，中断以外使用PSP(进程堆栈)
>
> 2. 为什么是0xE000ED08?
>    因为需从0xE000ED08获取向量表的偏移，为啥要获得向量表呢？因为向量表的第一个是MSP指针！
>    取MSP的初始值的思路是先根据向量表的位置寄存器VTOR(0xE000ED08)来获取向量表存储的地址，再根据向量表存储的地址，来访问第一个元素，也就是初始的MSP
>    CM3允许向量表重定位：从其它地址处开始定位各异常向量这个就是向量表偏移量寄存器，向量表的起始地址保存的就是主栈指针MSP的初始值实验

`vPortSVCHandler()`

当使能了全局中断，并且手动确触发SVC中断后，就会进入到SVC的中断服务函数中

1. 通过pxCurrentTCB获取优先级最高的绪态任务的任务栈地址，优先级最高的就绪态任务是系统将要运行的任务。

2. 通过任务的栈顶指针，将任务栈中的内容出栈到CPU寄存器中，任务栈中的内容在调用任务创建函数的时候，已初始化，然后设置PSP指针。

3. 通过往BASEPRI寄存器中写0，允许中断。

4. R14是链接寄存器LR,在ISR中（此刻我们在SVC的ISR中），它记录了异常返回值EXC_RETURN。而EXC RETURN只有6个合法的值(M4、M7)，如下表所示：

   | 描述                                 | 使用浮点单元 | 未使用浮点单元 |
   | ------------------------------------ | ------------ | -------------- |
   | 中断返回后进入Handler模式，并使用MSP | 0xFFFFFFE1   | 0xFFFFFFF1     |
   | 中断返回后进入线程模式，并使用MSP    | 0xFFFFFFE9   | 0xFFFFFFF9     |
   | 中断返回后进入线程模式，并使用 PSP   | 0xFFFFFFED   | 0xFFFFFFFD     |

> [!important]
>
> SVC中断只在启动第一次任务时会调用一次，以后均不调用

出入栈：

1. 出栈（恢复现场），方向：从下往上（低地址往高地址）：假设r0地址为0x04汇编指令示例：

   ```c
   ldmia r0!,{r4-r6}	/*任务栈r0地址由低到高，将r0存储地址里面的内容手动载到CPU寄存器r4、r5、r6*/
   
   r0地址(0x04内容加载到r4,此时地址r0=r0+4=0x08
   r0地址(0x08)内容加载到r5,此时地址r0=r0+4=0x0C
   r0地址(0x0C)内容加载到r6,此时地址r0=r0+4=0x10
   ```

2. 压栈（保存现场），方向：从上往下（高地址往低地址）：假设r0地址为0x10汇偏指令示例：

   ```c
   stmdb r0!,{r4-r6}}	/*r0的存储地址由高到低递咸，将r4、r5、r6里的内容存储到r0的任务栈里面。*/
   
   地址：r0=r0-4=0x0C,将r6的内容（寄存器值）存放到r0所指向地址(0x0C)
   地址：r0=r0-4=0x08,将r5的内容（寄存器值）存放到r0所指向地址(0x08)
   地址：r0=r0-4=0x04,将r4的内容（寄存器值）存放到0所指向地址(0x04)
   ```

现假设申请了堆，对于以下代码所申请的内存

```
头部 	| TCB1  | 头部 	| 100*4的栈大小   	|    头部 	| TCB2 	| 头部 	| 100*4的栈大小 	|   
         	 |----------------------------|            | -----------------------------------| 
		               Task1的栈                       				Task2的栈
          低地址<-----------高地址
|------------------------------------------|  			|-----------------------------------|    
        函数task1的占用堆             							     函数task2的占用堆
```

栈是高地址往下增长的，也就是从高地址往低地址，如果任务的栈中的数据过多将会破坏前面的头部

```c
xTaskCreate(Task1Function,"Task 1",100,NULL,1,&xHandeTask1);
xTaskCreate(Task2Function,"Task 2",100,NULL,1,NULL);

xTaskCreateStatic(Task3Function,"Task 3",100,NULL,1,xTask3Stack,&xTask3TCB);

xTaskCreate(TaskGenericFunction,"Task 4",100,(void *)4,1,NULL);
xTaskCreate(TaskGenericFunction,"Task 5",100,(void *)5,1,NULL);


void Task1Function( void * param)
{
	volatile char buf[500];        //该局部变量保存在栈中，然而我们申请的栈大小为100 * 4，所以栈空间大小将会被破坏

	while (1)
	{
		task1flagrun = 1; 
		task2flagrun = 0; 
		task3flagrun = 0; 
		printf("1");
		for (int i = 0; i < 500; i++)
		{
			buf[i] = 0;
		}
		
	}
	
}
	

```

![栈大小实验](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_10_52_38_202412231052787.png)



```c

void Task1Function( void * param)
{
    while (1)
    {
        printf("1");
    }
    
}

void Task2Function( void * param)
{
    while (1)
    {
        printf("2");
    }
    
}

int main( void )
{
#ifdef DEBUG
  debug();
#endif
    //初始化硬件
    prvSetupHardware();
    //创建一个任务,RTOS的核心就是多任务
    TaskHandle_t xHandeTask1;
    xTaskCreate(Task1Function,"Task 1",100,NULL,1,&xHandeTask1);    //参数列表：处理任务的函数名，任务名称，栈深度，参数列表,优先级，句柄
    xTaskCreate(Task2Function,"Task 1",100,NULL,1,NULL);
    //进行任务调度
    vTaskStartScheduler();
    return 0;
}
```



## 延时函数

接口API函数为：

```c
vTaskDelay(int counter)	/*相对延时，至少要等待指定个数的Tick Interrupt才能变为就绪态*/
vTaskDelayUntil(TickType_t baseTick,int counter)	/*绝对延时，等待到指定的绝对时刻才能变为就绪态*/
```

> [!important]
>
> 相对延时：指每次延时都是从执行函数VTaskDelay()开始，直到延时指定的时间结束
>
> 绝对延时：指将整个任务的运行周期看成一个整体，适用于需要按照一定频率运行的任务

![延时实验](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_13_52_19_202412231352486.png)

```c
static int rands[] = {3,56,23,5,99};
void Task1Function( void * param)
{
    TickType_t tStart = xTaskGetTickCount();
    int j = 0;
    while (1)
    {
        task1flagrun = 1; 
        task2flagrun = 0; 
        task3flagrun = 0; 
        for (int i = 0; i < rands[j]; i++)        //执行任务的时间是不固定的
        {
            printf("1");
        }
        j++;
        if(j == 5)
            j = 0;
        //使用两个不同的延时函数,vTaskDelayUntil是周期性的执行可以理解为绝对延迟，vTaskDelay就是相对性的延迟
        vTaskDelayUntil(&tStart,20);        //这里会延时20个tick，并且延时之后还会将当前的tick count赋值给tStart，这是一个周期性的
        // vTaskDelay(20);        //延时函数，单位为tick，能够保证这一次到下一次启动的时间一样，但休眠时间不一样
    }

}


```



## 空闲任务及钩子函数

- 删除任务后的清理工作，是在空闲任务中完成的，也就是说如果一个任务自杀，他就不能完成清理工作，依旧保持着内存的占用，所以需要在空闲任务中进行杀死
- 空闲任务会在main的任务调度器中自行创建，我们也可以使用fork（钩子）函数进行创建
- 空闲任务只能处于两个状态中的一个：running和ready
- 空闲任务钩子函数
  - 执行一些低优先级、后台的、需要连续执行的函数
  - 测量系统的空闲时间：空闲任务能被执行就意味着所有的高优先级的任务都停止了，所以测量空闲任务占据的时间，就可以算出处理器占用率
  - 让系统进入省电模式：空闲任务能被执行就意味着没有重要的事情要做，当然可以进入省电模式了
  - 据对不能导致任务进入blocked和suspended状态
  - 如果使用xTsakDelete函数来删除任务，那么钩子函数要非常高效地执行，如果空闲任务一直卡在钩子函数里的话，他就无法释放内存，所以钩子函数只能处于运行状态和就绪状态

```c
void Task2Function( void * param);
static int rands[] = {3,56,23,5,99};
void Task1Function( void * param)
{
    TickType_t tStart = xTaskGetTickCount();
    BaseType_t xReturn;        

    while (1)
    {
        //在任务1中创建任务2，这里的优先级为2，一旦task1执行，那么task2就不会再执行，所以如果想要task1也执行，那么就需要在task2中进行操作
        xReturn = xTaskCreate(Task2Function,"Task 2",100,NULL,2,&xHandeTask2);
        if(xReturn != pdPASS){
            printf("taks 2 create error\r\n");
        }
        //在这个循环中不断生成Taks2，且不断清理，这个时候程序是可以正常运行的，但是如果将这句话放到task2中，task2自杀之后，并没有任务清理task2的内存占用，所以如果还继续生成任务2的话，将会打印生成错误
        //所以当删除一个任务之后需要一个空闲任务（idle task）来进行内存的清理，但是实际上，我们并没有创建空闲任务，其实是在main函数的任务调度器中创建了一个空闲任务
        vTaskDelete(xHandeTask2);        
        task1flagrun = 1; 
        task2flagrun = 0; 
        task3flagrun = 0; 
        printf("1");

    }

}

void Task2Function( void * param)
{
    while (1)
    {
        task1flagrun = 0; 
        task2flagrun = 1; 
        task3flagrun = 0; 
        printf("2");
        vTaskDelay(2);        //这里让task2休息一会，这样task1才能继续执行
    }

}
```

当我们想要使用钩子函数创建空闲任务的时候，我们首先需要定义#define configUSE_IDLE_HOOK 1，还需要重写vApplicationIdleHook函数

```c
void vApplicationIdleHook(void)
{
	task1flagrun = 0; 
	task2flagrun = 0; 
	task3flagrun = 0; 
	taskidlerun = 1; 
	printf("0");
}
```

## 任务调度算法

### 任务切换

==任务切换的本质：CPU寄存器的切换。==

假设当由任务A切换到任务B时，主要分为两步：

1. 需暂停任务A的执行，并将此时任务A的寄存器保存到任务堆栈，这个过程叫做保存现场；
2. 将任务B的各个寄存器值（被存于任务堆栈中）恢复到CPU寄存器中，这个过程叫做恢复现场；

> [!tip] 
>
> 对任务A保存现场，对任务B恢复现场，这个整体的过程称之为：上下文切换
>
> ==任务切换的过程再PendSV中断服务函数里边完成的==，那么PendSV中断是如何触发的？
>
> 1. 滴答定时器中断调用
> 2. 执行FreeRTOS提供的相关API函数：portYIELD()
>
> 本质：通过向中断控制和状态寄存器ICSR的bit28写入1，挂起PendSV来启动PendSV中断
>
> | 位段  | 名称           | 类型 | 复位值 | 描述                                                         |
> | ----- | -------------- | ---- | ------ | ------------------------------------------------------------ |
> | 31    | NMIPENDSET     | R/W  | 0      | 写1以悬起NMI。因为NMI的优先级最高且从不掩藏，在置位此位后将立即进入NMI服务例程。 |
> | 28    | PENDSVSET      | R/W  |        | 写1以悬起 PendSV，读取它则返回 PendSV的状 态                 |
> | 27    | PENDSVCLR      | W    | 0      | 写1以清除PendSV悬起状态                                      |
> | 26    | PENDS          | R/W  | 0      | 写1以悬起SysTick。读取它则返回 PendSV的状态                  |
> | 25    | PENDSTCL       | W    | 0      | 写1以清除 SysTick悬起状态                                    |
> | 23    | ISRPREEMP      | R    | 0      | 为1时，则表示一个悬起的中断将在下一步时进 入活动状态（用于单步执行时的调试目的） |
> | 22    | ISRPENDING     | R    | 0      | 1=当前正有外部中断被悬起（不包括NMI）                        |
> | 21:12 | VECTPEND NDING | R    | 0      | 悬起的ISR的编号。如果不止一个中断悬起，则 它的值是这引动中断中，优先级最高的那一个。 |
> | 11    | RETTOBASE      | R    | 0      | 如果异常返回后将回到基级(baselevel),并且没有 其它异常悬起时，此位为1。若是在线程模式下， 在某个服务例程中，有不止一级的异常处于活动 状态，或者在异常没有活动时执行了异常服务例 程（此时执行返回指令将产生aut。此乃高危行 为，大虾也需慎用），则此位为0 |
> | 9:0   | VECTACTIVE     | R    | 0      | 当前活动的ISR编号，该位段指出当前运行中的ISR 是哪个中断的（提供异常序号），包括NMI和硬 fault。如果多个异常共享一个服务例程，该例程可根据 本位段的值来判定是哪一个异常的响应导致它的执行。把 本位段的值减去16,就得到了外中断的编号，并可以用此 编号来操作外中断相关的使能/除能等寄存器。 |

![上下文切换](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_14_7_47_202412231407666.png)

> [!important]
>
> 阻塞态的任务等待事件的触发，当事件发生之后，该任务就会变为就绪态
>
> 时间相关的事件:
>
> 1. 所谓事件相关的事件，就是设置超时时间，在指定时间内阻塞，时间到了就进入就绪态
> 2. 使用时间相关的事件，可以实现周期性的功能、可以实现超时功能
>
> 同步事件:同步事件就是某个任务在等待某些信息，别的任务或者中断服务程序会给它发送信息
>
> 发送信息的方法:
>
> 1. 任务通知（task notification）
> 2. 队列（queue）
> 3. 事件组（event group）
> 4. 信号量（semaphone）
> 5. 互斥量（mutex）

### 最高优先级任务

```c
vTaskSwitchContext();	/*查找最高优先级任务*/

taskSELECT_HIGHEST_PRIORITY_TASK();	/*通过这个函数完成*/
#define taskSELECT_HIGHEST_PRIORITY_TASK()
\{
\    UBaseType t uxTopPriority;
\    portGET_HIGHEST_PRIORITY(uxTopPriority,ux TopReady Priority )
\    configASSERT(listCURRENT LIST LENGTH(&(pxReadyTasksLists[uxTopPriority ]))>0);
\    listGET OWNER_OF NEXT_ENTRY(pxCurrentTCB,&pxReady TasksLists[uxTopPriority ]))
\}
```

#### 前导置零指令

```c
#define portGET_HIGHEST_PRIORITY(uxTopPriority,uxReadyPriorities)
uxTopPriority =(31UL-(uint32_t)_clz((uxReadyPriorities)))
```

![前导置零指令](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_14_56_21_202412231456767.png)

所谓的前导置零指令，大家可以简单理解为计算一个32位数，头部0的个数，==通过前导置零指令获得最高优先级==

#### 获取最高优先级任务控制块

通过以下函数可以获取当前最高优先级任务的任务控制块。

```c
#define listGET_OWNER_OF_NEXT_ENTRY(pxTCB,pxList)
\{
\	List_t *const pxConstList = (pxList)
\	(pxConstList)->pxIndex) = (pxConstList)->pxIndex->pxNext;
\	if((void *)pxConstList)->pxIndex == (void *)&(pxConstList )->xListEnd)
\	{
\     	(pxConstList)->pxIndex = (pxConstList)->pxlndex->pxNext;
\	}
\	(pxTCB) = (pxConstList)->pxlndex->pvOwner;
\}
```

![image-20241223150012050](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_15_0_12_202412231500428.png)



### 时间片调度

同等优先级任务轮流地享有相同的CPU时间（可设置)，叫时间片，在FreeRTOS中，一个时间片就等于SysTick中断周期

![时间片调度](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_15_8_15_202412231508320.png)

运行条件：

1. 创建三个任务：Task1、Task2、Task3
2. Task1、Task2、Task3的优先级均为1；即3个任务同等优先级
3. 同等优先级任务，轮流执行：时间片流转
4. 一个时间片大小，取决为滴答定时器中断频率
5. 注意没有用完的时间片不会再使用，下次任务Tsk3得到执行还是按照一个时间片的时钟节拍运行

运行过程如下：

1. 首先Task1运行完一个时间片后，切换至Task2运行
2. Task2运行完一个时间片后，切换至Task3运行
3. Task3运行过程中（还不到一个时间片），Task3阻塞了（系统延时或等待信号量等），此时直接切换到下一个任务Task1
4. Task1运行完一个时间片后，切换至Task2运行

当3个任务的优先级相同时，三个任务轮流执行

![优先级相同时，任务调度状况](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_10_46_24_202412231046650.png)

当3个任务的优先级不同时，只要高优先级的任务没有放弃运行，低优先级的任务就不会运行

![优先级不同时，任务调度情况](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/23_10_47_12_202412231047372.png)

# 同步和互斥通信

> 一句话理解同步与互斥：我等你用完厕所，我再用厕所。
>
> 什么叫同步？就是：哎哎哎，我正在用厕所，你等会。
>
> 什么叫互斥？就是：哎哎哎，我正在用厕所，你不能进来。
>
> 同步与互斥经常放在一起讲，是因为它们之的关系很大，“互斥”操作可以使用“同步”来实现。我“等”你用完厕所，我再用厕所。这不就是用“同步”来实现“互斥”吗？

假设有A、B两人早起抢厕所，A先行一步占用了；B慢了一步，于是就眯一会：当A用完后叫醒B,B也就愉快地上厕所了。

在这个过程中，A、B是互斥地访问“厕所”，“厕所”被称之为临界资源。我们使用了“休眠-唤醒”的同步机制实现了“临界资源”的“互斥访问”。

```c
void 抢厕所(void)
{
    if(有人在用)
    {
        我眯一会； 
    }
    用厕所;
    喂，醒醒，有人要用厕所吗；
}
```

## 自定义实现同步

虽然这样可以实现同步，但是当task1在执行的过程中，task2也在一直消耗cpu的资源，将会降低task1的计算速度。

```c
int sum = 0;
volatile int flagCalEnd = 0;

void Task1Function( void * param)
{
    volatile int i = 0;        //故意使执行时间变长
    while (1)
    {
        for(i = 0; i < 1000000; i++ ){
            sum++;
        }
        flagCalEnd++;        //表示计算完成
        vTaskDelete(NULL);
    }
}

void Task2Function( void * param)
{
    while (1)
    {
        if(flagCalEnd)
            printf("sum = %d\r\n",sum);
    }
}
```



## 自定义实现互斥

这里使用的printf就是互斥变量，当task3打印字符的时候，task4会进行打断，同理task4也会被task3打断，这时候打印的字符串就不连串了

```c
xTaskCreate(TaskGenericFunction,"Task 3",100,"Task3 is running",1,NULL);
xTaskCreate(TaskGenericFunction,"Task 4",100,"Task4 is running",1,NULL);

void TaskGenericFunction( void * param)
{
    while (1)
    {        
        printf("%s\r\n",(char *)param);
    }

}
```

实验输出为：

```text
Task4 is ruTask3 is runnning
Taskining
is runningrunning
Task4 is ruTask3 is runnning
Task4ning
is runningrunning
Task4 is ruTask3 is runnning
Task4ning
is Runningrunn1ng
Task4 is ruTask3 is runnning
```

我们可以设置一个互斥变量，使得串口互斥的运行

```c
int flagUartUsed = 0;

void TaskGenericFunction( void * param)
{
	while (1)
	{        
		if(!flagUartUsed){
			flagUartUsed = 1;
			printf("%s\r\n",(char *)param);
			flagUartUsed = 0;
		}
	}
	
}
```

实验输出为：

```
Task4 is running
Task3 is running
Task4 is running
Task3 is running
Task4 is running
Task3 is running
Task4 is runn1ng
Task3 is running
Task4 is running
Task3 is running
Task4 is running
```

但实际上这样写是有隐患的，当task3执行到printf之前，且task4也执行到pintf之前，这个时候两个任务的都会卡死

## FreeRTOS实现

能实现同步、互斥的内核方法有：任务通知(task notification)、队列(queue)、事件组(event group)、信号量(semaphoe)、互斥量(mutex)。

| 内核对象 | 生产者     | 消费者 | 数据/状态                                                    | 说明                                                         |
| -------- | ---------- | ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 队列     | ALL        | ALL    | 数据：若干个数据 谁都可以往队列里扔数据， 谁都可以从队列里读数据 | 用来传递数据， 发送者、接收者无限制， 一个数据只能唤醒一个接 收者 |
| 事件组   | ALL        | ALL    | 多个位：或、与 谁都可以设置[生产)多个位， 谁都可以等待某个位、若干个 位 | 用来传递事件， 可以是N个事件， 发送者、接受者无限制， 可以唤醒多个接收者：像 广播 |
| 信号量   | ALL        | ALL    | 数量：0~n 谁都可以增加一个数量， 谁都可消耗一个数量          | 用来维持资源的个数， 生产者、消费者无限制， 1 个资源只能唤醒1个接 收者 |
| 任务通知 | ALL        | 只有我 | 数据、状态都可以传输， 使用任务通知时， 必须指定接受者       | N 对1的关系: 发送者无限制， 接收者只能是这个任务             |
| 互斥量   | 只能A 开锁 | A上锁  | 位：0、1 我上锁：1变为0, 只能由我开锁：0变为1                | 就像一个空厕所, 谁使用谁上锁， 也只能由他开锁                |

1. 队列：

   里面可以放任意数据，可以放多个数据

   任务、 ISR 都可以放入数据；任务、 ISR 都可以从中读出数据

2.  事件组：

    一个事件用一 bit 表示， 1 表示事件发生了， 0 表示事件没发生

    可以用来表示事件、事件的组合发生了，不能传递数据

    有广播效果：事件或事件的组合发生了，等待它的多个任务都会被唤醒

3. 信号量：

    核心是"计数值"

    任务、 ISR 释放信号量时让计数值加 1

    任务、 ISR 获得信号量时，让计数值减 1

4.  任务通知：

    核心是任务的 TCB 里的数值

    会被覆盖

    发通知给谁？必须指定接收任务

    只能由接收任务本身获取该通知

5. 互斥量：

    数值只有 0 或 1

    谁获得互斥量，就必须由谁释放同一个互斥量

![FreeRTOS的同步互斥的方法](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/25_16_44_30_202412251644239.png)

# 队列

## 常规操作

==左边可以看作是生产者，右边是消费者，队列可以看作是一个传送带==

队列的简化操如入下图所示，从此图可知：

- 队列可以包含若干个数据：队列中有若干项，这被称为"长度"(length)
- 每个数据大小固定
- 创建队列时就要指定长度、数据大小
- 数据的操作采用先进先出的方法(FIFO，First In First Out)：写数据时放到尾部，读数据时从头部读
- 也可以强制写队列头部：覆盖头部数据

![队列常规操作](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/25_16_50_26_202412251650174.png)

==队列先入先出，从队尾入队，队头出队，使用环形缓冲区实现==，更详细的：

![队列详细过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/25_16_51_28_202412251651103.png)

## 传输数据

使用队列传输数据时有两种方法：

- 拷贝：把数据、把变量的值复制进队列里
- 引用：把数据、把变量的地址复制进队列里

FreeRTOS使用拷贝值的方法，这更简单：

- 局部变量的值可以发送到队列中，后续即使函数退出、局部变量被回收，也不会影响队列中的数据无需分配
- buffer来保存数据，队列中有buffer
- 局部变量可以马上再次使用
- 发送任务、接收任务解耦：接收任务不需要知道这数据是谁的、也不需要发送任务来释放数据如果数据实在太大，你还是可以使用队列传输它的地址
- 队列的空间有FreeRTOS内核分配，无需任务操心
- 对于有内存保护功能的系统，如果队列使用引用方法，也就是使用地址，必须确保双方任务对这个地址都有访问权限。使用拷贝方法时，则无此限制：内核有足够的权限，把数据复制进队列、再把数据复制出队列。

## 队列的阻塞访问

只要知道队列的句柄，谁都可以读、写该队列。任务、ISR都可读、写队列。可以多个任务读写队列。

任务读写队列时，简单地说：如果读写不成功，则阻塞；可以指定超时时间。

> [!tip]
>
> 口语化地说，就是可以定个闹钟：如果能读写了就马上进入就绪态，否则就阻塞直到超时。

某个任务读队列时，如果队列没有数据，则该任务可以进入阻塞状态：还可以指定阻塞的时间。如果队列有数据了，则该阻塞的任务会变为就绪态。如果一直都没有数据，则时间到之后它也会进入就绪态。

既然读取队列的任务个数没有限制，那么当多个任务读取空队列时，这些任务都会进入阻塞状态：有多个任务在等待同一个队列的数据。当队列中有数据时，哪个任务会进入就绪态？

- 优先级最高的任务
- 如果大家的优先级相同，那等待时间最久的任务会进入就绪态

跟读队列类似，一个任务要写队列时，如果队列满了，该任务也可以进入阻塞状态：还可以指定阻塞的时间。如果队列有空间了，则该阻塞的任务会变为就绪态。如果一直都没有空间，则时间到之后它也会进入就绪态。

既然写队列的任务个数没有限制，那么当多个任务写"满队列"时，这些任务都会进入阻塞状态：有多个任务在等待同一个队列的空间。当队列中有空间时，哪个任务会进入就绪态？

- 优先级最高的任务
- 如果大家的优先级相同，那等待时间最久的任务会进入就绪态

## 队列函数

### 创建

队列的创建有两种方法：

1. 动态分配内存

   动态分配内存：xQueueCreate，队列的内存在函数内部动态分配，函数原型如下：

   ```c
   QueueHandle_t xQueuecreate(UBaseType_t uxQueueLength,UBaseType_t uxItemsize);
   ```

   | 参数          | 说明                                                         |
   | ------------- | ------------------------------------------------------------ |
   | uxQueueLength | 队列长度，最多能存放多少个数据(item)                         |
   | uxltemSize    | 每个数据(item)的大小：以字节为单位                           |
   | 返回值        | 非0：成功，返回句柄，以后使用句柄来操作队列 <br>NULL：失败，因为内存不足 |

2. 静态分配内存  :xQueueCreateStatic，队列的内存要事先分配好,函数原型如下：  

   ```c
   QueueHandle_t xQueuecreatestatic(
       UBaseType_t uxQueueLength,
       UBaseType_t uxItemsize,
       uint8_t *pucQueuestorageBuffer,
       StaticQueue_t *pxQueueBuffer
   );
   ```

   | 参数                  | 说明                                                         |
   | --------------------- | ------------------------------------------------------------ |
   | uxQueueLength         | 队列长度，最多能存放多少个数据(item)                         |
   | uxltemSize            | 每个数据(item)的大小：以字节为单位                           |
   | pucQueueStorageBuffer | 如果uxltemSize非O，pucQueueStorageBuffer必须指向一个 uint8_t数组, 此数组大小至少为"uxQueueLength *uxltemSize" |
   | pxQueueBuffer         | 必须执行一个StaticQueue_t结构体，用来保存队列的数据结构      |
   | 返回值                | 非0：成功，返回句柄，以后使用句柄来操作队列<br>NULL：失败，因为pxQueueBuffer为NULL |

示例：

```c
#define QUEUE_LENGTH 10
#define ITEM_SIZE sizeof(uint32_t）

//xQueueBuffer用来保存队列结构体
StaticQueue_t xQueueBuffer;

//ucQueuestorage用来保存队列的数据
//大小为：队列长度*数据大小
uint8_t ucQueuestorage[QUEUE_LENGTH ITEM_SIZE];
void vATask(void *pvParameters)
{
    QueueHandle_t xQueue1;

    //创建队列：可以容纳QUEUE_LENGTH个数据，每个数据大小是ITEM_SIZE
    xQueue1 =  xQueuecreatestatic(QUEUE_LENGTH,
                                  ITEM_SIZE,
                                  ucQueuestorage,
                                  &xQueueBuffer);
}
```

### 复位

队列刚被创建时，里面没有数据；使用过程中可以调用 xQueueReset() 把队列恢复为初始状态，此函数原型为：  

```c
/*	pxQueue:复位哪个队列；
	返回值：pdPASS(必定成功)
*/
BaseType_t xQueueReset(QueueHandle_t pxQueue);
```

### 删除

删除队列的函数为 vQueueDelete() ，只能删除使用动态方法创建的队列，它会释放内存。原型如下：  

```c
void vQueueDelete(QueueHandle_t xQueue);
```

### 写队列

可以把数据写到队列头部，也可以写到尾部，这些函数有两个版本：在任务中使用、在ISR中使用。函数原型如下：  

```c
/*	等同于xQueueSendToBack
	往队列尾部写入数据，如果没有空间，阻塞时间为xTicksTowait
*/
BaseType_t xQueuesend(
    QueueHandle_t	xQueue,
    const void		*pvItemToQueue,
    TickType_t	    xTicksTowait
);

/*
	往队列尾部写入数据，如果没有空间，阻塞时间为xTicksTowait
*/
BaseType_t xQueuesendToBack(
    QueueHandle_t	xQueue,
    const void	    *pvItemToQueue,
    TickType_t	    xTicksTowait
);

/*
	往队列尾部写入数据，此函数可以在中断函数中使用，不可阻塞
*/
BaseType_t xQueuesendToBackFromISR(
    QueueHandle_t xQueue,
    const void *pvItemToQueue,
    BaseType_t *pxHigherpriorityTaskwoken
);

/*
    往队列头部写入数据，如果没有空间，阻塞时间为xTicksTowait
*/
BaseType_t xQueuesendToFront(
    QueueHandle_t	xQueue,
    const void	    *pvItemToQueue,
    TickType_t	    xTicksTowait
);

/*
    往队列头部写入数据，此函数可以在中断函数中使用，不可阻塞
*/
BaseType_t xQueuesendToFrontFromISR(
    QueueHandle_t 	xQueue,
    const void 		*pvItemToQueue,
    BaseType_t 		*pxHigherPriorityTaskwoken
)；
```



| 参数          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| xQueue        | 队列句柄，要写哪个队列                                       |
| pvltemToQueue | 数据指针，这个数据的值会被复制进队列， 复制多大的数据？在创建队列时已经指定了数据大小 |
| xTicksToWait  | 如果队列满则无法写入新数据，可以让任务进入阻塞状态， xTicksToWait表示阻塞的最大时间(Tick Count)。 如果被设为0，无法写入数据时函数会立刻返回; 如果被设为portMAX_DELAY，则会一直阻塞直到有空间可写 |
| 返回值        | pdPASS：数据成功写入了队列<br>errQUEUE_FULL：写入失败，因为队列满了。 |

### 读队列

使用 xQueueReceive() 函数读队列，读到一个数据后，队列中该数据会被移除。这个函数有两个版本：在任务中使用、在ISR中使用。函数原型如下  ：

```c
BaseType_t xQueueReceive(QueueHandle_t xQueue,
                         void* const pvBuffer,
                         TickType_t xTicksTowait
                        );
BaseType_t xQueueReceiveFromISR(
    QueueHandle_t	xQueue,
    void		    *pvBuffer,
    BaseType_t	    *pxTaskwoken);
```



| 参数         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| xQueue       | 队列句柄，要读哪个队列                                       |
| pvBuffer     | bufer指针，队列的数据会被复制到这个buffer 复制多大的数据？在创建队列时已经指定了数据大小 |
| xTicksToWait | 果队列空则无法读出数据，可以让任务进入阻塞状态， xTicksToWait表示阻塞的最大时间(Tick Count)。 如果被设为0，无法读出数据时函数会立刻返回; 如果被设为portMAX_DELAY，则会一直阻塞直到有数据可写 |
| 返回值       | pdPASS：从队列读出数据入 <br>errQUEUE_EMPTY：读取失败，因为队列空了。 |

### 查询

可以查询队列中有多少个数据、有多少空余空间。函数原型如下：  

```c
/*
	返回队列中可用数据的个数
*/
UBaseType_t uxQueueMessageswaiting(const QueueHandle_t xQueue );
/*
	返回队列中可用空间的个数
*/
UBaseType_t uxQueueSpacesAvailable(const QueueHandle_t xQueue );
```



### 覆盖



当队列长度为1时，可以使用 xQueueOverwrite() 或 xQueueOverwriteFromISR() 来覆盖数据。注意，队列长度必须为1。当队列满时，这些函数会覆盖里面的数据，这也意味着这些函数不会被阻塞。函数原型如下：

```c
/*
	覆盖队列
	xQueue:写哪个队列
	pvItemToQueue:数据地址
	返回值：pdTRUE表示成功，pdFALSE表示失败
*/
BaseType_t xQueueoverwrite(
    QueueHandle_t xQueue,
    const void pvItemToQueue
);

BaseType_t xQueueoverwriteFromISR(
    QueueHandle_t xQueue,
    const void pvItemToQueue,
    BaseType_t *pxHigherPriorityTaskwoken
);
```

### 窥视

如果想让队列中的数据供多方读取，也就是说读取时不要移除数据，要留给后来人。那么可以使用"窥视"，也就是 xQueuePeek() 或 xQueuePeekFromISR() 。这些函数会从队列中复制出数据，但是不移除数据。这也意味着，如果队列中没有数据，那么"偷看"时会导致阻塞；一旦队列中有数据，以后每次"偷看"都会成功。函数原型如下：

```c
/*	偷看队列
	xQueue:偷看哪个队列
	pvItemToQueue:数据地址，用来保存复制出来的数据
	xTicksTowait:没有数据的话阻塞一会
	返回值：pdTRUE表示成功，pdFALSE表示失败
*/
BaseType_t xQueuePeek(
    QueueHandle_t xQueue,
    void* const pvBuffer,
    TickType_t xTicksTowait
);
BaseType_t xQueuePeekFromISR(
    QueueHandle_t xQueue,
    void *pvBuffer,
);
```

## 示例

### 基本使用

本程序会创建一个队列，然后创建2个发送任务、1个接收任务：

- 发送任务优先级为1，分别往队列中写入100、200
- 接收任务优先级为2，读队列、打印数值

main函数中创建的队列、创建了发送任务、接收任务，代码如下：

```c
/*队列句柄，创建队列时会设置这个变量*/
QueueHandle_t xQueue;

int main(void)
{
    prvSetupHardware();

    /*创建队列：长度为5，数据大小为4字节（存放一个整数）*/
    xQueue = xQueuecreate(5,sizeof(int32_t ));
    if(xQueue != NULL)
    {
        /*	创建2个任务用于写队列，传入的参数分别是100、200
			任务函数会连续执行，向队列发送数值100、200
			优先级为1
		*/
        xTaskcreate(vSenderTask,"Sender1",1000,void 100,1,NULL);
        xTaskcreate(vSenderTask,"Sender2",1000,void 200,1,NULL);

        /*	创建1个任务用于读队列
			优先级为2，高于上面的两个任务
			这意味着队列一有数据就会被读走
		*/
        xTaskcreate(vReceiverTask,"Receiver",1000,NULL,2,NULL);

        /*启动调度器*/
        vTaskstartScheduler();
    }
    else
    {
        /*无法创建队列*/
    }
    
    /*如果程序运行到了这里就表示出错了，一般是内存不足*/
    return 0;
}
```

发送任务的函数中，不断往队列中写入数值，代码如下：  

```c
static void vSenderTask(void *pvParameters)
{
    int32_t 1valueToSend;
    BaseType_t xstatus;
    /*	我们会使用这个函数创建2个任务
		这些任务的pvParameters不一样
	*/
    lValueToSend = (int32_t)pvParameters;
    /*无限循环*/
    for(;;)
    {
        /*	写队列
            xQueue:写哪个队列
            &1Va1ueToSend:写什么数据？传入数据的地址，会从这个地址把数据复制进队列
            0:不阻塞，如果队列满的话，写入失败，立刻返回
        */
        xStatus = xQueueSendToBack(xQueue,&1valueToSend,0);
        if(xStatus != pdPASS)
        {
            printf("Could not send to the queue.\r\n")
        }
    }
}
```

接收任务的函数中，读取队列、判断返回值、打印，代码如下：  

```c
static void vReceiverTask(void *pvParameters)
{
    /*读取队列时，用这个变量来存放数据*/
    int32_t lReceivedvalue;
    BaseType_t xstatus;
    const TickType_t xTicksTowait = pdMS_TO_TICKS(100UL);
    /*无限循环*/
    for(;;)
    {
        /*	读队列
            xQueue:读哪个队列
            &lReceivedvalue:读到的数据复制到这个地址
            xTicksTowait:如果队列为空，阻塞一会
        */
        xStatus = xQueueReceive(xQueue,&lReceivedvalue,xTicksTowait);
        if(xStatus == pdPASS)
        {
            /*读到了数据*/
            printf("Received %d\r\n",lReceivedvalue);
        }
        else
        {
            /*没读到数据*/
            printf("Could not receive from the queue.\r\n")
        }
    }
}
```

程序运行结果如下：

![程序的运行结果](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/26_15_29_44_202412261529340.png)

任务的调度情况如下图所示：

![任务调度图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/26_15_30_17_202412261530492.png)



### 分辨数据源

当有多个发送任务，通过同一个队列发出数据，接收任务如何分辨数据来源？数据本身带有"来源"信息，比如写入队列的数据是一个结构体，结构体中的lDataSouceID用来表示数据来源：  

```c
typedef struct {
    ID_t eDataID;
    int32_t lDataValue;
}Data_t;
```

不同的发送任务，先构造好结构体，填入自己的eDataID ，再写队列；接收任务读出数据后，根据eDataID就可以知道数据来源了，如下图所示：

- CAN任务发送的数据：eDataID = eMotorSpeed
- HMI任务发送的数据：eDataID = eSpeedSetPoint

![不同的任务的不同id](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2025/01/2_9_0_40_202501020900849.png)















































































































































