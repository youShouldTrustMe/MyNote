

# 宏

## 宏

> [!important]
>
> 宏是一个纯粹的文本替换



```c
#define 宏名 宏体
```

当宏体包含运算符时，需要增加括号来==限制==宏体和外部的计算优先级

```c
#define 宏名 (带有运算的的宏体)
```



## 条件编译

条件编译语句：编译中的分支语句

```c
#ifdef your_macro	//条件编译语句

#else	//条件编译语句

#endif	//条件编译语句

#if your_macro == constant	//条件编译语句 

#endif	//条件编译语句
```

提示编译语句

```c
#ifdef your_macro	
#else
#error "error information durnning in the compile"	//在编译过程中会显示的错误信息
#warning "warning information durnning in the compile"	//在编译过程中会显示的警告信息
#endif
```



## 宏函数

> [!important]
>
> 宏函数可以减少内存开销

简单使用

```c
#define FUNC(a,b) a*b

//上面说了，带有运算的宏体需要添加括号，所以想要实现功能需要修改为如下代码
#define FUNC(a,b) ((a)*(b))
```

使用`\`可以实现宏函数的多行显示

```c
#define FUNC(a,b) ((a) = (a) * (b));
						\(a++)                           
```

> [!note]
>
> 在多行显示的代码中，可以看到(a++)后没有==冒号==
>
> 实际上由于宏是纯粹的文本替代，所以在调用FUNC函数时会加上冒号，所以在宏体中不需要加冒号

宏函数与分支语句的联动，如果不适用花括号来控制分支语句的作用域，可能会出现不符合预期的结果

```c
if(0)
    FUNC(a,b);

//此时可以通过在if语句中加花括号，亦可以在宏体中使用花括号
#define FUNC(a,b) {((a) = (a) * (b));
						\(a++)；}

//此时只能使用一条if语句，而不能使用else语句，否则会报错，因为我们在宏体中加了分号，展开为
if(0)
{
    ((a) = (a) * (b));
    (a++)；
};

//如果此时增加else语句
if(0)
	FUNC(a,b);
else
	FUNC(a,b);

//那么展开为
if(0)
{
    ((a) = (a) * (b));
    (a++)；
};	//这里会多出一个冒号，会截断if语句，导致下面的else语句成为独立的else，编译器会报错
else
{
    ((a) = (a) * (b));
    (a++)；
};

```

要保证多分支语句的正确运行，我们可以使用do-while语句来控制

```c
#define FUNC(a,b) do{((a) = (a) * (b));
						\(a++)；}while(0)//注意，此时的while(0)后面不能加分号，不然使用分支语句时依然会报错                     
```

## 调试宏

使用`#`来实现字符化

```c
#define LOG_INFO(x) printf("x is %d\n",(x))
//此时当我使用该宏时，printf函数中的x并不会被替换，如
int a = 10;
LOG_INFO(a);	//打印为x is 10

//如果使用#来进行字符化，可以将宏修改为
#define LOG_INFO(x) printf("%s is %d\n",#(x),(x))
LOG_INFO(a);	//打印为a is 10
```

使用`##`进行连接，一般用于有规律的变量

```c
#define LOG_INFO(x) printf("x is %d\n",(x))
#define DAY_DECLARE(x) int day##x
#define DAY(x) day##x

int a = 10;
DAY_DECLARE(1) = 100;
DAY_DECLARE(21) = 200;
LOG_INFO(DAY(1));	//打印为DAY（1） is 100
LOF_INFO(DAY(21));	//打印为DAY（21） is 200
```

使用系统自定义的宏来打印相关信息

```c
__FUNCTION__	//用于打印函数名
__LINE__		//用于打印行号
__FILE__		//用于打印文件名
__VA_ARGS__		//在使用可变长变量时用于解析变量
 
//知道这些之后，可以在打印日志的地方来显示调用的函数、行号和文件名
#define LOG_INFO(x) printf("%s:%s:%d %s is %d \n",__FILE__,__FUNCTION__,__LINE__,#x,(x));
//上面会打印信息文件名，调用函数名，调用的行号等等相关信息，所以打印结果为1.c:main:23:DAY(21)is 200
    
//但是上面的函数的使用限制会很大，不能实际实现printf函数的使用手法，所以可以使用...来控制可变长变量，改造代码如下
#define LOG_INFO(x,...) printf("%s:%s:%d:"x"",__FILE__,__FUNCTION__,__LINE__,##__VA_ARGS__)
//使用方法如下
LOG_INFO("DAY1 = %d\n",DAY(21));	//此处会打印1.c:main:23:DAY21=200
```



# 关键字

关键字可以被分为3大类：

1. 数据类型关键字
2. 修饰关键字
3. 逻辑关键字

## sizeof

> [!important]
>
> 在编译过程中就已经能算出大小，并不是在运行过程中计算的
>
> `sizeof`并不是函数，而是关键字，所以我们也可以使用`sizeof a`，来定义一个常量，但是这里需要注意的是`a`需要在之前已经定义了，且不可以使用`sizeof(int)`这样的表示

## 数据类型关键字

### 标准类型

#### char

`char`类型的关键字是最小内存空间的数据类型，也是最适合操作硬件的数据类型。

> [!tip]
>
> 我们知道硬件上最小的状态是高低电平，对于软件来讲就是0和1状态，那C语言中为什么没有byte这个数据类型关键字呢？
>
> 因为0-1只有这两个状态，对于内存来讲当然是充分利用，但对于CPU来讲处理效率非常低。比如要把32(0b100000)这个数据从内存读到CPU内部寄存器，至少需要执行6个指令周期(1个指令周期：寻址->发读的控制信息->数据通过数据总线送到寄存器)才能读到0b100000。所以兼顾CPU性能和内存管理，计算机科学家把操作内存最小单位为8bit,也称1byte,也就是char类型，可以表示256种状态。当编译器看到`chār`关键字的时候，就知道圈定的内存大小是1byte:
>
> 



> [!NOTE]
>
> 这里的`char a=10;`相当于在内存中圈出一块1byte的内存，内存的标签叫a(也可以理解成这块内存属于a),内存里面的值是10
>
> Q:想一想，声明和定义的区别？extern char a和char a;
>
> A:声明没有分配内存，定义会分配内存。(声明实际上是引用，并不会分配空间)

#### int

`int`类型是最适合CPU的数据类型，大小和编译器有关

![CPU处理数据过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/11/30_10_53_6_202411301053994.png)



为了充分发挥CPU的数据处理能力，数据总线尽量要充分利用，在系统一个周期内所能接受的最大处理单位就是`int`。所以对于32位的CPU,`int`就是32bit(4byte)，16位的CPU,int就是16bit(2byte)。对于一个数字常量默认就是int的，体会以下代码：

#### void

`void`一般用于占位符

> [!tip]
>
> 为了减少编译器报未使用参数的警告，我们可以使用void来规避
>
> ```c
> int main(void)
> {
>     int a = 0;
>     (void)a;	//此时可以规避编译器出现警告的情况
>     return 0 ;
> }
> ```
>
> 

> [!tip]
>
> void*是一个通用类型指针，可以将这个指针转化为任意类型的指针

