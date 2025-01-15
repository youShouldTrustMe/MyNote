# 参考链接

[Stateflow 快速入门 - MathWorks 中国](https://ww2.mathworks.cn/help/stateflow/getting-started.html)

# Stateflow简介

Stateflow是MATLAB和Simulink中的一个工具，用于设计和实现状态机和流程图。它非常适合用于建模逻辑系统，如控制逻辑、任务调度和事件驱动系统。Stateflow通过图形化的方式使复杂的逻辑系统更容易理解和实现。以下是一些关于Stateflow的基本概念和操作指南：

# Stateflow基本概念

1. **状态（State）**：

- 表示系统在某一特定时间的情况或条件。每个状态可以包含子状态、图表和动作（如进入和退出动作）。

2. **转换（Transition）**：

   - 用箭头表示，描述从一个状态到另一个状态的条件。转换可以包含条件、动作和事件。
3. **事件（Event）**：

   - 触发状态转换的信号。事件可以是外部事件（来自Simulink模型）或内部事件（在Stateflow图表中定义）。
4. **条件（Condition）**：

   - 在方括号内定义的逻辑表达式，用于决定状态转换是否发生。
5. **动作（Action）**：

   - 在状态或转换中定义的操作，可以在进入、退出状态时或在转换时执行。

# 基本命令

Stateflow是MATLAB和Simulink中的一个工具，用于设计和模拟状态机和逻辑系统。以下是一些基本的Stateflow命令和操作：

## 创建和编辑图表

1. **创建Stateflow图表**:

   ```matlab
   sfnew
   ```

2. **打开现有图表**:

   ```matlab
   sfopen
   ```

3. **保存图表**:

   ```matlab
   sfsave
   ```

## 添加状态和转换

1. **添加状态**:
   - 在图表中右键点击并选择“Add State”，或者使用工具栏中的“Add State”按钮。

2. **添加转换**:
   - 在两个状态之间绘制一条线，或者右键点击一个状态并选择“Add Transition”。

## 设置状态属性

1. **设置状态名称**:
   - 双击状态，并在对话框中输入名称。

2. **设置进入动作（Entry）**:
   - 双击状态，选择“Actions”选项卡，并在“Entry”框中输入代码。

3. **设置退出动作（Exit）**:
   - 双击状态，选择“Actions”选项卡，并在“Exit”框中输入代码。

## 设置转换条件

1. **添加条件**:
   - 双击转换线，并在对话框中输入条件，如 `[condition]`.

2. **添加动作**:
   - 在条件后面添加动作，如 `[condition]{action}`.

## 仿真和调试

1. **运行仿真**:
   - 点击Simulink工具栏中的“Run”按钮。

2. **设置断点**:
   - 在图表中右键点击状态或转换，并选择“Set Breakpoint”。

3. **查看仿真数据**:
   - 在MATLAB命令窗口中使用 `get` 函数查看数据，例如 `get_param('my_chart', 'SimulationStatus')`.

## MATLAB命令与Stateflow接口

1. **获取图表句柄**:

   ```matlab
   chart = sfroot.find('-isa', 'Stateflow.Chart', '-and', 'Name', 'my_chart');
   ```

2. **访问状态**:

   ```matlab
   state = chart.find('-isa', 'Stateflow.State', '-and', 'Name', 'my_state');
   ```

3. **访问转换**:

   ```matlab
   trans = chart.find('-isa', 'Stateflow.Transition', '-and', 'Source', state);
   ```

4. **设置状态动作**:

   ```matlab
   state.Entry = 'entry_action';
   state.During = 'during_action';
   state.Exit = 'exit_action';
   ```

5. **设置转换条件**:

   ```matlab
   trans.Condition = 'condition_expression';
   trans.ConditionAction = 'condition_action';
   ```

这些是Stateflow中的一些基本命令和操作，可以帮助你开始设计和仿真状态机和逻辑系统。如果你有更具体的问题或需要更详细的说明，请告诉我！

# 有限状态机的建模

## 步骤1：构造并运行Stateflow图

Stateflow® 图是有限状态机的图形表示，由状态、转移和数据组成。您可以创建 Stateflow 图来定义系统如何响应外部输入信号、事件和基于时间的条件。

例如，下面的 Stateflow 图展示半波整流器的基础逻辑。该图包含两个标签为 `On` 和 `Off` 的状态。在 `On` 状态下，图输出信号 `y` 等于输入 `x`。在 `Off` 状态下，输出信号设置为零。当输入信号跨越某个阈值 `t0` 时，图在这些状态之间转移。各个状态下的动作在仿真的每一时间步都会更新 `y` 的值。

![Stateflow chart with two states.](https://ww2.mathworks.cn/help/stateflow/gs/rectify-step4.png)



此示例说明如何创建和仿真此 Stateflow 图。

### 构造 Stateflow 图

#### 打开 Stateflow 编辑器

Stateflow 编辑器是一个图形环境，用于设计状态转移图、流程图、状态转移表和真值表。

1. 要打开包含 Stateflow 图的新 Simulink 模型，请在 MATLAB 命令行窗口中使用 [`sfnew`](https://ww2.mathworks.cn/help/stateflow/ref/sfnew.html) 函数：

   ```shell
   sfnew rectify
   ```

   Simulink® 创建一个名为 `rectify` 的模型，其中包含一个空的 Stateflow [Chart](https://ww2.mathworks.cn/help/stateflow/ref/chart.html) 模块。

2. 要打开 Stateflow 编辑器，请双击图模块。

Stateflow 编辑器的主要组件是图画布、对象选项板和**符号**窗格。

- 图画布是一个绘图区域，您可以在其中通过组合状态、转移和其他图形元素来创建图。
- 在画布的左侧有一个对象选项板，其中显示了一组可向图中添加图形元素的工具。
- 在画布的右侧有一个**符合**窗格，您可以用它向图添加新的数据、事件和消息并解析任何未定义或未使用的符号。



![Default view of the Stateflow Editor.](https://ww2.mathworks.cn/help/stateflow/gs/stateflow-editor.png)

#### 添加状态和转移

1. 在对象选项板中（组件面板），点击**状态**图标 ![img](https://ww2.mathworks.cn/help/stateflow/gs/palette-state.png) 并将指针移至图画布。将出现具有默认转移的状态。要放置该状态，请点击画布上的某个位置。在文本提示中，输入状态名称 `On` 和状态动作 `y = x`。

   ![Chart with one state, On.](https://ww2.mathworks.cn/help/stateflow/gs/rectify-step1.png)

   

2. 添加另一个状态。右键点击并拖动 `On` 状态。蓝色图形提示可以帮助您水平或垂直对齐状态。新状态的名称变为 `Off`。双击该状态并将状态动作修改为 `y = 0`。

   ![Chart with two states, On and Off.](https://ww2.mathworks.cn/help/stateflow/gs/rectify-step2.png)

   

3. 重新对齐两个状态并在两个状态之间的空白处停留片刻。蓝色转移提示指示您可以连接状态的几种方式。要添加转移，请点击适当的提示。

   或者，要绘制转移，请点击并从一个状态的边拖动到另一个状态的边。

   ![Chart with the two states joined by transitions.](https://ww2.mathworks.cn/help/stateflow/gs/rectify-step3.png)

   

4. 双击每个转移并输入适当的转移条件 `x<t0` 或 `x>=t0`。条件出现在方括号内。

   ![Chart with transition conditions.](https://ww2.mathworks.cn/help/stateflow/gs/rectify-step4.png)

   

5. 清理图：

   - 为使图更加清晰，将每个转移标签移到其对应转移上方或下方的方便位置。
   - 要对齐图的图形元素并调整其大小，请在**格式**选项卡中，点击**自动排列**或按 **Ctrl+Shift+A**。
   - 要调整图的大小以适合画布，请按空格键或点击**适应视图大小**图标 ![img](https://ww2.mathworks.cn/help/stateflow/gs/fit-to-view.png)。

#### 解析未定义的符号

在执行图之前，必须定义图中使用的每个符号并指定其作用域（例如，输入数据、输出数据或局部数据）。

1. 在**建模**选项卡的**设计数据**下，选择**符号窗格**。

   在**符号**窗格中，未定义的符号用红色错误标记 ![img](https://ww2.mathworks.cn/help/stateflow/gs/symbols-error.png) 进行标记。

2. 在**符号**窗格中，点击**解析未定义的符号** ![img](https://ww2.mathworks.cn/help/stateflow/gs/symbols-resolve-undefined.png)。**类型**列根据每个未定义符号在图中的用例显示其建议作用域。Stateflow 编辑器将符号 `x` 和 `t0` 解析为输入数据 ![img](https://ww2.mathworks.cn/help/stateflow/gs/symbols-input-data.png)，将 `y` 解析为 ![img](https://ww2.mathworks.cn/help/stateflow/gs/symbols-output-data.png)。

   ![Before and after views of the Symbols pane resolving the undefined symbols t0, x, and y.](https://ww2.mathworks.cn/help/stateflow/gs/rectify-resolve-symbols.png)

   

3. 由于阈值 `t0` 在仿真过程中不会更改，因此将其作用域更改为常量数据。在**类型**列中，点击 `t0` 旁边的数据类型图标，然后选择![img](https://ww2.mathworks.cn/help/stateflow/gs/symbols-constant-data.png) **常量数据**。

4. 设置阈值 `t0` 的值。在**值**列中，点击 `t0` 旁边的空白输入框，并输入值 0。

5. 保存您的 Stateflow 图。

### 对模型进行仿真

要仿真 Simulink 模型，请通过输入和输出端口将图模块连接到模型中的其他模块。

1. 要返回到 Simulink 编辑器，请在画布顶部的浏览器栏中点击 Simulink 模型的名称：![img](https://ww2.mathworks.cn/help/stateflow/gs/simulink-model.png)**rectify**。如果浏览器栏不可见，请点击对象选项板顶部的**隐藏/显示资源管理器栏**图标 ![img](https://ww2.mathworks.cn/help/stateflow/gs/palette-explorer-bar.png)。

2. 执行以下操作以将信源添加到模型中：

   - 从 Simulink Sources 库中，添加一个 [Sine Wave](https://ww2.mathworks.cn/help/simulink/slref/sinewave.html) (Simulink) 模块。
   - 双击 Sine Wave 模块并将**采样时间**设置为 0.2。
   - 将 Sine Wave 模块的输出连接到 Stateflow 图的输入。
   - 将信号标记为 `x`。

3. 向模型中添加一个信宿：

   - 从 Simulink Sinks 库中，添加一个具有两个输入端口的 [Scope](https://ww2.mathworks.cn/help/simulink/slref/scope.html) (Simulink) 模块。
   - 将 Sine Wave 模块的输出连接到 Scope 模块的第一个输入。
   - 将 Stateflow 图的输出连接到 Scope 模块的第二个输入。
   - 将信号标记为 `y`。

4. 保存 Simulink 模型。

   ![In a Simulink model, a Sine Wave block creates an input signal for the chart. A Scope block plots the input and output of the chart.](https://ww2.mathworks.cn/help/stateflow/gs/rectify-model.png)

   

5. 要仿真模型，请点击**运行** ![img](https://ww2.mathworks.cn/help/stateflow/gs/toolstrip-run-simulation.png)。在仿真过程中，Stateflow 编辑器通过图动画突出显示激活状态和转移。

6. 对模型进行仿真后，双击 Scope 模块。示波器显示 Stateflow 图的输入信号和输出信号图。

   ![Scope block showing the input and output of the chart.](https://ww2.mathworks.cn/help/stateflow/gs/rectify-scope.png)

   仿真结果显示整流器滤除了负输入值。

## 步骤2：使用状态动作和转移标签定义图行为

*状态动作*和*转移动作*是您分别在状态内部或转移上编写的指令，用于定义 Stateflow® 图在仿真期间的行为。例如，下图中的动作定义了一个以试验方式验证 Collatz 猜想实例的状态机。对于给定的数值输入 ![$u$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq15012583454694319273.png)，该图通过迭代以下规则来计算冰雹序列 ![$n_0 = u,$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq12219811583566878803.png) ![$n_1,$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq11325635569159405358.png) ![$n_2,$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq10936753333647109350.png) ![$n_3,$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq17216054076818118233.png)：

- 如果 ![$n_i$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq07382008818407024284.png) 为偶数，则 ![$n_{i+1} = n_i / 2$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq01795944520665549579.png)。
- 如果 ![$n_i$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq07382008818407024284.png) 为奇数，则 ![$n_{i+1} = 3n_i+1$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq05435671413409378115.png)。

Collatz 猜想指出，每个正整数都有一个最终达到 1 的冰雹序列。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_01.png)



该图由三个状态组成。在仿真开始时，`Init` 状态通过以下设置来初始化图数据：

- 将局部数据 `n` 设置为输入 `u` 的值。
- 当 `n` 除以 2 时，将局部数据 `n2` 设置为余数。
- 将输出数据 `y` 设置为 `false`。

根据输入的奇偶性，图转移到 `Even` 或 `Odd` 状态。当状态活动在 `Even` 和 `Odd` 状态之间切换时，图会计算冰雹序列中的数字。当序列达到 1 值时，输出数据 `y` 变为 `true`，并触发 Simulink® 模型中的 [Stop Simulation](https://ww2.mathworks.cn/help/simulink/slref/stopsimulation.html) (Simulink) 模块。

### 状态动作类型

状态动作定义当状态被激活时 Stateflow 图的动作。最常见的状态动作类型是 `entry`、`during` 和 `exit` 动作：

- 当状态被激活时，`entry` 动作发生。
- 当状态已激活并且图未转移出该状态时，`during` 动作在时间步上发生。
- 当图转移出该状态时，`exit` 动作发生。

您可以使用完整关键字（`entry`、`during`、`exit`）或缩写（`en`、`du` 和 `ex`）来指定状态动作的类型。您还可以使用逗号组合各状态动作类型。例如，具有组合类型 `entry, during` 的动作当状态被激活时在时间步上发生，并且在状态保持激活时在每个后续时间步上发生。

冰雹图包含以下状态下的动作：

- `Init` - 如果此状态在仿真开始时被激活，`entry` 动作确定 `n` 的奇偶性并将 `y` 设置为 `false`。当图在一个时间步后转移出 `Init` 时，`exit` 动作确定 `n` 是否等于 1。
- `Even` - 当此状态被激活时，在该状态保持激活的每个后续时间步上，组合的 `entry, during` 动作计算冰雹序列的下一个数字 `n/2` 的值和奇偶性。
- `Odd` - 当此状态被激活时，在该状态保持激活的每个后续时间步上，组合的 `entry, during` 动作检查 `n` 是否大于 1，如果大于 1，则计算冰雹序列的下一个数字 `3*n+1` 的值和奇偶性。

### 转移标签类型

转移标签定义当激活状态更改时 Stateflow 图执行的动作。最常见的转移标签类型是条件和条件动作。

```
  [Condition]{ConditionAction}
```

`Condition` 是布尔表达式，用于确定是否发生转移。如果不指定条件，转移将在源状态被激活后的下一个时间步发生。

`ConditionAction` 是在判断转移的条件为 true 时执行的指令。条件动作发生在条件后，但在任何 `exit` 或 `entry` 状态动作之前。

冰雹图包含发生以下转移时的动作：

- 到 `Init` 的默认转移 - 在仿真开始时，条件动作 `n = u` 将输入值 `u` 赋给局部数据 `n`。
- 从 `Init` 到 `Even` 的转移 - 条件 `n2 == 0` 确定当 `n` 为偶数时发生该转移。此转移起始处的数字 1 表示此转移将先于 `Init` 到 `Odd` 的转移进行计算。
- 从 `Odd` 到 `Even` 的转移 - 条件 `n2 == 0` 确定当 `n` 为偶数时发生该转移。
- 从 `Even` 到 `Odd` 的转移 - 条件 `n2 ~= 0` 确定当 `n` 为奇数时发生该转移。在这种情况下，条件动作 `y = isequal(n,1)` 确定 `n` 是否等于 1。

### 检查图行为

要计算从值 9 开始的冰雹序列，请执行下列步骤：

1.在 Constant 模块中，输入值 `9`。

2.在**仿真**选项卡中，点击**运行**。图用下列动作予以响应：

- 在时间 ![$t = 0$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq05490811019241878006.png) 处，发生到 `Init` 的默认转移。转移动作将 `n` 的值设置为 9。`Init` 状态被激活。`Init` 中的 `entry` 动作将 `n2` 设置为 1 且将 `y` 设置为 `false`。
- 在时间 ![$t = 1$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq12716245961660159691.png) 处，条件 `n2 == 0` 为 false，因此图准备转移到 `Odd`。`Init` 中的 `exit` 动作将 `y` 设置为 `false`。状态 `Init` 变为非激活，状态 `Odd` 被激活。`Odd` 中的 `entry` 动作将 `n` 设置为 28，将 `n2` 设置为 0。
- 在时间 ![$t = 2$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq03599317120891285387.png) 处，条件 `n2 == 0` 为 true，因此图准备转移到 `Even`。状态 `Odd` 变为非激活，状态 `Even` 被激活。`Even` 中的 entry 动作将 `n` 设置为 14，将 `n2` 设置为 0。
- 在时间 ![$t = 3$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq07738249947809783232.png) 处，条件 `n2 ~= 0` 为 false，因此图不会发生转移。`Even` 状态保持激活。`Even` 中的 `during` 动作将 `n` 设置为 7，将 `n2` 设置为 1。
- 在时间 ![$t = 4$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq15359278158169810469.png) 处，条件 `n2 ~= 0` 为 true，因此图准备转移到 `Odd`。转移动作将 `y` 设置为 false。状态 `Even` 变为非激活，状态 `Odd` 被激活。`Odd` 中的 `entry` 动作将 `n` 设置为 22，将 `n2` 设置为 0。
- 该图继续计算冰雹序列，直到它在时间 ![$t = 19$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq17315660474862548967.png) 处达到值 `n` ![$= 1$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq13562902622338161563.png)。
- 在时间 ![$t = 20$](https://ww2.mathworks.cn/help/examples/stateflow/win64/StateTransitionActionsGetStartedExample_eq05704149810581774953.png) 处，图准备从 `Even` 转移到 `Odd`。转移动作将 `y` 设置为 `true`。状态 `Even` 变为非激活，状态 `Odd` 被激活。`Odd` 中的 `entry` 动作不会修改 `n` 或 `n2`。连接到输出信号 `y` 的 Stop Simulation 模块停止仿真。

3.在**仿真**选项卡中的**查看结果**下，点击**数据检查器**。

4.要查看冰雹序列的值，请在仿真数据检查器中，选择记录的信号 `n`。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsf_collatz-sdi.png)



冰雹序列在 19 次迭代后达到值 1。

## 步骤3：创建层次结构来管理复杂系统

要控制系统中多层级的复杂度，可在 Stateflow® 图中创建嵌套状态的层次结构，方法是将一个或多个状态置于另一个状态的边界内。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/HierarchyGetStartedExample_01.png)



外部状态是内部状态的父级（或*父状态*）。内部状态是外部状态的子级（或*子状态*）。父状态的内容在行为上与较小的图类似。当父状态被激活时，子状态之一也被激活。当父状态变为非激活时，所有子状态都变为非激活。

### 媒体播放器建模

此示例创建了一个由 FM 广播和网络流媒体播放器组成的媒体系统模型。在仿真期间，您可以通过与 Media Player 用户界面上的按钮和旋钮交互来控制该媒体播放器。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsfMediaPlayerApp.png)



要运行媒体播放器，请执行以下操作：

1. 打开 Simulink 模型，然后点击**运行**。Media Player 将打开。媒体播放器最初处于关闭状态。在 App 的顶部，Media Player Status 框显示消息 `Standby (Off)`。
2. 将 Component Selection 旋钮转至 **Stream**。状态消息会短暂显示 `Connecting to Handel's Greatest Hits`。短暂停顿后，状态消息变为 `Playing: Handel's Greatest Hits`，音乐开始播放。
3. 点击 Fast-Forward 按钮。音乐停止，开始发出啁啾声。状态消息更改为 `Forward >> Handel's Greatest Hits`。流媒体的名称在显示画面上向前滚动。要继续正常播放模式，请点击 Play 按钮。
4. 点击 Reverse 按钮。将播放啁啾声，状态消息变为 `Reverse >> Handel's Greatest Hits`。流媒体的名称在显示画面上向后滚动。要继续正常播放模式，请点击 Play 按钮。
5. 在 Stream Name 框中，输入新流媒体的名称，然后点击 **Connect**。例如，尝试连接到流媒体 `Training Deep Networks` 或 `Fun With State Machines`。
6. 将 Component Selection 旋钮转至 **Radio**。状态消息显示 `Playing: 99.5 FM`。要选择另一个电台，请旋转 FM Radio Station 旋钮。
7. 要停止仿真，请关闭 Media Player。

### 通过使用状态层次结构实现行为

此示例通过一次只关注单个活动层级来实现媒体播放器。例如，流媒体播放器进入快进播放模式需要满足以下条件：

1. 您打开媒体播放器。
2. 您选择流媒体播放器。
3. 您开始播放流媒体。
4. 您点击 Fast-Forward 按钮。

该模型使用嵌套状态的层次结构来单独考虑每个条件。例如，模型资源管理器在 `Mode Manager` 图中显示状态的层次结构。要打开模型资源管理器，请在**建模**选项卡中，选择**模型资源管理器**。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsfMediaPlayer-model-explorer.png)



在层次结构的顶层或最外层，`Mode Manager` 图有两个用于控制媒体播放器活动的状态：

- 当您关闭媒体播放器时，`Off` 被激活。
- 当您将媒体播放器设置为流媒体或广播模式时，`On` 被激活。

`On` 的子状态控制媒体播放器组件：

- 当您选择流媒体播放器时，`Stream` 被激活。
- 当您选择 FM 广播时，`Radio` 被激活。

`Stream` 的子状态控制流媒体播放器的活动：

- 当您播放流媒体时，`Play` 被激活。
- 当您暂停流媒体播放器时，`Pause` 被激活。

`Play` 的子状态控制流媒体播放器的播放模式：

- `Normal`：在正常播放模式下，该状态被激活。
- 当您点击 Reverse 按钮时，`Reverse` 被激活。
- 当您点击 Fast-Forward 按钮时，`FastForward` 被激活。

下图显示了图中状态的布局。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/HierarchyGetStartedExample_02.png)

### 探索示例

此示例中的模型包含另外两个 Stateflow 图：

- `App Interface` 管理与 MATLAB App 的接口，并将输入传递给 `Mode Manager` 和 `Stream Player` 图。
- `Stream Player` 接收来自 `App Interface` 和 `Mode Manager` 图的输出，并仿真流媒体播放器的内部行为。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/HierarchyGetStartedExample_03.png)

在仿真过程中，您可以研究每个图如何响应与 Media Player 的交互。要在各图之间快速切换，请使用 Stateflow 编辑器顶部的选项卡。

## 步骤4：使用并行分解对同步子系统建模

要实现并发运行的工作模式，请在 Stateflow® 图中使用并行状态。例如，作为复杂系统设计的一部分，您可以使用并行状态对同时被激活的独立组件或子系统建模。

### 状态分解

图或状态的分解类型指定图或状态包含互斥状态还是并行状态：

- 互斥状态表示互斥的工作模式。在同一层级上不能有两个互斥状态同时被激活或执行。Stateflow 图用实线矩形表示每个互斥状态。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_01.png)

- 并行状态表示独立的工作模式。虽然并行状态会依次执行，但可以有两个或多个并行状态同时处于激活状态。Stateflow 图用虚线矩形表示每个并行状态，其中的数字表示其执行顺序。两个状态需要在父状态设置为并行执行才能设置为并行状态。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_02.png)

通过在状态层次结构的不同层级设置状态分解，您可以在 Stateflow 图中组合互斥和并行状态。默认的状态分解类型是 `Exclusive (OR)`。要将分解类型更改为 `Parallel (AND)`，请右键点击父状态并选择**分解** > **并行 (AND)**。要将分解类型改回 `Exclusive (OR)`，请右键点击父状态并选择**分解** > **互斥(OR)**。

### 气温控制器建模

此示例使用并行分解对一个空调控制器进行建模，该控制器将物理被控对象中的气温保持在 120 度。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_03.png)

在顶层，空调控制器图有两个互斥状态：`PowerOff` 和 `PowerOn`。该图使用互斥 (OR) 分解，因为控制器无法同时处于打开和关闭状态。

控制器使用两个风扇来工作。当气温升至 120 度以上时，启动第一个风扇。当气温升至 150 度以上时，启动第二个风扇以提供额外的冷却。图将这些风扇建模为顶层状态 `PowerOn` 的并行子状态 `FAN1` 和 `FAN2`。由于风扇作为独立的组件运行，根据所需冷却量的多少打开或关闭，`PowerOn` 使用并行 (AND) 分解来确保在控制器打开时两个子状态都被激活。

除工作阈值不同以外，这些风扇由具有相同子状态和转移配置的状态建模，表示两种风扇工作模式 `On` 和 `Off`。由于两个风扇都不能同时处于打开和关闭状态，`FAN1` 和 `FAN2` 具有互斥 (OR) 分解。

在 `PowerOn` 中，称为 `SpeedValue` 的第三个并行状态表示一个独立子系统，它计算在每个时间步内循环启动的风扇数。当 `FAN1` 的 `On` 状态被激活时，布尔表达式 `in(FAN1.On)` 的值为 1。否则，`in(FAN1.On)` 等于 0。同样，`in(FAN2.On)` 的值指示 `FAN2` 是处于循环打开还是关闭状态。这些表达式的总和表示在每个时间步中打开的风扇的数量。

### 指定并行状态的执行顺序

即使 `FAN1`、`FAN2` 和 `SpeedValue` 同时处于激活状态，这些状态在仿真过程中也是依次执行的。状态右上角的数字指定执行顺序。以下是此执行顺序的基本原理：

- `FAN1` 应首先执行，因为它循环打开的温度低于 `FAN2` 打开的温度。无论 `FAN2` 是打开还是关闭，它都可以打开。
- `FAN2` 以第二顺位执行，因为它循环打开的温度高于 `FAN1` 打开的温度。它仅在 `FAN1` 已打开时才能打开。
- `SpeedValue` 最后执行，因而可以观测 `FAN1` 和 `FAN2` 的最新状态。

默认情况下，Stateflow 根据您将并行状态添加到图的顺序来分配并行状态的执行顺序。要更改某并行状态的执行顺序，请右键点击该状态并从**执行顺序**下拉列表中选择值。

### 探索示例

此示例包含一个名为 `Air Controller` 的 Stateflow 图和一个名为 `Physical Plant` 的 Simulink® 子系统。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_04.png)

根据物理被控对象的气温，图打开风扇，并向子系统输出正在运行的风扇的数量 `airflow`。此值根据以下规则确定冷却活动因子 ![$k_{\mathrm Cool}$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq03091110158048648509.png)：

- `airflow` = 0 - 没有风扇运行。气温不会降低，因为 ![$k_{\mathrm{Cool}} = 0$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq14394746751260252312.png)。
- `airflow` = 1 - 一个风扇运行。气温降低，冷却活动因子 ![$k_{\mathrm{Cool}} = 0.05$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq16298905221389238741.png)。
- `airflow` = 2 - 两个风扇运行。气温降低，冷却活动因子 ![$k_{\mathrm{Cool}} = 0.1$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq12844725938971953643.png)。

Physical Plant 子系统块根据以下公式更新工厂内的气温 ![$temp$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq02137513487127062277.png)：

![$$temp(0) = T_{\mathrm{Initial}}$$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq15984319721611355580.png)

![$$temp'(t) = (T_{\mathrm{Ambient}}-temp(t)) \cdot (k_{\mathrm{Heat}}-k_{\mathrm{Cool}}),$$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq05892922162902343283.png)

其中：

- ![$T_{\mathrm{Initial}}$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq01572441086277914966.png) 是初始温度。默认值为 70°。
- ![$T_{\mathrm{Ambient}}$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq10604485442030496953.png) 是环境温度。默认值为 160°。
- ![$k_{\mathrm{Heat}}$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq07996253399464447700.png) 是被控对象的热传递因子。默认值为 0.01。
- ![$k_{\mathrm{Cool}}$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ParallelDecompositionGetStartedExample_eq07628547966429477945.png) 是对应于 `airflow` 的冷却活动因子。

新温度决定仿真的下一个时间步的冷却量。

## 步骤5：通过广播事件同步并行状态

局部事件允许一个状态触发同一个 Stateflow® 图中另一个状态的转移或动作，从而使您能够协调并行状态。要将事件从一个状态广播到另一个状态，请使用 `send` 运算符以及事件的名称和激活状态的名称：

```shell
send(eventName,stateName)
```

当您广播事件时，该事件将在接收状态以及该状态层次结构中的任何子状态中生效。

### 家庭安全系统建模

此示例使用局部事件作为家庭安全系统设计的一部分。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/EventsGetStartedExample_01.png)

该安全系统由一个警报器和三个防入侵传感器（门传感器、窗传感器和运动检测器）组成。在系统检测到入侵后，您可以在较短的一段时间内禁用警报。否则，系统会向警方报告。

该图通过使用以下并行状态之一对每个传感器进行建模：

- 并行状态 `Door` 对门传感器进行建模。输入信号 `D_mode` 在此传感器的 `Active` 和 `Disabled` 模式之间进行选择。当传感器被激活时，输入信号 `Door_sens` 指示可能存在入侵。
- 并行状态 `Win` 对窗传感器进行建模。输入信号 `W_mode` 在此传感器的 `Active` 和 `Disabled` 模式之间进行选择。当传感器被激活时，输入信号 `Win_sens` 指示可能存在入侵。
- 并行状态 `Motion` 对运动检测器进行建模。输入信号 `M_mode` 在此传感器的 `Active` 和 `Disabled` 模式之间进行选择。当传感器被激活时，输入信号 `Mot_sens` 指示可能存在入侵。

为减轻偶发误报的影响，运动检测器采用了一种去抖设计，因此只有持续的正触发信号才会产生警报。相反，门传感器和窗传感器将单个正触发信号解释为入侵并立即发出警报。

称为 `Alarm` 的第四种并行状态对警报系统的工作模式进行建模。输入信号 `Alarm_active` 在警报的 `On` 和 `Off` 模式之间进行选择。如果一个传感器在警报子系统开启时检测到入侵，该传感器会将局部事件 `Alert` 广播到 `Alarm` 状态。在状态 `Alarm` 的 `On` 子状态中，该事件触发从 `Idle` 子状态到 `Pending` 子状态的转移。当 `Pending` 被激活时，会发出警报声，提醒住户可能发生入侵。如果发生误报，则住户可以在较短的一段时间内关闭安全系统。如果在这段时间内没有禁用，系统会向警方报警求助，然后返回 `Idle` 模式。

### 与其他 Simulink 模块协调

Stateflow 图也可以使用事件与 Simulink® 模型中的其他模块进行通信。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/EventsGetStartedExample_02.png)

**输出事件**

输出事件是在 Stateflow 图中发生但在 Stateflow 图之外的 Simulink 模块中可见的事件。这种类型的事件支持 Stateflow 图将该图中发生的事件通知给其他模块。例如，在此示例中，输出事件 `Sound` 和 `call_police` 负责驱动用来处理警报声和向警方报警的外部模块。当局部事件 `Alert` 触发到状态 `Alarm` 的 `Pending` 子状态的转移时，图会广播这些事件。特别是，在 `Pending` 子状态中，entry 动作会广播 `Sound` 事件。同样，发生从 `Pending` 到 `Idle` 的转移时执行的条件动作会广播 `call_police` 事件。在每种情况下，广播输出事件的动作都使用 `send` 运算符和事件名称：

```
send(eventName)
```

每个输出事件映射到图上的一个输出端口。根据配置，对应的信号可以控制触发子系统或函数调用子系统。要配置输出事件，请执行以下操作：

1. 在**建模**选项卡的**设计数据**下，选择**符号窗格**和**属性检查器**。
2. 在**符号**窗格中，选择该输出事件。
3. 在**属性检查器**中，将**触发器**设置为以下选项之一：

- `Either edge` - 输出事件广播导致传出信号在 0 和 1 之间切换。
- `Function call` - 输出事件广播导致 Simulink 函数调用事件。

在此示例中，输出事件使用边沿触发器来激活 Simulink 模型中的一对锁存子系统。当每个锁存子系统检测到其输入信号中的值发生变化时，它会短暂输出值 1，然后再返回到 0 输出。

**输入事件**

输入事件是在 Simulink 模块中发生但在 Stateflow 图中可见的事件。这种类型的事件支持其他 Simulink 模块（包括其他 Stateflow 图）将在特定 Stateflow 图之外发生的事件通知该图。例如，在此示例中，输入事件 `sl_call` 控制运动检测器去抖器的时序以及向警方报警之前的短时延迟。在每个实例中，对时序运算符 `after` 的调用内会对该事件计数，并在图接收事件达一定次数后触发转移。

外部 Simulink 模块通过连接到 Stateflow 图上的触发端口的信号发送输入事件。根据具体配置，输入事件是由信号值的变化或通过 Simulink 模块中的函数调用产生的。要配置输入事件，请执行以下操作：

1. 在**建模**选项卡的**设计数据**下，选择**符号窗格**和**属性检查器**。
2. 在**符号**窗格中，选择该输入事件。
3. 在**属性检查器**中，将**触发器**设置为以下选项之一：

- `Rising` - 当输入信号从零或负值变为正值时，图被激活。
- `Falling` - 当输入信号从正值变为零或负值时，图被激活。
- `Either` - 当输入信号在任一方向变化且过零时，图被激活。
- `Function` 调用 - 图通过从 Simulink 模块的函数调用激活。

在此示例中，一个 Simulink Function-Call Generator 模块通过周期函数调用触发输入事件 `sl_call` 来控制安全系统的时序。

### 探索示例

在此示例中，Stateflow 图从几个 Manual Switch 模块接收输入，并输出到一对连接到 Display 模块的锁存子系统。在仿真过程中，您可以：

- 通过点击 Switch 模块启用警报和传感器子系统并触发入侵检测。
- 观看图动画，其中突出显示了图中的各种激活状态。
- 查看 Scope 模块和仿真数据检查器中的输出信号。

例如，假设您打开警报和传感器子系统，关闭传感器触发器，并开始仿真。在仿真过程中，您执行以下动作：

1. 在 ![$t = 250$](https://ww2.mathworks.cn/help/examples/stateflow/win64/EventsGetStartedExample_eq11392407565812330029.png) 秒的时刻，您触发门传感器。警报开始响起 (`Sound = 1`)，因此您立即禁用警报系统。您关闭门传感器触发器，然后重新打开警报。
2. 在 ![$t = 520$](https://ww2.mathworks.cn/help/examples/stateflow/win64/EventsGetStartedExample_eq08815713516279239351.png) 秒的时刻，您触发窗传感器并且警报开始响起 (`Sound = 0`)。这次，您不禁用警报。在大约 ![$t = 600$](https://ww2.mathworks.cn/help/examples/stateflow/win64/EventsGetStartedExample_eq18318288194079829825.png) 秒的时刻，安全系统向警方报警 (`call_police = 1`)。`Sound` 和 `call_police` 信号继续每隔 80 秒在 0 和 1 之间切换一次。
3. 在 ![$t = 1400$](https://ww2.mathworks.cn/help/examples/stateflow/win64/EventsGetStartedExample_eq01438897797695050524.png) 秒的时刻，您禁用警报。`Sound` 和 `call_police` 信号停止切换。

仿真数据检查器显示 `Sound` 和 `call_police` 信号对您的动作的响应。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsf_security-sdi.png)

## 步骤6：使用激活状态数据监视图的活动

如果 Stateflow® 图包含与图层次结构相关的数据，则可以使用激活状态数据来简化设计。通过启用激活状态数据，您可以：

- 避免手动更新反映图活动的数据。
- 在仿真数据检查器中记录并监视图活动。
- 使用图活动数据来控制其他子系统。
- 将图活动数据导出到其他 Simulink 模块。

例如，在以下交通信号模型中，被激活的状态决定输出信号 `color` 的值。您可以通过启用激活状态数据来简化图的设计。在本例中，Stateflow 图可以通过跟踪状态活动来提供交通信号的颜色，因此您不必显式更新 `color` 的值。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_01.png)



要启用激活状态数据，请选择一个要监控的状态。然后，在**属性检查器**中：

1.选择**创建监控输出**。

2.选择以下活动类型之一：

- `Self activity` - 指示状态是否激活的布尔值
- `Child activity` - 指示哪个子状态被激活的枚举值
- `Leaf state activity` - 指示哪个叶状态被激活的枚举值

3.输入激活状态数据符号的**数据称**。

4.（可选）对于 `Child activity` 或 `Leaf state activity`，输入激活状态数据类型的**枚举名称**。

默认情况下，Stateflow 图将状态活动作为输出数据报告给 Simulink 模型。要将激活状态数据符号的作用域更改为局部数据，请使用**符号**窗格。

### 交通信号控制器建模

此示例使用激活状态数据为一对交通信号灯的控制器系统建模。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_02.png)

在 `Traffic Controller` 图中，由两个平行的子图管理着控制交通信号灯的逻辑。这两个子图具有相同的层次结构，即包含三个子状态：`Red`、`Yellow` 和 `Green`。输出数据 `Light1` 和 `Light2` 对应于子图中的激活子状态。这些信号：

- 确定动画交通信号灯的阶段。
- 帮助统计在每个信号灯下等待的汽车数量。
- 驱动一个 Safety Assertion 子系统，确认两个交通信号灯永远不会同时呈绿色。

要查看 Traffic Controller 图中的子图，请点击图左下角的箭头。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_03.png)

每个交通控制器循环遍历其子状态，从 `Red` 到 `Green` 到 `Yellow`，然后再返回到 `Red`。每个状态对应于交通信号灯循环中的一个阶段。输出信号 `Light1` 和 `Light2` 指示在任意给定时刻哪个状态被激活。

**红灯**

当 `Red` 状态被激活时，交通信号灯循环开始。经过短暂的延迟后，控制器检查在十字路口等待的汽车。如果检测到有至少一辆汽车或者经过了一定的时间，则控制器会将 `greenLightRequest` 设置为 `true`，以此来请求绿灯。在发出请求后，控制器继续保持 `Red` 状态较短的一段时间，直到检测到另一个交通信号为红色，控制器才会将状态转移到 `Green`。

**绿灯**

当 `Green` 状态被激活时，控制器会将 `greenLightRequest` 设置为 `false`，以此来取消其绿灯请求。控制器将 `greenLightLocked` 设置为 `true`，以防止另一个交通信号变绿。在短暂延迟后，控制器会检查是否有来自另一个控制器的绿灯请求。如果它收到请求或经过了一定的时间，则控制器会转移到 `Yellow` 状态。

**黄灯**

当 `Yellow` 状态变为非激活时，控制器会将 `greenLightLocked` 设置为 false，指示另一个交通信号灯可以安全地变绿。在转移到 `Red` 状态之前，控制器将保持 `Yellow` 状态一定的时间。然后开始下一个交通信号灯循环。

**交通信号灯的时序**

交通信号灯循环的时序通过几个参数来定义。要更改交通信号灯的计时，请双击 `Traffic Controller` 图，并在对话框中输入以下参数的新值：

- `REDDELAY` - 从红灯激活到控制器检查十字路口等待车辆之间的时间长度。此值也是控制器发出绿灯请求后，交通信号灯可以变绿的最短时间长度。默认值为 6 秒。
- `MAXREDDELAY` - 控制器在发出绿灯请求前检查车辆的最长时间。默认值为 360 秒。
- `GREENDELAY` - 交通信号灯保持绿色的最长时间。默认值为 180 秒。
- `MINGREENDELAY` - 交通信号灯保持绿色的最短时间长度。默认值为 120 秒。
- `YELLOWDELAY` - 交通信号灯保持黄色的时间长度。默认值为 15 秒。

### 探索示例

1. 点击左下角的箭头打开图。
2. 在**符号**窗格中，选择 `greenLightRequested`。然后，在**属性检查器**中的**记录**下，选择**记录信号数据**。
3. 对 `greenLightLocked`、`Light1` 和 `Light2` 重复上一步骤。
4. 在**仿真**选项卡中，点击**运行**。
5. 在**仿真**选项卡中的**查看结果**下，点击**数据检查器**。
6. 在仿真数据检查器中，将记录的信号显示在单独的坐标区中。布尔信号 `greenLightRequested` 和 `greenLightLocked` 显示为数值 0 或 1。状态活动信号 `Light1` 和 `Light2` 为枚举数据，其值为 `Green`、`Yellow`、`Red` 和 `None`。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsf_trafficlight-sdi.png)



要在仿真过程中跟踪图活动，可以使用仿真数据检查器中的缩放和光标按钮。例如，下面列出了仿真前 300 秒内的关键时刻：

- ![$t = 0$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq05490811019241878006.png) - 在仿真开始时，两个交通信号灯都为红色。`Light1` 和 `Light2` 为 `Red`，`greenLightRequested` 为 `false`，`greenLightLocked` 为 `false`。
- ![$t = 6$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq09978131082994638030.png) - 6 秒（`REDDELAY` 的默认值)后，两条街道上都有汽车在等待，因此两个交通信号灯都请求绿灯。`Light1` 和 `Light2` 仍为 `Red`，`greenLightRequested` 为 `true`，`greenLightLocked` 为 `false`。
- ![$t = 12$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq11951804155291995028.png) - 再过 6 秒（`REDDELAY` 的默认值)后，信号灯 1 变为绿色，取消绿灯请求，并将 `greenLightLocked` 设置为 `true`。然后，信号灯 2 请求绿灯。`Light1` 为 `Green`，`Light2` 为 `Red`，`greenLightRequested` 变为 `false` 然后为 `true`，`greenLightLocked` 为 `true`。
- ![$t = 132$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq15887113488953338383.png) - 120 秒（`MINGREENDELAY` 的默认值)后，信号灯 1 变为黄色。`Light1` 为 `Yellow`，`Light2` 为 `Red`，`greenLightRequested` 为 `true`，`greenLightLocked` 为 `true`。
- ![$t = 147$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq13311095447044303409.png) - 15 秒（`YELLOWDELAY` 的默认值)后，信号灯 1 变为红色，并将 `greenLightLocked` 设置为 `false`。然后，信号灯 2 变为绿色，取消绿灯请求，并将 `greenLightLocked` 设置为 `true`。`Light1` 为 `Red`，`Light2` 为 `Green`，`greenLightRequested` 为 `false`，`greenLightLocked` 变为 `false` 然后为 `true`。
- ![$t = 153$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq01350805154886934534.png) - 6 秒（`REDDELAY` 的默认值)后，信号灯 1 请求绿灯。`Light1` 为 `Red`，`Light2` 为 `Green`，`greenLightRequested` 为 `true`，`greenLightLocked` 为 `true`。
- ![$t = 267$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq10304234090842083408.png) - 在信号灯 2 为绿色 120 秒（`MINGREENDELAY` 的默认值)后，信号灯 2 变为黄色。`Light1` 为 `Red`，`Light2` 为 `Yellow`，`greenLightRequested` 为 `true`，`greenLightLocked` 为 `true`。
- ![$t = 282$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq05922833764625562942.png) - 15 秒（`YELLOWDELAY` 的默认值)后，信号灯 2 变为红色，并将 `greenLightLocked` 设置为 `false`。然后，信号灯 1 变为绿色，取消绿灯请求，并将 `greenLightLocked` 设置为 `true`。`Light1` 为 `Green`，`Light2` 为 `Red`，`greenLightRequested` 为 `false`，`greenLightLocked` 变为 `false` 然后为 `true`。
- ![$t = 288$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq17718326725850168440.png) - 6 秒（`REDDELAY` 的默认值)后，信号灯 2 请求绿灯。`Light1` 为 `Green`，`Light2` 为 `Red`，`greenLightRequested` 为 `true`，`greenLightLocked` 为 `true`。

重复循环，直到仿真于 ![$t = 1000$](https://ww2.mathworks.cn/help/examples/stateflow/win64/ActiveStateDataGetStartedExample_eq01657301476922979099.png) 秒处结束。

## 步骤7：使用时序逻辑调度图动作

要定义 Stateflow 图在仿真时间的行为，请在图的状态和转移动作中包含时序逻辑运算符。时序逻辑运算符是内置函数，告知状态保持激活的时间长度或布尔条件保持为 true 的时间长度。使用时序逻辑，您可以控制以下各项的时序：

- 各状态之间的转移
- 函数调用
- 变量值的更改

以下是最常见的绝对时间时序逻辑运算符：

- `after` - 如果自包含该运算符的状态或包含该运算符的转移的源状态被激活以来经过的仿真时间达到 `n` 秒，则 `after(n,sec)` 返回 `true`。否则，运算符返回 `false`。此运算符支持以秒 (`sec`)、毫秒 (`msec`) 和微秒 (`usec`) 计的基于事件的时序逻辑和绝对时间时序逻辑。
- `elapsed` - `elapsed(sec)` 返回自关联状态激活以来经过的仿真时间的秒数。
- `duration` - `duration(C)` 返回自布尔条件 `C` 变为 `true` 以来经过的仿真时间的秒数。

### Bang-Bang 温度控制器建模

此示例使用时序逻辑对调节锅炉内部温度的 Bang-Bang 控制器建模。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_01.png)

该示例由 Stateflow 图和 Simulink® 子系统组成。`Bang-Bang Controller` 图将当前锅炉温度与参考设定值进行比较，并确定是否开启锅炉。`Boiler Plant Model` 子系统通过根据控制器的状态升高或降低温度，对锅炉内部的动态特性进行建模。然后，图使用锅炉温度进行下一步仿真。

`Bang-Bang Controller` 图使用时序逻辑运算符 `after` 来实现以下目的：

- 当锅炉在开启和关闭之间切换时，调节 Bang-Bang 循环的时序。
- 根据锅炉的工作模式控制以不同速率闪烁的状态 LED。

定义锅炉行为和 LED 子系统行为的计时器彼此独立运行，并不阻断或中断控制器的仿真。

### Bang-Bang 循环的时序

`Bang-Bang Controller` 图包含一对代表锅炉两种工作模式的子状态：`On` 和 `Off`。该图使用激活状态输出数据 `boiler` 来指示哪个子状态为激活状态。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_02.png)

