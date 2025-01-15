# 参考链接

[可视化的Keil工程配置模板，一招提高单片机开发效率-CSDN博客](https://blog.csdn.net/u010632165/article/details/124162184)

[Keil5 Configuration Wizard配置向导的使用方法_keil configuration wizard-CSDN博客](https://blog.csdn.net/zxy18791750520/article/details/140120610)

# 实现

Configuration Wizard可以实现可视化配置宏定义文件。

首先在官方文档（[Keil Product Manuals](https://www.keil.com/support/man/)）中搜索Configuration Wizard（[µVision User's Guide (arm.com)](https://developer.arm.com/documentation/101407/0541/Utilities/Configuration-Wizard?lang=en)）

网页中的文档描述得非常详细，也提供了一个配置模板，我们只需要照着文档描述写好相应的格式即可生成一个图形化的配置界面。

> [!note]
>
> **注意：Keil并没有那么智能，写完这个模板它并不能立马自动匹配到，需要重新关掉Keil工程再开才能加载成图形配置模板，然后才能够实现即时修改即时响应！！！**  

将h文件或者c文件转换为可视化界面是很简单的，只需要在文件的开头和结尾加上以下的标识符即可：

```c
// <<< Use Configuration Wizard in Context Menu >>>


// <<< end of configuration section >>>
```

然后关掉工程再重新打开，就可以实现可视化界面了

# 语法

## 注释

使用标签`i`来进行注释

```c
// <i>这里加上注释
```



## 组

使用标签`h`来控制组

```c
// <h> 组名

// </h>
```

## 复选框

### 复选框组

使用标签`e`来控制复选框组

```c
// <e>系统信息

// <q> 负载锁定
#define _EPCM_LOAD_LOCK      1      //BIT15;     0：不使能负载锁定;    1：使能负载锁定
// </e>
```



### 复选框

复选框`(<c> </c>` or `<!c> </c>`：创建一个复选框来取消注释或注释代码。

1. `<c> </c>`勾选不注释代码，不勾选注释代码。
2. `<!c> </c>`勾选注释代码，不勾选不注释代码。

```c
// <c>复选框1
	需要开关的代码段
// </c>

// <!c>复选框2
    需要开关的代码段
// </c>
```



## 标志位模式

标志位模式`<q>`：可以通过复选框设置的位值选项。使用符号`.`来控制位数

```c
// <q.0> E2ucID第0位
// <q.1> E2ucID第1位
#define E2ucID 0x00
```

上述代码中`E2ucID`初始值为`0x00`，当我选中`q.0`的复选框时，`E2ucID`的第一位就会变为1，变为`0x01`，当我勾选`q.1`的时候，`E2ucID`的值的第二位就会变为1，也就是`0x02`。以此类推。

## 组合框

组合框`<o>`用于选项与选择或数字的输入。

```c
// <o>电芯串数
// <7=> 7串
// <6=> 6串
// <5=> 5串
// <4=> 4串
#define EPCM_CELL_NUM 7
```

这里会展示下拉框，其中选项有`4串`,`5串`,`6串`,`7串`，当选中其中一个选项之后，`EPCM_CELL_NUM`的值就会变为相应（尖括号中的）的值。



## 字符串输入框

使用标签`s`来控制字符串输入：可修改包含ASCII字符串条目。

```c
// <s.12>制造商名称
#define   _E2ucMNFName           "tenxun"//U8  xdata E2ucMNFName[12]
```

其中的`.12`表示字符串的长度是12，初始值是tenxun，当在value中修改值的时候，宏`_E2ucMNFName`的值就会变为你修改的内容













































