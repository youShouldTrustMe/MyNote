# 参考链接

[KEIL MDK .map文件分析(必学知识点) - 正点原子倾力打造！-OpenEdv-开源电子网](http://www.openedv.com/thread-305913-1-1.html)

[充分理解Linux GCC 链接生成的Map文件 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/502051758)

> [!important]
>
> 可以使用AMAP可视化map文件



# GCC平台

## Archives linked

一般位于map文件的第一行

```
 Archive member included to satisfy reference by file (symbol)
    /usr/local/Cellar/arm-none-eabi-gcc/8-2018-q4-major/gcc/bin/../lib/gcc/arm-none-eabi/8.2.1/../../../../arm-none-eabi/lib/thumb/v7e-m+fp/hard/libc_nano.a(lib_a-exit.o)
                                  /usr/local/Cellar/arm-none-eabi-gcc/8-2018-q4-major/gcc/bin/../lib/gcc/arm-none-eabi/8.2.1/../../../../arm-none-eabi/lib/thumb/v7e-m+fp/hard/crt0.o (exit)
```

上面的文件信息格式如下：

```
The archive file location (compilation unit)
    			The compilation unit referencing the archive (symbol called)
```

上面内容的意思是crt0这个文件中会调用exit函数，exit函数在exit.o这个目标文件中，exit.o目标文件是被链接在libc_nano.a这个库文件里的。

## Memory configuration

Map文件中最直接的信息是实际的内存区域，这些区域具有位置、大小和访问权限:

```
Memory Configuration
    
    Name             Origin             Length             Attributes
    FLASH            0x0000000000001000 0x00000000000ff000 xr
    RAM              0x0000000020000008 0x000000000003fff8 xrw
    *default*        0x0000000000000000 0xffffffffffffffff
```

## Linker script and memory map

内存配置之后是Linker script and memory map，它给出了程序中符号的详细信息。一般来说，该段首先指示`text`区域的大小及其内容(`text`是编译的代码，而`data`是程序数据)。

在这里，中断向量(在.isr_vector下)出现在可执行文件的开头，定义在gcc_startup_nrf52840.S中:

```
Linker script and memory map
    
    .text           0x0000000000001000      0x8c8
     ***(.isr_vector)**
     .isr_vector    0x0000000000001000      0x200 _build/nrf52840_xxaa/**gcc_startup_nrf52840.S.o**
                    0x0000000000001000                __isr_vector
     *(.text*)
     .text          0x0000000000001200       0x40 /usr/local/Cellar/arm-none-eabi-gcc/8-2018-q4-major/gcc/bin/../lib/gcc/arm-none-eabi/8.2.1/thumb/v7e-m+fp/hard/crtbegin.o
     .text          0x0000000000001240       0x74 /usr/local/Cellar/arm-none-eabi-gcc/8-2018-q4-major/gcc/bin/../lib/gcc/arm-none-eabi/8.2.1/../../../../arm-none-eabi/lib/thumb/v7e-m+fp/hard/crt0.o
                    0x0000000000001240                _mainCRTStartup
                    0x0000000000001240                _start
     .text          0x00000000000012b4       0x3c _build/nrf52840_xxaa/gcc_startup_nrf52840.S.o
                    0x00000000000012b4                Reset_Handler
                    0x00000000000012dc                NMI_Handler
    
    [...]
    
     .text.**bsp_board_led_invert**
                    0x00000000000012f0       **0x34** _build/nrf52840_xxaa/**boards.c.o**
                    0x00000000000012f0                bsp_board_led_invert
```

这些行给出了每个函数的地址和大小。在上面，你可以读取bsp_board_led_invert的地址，它来自boards.c.o(如你所猜测的，board.c的编译单元)，在`text`区域中它的大小为0x34字节。这样，我们就可以定位程序中使用的每个函数。

我的常量字符串_delay_ms_str在程序初始化时显然包含在程序中，只读数据作为rodata保存在链接器脚本中指定的FLASH区域中(存储在Flash中，而不是复制在RAM中，因为它是常量)。我可以在这行下面找到:



# MDK平台

.map文件是编译器链接时生成的一个文件，它主要包含了交叉链接信息。通过.map文件，我们可以知道整个工程的函数调用关系、FLASH和RAM占用情况及其详细汇总信息，能具体到单个源文件（.c/.s）的占用情况，根据这些信息，我们可以对代码进行优化。

.map文件可以分为以下5个组成部分：

1. 程序段交叉引用关系（Section Cross References）
2. 删除映像未使用的程序段（Removing Unused input sections from the image）
3. 映像符号表（Image Symbol Table）
4. 映像内存分布图（Memory Map of the image）
5. 映像组件大小（Image component sizes）

## 基础概念

为了更好的分析map文件，我们先对需要用到的一些基础概念进行一个简单介绍，相关概念如下：

- Section：描述映像文件的代码或数据块，我们简称程序段
- RO：Read Only的缩写，包括只读数据（RO data）和代码（RO code）两部分内容，占用FLASH空间
- RW：Read Write的缩写，包含可读写数据（RW data，有初值，且不为0），占用FLASH（存储初值）和RAM（读写操作）
- ZI：Zero initialized的缩写，包含初始化为0的数据（ZI data），占用RAM空间。
- .text：相当于RO code
- .constdata：相当于RO data
- .bss：相当于ZI data
- .data：相当于RW data

## 组成部分

### 程序段交叉引用关系（Section Cross References）

 这部分内容描述了各个文件（.c/.s等）之间函数（程序段）的调用关系：

```
Section Cross References

    ble_simple_peripheral.o(i.app_gap_evt_cb) refers to os_timer.o(i.os_timer_start) for os_timer_start
    ble_simple_peripheral.o(i.app_gap_evt_cb) refers to gap_api.o(i.gap_start_advertising) for gap_start_advertising
    ble_simple_peripheral.o(i.app_gap_evt_cb) refers to gap_api.o(i.gap_get_connect_num) for gap_get_connect_num
    ble_simple_peripheral.o(i.app_gap_evt_cb) refers to os_timer.o(i.os_timer_stop) for os_timer_stop
```

 上述描述段中，` ble_simple_peripheral.o(i.app_gap_evt_cb) refers to os_timer.o(i.os_timer_stop) for os_timer_stop`表示：ble_simple_peripheral.c文件中的app_gap_evt_cb函数调用了os_timer.c中的os_timer_stop函数。其中：i.app_gap_evt_cb表示app_gap_evt_cb函数的入口地址，同理i.os_timer_stop表示os_timer_stop的入口地址。

### 删除映像未使用的程序段（RemovingUnused input sections from the image）

这部分内容描述了工程中由于未被调用而被删除的冗余程序段（函数/数据)