### 自定义类型

#### struct

C语言通过struct关键字表示新的组合类型，内存表现为累加且对齐，我们叫这种数据类型为结构体。声明规则如下：

```c
struct 结构体类型名
{
	int a;		//结构体成员变量
	short b;	//结构体成员变量
	...
}；
```



> [!note]
>
> 当我们申明了一个结构体时会分配内存吗？
>
> ```c
> struct abc
> {
> 	int a;
> 	int b;
> };
> ```
>
> 答案是否定的，注意，申明不会申请内存

> [!tip]
>
> 数据对齐和CPU总线有关，如果CPU总线为4字节，那么地址4字节对齐会对CPU的读写有利

![字节对齐](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/11/30_11_57_7_202411301157415.png)

#### union

union关键字表示新的类型叫联合体，内存表现为：==共享一份内存==（以最大数据类型作为分配空间）和内存的首地址，申明规则如下：

```c
union 名字
{
	char a;
	int b;
    ...
}
```

![大小端示例](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/2_20_34_10_202412022034339.png)

#### struct+union

```c
#include <stdio.h>

struct abc
{
    int a;
    short b;
    long c;
}__attribute__((packed));	//用于防止编译器进行字节对齐

struct efg
{
	int d;
    short e;
    long f;
}__atttribute__((packed));	//用于防止编译器进行字节对齐

#pragma anon_unions	//用于匿名union编译不报错
struct package
{
    unsigned char id;
    union
    {
        unsigned char payload[24];	//payload占用空间和a、b一致
        struct abc a;
        struct efg b;      
    };
};
```

> [!tip]
>
> 注意，编译器实际上是会对数据体进行对齐的，所以使用编译器选项`__attribute__((packed))`可以防止编译器进行字节对齐，也就是不会自动的填充0。
>
> 同时，上述的代码中的union并没有命名，这样不报错的原因是在结构体的开头增加了`#pragma anon_unions`

#### enum

enum关键字表示的类型叫枚举类型，内存表现为：内存根据定义值的大小默认选择整数常量大小(int),如果超出int大小，编译器会选择更大的整型类型，比如Iong,所以内存大小不是固定的。声明的规则如下：

```c
enum 名字
{
    A = 200，
   	B = 300，
    C = 400，    
}；
```

> [!note]
>
> 实际上，枚举类型是建议型类型，如下列代码
>
> ```c
> enum abc
> {
> 	A = 1,
>     B = 2,
>     C = 3,
> };
> 
> enum abc func(void)
> {
>     return 10;
> }
> 
> enum abc a = func();	//此时打印a的值可以是10，所以说是一个建议类型
> 
> ```
>
> 

### 地址（指针）类型

C语言中，地址类型也称指针类型，圈定的内存用来存放地址编号的值。==指针类型的内存表现限编译器有关或者说跟CPU的地址总线有关==。在32位的系统中，指针类型占用4byte,在64位的系统中，指针类型占用8byte。

![地址映射](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/3_19_22_58_202412031922432.png)

> [!TIP]
>
> 指针所占用的内存空间和申明的类型没有关系，只和编译器和CPU的地址总线宽度有关
>
> 申明类型只和访问的数据的范围有关，如int类型的指针就要访问4个字节的数据，char类型的指针只能访问1个字节的数据



```c
#include <stdio.h>

int main(void)
{
    int a = 10;
    int *p = &a;
    printf("symbol visit:a %d,addr visit:a =%d\n",a,*p);	//打印出symbol visit:a 10,addr visit:a 10
    return 0;
}

```

> [!tip]
>
> 1. 首先，p也是一个变量（没啥神秘的），这本质上也是圈定一块内存，内存的大小跟编译器有关，p是这块内存的标签。到这里和的在变量的视角中一摸一样
> 2. 然后，p的特殊性在于，它圈定的那块内存，存放的不是一般的值而是一个地址编号。所以除了通过标签读出这个地址编号外，C语言还提供一种直接访问这个地址编号的方法，那就是在p前面加一个`*`号。
>
> 举一个形象的例子：
>
> 你要去访问A朋友家，第一种是直接让A带你去，第二种是P知道A家的地址，通过P带你去。a和*p就是这种感觉

### typedef

typedef:关键字是给数据类型起一个别名，让程序的可读性更高，特别是复杂的类型（如函数指针类型），做到见字知意。使用的规则如下：

```c
#include <stdio.h>
typedef struct abc
{
    int a;
    int b;
    float c;
}abc_t;
typedef int len_t;
typedef unsigned char uint8_t;
typedef void (func*)(void);
int main(void)）
{
    len_t a = 0;
    uint8_t b;
    abc_t c;
    return 0;
}
```

## 修饰关键字

### auto

auto关键字是最无感的存在。编译器在默认缺省的情况下，所有变量都是uto的。你就当做它不存在，不曾来过。

### register

register关键字是一个建议性关键字。设计的初衷是想要定义的变量放到cpu的寄存器中，但是往往编译器看到这个关键字的时候，发出一句呐喊：臣妾做不到啊。寄存器相对于内存，cpu在访问速度上更快，但也很稀缺，不是说想放就放的。所以编译器只能尽量的去做，它不保证一定能做得到，大家也就不要奢望了。

> [!tip]
>
> 但值得注意的是加上registerf修饰后的变量，无法通过&来取地址。即使大概率放进寄存器会失败，但编译器就不给你寻址，万一成功了呢？编译器还是有梦想的。

### static

在C语言中static关键字有两个作用：

1. 规定变量的内存从静态全局数据区分配；
2. 限定变量或函数的作用域

**生命周期**：不管是修饰局部变量还是全局变量，最后都在静态全局数据区分配。这意味着变量在整个程序的生命周期内都是有效合法的。

**限定范围**：限定全局变量只在当前定义的文件可见，其他文件即使加上exterr修饰，也无法引用。会编译报错

### extern

extern用来声明一个没有被static修饰的变量或者函数。

在架构设计中尽量不要用extern,会导致代码耦合度非常高而且不太可控。建议使用`get`、`set`方法来获取成员变量的值，从而避免出现文件的互相调用。

```c
static int a = 0;

int get_a()
{
    return a;
}

void set_a(int para)
{
    a = para;
}
```

extern"C",用C++的编译器(g++)按照C的规则进行编译。C++和C语言在编译规则上有很多不同，所以在C++中想完全复用C写的代码，就可以用extern"C"来修饰。这种在C++使用C的Iib库的时候比较常见。

> [!tip]
>
> 实际上，extern c就是在c++代码中使用c的编译规则去编译代码



### const

const是constant的缩写，是恒定不变的意思，被修饰的变量经常被人误解成常数或者常量。这个理解不太准确，准确的理解应该是read only。.本质上还是变量，编译器只能尽量的不让你去修改，但是实际上总有一些方法可以做到修改c0st修饰变量的值，往往都是一些异常的操作导致（比如数组越界、指针越界访问、栈溢出等等）。我们要建立一个正确的编程理念：const修饰的变量技术上能改，但是不要去改。

