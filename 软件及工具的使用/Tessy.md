# 如何评判

1. **代码覆盖率**：
   - **语句覆盖率（Statement Coverage）**：检查是否每个代码语句都被执行过。
   - **分支覆盖率（Branch Coverage）**：测试代码中每个条件的所有分支是否都被执行。
   - **路径覆盖率（Path Coverage）**：所有可能的执行路径是否都被测试。

2. **边界值分析**：
   - 测试输入参数和输出值的边界，确保程序在极端条件下（如最大值、最小值、空值）能够正常运行。

3. **缺陷率（Defect Rate）**：
   - 代码中的缺陷数量与代码总行数或模块的比例。通过动态测试发现的缺陷越少，代码质量通常越高。

4. **执行时间与性能（Execution Time and Performance）**：
   - 测试代码在不同输入情况下的运行时间，确保其满足性能要求，特别是在嵌入式系统中。

5. **内存使用率（Memory Usage）**：
   - 测试代码的内存使用情况，确保在不同输入条件下不会发生内存溢出或过度使用内存。

6. **代码复杂度（Cyclomatic Complexity）**：
   - 动态测试可以帮助你识别高复杂度的代码部分，复杂度越高的代码更容易出错，因此测试应重点关注这些区域。

7. **错误检测与修复率（Defect Detection and Fix Rate）**：
   - 通过动态测试发现并修复的错误数量与整体代码中发现错误的比率，修复率越高，代码质量越好。

8. **功能正确性（Functional Correctness）**：
   - 确保代码的功能行为与预期一致，动态测试会运行各种输入并验证输出结果是否正确。

# 术语表

| **缩写** | **全称**           | **具体含义**                                                 |
| -------- | ------------------ | ------------------------------------------------------------ |
| TO       | Test Object        | 被测试的代码单元，通常是函数或模块，TESSY 对其进行测试。     |
| TC       | Test Case          | 测试用例，指在特定条件下对某个测试对象执行的具体测试，包含输入、预期输出和结果判定。 |
| TF       | Test Frame         | 测试执行环境，包含测试所需的输入数据、预期结果及其他资源和配置。 |
| TS       | Test Step          | 测试过程中执行的小步骤或操作，用于分步完成复杂的测试任务。   |
| TR       | Test Report        | 测试结果报告，包含执行情况、测试结果和代码覆盖率等详细信息。 |
| TD       | Test Data          | 输入到测试对象中的数据集，用于验证功能是否符合预期。         |
| TSP      | Test Specification | 测试说明文件，描述测试对象、测试条件、输入输出数据和预期结果等。 |
| Stub     | Stub               | 用于替代未实现或不适合在测试中执行的功能，模拟系统中某部分的行为。 |
| Driver   | Driver             | 测试驱动程序，调用测试对象并提供必要的输入，常用于嵌入式系统测试。 |
| Cov      | Test Coverage      | 测试覆盖率，指测试执行过程中涉及的代码行数或分支数。常见指标有语句覆盖率和分支覆盖率。 |
| TH       | Test Harness       | 测试框架或工具，用于自动化测试，包括驱动、桩和测试数据。     |
| RT       | Regression Test    | 回归测试，针对修改后的代码进行，确保改动未引入新错误。       |
| CA       | Code Access        | 代码通过                                                     |
| CR       | Coverage Review    | 覆盖评审                                                     |
| CV       | Coverage Viewer    | 覆盖视图                                                     |
| CC       | Code Complexity    | 代码复杂度                                                   |

# 单元测试

**单元测试**（Unit Testing）是软件测试中的一种方法，用来验证代码中的**最小可测试单元**（通常是一个函数、方法或类）的**功能正确性**。每个单元测试通常只测试代码中的一个小部分，确保其在特定的输入下产生预期的输出。

**单元测试的特点**：

1. **独立性**：单元测试应当独立进行，测试代码的每个部分不应该依赖于其他部分，以便能够单独运行和验证。
2. **小范围测试**：单元测试的目标是代码的最小单元，如函数或方法，而不是整个系统或大型模块。
3. **自动化**：单元测试通常是自动化的，使用测试框架（如JUnit、xUnit、pytest等）可以轻松重复执行测试。
4. **早期检测错误**：单元测试有助于在开发的早期阶段发现问题，节省后续集成或系统级别调试的时间。