```
Removing Unused input sections from the image.

    Removing ble_simple_peripheral.o(.rev16_text), (4 bytes).
    Removing ble_simple_peripheral.o(.revsh_text), (4 bytes).
    Removing ble_simple_peripheral.o(.rrx_text), (6 bytes).
    Removing proj_main.o(.rev16_text), (4 bytes).
    Removing proj_main.o(.revsh_text), (4 bytes).
    Removing proj_main.o(.rrx_text), (6 bytes).
    Removing driver_efuse.o(.rev16_text), (4 bytes).
    Removing driver_efuse.o(.revsh_text), (4 bytes).
    Removing driver_efuse.o(.rrx_text), (6 bytes).
    Removing driver_efuse.o(i.efuse_get_chip_unique_id_new), (182 bytes).
    Removing driver_efuse.o(i.efuse_write), (40 bytes).
    ......
    Removing dmul.o(.text), (228 bytes).
    Removing dfixul.o(.text), (48 bytes).
    Removing cdrcmple.o(.text), (48 bytes).

1183 unused section(s) (total 85261 bytes) removed from the image.
```

上面片段中，列出了所有被移除的程序段，比如driver_efuse.c里面的efuse_write函数就被移除了，因为该例程没用到delay_us函数。

另外，在最后还有一个统计信息：1183 unused section(s) (total 85261 bytes) removed from the image.表示总共移除了1183 个程序段（函数/数据），大小为85261字节。即给我们的MCU节省了85261字节的程序空间。



