# 参考链接

[Simulink 快速入门 - MathWorks 中国](https://ww2.mathworks.cn/help/simulink/getting-started-with-simulink.html?s_tid=CRUX_lftnav)

# Simulink

Simulink 是 MATLAB 环境中的一个扩展工具，用于多领域动态系统的建模、仿真和分析。它提供了一个图形化的用户界面，使用户能够通过拖放各种模块和连接它们来构建模型。Simulink 特别适用于以下几个领域：

1. **控制系统设计**：用户可以设计和仿真复杂的控制系统，包括 PID 控制、模糊控制、鲁棒控制等。
2. **信号处理**：Simulink 提供了广泛的信号处理工具箱，可以用于滤波、调制、解调等操作。
3. **通信系统**：可以建模和仿真通信系统，包括发射机、接收机、调制解调器等。
4. **物理系统建模**：可以建模机械、电气、热力、液压等物理系统，通过物理建模工具箱（如 Simscape）进行仿真。
5. **嵌入式系统**：可以设计嵌入式控制器，生成 C/C++ 代码，并将其部署到嵌入式硬件上。

Simulink 的主要功能包括：

1. **图形化建模**：通过图形化界面，用户可以拖放和连接各种模块来构建系统模型。
2. **仿真**：可以对构建的模型进行仿真，分析系统的动态行为。
3. **模块化设计**：支持模块化设计，可以将复杂系统分解为多个子系统，进行分层次建模。
4. **自动代码生成**：可以自动生成 C、C++ 或 HDL 代码，方便在嵌入式系统中实现和部署。
5. **与 MATLAB 的集成**：Simulink 和 MATLAB 无缝集成，可以使用 MATLAB 脚本控制 Simulink 仿真，或者在 Simulink 模型中调用 MATLAB 函数。。

# Simulink 模块图

Simulink® 是动态系统的图形建模和仿真环境。可以创建模块图，用模块表示系统的各个组成部分。模块可以表示物理组件、小型系统或函数。输入/输出关系则完整描述了模块特征。请思考下面这些示例：

- 一个水龙头往一个水桶里注入水 - 水以一定的流速进入水桶，水桶变重。模块可以表示水桶，水的流速为输入，水桶的重量为输出。
- 您用扩音器传递声音 - 扩音器一端产生的声音在另一端被放大。扩音器是模块，输入是声源的声波，输出是您听到的声波。
- 您推动购物车使它移动 - 购物车是模块，您施加的力是输入，购物车的位置是输出。

只有定义了输入和输出，模块的定义才算完成，并且此模型定义任务需与建模目的相关。例如，如果建模目的不涉及购物车的位置，则会自然选择购物车的速度作为输出。

Simulink 提供了一些模块库，它们是按功能分组的模块集合。例如，要对以常量倍数放大输入的扩音器进行建模，可以使用 Math Operations 库中的 Gain 模块。

![Gain block](https://ww2.mathworks.cn/help/simulink/gs/gain_block.png)

进入扩音器的声波作为输入，出来的同一声波的更大版本作为输出。

**>** 符号表示模块的输入和输出，可以连接到其他模块。

![Sine waves appear to the right and to the left of a Gain block with a value of 2.](https://ww2.mathworks.cn/help/simulink/gs/simple_amplify.png)

您可以将模块连接到其他模块以构成系统，从而表示更复杂的功能。例如，音频播放器可将数字文件转换为声音。软件从存储中读取数字表示，以数学方式对其进行解释，然后将其变为物理声音。处理数字文件以计算声音波形的软件可以是一个模块，接收波形并将其转换为声音的扬声器可以是另一个模块。生成输入的组件又是另一个模块。

要在 Simulink 中对扩音器的正弦波输入进行建模，需要包含 Sine Wave 源。

![A Sine Wave block connects to a Gain block.](https://ww2.mathworks.cn/help/simulink/gs/amplify_simple.png)

Simulink 的主要功能是对系统各个组件随时间流逝的行为变化进行仿真。简单来讲就是：采用一个时钟，按时间确定各个模块的仿真顺序，并在仿真过程中依次将在上一个模块图中计算得出的输出传播到下一个模块，直至最后一个模块。以扩音器为例。在每个时间步，Simulink 都必须计算正弦波的值，将其传播给扩音器，然后计算输出值。

![The time t=1. Sine waves appear to the left and right of a Gain block with a value of 2. The portion of the sine wave that has been traversed when the simulation time reaches t=1 is highlighted in blue.](https://ww2.mathworks.cn/help/simulink/gs/simple_compute.png)

在每个时间步，每个模块都要根据输入计算输出。当在一个给定时间步计算完图中的所有信号后，Simulink 将基于模型配置和数值求解器算法确定下一个时间步，并向前移动仿真时钟。接下来，每个模块将为这个新的时间步计算输出。

![The time is t=2. The portion of the sine wave that has been traversed when the simulation time reaches t=2 is highlighted in blue.](https://ww2.mathworks.cn/help/simulink/gs/simple_advance.png)

在仿真中，时间的移动与真实时钟不同。完成每个时间步的计算需要多长时间，该时间步就会花费多长时间，而不管它代表几分之一秒还是几年。

通常，组件的输入对其输出的影响不是瞬时的。例如，打开加热器不会导致温度立即发生变化。该动作为微分方程提供输入。历史温度（一个*状态*）也是一个输入因子。当仿真需要求解微分方程或差分方程时，Simulink 使用内存和数值求解器来计算时间步的状态值。

Simulink 处理三类数据：

- 信号 - 在仿真期间计算的模块输入和输出
- 状态 - 在仿真期间中计算的代表模块动态的内部值
- 参数 - 影响模块行为的值，由用户控制

在每个时间步，Simulink 都计算信号和状态的新值。相比之下，您可以在编译模型时指定参数，并且可以在仿真运行时偶尔更改它们。

# 创建简单的模型

## 问题描述

您可以使用 Simulink® 来对系统建模，然后仿真该系统的动态行为。本教程中创建简单模型所使用的基本方法也适用于创建更复杂的模型。此示例对简化的汽车运动进行仿真。当踩下油门踏板时，汽车通常处于行进状态。松开踏板后，汽车怠速并停下来。

当初速度为0时，位移公式为
$$
s=\frac{1}{2} \times at^2
$$
Simulink 模块是定义模块输入和模块输出之间数学关系的模型元素。要创建这个简单模型，您需要四个 Simulink 模块。

| 模块名称                | 模块目的                   | 模型目的                               |
| :---------------------- | :------------------------- | :------------------------------------- |
| Pulse Generator         | 为模型生成输入信号         | 表示加速踏板                           |
| Gain                    | 将输入信号乘以常量值       | 计算踩下加速踏板后如何影响汽车的加速度 |
| Second-Order Integrator | 将输入信号积分两次         | 根据加速度计算汽车位置                 |
| Outport                 | 指定一个信号作为模型的输出 | **指定汽车位置做为模型的输出**         |

那么二次积分器的的输入和输出为
$$
\frac {d^2s}{dt^2} = a
$$


![A Simulink model with Pulse Generator, Gain, Second-Order Integrator block, and two Outport blocks.](https://ww2.mathworks.cn/help/simulink/gs/car_completed_model.png)

此模型的仿真过程是将一个简短的脉冲信号积分两次，形成一个斜坡。结果显示在一个示波器窗口中。输入脉冲表示是否踩下油门踏板 - 1 表示踩下，0 表示未踩下。输出斜坡表示与起点之间的距离增加。

## 打开新模型

使用 Simulink 编辑器构建模型。

1. 启动 MATLAB®。从 MATLAB 工具条中，点击 **Simulink** ![img](https://ww2.mathworks.cn/help/simulink/gs/icon_simulink.png)。

   ![The Simulink start page has two tabs, New and Examples, from which you can open a new template or an example.](https://ww2.mathworks.cn/help/simulink/gs/simple_simulink_splash02.png)

2. 点击**空白模型**模板。

   Simulink 编辑器打开。

   为了避免遮蔽，Simulink 编辑器检查路径中加载的模型和文件，并使用下一个可用名称 `untitled`、`untitled1`、`untitled2` 等创建一个模型。

   ![Empty Simulink Editor](https://ww2.mathworks.cn/help/simulink/gs/simple_untitled.png)

3. 从**仿真**选项卡中，选择**保存 > 另存为**。在**文件名**文本框中，输入您的模型的名称。例如，`simple_model`。点击**保存**。模型使用文件扩展名 `.slx` 进行保存。

## 打开 Simulink库浏览器

Simulink 在 [Library Browser](https://ww2.mathworks.cn/help/simulink/slref/librarybrowser.html) 中提供了一系列按功能分类的模块库。下面是大多数工作流常用的一些模块库：

- Continuous - 表示具有连续状态的系统的模块
- Discrete - 表示具有离散状态的系统的模块
- Math Operations - 实现代数和逻辑方程的模块
- Sinks - 存储并显示所连接信号的模块
- Sources - 生成模型的驱动信号值的模块



要打开库浏览器，请在 Simulink 工具条的**仿真**标签页上，点击**库浏览器**。

![Library Browser](https://ww2.mathworks.cn/help/simulink/gs/docked-library-browser-annotated.png)

要浏览模块库，请在库树中展开一个类别，然后展开一个功能区。

要搜索所有可用的模块库，请输入搜索词。

例如，查找 Pulse Generator 模块。在搜索框中，输入 `pulse`，然后按 **Enter** 键。软件在库中搜索名称或描述中包含 `pulse` 的模块，然后在库浏览器的**搜索结果**标签页上显示这些模块。

**提示**

您可以通过点击**库选项卡**返回浏览库树。

![The Simulink Library Browser displays the results for the search term pulse, which include Pulse Generator and Continuous Pulse Generator blocks.](https://ww2.mathworks.cn/help/simulink/gs/docked-library-browser-search.png)

获取模块的详细信息。在**搜索结果**选项卡上，右键点击 Pulse Generator 模块，然后选择 **Pulse Generator 模块的帮助**。帮助浏览器随即打开并显示该模块的参考页。

模块通常有几个参数。您可以通过双击该模块来访问所有模块参数。

## 将模块添加到模型

要开始构建模型，请向模型画布添加模块。您可以使用库浏览器或快速插入菜单添加模块。

1. 首先从库浏览器添加一个 Pulse Generator 模块。

   从 Sources 库中，将 Pulse Generator 模块拖到 Simulink 编辑器中。您的模型中将出现 Pulse Generator 模块的副本，还有一个文本框用于输入**振幅**参数的值。输入 `1`。

   ![The Pulse Generator block with a value of 1 entered in the text box prompt to specify the Amplitude parameter.](https://ww2.mathworks.cn/help/simulink/gs/simple_sine_block1.png)

   参数值在整个仿真期间保持不变。

2. 使用快速插入菜单添加一个 Gain 模块。

   双击模型画布中的任意位置。在出现的快速插入菜单中，输入 `out`。将出现模块列表。对于列表中的每个模块，包含该模块的库的名称显示在模块名称下方。通过按 **Enter** 键选择存储在 Simulink 库中的 Out1 模块。

   ![Quick insert menu with the keyword "out" displayed in the search text box](https://ww2.mathworks.cn/help/simulink/gs/quick-insert-menu.png)

   有关快速插入菜单的详细信息，请参阅[Add Blocks to Models Using Quick Insert Menu](https://ww2.mathworks.cn/help/simulink/ug/add-blocks-to-models.html#mw_77b32c25-3d51-479b-9e71-db0c873401be)。

3. 使用库浏览器或快速插入菜单将这些模块添加到模型中。

   

   | 模块                    | 库                       | 参数          |
   | :---------------------- | :----------------------- | :------------ |
   | Gain                    | Simulink/Math Operations | 增益: `2`     |
   | Second-Order Integrator | Simulink/Continuous      | 初始条件: `0` |

   

   复制现有 Outport 模块，然后使用键盘快捷方式将其粘贴到另一个点，从而添加第二个 Outport 模块。

   您的模型现在已经包含您需要的模块。

4. 通过点击并拖动每个模块来排列模块。要调整模块大小，请拖动模块的一个角。

   ![Arranged blocks from left to right.](https://ww2.mathworks.cn/help/simulink/gs/simple_unconnected_blocks.png)



## 连接模块

通过在输出端口和输入端口之间创建线条来连接模块。

1. 点击 Pulse Generator 模块右侧的输出端口。

   该输出端口和所有适合连接的输入端口由蓝色向前符号 ![port hint symbol](https://ww2.mathworks.cn/help/simulink/gs/porthint_symbol.png) 指示。

   ![Five unconnected blocks include, from left to right, a Pulse Generator block, a Gain block, a Second Order Integrator block, and two Outport blocks. The output port of the Pulse Generator block and the input ports of the other blocks display blue chevron port hint symbols.](https://ww2.mathworks.cn/help/simulink/gs/simple_port_connectioncues.png)

   

2. 指向 ![port hint symbol](https://ww2.mathworks.cn/help/simulink/gs/porthint_symbol.png) 以查看连接提示。

   ![A connection cue is displayed between the Pulse Generator block and the Gain block.](https://ww2.mathworks.cn/help/simulink/gs/simple_port_connectioncues_line.png)

   点击提示，用一个线条和一个指示信号流动方向的箭头连接这些模块。

   ![An arrow represents the connection between the Pulse Generator block and the Gain block.](https://ww2.mathworks.cn/help/simulink/gs/simple_connected_line.png)

   

3. 将 Gain block 的输出端口连接到 Second-Order Integrator 模块的输入端口。

4. 将 Second-Order Integrator 模块的两个输出连接到两个 Outport 模块。

5. 保存模型。在**仿真**选项卡中，点击**保存**。

   

   ![All the blocks are connected.](https://ww2.mathworks.cn/help/simulink/gs/simple_connections_complete.png)

   

## 添加信号查看器

要查看仿真结果，请将第一个输出连接到一个 Signal Viewer。

点击信号。在**仿真**选项卡中的**准备**下，点击**添加查看器**。选择 **Scope**。信号上会出现查看器图标，并打开一个示波器窗口。

![A viewer icon appears on the signal between the Second-Order Integrator block and Outport block for output 1.](https://ww2.mathworks.cn/help/simulink/gs/simple_scope_added.png)

您可以随时通过双击该图标打开示波器。

我们可以使用多个信号查看器查看各模块的输出波形，用于理解各个模块的功能。

## 运行仿真

指定仿真的停止时间。然后，对模型进行仿真。

1. 在**仿真**选项卡上，设置仿真停止时间。在 Simulink 工具条的**仿真**选项卡上，在**停止时间**字段中输入值。

   ![Simulation stop time is displayed on the Simulation tab of the toolstrip](https://ww2.mathworks.cn/help/simulink/gs/toolbar_stoptime.png)

   默认停止时间 `10.0` 适合此模型。此时间值没有单位。Simulink 中的时间单位取决于方程的构造方式。此示例对简化的汽车运动进行 10 秒的仿真 - 其他模型的时间单位可以是毫秒或年。

2. 要运行仿真，请点击**运行** ![img](https://ww2.mathworks.cn/help/simulink/gs/start_button_ue.png)。



仿真开始运行并在波形查看器中生成输出。

![A scope viewer shows the output of the simulation.](https://ww2.mathworks.cn/help/simulink/gs/simple_viewer_afterrun.png)

## 优化模型

此示例使用现有模型 `moving_car.slx`，并基于此运动模型对接近传感器建模。

在这种情况下，数字传感器用于测量汽车与 10 米（30 英尺）外的障碍物之间的距离。模型基于下列条件来输出传感器的测量值和汽车的位置值：

- 汽车在到达障碍物时会紧急刹车。
- 在现实世界中，传感器对距离的测量不够精确，从而导致随机数值误差。
- 数字传感器以固定时间间隔运行。

### 更改模块参数

要开始，请打开 `moving_car` 模型。在 MATLAB 命令行中，输入：

```
open_system('moving_car.slx');
```

您首先需要对在汽车位置到达 `10` 时的紧急刹车进行建模。Integrator, Second-Order 模块有用于此目的的参数。

1. 双击 Integrator, Second-Order 模块。将出现“模块参数”对话框。
2. 选择 **x 限制**，然后为 **x 上限**输入 `10`。参数的背景色发生变化以指示模型存在未应用的修改。点击**确定**应用更改并关闭对话框。

### 添加新模块和连接

添加一个用来测量与障碍物之间距离的传感器。

1. 修改模型。根据需要展开模型窗口，以容纳新模块。

   - 求实际距离。要想求出障碍物位置和车辆位置之间的距离，需要从 **Math Operations** 库中添加 Subtract 模块。还要从 **Sources** 库中添加 Constant 模块来为障碍物的位置设置常量值 `10`。
   - 对真实传感器中常见的不完美测量进行建模。使用 **Sources** 库中的 Band-Limited White Noise 模块产生噪声。将**噪声功率**参数设置为 `0.001`。通过使用 **Math Operations** 库中的 Add 模块将噪声添加到测量中。
   - 对每 0.1 秒触发一次的数字传感器进行建模。在 Simulink 中，以给定时间间隔对信号进行采样需要一个样本和保持器。从 **Discrete** 库中添加 Zero-Order Hold 模块。将该模块添加到模型后，将**采样时间**参数更改为 `0.1`。
   - 添加另一个 Outport，用来连接传感器输出。保留**端口号**参数的默认值。

2. 连接新模块。Second-Order Integrator 模块的输出已连接到另一个端口。要在该信号中创建分支，请左键点击该信号以突出显示可供连接的端口，然后点击适当的端口。

   ![With the signal between the Second-Order Integrator block and the Outport block selected, the software suggests connecting the signal line to the minus input port of the Subtract block.](https://ww2.mathworks.cn/help/simulink/gs/branch_signal.png)



### 为信号添加注释

将信号名称添加到模型中。

1. 双击信号并键入信号名称。

   ![Highlighted signal is called pedal position](https://ww2.mathworks.cn/help/simulink/gs/add_signal_name_complete.png)

2. 要完成输入，请点击文本框外部。

3. 重复上述步骤以添加下图中所示的名称。

   ![Model with named signals. The signal between Pulse Generator block and the gain block is called pedal position. The signal between the Gain block and the Integrator, Second-Order block is called acceleration. The signal between two Subtract blocks is called actual distance. The signal between the Zero-Order Hold block and the Outport block is called measured distance.](https://ww2.mathworks.cn/help/simulink/gs/model_with_labels.png)

### 比较多个信号

将 `actual distance` 信号与 `measured distance` 信号进行比较。

1. 创建 Scope Viewer 并将其连接到 `actual distance` 信号。右键点击信号并选择**创建并连接查看器 > Simulink > Scope**。信号的名称显示在查看器标题中。

2. 将 `measured distance` 信号添加到同一个查看器中。右键点击信号，然后选择**连接到查看器 > Scope1**。确保您连接到在上一步中创建的查看器。

   ![Window after right clicking a signal. Scope 1 is selected.](https://ww2.mathworks.cn/help/simulink/gs/add_signal_to_viewer.png)

3. 对模型进行仿真。波形查看器显示两个信号：`actual distance`（黄色）和 `measured distance`（蓝色）。

   ![The scope viewer shows the actual distance and the measured distance values.](https://ww2.mathworks.cn/help/simulink/gs/signal_comparison.png)

4. 放大图形以观察噪声和采样的影响。点击**缩放** ![img](https://ww2.mathworks.cn/help/simulink/gs/zoom.png)。左键点击并拖动鼠标框住您想放大查看的区域。

   ![A rectangular box in the scope viewer window shows the region selected to zoom.](https://ww2.mathworks.cn/help/simulink/gs/select_zoom.png)

   您可以根据需要重复此操作来观察细节。

   ![The zoomed region in the scope viewer](https://ww2.mathworks.cn/help/simulink/gs/zoom_complete.png)

该图显示，测量值可偏离实际值达 0.3 米之多。这些信息在设计安全功能（如碰撞预警）时非常有用。

# 模型层次结构

Simulink® 模型可以组织成具有层次结构的组件。在分层模型中，您可以选择查看整体系统，或沿模型层次结构向下导航以逐级深入模型细节。

## 查看模型层次结构

要开始，请打开 `smart_braking` 模型。

```shell
openExample('simulink/NavigateThroughModelHierarchyExample')
```

在模型中：

- 当踩下油门踏板时，汽车移动。
- 接近传感器将测量车辆和障碍物之间的距离。
- 警报系统基于该接近度生成警报。
- 警报系统自动控制刹车以防止碰撞。

当您构建模型时，您可以将模块连接起来，以对表示系统动态的复杂组件进行建模。在此模型中，Vehicle、Proximity sensor 和 Alert system 都是包含多个模块的复杂组件，它们都存在于一个子系统层次结构中。要查看子系统的内容，请双击该子系统。

要查看完整模型层次结构的表示，请打开模型浏览器。

1. 垂直展开模型窗口，直到在 Simulink 编辑器的左下角可以看到“隐藏/显示模型浏览器”按钮。
2. 点击“隐藏/显示模型浏览器”按钮。



![img](https://ww2.mathworks.cn/help/examples/simulink/win64/xxmodel_browser.png)



模型浏览器显示，您在顶层查看的所有子系统都有自己的子系统。展开每个子系统节点以查看它包含的子系统。您可以在模型浏览器中导航层次结构。例如，展开 Proximity sensor 节点，然后选择 Sensor model 子系统。



![img](https://ww2.mathworks.cn/help/examples/simulink/win64/xxbrowse_subsystem.png)



地址栏会显示您正在查看哪个子系统。要在单独的窗口中打开子系统，请右键点击该子系统，然后选择**在新窗口中打开**。

子系统上的每个输入端口或输出端口在子系统内都有对应的 Inport 或 Outport 模块。这些模块表示子系统与父系统之间的数据传输。当一个系统包含多个输入或输出端口时，Inport 或 Outport 模块上的数字表示该端口在子系统接口上的位置。

## 查看信号属性

Simulink 中的信号线表示从模块到模块的数据传输。信号具有与其在模型中的函数对应的属性：

- 维度 - 标量、向量或矩阵
- 数据类型 - 字符串、双精度、无符号整数等
- 采样时间 - 信号产生更新值的固定时间间隔（对于连续采样，则为 `0`）

要显示一个模型中所有信号的数据类型，请在**调试**选项卡的**叠加信息**下，点击**基本数据类型**。

![The top level view of the model smart_breaking has an annotation that indicates the data type on each signal line.](https://ww2.mathworks.cn/help/simulink/gs/display_datatypes.png)

模型将在信号线旁显示数据类型。大多数信号均为双精度，只有名为 `Alert system` 的子系统的输出例外。双击子系统进行调查。

![The contents of the subsystem named Alert system show the signal data type annotations on the signal lines.](https://ww2.mathworks.cn/help/simulink/gs/alert_subsystem.png)

此子系统中的数据类型标签显示，数据类型在名为 `Alert device` 的子系统中发生改变。双击子系统进行调查。

![The contents of the subsystem named Alert device contains a Data Type Conversion block. The signal data type annotations indicate the data type for the input and output signals of the Data Type Conversion block.](https://ww2.mathworks.cn/help/simulink/gs/alert_device.png)

Alert device 组件将信号 `Alert index` 从双精度转换为整数。您可以在源头设置数据类型，也可以使用 Signal Attributes 库中的 Data Type Conversion 模块。双精度（默认数据类型）提供最佳数值精度，是所有模块都支持的数据类型。双精度数据类型占用的内存和计算能力也最多。要对内存和计算能力有限的嵌入式系统建模，可使用其他数值数据类型。

要显示采样时间，请在**调试**选项卡的**叠加信息**下，点击“采样时间”部分的**颜色**。模型会更新以将模型中的各个采样时间显示为不同颜色，并显示图例。

![Color coding in the top level view of the model smart_breaking indicates the sample time associated with each block, port, and signal line.](https://ww2.mathworks.cn/help/simulink/gs/sample_time_colors.png)

![The Timing Legend groups sample times based on type and indicates the sample time that corresponds to each color in the block diagram.](https://ww2.mathworks.cn/help/simulink/gs/sample_time_legend.png)

- 具有连续动态的模块或信号显示为黑色。具有连续采样时间的信号会根据求解器的要求频繁更新以满足指定的容差值。
- 保持不变的模块或信号显示为品红色。它们在仿真过程中保持不变。
- 以最低固定时间间隔更新的离散模块或信号显示为红色。具有离散采样时间的信号会以固定间隔更新。如果模型中包含具有不同固定采样时间的组件，则每个离散采样时间显示为不同的颜色。
- 同时包含离散和连续信号的多速率子系统显示为黄色。

## 跟踪信号

此模型有常量输入和离散输出。要确定采样方案从哪里发生改变，可以通过其信源模块跟踪各个模块的输出信号。

1. 要打开模型浏览器，请垂直展开模型窗口，直到“隐藏/显示模型浏览器”按钮 ![img](https://ww2.mathworks.cn/help/simulink/gs/icon_modelbrowser.png) 出现在 Simulink 编辑器的左下角。然后，点击“隐藏/显示模型浏览器”按钮。

2. 要开始跟踪输出信号的源，请选择该信号。然后，在**信号**选项卡中，点击**跟踪到源** ![img](https://ww2.mathworks.cn/help/simulink/gs/highlight_signal_source.png)。

   Simulink 编辑器进入信号跟踪模式。在信号跟踪模式下，模型画布变为灰色而不是白色，以帮助突出跟踪的路径。

   提示面板显示在信号跟踪模式下可以采取的操作的提示，以及对应于每项操作的键盘快捷方式。要最小化该提示面板，请按键盘上的 **?** 键。如果您要还原该提示面板，请按再次键盘上的 **?**。

   ![In the top level of the model smart_breaking, the output signal and the subsystem named Alert system, which produces the output signal, are highlighted yellow.](https://ww2.mathworks.cn/help/simulink/gs/source_highlight_1.png)

3. 要继续跟踪输出信号的源，请按向左箭头键。

   ![The Simulink Editor navigates inside the subsystem named Alert system to highlight the next source block that affects the value of the output signal.](https://ww2.mathworks.cn/help/simulink/gs/source_highlight_2.png)

4. 持续按向左箭头键以跟踪输出信号的源，直到到达名为 `Alert logic` 的子系统内的 Subtract 模块。当您到达信号路径中的 Subtract 模块时，您必须选择一条路径来继续跟踪，因为 Subtract 模块有两个输入端口。软件以蓝色突出显示下一个要跟踪的段，以指示所选路径。默认情况下，选择第一个输入端口进行持续跟踪。通过按向下箭头键选择负输入端口的路径。

   ![Inside the subsystem named Alert logic, the signal path is highlighted in yellow up to the Subtract block. The signal connected to the minus input port of the Subtract block is highlighted blue.](https://ww2.mathworks.cn/help/simulink/gs/source_highlight_3.png)

5. 要找到离散化的源，请继续按向左箭头并注意每个端口名称的颜色，它反映端口的采样时间。

   ![Inside the subsystem named Sensor model, the Zero Order Hold block and the Outport block named proximity sensor output are highlighted.](https://ww2.mathworks.cn/help/simulink/gs/source_highlight_5.png)

Sensor model 子系统中的 Zero-Order Hold 模块将信号从连续转换为离散。

# 使用Simulink进行基于模型的设计

## 步骤1：系统定义和布局

Simulink® 模型的顶层系统布局是许多工程团队可以使用的公共环境，是基于模型的设计范式：分析、设计、检验和实现。您可以通过确定模型的结构和各个组件来定义顶层系统。然后，您可以将模型按照层次结构进行组织，分别与系统的各个组件相对应。接下来，您可以定义每个组件的接口以及组件之间的连接。

本教程中讨论的模型是一个平板机器人，它可以借助两个轮子进行移动或旋转，类似于家用扫地机器人。此模型假设机器人以下列两种方式之一进行移动：

- 线性 - 两个轮子以相同的速度向同一方向转动，机器人线性移动。
- 旋转 - 两个轮子以相同的速度向相反方向转动，机器人原地旋转。

![A flat circular robot is shown from the top. The center of the circular robot perimeter is the origin of the polar coordinate system. The direction of motion is indicated by the angle theta, which is marked as about 30 degrees above the horizontal axis pointing right. The wheel axis is perpendicular to the direction of motion. The wheels are lined up parallel to the direction of motion.](https://ww2.mathworks.cn/help/simulink/gs/robot_coordinates.png)

这两种运动都从静止状态开始，也就是说，旋转速度和线性速度均为零。按照这些假设，您可以对线性运动组件和旋转运动组件分别建模。

### 确定建模目的

在设计模型之前，请明确您的目的和需求。目的决定模型的结构和详细程度。如果目的只是弄清楚机器人可以走多快，那么只对线性运动进行建模就足够了。如果目的是为设备设计一组输入，使它按照给定的路径移动，那就会涉及到旋转组件。如果目的是避障，那么系统就需要一个传感器。本教程的模型构建目的是设计一些传感器参数，使机器人能够在检测到路径中的障碍物时及时停止。要实现此目的，模型必须：

- 确定电机停止后机器人多长时间能停止下来
- 提供一系列线性运动和旋转运动的命令，使机器人能够在一个二维空间内移动

第一个建模目的是让您能够分析运动，以便设计传感器。第二个目的是让您能够对设计进行测试。

### 确定系统组件和接口

明确建模需求之后，即可开始确定系统的组件。确定顶层结构的各个组件及其关系有助于您以系统化方式构建比较复杂的模型。这些步骤是在您构建模型之前在 Simulink 外部进行的。

执行此任务需要回答以下问题：

- 系统需要哪些结构组件和功能组件？当布局能够反映物理结构和功能结构时，将有助于理解、构建、进行通信和测试系统。当需要在设计过程中的不同阶段实现系统的各个组成部分时，这一点将变得更加重要。
- 每个组件的输入和输出是什么？绘制一个显示各组件连接情况的图形。此图可帮助您可视化模型中的信号流，确定每个信号的信源和信宿，并确定是否所有必要的组件都存在。
- 需要达到多高的详细程度？在图中包括主要系统参数。绘制系统图可以帮助您识别并建模对要观察的行为非常重要的部件。实现建模目的所需要的每个组件和参数在模型中都要有一个对应的表示，但需要在模型的复杂程度和可读性之间进行权衡。建模可以是一个反复的过程。您可以从具有很少细节的简要模型开始，然后根据需要逐渐提高复杂程度。

思考以下问题通常也很有帮助：

- 系统的哪些组成部分需要测试？
- 测试数据和成功的标准是什么？
- 哪些输出是分析和设计任务所必需的？

#### 确定机器人运动组件

本教程中的系统定义了一个机器人，它通过两个电动轮子在两个维度上移动。其中包括：

- 线性运动特性
- 旋转运动特性
- 用于确定系统在两个维度上的位置的变换
- 用于测量机器人与障碍物之间的距离的传感器

![The block diagram lists these components from left to right: force inputs, left wheel, right wheel, rotation computation, coordinate transformation, sensor.](https://ww2.mathworks.cn/help/simulink/gs/system_layout.png)

此系统的模型包括两个相同的轮子、施加在轮子上的输入作用力、旋转动态特性、坐标变换和一个传感器。此模型使用 Subsystem 表示每个组件：

1. 打开一个新的 Simulink 模型。请参阅[打开新模型](https://ww2.mathworks.cn/help/simulink/gs/create-a-simple-model.html#bu3nd7o-1)。

2. 打开库浏览器。请参阅[打开 Simulink库浏览器](https://ww2.mathworks.cn/help/simulink/gs/create-a-simple-model.html#bt9ataa-1)。

3. 添加 Subsystem 模块。将五个 Subsystem 模块从 Ports & Subsystems 库拖放到新模型中。

4. 点击一个子系统。在**格式**选项卡中，点击**自动**下拉列表。清除**隐藏自动模块名称**复选框。

   ![The Auto drop-down list is expanded. The Hide Automatic Block Names check box is cleared.](https://ww2.mathworks.cn/help/simulink/gs/hide_automatic_block_names.png)

5. 按如下所示放置并重命名这五个 Subsystem 模块。要更改模块名称，请双击模块名称并编辑文本。

   

   ![Five Subsystem blocks have the names: Inputs, Left Wheel, Right Wheel, Rotation, and Coordinate Transformation.](https://ww2.mathworks.cn/help/simulink/gs/named_subsystems.png)

   

#### 定义组件之间的接口

确定子系统之间的输入和输出连接。输入和输出值在仿真过程中动态变化。模块之间的连接线代表数据传输。下表显示每个组件的输入和输出。

| 模块     | 输入               | 输出                 | 相关信息                   |
| :------- | :----------------- | :------------------- | :------------------------- |
| 输入     | 无                 | 右轮作用力左轮作用力 | 不适用                     |
| 右轮     | 右轮作用力         | 右轮速度             | 有方向性，负值表示相反方向 |
| 左轮     | 左轮作用力         | 左轮速度             | 有方向性，负值表示相反方向 |
| 旋转     | 左右轮之间的速度差 | 旋转角度             | 逆时针测量                 |
| 坐标变换 | 正常速度旋转角度   | X 速度Y 速度         | 不适用                     |
| 传感器   | X 坐标Y 坐标       | 无                   | 建模不需要任何模块。       |

一些模块输入与模块输出并不完全匹配。因此，除了各个组件的动态特性之外，模型还必须计算以下内容：

- 旋转计算的输入 - 两个轮子的速度相减并除以 2。
- 坐标变换的输入 - 两个轮子的平均速度。
- 传感器的输入 - 对坐标变换的输出进行积分。

轮子的速度大小始终相等，并且计算在该假设内是精确的。

添加必要的组件并完成连接：

1. 为每个子系统添加必要的输入端口和输出端口。双击 Subsystem 模块。

   

   ![An Inport block named In1 connects to an Outport block named Out1.](https://ww2.mathworks.cn/help/simulink/gs/model03_subsystems_template.png)

   

   每个新的 Subsystem 模块都包含一个 Inport (In1) 和一个 Outport (Out1) 模块。这些模块定义信号与模型层次结构中下一个更高层级进行交互的接口。

   每个 Inport 模块在 Subsystem 模块上创建一个输入端口，每个 Outport 模块创建一个输出端口。模型通过输入/输出端口名称反映这些模块的名称。为更多的输入和输出信号添加更多模块。在 Simulink 编辑器工具栏上，点击**向上导航到父级**按钮 ![img](https://ww2.mathworks.cn/help/simulink/gs/icon_up_to_parent.png) 返回顶层。

   对于每个模块，添加并重命名 Inport 和 Outport 模块。

   ![The five Subsystem blocks are shown with previews of the subsystem contents visible. In the Subsystem block named Inputs, Outport blocks named Left force and Right force are visible.](https://ww2.mathworks.cn/help/simulink/gs/subsystems_ports.png)

   在复制 Inport 模块以创建新模块时，请使用**粘贴** (Ctrl+V) 选项。

2. 根据左轮和右轮速度计算 Coordinate Transform 和 Rotation 子系统所需的输入。

   1. 计算 Coordinate Transform 子系统的线性速度输入。从 Math Operations 库中添加一个 Add 模块，并将两个轮子组件的输出连接起来。添加一个 Gain 模块，并将 Gain 参数设置为 `1/2`。将 Add 模块的输出连接到此 Gain 模块。

   2. 计算 Rotation 子系统的速度差输入。从 Math Operations 库中添加一个 Subtract 模块。将右轮速度连接到 **+** 输入，将左轮速度连接到 **-** 输入。连接两个轮子组件的输出。添加一个 Gain 模块，并将 Gain 参数设置为 `1/2`。将 Subtract 模块的输出连接到此 Gain 模块。

      ![The Subsystem block named Left Wheel is above the Subsystem block named Right Wheel. The input port of both blocks is named Force, and the output port is naemd Speed. Both output port connect to an Add block that connects to a Gain block with the value of 1/2. Both output ports also connect to a Subtract block that subtracts the speed of the left wheel from the speed of the right wheel and then connects to a Gain block with a value of 1/2.](https://ww2.mathworks.cn/help/simulink/gs/compute_inputs.png)

3. 根据 X 速度和 Y 速度计算 X 坐标和 Y 坐标。从 Continuous 库中添加两个 Integrator 模块，并连接 Coordinate Transform 模块的输出。将 Integrator 模块的初始条件设置保留为 `0`。

   ![The Subsystem named Coordinate Transform has two input ports, named Linear speed and Angle, and two output ports named X speed and Y speed. Both output ports connect to an Integrator block.](https://ww2.mathworks.cn/help/simulink/gs/integrators.png)

4. 完成系统连接。

   ![In the completed model, the Subsystem block named Subsystem connects to the Subsystem blocks named Left Wheel and Right Wheel. The wheel subsystems connect to an Add block that connects to a Gain block with the value of ½ that connects to the Subsystem block named Coordinate Transform. The wheel subsystems also connect to a Subtract block that subtracts the left wheel speed from the right wheel speed and connects to a Gain block with the value of ½. The Gain block connects to the Subsystem block named Rotation that connects to the Subsystem block named Coordinate Transform. Both of the output ports of the Coordinate Transform subsystem connect to an integrator block.](https://ww2.mathworks.cn/help/simulink/gs/connections_complete.png)

#### 参数和数据

确定模型需要的参数以及参数的值。根据建模目的决定这些值是始终固定，还是针对不同的仿真而更改。实现建模目的所需要的参数必须在模型中显式表示。下表可以帮助您确定对每个组件进行建模时的详细程度。

| 参数             | 模块     | 符号     | 值         | 类型 |
| :--------------- | :------- | :------- | :--------- | :--- |
| 质量             | 左轮右轮 | `m`      | 2.5 kg     | 可变 |
| 滚阻             | 左轮右轮 | `k_drag` | 30 Ns2/m   | 可变 |
| 机器人半径       | 旋转     | `r`      | 0.15 m     | 可变 |
| 初始角度         | 旋转     | 无       | 0 rad      | 固定 |
| 初始速度         | 左轮右轮 | 无       | 0 m/s0 m/s | 固定 |
| 初始 (X, Y) 坐标 | 积分器   | 无       | (0, 0) m   | 固定 |

Simulink 使用 MATLAB® 工作区来计算参数。可在 MATLAB 命令行窗口中设置这些参数：

```matlab
m = 2.5;
k_drag = 30;
r = 0.15;
```

## 步骤2：建模并验证系统

### 打开系统布局

对各个组件进行建模时，需要从大局上把握整个系统布局。首先加载布局模型。在 MATLAB® 命令行中，输入：

```
open_system('system_layout.slx')
```

![img](https://ww2.mathworks.cn/help/examples/simulink/win64/ModelTheComponentsExample_01.png)

### 对组件进行建模

包含一个组件的 Simulink® 模型基于以下几点：

- 物理组件的输出和输入之间的显式数学关系 - 您可以根据组件的输入通过代数计算和微分方程积分，直接或间接计算出组件的输出。例如，按照给定的进水速率计算水箱中的水位就是一种显式关系。每个 Simulink 模块基于从输入到输出的计算定义来执行。
- 物理组件的模型变量之间的隐式数学关系 - 由于变量之间相互依赖，因此为组件指定输入和输出并不容易。例如，电路中连接的电机的 `+` 极电压和 `-` 极电压之间就存在一种隐式关系。要在 Simulink 中对这种关系进行建模，您可以使用 Simscape™ 等物理建模工具，也可以将这些变量建模为允许定义输入/输出的更大组件的一部分。有时候，更仔细地审视建模目的和组件定义有助于定义输入/输出关系。
- 从实际系统获得的数据 - 您已经测得实际组件的输入/输出数据，但不存在完全定义的数学关系。许多设备具有符合此描述的未建模组件。例如，电视机散发的热量。您可以使用 System Identification Toolbox™ 来定义此类系统的输入/输出关系。
- 显式功能定义 - 您可以根据功能组件的输入通过代数计算和逻辑计算来定义功能组件的输出。例如，调温器的开关逻辑。您可以将大多数功能关系建模为 Simulink 模块和子系统。

本教程对具有显式输入/输出关系的物理组件和功能组件进行建模。在本教程中，您将：

1. 使用系统方程创建一个 Simulink 模型。
2. 在 Simulink 编辑器中添加并连接 Simulink 模块。模块代表方程中的系数和变量。
3. 分别为每个组件构建模型。构建系统模型最有效的方法是首先独立地考虑各个组件。
4. 首先，使用接近于系统的模型构建简单的模型。找出可能会影响模型准确性的假设条件。以迭代方式添加细节，直到复杂度满足建模和准确性要求。

#### 对物理组件进行建模

描述组件之间的关系，例如数据、能量和力的传递。在 Simulink 中使用系统方程构建系统的图形化模型。

为组件建模之前，需要思考以下问题：

- 每个组件的常量是什么？什么值不会更改，除非您更改它们？
- 每个组件的变量是什么？什么值会随着时间而更改？
- 一个组件有多少个状态变量？

根据科学原理推导出每个组件的方程。许多系统方程不外乎以下三种类别：

- 对于连续系统，微分方程描述变量的变化率，为所有时间值都定义方程。例如，一阶微分方程给出车速：
- 对于离散系统，差分方程描述变量的变化率，但只在特定时间定义方程。例如，来自离散比例微分控制器的控制信号：
- 没有导数的方程是代数方程。例如，用代数方程表示包含两个组件的并联电路中的总电流：

**轮子和线性运动.** 作用在轮子上的力有两个：

- 电机施加的力 - 此力 F 作用在速度变化的方向上，是轮子子系统的输入。
- 阻力 - 此力 Fdrag 作用在速度变化的相反方向上，是速度的函数。

加速度与这两个力之和成正比：

其中 kdrag 是阻力系数，m 是机器人的质量。每个轮子承载机器人一半的质量。

构建轮子模型：

1. 在 `system_layout` 模型中，双击名为 `Right Wheel` 的子系统以显示空子系统。

2. 要对速度和加速度进行建模，请添加一个 Integrator 模块。将初始条件设置保留为 `0`。此模块的输入是加速度 Vdot，输出是速度 V。

   ![The Integrator block has an unconnected input signal named Vdot and an output signal labeled V that is connected to an Outport block named Speed. An unconnected Inport block named Force is to the left of the unconnected signal line for the signal Vdot.](https://ww2.mathworks.cn/help/simulink/gs/model_wheel_add_integrator_block.png)

3. 要对阻力进行建模，请从用户定义的函数库中添加一个 MATLAB Function 模块。MATLAB Function 模块提供一种在模型中实现数学表达式的快速方法。要编辑函数，请双击该模块以打开 MATLAB Function 模块编辑器。

4. 在 MATLAB Function 模块编辑器中，输入 MATLAB® 代码以计算阻力。

   ```
   function Fdrag=get_fdrag(V,k_drag)
   
   Fdrag=k_drag*V*abs(V);
   ```

   ![The MATLAB Function Block Editor shows the code for a MATLAB function named Fdrag that calculates the drag force.](https://ww2.mathworks.cn/help/simulink/gs/matlab_function_block_fdrag.png)

5. 定义 MATLAB Function 模块的参量。在 MATLAB Function 模块编辑器中，点击“编辑数据”![img](https://ww2.mathworks.cn/help/simulink/gs/icon_edit_data.png)。点击 k_drag，将**作用域**设置为**参数**并点击**应用**。

6. 使用 Subtract 模块从电机的作用力中减去阻力。通过使用 Gain 模块并将**增益**参数指定为 `1/(m/2)`，完成力-加速度方程。

7. 要反转 MATLAB Function 模块的方向，请选择该模块。然后，在 Simulink 工具条中的**格式**选项卡上，点击“左右翻转”![img](https://ww2.mathworks.cn/help/simulink/gs/flip-left-right-button.png)。连接这些模块。

   ![In the wheel model, the MATLAB Function block is in a feedback loop between the speed output signal. The Subtract block subtracts the drag force calculated by the MATLAB Function block from the force input to the subsystem. The Subtract block and Gain block are between the force input and the input to the Integrator block.](https://ww2.mathworks.cn/help/simulink/gs/model_wheel.png)

8. 这两个轮子的动态特性相同。复制刚刚创建的名为 `Right Wheel` 的子系统，并将其粘贴到名为 `Left Wheel` 的子系统中。

9. 要查看模型的顶层，请点击“向上导航到父级”![img](https://ww2.mathworks.cn/help/simulink/gs/icon_up_to_parent.png)。

**旋转运动.**  当两个轮子沿相反方向转动时，它们沿半径为 *r* 的圆周运动，从而产生机器人的旋转运动。当这些轮子向相同方向转动时，没有旋转。假设轮子的速度大小始终相等，则可将旋转运动视为两个轮子速度 VR 与 VL 之差的因变量来对其进行建模：

构建 Rotation Dynamics 模型：

1. 在 `system_layout` 模型的顶层，双击名为 `Rotation` 的子系统以显示空子系统。删除 Inport 和 Outport 模块之间的连接。

2. 要对角速度和位置进行建模，请添加一个 Integrator 模块。将初始条件设置保留为 `0`。此模块的输出是旋转位置 theta，输入是角速度 theta_dot。

3. 根据切向速度计算角速度。添加一个 Gain 模块，并将该模块的**增益**参数设置为 `1/(2*r)`。

4. 连接这些模块。

   ![An Inport block named Speed difference connects to the Gain block, which connects to an Integrator block, which connects to an Outport block named Angle](https://ww2.mathworks.cn/help/simulink/gs/model_rotation.png)

5. 要查看模型的顶层，请点击“向上导航到父级”![img](https://ww2.mathworks.cn/help/simulink/gs/icon_up_to_parent.png)。

#### 对功能组件进行建模

通过一个函数从输入到输出的整个过程来描述功能。此描述可以包含代数方程和逻辑构造，您可以使用它们在 Simulink 中构建系统的图形化模型。

**坐标变换.** 机器人在 x 和 y 坐标上的速度 VX 和 VY 与线性速度 VN 和角度 theta 相关：

构建坐标变换模型：

1. 在 `system_layout` 模型的顶层，双击名为 `Coordinate Transform` 的子系统以显示空子系统。

2. 要对三角函数进行建模，请从 Math Operations 库中添加一个 SinCos 模块。

3. 要对乘法进行建模，请从 Math Operations 库中添加两个 Product 模块。

4. 连接这些模块。

   ![The coordinate transformation model has two inputs named Linear speed and Angle and two outputs named Y speed and X speed. The X and Y speeds are calculated by multiplying the sine and cosine of the angular position by the linear speed input.](https://ww2.mathworks.cn/help/simulink/gs/model_transform.png)

5. 要查看模型的顶层，请点击“向上导航到父级”![img](https://ww2.mathworks.cn/help/simulink/gs/icon_up_to_parent.png)。

#### 设置模型参数

您可以使用多个源来确定模型中参数的适当值，包括：

- 书面规范，如标准属性表或制造商的数据表
- 直接测量现有系统所得的测量值
- 基于系统输入/输出的估计值

此模型使用以下参数：

| 参数       | 符号     | 值       |
| :--------- | :------- | :------- |
| 质量       | `m`      | 2.5 kg   |
| 滚阻       | `k_drag` | 30 Ns2/m |
| 机器人半径 | `r`      | 0.15 m   |

Simulink 模型可以访问使用 MATLAB 工作区中的变量定义的参数值。通过在 MATLAB 命令行窗口中输入命令来定义以下变量。

```
m = 2.5;
k_drag = 30;
r = 0.15;
```

### 通过仿真来验证组件

通过提供输入并观察输出来验证组件。即使这样简单的验证也能指出改进模型的直接方法。此示例验证以下行为：

- 当向轮子连续施加力时，速度会增加，直到达到稳定状态的速度为止。
- 当两个轮子向相反方向转动时，旋转角度以恒定速率增加。

#### 验证轮子组件

为轮子组件创建并运行测试模型：

1. 创建一个新模型。在**仿真**选项卡中，点击**新建** ![img](https://ww2.mathworks.cn/help/simulink/gs/icon_newmodel.png)。将名为 `Right Wheel` 的 Subsystem 模块复制到新模型中。

2. 要创建一个测试输入，请从源代码库中添加一个 Step 模块，并将其连接到名为 `Right Wheel` 的子系统的输入。将**步长时间**参数设置保留为 `1`。

3. 为输出添加一个波形查看器。右键点击名为 `Right Wheel` 的 Subsystem 模块的输出端口，然后选择**创建并连接查看器 > Simulink > Scope**。

   ![A Step block connects to the input of a Subsystem block named Right Wheel. The input port of the subsystem is named Force, and the output port of the subsystem is named Speed. A scope icon next to the subsystem output port indicates that the output port is connected to a scope viewer..](https://ww2.mathworks.cn/help/simulink/gs/test_wheel.png)

4. 对模型进行仿真。在**仿真**选项卡中，点击**运行** ![img](https://ww2.mathworks.cn/help/simulink/gs/simulink_run_icon.png)。

   ![The scope viewer plots the output signal of the subsystem named Right Wheel.](https://ww2.mathworks.cn/help/simulink/gs/test_wheel_result.png)

仿真结果表明，该模型表现出的行为与预期大致相符。在步长时间处施加力之前，不会发生运动。施加力后，速度开始增加，当施加的力和阻力达到平衡后，速度将保持稳定。除验证外，此仿真还显示在给定的作用力下轮子的最大速度。

#### 验证旋转组件

为旋转组件创建并运行测试模型：

1. 要创建新模型，请点击“新建”![img](https://ww2.mathworks.cn/help/simulink/gs/icon_newmodel.png)。将名为 `Rotation` 的子系统模块复制到新模型中。

2. 要在新模型中创建一个测试输入，请从 Sources 库中添加一个 Step 模块。将**步长时间**参数设置保留为 `1`。将 Step 模块的输出连接到名为 `Rotation` 的子系统的输入。此输入表示当两个轮子沿相反方向旋转时的轮子速度之差。

3. 将波形查看器添加到名为 `Rotation` 的子系统的输出中。右键点击子系统的输出端口，然后选择**创建并连接查看器 > Simulink > Scope**。

   ![A Step block connects to the input of a Subsystem block named Rotation. The input port of the subsystem is named Speed difference, and the output port of the subsystem is named Angle. A scope icon next to the subsystem output port indicates that the output port is connected to a scope viewer.](https://ww2.mathworks.cn/help/simulink/gs/test_rotation.png)

4. 对模型进行仿真。在**仿真**选项卡中，点击**运行** ![img](https://ww2.mathworks.cn/help/simulink/gs/simulink_run_icon.png)。

   ![The scope viewer plots the output signal of the subsystem named Rotation.](https://ww2.mathworks.cn/help/simulink/gs/test_rotation_result.png)

此仿真显示，当两个轮子以相同速度向相反方向转动时，角度会稳定增加。您可以改进模型，使角度输出更容易解释。例如，您可以：

- 通过添加增益为 `180/pi` 的 Gain 模块，将输出信号单位从弧度转换为角度。
- 通过添加具有函数 `mod` 的 Math Function 模块，以 360 度为周期显示输出值。

MATLAB 三角函数采用弧度输入。

### 验证模型

验证单个组件后，您可以对整个模型进行类似的验证。此示例验证以下行为：

- 当沿相同方向对两个轮子施加相同的力时，机器人沿直线运动。
- 当沿相反方向对两个轮子施加相同的力时，机器人原地旋转。

1. 在 `system_layout` 模型中，双击名为 `Inputs` 的子系统以显示空子系统。

2. 通过添加 Step 模块创建测试输入。将**步长时间**参数设置保留为 `1`。将 Step 模块输出连接到两个 Outport 模块。

   ![A Step block connects to two Outport blocks, one named Left force and one named Right force](https://ww2.mathworks.cn/help/simulink/gs/test_integrate_1.png)

3. 在模型的顶层，将两个输出信号连接到同一个波形查看器。

   ![The Coordinate Transform subsystem has two output ports named X speed and Y speed. Each output port connects to an Integrator block, and the outport of each Integrator block connects to a scope viewer.](https://ww2.mathworks.cn/help/simulink/gs/test_integrate_view.png)

4. 对模型进行仿真。

   ![The scope viewer plots the two output signals on the same graph, one in yellow and one in blue.](https://ww2.mathworks.cn/help/simulink/gs/test_integrate_1_result.png)

   波形查看器中的黄线是 X 方向，蓝线是 Y 方向。由于角度为零并且保持不变，因此机器人只在 X 方向上移动，跟预期一样。

5. 双击名为 `Inputs` 的子系统。要反转左轮的方向，请在源和第二个输出之间添加一个 Gain 模块，并将**增益**参数设置为 `-1`。

   ![The Step block connects directly to the Outport block named Right Force. The Gain block with a gain of -1 inverts the Step block output. The output from the Gain block connects to the Outport block named Left force.](https://ww2.mathworks.cn/help/simulink/gs/test_integrate_2.png)

6. 为角度输出添加一个波形查看器。

7. 对模型进行仿真。

连接到 x 和 y 速度信号的波形查看器显示 X-Y 平面上没有发生运动。

![The scope viewer plots the X and Y speed signals on the same graph.](https://ww2.mathworks.cn/help/simulink/gs/test_integrate_2_result.png)

连接到角度信号的波形查看器显示稳定的角运动。

![The scope viewer plots the angle output.](https://ww2.mathworks.cn/help/simulink/gs/test_integrate_2_result2.png)

通过更改输入，您可以使用这个最终模型来回答有关模型的许多问题。

- 当初始角度非零时会发生什么？
- 当作用力下降到零时，运动需要多长时间才能停止？
- 当机器人更重时会发生什么？
- 当机器人在阻力系数更小的更平滑曲面上移动时会发生什么？

## 步骤3：在simulink中设计系统

### 打开系统模型

此模型是一个平板机器人，它可以借助两个轮子进行移动或旋转，类似于家用扫地机器人。通过在 MATLAB® 命令行中输入以下命令打开模型：

```shell
open_system('system_model.slx')
```

![img](https://ww2.mathworks.cn/help/examples/simulink/win64/SystemModelExample_01.png)

本教程将分析此系统并为其添加功能。

### 确定设计的组件和设计目的

设定目标的说明是完成设计任务的关键第一步。即使是一个简单的系统，也可能存在多个甚至是相互矛盾的设计目的。对于示例模型，请考虑以下目标：

- 设计一个控制器，它可以改变作用力输入，使轮子按所需的速度转动。
- 设计输入，使设备沿预定的路径移动。
- 设计一个传感器和一个控制器，使设备沿直线移动。
- 设计一种规划算法，使设备沿可能的最短路径到达某个点并避开障碍物。
- 设计一个传感器和一种算法，使设备在某个区域内移动并避开障碍物。

本教程将设计一个警报系统。您需要确定用于测量障碍物距离的传感器的参数。完美的传感器精确地测量它与障碍物的距离。警报系统按固定时间间隔对这些测量值进行采样，使输出与测量值相差始终不超过 0.05 米。系统会及时生成警报，让机器人在碰到障碍物前停下来。

### 通过仿真来分析系统行为

设计新组件需要分析机器人的线性运动以确定：

- 如果切断轮子的电源，机器人能够以最高速度行驶多远
- 机器人的最高速度

使用能够使机器人开始运动的作用力输入信号来仿真模型，等到机器人达到稳定速度后，将输入作用力设置为零：

1. 在模型中，双击名为 `Inputs` 的子系统。

2. 删除现有阶跃输入，并添加一个 Pulse Generator 模块。

3. 为 Pulse Generator 模块设置以下参数：

   - 振幅：`1`
   - 周期：`20`
   - 脉冲宽度：`15`

   采用以上参数值是为了确保达到最高速度。您可以更改参数以查看其效果。

4. 对模型进行 20 秒的仿真。

   要分析仿真结果，请查看连接到模型中浮动示波器的信号。

   第一个示波器显示，在仿真时间 `3` 秒处，机器人的速度在表示输入力的脉冲降至零后迅速降低。速度渐近于零，但永远不会达到零。没有外力的低速动态的精确建模需要更复杂的系统表示形式。然而，对于此处的目标，此系统的近似表示已经足够。

   ![The scope shows the speed of the robot over the 20 second simulation.](https://ww2.mathworks.cn/help/simulink/gs/velocity.png)

   第二个示波器显示仿真期间机器人的位置。开始时，位置变化更快。经过大约 `3` 秒的仿真时间，随着机器人速度的降低，位置变化变得更缓慢。

   ![The scope shows the position of the robot over the 20 second simulation.](https://ww2.mathworks.cn/help/simulink/gs/position.png)

放大显示机器人位置的示波器图。在时间 `3` 处，机器人的位置大约在 `0.55` 米处。在仿真结束时，机器人的位置大约在 `0.7` 米处。由于机器人的速度在仿真结束时非常接近于零，因此结果显示机器人在外力降至零后移动不到 `0.16` 米。

![The scope shows a closer view of the robot position over the 20 second simulation. The maximum value on the y-axis is 0.7.](https://ww2.mathworks.cn/help/simulink/gs/position_zoom.png)

要求出最高速度，请执行以下操作：

1. 放大速度信号在时间上的平台区，即从 1 秒到 3 秒处。

2. 再次点击缩放按钮，退出缩放模式。

3. 点击游标测量值 ![img](https://ww2.mathworks.cn/help/simulink/gs/icon_cursor.png)。

4. 将第二个游标放在速度曲线平坦的区域。

   ![The scope shows a closely zoomed view of the robot speed in the part of the simulation where the pulse signal is high. The cursor measurements panel is shown in the Scope window to the right of the plot.](https://ww2.mathworks.cn/help/simulink/gs/viewer_measure.png)

**游标测量**面板中的**值**列表明机器人的最高速度为 `0.183` 米/秒。要计算机器人行驶 0.05 米所需的时间，用 0.05 米除以 0.183 米/秒，得到结果为 0.27 秒。

### 设计组件并验证设计

传感器设计包括以下组件：

- 机器人与障碍物之间的距离测量值 - 此示例假定测量值非常精确。
- 警报系统设置的每次距离测量的时间间隔 - 要使测量误差低于 0.05 米，采样间隔必须小于 0.27 秒。采用 0.25 秒。
- 传感器发出警报时的距离 - 分析表明，当机器人距离障碍物大约 0.16 米时，必须开始减速。但实际警报距离还必须将离散测量误差 0.05 米考虑在内。

#### 添加设计的组件

构建传感器：

1. 创建一个具有四个输入端口和一个输出端口的子系统。该子系统接收机器人的 x 和 y 坐标以及障碍物的 x 和 y 坐标的输入。传感器产生的警报信号连接到输出端口。

   ![The view of the sensor subsystem in the parent diagram shows the input and output ports.](https://ww2.mathworks.cn/help/simulink/gs/sensor_model.png)

2. 构建距离测量值子系统。在名为 `Sensor model` 的子系统中，使用 Subtract 模块、具有函数 `magnitude^2` 的 Math Function 模块、Sum 模块和 Sqrt 模块来实现距离计算。请注意，在子系统内，输入端口的排列不需要与 Subsystem 模块接口上的端口排列匹配。

   ![Blocks inside the subsystem that represents the sensor implement the distance calculation.](https://ww2.mathworks.cn/help/simulink/gs/sensor_distance.png)

3. 要对采样进行建模，请从 Discrete 库向子系统添加一个 Zero-Order Hold 模块，并将该模块的**采样时间**参数设置为 `0.25`。

4. 将距离计算的结果连接到 Zero-Order Hold 模块的输入。

5. 要进行警报逻辑建模，请从 Logic and Bit Operations 库中添加 Compare to Constant 模块，并设置以下模块参数：

   - **运算符**：`<=`
   - **常量值**：`0.21`
   - **输出数据类型**：`boolean`

   使用这些参数值，当输入值小于或等于 `0.21` 时，模块输出值为 `1`。

6. 将 Zero-Order Hold 模块的输出连接到 Compare to Constant 模块的输入。

7. 最后，将 Compare to Constant 模块的输出连接到名为 `Alert` 的 Outport 模块。

   ![The block diagram shows the contents of the subsystem that models the sensor.](https://ww2.mathworks.cn/help/simulink/gs/alert_logic.png)

#### 验证设计

使用 Constant 模块作为 Sensor model 子系统的输入，以 X = 0.65, Y = 0 的障碍物位置来测试设计。此测试验证 X 方向的设计功能。您可以针对不同路径创建类似的测试。此模型仅生成警报，不控制机器人。

1. 设置障碍物位置。从 Sources 库中添加两个 Constant 模块，并将常量值设置为 `0.65` 和 `0`。将机器人的位置输出连接到传感器的输入端口。

2. 为 Alert 输出添加一个示波器。

   ![The view of the top model shows the floating scope added to the Alert output of the subsystem that represents the sensor.](https://ww2.mathworks.cn/help/simulink/gs/complete_model.png)

3. 对模型进行仿真。

示波器中的机器人位置图看起来与上一次运行相同。

![The scope shows the position of the robot over the 20 second simulation.](https://ww2.mathworks.cn/help/simulink/gs/verify_design1.png)

连接到警报信号的示波器显示，当机器人进入障碍物位置的 `0.21` 米范围内时，警报信号值变为 `1`，这满足该组件的设计需求。

![The scope shows the value of the alert signal over the 20 second simulation.](https://ww2.mathworks.cn/help/simulink/gs/verify_design2.png)

对于具有复杂组件和严格要求的真实系统，Simulink® 产品系列提供了一些附加工具，可以细化和自动完成设计过程。Requirements Toolbox™ 可以对要求进行形式化定义，并将它们与模型组件联系起来。如果您要为此机器人构建一个控制器，Simulink Control Design™ 可以帮助您完成设计。Simulink Verification and Validation™ 产品则建立了一套严格的框架，可用于测试组件和系统。