**单元测试的好处**：
- **提高代码质量**：通过测试代码的每个部分，可以在开发过程中及早发现并修复错误。
- **便于维护**：随着软件的发展，单元测试能够帮助验证新代码的正确性，确保修改或新增的代码不会破坏原有功能。
- **简化调试**：当一个测试失败时，开发人员可以快速定位到具体的单元，减少调试范围。

## 测试步骤

1. **创建项目**
   1. **导入代码**
   2. **选择测试对象**
   3. **生成测试框架**
2. **创建测试用例**
   1. **设置测试环境**
   2. **执行测试**
3. **分析结果**
   1. **记录和保存测试结果**
   2. **重复测试**
## 新建工程

### 新建及导入

新建：点击*==file->New Project==*:

1. Name:给项目命名
2. Project Root：项目路径地址
3. 选择刚刚建立的工程，打开

选择&导入：点击*==file>Select Project==*打开相应的工程

### 导入源码

将待测的工程复制到TESSY创建工程的目录下，使得待测工程文件与TESSY工程处于同一目录  

```
Root/
├── tessy工程
└── 待测项目
    └── test.c
```

1. 当移动待测文件之后，刷新一下

   ![刷新](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_9_38_17_202409190938165.png)

2. 打开创建的 TESSY 工程，在 *`Test Cockpit `*视图下选中需要的测试工程右键并点击 *`New Modules  `*

3. 在 General 界面选择编译器 Environment 选择 *`GNU GCC Eclipse CDT (Default)`*， Test Type 选择 *`Unit Test`*后点击 OK  

4. 在 Test Project 视图下会自动创建测试集合和自动导入代码并进行代码分析  

5. 在 Test Cockpit 视图下会自动分析 CA覆盖情况，将鼠标悬停在需要测试的*`modules`*上

6. 在 CV 页面下的 CR (依次点击*==CV>CR==*)支持处理无法访问的源代码行， 在 CV 页面选中待测的代码，不可达的代码会标记为黄色  

7. 在 CR（Coverage, Review）视图下选中代码不可达的原因进行提交后，不可达代码标记为蓝色且 CA 覆盖率变为 100%，最终提交的原因会在 Summary Report 中显示  

   > 这里相当于是忽略没有覆盖到的代码,然后解释一下为什么没有覆盖
   >
   > 创建依次点击*==CR>印章形状(新建CR)>添加代码不可达的原因==*
   >
   > 需要手动输入不可达的代码行号，并且提交时comment部分必须要写

### 导入宏和头文件

如果待测工程涉及到头文件，需要在 TESSY 中加入头文件的路径信息， TESSY 提供了两种方式进行头文件路径添加  

1. 自动创建测试集合==前==
   1. 导入头文件：在 Test Cockpit 视图下选中待测工程右键点击 New Modules 在 Create Modules 窗口下选中 Includes 后，将头文件路径信息进行添加，点击图标 ，可以指定头文件路径

      ![导入头文件](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_10_28_50_202409191028918.png)
   
   2. 定义宏：在create new modulues窗口下选择*`Defines`*，点击加号即可定义宏
   
      ![定义宏](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_10_42_54_202409191042735.png)
   
2. 自动创建测试集合==后==
   1. 在 Test Project 视图下选中待测的模块，在 Sources 页面点击 Include ，导入头文件或者手动定义宏	
   1. ![宏定义及头文件的导入](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_10_51_17_202409191051501.png)

### 修改相关参数

当一切准备就绪之后，可以点击Tessy解析出来的函数，点击函数就可以看见该函数的相关配置，步骤为：

1. 选中被测函数
2. 点击 TIE 界面，可以看到 TESSY 自动识别出来的输入和输出接口， 如果需要进行改动，可在想要修改的位置修改测试对象接口  

![image-20240919110047817](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_11_0_47_202409191100958.png)

## 设计测试用例

### 新建测试用例

测试步骤如下：

1. 点击TDE页面
2. 选择需要测试的单元项目
3. 选择测试test item页面
4. 新建一个测试用例
5. 填入测试用例的相关参数
6. 保存

![新建一个测试用例](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_11_19_32_202409191119529.png)

点击运行，查看用例的通过情况

![image-20240919112400605](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_11_24_0_202409191124747.png)

### 设置覆盖度



点击测试执行绿色按钮右边的下拉小三角 ，选择 Edit Test Execution Settings . . . ，可以对覆盖度进行设置，保存后执行  