>  [!note] 
>
> 为了更好的节省空间，我们一般在MDK魔术棒C/C++选项卡里面勾选：One ELF Section per Function



### **向映像中添加跳转代码**(Adding Veneers to the image)

在嵌入式系统开发中，特别是在使用ARM架构的微控制器时，`veneer`（翻译为“贴面”或“桥接”）是一种用于处理函数调用的机制。它通常用于处理跨不同内存区域（如从Thumb代码调用ARM代码，或从非安全代码调用安全代码）的函数调用。

```
Adding Veneers to the image

    Adding TT veneer (10 bytes, Long) for call to 'ke_state_set' from app_proj_event.o(i.app_act_created_ind_func).
    Adding TT veneer (10 bytes, Long) for call to 'srand' from app_proj_event.o(i.app_db_init_complete_ind_func).
    Adding TT veneer (10 bytes, Long) for call to 'co_printf' from ble_simple_peripheral.o(i.app_gap_evt_cb).
    ...
    Adding TT veneer (10 bytes, Long) for call to 'mesh_fatal_error_report' from m_al_scan.o(ram_code).

146 Veneer(s) (total 1460 bytes) added to the image.
```

在上面的代码段中，`ke_state_set`：从`app_proj_event.o`中的`app_act_created_ind_func`函数调用`ke_state_set`时，链接器添加了一个`veneer`。

### 映像符号表（ImageSymbol Table）

映像符号表（Image Symbol Table）描述了被引用的各个符号（程序段/数据）在存储器中的存储地址、类型、大小等信息。映像符号表分为两类：

1. 本地符号（Local Symbols）
2. 全局符号（Global Symbols）。

#### 本地符号（Local Symbols）

​    本地符号（Local Symbols）记录了用static声明的全局变量地址和大小，c文件中函数的地址和用static声明的函数代码大小，汇编文件中的标号地址（作用域：限本文件)：

```
Image Symbol Table

    Local Symbols

    Symbol Name                              Value     Ov Type        Size  Object(Section)

    ../clib/microlib/division.c              0x00000000   Number         0  uldiv.o ABSOLUTE
    ../clib/microlib/division.c              0x00000000   Number         0  uidiv.o ABSOLUTE
    ../clib/microlib/longlong.c              0x00000000   Number         0  llshl.o ABSOLUTE
    ../clib/microlib/longlong.c              0x00000000   Number         0  llushr.o ABSOLUTE
    ../clib/microlib/longlong.c              0x00000000   Number         0  llsshr.o ABSOLUTE
    i.gatt_clr_peer_svc                      0x01009ff4   Section        0  gatt_api.o(i.gatt_clr_peer_svc)
    gatt_clr_peer_svc                        0x01009ff5   Thumb Code    58  gatt_api.o(i.gatt_clr_peer_svc)
```

上述片段中，表示gatt_api.c文件中的gatt_clr_peer_svc函数的入口地址为：0x01009ff4，类型为：Section（程序段），大小为0。因为：i.gatt_clr_peer_svc仅仅表示gatt_clr_peer_svc函数入口地址，并不是指令，所以没有大小。在全局符号段，会列出gatt_clr_peer_svc函数的大小。

#### 全局符号（Global Symbols）

全局符号（Global Symbols）记录了全局变量的地址和大小，C文件中函数的地址及其代码大小，汇编文件中的标号地址（作用域：全工程）。