`On` 和 `Off` 子状态之间转移上的条件定义 Bang-Bang 控制器的行为：

- 在发生从 `On` 到 `Off` 的第一个转移时，根据条件 `after(20,sec)`，锅炉在开启 20 秒后关闭。
- 在发生从 `Off` 到 `On` 的转移时，如果图形函数 `cold()` 指示锅炉温度低于参考设定值至少 40 秒，则条件 `after(40,sec)[cold()]` 开启锅炉。
- 在发生从 `On` 到 `Off` 的第二个转移时，`On` 状态中的内部转移逻辑确定在锅炉温度等于或高于参考设定值时关闭锅炉。

由于存在这些转移动作，Bang-Bang 循环的时序取决于锅炉的当前温度。在仿真开始时，锅炉较冷，控制器在 `Off` 状态下保持 40 秒，在 `On` 状态下保持 20 秒。在时间到达 ![$t = 478$](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_eq05317045536900750900.png) 秒时，锅炉的温度达到参考设定值。自这一时刻起，锅炉仅需补偿在 `Off` 状态下的热损失。因而控制器在 `Off` 状态下保持 40 秒，在 `On` 状态下保持 4 秒。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsf_boiler-sdi-temp.png)



### 状态 LED 的时序

`Off` 状态包含子状态 `Flash`，其自环转移由动作 `after(5,sec)` 触发。由于这种转移，当 `Off` 状态被激活时，子状态将执行其 `entry` 动作并每隔 5 秒就调用一次图形函数 `flash_LED`。该函数使输出符号 `LED` 的值在 0 和 1 之间切换。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_03.png)

