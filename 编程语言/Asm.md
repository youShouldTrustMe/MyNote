# 参考链接

[汇编语言入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/01/assembly-language-primer.html)

[汇编语言 教程 | 参考手册 (cankaoshouce.com)](https://cankaoshouce.com/assembly/assembly-course.html)

[Guide to x86 Assembly](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)

[GCC-Inline-Assembly-HOWTO --- GCC-Inline-Assembly-HOWTO](https://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)

[译：GCC内联汇编入门 - 简书 (jianshu.com)](https://www.jianshu.com/p/10e8a7b4f980)

[Arm A-profile A32/T32 Instruction Set Architecture](https://developer.arm.com/documentation/ddi0597/2024-09?lang=en)

# 基础知识

## 引言

我们知道，CPU 只负责计算，本身不具备智能。你输入一条指令（instruction），它就运行一次，然后停下来，等待下一条指令。

这些指令都是二进制的，称为操作码（opcode），比如加法指令就是`00000011`。[编译器](https://www.ruanyifeng.com/blog/2014/11/compiler.html)的作用，就是将高级语言写好的程序，翻译成一条条操作码。

对于人类来说，二进制程序是不可读的，根本看不出来机器干了什么。为了解决可读性的问题，以及偶尔的编辑需求，就诞生了汇编语言。

**汇编语言是二进制指令的文本形式**，与指令是一一对应的关系。比如，加法指令`00000011`写成汇编语言就是 ADD。只要还原成二进制，汇编语言就可以被 CPU 直接执行，所以它是最底层的低级语言。

> [!important]
>
> 以下教程开发环境为NAsm，使用Inte格式。
>
> - **NASM** 更适合跨平台、轻量级的汇编开发，语法简洁，灵活性高。
>
> - **MASM** 则更适合 Windows 平台上的专业开发，功能丰富，语法较复杂，集成度高，适合开发大规模的 Windows 应用程序和驱动。
>
>   



> [!TIP]
>
> 汇编语法主要有两大派系：AT&T语法 和 Intel语法。
>
> GAS (GNU Assembler) 编译器默认是基于AT&T语法；[MASM](https://so.csdn.net/so/search?q=MASM&spm=1001.2101.3001.7020)、NASM等编译器默认基于Intel语法。
>
> 需要说明的是，GAS汇编器除了支持AT&T语法之外，自己也定义了一些额外的`directives`，用于辅助完成汇编操作。关于GAS汇编器及其语法可以参考GAS的官方文档：https://sourceware.org/binutils/docs/as/

## 寄存器

CPU 本身只负责运算，不负责储存数据。数据一般都储存在内存之中，CPU 要用的时候就去内存读写数据。但是，CPU 的运算速度远高于内存的读写速度，为了避免被拖慢，CPU 都自带一级缓存和二级缓存。基本上，CPU 缓存可以看作是读写速度较快的内存。

但是，CPU 缓存还是不够快，另外数据在缓存里面的地址是不固定的，CPU 每次读写都要寻址也会拖慢速度。因此，除了缓存之外，CPU 还自带了寄存器（register），用来储存最常用的数据。也就是说，那些最频繁读写的数据（比如循环变量），都会放在寄存器里面，CPU 优先读写寄存器，再由寄存器跟内存交换数据。

![计算机中的存储结构](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012206.png)

> [!TIP]
>
> 寄存器不依靠地址区分数据，而依靠名称。每一个寄存器都有自己的名称，我们告诉 CPU 去具体的哪一个寄存器拿数据，这样的速度是最快的。有人比喻寄存器是 CPU 的零级缓存。

早期的 x86 CPU 只有8个寄存器，而且每个都有不同的用途。现在的寄存器已经有100多个了，都变成通用寄存器，不特别指定用途了，但是早期寄存器的名字都被保存了下来。

### 处理器寄存器

**IA-32** 体系结构中有 **10** 个 32 位和 **6** 个 16 位处理器寄存器。

寄存器分为三类：

1. 通用寄存器
   1. 数据寄存器
   2. 指针寄存器
   3. 索引寄存器
   4. 数据寄存器
2. 控制寄存器
3. 段寄存器

### 数据寄存器

**4** 个 **32** 位数据寄存器用于算术、逻辑和其他操作。这些 **32** 位寄存器可以用 **3** 种方式使用：

1. 作为完整的 32 位数据寄存器：**EAX**、**EBX**、**ECX**、**EDX**。
2. 32 位寄存器的下半部分可以用作 4 个 16 位数据寄存器：**AX**、**BX**、**CX** 和 **DX**。
3. 上述四个 16 位寄存器的下半部和上半部可以用作八个 8 位数据寄存器：AH、AL、BH、BL、CH、CL、DH和DL。

![数据寄存器](https://cankaoshouce.com/znadmin/md/5124/0.jpg)

其中一些数据寄存器在算术运算中有特定用途。

- **AX 是主要的累加器**; 它用于输入/输出和大多数算术指令。例如，在乘法运算中，根据操作数的大小，将一个操作数存储在 **EAX** 或 **AX** 或 **AL** 寄存器中。
- **BX 称为基址寄存器**，因为它可以用于索引寻址。
- **CX 称为计数寄存器**，因为 **ECX**，**CX** 寄存器在迭代操作中存储循环计数。
- **DX 称为数据寄存器**。它也用于输入/输出操作。它还与 **AX** 寄存器以及 **DX** 一起使用，用于涉及大数值的乘法和除法运算。

### 指针寄存器

指针寄存器是 32 位 **EIP**，**ESP** 和 **EBP** 寄存器以及相应的 16 位右部分 **IP**，**SP** 和 **BP**。

指针寄存器分为三类：

- **指令指针（IP）** -16 位 **IP** 寄存器存储要执行的下一条指令的偏移地址。与 CS 寄存器关联的 IP（作为 CS:IP）给出了代码段中当前指令的完整地址。
- **堆栈指针（SP）** -16 位 **SP** 寄存器提供程序堆栈内的偏移值。与 SS 寄存器（SS:SP）关联的 SP 是指程序堆栈中数据或地址的当前位置。
- **基本指针（BP）** -16 位 **BP** 寄存器主要帮助参考传递给子例程的参数变量。SS 寄存器中的地址与 BP 中的偏移量相结合，以获取参数的位置。BP 也可以与 DI 和 SI 组合用作特殊寻址的基址寄存器。

![指针寄存器](https://cankaoshouce.com/znadmin/md/5124/1.jpg)

### 索引寄存器

32 位索引寄存器 **ESI** 和 **EDI** 及其最右边的 16 位部分。**SI** 和 **DI** 用于索引寻址，有时用于加法和减法。

索引指针分为两种：

- 源索引（SI） -用作字符串操作的源索引。
- 目标索引（DI） -用作字符串操作的目标索引。

![索引寄存器](https://cankaoshouce.com/znadmin/md/5124/2.jpg)

### 控制寄存器

将 32 位指令指针寄存器和 32 位标志寄存器组合起来视为控制寄存器。许多指令涉及比较和数学计算，并更改标志的状态，而其他一些条件指令则测试这些状态标志的值，以将控制流带到其他位置。

通用标志位是：

- **溢出标志（OF）**：指示有符号算术运算后数据的高阶位（最左位）的溢出。
- **方向标记（DF）**：它确定向左或向右移动或比较字符串数据的方向。**DF** 值为 0 时，字符串操作为从左至右的方向；当 **DF** 值为 1 时，字符串操作为从右至左的方向。
- **中断标志（IF）**：确定是否忽略或处理外部中断（例如键盘输入等）。当值为 0 时，它禁用外部中断，而当值为 1 时，它使能中断。
- **陷阱标志（TF）**：允许在单步模式下设置处理器的操作。我们使用的 `DEBUG` 程序设置了陷阱标志，因此我们可以一次逐步执行一条指令。
- **符号标志（SF）**：显示算术运算结果的符号。根据算术运算后数据项的符号设置此标志。该符号由最左位的高位指示。正结果将 **SF** 的值清除为 0，负结果将其设置为 1。
- **零标志（ZF）**：指示算术或比较运算的结果。非零结果将零标志清零，零结果将其清零。
- **辅助进位标志（AF）**：包含经过算术运算后从位 3 到位 4 的进位；用于专业算术。当1字节算术运算引起从第 3 位到第 4 位的进位时，将设置 **AF**。
- **奇偶校验标志（PF）**：指示从算术运算获得的结果中 1 位的总数。偶数个 1 位将奇偶校验标志清为 0，奇数个 1 位将奇偶校验标志清为 1。
- **进位标志（CF）**：在算术运算后，它包含一个高位（最左边）的 0 或 1 进位。它还存储移位或旋转操作的最后一位的内容。

| 标志 |      |      |      |      | O    | D    | I    | T    | S    | Z    |      | A    |      | P    |      | C    |
| :--: | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 位号 | 15   | 14   | 13   | 12   | 11   | 10   | 9    | 8    | 7    | 6    | 5    | 4    | 3    | 2    | 1    | 0    |

### 段寄存器

段是程序中定义的特定区域，用于包含数据，代码和堆栈。有 3 个主要部分：

- **代码段**：它包含所有要执行的指令。16 位代码段寄存器或 CS 寄存器存储代码段的起始地址。
- **数据段**：它包含数据，常量和工作区。16 位数据段寄存器或 DS 寄存器存储数据段的起始地址。
- **堆栈段**：它包含数据或过程或子例程的返回地址。它被实现为 "堆栈" 数据结构。堆栈段寄存器或 SS 寄存器存储堆栈的起始地址。

除了 DS，CS 和 SS 寄存器外，还有其他段寄存器 - **ES**（额外段），**FS** 和 **GS**，它们提供了用于存储数据的其他段。在汇编语言中，程序需要访问存储器位置。段中的所有存储位置都相对于段的起始地址。段的起始地址可以是 16 或十六进制的整数，因此，所有此类存储地址中最右边的十六进制数字为 0，通常不存储在段寄存器中。

段寄存器存储段的起始地址。为了获得数据或指令在段中的确切位置，需要一个偏移值（或位移）。为了引用段中的任何存储位置，处理器将段寄存器中的段地址与该位置的偏移值进行组合。

栗子：

```assembly
section .text
   global _start         ;必须声明链接器(gcc)
_start:          ;告诉链接器入口点
   mov  edx,len  ;消息长度
   mov  ecx,msg  ;消息内容
   mov  ebx,1    ;文件描述 (stdout)
   mov  eax,4    ;系统调用编号 (sys_write)
   int  0x80     ;调用内核
   mov  edx,9    ;消息长度
   mov  ecx,s2   ;消息内容
   mov  ebx,1    ;文件描述 (stdout)
   mov  eax,4    ;系统调用编号 (sys_write)
   int  0x80     ;调用内核
   mov  eax,1    ;系统调用编号 (sys_write)
   int  0x80     ;调用内核
section .data
	msg db 'Displaying 9 stars',0xa ;消息内容
	len equ $ - msg  ;消息长度
	s2 times 9 db '*'
```

> [!tIP]
>
> 我们常常看到 32位 CPU、64位 CPU 这样的名称，其实指的就是寄存器的大小。32 位 CPU 的寄存器大小就是4个字节。

## 堆（Heap）

寄存器只能存放很少量的数据，大多数时候，CPU 要指挥寄存器，直接跟内存交换数据。所以，除了寄存器，还必须了解内存怎么储存数据。

程序运行的时候，操作系统会给它分配一段内存，用来储存程序和运行产生的数据。这段内存有起始地址和结束地址，比如从`0x1000`到`0x8000`，起始地址是较小的那个地址，结束地址是较大的那个地址。

![堆](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012208.png)

程序运行过程中，对于==动态的内存占用请求==（比如新建对象，或者使用`malloc`命令），系统就会从预先分配好的那段内存之中，划出一部分给用户，具体规则是从起始地址开始划分（实际上，起始地址会有一段静态数据，这里忽略）。举例来说，用户要求得到10个字节内存，那么从起始地址`0x1000`开始给他分配，一直分配到地址`0x100A`，如果再要求得到22个字节，那么就分配到`0x1020`。

![从堆中申请空间](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012209.png)

这种因为用户主动请求而划分出来的内存区域，叫做 Heap（堆）。它由起始地址开始，从低位（地址）向高位（地址）增长。==Heap 的一个重要特点就是不会自动消失，必须手动释放，或者由垃圾回收机制来回收==。

## 栈（Stack）

除了 Heap 以外，其他的内存占用叫做 Stack（栈）。简单说，Stack 是由于函数运行而临时占用的内存区域。

![栈](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012210.png)

现假设有以下函数：

```c
int main() {
   int a = 2;
   int b = 3;
}
```

上面代码中，系统开始执行`main`函数时，会为它在内存里面建立一个帧（frame），所有`main`的内部变量（比如`a`和`b`）都保存在这个帧里面。`main`函数执行结束后，该帧就会被回收，释放所有的内部变量，不再占用空间。

![帧](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012211.png)



如果函数内部调用了其他函数，会发生什么情况？

```c
int main() {
   int a = 2;
   int b = 3;
   return add_a_and_b(a, b);
}
```

上面代码中，`main`函数内部调用了`add_a_and_b`函数。执行到这一行的时候，系统也会为`add_a_and_b`新建一个帧，用来储存它的内部变量。也就是说，此时同时存在两个帧：`main`和`add_a_and_b`。一般来说，调用栈有多少层，就有多少帧。

![嵌套函数帧](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012212.png)

等到`add_a_and_b`运行结束，它的帧就会被回收，系统会回到函数`main`刚才中断执行的地方，继续往下执行。通过这种机制，就实现了函数的层层调用，并且每一层都能使用自己的本地变量。

所有的帧都存放在 Stack，由于帧是一层层叠加的，所以 Stack 叫做栈。生成新的帧，叫做"入栈"，英文是 push；栈的回收叫做"出栈"，英文是 pop。Stack 的特点就是，最晚入栈的帧最早出栈（因为最内层的函数调用，最先结束运行），这就叫做"后进先出"的数据结构。每一次函数执行结束，就自动释放一个帧，所有函数执行结束，整个 Stack 就都释放了。

![入栈](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012213.jpg)

![出栈](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012214.jpg)

Stack 是由内存区域的结束地址开始，从高位（地址）向低位（地址）分配。比如，内存区域的结束地址是`0x8000`，第一帧假定是16字节，那么下一次分配的地址就会从`0x7FF0`开始；第二帧假定需要64字节，那么地址就会移动到`0x7FB0`。

![栈空间分布](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012215.png)

> [!IMPORTANT]
>
> 栈向下增长，堆向上增长

## 栗子

现有一个简单的程序：

*==exapmle.c==*

```c
int add_a_and_b(int a, int b) {
   return a + b;
}

int main() {
   return add_a_and_b(2, 3);
}
```

使用指令将该文件编译为汇编文件：

```bash
gcc -S example.c
```

上面的命令执行以后，会生成一个文本文件`example.s`，里面就是汇编语言，包含了几十行指令。这么说吧，一个高级语言的简单操作，底层可能由几个，甚至几十个 CPU 指令构成。CPU 依次执行这些指令，完成这一步操作。

`example.s`经过简化以后，大概是下面的样子：

```assembly
_add_a_and_b:
   push   %ebx
   mov    %eax, [%esp+8] 
   mov    %ebx, [%esp+12]
   add    %eax, %ebx 
   pop    %ebx 
   ret  

_main:
   push   3
   push   2
   call   _add_a_and_b 
   add    %esp, 8
   ret
```

可以看到，原程序的两个函数`add_a_and_b`和`main`，对应两个标签`_add_a_and_b`和`_main`。每个标签里面是该函数所转成的 CPU 运行流程。

每一行就是 CPU 执行的一次操作。它又分成两部分，就以其中一行为例：

```assembly
push   %ebx
```

这一行里面，`push`是 CPU 指令，`%ebx`是该指令要用到的运算子。一个 CPU 指令可以有零个到多个运算子。

### Push指令

根据约定，程序从`_main`标签开始执行，这时会在 Stack 上为`main`建立一个帧，并将 Stack顶所指向的地址，写入 ESP 寄存器。后面如果有数据要写入`main`这个帧，就会写在 ESP 寄存器所保存的地址。

然后，开始执行第一行代码。

 ```assembly
 push   3
 ```

`push`指令用于将运算子放入 Stack，这里就是将`3`写入`main`这个帧。

虽然看上去很简单，`push`指令其实有一个前置操作。它会先取出 ESP 寄存器里面的地址，将其减去4个字节，然后将新地址写入 ESP 寄存器。使用减法是因为 Stack 从高位向低位发展，4个字节则是因为`3`的类型是`int`，占用4个字节。得到新地址以后， 3 就会写入这个地址开始的四个字节。

 ```assembly
 push   2
 ```

第二行也是一样，`push`指令将`2`写入`main`这个帧，位置紧贴着前面写入的`3`。这时，ESP 寄存器会再减去 4个字节（累计减去8）。

![栈增长情况](https://www.ruanyifeng.com/blogimg/asset/2018/bg2018012216.png)

### Call指令

第三行的`call`指令用来调用函数。

```assembly
call   _add_a_and_b
```

上面的代码表示调用`add_a_and_b`函数。这时，程序就会去找`_add_a_and_b`标签，并为该函数建立一个新的帧。

下面就开始执行`_add_a_and_b`的代码。

```assembly
push   %ebx
```



这一行表示将 EBX 寄存器里面的值，写入`_add_a_and_b`这个帧。这是因为后面要用到这个寄存器，就先把里面的值取出来，用完后再写回去。

这时，`push`指令会再将 ESP 寄存器里面的地址减去4个字节（累计减去12）。

### Mov指令

`mov`指令用于将一个值写入某个寄存器。

 ```assembly
 mov    %eax, [%esp+8] 
 ```

这一行代码表示，先将 ESP 寄存器里面的地址加上8个字节，得到一个新的地址，然后按照这个地址在 Stack 取出数据。根据前面的步骤，可以推算出这里取出的是`2`，再将`2`写入 EAX 寄存器。

下一行代码也是干同样的事情。

 ```assembly
 mov    %ebx, [%esp+12] 
 ```

上面的代码将 ESP 寄存器的值加12个字节，再按照这个地址在 Stack 取出数据，这次取出的是`3`，将其写入 EBX 寄存器。

### Add指令

`add`指令用于将两个运算子相加，并将结果写入第一个运算子。

 ```assembly
add    %eax, %ebx
 ```

上面的代码将 EAX 寄存器的值（即2）加上 EBX 寄存器的值（即3），得到结果5，再将这个结果写入第一个运算子 EAX 寄存器。

### Pop指令

`pop`指令用于取出 Stack 最近一个写入的值（即最低位地址的值），并将这个值写入运算子指定的位置。

 ```assembly
 pop    %ebx
 ```

上面的代码表示，取出 Stack 最近写入的值（即 EBX 寄存器的原始值），再将这个值写回 EBX 寄存器（因为加法已经做完了，EBX 寄存器用不到了）。

注意，`pop`指令还会将 ESP 寄存器里面的地址加4，即回收4个字节。

### Ret指令

`ret`指令用于终止当前函数的执行，将运行权交还给上层函数。也就是，当前函数的帧将被回收。
 ```assembly
 ret
 ```

可以看到，该指令没有运算子。

随着`add_a_and_b`函数终止执行，系统就回到刚才`main`函数中断的地方，继续往下执行。

 ```assembly
 add    %esp, 8 
 ```

上面的代码表示，将 ESP 寄存器里面的地址，手动加上8个字节，再写回 ESP 寄存器。这是因为 ESP 寄存器的是 Stack 的写入开始地址，前面的`pop`操作已经回收了4个字节，这里再回收8个字节，等于全部回收。

 ```assembly
 ret
 ```

最后，`main`函数运行结束，`ret`指令退出程序执行。

# 语法

## 段

汇编程序可以分为 **3** 个段：

- **data** 段：**data**（数据）段被用于声明初始化的数据或常数。此数据在运行时不会更改。您可以在段中声明各种常量值，文件名或缓冲区大小等。

  ```assembly
  section.data
  ```

- **bss** 段：在 **bss** 段用于声明变量。

  ```assembly
  section.bss
  ```

  

- **text** 段：**text** 段被用于保持实际的代码。该段必须以全局声明 `_start` 开头，该声明告诉内核程序从何处开始执行。

  ```assembly
  section.text
     global _start
  _start:
  ```

## 注释

汇编语言注释以分号（`;`）开头。它可以包含任何可打印字符，包括空格。它可以单独出现在一行上，例如

```assembly
; 该程序在屏幕上显示一条消息
add eax, ebx     ; 加上 ebx 到 eax
```



## 申明

汇编语言程序包含 **3** 种类型的语句：

- 可执行指令或说明：**可执行指令** 或简单的 **指令** 告诉处理器做什么。每个指令由一个 **操作码**（opcode）组成。每个可执行指令生成一个机器语言指令。
- 汇编程序指令或伪操作：**汇编指令** 或 **伪操作** 告诉汇编器关于程序的各个方面。这些是不可执行的，不会生成机器语言指令。
- 宏：宏基本上是一种代码替换机制。

## 语法结构



汇编语言语句每行输入一个语句。每个语句遵循以下格式：

```assembly
[label]   mnemonic   [operands]   [;comment]
```

方括号中的字段是可选的。基本指令包括两段，第一段是要执行的指令（或助记符）的名称，第二段是命令的操作数或参数。

以下是一些典型汇编语言语句的实例：

```assembly
INC COUNT        ; 增加内存变量 COUNT
MOV TOTAL, 48    ; 将值 48 转移到内存变量 TOTAL
ADD AH, BH       ; 添加寄存器 BH 内容到AH 寄存器
AND MASK1, 128   ; 对变量 MASK1 和 128 执行 AND 操作
ADD MARKS, 10    ; 将 10 加到变量 MARKS
MOV AL, 10       ; 将值 10 传送到 AL 寄存器
```



栗子：

```assembly
section    .text
   global _start     ;必须为链接器(ld)声明
   
_start:                ;告诉链接器入口点
   mov    edx,len     ;消息长度
   mov    ecx,msg     ;写消息
   mov    ebx,1       ;文件描述符 (stdout)
   mov    eax,4       ;系统调用号 (sys_write)
   int    0x80        ;调用内核
   mov    eax,1       ;系统调用号 (sys_exit)
   int    0x80        ;调用内核
section    .data
	msg db 'Hello, world!', 0xa  ;要打印的字符串
	len equ $ - msg     ;字符串的长度
```

## 内存段

如果将 `section` 关键字替换为 `segment`，则会得到相同的结果。尝试以下代码：

```assembly
segment    .text
   global _start     ;必须为链接器(ld)声明
_start:                ;告诉链接器入口点
   mov    edx,len     ;消息长度
   mov    ecx,msg     ;写消息
   mov    ebx,1       ;文件描述符 (stdout)
   mov    eax,4       ;系统调用号 (sys_write)
   int    0x80        ;调用内核
   mov    eax,1       ;系统调用号 (sys_exit)
   int    0x80        ;调用内核
segment    .data
	msg db 'Hello, world!', 0xa  ;要打印的字符串
	len equ $ - msg     ;字符串的长度
```

分段存储器模型将系统存储器分为独立的分段组，这些分段由位于分段寄存器中的指针引用。每个段用于包含特定类型的数据。一个段用于包含指令代码，另一段用于存储数据元素，第三段保留程序堆栈。

根据以上讨论，我们可以将各种内存段指定为：

- **数据段** - 由 **.data** 段和 **.bss** 表示。**.data** 段用于声明存储区，在该存储区中为程序存储了数据元素。声明数据元素后，无法扩展此部分，并且在整个程序中它保持静态。**.bss** 段也是一个静态内存部分，它包含缓冲区，供稍后在程序中声明的数据使用。这个缓冲区内存是零填充的。
- **代码段** - 它由 **.text** 表示。这在内存中定义了一个存储指令代码的区域。这也是一个固定区域。
- **堆栈** - 该段包含传递给程序中的函数和过程的数据值。

# 系统调用

系统调用是用户空间和内核空间之间接口的 API。我们已经使用了系统调用。`syswrite` 和 `sysexit`，分别用于写入屏幕和退出程序。

> 如何在linux中使用系统调用？
>
> 1. 将系统呼叫号放入 **EAX** 寄存器中。
> 2. 将参数保存到系统调用中的寄存器 **EBX**，**ECX** 等中。
> 3. 调用相关的中断（**80h**）。
> 4. 结果通常在 **EAX** 寄存器中返回。
>
> 有 **6** 个寄存器，用于存储所用系统调用的参数。它们是 **EBX**，**ECX**，**EDX**，**ESI**，**EDI** 和 **EBP**。这些寄存器采用从 **EBX** 寄存器开始的连续参数。如果有 6 个以上的自变量，则第一个自变量的存储位置将存储在 **EBX** 寄存器中。
>
> 以下代码显示了系统调用 `sys_exit` 的使用：
>
> ```assembly
> moveax,1; 系统调用号 (sys_exit)
> int0x80; 调用内核
> ```
>
> 以下代码显示了系统调用 `sys_write` 的使用：
>
> ```assembly
> movedx,4; 消息长度
> movecx,msg; 消息内容
> movebx,1; 文件描述 (stdout)
> moveax,4; 系统调用号 (sys_write)
> int0x80; 调用内核
> ```
>
> > [!IMPORTANT]
> >
> > 常见系统调用：
> >
> > | %eax |   名称    |      %ebx      |    %ecx    |  %edx  | %esx | %edi |
> > | :--: | :-------: | :------------: | :--------: | :----: | :--: | :--: |
> > |  1   | sys_exit  |      int       |     -      |   -    |  -   |  -   |
> > |  2   | sys_fork  | struct pt_regs |     -      |   -    |  -   |  -   |
> > |  3   | sys_read  |  unsigned int  |    char    | size_t |  -   |  -   |
> > |  4   | sys_write |  unsigned int  | const char | size_t |  -   |  -   |
> > |  5   | sys_open  |  const char *  |    int     |  int   |  -   |  -   |
> > |  6   | sys_close |  unsigned int  |     -      |   -    |  -   |  -   |

栗子：

```assembly
section .data              ;数据段
  userMsg db 'Please enter a number: ' ;让用户输入数字
  lenUserMsg equ $-userMsg       ;消息长度
  dispMsg db 'You have entered: '
  lenDispMsg equ $-dispMsg
section .bss      ;未初始化的数据
  num resb 5
section .text     ;代码段
  global _start
_start:        ;用户提示
  mov eax, 4
  mov ebx, 1
  mov ecx, userMsg
  mov edx, lenUserMsg
  int 80h
  ;Read and store the user input
  mov eax, 3
  mov ebx, 2
  mov ecx, num
  mov edx, 5     ;该信息的 5 个字节（数字，1表示符号）
  int 80h
  ;Output the message 'You have entered:'
  mov eax, 4
  mov ebx, 1
  mov ecx, dispMsg
  mov edx, lenDispMsg
  int 80h
  ;Output the number entered
  mov eax, 4
  mov ebx, 1
  mov ecx, num
  mov edx, 5
  int 80h
  ; Exit code
  mov eax, 1
  mov ebx, 0
  int 80h
```



输出结果如下：

```assembly
Please enter a number:
123456
You have entered:123456
```



# 寻址模式



大多数汇编语言指令都需要处理操作数。操作数地址提供了要处理的数据存储的位置。有些指令不需要操作数，而另一些指令则需要一个，两个或三个操作数。

当一条指令需要两个操作数时，第一个操作数通常是==目的地==，它在寄存器或存储器位置中包含数据，第二个==操作数是源==。源包含要传递的数据（立即寻址）或数据的地址（在寄存器或存储器中）。通常，操作后源数据保持不变。

寻址的三种基本模式是：

- 寄存器寻址
- 立即寻址
- 内存寻址

## 寄存器寻址

在这种寻址模式下，寄存器包含操作数。根据指令，寄存器可以是第一个操作数，第二个操作数或两者。

```assembly
MOV DX, TAX_RATE  ; 寄存器是第一个操作数
MOV COUNT, CX  ; 寄存器是第二个操作数
MOV EAX, EBX  ; 两个操作数都是寄存器
```



由于寄存器之间的数据处理不涉及内存，因此可以最快地处理数据。

## 立即寻址

立即数操作数具有常量值或表达式。当具有两个操作数的指令使用立即寻址时，第一个操作数可以是寄存器或存储器位置，而第二个操作数是立即数。第一个操作数定义数据的长度。

```assembly
BYTE_VALUE DB 150  ; 一个字节值被定义
WORD_VALUE DW 300  ; 一个字值被定义
ADD BYTE_VALUE, 65  ; BYTE_VALUE 加一个立即操作数 65
MOV AX, 45H      ; 立即常数 45H 转移到 AX
```



## 直接内存寻址

在存储器寻址模式下指定操作数时，通常需要直接访问主存储器，通常是数据段。这种寻址方式导致数据处理变慢。为了找到数据在内存中的确切位置，我们需要段起始地址（通常在 **DS** 寄存器中找到）和偏移值。此偏移值也称为 **有效地址**。

在直接寻址模式下，偏移量值直接作为指令的一部分指定，通常由变量名指示。汇编器计算偏移值并维护一个符号表，该表存储程序中使用的所有变量的偏移值。在直接存储器寻址中，一个操作数引用一个存储器位置，另一个操作数引用一个寄存器。

```assembly
ADDBYTE_VALUE, DL; 将寄存器添加到存储位置
MOVBX, WORD_VALUE; 将内存中的操作数添加到寄存器中
```

## 直接偏移寻址

此寻址模式使用算术运算符修改地址。例如，查看以下定义数据表的定义：

```assembly
BYTE_TABLE DB 14, 15, 22, 45   ; 字节表
WORD_TABLE DW 134, 345, 564, 123 ; 字表
```



以下操作将数据从内存中的表访问到寄存器中：

```assembly
MOV CL, BYTE_TABLE[2]; 获取 BYTE_TABLE 的第 3 个元素
MOV CL, BYTE_TABLE + 2; 获取 BYTE_TABLE 的第 3 个元素
MOV CX, WORD_TABLE[3]; 获取 WORD_TABLE 的第 4 个元素
MOV CX, WORD_TABLE + 3; 获取 WORD_TABLE 的第 4 个元素
```



## 间接内存寻址



此寻址模式利用计算机的 Segment：Offset 寻址功能。通常，在方括号内编码的基址寄存器 **EBX**，**EBP**（或 **BX**，**BP**）和索引寄存器（**DI**，**SI**）用于内存引用。

间接寻址通常用于包含多个元素（如数组）的变量。阵列的起始地址存储在 **EBX** 寄存器中。

以下代码段显示了如何访问变量的不同元素。

```assembly
MY_TABLE TIMES 10 DW 0  ; 分配 10 个字（2 个字节），每个字都初始化为 0
MOV EBX, [MY_TABLE]     ; EBX中MY_TABLE的有效地址
MOV [EBX], 110          ; MY_TABLE[0] = 110
ADD EBX, 2              ; EBX = EBX +2
MOV [EBX], 123          ; MY_TABLE[1] = 123
```



> [!NOTE]
>
> 注意：
>
> `[]`在这里实际上是解引用，用来访问地址中的值
>
> 这里给EBX加2是给地址增加两字节，增加n就是增加n个字节，并不随着变量的类型而改变

## MOV指令

我们已经使用了 `MOV` 指令，该指令用于将数据从一个存储空间移动到另一个存储空间。`MOV` 指令采用两个操作数。

```assembly
MOV  destination, source
```

MOV指令可以有以下五种形式：

```assembly
MOV register, register
MOV register, immediate
MOV memory, immediate
MOV register, memory
MOV memory, register
```

> [!NOTE]
>
> - Mov操作中的两个操作数应具有相同的大小
> - 源操作数的值保持不变
>
> `MOV` 指令有时会引起歧义。例如：
>
> ```assembly
> MOV EBX, [MY_TABLE] ; Effective Address of MY_TABLE in EBXMOV [EBX], 110   ; MY_TABLE[0] = 110
> ```
>
> 目前尚不清楚是要移动等于 **110** 的字节等效值还是等效于字的字符。在这种情况下，使用**类型说明符** 是明智的。
>
> 下表是一些常见的类型说明符：
>
> | 类型说明符 | 寻址字节 |
> | :--------: | :------: |
> |    BYTE    |    1     |
> |    WORD    |    2     |
> |   DWORD    |    4     |
> |   QWORD    |    8     |
> |   TBYTE    |    10    |

栗子：

下面的程序说明了上面讨论的一些概念。它在存储器的数据部分中存储名称 "Alex Mo"，然后以编程方式将其值更改为另一个名称 "Feng Mo" 并显示这两个名称。

```assembly
section .text
   global _start     ;必须声明为链接器（ld）
_start:             ;告诉链接器入口点
   ;写名字“Alex Mo”
   mov  edx,9       ;消息长度
   mov  ecx, name   ;消息内容
   mov  ebx,1       ;文件描述 (stdout)
   mov  eax,4       ;系统调用号 (sys_write)
   int  0x80        ;调用内核
   mov  [name],  dword 'Feng'    ; 将名称修改为 Feng Mo
   ;writing the name 'Feng Mo'
   mov  edx,8       ;消息长度
   mov  ecx,name    ;消息内容
   mov  ebx,1       ;文件描述 (stdout)
   mov  eax,4       ;系统调用号 (sys_write)
   int  0x80        ;调用内核
   mov  eax,1       ;系统调用号 (sys_exit)
   int  0x80        ;调用内核
section .data
	name db 'Alex Mo '
```



# 数据类型

## 变量

### 初始化数据分配空间



初始化数据的存储分配语句的语法为：

```assembly
[variable-name]    define-directive    initial-value   [,initial-value]...
```

其中，*变量名* 是每个存储空间的标识符。汇编器为数据段中定义的每个变量名称关联一个偏移值。

`define` 指令有五种基本形式：



| 指令 |     作用     |    储存空间    |
| :--: | :----------: | :------------: |
|  DB  |   定义字节   | 分配 1 个字节  |
|  DW  |    定义字    | 分配 2 个字节  |
|  DD  |   定义双字   | 分配 4 个字节  |
|  DQ  |   定义四字   | 分配 8 个字节  |
|  DT  | 定义十个字节 | 分配 10 个字节 |

栗子：

```assembly
choice          DB      'y'
number          DW      12345
neg_number      DW      -12345
big_number      DQ      123456789
real_number1    DD      1.234
real_number2    DQ      123.456
```

> [!NOTE]
>
> - 字符的每个字节均以十六进制形式存储为其 ASCII 值。
> - 每个十进制值都将自动转换为其等效的 16 位二进制数，并以十六进制数形式存储。
> - 处理器使用小尾数字节顺序。
> - 负数将转换为其 2 的补码表示形式。
> - 短浮点数和长浮点数分别使用 32 位或 64 位表示。

```assembly
section .text
   global _start          ;必须声明链接器 (ld)
_start:                   ;告诉链接器入口点
   mov  edx,1             ;消息长度
   mov  ecx,choice        ;消息内容
   mov  ebx,1             ;文件描述 (stdout)
   mov  eax,4             ;系统调用号 (sys_write)
   int  0x80              ;调用内核
   mov  eax,1             ;系统调用号 (sys_exit)
   int  0x80              ;调用内核
section .data
	choice DB 'y'
```

### 未初始化数据分配空间

`reserve` 指令用于为未初始化的数据保留空间。`reserve` 指令采用单个操作数，该操作数指定要保留的空间单位数。每个 `define` 指令都有一个相关的 `reserve` 指令。

`reserve` 指令有五种基本形式：

| 指令 |      作用      |
| :--: | :------------: |
| RESB | 保留 1 个字节  |
| RESW |     保留字     |
| RESD | 保留 1 个双字  |
| RESQ |  保留 4 个字   |
| REST | 保留 10 个字节 |

### 多重初始化

`TIMES` 指令允许多次初始化为相同的值。例如，可以使用以下语句定义一个大小为 9 的标记的数组并将其初始化为 0：



```assembly
section .text
   global _start        ;必须声明链接器 (ld)
_start:                 ;告诉链接器入口点
   mov  edx,9           ;消息长度
   mov  ecx, stars      ;消息内容
   mov  ebx,1           ;文件描述 (stdout)
   mov  eax,4           ;系统调用号 (sys_write)
   int  0x80            ;调用内核
   mov  eax,1           ;系统调用号 (sys_exit)
   int  0x80            ;调用内核
section .data
	stars times 9 db '*'
```

## 常量



常见常量指令：

- `EQU`
- `%assign`
- `%define`

### EQU指令

`EQU` 指令用于定义常量。`EQU` 指令的语法如下：

```assembly
CONSTANT_NAME EQU expression
```

栗子：

```assembly
TOTAL_STUDENTS equ 50
mov ecx, TOTAL_STUDENTS
cmp eax, TOTAL_STUDENTS
```



当然，`EQU`语句的操作数也可以是表达式

```assembly
LENGTH equ 20
WIDTH equ 10
AREA  equ length * width
```

更加复杂的栗子：

```assembly
SYS_EXIT equ 1
SYS_WRITE equ 4
STDIN   equ 0
STDOUT  equ 1
section .text
  global _start  ;必须声明 gcc
_start:       ;告诉链接器入口
  mov eax, SYS_WRITE
  mov ebx, STDOUT
  mov ecx, msg1
  mov edx, len1
  int 0x80
  mov eax, SYS_WRITE
  mov ebx, STDOUT
  mov ecx, msg2
  mov edx, len2
  int 0x80
  mov eax, SYS_WRITE
  mov ebx, STDOUT
  mov ecx, msg3
  mov edx, len3
  int 0x80
  mov eax,SYS_EXIT  ;系统调用号 (sys_exit)
  int 0x80      ;调用内核
section .data
msg1 db'Hello, programmers!',0xA,0xD
len1 equ $ - msg1
msg2 db 'Welcome to cankaoshouce.com,', 0xA,0xD
len2 equ $ - msg2
msg3 db 'Learn assembly programming! '
len3 equ $- msg3
```



### %assign 指令

`％assign` 指令像 `EQU` 指令一样可以用来定义数字常量。该指令支持重新定义。例如，您可以将常量 `TOTAL` 先定义为：

```assembly
%assign TOTAL 10
```

在代码的后面，您可以将其重新定义为：

```assembly
%assign  TOTAL  20
```

> [!NOTE]
>
> **注意** ：指令区分大小写。

### %define 指令

`％define` 指令可以定义数值和字符串常量。该指令类似于 **C** 语言中的 `#define`。例如，您可以将常量 `PTR` 定义为：

```assembly
%define PTR [EBP+4]
```

上面的代码用 **[EBP + 4]** 替换了 **PTR**。

该指令也支持重新定义，并且区分大小写。

## 数字

数值数据通常用二进制表示。算术指令对二进制数据进行操作。==当数字显示在屏幕上或从键盘输入时，它们是 **ASCII** 形式==。到目前为止，我们已经将该输入数据以 **ASCII** 形式转换为二进制以进行算术计算，并将结果转换回二进制。

```assembly
section .text
   global _start        ;必须声明 gcc
_start:                 ;告诉链接器入口点
   mov  eax,'3'
   sub  eax, '0'
   mov  ebx, '4'
   sub  ebx, '0'
   add  eax, ebx
   add  eax, '0'
   mov  [sum], eax
   mov  ecx,msg
   mov  edx, len
   mov  ebx,1            ;文件描述 (stdout)
   mov  eax,4            ;系统调用号 (sys_write)
   int  0x80             ;调用内核
   mov  ecx,sum
   mov  edx, 1
   mov  ebx,1            ;文件描述 (stdout)
   mov  eax,4            ;系统调用号 (sys_write)
   int  0x80             ;调用内核
   mov  eax,1            ;系统调用号 (sys_exit)
   int  0x80             ;调用内核
section .data
	msg db "The sum is:", 0xA,0xD
	len equ $ - msg
segment .bss
	sum resb 1
```

然而，这种转换有性能损耗，汇编语言编程支持以更有效的方式处理二进制形式的数字。

十进制数字可以用两种形式：

- ASCII 格式
- BCD 或二进制编码十进制形式

### ASCII表示

在 ASCII 表示中，十进制数字存储为 ASCII 字符字符串。

例如，十进制值 1234 存储为：

```assembly
31323334H
```

其中，**31H** 是 1 的 ASCII 值，**32H** 是 2 的 ASCII 值，依此类推。

有 4 种指令用于处理 ASCII 表示形式的数字：

- `AAA` - 加法后 的ASCII 调整
- `AAS` - 减法后的 ASCII 调整
- `AAM` - 乘法后的 ASCII 调整
- `AAD` - 除法前的 ASCII 调整

这些指令不使用任何操作数，并假定所需的操作数位于 **AL** 寄存器中。

比如下面的实例：

```assembly
section .text
   global _start        ;必须声明 gcc
_start:                 ;告诉链接器入口点
   sub  ah, ah
   mov  al, '9'
   sub  al, '3'
   aas
   or   al, 30h
   mov  [res], ax
   mov  edx,len         ;消息长度
   mov  ecx,msg         ;消息
   mov  ebx,1           ;文件描述 (stdout)
   mov  eax,4           ;系统调用号 (sys_write)
   int  0x80            ;调用内核
   mov  edx,1           ;消息长度
   mov  ecx,res         ;消息
   mov  ebx,1           ;文件描述 (stdout)
   mov  eax,4           ;系统调用号 (sys_write)
   int  0x80            ;调用内核
   mov  eax,1           ;系统调用号 (sys_exit)
   int  0x80            ;调用内核
section .data
	msg db 'The Result is:',0xa
	len equ $ - msg
section .bss
	res resb 1
```



### BCD表示

**BCD** 表示有两种类型：

- 未打包的 **BCD** 表示
- 打包的 **BCD** 表示

在未压缩的 BCD 表示形式中，每个字节都存储一个十进制数字的二进制等效项。例如，数字 1234 存储为：

```assembly
01      02      03      04H
```

有两种指令来处理这些数字：

- `AAM` - 乘法后 ASCII 调整
- `AAD` - 除法前 ASCII 调整

四个 ASCII 调整指令 `AAA`，`AAS`，`AAM` 和 `AAD` 也可以与未打包的 BCD 表示一起使用。在打包的 BCD 表示中，每个数字使用四位存储。两个十进制数字打包成一个字节。

例如，数字 **1234** 存储为：

```assembly
1234H
```

有两种指令来处理这些数字：

- `DAA` - 加法后的十进制调整
- `DAS` - 减后的十进制调整

> [!TIP]
>
> 打包的 **BCD** 表示形式不支持乘法和除法。

栗子：

以下程序将两个 5 位十进制数字加起来并显示总和。

```assembly
section .text
   global _start        ;必须声明才能使用 gcc
_start:                 ;告诉链接器入口点
   mov     esi, 4       ;指向最右边的数字
   mov     ecx, 5       ;数字位数
   clc
add_num:
   mov  al, [num1 + esi]
   adc  al, [num2 + esi]
   aaa
   pushf
   or   al, 30h
   popf
   mov  [sum + esi], al
   dec  esi
   num  add_num
   mov  edx,len         ;消息长度
   mov  ecx,msg         ;消息
   mov  ebx,1           ;文件描述 (stdout)
   mov  eax,4           ;系统调用数 (sys_write)
   int  0x80            ;调用内核
   mov  edx,5           ;消息长度
   mov  ecx,sum         ;消息
   mov  ebx,1           ;文件描述 (stdout)
   mov  eax,4           ;系统调用数 (sys_write)
   int  0x80            ;调用内核
   mov  eax,1           ;系统调用数 (sys_exit)
   int  0x80            ;调用内核
section .data
	msg db 'The Sum is:',0xa
	len equ $ - msg
	num1 db '12345'
	num2 db '23456'
	sum db '     '
```

## 字符串

在前面的实例中，我们已经使用了可变长度字符串。可变长度字符串可以包含所需的任意多个字符。

通常，我们通过以下两种方法之一指定字符串的长度：

- 显式存储字符串长度
- 使用哨兵字符

我们可以使用表示位置计数器当前值的 `$location` 计数器符号来显式存储字符串长度。

在以下实例中：

```assembly
msg  db  'Hello, world!',0xa ;我们常见的字符
len  equ  $ - msg            ;长度
```

> [!TIP]
>
> 这里可以注意到使用db来定义字符串，在变量一节中，db是用来定义一个字节的。
>
> 所以实际上以上的代码我们可以理解为
>
> ```assembly
> msg db 'H','e','l','l','o',',' ', ','w','o','r','l','d','\n'	;上面的0xa实际上是换行符\n
> ```
>
> 

$指向字符串变量 `msg` 的最后一个字符之后的字节。因此，`$ - msg` 给出字符串的长度。我们也可以这样写：

```assembly
msg db 'Hello, world!',0xa ;字符串
len equ 13                 ;字符串的长度
```

另外，您可以存储带有尾部定点字符的字符串来定界字符串，而不是显式存储字符串长度。前哨字符应为不出现在字符串中的特殊字符。

例如，

```assembly
message DB 'I am loving it!', 0
```

### 字符串指令

每个字符串指令可能需要一个源操作数，一个目标操作数或两者。对于 32 位段，字符串指令使用 `ESI` 和 `EDI` 寄存器分别指向源和目标操作数。但是，对于 16 位段，`SI` 和 `DI` 寄存器分别用于指向源和目标。

有 5 种用于处理字符串的基本说明。他们是：

- `MOVS` - 该指令将 1 字节，字或双字数据从存储位置移到另一个位置。
- `LODS` - 该指令从存储器加载。如果操作数是一个字节，则将其加载到 **AL** 寄存器中；如果操作数是一个字，则将其加载到 **AX** 寄存器中，并将双字加载到 **EAX** 寄存器中。
- `STOS` - 该指令将数据从寄存器（**AL**，**AX** 或 **EAX**）存储到存储器。
- `CMPS` - 该指令比较内存中的两个数据项。数据可以是字节大小，字或双字。
- `SCAS` - 该指令将寄存器（**AL**，**AX** 或 **EAX**）的内容与内存中项目的内容进行比较。

上面的每个指令都有字节，字和双字版本，并且可以通过使用重复前缀来重复字符串指令。

这些指令使用 **ES:DI** 和 **DS:SI** 对寄存器，其中 **DI** 和 **SI** 寄存器包含有效的偏移地址，这些地址指向存储在存储器中的字节。**SI** 通常与 **DS**（数据段）相关联，**DI** 通常与 **ES**（额外段）相关联。

**DS:SI**（或 **ESI**）和 **ES:DI**（或 **EDI**）寄存器分别指向源和目标操作数。假定源操作数位于内存中的 **DS:SI**（或 **ESI**），目标操作数位于 **ES:DI**（或 **EDI**）。

对于 16 位地址，使用 **SI** 和 **DI** 寄存器，对于 32 位地址，使用 **ESI** 和 **EDI** 寄存器。

下表提供了各种版本的字符串指令和假定的操作数空间：

| 基础指令                                                     | 操作的寄存器  | 字节运算 | 字运算 | 双字运算 |
| ------------------------------------------------------------ | ------------- | -------- | ------ | -------- |
| [MOVS](https://cankaoshouce.com/assembly-strings/assembly-movs-instruction.html) | ES:DI, DS:SI  | MOVSB    | MOVSW  | MOVSD    |
| [LODS](https://cankaoshouce.com/assembly-strings/assembly-lods-instruction.html) | AX, DS:SI     | LODSB    | LODSW  | LODSD    |
| [STOS](https://cankaoshouce.com/assembly-strings/assembly-stos-instruction.html) | ES:DI, AX     | STOSB    | STOSW  | STOSD    |
| [CMPS](https://cankaoshouce.com/assembly-strings/assembly-cmps-instruction.html) | DS:SI, ES: DI | CMPSB    | CMPSW  | CMPSD    |
| [SCAS](https://cankaoshouce.com/assembly-strings/assembly-scas-instruction.html) | ES:DI, AX     | SCASB    | SCASW  | SCASD    |

#### MOVS

`MOVS` 指令用于将数据项（字节，字或双字）从源字符串复制到目标字符串。源字符串由 **DS:SI** 指向，目标字符串由 **ES:DI** 指向。



```assembly
section .text
   global _start        ;must be declared for using gcc
_start:                 ;tell linker entry point
   mov  ecx, len
   mov  esi, s1
   mov  edi, s2
   cld
   rep  movsb
   mov  edx,20          ;message length
   mov  ecx,s2          ;message to write
   mov  ebx,1           ;file descriptor (stdout)
   mov  eax,4           ;system call number (sys_write)
   int  0x80            ;call kernel
   mov  eax,1           ;system call number (sys_exit)
   int  0x80            ;call kernel
section .data
	s1 db 'Hello, world!',0 ;string 1
	len equ $-s1
section  .bss
	s2 resb 20              ;destination
```

#### LODS

`LODS` 指令从存储器加载。如果操作数是一个字节，则将其加载到 **AL** 寄存器中；如果操作数是一个字，则将其加载到 **AX** 寄存器中，并将双字加载到 **EAX** 寄存器中。

在密码术中，凯撒密码是最简单的已知加密技术之一。在这种方法中，要加密的数据中的每个字母都被替换为字母下方固定数量位置的字母。

在这个实例中，让我们通过简单地将其中的每个字母替换为两个字母来加密数据，因此 **a** 将被 **c**，**b** 替换为 **d** 等。我们使用 `LODS` 将原始字符串 'password' 加载到内存中。

```assembly
section .text
   global _start         ;must be declared for using gcc
_start:                  ;tell linker entry point
   mov    ecx, len
   mov    esi, s1
   mov    edi, s2
loop_here:
   lodsb
   add al, 02
   stosb
   loop    loop_here
   cld
   rep     movsb
   mov     edx,20        ;message length
   mov     ecx,s2        ;message to write
   mov     ebx,1         ;file descriptor (stdout)
   mov     eax,4         ;system call number (sys_write)
   int     0x80          ;call kernel
   mov     eax,1         ;system call number (sys_exit)
   int     0x80          ;call kernel
section .data
	s1 db 'password', 0 ;source
	len equ $-s1
section .bss
	s2 resb 10               ;destination
```



#### STOS

`STOS` 指令将数据项从 **AL**（用于字节-STOSB）、**AX**（用于字-STOSW）或 **EAX**（用于双字-STOSD）复制到内存中 **ES:DI** 指向的目标字符串。

以下实例演示如何使用 `LODS` 和 `STOS` 指令将大写字符串转换为小写值：

```assembly
section .text
   global _start        ;must be declared for using gcc
_start:                 ;tell linker entry point
   mov    ecx, len
   mov    esi, s1
   mov    edi, s2
loop_here:
   lodsb
   or      al, 20h
   stosb
   loop    loop_here
   cld
   rep  movsb
   mov  edx,20          ;message length
   mov  ecx,s2          ;message to write
   mov  ebx,1           ;file descriptor (stdout)
   mov  eax,4           ;system call number (sys_write)
   int  0x80            ;call kernel
   mov  eax,1           ;system call number (sys_exit)
   int  0x80            ;call kernel
section .data
	s1 db 'HELLO, WORLD', 0 ;source
	len equ $-s1
section .bss
	s2 resb 20              ;destination
```

#### CMPS

`CMPS` 指令比较两个字符串。此指令比较 **DS:SI** 和 **ES:DI** 寄存器指向的单字节、字或双字的两个数据项，并相应设置标志。您还可以将条件跳转指令与此指令一起使用。

以下实例演示如何使用 `CMPS` 指令比较两个字符串：

```assembly
section .text
   global _start            ;must be declared for using gcc
_start: ;tell linker entry point
   mov esi, s1
   mov edi, s2
   mov ecx, lens2
   cld
   repe  cmpsb
   jecxz  equal             ;jump when ecx is zero
   ;If not equal then the following code
   mov eax, 4
   mov ebx, 1
   mov ecx, msg_neq
   mov edx, len_neq
   int 80h
   jmp exit
equal:
   mov eax, 4
   mov ebx, 1
   mov ecx, msg_eq
   mov edx, len_eq
   int 80h
exit:
   mov eax, 1
   mov ebx, 0
   int 80h
section .data
	s1 db 'Hello, world!',0      ;our first string
	lens1 equ $-s1
	s2 db 'Hello, there!', 0     ;our second string
	lens2 equ $-s2
	msg_eq db 'Strings are equal!', 0xa
	len_eq  equ $-msg_eq
	msg_neq db 'Strings are not equal!'
	len_neq equ $-msg_neq
```

#### SCAS

`SCAS` 指令用于搜索字符串中的特定字符或一组字符。要搜索的数据项应该在 **AL**（对于 **SCASB**），**AX**（对于 **SCASW**）或 **EAX**（对于 **SCASD**）寄存器中。要搜索的字符串应在内存中，并由 **ES:DI**（或 **EDI**）寄存器指向。

参考以下实例来了解概念：

```assembly
section .text
   global _start        ;must be declared for using gcc
_start:                 ;tell linker entry point
   mov ecx,len
   mov edi,my_string
   mov al , 'e'
   cld
   repne scasb
   je found ; when found
   ; If not not then the following code
   mov eax,4
   mov ebx,1
   mov ecx,msg_notfound
   mov edx,len_notfound
   int 80h
   jmp exit
found:
   mov eax,4
   mov ebx,1
   mov ecx,msg_found
   mov edx,len_found
   int 80h
exit:
   mov eax,1
   mov ebx,0
   int 80h
section .data
	my_string db 'hello world', 0
	len equ $-my_string
	msg_found db 'found!', 0xa
	len_found equ $-msg_found
	msg_notfound db 'not found!'
	len_notfound equ $-msg_notfound
```



### 重复前缀

`REP` 前缀在字符串指令（例如 - REP MOVSB）之前设置时，会根据放置在 CX 寄存器中的计数器使该指令重复。REP 执行该指令，将 CX 减 1，然后检查 CX 是否为 0。重复指令处理，直到 CX 为 0 为止。

方向标志（DF）确定操作的方向。

- 使用 CLD（清除方向标志，DF = 0）使操作从左到右。
- 使用 STD（设置方向标志，DF = 1）使操作从右到左。

`REP` 前缀也有以下变化:

- **REP**: 这是无条件的重复。重复该操作，直到 CX 为零为止。
- **REPE** 或 **REPZ**: 这是有条件的重复。当零标志指示等于/零时，它将重复操作。当 **ZF** 表示不等于 0 或 CX 为 0 时，它将停止。
- **REPNE** 或 **REPNZ**: 这也是有条件的重复。当零标志指示不等于/ 0 时，它将重复操作。当 **ZF** 指示等于/ 0 或 CX 减为 0 时，它将停止。

## 数组

我们已经讨论过，汇编器的数据定义指令用于为变量分配存储。还可以使用某些特定值初始化变量。可以十六进制、十进制或二进制形式指定初始化值。

例如，我们可以用以下任一方式定义单词变量 "months":

```assembly
MONTHS DW 12
MONTHS DW 0CH
MONTHS DW 0110B
```

数据定义指令也可用于定义一维数组。让我们定义一个一维数字数组。

```assembly
NUMBERS DW 34, 45, 56, 67, 75, 89
```

上面的定义声明了一个由 6 个单词组成的数组，每个单词用数字 34、45、56、67、75、89 初始化。这将分配 `2x6=12` 字节的连续内存空间。第一个数字的符号地址为 **NUMBERS**，第二个数字的字符地址为 **NUMBERS+2** ，依此类推。

让我们再举一个例子。您可以定义一个名为 `inventory` 的数组，大小为 8，并将所有值初始化为 0，比如：

```assembly
INVENTORY  	DW 0
			DW 0
            DW 0
            DW 0 
            DW 0 
            DW 0  
            DW 0   
            DW 0
```

可以缩写为：

```assembly
INVENTORY  DW 0, 0 , 0 , 0 , 0 , 0 , 0 , 0
```

`TIMES` 指令还可用于对同一值进行多次初始化。使用 `TIMES`，可以将 `INVENTORY` 数组定义为：

```assembly
INVENTORY TIMES 8 DW 0
```

栗子：

以下实例通过定义一个 3 元素数组 x 来演示上述概念，该数组存储三个值：2、3 和 4。它将值添加到数组中并显示总和 9：

```assembly
section .text
   global _start   ;必须为链接器声明 (ld)
_start:
   mov  eax,3      ;要求和的字节数
   mov  ebx,0      ;EBX 将存储总和
   mov  ecx, x     ;ECX 将指向要求和的当前元素
top:  
   add  ebx, [ecx]
   add  ecx,1      ;将指针移动到下一个元素
   dec  eax        ;计数器自减
   jnz  top        ;如果计数器不是 0，则再次循环
done:
   add   ebx, '0'
   mov  [sum], ebx ;完成，将结果存储为 sum
display:
   mov  edx,1      ;消息长度
   mov  ecx, sum   ;消息
   mov  ebx, 1     ;文件描述 (stdout)
   mov  eax, 4     ;系统调用号 (sys_write)
   int  0x80       ;调用内核
   mov  eax, 1     ;系统调用号 (sys_exit)
   int  0x80       ;调用内核
section .data
	global x
x:
   db  2
   db  4
   db  3
sum:
   db  0
```



## 宏

编写宏是确保用汇编语言进行模块化编程的另一种方法。

- 宏是一系列指令，由名称指定，可以在程序中的任何位置使用。
- 在 **NASM** 中，宏是用 `%macro` 和 `%endmacro` 指令定义的。
- 宏以 `%macro` 指令开始，以 `%endmacro` 指令结束。

宏的语法如下：





```assembly
%macro macro_name  number_of_params
<macro body>
%endmacro
```

其中，*number_of_params* 指定数字参数，*macro_name* 指定宏的名称。

通过使用宏名称和必要的参数来调用宏。当您需要在程序中多次使用某个指令序列时，可以将这些指令放在宏中并使用它，而不是一直编写指令。

例如，程序的一个常见需求是在屏幕上写入字符串。要显示字符串，您需要以下指令序列：

```assembly
move dx,len  ;message length
move cx,msg  ;message to write
move bx,1    ;file descriptor (stdout)
move ax,4    ;system call number (sys_write)
int  0x80    ;call kernel
```

在上面显示字符串的实例中，`INT 0x80` 函数调用使用了寄存器 **EAX**、**EBX**、**ECX** 和 **EDX**。因此，每次需要在屏幕上显示时，都需要将这些寄存器保存在堆栈上，调用 `INT 80H`，然后从堆栈中恢复寄存器的原始值。因此，编写两个宏来保存和恢复数据可能很有用。

我们注意到，一些指令，如 **IMUL**、**IDIV**、**INT** 等，需要将一些信息存储在某些特定寄存器中，甚至在某些特定的寄存器中返回值。如果程序已经使用这些寄存器来保存重要数据，那么这些寄存器中的现有数据应该保存在堆栈中，并在指令执行后恢复。

栗子：

```assembly
; 具有两个参数的宏
; 实现写入系统调用
%macro write_string 2
	mov   eax, 4
    mov   ebx, 1
    mov   ecx, %1
    mov   edx, %2
    int   80h
%endmacro
section .text
   global _start            ;必须声明才能使用 gcc
_start:                     ;告诉链接器入口点
   write_string msg1, len1
   write_string msg2, len2
   write_string msg3, len3
   mov eax,1                ;系统调用号 (sys_exit)
   int 0x80                 ;调用内核
section .data
	msg1 db 'Hello, programmers!',0xA,0xD
	len1 equ $ - msg1
	msg2 db 'Welcome to cankaoshouce.com,', 0xA,0xD
	len2 equ $- msg2
	msg3 db 'Learn assembly programming! '
	len3 equ $- msg3
```

# 运算

## 算术运算

### INC指令



`INC` 指令用于将操作数加 **1**。它对可以在寄存器或内存中的单个操作数起作用。

`INC` 指令的语法如下：

```assembly
INC destination
```

**目标操作数** 可以是 8 位，16 位或 32 位操作数。

小栗子：

```assembly
INC EBX      ; 32 位寄存器 自增 1
INC DL       ; 8 位寄存器 自增 1
INC [count]  ; 变量 count 自增 1
```

### DEC指令

`DEC` 指令用于将操作数减 1。它对可以在寄存器或内存中的单个操作数起作用。

`DEC` 指令的语法如下：

```assembly
DEC destination
```

**目标操作数** 可以是 8 位，16 位或 32 位操作数。

小栗子：

```assembly
segment .data
  count dw 0
  value db 15
segment .text
  inc [count]
  dec [value]
  mov ebx, count
  inc word [ebx]
  mov esi, value
  dec byte [esi]
```

### ADD和SUB

`ADD` 和 `SUB` 指令用于对字节，字和双字大小的二进制数据进行简单的 **加/减**，即分别用于加或减去 8 位，16 位或 32 位操作数。

`ADD` 和 `SUB` 指令的语法如下:

```assembly
ADD/SUB destination, source
```

`ADD` / `SUB` 指令可以发生在：

- 寄存器 到 寄存器
- 内存 到 寄存器
- 寄存器 到 内存
- 寄存器 到 常量数据
- 内存 到 常量数据

但是，像其他指令一样，使用 `ADD` / `SUB` 指令也无法进行存储器到存储器的操作。`ADD` 或 `SUB` 操作设置或清除溢出和进位标志。

栗子：

下面的实例将要求用户输入两位数字，分别将这些数字存储在 **EAX** 和 **EBX** 寄存器中，将这些值相加，将结果存储在 "res" 存储位置中，最后显示结果：

```assembly
SYS_EXIT  equ 1
SYS_READ  equ 3
SYS_WRITE equ 4
STDIN     equ 0
STDOUT    equ 1
segment .data
   msg1 db "Enter a digit ", 0xA,0xD
   len1 equ $- msg1
   msg2 db "Please enter a second digit", 0xA,0xD
   len2 equ $- msg2
   msg3 db "The sum is: "
   len3 equ $- msg3
segment .bss
   num1 resb 2
   num2 resb 2
   res resb 1
section .text
   global _start    ;必须声明 gcc
_start:             ;告诉链接器入口
   mov eax, SYS_WRITE
   mov ebx, STDOUT
   mov ecx, msg1
   mov edx, len1
   int 0x80
   mov eax, SYS_READ
   mov ebx, STDIN
   mov ecx, num1
   mov edx, 2
   int 0x80
   mov eax, SYS_WRITE
   mov ebx, STDOUT
   mov ecx, msg2
   mov edx, len2
   int 0x80
   mov eax, SYS_READ
   mov ebx, STDIN
   mov ecx, num2
   mov edx, 2
   int 0x80
   mov eax, SYS_WRITE
   mov ebx, STDOUT
   mov ecx, msg3
   mov edx, len3
   int 0x80
   ; 将第一个数字移动到 eax 寄存器，将第二个数字移动至 ebx
   ; 并减去 ascii 0 将其转换为十进制数
   mov eax, [num1]
   sub eax, '0'
   mov ebx, [num2]
   sub ebx, '0'
   ; 添加 eax 和 ebx
   add eax, ebx
   ; 将 0 添加到以将十进制和转换为 ASCII
   add eax, '0'
   ; 将总和存储在内存位置 res 中
   mov [res], eax
   ; 打印总和
   mov eax, SYS_WRITE
   mov ebx, STDOUT
   mov ecx, res
   mov edx, 1
   int 0x80
exit:
   mov eax, SYS_EXIT
   xor ebx, ebx
   int 0x80
```



运算结果如下：

```assembly
Enter a digit:
3
Please enter a second digit:
4
The sum is:
7
```



### MUL和IMUL

二进制数据相乘有两条指令。`MUL`（乘法）指令处理无符号数据，`IMUL`（整数乘法）处理有符号数据。这两条指令都会影响进位和溢出标志。

`MUL`/`IMUL` 指令的语法如下：

```assembly
MUL/IMUL multiplier
```

在这两种情况下，被乘数都将在一个累加器中，具体取决于被乘数和乘数的大小，并且根据操作数的大小，生成的乘积还将存储在两个寄存器中。

| 编号 |                             情景                             |
| :--: | :----------------------------------------------------------: |
|  1   | **当两个字节相乘时：**被乘数在 **AL** 寄存器中，而乘数在存储器或另一个寄存器中为一个字节。该产品使用 **AX**。乘积的高 8 位存储在 **AH** 中，低 8 位存储在 **AL** 中。<br>![Arithmetic1](https://cankaoshouce.com/znadmin/md/5129/0.jpg) |
|  2   | **当两个单字值相乘时：**被乘数应位于 **AX** 寄存器中，并且乘数是内存或其他寄存器中的一个字。例如，对于 **MUL DX** 之类的指令，必须将乘数存储在 **DX** 中，将被乘数存储在 **AX** 中。结果乘积是一个双字，将需要两个寄存器。高阶（最左侧）部分存储在 **DX** 中，而低阶（最右侧）部分存储在 **AX** 中。<br>![Arithmetic2](https://cankaoshouce.com/znadmin/md/5129/1.jpg) |
|  3   | **当两个双字值相乘时：**当两个双字值相乘时，被乘数应位于 **EAX** 中，并且该乘数是存储在存储器或另一个寄存器中的双字值。生成的乘积存储在 **EDX:EAX** 寄存器中，即，高 32 位存储在 **EDX** 寄存器中，低 32 位存储在 **EAX** 寄存器中。<br>![Arithmetic3](https://cankaoshouce.com/znadmin/md/5129/2.jpg) |

栗子：

```assembly
MOV AL, 10
MOV DL, 25
MUL DL
...
MOV DL, 0FFH; DL= -1
MOV AL, 0BEH; AL = -66
IMUL DL
```

以下实例将 3 乘以 2，并显示结果：

```assembly
section .text
   global _start    ;必须声明 gcc
_start:             ;告诉链接器入口点
   mov  al, '3'
   sub  al, '0'
   mov  bl, '2'
   sub  bl, '0'
   mul  bl
   add  al, '0'
   mov  [res], al
   mov  ecx,msg
   mov  edx, len
   mov  ebx,1   ;文件描述 (stdout)
   mov  eax,4   ;系统调用号 (sys_write)
   int  0x80    ;调用内核
   mov  ecx,res
   mov  edx, 1
   mov  ebx,1   ;文件描述 (stdout)
   mov  eax,4   ;系统调用号 (sys_write)
   int  0x80    ;调用内核
   mov  eax,1   ;系统调用号 (sys_exit)
   int  0x80    ;调用内核
section .data
	msg db "The result is:", 0xA,0xD
	len equ $- msg
segment .bss
	res resb 1
```



### DIV/IDIV

除法运算生成两个元素 —— **商** 和 **余数**。在乘法的情况下，不会发生溢出，因为使用双长度寄存器来保存乘积。然而，在除法的情况下，可能会发生溢出。如果发生溢出，处理器将生成中断。`DIV`（Divide）指令用于无符号数据，`IDIV`（整数除法）用于有符号数据。

`DIV`（Divide）指令用于无符号数据，`IDIV`（整数除法）用于有符号数据。

语法

`DIV` / `IDIV` 指令的格式：

```assembly
DIV/IDIV        divisor
```

被除数在累加器中。两条指令都可以使用 8 位，16 位或 32 位操作数。该操作影响所有 6 个状态标志。

以下部分说明了 3 种操作数大小不同的除法情况：

| 编号 |                             情景                             |
| :--: | :----------------------------------------------------------: |
|  1   | **当除数为 1 个字节时：**假设被除数在 **AX** 寄存器中（16 位）除法后，商进入 **AL** 寄存器，余数进入 **AH** 寄存器![Arithmetic4](https://cankaoshouce.com/znadmin/md/5129/3.jpg) |
|  2   | **当除数为 1 个单字时：**假设被除数为 32 位长，并且在 **DX:AX** 寄存器中高位 16 位在 **DX** 中，低位 16 位在 **AX** 中除法后，16 位商进入 **AX** 寄存器，16 位余数进入 **DX** 寄存器。<br>![Arithmetic5](https://cankaoshouce.com/znadmin/md/5129/4.jpg) |
|  3   | **当除数是双字：**假设被除数为 64 位长，并且在 **EDX:EAX** 寄存器中高位 32 位在 **EDX** 中，低位 32 位在 **EAX** 中除法后，32 位商进入 **EAX** 寄存器，32 位余数进入 **EDX** 寄存器。<br>![Arithmetic6](https://cankaoshouce.com/znadmin/md/5129/5.jpg) |

以下实例将 8 除以 2。**被除数 8** 存储在 16 位 **AX** 寄存器中，**除数 2** 存储在 **8 位 BL 寄存器** 中。

```assembly
section .text
   global _start    ;必须声明 gcc
_start:             ;告诉链接器入口点
   mov  ax,'8'
   sub     ax, '0'
   mov  bl, '2'
   sub     bl, '0'
   div  bl
   add  ax, '0'
   mov  [res], ax
   mov  ecx,msg
   mov  edx, len
   mov  ebx,1   ;文件描述 (stdout)
   mov  eax,4   ;系统调用号 (sys_write)
   int  0x80    ;调用内核
   mov  ecx,res
   mov  edx, 1
   mov  ebx,1   ;文件描述 (stdout)
   mov  eax,4   ;系统调用号 (sys_write)
   int  0x80    ;调用内核
   mov  eax,1   ;系统调用号 (sys_exit)
   int  0x80    ;调用内核
section .data
	msg db "The result is:", 0xA,0xD
	len equ $- msg
segment .bss
	res resb 1
```

## 逻辑运算



处理器指令集提供指令 `AND`、`OR`、`XOR`、`TEST` 和 `NOT` 布尔逻辑，这些逻辑根据程序的需要测试、设置和清除位。

这些指令的格式如下：

| 编号 | 指令 |          格式           |
| :--: | :--: | :---------------------: |
|  1   | AND  | AND 操作数 1，操作数 2  |
|  2   |  OR  |  OR 操作数 1，操作数 2  |
|  3   | XOR  | XOR 操作数 1，操作数 2  |
|  4   | TEST | TEST 操作数 1，操作数 2 |
|  5   | NOT  |      NOT 操作数 1       |

在所有情况下，第一个操作数都可以在寄存器或内存中。第二个操作数可以是寄存器/内存，也可以是立即数（常数）。但是，内存到内存操作是不可能的。这些指令比较或匹配操作数的位，并设置 **CF**，**OF**，**PF**，**SF** 和 **ZF** 标志。

### AND指令

`AND` 指令用于通过执行逐位 `AND` 运算来支持逻辑表达式。如果两个操作数的匹配位都是 1，则按位 `AND` 操作返回 1，否则返回 0。例如：

```assembly
  			Operand1:  0101
            Operand2:  0011
----------------------------
After AND -> Operand1:       0001
```

`AND` 操作可用于清除一个或多个位。例如，假设 **BL** 寄存器包含 **0011 1010** 。如果需要将高位清除为 0，则使用 **0FH** 对其进行 **与运算**。

```assembly
AND BL,  0FH  ; BL 设置为 0000 1010
```

让我们来看另一个例子。如果要检查给定数字是奇数还是偶数，一个简单的测试将是检查数字的最低有效位。如果为 1，则数字为奇数，否则为偶数。

假设数字在 **AL** 寄存器中，我们可以这样写：

```assembly
AND   AL, 01H     ; ANDing with 0000 0001
JZ    EVEN_NUMBER
```

栗子：

```assembly
section .text
   global _start            ;必须声明 gcc
_start:                     ;告诉链接器入口点
   mov   ax, 8h           ;在 ax 中获得 8
   and   ax, 1              ;进行与运算
   jz    evnn
   mov   eax, 4             ;系统调用号 (sys_write)
   mov   ebx, 1             ;文件描述 (stdout)
   mov   ecx, odd_msg       ;消息
   mov   edx, len2          ;消息长度
   int   0x80               ;调用内核
   jmp   outprog
evnn:
   mov   ah,  09h
   mov   eax, 4             ;系统调用号 (sys_write)
   mov   ebx, 1             ;文件描述 (stdout)
   mov   ecx, even_msg      ;消息
   mov   edx, len1          ;消息长度
   int   0x80               ;调用内核
outprog:
   mov   eax,1              ;系统调用号 (sys_exit)
   int   0x80               ;调用内核
section   .data
	even_msg  db  'Even Number!' ;显示偶数的消息
	len1  equ  $ - even_msg
	odd_msg db  'Odd Number!'    ;显示奇数的消息
	len2  equ  $ - odd_msg
```

结果如下：

```assembly
Even Number!
```

用一个奇数位更改 **AX** 寄存器中的值，例如：

```assembly
mov ax, 9h         ; ax 中获取 9
```

结果如下：

```assembly
Odd Number!
```

同样，要清除整个寄存器，您可以将其与 **00H** 进行 **与运算**。

### OR指令

`OR` 指令用于通过执行逐位 `OR` 运算来支持逻辑表达式。如果其中一个或两个操作数的匹配位为 1，则按位 `OR` 运算符返回 1。如果两个位都为 0，则返回 0。

比如：

```assembly
 			 Operand1:     0101
             Operand2:     0011
----------------------------
After OR -> Operand1:    0111
```

**或运算** 可用于设置一个或多个位。例如，假设AL寄存器包含 **0011 1010**，则需要设置 4 个低阶位，您可以将其与值 **0000 1111**（即 **FH**）进行 **或运算**。

```assembly
OR BL, 0FH          ; 设置 BL 为 0011 1111
```

栗子：

下面的实例演示 `OR` 指令。让我们将值 5 和 3 分别存储在 **AL** 和 **BL** 寄存器中，然后是指令。

```assembly
OR AL, BL
```

应在 **AL** 寄存器中存储 7：

```assembly
section .text
   global _start            ;必须设置 gcc
_start:                     ;告诉链接器入口点
   mov    al, 5             ;al 中获得 5
   mov    bl, 3             ;bl 中获得 3
   or     al, bl            ;进行或运算, 结果应为 7
   add    al, byte '0'      ;将十进制转换为 ascii
   mov    [result],  al
   mov    eax, 4
   mov    ebx, 1
   mov    ecx, result
   mov    edx, 1
   int    0x80
outprog:
   mov    eax,1             ;系统调用号 (sys_exit)
   int    0x80              ;调用内核
section    .bss
	result resb 1
```

### XOR指令

`XOR` 指令实现按位异或运算。当且仅当来自操作数的位不同时，`XOR` 运算将结果位设置为 1。如果来自操作数的位相同（均为 0 或均为 1），则将结果位清除为 0。

例如：

```assembly
 			 Operand1:     0101
             Operand2:     0011
----------------------------
After XOR -> Operand1:    0110
```

将操作数与自身进行 `XOR` 会将操作数更改为 0。这用于清除寄存器。

```assembly
XOR     EAX, EAX
```

### TEST指令

`TEST` 指令与 `AND` 运算的工作原理相同，但与 `AND` 指令不同的是，它不会更改第一个操作数。因此，如果我们需要检查寄存器中的数字是偶数还是奇数，我们也可以使用 `TEST` 指令执行此操作，而无需更改原始数字。

```assembly
TEST    AL, 01H
JZ      EVEN_NUMBER
```



### NOT指令

`NOT` 指令实现按位否操作。`NOT` 运算反转操作数中的位。操作数可以在寄存器中，也可以在内存中。

例如：

```assembly
     		 Operand1:    0101 0011
After NOT -> Operand1:    1010 1100
```

# 控制结构

## 跳转指令

汇编语言中的条件执行是通过几个循环和分支指令来完成的。这些指令可以改变程序中的控制流。

在两种情况下观察到有条件执行：

| 编号 |                           条件指令                           |
| ---- | :----------------------------------------------------------: |
| 1    | **无条件跳转**这由 **JMP** 指令执行条件执行通常涉及将控制权转移到当前执行指令后面的指令的地址控制权的转移可以向前，以执行一组新的指令，也可以向后，以重新执行相同的步骤。 |
| 2    | **条件跳转**这由一组跳转指令 **j <condition>**  执行，具体取决于条件条件指令通过中断顺序流来传输控制，并通过更改 IP 中的偏移值来实现 |

### CMP指令

**CMP** 指令比较两个操作数。它通常用于条件执行。此指令基本上是从另一个操作数中减去一个，以比较操作数是否相等。它不会干扰目标或源操作数。它与条件跳转指令一起用于决策。

**CMP** 比较两个数字数据字段。目标操作数可以在寄存器中，也可以在内存中。源操作数可以是常量数据、寄存器或内存。

```assembly
CMP DX,00 ; 将 DX 值与 0 进行比较
JE L7   ; 如果等于，则跳转到标签 L7
.
.
L7: ...
```

**CMP** 通常用于比较计数器值是否已达到需要运行循环的次数。

比如下面的典型条件：

```assembly
INC EDX
CMP EDX, 10; 比较计数器是否达到 10
JLE LP1   ; 如果它小于或等于 10，则跳转到 LP1
```

### 无条件跳转

如前所述，这是通过 **JMP** 指令执行的。条件执行通常涉及将控制权转移到不遵循当前执行指令的指令的地址。控制权的转移可以是前进的（执行新的指令集），也可以是后退的（重新执行相同的步骤）。

**JMP** 指令提供了一个标签名称，控制流将立即转移到该标签名称。JMP 指令的语法是：

```assembly
JMP     label
```

栗子：

```assembly
MOV  AX, 00    ; 将AX初始化为0
MOV  BX, 00    ; 将BX初始化为0
MOV  CX, 01    ; 初始化CX为1
L20:
	ADD  AX, 01    ; 增量AX
	ADD  BX, AX    ; 将AX添加到BX
	SHL  CX, 1     ; 向左移动CX，这反过来使CX的值翻倍
	JMP  L20       ; 重复的语句
```

### 条件跳转

如果在条件跳转中满足某些指定条件，则控制流将转移到目标指令。根据条件和数据，有许多条件跳转指令。

1. 以下是用于算术运算的有符号数据的条件跳转指令：

   |  指令   |           描述            |  标志测试  |
   | :-----: | :-----------------------: | :--------: |
   |  JE/JZ  |     跳转等于或跳转零      |     ZF     |
   | JNE/JNZ |  跳转不等于或跳转不为零   |     ZF     |
   | JG/JNLE | 跳转大于或跳转不小于/等于 | OF, SF, ZF |
   | JGE/JNL | 跳转大于/等于或不小于跳转 |   OF, SF   |
   | JL/JNGE |   跳转小于或不大于/等于   |   OF, SF   |
   | JLE/JNG |    跳少/等于或跳不大于    | OF, SF, ZF |

2. 以下是对用于逻辑运算的无符号数据使用的条件跳转指令：

   |  指令   |           描述            | 标志测试 |
   | :-----: | :-----------------------: | :------: |
   |  JE/JZ  |     跳转等于或跳转零      |    ZF    |
   | JNE/JNZ |  跳转不等于或跳转不为零   |    ZF    |
   | JA/JNBE |   跳转向上或不低于/等于   |  CF, ZF  |
   | JAE/JNB |     高于/等于或不低于     |    CF    |
   | JB/JNAE | 跳到以下或跳到不高于/等于 |    CF    |
   | JBE/JNA | 跳到下面/等于或不跳到上方 |  AF, CF  |

3. 以下条件跳转指令有特殊用途，并检查标志的值：

   |  指令   |             描述             | 标志测试 |
   | :-----: | :--------------------------: | :------: |
   |  JXCZ   |      如果 CX 为零则跳转      |   none   |
   |   JC    |        如果携带则跳转        |    CF    |
   |   JNC   |       如果不携带则跳转       |    CF    |
   |   JO    |       Jump If Overflow       |    OF    |
   |   JNO   |      如果没有溢出则跳转      |    OF    |
   | JP/JPE  |        跳校验或偶校验        |    PF    |
   | JNP/JPO | 跳转无奇偶校验或跳转奇偶校验 |    PF    |
   |   JS    |       跳跃符号（负值）       |    SF    |
   |   JNS   |      跳转无符号（正值）      |    SF    |

   **J <condition>** 指令集的语法：

   ```assembly
   CMP AL, BL
   JE EQU AL
   CMP AL, BH
   JE EQU AL
   CMP AL, CL
   JE EQU AL
   NON_EQUAL: ...
   EQUAL: ...
   ```

   栗子：

   以下程序显示 3 个变量中最大的一个。变量是两位数的变量。3 个变量 `num1`，`num2` 和 `num3` 分别具有值 **47**、**22** 和 **31**：

   ```assembly
   section .text
      global _start         ;必须声明 gcc
   _start:                  ;告诉链接器入口点
      mov   ecx, [num1]
      cmp   ecx, [num2]
      jg    check_third_num
      mov   ecx, [num2]
   check_third_num:
      cmp   ecx, [num3]
      jg    _exit
      mov   ecx, [num3]
   _exit:
      mov   [largest], ecx
      mov   ecx,msg
      mov   edx, len
      mov   ebx,1  ;文件描述 (stdout)
      mov   eax,4  ;系统调用号 (sys_write)
      int   0x80   ;调用内核
      mov   ecx,largest
      mov   edx, 2
      mov   ebx,1  ;文件描述 (stdout)
      mov   eax,4  ;系统调用号 (sys_write)
      int   0x80   ;调用内核
      mov   eax, 1
      int   80h
   section .data
      msg db "The largest digit is: ", 0xA,0xD
      len equ $- msg
      num1 dd '47'
      num2 dd '22'
      num3 dd '31'
   segment .bss
      largest resb 2
   ```

## 循环

**JMP** 指令可用于实现循环。例如，以下代码段可用于执行循环体 10 次。

```assembly
MOV CL, 10
L1:
	<LOOP-BODY>
	DEC	CL
	JNZ	L1
```



但是，处理器指令集包括一组用于实现迭代的循环指令。

基本的 LOOP 指令语法如下：

```assembly
LOOP    label
```

其中，*label* 是标识目标指令的目标标签，如跳转指令中所述。循环指令假定 **ECX** 寄存器包含循环计数。当执行循环指令时，**ECX** 寄存器递减，并且控制跳至目标标签，直到 **ECX** 寄存器的值（即计数器达到零）为止。那么下面的代码可以写成：

```assembly
mov ECX,10
l1:
<loop body>
loop l1
```

栗子：

以下程序在屏幕上打印数字 1 到 9：

```assembly
section .text
   global _start        ;must be declared for using gcc
_start:                 ;tell linker entry point
   mov ecx,10
   mov eax, '1'
l1:
   mov [num], eax
   mov eax, 4
   mov ebx, 1
   push ecx
   mov ecx, num
   mov edx, 1
   int 0x80
   mov eax, [num]
   sub eax, '0'
   inc eax
   add eax, '0'
   pop ecx
loop l1
   mov eax,1             ;system call number (sys_exit)
   int 0x80              ;call kernel
section .bss
	num resb 1
```



# 过程

过程或子程序在汇编语言中非常重要，因为汇编语言程序往往很大。程序由名称标识。在此名称之后，描述了执行定义良好的作业的过程主体。返回语句表示过程结束。

语法：

```assembly
proc_name:
  procedure body
  ...
  ret
```

通过使用 `CALL` 指令从另一个函数调用该过程。`CALL` 指令应将被调用过程的名称作为参数，如下所示：

```assembly
CALL proc_name
```

被调用的过程使用 `RET` 指令将控件返回给调用过程。

让我们编写一个名为 `sum` 的非常简单的过程，该过程将存储在 **ECX** 和 **EDX** 寄存器中的变量相加，并在 **EAX** 寄存器中返回总和：

```assembly
section .text
   global _start        ;必须声明才能使用 gcc
_start:                 ;告诉链接器入口点
   mov  ecx, '4'
   sub  ecx, '0'
   mov  edx, '5'
   sub  edx, '0'
   call sum          ;调用 sum 过程
   mov  [res], eax
   mov  ecx, msg
   mov  edx, len
   mov  ebx,1           ;文件描述 (stdout)
   mov  eax,4           ;系统调用号 (sys_write)
   int  0x80            ;调用内核
   mov  ecx, res
   mov  edx, 1
   mov  ebx, 1          ;文件描述 (stdout)
   mov  eax, 4          ;系统调用号 (sys_write)
   int  0x80            ;调用内核
   mov  eax,1           ;系统调用号 (sys_exit)
   int  0x80            ;调用内核
sum:
   mov     eax, ecx
   add     eax, edx
   add     eax, '0'
   ret
section .data
	msg db "The sum is:", 0xA,0xD
	len equ $- msg
segment .bss
	res resb 1
```

## 栈数据结构

栈是内存中类似数组的数据结构，可以在其中存储数据并从称为栈 "顶部" 的位置删除数据。需要存储的数据被 "推" 到栈中，要检索的数据被从栈中 "弹出" 出来。栈是后进先出的数据结构，即先存储的数据最后检索。

汇编语言为栈操作提供了两条指令：`PUSH` 和 `POP`。这些指令的语法如下:

```assembly
PUSH    operand
POP     address/register
```

栈段中保留的内存空间用于实现栈。寄存器 **SS** 和 **ESP**（或 **SP**）用于实现栈。**SS:ESP** 寄存器指向栈的顶部，该顶部指向插入到栈中的最后一个数据项，其中 **SS** 寄存器指向栈段的开头，而 **SP**（或 **ESP**）将偏移量设置为栈段。

栈实现具有以下特征:

- 只能将字或双字保存到栈中，而不是字节。
- 栈朝反方向增长，即朝着较低的存储器地址增长
- 栈的顶部指向插入栈中的最后一个项目。它指向插入的最后一个字的低字节。

正如我们所讨论的，在将寄存器的值用于某些用途之前将其存储在栈中。它可以通过以下方式完成：

```assembly
; 将 AX 和 BX 寄存器内容保存在堆栈中
PUSH    AX
PUSH    BX
;将寄存器用于其他用途
MOV     AX, VALUE1
MOV     BX, VALUE2
...
MOV     VALUE1, AX
MOV     VALUE2, BX
;使用完之后恢复寄存器原始值
POP     BX
POP     AX
```

栗子：

以下程序显示整个 **ASCII** 字符集。主程序调用一个名为 `display` 的过程，该过程显示 **ASCII** 字符集：

```assembly
section .text
   global _start        ;必须声明才能使用 gcc
_start:                 ;告诉链接器入口点
   call    display
   mov  eax,1           ;系统调用号 (sys_exit)
   int  0x80            ;调用内核
display:
   mov    ecx, 256
next:
   push    ecx
   mov     eax, 4
   mov     ebx, 1
   mov     ecx, achar
   mov     edx, 1
   int     80h
   pop     ecx
   mov  dx, [achar]
   cmp  byte [achar], 0dh
   inc  byte [achar]
loop  next
   ret
section .data
	achar db '0'
```

## 递归

递归过程是调用自身的过程。有两种递归：直接递归和间接递归。在直接递归中，过程调用自身，而在间接递归中，第一个过程调用第二个过程，后者又调用第一个过程。

在许多数学算法中可以观察到递归。例如，考虑一下计算一个数的阶乘的情况。一个数的阶乘由方程式给出：

```assembly
Fact (n) = n * fact (n-1) for n > 0
```

下面的程序展示了如何在汇编语言中实现阶乘 **n**。为了保持程序简单，我们将计算阶乘 **3**。

```assembly
section .text
   global _start         ;必须声明才能使用 gcc
_start:                  ;告诉链接器入口点
   mov bx, 3             ;用于计算阶乘 3
   call  proc_fact
   add   ax, 30h
   mov  [fact], ax
   mov    edx,len        ;消息长度
   mov    ecx,msg        ;消息
   mov    ebx,1          ;文件描述 (stdout)
   mov    eax,4          ;系统调用号 (sys_write)
   int    0x80           ;调用内核
   mov   edx,1           ;消息长度
   mov    ecx,fact       ;消息
   mov    ebx,1          ;文件描述 (stdout)
   mov    eax,4          ;系统调用号 (sys_write)
   int    0x80           ;调用内核
   mov    eax,1          ;系统调用号 (sys_exit)
   int    0x80           ;调用内核
proc_fact:
   cmp   bl, 1
   jg    do_calculation
   mov   ax, 1
   ret
do_calculation:
   dec   bl
   call  proc_fact
   inc   bl
   mul   bl        ;ax = al * bl
   ret
section .data
	msg db 'Factorial 3 is:',0xa
	len equ $ - msg
section .bss
	fact resb 1
```

# 文件管理



系统将任何输入或输出数据视为字节流。有 3 种标准文件流：

- 标准输入（stdin）
- 标准输出（stdout）
- 标准错误（stderr)

> ==文件描述符：==
>
> **文件描述符** 是作为 **文件id** 分配给文件的 **16** 位整数。创建新文件或打开现有文件时，文件描述符用于访问文件。
>
> 标准文件流的文件描述符 `stdin`、`stdout` 和 `stderr` 分别为 **0**、**1** 和 **2**。



> ==文件指针==
>
> **文件指针** 以字节为单位指定文件中后续读/写操作的位置。每个文件都被视为一个字节序列。每个打开的文件都与一个文件指针相关联，该指针指定相对于文件开头的偏移量（以字节为单位）。打开文件时，文件指针设置为 0。



> ==文件处理系统调用==
>
> 下表简要介绍了与文件处理相关的系统调用：
>
> | %eax |   名称    |      %ebx      |    %ecx    |     %edx     |
> | :--: | :-------: | :------------: | :--------: | :----------: |
> |  2   | sys_fork  | struct pt_regs |     -      |      -       |
> |  3   | sys_read  |  unsigned int  |    char    |    size_t    |
> |  4   | sys_write |  unsigned int  | const char |    size_t    |
> |  5   | sys_open  |   const char   |    int     |     int      |
> |  6   | sys_close |  unsigned int  |     -      |      -       |
> |  8   | sys_creat |   const char   |    int     |      -       |
> |  19  | sys_lseek |  unsigned int  |   off_t    | unsigned int |
>
> 使用系统调用所需的步骤与前面讨论的相同：
>
> - 将系统呼叫号码放入 **EAX** 寄存器。
> - 将系统调用的参数存储在寄存器 **EBX**、**ECX** 等中。
> - 调用相关中断（**80h**）。
> - 结果通常返回到 **EAX** 寄存器中。



> ==创建和打开文件==
>
> 要创建和打开文件，请执行以下任务:
>
> - 将系统调用 `sys_creat()` 放入 **EAX** 寄存器中。
> - 将文件名放入 **EBX** 寄存器。
> - 将文件权限放入 **ECX** 寄存器。
>
> 系统调用返回 **EAX** 寄存器中创建的文件的文件描述符，如果出现错误，错误代码位于 **EAX** 寄存器。

> ==打开现有文件==
>
> 要打开现有文件，请执行以下任务:
>
> - 将系统调用 `sys_open()` 放入 **EAX** 寄存器。
> - 将文件名放入 **EBX** 寄存器。
> - 将文件访问模式放入 **ECX** 寄存器。
> - 将文件权限放入 **EDX** 寄存器。
>
> 系统调用返回 **EAX** 寄存器中创建的文件的文件描述符，如果出现错误，错误代码位于EAX寄存器。
>
> 在文件访问模式中，最常用的是：**只读(0)**、**只写(1)** 和 **读写(2)**。



> ==从文件中读取==
>
> 要读取文件，请执行以下任务:
>
> - 将系统调用 `sys_read()` 放入 **EAX** 寄存器中。
> - 将文件描述符放入 **EBX** 寄存器。
> - 将指针指向 **ECX** 寄存器中的输入缓冲区。
> - 将缓冲区大小，即要读取的字节数，放入 **EDX** 寄存器。
>
> 系统调用返回在 **EAX** 寄存器中读取的字节数，如果出现错误，错误代码位于 **EAX** 寄存器。





> ==写入文件==
>
> 要写入文件，请执行以下任务:
>
> *   将系统调用 `sys_write()` 放入 **EAX** 寄存器中。
> *   将文件描述符放入 **EBX** 寄存器。
> *   将指针指向 **ECX** 寄存器中的输出缓冲区。
> *   将缓冲区大小，即要写入的字节数，放入 **EDX** 寄存器。
>
> 系统调用返回写入 **EAX** 寄存器的实际字节数，如果出现错误，错误代码位于 **EAX** 寄存器中。



> ==关闭文件==
>
> 要关闭文件，请执行以下任务:
>
> *   将系统调用 `sys_close()` 放入 **EAX** 寄存器中。
> *   将文件描述符放入 **EBX** 寄存器。
>
> 如果出现错误，系统调用将返回 **EAX** 寄存器中的错误代码。

> ==更新文件==
>
> 要更新文件，请执行以下任务 & minus；
>
> *   将系统调用 `sys_lseek()` 放入 **EAX** 寄存器中。
> *   将文件描述符放入 **EBX** 寄存器。
> *   将偏移值放入 **ECX** 寄存器。
> *   将偏移量的参考位置放入 **EDX** 寄存器。
>
> 参考位置可以是：
>
> *   文件开头 - 值 0
> *   当前位置 - 值 1
> *   文件结尾 - 值 2
>
> 如果出现错误，系统调用将返回 **EAX** 寄存器中的错误代码。

栗子：

以下程序创建并打开名为 _myfile.txt_ 的文件，并在此文件中写入文本 “Welcome to Cankaoshouce.com”。接下来，程序读取文件并将数据存储到名为 `info` 的缓冲区中。最后，它显示存储在 `info` 中的文本。

```assembly
section    .text
   global _start         ;必须声明才能使用 gcc
_start:                  ;告诉链接器入口点
   ; 创建文件
   mov  eax, 8
   mov  ebx, file_name
   mov  ecx, 0777        ;所有人的读、写和执行
   int  0x80             ;调用内核
   mov [fd_out], eax
   ; 写入文件
   mov    edx,len          ;字节数
   mov    ecx, msg         ;消息
   mov    ebx, [fd_out]    ;文件描述
   mov    eax,4            ;系统调用号 (sys_write)
   int    0x80             ;调用内核
   ; 关闭文件
   mov eax, 6
   mov ebx, [fd_out]
   ; 写入指示文件写入结束的消息
   mov eax, 4
   mov ebx, 1
   mov ecx, msg_done
   mov edx, len_done
   int  0x80
   ; 打开文件进行读取
   mov eax, 5
   mov ebx, file_name
   mov ecx, 0             ;用于只读访问
   mov edx, 0777          ;所有人的读、写和执行
   int  0x80
   mov  [fd_in], eax
   ; 读取文件
   mov eax, 3
   mov ebx, [fd_in]
   mov ecx, info
   mov edx, 26
   int 0x80
   ; 关闭文件
   mov eax, 6
   mov ebx, [fd_in]
   int  0x80
   ; 打印信息
   mov eax, 4
   mov ebx, 1
   mov ecx, info
   mov edx, 26
   int 0x80
   mov    eax,1             ;系统调用号 (sys_exit)
   int    0x80              ;调用内核
section    .data
	file_name db 'myfile.txt'
	msg db 'Welcome to Cankaoshouce.com'
	len equ  $-msg
	msg_done db 'Written to file', 0xa
	len_done equ $-msg_done
section .bss
	fd_out resb 1
	fd_in  resb 1
	info resb  26
```



# 内存管理

内核提供 `sys_brk()` 系统调用，用于分配内存，而无需稍后移动内存。此调用在内存中应用程序映像的正后方分配内存。此系统功能允许您设置数据段中的最高可用地址。

此系统调用采用一个参数，这是需要设置的最高内存地址。该值存储在 **EBX** 寄存器中。

如果出现任何错误，`sys_brk()` 返回 **-1** 或返回负错误代码本身。下面的实例演示了动态内存分配。

栗子：

以下程序使用 `sys_brk()` 系统调用分配 **16kb** 内存:

```assembly
section .text
   global _start         ;必须声明才能使用 gcc
_start:                  ;告诉链接器入口点
   mov  eax, 45          ;sys_brk
   xor  ebx, ebx
   int  80h
   add  eax, 16384       ;要保留的字节数
   mov  ebx, eax
   mov  eax, 45          ;sys_brk
   int  80h
   cmp  eax, 0
   jl   exit             ;exit, if error
   mov  edi, eax         ;EDI = highest available address
   sub  edi, 4           ;pointing to the last DWORD
   mov  ecx, 4096        ;number of DWORDs allocated
   xor  eax, eax         ;clear eax
   std                   ;backward
   rep  stosd            ;repete for entire allocated area
   cld                   ;put DF flag to normal state
   mov  eax, 4
   mov  ebx, 1
   mov  ecx, msg
   mov  edx, len
   int  80h              ;打印消息
exit:
   mov  eax, 1
   xor  ebx, ebx
   int  80h
section .data
	msg     db      "Allocated 16 kb of memory!", 10
	len     equ     $ - msg
```

# 内联汇编（c）

## 基础知识

GCC使用AT&T/UNIX汇编语法。其与Intel语法区别较大，主要区别有：

1. 源-目标顺序
   1. Intel：`Op-code dst src`
   2. AT&T：`Op-code src dst`

2. 寄存器命名
   1. AT&T以`%`为前缀，如：使用`eax`写作`%eax`。

3. 立即操作数
   1. 立即操作数以`$`开头，对staic “C”变量也前置`$`。16进制常量.
   1. AT&T立即数的前缀为`0x`
   1. Intel语法后缀`h`
   1. 所以对于16进制数，我们会先看到`$`,然后是`0x`,最后是常量(AT&T语法结构)。
   
4. 操作数大小

   1. AT&T语法中操作数大小取决于操作码最后一个字符。操作码后缀`b`,`w`,`l` 对应 byte(8-bit),  word(16-bit), 和 long(32-bit)。
   2. Intel语法中，通过在操作数（非操作码）前缀 `byte ptr`, `word ptr`, 和 `dword ptr` 实现该功能。
   3. 因此, Intel 之 `mov al, byte ptr foo` 即 `movb foo, %al` 于 AT&T.

5. 内存操作数

   1. Intel语法中基址寄存器（The base register）内于`[`、`]`之间

   2. 而AT&T于`(`、`)` 之间。

   3. 此外，间接内存引用（indirect memory reference）Intel风格为`section:[base + index*scale + disp]` ，改变为`section:disp(base, index, scale)`于 AT&T.

   4. 需指出，当常量使用disp/scale，`$` 无需前置。

      | Intel Code                 | AT&T Code                        |
      | -------------------------- | -------------------------------- |
      | `mov eax,1`                | `movl $1,%eax`                   |
      | `mov ebx,0ffh`             | `movl $0xff,%ebx`                |
      | `int 80h`                  | `int $0x80`                      |
      | `mov ebx, eax`             | `movl %eax, %ebx`                |
      | `mov eax,[ecx]`            | `movl (%ecx),%eax`               |
      | `mov eax,[ebx+3]`          | `movl 3(%ebx),%eax`              |
      | `mov eax,[ebx+20h]`        | `movl 0x20(%ebx),%eax`           |
      | `add eax,[ebx+ecx*2h]`     | `addl (%ebx,%ecx,0x2),%eax`      |
      | `lea eax,[ebx+ecx]`        | `leal (%ebx,%ecx),%eax`          |
      | `sub eax,[ebx+ecx*4h-20h]` | `subl -0x20(%ebx,%ecx,0x4),%eax` |

   



> [!important]
>
> - 内联是什么？
>
> 我们可以指导编译器将函数的代码直接插入调用的位置，这类函数叫做内联函数。
>
> - 内联函数有什么好处？
>
> 内联的方法降低了函数调用的问题。而且如果任何参数是常量的话，在编译器将得到明显优化，而不是所有的内联函数代码都被包含。代码量会更少，取决于具体的情况。为了定义内联函数，我们使用关键字`inline`声明。
>
> - 什么是内联汇编？
>
> 内联汇编是写在内联函数中的汇编过程(assembly routines)。它非常方便、快速，在系统编程中非常有用。我们主要关注学习GCC内联汇编函数的基础格式和用法。要声明内联汇编函数，我们使用关键字`asm`。

## 基本内联汇编

基本内联汇编语法如下：

```c
asm asm_qualifiers ( AssembleInstructions )
```

1. asm:asm不是 ISO C 中的关键字，如果我们开启了 `‑std=c99` 等启用 ISO C 的编译选项，代码将无法成功 编译。然而，内联汇编对于许多 ISO C 程序是必须的，GCC 通过 `__asm` 给程序员开了个后门。使用 `__asm` 替代 asm 可以让程序作为 ISO C 程序成功编译。volatile 和 inline 也有加 `__` 的版本。 
2. asm_qulifiers包括以下两个修饰符：
   1. volatile: 指示编译器不要对 asm 代码段进行优化
   2. inline: 指示编译器尽可能小的假设 asm 指令的大小
3. AssembleInstructions是我们手写的汇编指令。

基本内联汇编的例子如下：

```c
__asm__ __valatile__(
 "movq %rax, %rdi \n\t"
 "movq %rbx, %rsi \n\t"
);
```

 编译器不解析 asm 块中的指令，直接把它们插入到生成的汇编代码中，剩下的任务有汇编器完成。 这个过程有些类似于宏。为了避免我们手写的汇编代码挤在一起，导致指令解析错误，通常在每一条 指令后面都加上`\n\t`获得合适的格式。

编译器不解析 asm 块中的指令的一个推论是：GCC 对我们插入的指令毫不知情。这相当于我们人为 地干涉了 GCC 自动的代码生成，如果我们处理不当，很可能导致最终生成的代码是错误的。考虑以下代码段：

```c
#include <stdio.h>

int main()
{
    unsigned long long sum = 0;
    for (size_t i = 1; i <= 10; ++i)
    {
        sum += i;
    }
    printf("sum: %llu\n", sum);
    return 0;
}
```



运行结果为

```
sum = 55
```

反汇编代码如下：

```assembly
	.file "basic-asm.c"
	.text
	.section .rodata
.LC0:
	.string "sum: %llu\n"
	.text
	.globl main
	.type main, @function
main:
.LFB0:
	.cfi_startproc ### 进 入 函 数
	pushq %rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq %rsp, %rbp
	.cfi_def_cfa_register 6 ### 分 配 局 部 变 量
	subq $16, %rsp
	movq $0, -8(%rbp) ### sum
	movq $1, -16(%rbp) ### i
	jmp .L2
.L3: ### for body
	movq -16(%rbp), %rax ### sum += i
	addq %rax, -8(%rbp)
	addq $1, -16(%rbp) ### ++i
.L2:
	cmpq $10, -16(%rbp) ### for 条 件 判 断
	jbe .L3
	movq -8(%rbp), %rax ### 传 递 参 数 给 printf
	movq %rax, %rsi ### x86-64 通 常 可 以 使 用 6 个 寄 存 器 传 递 参 数
	movl $.LC0, %edi ### 从 做 往 右 依 次 为 %rdi, %rsi, %rdx, %rcx, %r8, %r9
	movl $0, %eax ### 更 多 的 参 数 通 过 堆 栈 传 递
	call printf
	movl $0, %eax
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size main, .-main
	.ident "GCC: (GNU) 10.2.1 20201016 (Red Hat 10.2.1-6)"
	.section .note.GNU-stack,"",@progbits

```

可以看到在 for body 中，变量i被分配到-16(%rbp)中，我们在sum += i前插入这段代码来验证 基本内联汇编的处理过程。

```C
__asm__ __volatile__(
    "movq $100, -16(%rbp)\n\t"
);
```

此时，反汇编代码如下：

```assembly
	.file "basic-asm.c"
	.text
	.section .rodata
.LC0:
	.string "sum: %llu\n"
	.text
	.globl main
	.type main, @function
main:
.LFB0:
	.cfi_startproc
	pushq %rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq %rsp, %rbp
	.cfi_def_cfa_register 6
	subq $16, %rsp
	movq $0, -8(%rbp)
	movq $1, -16(%rbp)
	jmp .L2
.L3:
	#APP ### 可 以 看 到 编 译 器 直 接 将 我 们 的 指 令 插 入 到 了 汇 编 文 件 中
	## 9 "basic-asm.c" 1
	movq $100, -16(%rbp)
	## 0 "" 2
	#NO_APP
	movq -16(%rbp), %rax
	addq %rax, -8(%rbp)
	addq $1, -16(%rbp)
.L2:
	cmpq $10, -16(%rbp)
	jbe .L3
	movq -8(%rbp), %rax
	movq %rax, %rsi
	movl $.LC0, %edi
	movl $0, %eax
	call printf
	movl $0, %eax
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size main, .-main
	.ident "GCC: (GNU) 10.2.1 20201016 (Red Hat 10.2.1-6)"
	.section .note.GNU-stack,"",@progbits
```

运行结果为

```
sum = 100
```

## 拓展内联汇编

### 基本原理和思路

在编译器生成代码的过程是一个动态的过程：

- 变量可能被分配到寄存器（如 rax）中，也可能被分配 到内存中;
-  一个整型字面值可能是 32 位立即数，也可能是 64 位大立即数;
-  可能使用 rax 寄存器，也可 能使用 rbx 寄存器。

程序员任何擅自的篡改都会导致生成错误的代码。

拓展内联汇编从程序员处获取信息，并根据获取的信息调整自己生成代码的行为。比如，程序员要求 将某个变量分配到 rax 寄存器中，编译器就会将该变量分配在 rax 中，并调整其他部分的代码，使程 序员的要求不影响正确代码的生成。 

因此，使用拓展内联汇编的基本思路就是：提供尽可能多的信息给编译器。程序员提供的信息越多， 出错的概率就越小。除了提供信息，程序员还应该清楚地明白 GCC 对内联汇编做的假设和限制。

### 语法结构

基本汇编中我们只有指令。在扩展汇编中，我们可以指定操作对象(operand)。它允许我们指定输入寄存器，输出寄存器及一列受影响（clobbered）寄存器。它不是mandatory to指定寄存器使用，我们可以将麻烦留给GCC而GCC有可能更好的适配GCC的优化机制。

```c
asm asm-qualifiers ( 
    				Assembler Template Code
                    : Output Operands
					[
                        :Input Operands
						[:Clobbers]
                    ]
                   )
asm asm-qualifiers ( 
    			AssemblerTemplate
				:Input Operands
				:Clobbers
				:Goto Labels
				)
```



1. asm、asm-qualifiers和基本内联汇编基本相同。基本内联汇编提供了在汇编中跳转到 C Label 的能力，因此asm_qualifiers中增加了 goto。goto 修饰符只能用于第二种形式中。 
2. Assembler Template Code是程序员手写的汇编指令，但是增加了几种更方便的表示方法。 可以将拓展内联汇编 asm 块看成一个黑盒，我们给一些变量、表达式作为输入，指定一些变量作为 输出，指明我们指令的副作用，运行后这个黑盒会按照我们的要求将结果输出到输出变量中。
3. 使用冒号分割汇编模板、输出操作数组、输入操作数组、clobbered寄存器组，使用逗号分割每个组内的操作数，如果没有输出操作数，但是有两个输入操作数，那么就需要放置两个连续的冒号
4. Output Operands表示输出变量，
5. Input Operands表示输入变量，
6. Clobbers表示副作用（asm 块中可能修改的寄存器、内存）等。 

栗子：

```c
int a=10, b;
asm ("movl %1, %%eax; movl %%eax, %0;"
    :"=r"(b) /* output */
    :"r"(a) /* input */
    :"%eax" /* clobbered register */
    );
```

#### Assembler Template



汇编模板包含一组嵌入到C程序中的指令。格式类似：或者每个指令包围在双引号中，或整组指令包含在双引号中。每个指令也应该以一个分隔符结束。合法的分隔符可以是`\n`和`;`。`\n`可以跟随一个`\t`。C表达式的操作数呈现为 `%0`, `%1` ...等。

#### Operands

每个操作数首先写作一个双引号内的操作数限制符(operand constraint，可选择性填写)。然后跟随操作数对应的 C 表达式 。 即，`"constraint" (C expression)` 。对输出操作数会有一个额外的修饰符。限制符（constraint）主要用于决定操作数的地址模式。他们也被用于指定要使用的寄存器。

如果我们使用超过一个操作数，以逗号`,`分隔。

> [!important]
>
> 在汇编模板中，每个操作数按数字被引用。数字按如下规则排列。如果有n个操作数（包括输入、输出），那么第一个输出操作数是数字`0`，连续增加，最后一个输入操作数是数字`n-1`。最大操作数数量如上一段所述。

输出操作数表达式必须是`lvalues`(32-bit)。输入操作数无此限制。他们必须是表达式。扩展汇编功能是最常用于编译器自身不知晓的机器指令)。如果输出表达式无法被直接寻址（addressed）（比如，它是一个bit-field），我们限制符必须“允许”(allow)一个寄存器。在那种情况下，GCC将使用该寄存器为asm的输出，然后将寄存器内容存储到输出。

如上所述，原始输出操作数必须是只写的；GCC将假设那个操作对象中的值在指令前已失效且无需生成。扩展汇编也支持“输入-输出”或“读-写”操作数。

我们现在看一些例子。我们希望将一个数乘以5。对此我们使用`lea`指令。

```c
asm ("leal (%1,%1,4), %0"
    : "=r" (five_times_x)
    : "r" (x)
    );
```

此处我们的输入是`x`。我们没有指定使用哪个寄存器。GCC会为输入选择一些寄存器用来输入，一个用来输出，执行我们的要求。如果我们希望输入和输出放在（reside）同一个寄存器中，我们可以让GCC来实现。这里我们使用那种"读-写"操作数，通过指定合适的限制符，这里我们来实现它：

```c
asm ("leal (%0,%0,4), %0"
    : "=r" (five_times_x)
    : "0" (x)
    );
```

现在输入和输出操作数在同一个寄存器内了。但我们不知道是哪个寄存器。现在如果我们也想要指定，有一个办法：

```c
asm ("leal (%%ecx,%%ecx,4), %%ecx"
    : "=c" (x)
    : "c" (x)
    );
```

以上三个例子中，我们没有把任何一个寄存器放在受影响列表中。为什么？前两个例子中，GCC决定使用哪个寄存器，因此知道发生了什么改变。在最后一个中，我们不需要将`ecx`放在受影响列表中，gcc知道它会放入x中。因为它可以知道`ecx`的值，它不会被视为受影响的。

#### Clobber List

一些指令会影响一些硬件寄存器。我们必须在受影响列表中列出那些寄存器，即`asm`函数第三个`:`后的区域。这用于指示gcc我们将使用并修改它们。所以gcc将不会假设它加载到这些寄存器中的值是合法的。我们不应该列出输入和输出寄存器。因为gcc知道`asm`使用它们（因为它们被明确指定为限制符(constraints)）。如果指令使用了任何其他寄存器，显式或隐式的（并且这些寄存器没有出现在输入和输出列表上），那么那些寄存器必须在受影响列表中指定。

如果我们的指令可以修改条件码寄存器（the condition code register），我们必须增加`cc`到受影响寄存器列表。

如果我们的指令用一个不可预期的方法(fashion)修改了内存，添加`memory`到受影响寄存器。这会使GCC在汇编指令期间不在寄存器内保持内存值的缓存。我们也必须添加**`volatile`**关键字，如果内存影响（memory affected）未列在`asm`的输入和输出中。

我们可以读写受影响寄存器任意多次。注意模板中乘法指令的例子；它假设子过程(subroutine) `_foo` 接受`eax` 、`ecx`寄存器中的参数。

```c
asm ("movl %0,%%eax; movl %1,%%ecx; call _foo"
    : /* no outputs */
    : "g" (from), "g" (to)
    : "eax", "ecx"
    );
```

> [!note]
>
> volatile(不稳定的)
>
> 如果你熟悉内核源码或者一些类似的优美代码，你必然已见过很多函数声明为`volatile`或`__volatile__`，跟随在`__asm__`之后。我之前提到过关于关键字`asm`和`__asm__`。所以什么是`volatile`？
>
> 如果我们的汇编语句必须在我们放置它的地方执行，(即，必须不被作为一个优化而移出循环)，则将`volatile`放在`asm`之后。所以防止它被移动、删除和任何改变，我们如此声明`asm volatile(... : ... : ... : ...);` 当我们必须非常小心时，使用`__volatile__`。
>
> 如果我们的汇编只是做一些计算而没有任何副作用，最好不要使用`volatile`关键字。忽略它将帮助GCC优化代码使其更优美。

#### constraints

##### 限制符修饰符

当使用限制符时，若要精确控制其效果，GCC提供了修饰符。常用当有：

- `=` : 意味着操作数对该指令是只写的；前一个值将被忽略并替换为输出数据。
- `&`: 意味着操作数是一个早期受影响的操作数，也就是在指令结束前已被修改。因此，该操作数不可停留在输入寄存器中或任何内存中。在被写入前仅用于输入的输入操作数可设为一个早期受影响操作数 （An input operand can be tied to an earlyclobber operand if its only use as an input occurs before the early result is written）。

##### 寄存器操作数限制符

当操作数指定使用此限制符时，它们会存储在常规寄存器中（General Purpose Registers(GPR)）。如：



```c
asm ("movl %%eax, %0\n"
    :"=r"(myval)
    );
```

此处`myval`变量保存在一个寄存器中，`eax`的值会复制到那个寄存器，而`myval`的值会从这个寄存器中更新到内存。当`"r"`限制符被指定后，gcc可以在任何可用的GPR中保存这个变量。要指定该寄存器，你必须使用特定寄存器限制符指定寄存器名称。它们是：

| 限制符 | 作用寄存器     |
| ------ | -------------- |
| r      | Register (s)   |
| a      | %eax, %ax, %al |
| b      | %ebx, %bx, %bl |
| C      | %ecx, %cx, %cl |
| d      | %edx, %dx, %dl |
| S      | %esi, %si      |
| D      | %edi, %di      |

##### 内存操作数限制符

当操作数是在内存中时，那么任何在它上的操作将直接在内存位置进行。而寄存器限制符，则优先存于寄存器而后修改再写回内存。但寄存器限制符通常只在指令必需或者明显提升性能时使用。当C变量需在`asm`中修改且无需寄存器保持其值时，内存限制符可最大化性能。如，将`idtr`的值存储于`loc`的内存位置中：

```c
asm("sidt %0\n" : :"m"(loc));
```

##### 匹配（数字）限制符

有时，一个单独变量既是输入也是输出操作符，这时可使用匹配限制符。

```c
asm ("incl %0" :"=a"(var):"0"(var));
```

我们在操作数一节看到了类似的例子，在这个例子中寄存器`%eax`既是输入也是输出变量。`var`输入读入`%eax`并更新到`%eax`最后在自增后存入`var`。这里的`"0"`指定了和输出变量一样的第0个限制符。也就是说，它指定了var的输出过程应该只存于`%eax`中。这类限制符可用于：

- 输入输出是统一变量，或变量被修改并被写会同一变量时。
- 将输入和输出操作符分开是不必要的时候。

##### 其他限制符

- `m`: 接受内存操作数，任意的机器支持的地址。
- `o`: 接受内存操作数，只接受偏移地址（offsettable）。即对某个合法地址添加一个微小的偏移量。
- `V`: 非偏移内存操作数。换句话说，任何符合`"m"`但不符合`"o"`限制符的地址。
- `i`: 立即整型操作数，允许在编译期(assembly-time)可知常量符号。
- `n`: 立即整型操作数，允许已知数字值。许多系统不支持小于16-bit的（word wide)编译期(assembly-time)常量作为操作数。这些操作数应该使用`n`而不是`i`。
- `g`: 任何寄存器，内存或立即整型操作数都可用，要求寄存器不是常规寄存器(general registers)。

x86限定：

- `r` : Register operand constraint, look table given above.
- `q` : Registers a, b, c or d.
- `I` : Constant in range 0 to 31 (for 32-bit shifts).
- `J` : Constant in range 0 to 63 (for 64-bit shifts).
- `K` : 0xff.
- `L` : 0xffff.
- `M` : 0, 1, 2, or 3 (shifts for lea instruction).
- `N` : Constant in range 0 to 255 (for out instruction).
- `f` : Floating point register
- `t` : First (top of stack) floating point register
- `u` : Second floating point register
- `A` : Specifies the `a’ or`d’ registers. This is primarily useful for 64-bit integer values intended to be returned with the `d’ register holding the most significant bits and the`a’ register holding the least significant bits.



### 栗子





```c
/* Assembly function to jump execution to a location */
__attribute__( ( naked, noreturn ) ) void BootJumpASM( uint32_t SP, uint32_t RH )
{
		__asm("MSR      MSP,r0");
		__asm("BX       r1");
}
```

在C语言中，关键字`naked`使得拓展汇编写法更加简洁，使用 `naked` 属性表示这个函数不需要标准的函数序列，如保存返回地址或帧指针，也不需要进行常规的参数传递。因此，函数的实现完全依赖于汇编，省略了通常的输入/输出约束，直接操作寄存器中的值。

- 第一个参数 (`SP`) 被传递到 `r0` 寄存器。
- 第二个参数 (`RH`) 被传递到 `r1` 寄存器。

> [!tip]
>
> *使用Naked之后：
>
> **禁用栈帧**： 使用 `naked` 后，编译器不会为函数生成栈帧，因此栈指针（`SP`）不会被修改，寄存器 `r0` 和 `r1` 的值可以直接传递到汇编指令中使用，而无需在函数内部处理栈。
>
> **适用于嵌入式和裸机编程**： 例如在启动代码中，`naked` 常常用于设定堆栈指针并进行跳转，或者在中断服务程序中，编译器生成的栈操作和局部变量等会占用不必要的空间，所以通常会使用 `naked` 来省略这些操作。
>
> **裸函数的注意事项**：
>
> - 由于没有自动保存寄存器和栈帧，你需要显式地管理这些。否则，寄存器的值会丢失，函数调用栈可能会被破坏。
> - 如果函数需要返回值或调用其他函数，开发者需要手动控制堆栈和返回地址。

