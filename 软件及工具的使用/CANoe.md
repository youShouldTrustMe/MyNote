# 软件简介

CANoe 是一款由 Vector Informatik 提供的专业软件工具，用于开发、测试和分析基于 CAN（Controller Area Network）总线和其他汽车网络协议（如 LIN、FlexRay、Ethernet、MOST）的嵌入式系统。CANoe 是一款功能强大且广泛应用于汽车电子系统开发和测试的工具。

## CANoe 的主要功能

1. **总线仿真和分析**：
   - 支持 CAN、LIN、FlexRay、MOST、Ethernet 等多种总线协议。
   - 提供实时总线仿真和消息分析功能，可以捕获和记录总线数据。
2. **节点仿真**：
   - 可以仿真汽车电子系统中的多个节点，支持仿真真实硬件环境下的行为。
   - 支持通过 CAPL（CAN Access Programming Language）脚本编写复杂的仿真逻辑。
3. **网络管理**：
   - 提供网络管理功能，支持 CAN、LIN 和 FlexRay 等网络协议的管理。
4. **诊断功能**：
   - 支持 UDS（Unified Diagnostic Services）等诊断协议，能够执行故障诊断和错误处理。
5. **集成开发环境**：
   - 提供图形化的用户界面和脚本编写环境，方便用户进行配置和测试。
## 使用场景

- **汽车电子系统开发**：CANoe 常用于汽车电子控制单元（ECU）的开发和测试，可以仿真和测试不同的汽车总线网络。
- **功能验证和测试**：在汽车开发过程中，用于功能验证、系统集成测试和回归测试。
- **故障诊断和分析**：用于捕获和分析总线数据，帮助工程师进行故障诊断和问题定位。

## 主要组件

- **Measurement Setup**：配置和管理测试环境，包括总线配置、节点配置和信号监控。
- **CAPL Browser**：用于编写和调试 CAPL 脚本，实现复杂的仿真和测试逻辑。
- **Trace Window**：实时显示和记录总线上的消息和事件，方便进行分析和调试。
- **Panel Designer**：设计用户界面面板，用于控制和监控仿真环境中的节点和信号。

# CANoe在ECU项目开发中的作用

## 第一阶段：全仿真网络系统

**也就是所有的节点都是虚拟的**

![image-20240705110725435](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_11_7_25_202407051107514.png)

## 第二阶段：真实节点和部分仿真节点共存

可以使用部分真实的ECU代替虚拟节点，形成半虚拟半真实的网络

![image-20240705114557104](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_11_45_57_202407051145212.png)

## 第三阶段：全真实节点的网络系统

使用真实的节点代替所有的仿真节点，形成全真实的节点

![image-20240705114705240](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/5_11_47_5_202407051147323.png)

# CANoe软件界面功能介绍

CANoe主页面

![image-20240704145144606](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_14_51_44_202407041451779.png)
CANoe主界面主要有以下功能区部分。

- File（文件）
- Home（主功能区）
- Analysis（分析）
- Simulation（仿真）
- Test（测试）
- Diagnostics（诊断）
- Environment（环境）
- Hardware（硬件）
- Tools（工具）
- Layout（布局）

## 文件菜单

File菜单主要用于工程文件的相关操作及属性设定(CANoe项目为cfg结尾).

![image-20240704145726442](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_14_57_26_202407041457590.png)

## Home功能区

![image-20240704145935059](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_14_59_35_202407041459134.png)

![image-20240704150033217](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_0_33_202407041500352.png)

![image-20240704150053176](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_0_53_202407041500258.png)

## Analysis功能区

![image-20240704150705078](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_7_5_202407041507149.png)

![image-20240704150741920](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_7_42_202407041507030.png)

![image-20240704150809762](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_8_9_202407041508823.png)

## Simulation功能区

![image-20240704152233634](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_22_33_202407041522724.png)

![image-20240704152321156](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_23_21_202407041523244.png)

## Test功能区

![image-20240704152601509](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_26_1_202407041526577.png)

![image-20240704152619163](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_26_19_202407041526244.png)

## Diagnostics功能区

![image-20240704152705627](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_27_5_202407041527712.png)

![image-20240704152722114](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_27_22_202407041527194.png)

## Environment功能区

![image-20240704152807056](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_28_7_202407041528142.png)

![image-20240704152819532](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_28_19_202407041528626.png)