```
Global Symbols

    Symbol Name                              Value     Ov Type        Size  Object(Section)

    BuildAttributes$$THM_ISAv4$P$D$K$B$S$PE$A:L22UL41UL21$X:L11$S22US41US21$IEEE1$IW$USESV6$~STKCKD$USESV7$~SHL$OTIME$ROPI$EBA8$MICROLIB$REQ8$PRES8$EABIv2 0x00000000   Number         0  anon$$obj.o ABSOLUTE
    __ARM_use_no_argv                        0x00000000   Number         0  syscall.txt ABSOLUTE
    __Vectors                                0x00000000   Data           0  syscall.txt ABSOLUTE
    _printf_a                                0x00000000   Number         0  stubs.o ABSOLUTE
    _printf_c                                0x00000000   Number         0  stubs.o ABSOLUTE
    _printf_charcount                        0x00000000   Number         0  stubs.o ABSOLUTE
    _printf_d                                0x00000000   Number         0  stubs.o ABSOLUTE
    ...
    adc_set_ref_voltage                      0x010008d1   Thumb Code     2  driver_pmu.o(i.adc_set_ref_voltage)
    app_act_created_ind_func                 0x010008d5   Thumb Code    66  app_proj_event.o(i.app_act_created_ind_func)
    app_act_deleted_ind_func                 0x0100091d   Thumb Code    30  app_proj_event.o(i.app_act_deleted_ind_func)
    app_addr_resolve_result_ind_func         0x01000941   Thumb Code    28  app_proj_event.o(i.app_addr_resolve_result_ind_func)
    app_adv_complete_ind_func                0x01000961   Thumb Code    48  app_proj_event.o(i.app_adv_complete_ind_func)
    app_adv_report_ind_func                  0x01000995   Thumb Code   160  app_proj_event.o(i.app_adv_report_ind_func)
```

` app_act_created_ind_func   0x010008d5   Thumb Code  66  app_proj_event.o(i.app_act_created_ind_func)`表示app_proj_event.c文件中的app_act_created_ind_func函数的入口地址为：0x010008d5，类型为：Thumb Code（程序段），大小为66字节。  

> [!note] 
>
> 注意，全局符号的地址和局部符号的地址不符，假如全局符号用的0x0800046D，那么局部符号就用0x0800046C，这是因为ARM规定Thumb指令级的所有指令，其最低位必须为1，0x0800046D=0x0800046C+1，所以才会有2个不同的地址，且总是差1，实际上就是同一个函数。

### 映像内存分布图（MemoryMap of the image）

映像文件分为加载域（Load Region）和运行域（Execution Region），一个加载域必须有至少一个运行域（可以有多个运行域），而一个程序又可以有多个加载域。加载域为映像程序的实际存储区域，而运行域则是MCU上电后的运行状态。加载域和运行域的简化关系（这里仅表示一个加载域的情况）如下图所示：

![加载域和运行域的关系](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2025/01/14_8_17_40_202501140817123.png)

 由图可知，RW区也是存放在ROM（FLASH）里面的，在执行main函数之前，RW（有初值且不为0的变量）数据会被拷贝到RAM区，同时还会在RAM里面创建ZI区（初始化为0的变量）。

映像内存分布如下：

```
Memory Map of the image

  Image Entry point : 0x01000f3d
```

上述片段表示映像的入口地址，也就是整个程序运行的起始地址，为：0x01000f3d。实际地址为：0x01000f3c（Thumb指令最低位是1）。

```
  Load Region ROM (Base: 0x01000000, Size: 0x000184cc, Max: 0x00030000, ABSOLUTE)
```

上述片段表示ROM加载域，其起始地址为：0x01000000；占用大小为：0x000184cc；最大地址范围为：0x00030000。其内部包含两个运行域：ER_TABLE和ER_RO（由下述片段可以看出）。

```
   Execution Region ER_TABLE (Exec base: 0x01000000, Load base: 0x01000000, Size: 0x0000005c, Max: 0xffffffff, ABSOLUTE)

    Exec Addr    Load Addr    Size         Type   Attr    Idx    E Section Name        Object

    0x01000000   0x01000000   0x00000004   Data   RO       1542    jump_table_0        fr8010h_stack.lib(jump_table.o)
    0x01000004   0x01000004   0x00000008   Data   RO       127    jump_table_1        proj_main.o
    0x0100000c   0x0100000c   0x0000000c   Data   RO       1543    jump_table_2        fr8010h_stack.lib(jump_table.o)
    0x01000018   0x01000018   0x00000004   Data   RO       128    jump_table_3        proj_main.o
    0x0100001c   0x0100001c   0x00000040   Data   RO       1544    jump_table_4        fr8010h_stack.lib(jump_table.o)
```