```c
//const 数据类型 变量名 or 数据类型 const 变量名
#include <stdio.h>
int main(void)
{
    const int a = 10;
    int const b = 20;
    //a=20;//编译错误，编译不给修改
    printf("a =%d,b =%d\n",a,b);
    return 0;
}
```



> [!tip]
>
> 实际上这里的变量a和b也是可以修改的，之前说过，访问一个变量有两种方法：
>
> 1. 通过标签去访问值，如访问a和b
> 2. 通过地址去访问值，如访问&a和&b
>
> 上面的代码如果使用地址来修改就可以修改值，在c++中编译器会自动拦截修改const类型的值，但是c中不会

==const修饰指针==

很多同学对const修饰指针变量，有时候分不清楚是修饰指针变量还是限定指针指向的内容。今天教大家一个方法，叫"近水楼台先得月"：

1. 忽略数据类型
2. 观察const向右靠近什么
   1. 靠近 *：表示指针指向的内容不能变
   2. 靠近变量：表示指针的指向（指针中存的内容）不能变
   3. 既靠近*又靠近变量：表示指针内容和指针指向的内容都不能变

> [!tip]
>
> 在定义指针的时候，编译器会先向右遍历，再向左遍历，然后才会确定一个变量的具体限制

```c
#include <stdio.h>
const int* a;		//-> const *a -> 靠近*，所以地址指向的内容（*a)不能变
int const *b;		//-> const *b -> 靠近*，所以地址指向的内容(*b)不能变
int* const c;		//-> * const c-> 靠近c,所以c指针变量不能变
const int* const d;	//-> const * const d->靠近*又靠近c,所以d和*d都不能变
int main(void)
{
    //下面5个语句编译器会报错烂截
    *a = 1;
    *b=1;
    c = 1;
    *d = 1;
    d = 1;
    return 0;
}
```



使用const可以提高运行效率：如果用const修饰，编译器会直接把立即数200赋值给变量a,而没有const修饰的则每次都需要读取内存中ABC值。这样用const修饰时，执行效率就比较高（我们知道读取内存是相对慢的操作)

```c
#include <stdio.h>
const int ABC 200;
int main(void)
{
    int a ABC;
    return 0;
}
```

```assembly
//int a ABC;
movl ABC(%rip),%eax	//把内存中ABC的值读到寄存器中去
movl eax,-4(%rbp)

//const int a ABC;
movl $200,-4(%rbp)】
```

> [!tip]
>
> Cost设计的初衷就是为了代蒂宏，消除它的缺点，继承它的优点。
>
> - 编译器处理方式：define宏是在预处理阶段展开；const变量是编译运行阶段使用
> - 安全检查：define宏不做任何类型检查，仅仅是展开；const变量编译阶段会执行类型检查
> - 内存位置：define宏在代码段；const常量可以在静态全局数据段、栈中

### volatile

volatile关键字用于告诉编译器，被声明为volatile的变量的值可能在程序的控制之外发生变化，因此编译器不应该对其进行某些优化。主要用于处理硬件寄存器、中断服务程序和多线程等情况：

下面的代码中，如果buff变量是uart的DR寄存器映射，即使代码中设有地方更新buff的值，也应该告诉编译器不要优化，如果有地方读取buff的值的时候，都需要从内存中读取，因为外部uart会触发更新。

```c
#include <stdio.h>
volatile unsigned char buff;
int main(void)
{
    whi1e(buff){//等待buff不为0
        //do something
    }
    return 0;
}
```



![存储结构](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/4_20_4_25_202412042004317.png)

## 逻辑关键字

在Cpu的眼里很单纯，程序指针(PC)指到哪里就执行哪里，默认情况下顺序往下执行，而逻辑关键字作用就是改变PC指针的指向，这个最基本的思想就构成了程序里各种逻辑设计（条件、选择、跳转、循环)。

![CPU执行过程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/4_20_17_41_202412042017864.png)

1. 条件关键字：if-else
2. 选择关键字：switch-case-default
3. 循环关键字：for、while、do-while
4. 跳转关键字：continue、break、return、goto

# 运算符

程序的本质是逻辑和数据。狭义上运算是指是数学运算，广义的运算是数据处理。在C语言中运算符分为：算数运算、逻辑运算、位运算、赋值更新、内存访问。这里提一个理念：对于运算符的优先级问题，不建议死记硬背，用()万能钥匙来人为定义优先级。不要吝啬用()，()会让你的程序变得清晰可读。

## 算术运算

加减乘除大家从小学就开始学了，所以非常容易理解。这里提一个观念就是：对于CPU来讲，加减法的运行速率比乘除要快。这是因为CPU进行运算主要依赖于算术逻辑单元(ALU)和浮点运算单元(FPU),而ALU的本质是累加器，所以加减法在硬件处理就很快，乘除法有时候编译器会转换成动加减法或者移位操作。随着计算机的发展，现在已经有了"乘法器"的硬件支持，速度上也已经很快。

我们感受一下编译器对乘除法的优化：

- 直接调用"乘法器"进行运算

  ```c
  #include <stdio.h>
  int main()
  {
      int a = 10;
      int b = a * 34;
      printf("b =%d \n",b);
      return 0;
  }
  
  ```

  ```assembly
  movl $16,-8(%rbp）
  movl -8(%rbp),%eax
  imull $34,%eax,%eax	//这里调用乘法器
  movl  %eax,-4(%rbp)
  ```

- 将乘法转换为左移位和加法操作

==mod运算==：表示取余数

一般可用于

1. 循环队列的索引更新

   ```c
   #define LEN 10
   int buff[LEN];
   int index = 0;
   void data_process(int data)
   {
       index = index % LEN;
       buff[index] = data;
       index++;
   }
   ```

   