## Hardware功能区

![image-20240704152846427](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_28_46_202407041528521.png)

![image-20240704152904056](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_29_4_202407041529128.png)

![image-20240704152922417](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_29_22_202407041529504.png)

## Tools功能区

![image-20240704153003389](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_30_3_202407041530459.png)

![image-20240704153037554](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_30_37_202407041530615.png)

## Layout功能区

![image-20240704153104354](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_31_4_202407041531425.png)

![image-20240704153115471](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_31_15_202407041531523.png)

# 第一个CANoe仿真工程

## CANoe仿真环境基本架构

![image-20240704154020245](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_40_20_202407041540387.png)

## 需求

- 创建两个节点（Switch模块和Light模块）
- 创建两个控制面板（开关面板和指示灯面板）
- 通过CAPL代码实现两个节点间的通信

当用户操作开关以后，节点Switch将这个动作通过CAN通知给节点Light；节点Light收到这个CAN报文后，根据信号的值将指示灯点亮或熄灭。  

![image-20240704154812020](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_48_12_202407041548107.png)

## 新建项目

打开CANoe主界面，单击File→New可以看到CANoe提供的工程模板.这里双击选择模板CAN 500kBaud 1ch，将生成一个空白的支持单通道的CAN总线仿真工程。根据之前介绍的工程文件夹的命名习惯，在文件夹FirstDemo下面分别创建文件夹CANdb、Panels和Nodes。  

![image-20240704155218665](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_52_18_202407041552775.png)

## 添加CAN数据库

### CANdb的主要功能和用途

1. **定义消息和信号**：
    - CANdb文件详细定义了网络中每个消息的格式，包括消息ID、数据长度、周期时间等。
    - 还定义了每条消息包含的信号，以及这些信号的属性，如起始位、长度、缩放因子和偏移量。

2. **提供信号解释和转换**：
    - CANdb文件将原始的CAN数据帧转换为有意义的信号值（如速度、温度等），便于工程师理解和使用。
    - 反过来也可以将信号值转换为CAN数据帧，方便数据发送。

3. **支持网络仿真和分析**：
    - 在CANoe中使用CANdb文件可以仿真和分析CAN网络通信。
    - CANoe通过CANdb文件识别并解析收到的CAN消息，显示信号的实际值。

4. **自动化测试和诊断**：
    - 自动化测试脚本可以使用CANdb文件中的定义来生成和验证CAN消息。
    - CANdb文件支持故障注入和诊断测试，帮助发现和解决通信问题。

5. **生成报表和文档**：
    - CANdb文件中的信息可以用于生成通信协议和信号定义的文档。
    - 有助于在开发团队和供应商之间共享和维护网络通信规范。

### 使用CANdb文件的具体步骤

1. **创建和编辑CANdb文件**：
    - 使用Vector的CANdb++ Editor或其他支持DBC格式的工具创建和编辑CANdb文件。
    - 定义网络中的节点、消息和信号，并设置相应的属性。

2. **加载CANdb文件到CANoe**：
    - 在CANoe项目中加载创建好的CANdb文件。
    - CANoe会解析该文件，并在网络仿真和分析中使用这些定义。

3. **仿真和测试**：
    - 使用CANoe的仿真功能，根据CANdb文件定义的消息和信号进行网络仿真。
    - 自动化测试脚本可以引用CANdb文件中的信号定义，生成测试用例和检查结果。

4. **监控和分析**：
    - 在CANoe的信号监控窗口中查看和分析通过CAN网络传输的信号。
    - 使用CANdb文件中的信息，将原始CAN数据帧转换为易于理解的信号值。

### 示例

假设你有一个CANdb文件，定义了以下信息：

- **节点**：ECU1, ECU2
- **消息**：Message1 (ID: 0x100, 周期时间: 100ms)
- **信号**：Signal1 (起始位: 0, 长度: 16位, 缩放因子: 0.1, 偏移量: 0)

在CANoe中，你可以：

1. 加载这个CANdb文件。
2. 配置仿真环境，使得ECU1和ECU2按照CANdb文件定义的规则通信。
3. 使用监控窗口查看Signal1的实际值，并根据这个信号进行进一步分析或故障排除。

### 新建CAN数据库

现在创建一个含有报文Msg1和信号bsSwitch的数据库。

1. 单击Tools功能区打开CANdb＋＋Editor（CAN数据库编辑器）。