![设置覆盖度](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_11_46_31_202409191146408.png)

1. 首先需要选择是否需要覆盖：

   1. None：表示不计算任何覆盖度

   1. Test object only ：表示只计算当前被测函数的覆盖度

   1. Test object and called functions： 表示计算被测函数和调用函数的覆盖度

![是否需要覆盖率](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_11_46_16_202409191146646.png)

2. 单元测试时，选Test object only，覆盖度选C0， C1， MC/DC（注意，自定义选择时需要将Use preselected coverage选项关掉）
3. 集成测试时，选Test object and called functions，覆盖度选CPC和FC

当配置好单元测试之后，就可以在Test project选项卡中看到代码的通过情况了

![代码通过情况](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_11_55_35_202409191155505.png)

> [!IMPORTANT]
>
> 测试的**覆盖度**（Coverage）是指在测试过程中，代码被执行和验证的程度。覆盖度衡量了测试用例对代码的覆盖范围，反映了代码被测试的全面性。以下是一些常见的覆盖度类型：
>
> 1. **语句覆盖（Statement Coverage）**
>    - 也叫**行覆盖**。衡量测试用例是否执行了程序中的每一条语句，确保每个代码行至少被执行一次。
>    - 示例：在 `if` 语句中，即使条件不成立导致某些分支未执行，语句覆盖也可能被视为已满足。
>
> 2. **分支覆盖（Branch Coverage）**
>    - 衡量测试用例是否执行了程序中的每一个分支路径。每个条件判断的**真**和**假**路径都必须被执行。
>    - 示例：对于 `if (x > 0)`，必须测试 `x > 0` 为真和 `x > 0` 为假的两种情况。
>
> 3. **条件覆盖（Condition Coverage）**
>    - 每个布尔条件都必须在测试中至少有一次为真和一次为假。它比分支覆盖更详细。
>    - 示例：`if (A && B)`，需要测试 A 和 B 各自为真和假的情况。
>
> 4. **路径覆盖（Path Coverage）**
>    - 测试用例应执行代码中所有可能的独立路径。路径覆盖比分支覆盖更复杂，尤其是当代码中存在多重嵌套和循环时，路径数量会快速增加。
>
> 5. **函数覆盖（Function Coverage）**
>    - 确保程序中每个函数都被至少调用一次。
>
> 6. **MC/DC（Modified Condition/Decision Coverage，修正条件/判定覆盖）**
>    - 一种严格的覆盖度标准，确保每个条件可以**独立**影响判定结果。它需要测试每个条件在不影响其他条件的情况下独立为真或假。
>    - 这是航空、汽车等高安全性领域常用的覆盖度标准。
>

点击 CV 界面可查看覆盖度情况， 在 Statement(C0) Coverage 视图下能看到测试的 C0 覆盖情况  

![CV覆盖率的图形化界面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_12_0_18_202409191200100.png)

可以通过切换不同的视图，查看不同测试图形化

![分支测试的图形化显示](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_13_39_51_202409191339969.png)

> [!TIP]
>
> 在切换视图的时候可以看到测试用例分支的覆盖情况
>
> 双击图中的分支在下方的源代码视图中会显示相关分支所对应的代码

### 插入需求

打开 *`Requirement Management `*界面，右键选择*` Import `* 

![插入需求](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_14_23_0_202409191423618.png)

>   [!NOTE]
>
> 导入的文件可以为csv、tsv、txt、vxv、xml、reqif格式
>
> 使用txt格式时，Tessy会认为一行文字为一条需求

提交：提交时可以选中一个提交，也可以提交全部

![提交需求](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_14_25_40_202409191425923.png)

将需求和测试用例链接：依次点击*==TDE>Requirements coverage==*

![image-20240919144518358](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_14_45_18_202409191445539.png)

> [!NOTE]
>
> 这里使用可视化页面链接测试用例和需求，通过在需求项下勾选测试用例，从而达到链接测试用例的功能

测试用例与需求链接完成后，在 TDE 界面的TEST item页面中选中对应的测试用例，在 Test Definition 标签下可以查看链接到该测试用例的需求  

![查看链接](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_14_50_33_202409191450038.png)

## 导出测试报告

点击 下拉小三角可选择导出测试报告， 点击 Edit Test Details Report Settings…可设置报告选项和报告存放路径， 击 Generate Test Detail Report 可按默认设置生成详细测试报告  