2. 生成[L,R]区间内的随机数

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <malloc.h>
   #include <time.h>
   int* generate_array(int n,int L,int R)
   {
       int*array = (int*)malloc(sizeof(int) * n);
       srand(time(NULL));
       for (int i = 0;i < n;i++){
           array[i] = rand()%(R - L + 1) + L;
       }
       return array;
   }
   
   void print_array(int* data,int n)
   {
       for (int i = 0;i < n;i++)
       {
           printf("%d "data[i]);
       }
       printf("\n");
   }
   
   int main()
   {
       int *arr = generate_array(10,0,10);
       print_array(arr,10);
       return 0;
   }
   //运行结果
   //04708611035
   ```

3. %还可以做占位符，在格式转换时用

   ```c
   #include <stdio.h>
   int main()
   {
       int a = 10;
       printf("a %d %x %o \n",a,a,a);
       return 0;
   }
   ```

## 位运算

- 左移`<<`:在没有溢出之前，每移动一位等价于乘于2，且移位后右边补0

- 右移`>>`：每移动一位等价于除以2，signed类型和unsigned在右移的时候有差别：

  - 如果是负数，移位后补1，所以负数通过右移永远不会等于0
  - 正数补0

- 与或非

  ```c
  |:1|1=1; 1|0=1; 0|1=1; 0|0=0 ->俗称置位器(set)
  &:1&1=1; 1&0=0; 0&1=0; 0&0=0 ->俗称清零器(c1r)
  ~:~(0x80) = 0x7F -> 按位取反
  ```

- 位异或`^`

## 逻辑运算

- 条件运算：&&（与）||（或）

  > A && B //只要A为假，B就不会被执行
  >
  > A || B //只要B为真，B就不会被执行

- 大小判断：==、<、>、>=、<=、？：

  > [!tip]
  >
  > 建议在使用`==`的时候，可以将数字放在`==`的左边，变量写在右边，这样如果写成`=`编译器就会报错拦截出来

- 条件取反：！

  > 使用取反可以实现二值化处理
  >
  > ```c
  > int a = xx;
  > unsigned int b = 0;
  > b = !!a;	//将非零的值a归零化到1
  > ```
  >
  > 

- 赋值更新：=、+=、-=、&=、|=、++、--

  > 赋值更新的本质是：通过标签（变量名）改变内存中的值

## 内存操作

- 函数访问`()`：用于访问函数名圈定的地址
- 取值操作:
  - `[]`
  - `*`
  - `->`
  - `.`
- 取址操作`&`
- 内存打包`{}`:结构体、联合体、枚举、函数就是典型的通过`{}`进行打包

# 函数视角

希望大家养成一个好的编程"洁癖"：Don't Repeat Yourself.,不要写重复的代码。所以当别人问你函数有什么用时，你就可以这么回答他。C语言是一个面向过程的语言，即面向方法（函数）编程(C++/门ava是面向对象编程)，所以理解和应用函数的重要性不言而喻。

## 函数的世界

![形象比喻](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/5_19_54_32_202412051954990.png)



![数学表示](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/5_19_54_59_202412051954179.png)

### 函数的三大属性

将上面的栗子进行抽象，就可以得到函数的三大属性：输入参数、返回值、函数名

1. 函数名（地址）
2. 输入参数（可多个）
3. 返回值（至多一个）

```c
//输出：函数名：输入
int function(int,char)
{
	//XXX
}
```

> [!tip]
>
> 编译器只要看到有这三大属性，就判定为函数（即使函数体是空的）

函数名本质上是一个==地址标签==，如果知道函数的地址，就可以直接用`()`调过去

### 函数的参数传递

#### 参数传递的本质

调用函数时，需要传入和返回参数。传入和返回的参数过程本质上是：==内存拷贝==。既然是拷贝，那一定存在两个对象：目的地(dst)，源(src)。在C语言中，传入参数时，目的地叫==形参==、源叫==实参==：返回参数时，目的地和源都叫返回值

```c
#include <stdio.h>
int show(int a)	//a是形参，传递过程，就把num的值拷贝给了a
{
    printf("%d \n",a);
    a++;
    return a;	//a此时是返回值
}
void main(void)
{
    int num = 10086;
    int ret =0;
    
    //num是实参
    ret = show(num);//show函数返回a的值会拷贝给ret
}
```

> [!note]
>
> 在执行函数的时候实际上是一个内存拷贝的过程，我们知道每次调用一个函数，在内存中都会为这个函数分配一部分内存空间作为函数栈
>
> 1. 当main调用show的时候，内存中会申请一个栈，用于存函数中的变量，比如这里的num中的10086就会拷贝到show函数的形参a中
> 2. 当对a进行自增完之后，返回的值同样会返回到main函数的栈中
> 3. 对show函数的空间进行销毁
>
> ```mermaid
> block-beta
> 	columns 5
> 	space 	main 	space 	show 	space
> 	num	  	data1["10086"]	space	data2["10086"]	a
>     data1 --> data2
>     ret		data3["10087"]	space data4["10087"]	return
>     data4 --> data3
>     
> 
> ```
>
> 

#### 值传递

==对数据进行隔离和保护==

因为存在拷贝的机制，值传递的时，不会对调用者的源数据进行破坏，所以值传递对数据起到保护和隔离的作用。体会一下下面的例子：

```c
#include <stdio.h>
int show(int a)
{
    a++;
    printf("a in show %d \n",a);
    return a;
}
void main(void)
{
    int num = 10086;	//源数据
    int ret = 0;	//返回值
    ret = show(num);
    printf("a in main %d,ret %d \n",num,ret);
}

//程序运行结果：
//a in show 10087
//a in main 10086,ret 10087
```

另外，大家要养成一个思维习惯：当函数结束时，函数里的局部变量（形参也是局部变量）都会被”销毁"，即使内存中原来的值还存在，但已经不受到系统保护，数据随时可能被覆盖、改写等等。所以当我们要返回一个局部变量的指针时，需要考虑指针指向的内存的生命周期。

> [!tip]
>
> 恢复栈中的局部变量：在分析crash问题的时候，通常需要分析调用栈，可以通过保存之前的栈针地址，查看原来内存的值。可参考：
>
> ```c
> #include <stdio.h>
> static unsigned long addr = 0;
> void show(void)
> {
>     int a = 10086;
>     int b = 10010;
>     asm volatile("movq %rbp,addr(%rip)");
> }
> void main(void)
> {
>     show();
>     int *p = (int*)addr;
>     printf("show bp 0x%1x,a =%d,b =%d In",addr,p[-1],p[-2]);
> }
> //程序运行结果：
> //show bp 0x7fffff2cacbe,a 10010,b 10086
> ```
>
> 

#### 地址传递

地址传递本质上跟值传递一样，只不过这个值有特殊的含义：代表了一个地址编号。地址传递一般用于返回结果和连续空间传递。

```c
#include <stdio.h>
void func(int *a)
{
    *a = 100 * 2;
}
void main(void)
{
    int ret;
    func(&ret);
    printf("ret %d \n",ret);
}
//#程序运行结果：
//ret 200
```

## C与面向对象

C语言是面向过程的语言，但我们可以借助C++/JAVA面向对象思想，实现类似"继承"、"多态"、"封装"、"重载”的功能。从而服务于架构设计。

### C与继承

继承是使用已存在的类作为基础建立新的类的技术。新的类可以增加新的数据类型或者方法，也可以使用父类的功能，但不能选择性的继承父类。通过继承可以很好的复用以前的代码，提高开发效率。

```c
struct parent_class
{
    int width;
    int height;
    void (*setWidth)(struct parent_class* s,int w);
}

struct child_class
{
    struct parent_calss s;
    int (*getArea)(struct parent_class* s);
}
```

### C与封装

封装的概念是指：将抽象的数据和行为（成者方法）相结合，形成一个有机的整体。是面向对象思想的核心，目的是增强安全性和简化编程，使用者不必理解具体的实现细节，通过调用接口使用数据成员。

上节代码中的函数指针就是就做到了封装的概念，通过函数来改变一个对象的值

### C与多态

多态的概念是指：通过父类的指针来调用子类中的方法。作用是把不同的子类对象当做父类来看，屏蔽不同子类之间的差异，用于适应需求的不断变化，本质上是面向抽象编程，在C++中，多态的实现是通过虚函数来实现。体会一下以下这个例子：

```c
#include <stdio.h>
#include <string.h>

struct person
{
    char name[24];
    int age;
    void (*get_name)(struct person *p,char *name);
};