2. 在CANdb＋＋Editor界面中单击File→Create database并选择CAN Template.dbc作为模板。

3. 将新建文件命名为FirstDemo.dbc并保存在工程FirstDemo下面的文件夹CANdb中。

### 添加报文和信号(在CANdb++Editor页面)

在Messages下面创建一条报文Msg1  

![image-20240704155621076](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_15_56_21_202407041556160.png)

在Signals下面创建一个信号bsSwitch，单击OK按钮保存。  

![image-20240704160138611](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_16_1_38_202407041601693.png)

可 以 将 信 号 bsSwitch 拖 曳 到 报 文 Msg1 下 面 ， 这 样bsSwitch就变成报文Msg1的一条信号  

![image-20240704161805165](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_16_18_5_202407041618254.png)

### 添加数据库到工程中

进 入 Simulation Setup 窗 口 ， 在 System View 视 图 中 单 击Networks→CAN Networks→CAN→Databases ， 右 击 鼠 标 选 择Add，如图7.6所示，可以将FirstDemo.dbc文件加入仿真工程.

![image-20240704161937596](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_16_19_37_202407041619682.png)

## 定义系统变量

在CANoe中，系统变量（System Variables）是用于在仿真和测试过程中**存储和共享数据的变量**。它们为用户提供了一种方便的方法来定义和管理仿真环境中的全局状态和参数。系统变量在多种应用场景中非常有用，包括自动化测试、仿真控制和数据监控等。以下是系统变量的主要用途和具体应用：

### 系统变量的主要用途

1. **数据共享和通信**：
    - 系统变量可以在不同的CAPL脚本、测试模块和仿真模型之间共享数据。
    - 提供一种全局数据存储机制，使不同组件可以方便地访问和修改变量的值。

2. **仿真控制**：
    - 系统变量可以用于控制仿真场景的行为，例如启动和停止仿真、触发特定事件或模式等。
    - 可以通过改变系统变量的值来动态调整仿真的运行参数。

3. **自动化测试**：
    - 在自动化测试脚本中使用系统变量来存储测试参数和结果。
    - 系统变量可以用作测试条件和断言的基础，帮助验证系统行为是否符合预期。

4. **监控和记录**：
    - 系统变量可以用来监控仿真过程中重要参数的变化。
    - 可以记录系统变量的值，用于后续分析和报告生成。

### 如何在CANoe中定义和使用系统变量

#### 1. 定义系统变量

在CANoe中，可以通过如下步骤定义系统变量：

1. 打开CANoe项目。
2. 导航到`Variables`窗口（通常可以通过“View”菜单找到）。
3. 右键点击变量窗口，选择`New System Variable`。
4. 为系统变量指定一个名称和类型（如整型、浮点型、布尔型等）。
5. 设置变量的初始值（可选）。

#### 2. 使用系统变量

##### 在CAPL脚本中使用

CAPL（CAN Access Programming Language）是CANoe中用于编写测试脚本的语言。可以在CAPL脚本中读取和写入系统变量。

```c
variables {
    msTimer myTimer;
}

on start {
    // 初始化系统变量
    setValue("SystemVarName", 0);
}

on timer myTimer {
    // 读取系统变量的值
    int value = getValue("SystemVarName");
    write("Current value: %d", value);
    
    // 更新系统变量的值
    setValue("SystemVarName", value + 1);
    
    // 重新启动定时器
    setTimer(myTimer, 1000); // 每秒更新一次
}

on key 'a' {
    // 手动修改系统变量的值
    setValue("SystemVarName", 42);
}
```

##### 在测试模块中使用

在CANoe的测试模块中，也可以使用系统变量来控制测试流程和检查条件。

```plaintext
// 定义一个系统变量
systemvar myTestVar;

// 在测试用例中使用系统变量
testcase myTest {
    // 设置系统变量的值
    myTestVar := 10;

    // 使用系统变量的值作为测试条件
    if myTestVar == 10 {
        teststep "Check myTestVar is 10" {
            // 测试步骤
        }
    }
}
```

### 实际应用示例

假设你有一个CAN网络仿真项目，包含两个ECU。你可以使用系统变量来监控和控制仿真过程。例如，你可以定义一个布尔型系统变量`isEngineRunning`，用于表示发动机是否正在运行。