![导出测试报告](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_13_56_45_202409191356679.png)

可以生成以下几种报告：

1. Test Detail Report，详细测试报告  
2. Test Overview Report，概要测试报告  
3. Planning Coverage Report ：计划需求覆盖报告（需要需求文件）
4. Execution Coverage Report：需求执行覆盖率报告（需要需求文件）

# 组件测试

## 创建工程

新建工程和单元测试相同，导入源码与单元测试部分基本相同，不同之处在于新建测试集合的时候 Test Type 选择 Component Test

![组件测试新建工程页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_15_26_47_202409191526690.png)

配置好编译器、头文件、宏定义等其他选项后， 点击 OK，此时 Test Project视图会自动创建测试集合并分析代码。 与单元测试不同的是，组件测试分析完成后不会列出所有源码文件中的函数列表，而是只显示 Scenarios 一个测试对象  

![单元测试项详情](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_15_33_30_202409191533972.png)

可根据被测对象功能以及对外接口，在 TIE 界面中进行接口设置  

对于一些依赖或者外部函数，可以使用*`桩函数`*进行模拟（这里的桩函数是LightOff和LightOn），选择需要打桩的函数，右键，点击*`create stub`*

![桩函数](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_15_49_23_202409191549491.png)

> [!IMPORTANT]
>
> 在 TESSY 中，**桩函数**（Stub Function）是一种用于替代实际函数的测试工具，通常在被测代码（单元）依赖于其他模块或函数，但这些模块或函数尚未实现、不可用或不便在测试中使用时，使用桩函数来模拟其行为。桩函数可以控制依赖的输入和输出，以便对单元进行隔离测试，确保测试环境的独立性和稳定性。
>
> 桩函数的作用：
>
> 1. **隔离测试单元**：
>    - 在单元测试中，测试的目的是针对某个具体的函数或模块进行验证。如果该函数依赖于其他未实现的模块或外部库，使用桩函数可以避免实际依赖的复杂性，确保测试的集中性和独立性。
>
> 2. **模拟外部依赖**：
>    - 桩函数可以模拟外部依赖（如硬件接口、数据库访问、网络调用等），并且可以根据测试用例设置返回值，确保不同场景下被测单元的行为能够被充分验证。
>
> 3. **控制测试环境**：
>    - 桩函数提供了对外部依赖行为的完全控制，这样可以模拟异常情况、边界条件或极端场景（例如硬件故障、超时等），而不必实际依赖外部系统。
>
> 4. **提高测试效率**：
>    - 使用桩函数可以加速测试执行，因为它避免了对外部资源（如硬件设备或网络）的依赖，从而使测试更加快速和稳定。
>
> TESSY 中的桩函数实现步骤：
>
> 1. **识别依赖函数**：
>    - 首先确定在被测试的模块中哪些函数是依赖外部模块的，这些函数是需要替换为桩函数的目标。
>
> 2. **自动生成桩函数**：
>    - TESSY 支持自动生成桩函数的基本框架。被测单元中的依赖函数可以由 TESSY 自动创建对应的桩函数。
>
> 3. **配置桩函数行为**：
>    - 在 TESSY 中，可以手动配置桩函数的行为。例如，你可以为桩函数设定不同的返回值，或模拟异常返回，以便测试被测单元在各种情况下的响应。
>    
> 4. **执行测试**：
>    - 使用配置好的桩函数执行测试，确保被测单元在隔离的环境中正常运行。
>
> 5. **分析测试结果**：
>    - 测试完成后，TESSY 会提供详细的报告，包括桩函数调用的次数、输入参数、返回值等信息，帮助开发人员分析测试结果。
>
> 使用桩函数的典型场景：
>
> - **硬件接口的模拟**：当软件依赖硬件设备时，无法直接在测试中访问这些设备，可以使用桩函数模拟硬件行为。
> - **尚未实现的功能**：如果依赖的某些模块尚未开发完成，桩函数可以临时替代这些模块，确保其他部分的代码可以正常测试。
> - **外部库或服务调用**：对于依赖于外部服务或库的代码，桩函数可以模拟服务的行为，避免实际调用外部系统。
>

## 导入源码