struct usa_person
{
    struct person p;
    void (*get_name)(struct person *p,char *name);
};
static void get_name(struct person *p,char *name)
{
    struct usa_person *child =(struct usa_person *)p;
    child->get_name(p,name);
}
static void get_usa_name(struct person *p,char *name)
{
    sprintf(name,"%s",p->name);
    sprintf(name+strlen(p->name),"%s","-usa");
}
```

### C与重载

重载的概念是指：在同一个范围中声明几个同名函数，但同名函数的形参不同（个数、类型、顺序），一般是用来处理功能类似但是数据类型不同的问题。严格意义上C的编译器不支持重载，但我们有一些方法达到相同的目的。

> [!tip]
>
> `printf`函数就是典型的具有重载效果的函数

#### 可变参数函数

可变参数函数的格式如下：

```c
void va_func(强制参数，可变参数)
```

1. 强制参数有至少有一个，代表以一种规则，由函数定义者自行定义和解析。
2. 可变参数可以有多个，函数定义者和调用者自行决定，实际上可变参数

实际上，可变参数函数的实现依赖的是四个主键：

1. va_list:实际上是一个结构体数组，但是结构体中只有一个元素

   ```c
   va_list = struct _va_list_tag
   {
       unsigned int gp_offset;
       unsigned int fp_offset;
       void *overflow_arg_area;
       void *reg_save_area;
   }[1]
   ```

2. va_start：定位到可变参数的地址

3. va_arg：遍历可变参数

   ```
   type va_arg(va_list ap, type);
   ```

4. va_end：终止va_list遍历

```c
#include <stdio.h>
#include <stdarg.h>
//这里约定n为后面参数的个数（一种规则)
static void va func(int n,...)
{
    va_list ptr;	//定义一个可变参数列表，也就是形参中的...
    va_start(ptr,n);	//传n进去，来计算可变参数的起始位置，n后面的第一位就是可变参数的地址
    while (n--){
        printf("%d \n",va arg(ptr,int));
    }
    va_end(ptr);
}
int main(void)
{
    va_func(1,100);
    va_func(2,200,300);
    return 0;
}
```

> [!important]
>
> 注意：这里使用可变函数列表的时候，需要引入`#include <stdarg.h>`的头文件

#### 回调函数

```c
#include <stdio.h>
#include <stdarg.h>
void exchange_int(void *a,void *b)
{
    int *_a = (int*)a;
    int *_b = (int*)b;
    int tmp = 0;
    tmp = *_a;
    *_a = *_b;
    *b = tmp;
}
void exchange_double(void *a,void *b)
{
    double *_a = (double*)a;
    double *_b = (double*)b;
    double tmp =0;
    tmp =*_a;
    *a = *_b;
    *_b = tmp;
}
typedef void (*func)(void*,void*);
void swap(func exchange,void *a,void *b)
{
    exchange(a,b);
}
typedef void (*func)(void*,void*);
void swap(func exchange,void *a,void *b)
{
    exchange(a,b);
}
void main(void)
{
    int a = 10;
    int b = 20;
    double c = 1.23;
    double d = 4.56;
    swap(exchange_int,&a,&b);
    swap(exchange_double,&c,&d);
    printf("a = %d,b = %d \n",a,b);
    printf("c = %f,d = %f \n",c,d);
}
```

#### 弱连接函数

在驱动设计时经常用到。weak修饰的函数属于弱连接函数，==当前工程中如果有定义跟其相同的函数时，weak修饰的函数会被编译覆盖==，达到overridel的效果。

*==1.c==*

```c
//1.c
#include <stdio.h>
extern void config(void);
void __attribute_((weak)) config(void)
{
 printf("config default\n");
}
void main(void)
{
 config();
}
```

*==2.c==*

```c
#include <stdio.h>
void config(void)
{
    printf("config one \n");
}
```

最终执行的效果为：

```
config one
```

## Solid设计原则

solid设计是面向对象的原则，是由Gang of Four(四人帮)，即Erich Gamma,Richard Helm,Ralph Johnson&John Vlissides四人的《设计模式》
(1995年出版)是第一次将设计模式提升到理论高度，并将之规范化。

solid有五个设计原则：

1. 单一职责原则：一个类只负责一件事情。在C语言中，就是一个结构体在设计和封装的时候只负责一个事情。

2. 开闭原则：软件中的对象（类、模块、函数等）应该满足对于拓展是开放的，对修改是封闭的。

3. 里氏替换原则：简单来讲就是子类可以去拓展父类的功能，但不能改变父类原有的功能。技术上，所有引用基
   类的地方必须能够透明的使用其子类的对象。价值：继承一定程度上破坏了封装，此原则侧用来包含封装性。

4. 接口隔离原则

   定义：不应该强迫用户端实现一个它用不上的接口。具体两个的例子：

   1. 抽象了面积和体积的两个接口，客户瑞在使用的这个接口的时候会发现，对于正方形来说设有体积，所以实现体积的函数对它来说就是冗余的。

   2. 抽象了器件初始化和读取数据的接口。对于LED灯来说，一般都是控制型，没有必要实现一个虬ED数据的接口，所以对LED来说，读取数据就是冗余的。

      ![接口隔离原则](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/9_19_56_35_202412091956560.png)

      上图中的红色部分就是冗余设计

5. 依赖倒置原则

   高层次模块不应该依赖低层次的模块，实际上是面向抽象编程思想，这样可以降低代码间的耦合度，使其看扩展、易维护。举一个驱动的
   列子：荏这个架构设计中，IMU有很多公共的业务逻辑：初始化、读取数据、数据校准等等，这些业务逻辑不应该耦合具体驱动的接口，而是
   依赖于抽象接口：同时驱动与驱动之间也不能互相调用，而是调用抽象接口。后续的器件增加拓展，只需要修改驱动部分，抽象层都不会改
   动。

   ![依赖倒置原则](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/9_19_59_5_202412091959280.png)

# 内存视角

前面在学习数据类型关键字的时候我们了解到，定义一个变量的本质是在内存中圈定一块特定大小的内存。本章将会对内存的位置、生命
周期、操作权限等做更多的深讨。

![内存视角](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/10_19_21_41_202412101921288.png)

## 内存分布

内存分布分成静态区和动态区，静态区在编译时已经决定了内存分配大小，动态区是运行时分配。

![内存分布](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/10_19_23_9_202412101923962.png)

```c
#include<stdio.h>
#include<stdlib.h>

int global_inited = 10;	//全局初始化变量
int global_uninited;	//全局未初始化变量

int main(void)
{
    int local_var = 20;	//局部变量
    char* str = "Hello";	//字符串常量
    static int static_var=30;	//静态变量
    int* heap_var = (int*)malloc(sizeof(int));	//动态分配的内存
    
    printf("address of code:%p\n",main);	//将会打印main函数的地址，也就是代码段的首地址，0x55f7c567f1a9
    printf("address of str:%p\n",str);	//打印出来的地址实际上是只读数据段，因为是一个常量的地址，0x55f7c5680008
    printf("address of global inited %p\n",(void *)&global_inited );	//全局数据初始化段，0x55f7c5682010
    printf("address of static var:%p\n",(void *)&static_var);	//	全局数据未初始化段，0x55f7c5682014
    printf("address of global uninited:%p\n",(void *)&global_uninited);	//全局数据未初始化段，0x55f7c568201c
    printf("address of heap var %p\n",(void *)heap_var);	//该指针位于堆空间，0x55f7c71392a0
    printf("address of local var %p\n",(void *)&local_var);	//该便变量位于栈空间，0x7ffc5ef3b5f4
    
    //释放动态分配的内存
    free(heap_var);
    return 0;
}
```