1. **定义系统变量**：
    ```plaintext
    System Variable: isEngineRunning (Boolean, initial value: false)
    ```

2. **在CAPL脚本中使用**：
    ```c
    on message CAN1.EngineStatus {
        // 更新系统变量
        if (this.EngineRunning) {
            setValue("isEngineRunning", true);
        } else {
            setValue("isEngineRunning", false);
        }
    }
    ```

3. **在测试模块中使用**：
    ```plaintext
    testcase checkEngineStatus {
        // 检查系统变量的值
        teststep "Engine should be running" {
            if isEngineRunning == true {
                write("Test passed: Engine is running.");
            } else {
                write("Test failed: Engine is not running.");
            }
        }
    }
    ```

### 创建系统变量

单击Environment功能区打开System Variables（系统变量）配置对话框。创建一个系统变量svLight.  

![image-20240704163505398](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_16_35_5_202407041635526.png)

按 相 同 的 方 法 创 建 另 一 个 系 统 变 量 svSwitch ， 相 关 设 置 ： Namespace ： MyNamespace；Name ： svSwitch；Data type ： Integer。设置完毕后，可以在列表中看到已设置的两个系统变量.

## 创建仿真面板

CANoe中为了方便用户模拟真实环境中的操作和显示，可以把需要的相关控制或显示控件设计在一个或多个面板中。一般用户很容易上手操作这些面板，并可以查看相关数据的图形化显示。  

首先创建一个开关面板，用于模拟开关操作。

1. 单击Tools功能区的Panel Designer（面板设计器）。
2. 新建一个Panel，命名为SWITCH，并保存在文件夹Panels下。
3. 在Panel Designer窗口的Toolbox（工具箱）中选择添加控件Switch/Indicator。在Properties（属性）对话框里将相关参数设置如下，其他属性保持默认值。
   -  State Count:2。
   -  Mouse Activation Type:LeftRight。
   -  Symbol Filter:Variable。
   -  Symbol:svSwitch。
   -  Namespace:MyNamespace。
4.  在 开 关 左 边 添 加 控 件 Static Text ， 将 Text 属 性 设 置 为Switch。

创建一个指示灯面板
1. 新建另一个Panel，命名为LIGHT，也保存在Panels文件夹下。
2. 在工具栏中选择添加控件LED Control。在“属性”对话框里将相关属性设置如下，其他属性保持默认值。

   - Display Only:True。

   - Symbol Filter:Variable。

   - Symbol:svlight。

   - Namespace:MyNamespace。


3. 在 开 关 左 边 添 加 控 件 Static Text ， 将 Text 属 性 设 置 为Light。

![image-20240704165255605](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_16_52_55_202407041652670.png)

## 创建网络节点

系统变量的变化处理以及报文发送与接收等功能需要由CAPL实现，接下来需要创建节点，并在节点中添加对应的CAPL程序。  

### 添加网络节点

 可 以 在 Simulation Setup 窗 口 中 添 加 两 个 ECU 节 点（ECU1和ECU2），当开关按下后，ECU1发送一个CAN报文，ECU2收到报文后将指示灯点亮。
在CAN1的连线上右击，选择Insert Network Node命令，分别创建两个节点为ECU1和ECU2。

![image-20240704165610147](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_16_56_10_202407041656248.png)

右 击 ECU1 ， 选 择 Configuration 命 令 ， 可 以 打 开 Node Configuration（节点配置）对话框。在Node Configuration对话框中，单击File按钮，为该节点创建一个Switch.can文件，并将Title改为Switch.

![image-20240704170003146](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/4_17_0_3_202407041700253.png)
Node Configuration对话框是最常用的界面之一，下面将相关设定简介如下。

1. Title（名称）：用户可以在该文本框中给Network Node指定名称。需要注意的是，该名称只用于在Simulation Setup窗口中的System View中显示（图标节点名称也会改变）。

2. Network Node（网络节点）：用户可以在该下拉菜单中选择database中已经定义的网络节点。

3. State（状态）：用户可以将该Network Node设置为真实节点或仿真节点。simulated代表仿真节点，off代表真实节点（off只是将该节点Block掉）。

4. Execution（执行）：在Execution区域，用户可以指定CAPL程序在该节点的运行方式。①Standard：指定CAPL或C#等将会在仿真工程中执行。②On hardware interface（CAPL-on-Board）： CAPL程序可以在Vector的硬件接口卡上运行（VN1630/VN3000／ VN7600）。