```c
typedef enum
{
    closed, open
} co_states;

typedef enum
{
    off, on
} oo_states;

co_states sensor_door, state_door;
oo_states sensor_ignition, state_light;

extern void LightOn(void);
extern void LightOff(void);

void init(void)
{
    sensor_door = open;
    sensor_ignition = off;
    state_door = open;
    state_light = off;
}

void set_sensor_ignition(oo_states i)
{
    sensor_ignition = i;
}

void set_sensor_door(co_states d)
{
    sensor_door = d;
}

static void iLightOn(void)
{
    if (on == state_light)
        return;
    else
    {
        state_light = on;
        LightOn();
    }
}

static void iLightOff(void)
{
    if (off == state_light)
        return;
    else
    {
        state_light = off;
        LightOff();
    }
}

void tick()
{
    static int Timer = 0;

    if ((state_door == open) && (sensor_door == closed))
    { //Door was closed ---> start timer!
        Timer = 500;   // 5 seconds = 500 cycles of 10 ms
        iLightOn();
    }
    else if (on == sensor_ignition)
    {
        iLightOff();
    }

    if (Timer > 0)
        Timer--;

    if (0 == Timer)
        iLightOff();

    // Store current value for next tick
    state_door = sensor_door;
}

```



## 设计测试用例

针对本示例工程设计如下测试用例：当关闭车门时(set_sensor_door = closed)，内部照明灯点亮； 当有打火动作时(set_sensor_ignition = on)，照明灯立即熄灭。  

设计步骤如下：

1. 设置入口函数。 进入 SCE 界面，选中组件函数 tick()，点击 ，设置为 work task（入口函数选取条件：==void 类型且没有形参==）  

   ![设置入口函数](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_15_54_57_202409191554977.png)

   同时需要设置该函数的开始时间及循环周期

   ![入口函数参数设置](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/27_16_18_10_202409271618981.png)

2. 在 SCE 界面下的选中 Test Data 后鼠标放在在 Test Items 空白处点击创建测试用例  

   ![新建测试用例页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_15_58_11_202409191558077.png)

   ![测试用例详情](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_16_3_1_202409191603714.png)

3. 首先选择刚刚新建的测试用例，点击 Insert Time Step 添加时间节点  

   ![image-20240919160852514](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_16_8_52_202409191608649.png)

4. 按照需要将组件函数拖到指定时间节点（鼠标左键按住函数可拖动到时间节点相应位置），搭建场景。 具体场景为： 在 30ms 时关闭车门，则 40ms 时车灯会亮； 在 70ms 时将发动机点火，则 80ms 时车灯会灭   

5. 选中 set_sensor_door()函数，在 Properties 界面设置其参数值 d=closed。set_sensor_ignition()函数同理，设置 i=on 

   ![修改函数参数](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/19_16_26_44_202409191626320.png)

6. 将function call页面中的 LightOn()、 LightOff()和静态函数 iLgihtOn()、 iLightOff()拖到指定时间节点，用于监控车灯变化情况。 设置外部函数和静态函数预估调用时间（在 10ms内预计调用一次）。选中 LightOn()函数， 在 Properties 界面设置 Time Frame 为 10。iLightOn()、 LightOff()、 iLightOff()同理 

   ![函数调用](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_9_45_49_202409200945562.png)

   > [!NOTE]
   >
   > 注意：这里的两个静态函数iLightOn()和iLightOff()函数分别调用了两个外部函数（桩函数）
   >
   > 从function call中拖拽函数是用来监控该函数是否被调用及调用次数和从component function中拖拽是不一样的

7. 编写测试用例

   ![测试用例](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_9_50_3_202409200950799.png)

8. 设置测试需要的参数

   ![测试参数](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_8_41_1_202409200841376.png)

9. 点击运行按钮，查看测试运行的结果

   ![运行结果](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_9_53_40_202409200953551.png)

10. 点击 CV 界面的 Called Functions 视图和 Call Pair Coverage 视图下可查看函数覆盖和调用覆盖的情况  

![测试用例覆盖情况](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_9_53_31_202409200953285.png)

# 桩函数

## 普通桩

以下四种情况通常可以打普通桩：

1) 函数没有返回值、 没有形参
2) 函数不影响后续实现以及变量
3) 函数本身有返回值但无需使用
4) 函数有形参，  

> 栗子：
>
> STUB_1()在当前.c 文件中没有定义， 如果直接执行测试用例会报未定义错误 。
>
> ```c
> typedef unsigned char hirain_u8;
> extern void STUB 1();
> extern hirain u8 STUB 2();
> void test_fun_stub_1(void){
>     STUB_1();
> }
> ```
>
> 只需要在 TIE 界面将其打普通桩， 并点击保存， 解决报错  
>
> ![函数打桩](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_9_57_6_202409200957979.png)