> [!tip]
>
> 函数名实际上是一个指针常量，所以可以通过函数名来打印地址
>
> 堆和栈的生长空间是相反的

### 代码段

代码段只能读不可以写，写会触发Segmentation fault(core dumped),俗称"段错误"。在整个程序生命周期内合法有效。

```c
#include <stdio.h>

void func(void)
{
    printf("hello func \n");
}

int main(void)
{
    printf("func addr %p \n",func);
    void (*p)(void) = func;
    //try to access
    p();
    
    int *p1 =  (int *)func;
    //try to read
    printf("func addr %d \n",*p1);
    //try to write
    *p1 = 1;
    
    return 0;
}
```



### 只读数据段

只读数据段存放字符传常量，地址比代码段更高。权限是：只能读不可以写，写会触发Segmentation fault(core dumped),在整个程序生
命周期内合法有效。

```c
#include <stdio.h>
int main(void)
{
    printf("%s read only data addr:%p \n","hello world","hello world");	//hello world read only data addr:0x56014766f008
    char *s = "hallo world";
    s[1] = 'e';//try to write rodata Segmentation，Segmentation fault (core dumped)
    return 0;
}
```



### 全局数据段

全局变量和局部static修饰的变量会放在全局数据段(data/bbs),允许读写，在整个程序生命周期内合法有效。

```c
#include <stdio.h>
int a = 10;
int b;
int func(void)
{
    static int d 10;
    b++;
    return ++d;
}
int main(void)
{
    static int c;
    printf("%p %p,%p \n",&a,&b,&c);
    //try to write data Segmentation
    a=20;
    b=30;
    c=40;
    printf("a =%d b =%d,c %d \n",a,b,c);
    printf("b %dd =%d,d %d\n",b,func(),func());
    return 0;
}
//运行结果：
//0x5556e9e9b010,0x5556e9e9b020,0x5556e9e9b81c
//a=20,b=30,c=40
//b=32,d=12,d=11
```

### 堆空间

堆空间在运行时由程序员来分配(malloc)和释放(free)，可读可写，生命周期是程序员来决定。值得注意的是，如果在一个函数中分配
堆内存，函数执行结束后māoc空间不会被释放，如果不手动进行释放的话，会造成内存泄漏。

```c
#include <stdio.h>
#include <stdlib.h>
void func(void)
{
    int* heap_var = (int*)malloc(sizeof(int));//动态分配的内存，单位是B,默认反馈的类型是void*
    if (!heap_var){
        return;
    }
    //try to write heap segmentation
    *heap_var = 10;
    printf("heap addr =%p heap_var %d\n",heap_var,*heap_var);
    free(heap_var);//如果没有free动作，heap_var指向的内存就存在泄漏：不会再使用但又不能分配给别人
}
int main(void)
{
    func();
    return 0;
}
//运行结果：
// heap addr 0x55bf2b8db2a0,heap_var 10
```

### 栈空间

栈空间指的是在函数运行时的上下文分配的空间，可读可写，生命周期在函数执行结束后结束。看下面的一个例子：buff和s都是局部变量，系统会给buff分配sizeof("hello world")栈空间，而s的栈空间是4byte,它指向只读数据段中的字符串"hello world"的地址编号。所以这里返回buff就会有问题，返回指针没有问题。

```c
#include <stdio.h>
char*func(void)
{
    char *s = "hello world";	//字符串会放在只读数据区，所以该函数返回指针是没问题的
    char buff[] = "hello world";	//但是使用数组，将会从只读数据区中拷贝字符串到栈中，此时返回这个数组就会有问题了，因为函数运行结束就要释放栈
    printf("func:%s \n",buff);
    return s;
}
void main(void)
{
    char *p = func();
    printf("main:%s \n",p);
}
```

![内存分布图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/10_20_27_31_202412102027864.png)

## 内存溢出

内存溢出指的程序运行过程中，访问超过其分配空间范围的内存区域。

### 栈溢出

- 递归函数：递归函数如果没有正常的退出条件，最后会把整个栈空间消耗殆尽，最后出现段错误。

  ```c
  #include <stdio.h>
  void try_overflow(void)
  {
      int a = 10;
      try_overflow();
  }
  void main(void)
  {
      try_overflow();
  }
  ```

  ```assembly
  #运行结果：
  Segmentation fault (core dumped)
  #汇编
  try_overflow:
  .LFBO:
  	endbr64
  	pushq	%rbp
  	movq	%rsp,%rbp
  	subq	$16,%rsp	#分配栈空间的内存
  	movl	$10,-4(%rbp)
  	call	try_overf1ow	#反复调用
  	nop
  	leave
  	ret
  ```

- 栈缓冲区溢出

  ```c
  #include <stdio.h>
  #include <string.h>
  #define N 10
  void main(void)
  {
      char arr[N];
      char arr1[] = "hello world";
      strcpy(arr,arr1);	//工程上建议不要用strcpy,用strncpy,因为原理上是使用\0作为结束符的
      //strncpy(arr,arr1,N);
      printf("arr:%s \n",arr);
      printf("arr1:%s \n",arr1);
  }
  //strcpy.运行结果
  //arr:hello world
  //arr1:d
  //strncpy.运行结果
  //arr:he1 1o wor1hel1 o world/原因是arr长度不够，copy N个字符中缺少结束符'\0'
  //arr1:hello world
  ```

  ### 堆溢出

  ```c
  #include <stdio.h>
  #include <string.h>
  #include <stdlib.h>
  #define N 5
  void main(void)
  {
      char *str = (char*)malloc(sizeof(char)*N);
      char* strl = (char*)malloc(sizeof(char)*N);
      strcpy(str1,"hello world");
      strcpy(str,"hello world");
      printf("str:%s \n",str);
      printf("str1:%s \n",strl);
  }
  ```

# 指针

前面在介绍指针类型时，已经知道指针变量圈定的内存大小跟编译器有关或者说跟CPU的地址总线有关。在32位的系统中，指针类型占用4byte,在64位的系统中，指针类型占用8byte。指针变量的值有特殊意义，代表了一个地址编号，本章讲重点讲指针访问内存的规则和技巧。另外，希望大家对这两个概念：指针和指针变量，有清晰的区分。我们一般说指针，代表就是这个指针变量的值，即内存地址；指针变量就是一个变量，用于存放内存地址。

![指针访问](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/10_20_52_43_202412102052192.png)

##   指针访问内存

> [!note]
>
> 要访问指针指向的地址，那就必然需要回答两个问题：
>
> 1. 内存的可读可写性是什么？
> 2. 内存的访问规则是什么？

### 指针变量初始化

大家要养成思维习惯：每当看到一个指针变量，要对其值的合法性保持敬畏。