5. Extended（扩展）：为用户提供了Start Delay和Dift/Jitter选项，前者设置了延迟开始的时间，后者则影响了该节点的定时器。

6. Node specification（节点规范）：用户可以在该处选择File区添加CAPL程序，也可以编辑（Edit）和编译（Compile）所添加的CAPL程序文件。

7. Components（模块）：在该选项卡下，列出了该节点所用的所有模块（这里主要指动态链接库文件*.dll），用户也可以在此处添加自己所需的dll文件。

8. Buses（总线)：在该选项卡下，列出了该节点可用的和已指定的总线。

   使用同样的方法配置ECU2，命名为Light，代码文件设定为Light.can。

### 添加Hello world代码

使用CANoe的CAPL编写一个类似程序。双击节点Switch或者单击该节点的Edit图标，可以进入CAPL Browser（CAPL浏览器）。读者可以看到系统为Switch.can自动创建了如下空白的CAPL模板 .
现 在 添 加 一 个 输 出 类 似 “Hello World” 的 代 码 。 单 击 CAPL Functions浏览框，右键System→New Event Hander→On preStart到代码行。添加一行write函数，仿真工程开始之前将在Write窗口输出一行“This is my first CANoe Simulation！”。  

```c
on preStart
{
	write("This is my first CANoe Simulation!\n");
}
```

### 添加Switch代码

基 于 上 面 的 代 码 ， 在 CAPL Functions 浏 览 框 ， 拖 曳 Value Objects→On sysvar＜sysvar＞到代码行，并将代码修改如下。  

```c
on sysvar sysvar::MyNamespace::svSwitch{
message Msg1 msg;
msg.bsSwitch = SysGetVarialeInt(sysvar::MyNamespace::svSwitch);
output(msg);
}
```

这段代码使得节点Switch根据系统变量svSwitch的变化，修改bsSwitch 信 号 值 ， 并 将 更 新 的 报 文 发 送 到 总 线 上 。 单 击 工 具 栏Compile（F9）完成编译，并退出CAPL Browser。  

### 添加Light代码

双击Light节点打开对应的CAPL程序文件Light.can。在CAPL Functions浏览框，拖曳CAN→on message＜newMessage＞到代码行，并将代码修改如下  

```C
on message Msg1{
SysSetVariableInt(sysvar::MyNamespace::svLight,this.bsSwitch);
}
```

这段代码将在Light节点中处理收到的CAN报文Msg1，根据报文中信号bsSwitch修改系统变量svLight的值，从而实现LED指示灯的点亮或熄灭。单击工具栏中Compile（F9）图标 完成编译，并退出CAPL Browser。  

# 总线数据库的设计

## 概述

总线系统中，ECU之间的通信、信息的交互以及相互之间的关系，都是通过总线数据库来管理的。总线数据库定义了总线系统中各个ECU所要发送和接收的报文，以及每个报文所有比特值的具体定义。

> [!TIP] 
>
> 提示：数据库中的message中包含了signal，在报文中设置信号就是设置报文的组成形式

## CANdb++Editor

可以使用CANdb++Editor对数据库文件进行编辑

![image-20240722172236897](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/22_17_22_37_202407221722112.png)

### File菜单

![image-20240722172415919](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/22_17_24_16_202407221724120.png)

### 工具栏

![image-20240722172438608](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/22_17_24_38_202407221724859.png)

## 创建CAN数据库（dbc）

### 基于模板新建总线数据库

在CANdb＋＋Editor主界面，选择File→Create Database命令新建数据库文件，此时可以查看到软件自带的数据库模板  。

![image-20240723164406330](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_4_33_202407231704001.png)

根据不同的需求在模板的基础上进行创建数据库。

![image-20240723165144937](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_4_52_202407231704099.png)

### Network

双击新建的xvehicle

![image-20240723170632263](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_6_32_202407231706364.png)

在窗口中有4个选项卡，分别为Definition、Nodes、Attributes和Comment。此处将Definition中的Protocol属性改为CAN总线。  

### ECUs

在ECUs（电子控制单元）项下，列出了当前网络中所含的电控单元，它们之间通过网络节点（Network Nodes）实现信息的交互。通常情况下，ECU与网络节点是一一对应的。当ECU作为网关时，一个ECU可以包含多个网络节点。在CAN数据库中，双击某个ECU可以查看该ECU所对应的网络节点以及环境变量等信息。