## 高级桩

以下两种情况建议打高级桩：

1) 使用到桩函数的返回值

2) 如果函数有形参，并且需要接口传参检测

> 栗子1：
>
> 被测函数中用到了 STUB_2()函数的返回值，如果对 STUB_2()函数打普通桩会报无返回值的错误  
>
> ```c
> typedef unsigned char hirain_u8;
> extern void STUB_1();
> extern hirain u8 STUB_2();
> hirain_u8 test_fun_stub_2(void){
>     hirain_u8 temp=STUB_2();
>     return temp;
> }
> ```
>
> 在 TIE 界面将其打高级桩并保存，并在 TDE 界面设置其预期返回值  
>
> ![高级桩](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_10_4_1_202409201004600.png)
>
> > [!NOTE]
> >
> > 注意：这里设置的IN是相较于被测函数来说是IN，如果传参数的话，那么passing应该设置为out，因为对于被测函数来说，形参是输出
>
> ![设置期待返回值](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_10_12_32_202409201012622.png)
>
> 栗子2：
>
> 对 STUB_3()函数进行传参检测，需要给该函数打高级桩，第一个接口为返回值，其余为形参接口  
>
> ```c
> typedef unsigned char hirain_u8;
> extern void STUB_1();
> extern hirain_u8 STUB_2();
> static hirain_u8 STUB_3(hirain_u8 temp)
> {
>     return temp*temp;
> }
> 
> hirain_u8 test_fun_stub_4(hirain_u8 temp)
> {
>     hirain_u8 tem STUB_3(temp);
>     return tem;
> }
> ```
>
> ![带有形参的桩函数](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_10_12_49_202409201012162.png)

## 手写桩

如果需要让桩函数有额外的功能，如： 传参检测、局部数据处理、多传参检测、函数实现变更等，可以进行手写桩。  

> 栗子：
>
> ```c
> typedef unsigned char hirain_u8;
> extern hirain_u8 STUB_3(hirain_u8 temp);
> hirain_u8 loop_test3(hirain_u8 temp){
>     for(hirain_u8 i=0;i<temp;i++)
>     {
>         STUB_3(i);
>     }
>     return 1;
> }
> ```
>
> 想要监控对 STUB_3()函数每一次参数传递是否正确，可以对其打普通桩，通过手写桩函数实现传参检测。步骤如下：  
>
> 1. 在test cockpit页面选中待测函数，然后在 TIE 界面点击新建变量，输入 name 选择类别为普通变量， 新建一个数组并在Type 设置为对应的变量类型，示例为 unsigned char 类型 
>
>    ![新建变量](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_10_20_23_202409201020925.png)
>
> 2. 对 STUB_3()打普通桩并保存
>
>    ![打桩](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_11_18_12_202409201118004.png)
>
> 3. 设置创建的全局变量数组为 OUT
>
>    ![设置数组为全局变量](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_11_38_57_202409201138360.png)
>
>    > [!NOTE]
>    >
>    > 注意：当修改被测函数之后，工程的接口就需要变更，在TDE页面就会显示接口变更
>    >
>    > ![接口变更](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_13_49_25_202409201349144.png)
>    >
>    > 然后切换到IDA页面，双击变更的函数，将会展示新旧之间的对比（Tessy只列出相关属性，需要测试工程师自己匹配是否正确）
>    >
>    > ​	![提交页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_13_53_10_202409201353738.png)
>
> 4. 然后在TDE 的 Stub Functions 界面手写桩函数的代码  
>
>    ![手写桩函数代码的编写](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_13_56_48_202409201356456.png)

> [!NOTE]
>
> 注意：这里的桩函数填写时，需要先在右边的usercode Outline页面，选中该桩函数的作用域，可以不同的用例使用不同的桩函数，写在顶层会导致所有的测试用例都使用该桩函数

4. TDE 界面选中新建的全局变量数组右键选择 Show All Array Elements 显示所有数组元素，然后对 new_array 数组输入预期值，检查测试结果  

   ![填写数组期待值](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_14_57_7_202409201457306.png)

5. 点击运行即可查看相关测试结论

# 指针相关测试