`On` 状态调用图形函数 `flash_LED` 作为 `entry, during` 组合状态的动作。当 `On` 状态被激活时，此动作在仿真的每个时间步调用该函数，以在 0 和 2 之间切换输出符号 `LED` 的值。

![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_04.png)

因此，状态 LED 的时序取决于锅炉的工作模式。例如：

- 从 ![$t = 0$](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_eq05490811019241878006.png) 秒到 ![$t = 40$](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_eq03650817669656050065.png) 秒，锅炉关闭，并且 LED 信号每隔 5 秒就在 0 和 1 之间交替一次。
- 从 ![$t = 40$](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_eq03650817669656050065.png) 秒到 ![$t = 60$](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_eq01915627481382864964.png) 秒，锅炉开启，并且 LED 信号每隔一秒就在 0 和 2 之间交替一次。
- 从 ![$t = 60$](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_eq01915627481382864964.png) 秒到 ![$t = 100$](https://ww2.mathworks.cn/help/examples/stateflow/win64/TemporalLogicGetStartedExample_eq09033346722860809468.png) 秒，锅炉关闭，并且 LED 信号每隔 5 秒就在 0 和 1 之间交替一次。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsf_boiler-sdi-led.png)



### 探索示例

使用其他时序逻辑来研究随着锅炉温度接近参考设定值，Bang-Bang 循环的时序如何变化。

1.输入调用 `elapsed` 和 `duration` 运算符的新状态动作：

- 在 `On` 状态下，将 `Timer1` 设置为 `On` 状态被激活的时间长度：

```
  en,du,ex: Timer1 = elapsed(sec);
```

- 在 `Off` 状态下，将 `Timer2` 设置为锅炉温度等于或高于参考设定值的时间长度：

```
  en,du,ex: Timer2 = duration(temp>=reference);
```

2.在**符号**窗格中，点击**解析未定义的符号**。Stateflow 编辑器将符号 `Timer1` 和 `Timer2` 解析为输出数据。

3.对 `Timer1` 和 `Timer2` 启用记录。在**符号**窗格中，选择每个符号。然后，在**属性检查器**中的**记录**下，选择**记录信号数据**。

4.在**仿真**选项卡中，点击**运行**。

5.在**仿真**选项卡中的**查看结果**下，点击**数据检查器**。

6.在仿真数据检查器中，在同一坐标区中显示信号 `boiler` 和 `Timer1`。绘图表明：

- 通常情况下，在锅炉冷时 Bang-Bang 循环的 `On` 阶段持续 20 秒，在锅炉热时持续 4 秒。
- 在锅炉第一次达到参考温度时，循环会提前中断，控制器在 `On` 状态仅保持 18 秒。
- 在锅炉变热后，第一个循环的时间比随后的循环时间略短，在该循环中控制器在 `On` 状态仅保持了 3 秒。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsf_boiler-sdi-timer1.png)



7.在仿真数据检查器中，在同一坐标区中显示信号 `boiler` 和 `Timer2`。绘图表明：

- 在锅炉变热后，Bang-Bang 循环的 `Off` 阶段通常需要 9 秒才能冷却。
- 锅炉第一次达到参考温度时，需要 19 秒来冷却，这是其他循环冷却时间的两倍多。



![img](https://ww2.mathworks.cn/help/examples/stateflow/win64/xxsf_boiler-sdi-timer2.png)



较短的循环和较长的冷却时间是由于 `On` 状态内的子状态层次结构导致的。当锅炉首次达到参考温度时，从 `HIGH` 到 `NORM` 的转移会使控制器在一个额外的时间步内保持开启，导致锅炉温度超过正常值。而在后面的循环中，历史结点导致 `On` 阶段以激活的 `NORM` 子状态开始。然后，控制器在锅炉达到参考温度后立即关闭，从而冷却锅炉。