需要提醒读者的是，在CAN数据库中并不能直接创建ECU, CANdb＋＋会在创建网络节点的同时，创建一个名称相同的ECU。

### Network Nodes

Network Nodes（网络节点）是ECUs的通信接口，各ECU通过Network Nodes实现总线上信息的发送和接收，每个Network Nodes包含对应的名称和地址 。

新建一个node之后会弹出设置这个node的页面，设置地址为0x01

![image-20240723171040382](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_10_40_202407231710512.png)

同样的再新建一个ndoe--door，地址为0x02，display地址为0x03

![image-20240723171400640](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_14_0_202407231714843.png)

当新建node之后会发现同步新建了三个同门ECU，并且同名ECU和node之间是关联的。

### Message

Messages（报文）是总线上节点相互通信的数据，数据库中每个报文应包含下列属性

- Name（报文名称）

- CAN ID（CAN标识符）
- DLC（Data Length Code，数据长度）
- Type（传输类型）
- Cycle Time（周期）
- Signals（信号）
- Transmitters（发送节点） 
- Receivers（接收节点） 
- Layout（布局） 
- Attributes（通用属性） 
- Comment（说明）

==在报文中可以添加信号，信号组成报文==

在 导 航 区 栏 中 右 击 Message 选 择 New ，创 建 一 个 名 为 EngineState 的 报 文 ， 选 择 标 准CAN（CAN Standard）报文，标识符为（ID）0x150，数据长度（DLC）为2。

![image-20240723171826027](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_18_26_202407231718154.png)

在Transmitters选项卡下单击Add按钮，将Engine节点添加到发送节点。在Transmitters设定界面中指定了该报文由节点Engine发送，  

![image-20240723172408309](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_24_8_202407231724478.png)

用同样的方法，创建名为DoorState的报文，选择标准CAN（CAN Standard）报文，标识符（ID）为0x200，数据长度（DLC）为1，发送节点为Door。

![image-20240723172520033](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_25_20_202407231725157.png)

### Signals

Signal（信号）是总线通信的最小单元，数据库中一个信号由下列属性组成。

- Name（信号名称）
- Length［Bits］（信号长度）
- Byte Order（字节顺序）
- Value Type（数据类型）
- Unit（物理单位）
- Init.Value（初始值）
- Factor（加权）
- Maximum（最大值）
- Minimum（最小值）
- Value Table（数值表）
- Messages（报文）
- Receivers（接收节点）
- Attributes（通用属性）
- Value Descriptions（数值描述）
- Comment（说明）

CANdb ＋ ＋ 中 ， 信 号 的 数 据 类 型 分 为 4 种 ： Signed 、 Unsigned 、 IEEE Float 和 IEEE Double，  

![image-20240723173036800](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/23_17_30_36_202407231730951.png)

接下来，将创建4个信号：引擎速度（EngineSpeed）

在左侧栏中右击Signal选择New，创建信号EngineSpeed, Length为15b, Byte Order为Intel, Unit为r.p.m.，Value Type为Signed, Maximum为5500，其他设置可以使用默认值，如图8.10所示。

![image-20240724170204251](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/24_17_2_4_202407241702402.png)

Factor和Offset定义了raw value与physical value之间的关系。

- raw value是CAN报文发到总线上的十六进制数据
- physical value是信号所代表的物理量的值
- 例如，车速、转速、温度等。Init.Value、Minimum和Maximum均为physical value。  

raw value 与 physical value 之 间 的 关 系 为 ： 
$$
physical value ＝ （［ raw value ］ \times ［Factor］）＋［Offset］。
$$
在Message选项卡中单击Add按钮将该信号关联到报文EngineState中 

![image-20240724170903532](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/24_17_9_3_202407241709629.png)

前面设置了Message的发送方Node，实际上Node还有接受方，接受使用singal的形式。

单击左侧栏中的Network Nodes，右击Display，选择Edit Node，在弹出的对话框中，选择Mapped Rx Sig.选项卡，单击Add按钮，将信号EngineSpeed添加到其中。

![image-20240724172226674](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/24_17_22_26_202407241722770.png)

### Environment Variable  