上述片段表示ER_TABLE运行域，其起始地址为：0x01000000；占用大小为：0x0000005c；最大地址范围为：0xffffffff；即内部FLASH运行域，所有需要放内部FLASH的代码，都应该放到这个运行域里面。

```
    Execution Region ER_RO (Exec base: 0x0100005c, Load base: 0x0100005c, Size: 0x000156bc, Max: 0xffffffff, ABSOLUTE)

    Exec Addr    Load Addr    Size         Type   Attr      Idx    E Section Name        Object

    0x0100005c   0x0100005c   0x000000b0   Code   RO         5006    .text               mf_w.l(fadd.o)
    0x0100010c   0x0100010c   0x00000064   Code   RO         5008    .text               mf_w.l(fmul.o)
    0x01000170   0x01000170   0x0000007c   Code   RO         5010    .text               mf_w.l(fdiv.o)
    0x010001ec   0x010001ec   0x0000014e   Code   RO         5012    .text               mf_w.l(dadd.o)
    0x0100033a   0x0100033a   0x000000de   Code   RO         5014    .text               mf_w.l(ddiv.o)
    0x01000418   0x01000418   0x00000026   Code   RO         5016    .text               mf_w.l(f2d.o)
    0x0100043e   0x0100043e   0x00000038   Code   RO         5018    .text               mf_w.l(d2f.o)
    0x01000476   0x01000476   0x0000001e   Code   RO         5024    .text               mc_w.l(llshl.o)
    0x01000494   0x01000494   0x00000024   Code   RO         5026    .text               mc_w.l(llsshr.o)
    0x010004b8   0x010004b8   0x00000000   Code   RO         5028    .text               mc_w.l(iusefp.o)
    0x010004b8   0x010004b8   0x0000006e   Code   RO         5029    .text               mf_w.l(fepilogue.o)
    0x01000526   0x01000526   0x000000ba   Code   RO         5031    .text               mf_w.l(depilogue.o)
    0x010005e0   0x010005e0   0x00000020   Code   RO         5039    .text               mc_w.l(llushr.o)
```

上述片段表示ER_RO运行域，其起始地址为：0x0100005c；占用大小为：0x000156bc；最大地址范围为：0xffffffff；即内部SRAM运行域，所有RAM（包括RW和ZI）都是放在这个运行域里面。

上面只列举了部分加载域及其运行域的具体内存分布，我们可以很方便的查看任何一个函数所在的运行域、入口地址、占用空间等信息。

### 映像组件大小（Imagecomponent sizes）

映像组件大小（Image component sizes）给出了整个映像所有代码（.o）占用空间的汇总信息。

```
Image component sizes

      Code (inc. data)   RO Data    RW Data    ZI Data      Debug   Object Name

        96          8        128          0          0        576   app_boot_vectors.o
       860        454         12         26         20      24402   ble_simple_peripheral.o
        32          8          0         24         60       2298   button.o
       100         60          0          0          0        590   core_cm3_isr.o
       198          4          0          0          0       2340   driver_efuse.o
        32         20          0          0          0       2843   driver_exti.o
        48          4          0          4          0       3057   driver_i2s.o
        32          0          0          0          0        622   driver_keyscan.o
      1786         98          0          0          0       6985   driver_pmu.o
        64         10          0          0          0       2194   driver_pmu_qdec.o
         2          0          0          0          0        498   driver_rtc.o
        92         12          0          0          0       2534   driver_timer.o
       384        158          0          0          0       1809   driver_wdt.o
      1116         52          0        256         24      30602   patch.o
       276         36         12          0          0      12900   proj_main.o
       556        210        591          4         12       4867   simple_gatt_service.o
        96         38        489          2          0       2044   user_task.o

    ----------------------------------------------------------------------
      7236       1172       1236        332        116     101161   Object Totals
      1460          0          0          0          0          0   (incl. Generated)
         6          0          4         16          0          0   (incl. Padding)
    ----------------------------------------------------------------------
```