```c
#include <stdio.h>
#include <stdlib.h>
int b=  20;
void main(void)
{
    int a = 10;
    int *p1 = &a;
    int *p2 = &b;
    int *p3 = (int*)malloc(sizeof(int));
    if (!p3){
        return;
    }

    *p3 = 30;
    int *p;
    p = (int*)100;	//从复制语法上可行，但内存不可读，不可写
    printf("*p1=%d,*p2=%d,*p3=%d\n",*p1,*(&b),p3[0]);
    printf("p %p \n",p);
    printf("*p %d \n",*p);
    free(p3);
}
//运行结果：
//*p1=10,*p2=20,*p3=30
//p=0x64
//Segmentation fault (core dumped)
```

### 空指针和野指针

空指针很好理解，就是值为0(NULL)的指针变量，如果访问了0地址就会出现非法访问的错误。野指针是指值为非法地址的指针变量。野
指针在实际工程中有很大的危害，会造成代码的稳定问题（不可预知的bug)。要形成free后，把指针变量置空(NULL)的习惯。

空指针的例子：

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void main(void)
{
    int *ptr = NULL;
    printf("*p = %d \n",*ptr);	//访问空地址将会触发segmentation fault
    ptr = (int*)malloc(sizeof(int));
    *ptn = 102;
    printf("*p %d \n",*ptr);
    free(ptr);
}
```

野指针的例子：

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void main(void)
{
    int *ptr = (int*)malloc(sizeof(int));
    free(ptr);	//当使用free之后，仅仅只是把标志位置为未使用，并不会将ptr的内容改为NULL
    //ptr = NULL,
    if (NULL == !ptr)
        *ptr=102;	//将会进入这行代码

    int *p = (int*)malloc(sizeof(int));
    printf("*p %d \n",*p);
    printf("ptr %p \n",ptr);	//和下面的p将会打印同样的地址，因为在堆里进行申请内存，刚刚又释放了一个int类型的指针，所以现在
    printf("p %p\n",p);			//申请的地址就是一致的
    //*ptr=101;
}
```



### 指针访问内存规则

#### 标准数据类型指针

我们前面学过几种指针访问内存的方式：* p、p[x]、p->X。那每次访问的大小是多少呢？我们知道指针变量的定义的形式是：数据类型*p;所以每次访问的大小由前面修饰的数据类型决定。

```c
#include <stdio.h>
void main(void)
{
    int a = 0x12345678;
    int *p = &a;
    char *p1 = (char*)&a;
    printf("*p = 0x%x \n",*p);
    printf("*p1 = 0x%x In",*p1);
    printf("p1[0] = 0x%x \n",p1[0]);
    printf("p1[1] = 0x%x\n",p1[1]);
    printf("p1[2] = 0x%x\n",p1[2]);
    printf("p1[3] = 0x%x\n",p1[3]);

```

运行结果为：

```
*p=0x12345678
*p1=0x78
p1[9]=0x78
p1[1]=0x56
p1[2]=0x34
p1[3]=0x12
```



> [!tip]
>
> 不管大端还是小端，指针p和p1都是指向低地址。

#### 连续空间类型指针



```c
#include <stdio.h>

//会进行4字节对齐
struct abc
{
    int a;
    int b;
    char c;
};

int arr[3]={1,2,3};

int main(void)
{
    struct abc data =
    {
        .a=1,
        .b=2,
        .C=3,
    };
    printf("size of struct abc %ld\n",sizeof(struct abc));
    struct abc *p = &data;
    printf("a =%d,b %d,c =%d\n",p->a,p->b,p->c);
    int*p1 = (int *)&data;
    printf("struct p1[0]%d \n",p1[0]);
    printf("struct p1[1]%d \n",p1[1]);
    printf("struct p1[2]%d \n",p1[2]);	//将不会展示正确的数值
    int*p2 = arr;
    printf("arr p2[0]=%d \n",p2[0]);
    printf("arr p2[1]=%d \n",p2[1]);
    printf("arr p2[2]%d \n",p2[2]);
    return 0;
}
```

可以通过结构体成员的地址从而获取结构体的地址，如下代码，假设现在有一个结构体abc，先已经知道c的地址，求这个结构体的地址？

首先先设这个结构体的体的偏移地址是`x`，那么则有：

```c
&(struct abc*)x->c - x = sizeof(struct abc)
```



```c
//Linux container_of宏的思想
#define offsetof(TYPE,MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
#define container_of(ptr,type,member) ({	\
	const typeof(((type *)0)->member)*__mptr = (ptr);	\
	(type *)(char *)__mptr - offsetof(type,member));})
#include <stdio.h>

struct abc
{
    int a;
    int b;
    char c;
}

void find_struct(int *member)
{
    unsigned long offset = 0;
    struct abc *p = NULL;
    offset =  (unsigned long)&((struct abc*)0)->b;	//这里使用0地址就是为了方便计算b相对于a的offset
    printf("member offset:%ld \n",offset);

    p = (struct abc *)((char*)member - offset);	//这里使用char*是为了取最低位
    printf("a:%d,b:%d c:%d \n",p->a,p->b,p->c);
}
int main(void)
{
    struct abc data =
    {
        .a=1,
        .b=2,
        .c=3,
    }
    printf("size of struct abc =%ld\n",sizeof(struct abc));
    struct abc *p = &data;
    printf("a %d,b %d,c=%d\n",p->a,p->b,p->c);
    
    find struct(&data.b);
    return 0;
}
```

#### 函数类型的指针

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
void (*show)(void);	//编译器会先向右看再向左看
void func(void)
{
    printf("func call \n");
}
int main(void)
{
    show - func;
    func();
    return 0;
}
```

> [!note]
>
> 建议使用`typedef`来重命名函数，使代码通俗易懂

## 指针运算

### 算术运算

指针变量本质也是一个变量，所以进行算数运算语法上是可以的。但是实际的工程应用中，指针变量的乘除法没有意义，更多的是加减法。总的来说，+运算符用于指针的算术运算，允许将指针移动到任意偏移位置，而++运算符是+运算符的特例，它递增指针指向的位置，移动一个对象的大小。看一个例子：

```c
#include <stdio.h>
int main(void)
{
    int array[] = {1,2,3,4,5};
    int *p = array;
    p = p+2;//p[2]
    printf("*p %d \n",*p);
    p++;//p[3]
    printf("*p = %d \n",*p);
    return 0;
}
//运行结果
//*p=3
//*p=4
```

### 逻辑运算

逻辑运算中，判断两个指针是否相等比较常用，其他的不常用。另外，只有两个相同类型的指针比较才有意义，一般编译也会包警告，如果类型不同。

```c
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    int *p = (int*)malloc(sizeof(int));
    if (p == NULL)
        return -1;
    char *p1 NULL;
    if (p1 == p){	//这里将会报警告，因为不同类型的指针不能比较
        //do something
    }
    free(p);
    return 0;
}
```

## 多级指针

多级指针本质上也是一个指针，也需要一个指针变量来存放。这个指针变量的内存大小跟一级指针一样，跟系统有关。

```c
int **p
```

1. `*p`第一`*`决定变量p是一个指针变量
2. `**p`第二`*`决定指针p访问内存规则是以==指针类型==进行访问，所以`*p`的值还是一个指针（地址）
3. int决定指针(`*p`)访问内存规则是以int类型进行访问，所以`*(*p)`是int类型

![32位系统下多级指针访问](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/11_20_53_43_202412112053165.png)

```c
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    int a = 100;
    int *p = &a;
    int **pp = &p;
    printf("**pp %d,*pp[0]=%d \n",**pp,*pp[0]);
    printf("size of pp %ld \n",sizeof(pp));
    return 0;
}
```

### 指针的地址传值

```c
#include <stdio.h>
#include <stdlib.h>
void funcl(char p)
{
    printf("funcl:%s \n",p);
    p = "hello linux";
}
void func2(char **ss)
{
    printf("func2:%s \n",*ss);
    ǐss="hello linux";
}
int main(void)
{
    char *s = "hello world";
    func1(s);
    //func2(&s);
    printf("s %s \n",s);
    return 0;
}
```



### 无序变有序

多级指针用于物理无序映射到逻辑有序的数据结构设计：

```c
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    char *arr[3] = {"welcome","to","linux"};
    char **s = arr;
    printf("%s:%p\n",s[e],s[e]);
    printf("%s:%p\n",s[1],s[1]);
    printf("%s:%p\n",s[2],s[3]);
    printf("&s[0]:%p\n",&s[0]);
    printf("&s[1]:%p\n",&s[1]);
    printf("&s[2]:%p\n",&s[2]);
    return 0;
}
```

![多级指针访问](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/11_21_5_22_202412112105963.png)

# 编译器指令

## pragma

在 **C语言** 中，`#pragma` 是用于为编译器提供特定指令的预处理指令。具体的使用和支持的 `#pragma` 类型取决于 **编译器**（如 GCC、IAR、Keil、MSVC 等）。