Environment Variable（环境变量）是==ECU、面板和CAPL程序相连接的媒介==。例如，在CAPL程序中，通过改变或监控某一环境变量的值可以触发特定的动作，同样，环境变量的值也可以与面板上控制控件或显示控件相关联。

在CANdb＋＋的导航区中，右击Environment Variable，选择New命令创建一个名为EnvDoorState的环境变量

![image-20240724172709902](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/24_17_27_10_202407241727003.png)

参数解释如下

![image-20240724172732103](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/24_17_27_32_202407241727244.png)

### Attribute  

Attribute（属性）定义了Vector CAN工具的通用属性，有预定义属性和用户自定义属性。在CANdb＋＋主界面上，单击工具栏中属性图标进入Attribute Definitions界面，也可以通过View→Attribute Definitions打开。

![image-20240726144431554](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/26_14_44_31_202407261444716.png)

双击进去就可以看到各个参数的页面。

数据库的常见属性

![image-20240726144859022](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/26_14_48_59_202407261448229.png)



### Value Table  

Value Table（数值表）用来文字化地指定信号或环境变量的值所代表的含义，例如，前面创建的信号OnOff，0代表Off状态，1代表On状态。  类似于c语言中的宏定义。

![image-20240726145334516](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/26_14_53_34_202407261453594.png)



### Byte Order  

数据库中信号Byte Order（字节顺序）分为Motorola和Intel两种数据格式（也称为大端模式和小端模式），两种格式的字节顺序排列如下。  

![image-20240726145449716](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/26_14_54_49_202407261454778.png)

# PANEL设计



点击*==Tools > Panel Designer==*打开设计面板

![设计面板介绍](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240802155735045.png)

## 面板介绍

![PANEL的HOME功能区](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/2_16_1_7_202408021601738.png)



## 控件介绍



![控件信息](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/2_16_7_36_202408021607172.png)



## 和系统变量交互

首先需要新建一个系统变量

点击*==Environment > System Variable==*新建一个系统变量

![打开创建系统变量面板](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/2_17_24_15_202408021724943.png)

点击左侧的新建按钮

![新建系统变量](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/2_17_27_7_202408021727773.png)

创建完成系统变量之后就是将系统变量和面板上的控件关联起来。这样才能实现系统变量和面板的交互。

切换到panel编辑器中，点击控件，首先点击symbol filter，选择variable，之后选择sysmbol中的环境变量即可。

访问和设置环境变量中CAPL笔记中有相关的读取和设置参数。

# 仿真工程编译和调试

为了创建==CBF==（CAPL Binary Format）格式的可执行文件，用户需要使用CAPL编译器对所编写的程序进行编译。用户可以单击工具栏中的Compile图标或者按F9键启动编译命令。如果CAPL程序还未命名，CAPL浏览器会提示用户输入文件名并保存。如果程序已命名，则*.cbf文件的名称将直接来自源文件名。编译成功后，CBF文件会自动保存在与CAPL程序文件相同的文件夹下  

> [!TIP]
>
> 提示：CAPL语言基本上和C语言很类似，所以也支持条件编译
>
> ```c
> #ifdef
> 
> #elif
> 
> #endif
> ```
>
> 

CAPL的调试和一般调试相同。

# CAN仿真

## 整体架构

仿真工程的工作流如下图所示

![仿真工程开发流程 ](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/6_16_50_38_202408061650230.png)

## 工程仿真

1. 创建仿真工程
2. 设计或导入DBC文件
3. 创建相关的系统变量
4. 设计面板，将面板中的控件和系统变量绑定在一起
5. 使用CAPL语言进行编程

完成上述步骤之后就可以开始仿真

### Trace窗口

Trace（追踪）窗口主要用于记录和显示测量过程中的所有活动，包括收发报文、错误帧、系统变量、环境变量和诊断服务等。在已加载了Database的工程文件中，Trace窗口可以解析出每个报文和信号。Trace窗口主要有以下功能。  

![trace窗口](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/16_14_45_8_202408161445145.png)