上述片段表示.c/.s文件生成对象所占空间大小（单位：字节，下同），即.c/.s文件编译后所占代码空间的大小。每个项所代表的意义如下：

- Code（inc.data）：表示包含内联数据（inc.data）后的代码大小。如app_boot_vectors.o（即app_boot_vectors.c）所占的Code大小为576字节，其中 8 字节是内联数据。
- RO Data：表示只读数据所占的空间大小，一般是指const修饰的数据大小。
- RW Data：表示有初值（且非0）的可读写数据所占的空间大小，它同时占用FLASH和RAM空间。
- ZI Data：表示初始化为0的可读写数据所占空间大小，它只占用RAM空间。
- Debug：表示调试数据所占的空间大小，如调试输入节及符号和字符串。
- Object Totals：表示以上部分链接到一起后，所占映像空间的大小。
- (incl.Generated)：表示链接器生产的映像内容大小，它包含在Object Totals里面了，这里仅仅是单独列出，我们一般不需要关心。
- (incl.Padding)：表示链接器根据需要插入填充以保证字节对齐的数据所占空间的大小，它也包含在Object Totals里面了，这里单独列出，一般无需关心。

```
      Code (inc. data)   RO Data    RW Data    ZI Data      Debug   Library Member Name

      1368         44          0        102          0        480   adv_check_mode.o
      2560        194          4         12        204       2212   app.o
       432         56          0          8         20        296   app_conn.o
        40          4          0          0          0         84   app_prf.o
      1426        162          0          1          0       2140   app_proj_event.o
      1546        244         56         16         82        952   app_sec.o
      2308        188        228          9          0       2316   app_task.o
      2482         22        112          0          0       1444   attc.o
      1476         26          0          0          0        844   attm.o
      1890         28          0          0          0        988   attm_db.o
      4668         76        140          0          0       2656   atts.o
      1782        206          0         16         40        576   entry.o
       968         36         16          0          0        312   flash.o
       ...
       100          0          0          0          0         76   fmul.o

    ----------------------------------------------------------------------
     86908       4750       3124        696       3192      63128   Library Totals
       128          0          7         11          2          0   (incl. Padding)

    ----------------------------------------------------------------------
    
     Code (inc. data)   RO Data    RW Data    ZI Data      Debug   Library Name

     85336       4750       3117        685       3190      61872   fr8010h_stack.lib
        98          0          0          0          0        204   mc_w.l
      1346          0          0          0          0       1052   mf_w.l

    ----------------------------------------------------------------------
     86908       4750       3124        696       3192      63128   Library Totals

    ----------------------------------------------------------------------
```

上述片段表示被提取的库成员（.lib）添加到映像中的部分所占空间大小。各项意义同object中的说明。我们一般只用看LibraryTotals来分析库所占空间的大小即可。

```
==============================================================================
      Code (inc. data)   RO Data    RW Data    ZI Data      Debug   

     94144       5922       4360       1028       3308     123677   Grand Totals
     94144       5922       4360       1028       3308     123677   ELF Image Totals
     94144       5922       4360       1028          0          0   ROM Totals
==============================================================================

    Total RO  Size (Code + RO Data)                98504 (  96.20kB)
    Total RW  Size (RW Data + ZI Data)              4336 (   4.23kB)
    Total ROM Size (Code + RO Data + RW Data)      99532 (  97.20kB)

==============================================================================
```

上述片段表示本工程全部程序汇总后的占用情况。其中：

- Grand Totals：表示整个映像所占空间大小。
- ELF Image Totals：表示ELF可执行链接格式映像文件的大小，一般和Grand Totals一样大小。
- ROM Totals：表示整个映像所需要的ROM空间大小，不含ZI和Debug数据。
- Total RO Size：表示Code和RO数据所占空间大小，本例程为：98504字节。
- Total RW Size：表示RW和ZI数据所占空间大小，即本映像所需SRAM空间的大小，本例程为：4336字节。
- Total ROM Size：表示Code、RO和RW数据所占空间大小，即本映像所需FLASH空间的大小，本例程为：99532字节。













