### 基本语法

```c
#pragma <指令>  // 根据指令内容执行不同的编译行为
```

如果编译器遇到它不支持的 `#pragma` 指令，通常会**忽略**它，而不会产生错误。这使得 `#pragma` 在处理平台相关编译选项时非常灵活。

### 常见使用场景

#### `once`
防止头文件重复包含，是一种常见的替代 **include guards** 的方式。

```c
#pragma once

// 头文件内容
void myFunction(void);
```

- **作用**：确保头文件在一次编译过程中只会被包含一次。
- **支持**：大部分现代编译器（如 GCC、Clang、MSVC、IAR）。

#### `pack`
用于**结构体对齐**控制。

```c
#pragma pack(4)  // 设置为1字节对齐
typedef struct {
    char a;
    int b;
} PackedStruct;
#pragma pack()  // 恢复原始对齐

```

- **作用**：确保结构体字段按照指定的字节数对齐。`push` 和 `pop` 用于保存和恢复对齐状态。
- **应用**：减少内存浪费，或在与硬件设备通信时保持字节一致性。
- **支持**：GCC、MSVC、IAR 等大部分编译器。

#### `location` （IAR 特有）
用于**指定变量或函数的内存地址**。

```c
#pragma location=0x20001000  // 指定变量存放位置
volatile int myVar = 0;
```

- **作用**：将变量或函数放置在特定的内存地址中。这在嵌入式开发中很常见，比如初始化存储在特定 RAM/ROM 地址的配置数据。
- **支持**：IAR 编译器。

#### `warning`
控制**编译器警告**的显示和抑制。适用于 MSVC 或 GCC。

```c
#pragma warning(push)       // 保存当前警告状态
#pragma warning(disable: 4996)  // 禁用警告 4996（示例）
void deprecatedFunction();  // 调用旧函数
#pragma warning(pop)        // 恢复警告状态
```

- **作用**：用于临时禁用某些特定的警告。
- **支持**：MSVC，GCC 使用 `#pragma GCC diagnostic`。

#### `diag_suppress` / `diag_default` / `diag_error`（IAR）
用于控制 IAR 编译器的警告或错误。

```c
#pragma diag_suppress=Pe940  // 禁用警告 Pe940
void someFunction() {
    // 代码内容
}
#pragma diag_default=Pe940  // 恢复默认警告设置
```

#### `optimize`（GCC/MSVC）
用于控制**代码优化**级别。

```c
#pragma optimize("", off)  // 禁用优化
void criticalFunction() {
    // 关键代码
}
#pragma optimize("", on)   // 启用优化
```

- **作用**：临时启用或禁用特定区域的优化。
- **支持**：GCC、MSVC。

#### `section`（IAR）
用于将代码或数据放入指定的段中，常用于嵌入式系统的内存管理。

```c
#pragma section = "MY_SECTION"  // 声明段
const int myArray[10] @ "MY_SECTION";  // 将数组放入指定段
```

- **作用**：将数据或代码放入特定段，方便嵌入式开发中的内存分区。
- **支持**：IAR 等嵌入式编译器。

## attribute

在 C 语言中，`__attribute__` 是 GCC 编译器提供的一种扩展，用于指定函数或变量的特性。它可以帮助编译器优化代码或改变函数的行为。

### 函数属性

####  `noreturn`
- **作用**：告知编译器该函数不会返回，通常用于 `exit()`、`abort()` 等函数。
- **用法**：
  ```c
  void my_exit() __attribute__((noreturn));
  
  void my_exit() {
      // 终止程序
      exit(0);
  }
  ```

#### `pure`
- **作用**：表示函数是纯函数，不会产生副作用，返回值只依赖于输入参数。
- **用法**：
  ```c
  int square(int x) __attribute__((pure));
  
  int square(int x) {
      return x * x;
  }
  ```

#### `malloc`
- **作用**：指示返回的指针指向未初始化的内存，告诉编译器该函数会分配内存。
- **用法**：
  ```c
  void* my_malloc(size_t size) __attribute__((malloc));
  
  void* my_malloc(size_t size) {
      return malloc(size);
  }
  ```

### 变量属性

####  `section`
- **作用**：将变量放置在特定的内存段中，常用于嵌入式开发。
- **用法**：
  ```c
  int my_var __attribute__((section(".my_section")));
  ```

####  `aligned`
- **作用**：设置变量的对齐方式，确保变量在特定的内存边界上。
- **用法**：
  ```c
  int my_array[10] __attribute__((aligned(16)));
  ```

###  参数属性

###  `unused`
- **作用**：防止编译器对未使用的参数发出警告。
- **用法**：
  ```c
  void my_function(int unused_param __attribute__((unused))) {
      // 该参数可以不使用
  }
  ```

### 对齐属性

####  结构体对齐
- **作用**：确保结构体的某些成员或整个结构体的对齐方式。
- **用法**：
  ```c
  struct my_struct {
      char c;
      int i __attribute__((aligned(4))); // 确保 i 在 4 字节边界上
  };
  ```

#### 性能优化

##### `hot` 和 `cold`
- **作用**：用于提示编译器对频繁执行的代码进行优化（`hot`）或对不常执行的代码进行优化（`cold`）。
- **用法**：
  ```c
  void my_hot_function() __attribute__((hot));
  void my_cold_function() __attribute__((cold));
  
  void my_hot_function() {
      // 频繁调用的代码
  }
  
  void my_cold_function() {
      // 不常调用的代码
  }
  ```