> [!IMPORTANT]
>
> 带有指针的测试不可以直接给指针赋值，而需要使指针指向数组或变量，给数组和变量赋值才可以使用

## 指针

与指针有关的测试要点在于构建合适的对象，将地址传入指针接口。  

```c
typedef unsigned char hirain_u8;
typedef int hirain_32;
typedef struct{
    hirain_32 str1;
    hirain_32 str2;
}STR;
hirain_u8 a;
void TS_FUN_PON_2(hirain_u8 *p,STR *p1)
{
    //p++s
    a=*p;
    p1->str2 = p1->str1;
}
```

1. 在 TIE 界面设置输入输出接口，根据不同的测试需求进行设置，示例的输入输出接口如下： 

   ![输入输出接口](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_15_22_56_202409201522961.png)

2. 在 TDE 界面创建测试用例后， 设置指针的具体指向，赋值后执行测试  

   ![image-20240920153238924](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_15_32_39_202409201532104.png)
   
   > [!NOTE]
   >
   > 注意：这里需要选中测试用例之后，才能右键创建指针指向的目标值

![完善测试用例](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_15_38_25_202409201538136.png)

当在测试过程中，使用了字符串的指针，需要先设置一个目标数组，然后将需要测试的指针指向这个数组

![目标数组](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/25_10_33_34_202409251033052.png)

> [!NOTE]
>
> 需要注意的是，在字符指针所要指向的数组中不可以直接填写字符，需要填写字符所要对应的ASSIC码，并且字符串的结尾需要以`\0`结尾

## 函数指针

与函数指针有关的测试要点在于构建与函数指针类型相同的函数对象，并将地址传入指针接口

```c
typedef int hirain_32;
typedef unsigned int hirain_u32;
hirain_u32 (*T_pon)(hirain_u32 i);
hirain_u32 hirain_stuv_4(hirain_u32 temp2){
    int temp=T_pon(temp2);
    return temp;
}
```

1. 在 TIE 界面设置函数指针的输入输出接口  

   ![设置函数指针的输入输出接口](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_15_46_18_202409201546123.png)

2. 在 TDE 的 Declarations/Definition 界面实现函数的声明和定义（注意要与函数指针类型一致），并让函数指针指向这个函数，赋值后执行测试

   ![函数的申明与定义](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_15_53_47_202409201553725.png)  

![image-20240920155553108](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_15_56_26_202409201556964.png)

## void型指针

```c
typedef unsigned char hirain_u8;
hirain_u8 a;
hirain_u8 TS_FUN_PON_1(int *p,void *p1){
    hirain_u8 temp =0;
    int *temp1 = p1;
    a=*p;
    return *temp1;
}
```

1. 需要在 TIE 界面新建一个有类型的全局变量并根据不同的测试需求设置输入或输出，然后保存，在TDE页面 右键void类型指针，选择 set pointer target，当鼠标变成带有瞄准框的样式，选中你想要指向的变量

   ![image-20240920160506507](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_16_5_28_202409201605361.png)

2. 然后在 TDE 界面将指针指向该变量，将变量地址传入指针接口，设置测试数据并运行  

![定义指针指向](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_16_9_16_202409201609191.png)

# 故障注入

在单元测试过程中可能会遇到死循环、 先写后读等特殊情况下的代码， 无法正常实施测试或无法达到相应的触发条件，满足不了覆盖要求。 对于这种类型函数的测试，可以利用故障注入的方式进行测试（这里以死循环为例）  

```c
typedef unsigned char hirain_u8;
hirain_u8 loop_test2(){
    while(1)
    {
    }
}
```

1. 对于上述示例待测函数，若直接在 TDE 界面运行测试用例， TESSY 会一直处于执行测试过程  

   ![死循环](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_16_14_8_202409201614768.png)

2. 此时需要建立故障注入的测试用例来进行测试，首先在 TDE 界面的 Test Items 视图下鼠标右键选中 New Fault Injection Test Case 建立故障注入的测试用例  

   ![故障注入](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_16_20_24_202409201620676.png)

3. 然后跳转至 CV 界面，在测试函数的框图处选择死循环的部分，然后鼠标双击会弹出故障注入的窗口，输入对应的故障注入代码并保存  

   ![代码注入](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/20_16_46_20_202409201646521.png)

4. 最后跳转至 TDE 界面根据不同的测试需求设计测试用例，执行测试用例  







































