1. 预定义过滤器（Predefined Filter）：单击Trace窗口工具栏图标 ，打开预定义过滤器视图。这些不同形式的过滤器可以设置Measurement Setup的过滤器、总线系统事件（Bus Systems）过滤器、系统变量／环境变量过滤器、系统报文（System Messages）过滤器等。
2. 分析过滤器（Analysis Filter）：单击Trace窗口工具栏图标 ，打开分析过滤器视图。用于减少窗口中的显示数据，可以把Trace 中 不 关 心 的 项 拖 曳 到 Stop Filter ， 将 关 心 的 项 拖 曳 到 Pass Filter。
3. 淡出无变化的数据：为了提高查看的便利性，固定时间内没有更新的数据显示将逐渐淡出。单击Trace窗口工具栏图标 ，将删除淡出的事件或数据。
4. 设定marker（标识）：marker可以将指定的事件标识出来，并与对应的时间戳相关联。设定的maker也可以在其他分析窗口中显示（如Graphic窗口）。该功能只能在measurement暂停或停止时设定。
5. 显示统计数据：显示报文、信号、变量等各种信息，包括当前数值、时间戳等，可以详细地以不同形式显示。
6. 日志数据：窗口输出可以部分或全部导出，也可以直接保存到指定的文件中。已保存的日志文件，也可以导入到Trace窗口，进行离线分析。日志数据的设定、导入、导出等相关详细介绍.

### Graphics窗口

在Graphic（图形）窗口中，信号、变量和诊断参数可以以图形化形式显示（XY图形）。用户可以将X轴配置成时间或其他变量。以下为Graphic窗口支持的主要功能。  

![Graphics页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/16_15_15_18_202408161515550.png)

1. 设定marker（标识）：marker可以将指定的事件标识出来，并与对应的时间戳相关联。设定的maker也可以在其他分析窗口中显示。该功能只能在measurement暂停或停止时设定。
2. 显示测量栏：在图例中，每个信号的全局和局部数据的最大值和最小值会自动表示出来，同一类型的信号可以比较数值的差异。
3. 显示数据统计：信号的最小值、最大值、平均值和标准偏差显示在Graphic窗口中。该功能只能在measurement停止时可用。
4. X/Y mode：在信号列表中右击任何一条信号，都可以设置为X轴的变量。
5. 日志数据：Graphic窗口中的信号可以自动保存到事先定义的log文件中（MDF格式），也可以导入已保存的文件分析signal数据

### State Tracker窗口

State Tracker（状态追踪）窗口是用来显示比特值和一些状态值，特别适合显示数字输入量和输出量，以及状态信息，如图4.9所示。该功能适合用于分析状态及状态转换相关的信号和变量。以下为State Tracker窗口支持的主要功能。    

![State trace窗口](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/16_15_31_31_202408161531760.png)

1. 发现错误：基于状态响应时间、信号和状态切换的分析，可以有效地监控相关功能，及时发现错误。
2. 分析信息：不同的信息，如ECU内部通信，总线信号以及I/O口的状态，可以放在一起分析。
3. 设定触发：用户可以定义开始测量后的触发条件。
4. 设定marker：可以用marker标识不同的时间点，两个marker点的时间间隔可以测量出来。

### Data窗口

Data（数据）窗口可以显示信号、系统变量和诊断参数的数值、单位等信息，并以不同的方式显示出来。  

![data窗口](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/16_16_15_57_202408161615263.png)

1. 显示数值：可以显示数据的原始值（Raw Value）、物理值、单位等信息。可以显示信号在全部或部分时间内的最大值和最小值。
2. 日志数据：测量过程中可记录信号并保存到log文件中（MDF格式）。

### Statistics窗口  

Statistics（统计）窗口用于统计Measurement Setup窗口运行过程中的总线（CAN、LIN、FlexRay等）活动，可以在Measurement Setup窗口中插入Statistics功能。下图所示为CAN总线活动情况，主要显示的信息包括总线负载（也可基于节点级、报文级）、突发帧、标准帧、扩展帧、远程帧、错误帧和控制器状态等。  

![static窗口](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/16_16_22_59_202408161622685.png)

1. 显示各个通道的统计数据：显示数据可以限定在指定的通道，或者配置为针对所有可用通道。

2. 设定刷新间隔：可以修改刷新间隔修改统计的时间间隔。

3. 暂停统计：用户可以在Measurement运行过程中暂停统计
### Scope窗口

Scope （ 示 波 器 ） 窗 口 需 要 配 置 额 外 的 授 权 选 项 （ CANoe Option.Scope）和对应的硬件接口，可以观察总线电平，用于分析总线协议，可以使用EYE图表评估信号品质。
![Scope窗口](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/16_16_45_7_202408161645481.png)

















