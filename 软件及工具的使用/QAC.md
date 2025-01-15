###### QAC<br><br>**你要信我啊**<br><br><br><br>*COPYRIGHT ©ryf*

# 相关基础知识

## 评估标准

1. **缺陷数量**：报告中列出的所有缺陷和警告的数量。
2. **缺陷严重性**：缺陷的等级分类（如严重、中等、轻微），帮助优先处理高风险问题。
3. **代码复杂度**：圈复杂度等度量，反映代码的复杂程度。
4. **代码规范违例**：违反编码标准和最佳实践的数量和类型。
5. **代码覆盖率**：静态分析所覆盖的代码比例，评估代码的全面性。
6. **重复代码**：检测到的重复代码块的数量。
7. **安全漏洞**：识别的安全问题和潜在的漏洞。
8. **性能警告**：关于性能问题的警告和建议。

## 规则

在 **QAC**（Quality Assurance C）中，**规则** 是一组编码标准或检查项，用于静态代码分析。它们的主要作用是自动检测源代码中的潜在错误、不符合编码规范的地方，以及可能导致软件漏洞或性能问题的代码。具体来说，QAC中的规则主要有以下几个作用：

1. **编码标准一致性**：确保代码符合编码标准（例如MISRA-C标准），提升代码的一致性和可维护性。
2. **错误检测**：帮助识别常见的编程错误，如空指针引用、数组越界、未初始化的变量等。
3. **代码质量提高**：通过检测潜在的错误和坏习惯，提升代码的整体质量和可读性。
4. **安全性增强**：识别可能导致安全漏洞的代码，如未正确处理用户输入、内存泄漏等。
5. **减少调试时间**：在代码早期阶段检测并修复问题，减少后期调试和修复的工作量。
6. **符合行业规范**：某些行业（如汽车、医疗设备）对软件安全性和可靠性要求很高，QAC的规则有助于确保代码符合这些行业规范。

## 检测原理

1. **代码解析（Parsing）**
   - **词法分析和语法分析**：QAC 首先对源代码进行词法分析，将代码分解成标记（tokens），然后进行语法分析，构建语法树（Abstract Syntax Tree, AST）。这一步确保代码的语法结构正确，并为后续的分析提供基础。
   - **语义分析**：在语法树基础上，QAC 进行语义分析，理解代码的实际含义，包括变量类型、作用域、函数调用等信息。

2. **规则应用（Rule Application）**
   - **预定义规则**：QAC 内置了大量的规则，这些规则可能基于行业标准（如 MISRA-C、CERT C）或最佳编码实践。这些规则涵盖了代码风格、潜在错误、安全漏洞等多个方面。
   - **自定义规则**：用户可以根据项目需求定义自有规则，进一步细化代码检查标准。
   - **规则匹配**：QAC 将解析后的代码与这些规则进行匹配，识别出不符合规范或存在潜在问题的代码片段。

3. **控制流和数据流分析（Control Flow and Data Flow Analysis）**
   - **控制流分析**：QAC 分析代码中的控制结构（如条件分支、循环、跳转等），识别复杂或不合理的控制流模式，预防死循环、未定义行为等问题。
   - **数据流分析**：通过追踪变量的定义、使用和传递，QAC 检测未初始化变量、变量覆盖、内存泄漏、数据竞争等问题。

4. **抽象语法树（AST）和中间表示（Intermediate Representation）**
   - **AST 构建**：通过构建 AST，QAC 能够深入理解代码的结构和层次关系，支持复杂的代码模式识别。
   - **中间表示**：QAC 可能将代码转换为中间表示，以便进行更高效和全面的分析，如路径分析、依赖分析等。

5. **错误检测与报告（Error Detection and Reporting）**
   - **错误分类**：QAC 将检测到的问题按照严重性、类型进行分类，如警告、错误、建议等。
   - **详细报告**：生成详细的报告，指出问题所在的具体位置（文件、行号）、问题描述以及可能的解决建议，帮助开发人员快速定位和修复问题。

6. **集成与自动化（Integration and Automation）**
   - **集成开发环境（IDE）支持**：QAC 可以集成到各种 IDE 中，实时反馈代码问题，提高开发效率。
   - **持续集成（CI）支持**：与持续集成工具集成，实现自动化的代码检查，确保每次代码提交都符合质量标准。

7. **增量分析与优化（Incremental Analysis and Optimization）**
   - **增量分析**：QAC 支持对代码的增量分析，只分析发生变化的部分，提高分析效率。
   - **性能优化**：通过优化算法和数据结构，QAC 能在保证分析深度和准确性的同时，提高分析速度。

**总结**

QAC 通过全面的静态代码分析方法，包括语法和语义解析、规则匹配、控制流和数据流分析，结合预定义和自定义的规则集，实现对代码质量、安全性和规范性的多维度检查。其自动化、集成化的特点，使得开发团队能够在早期阶段发现并修复潜在问题，提升软件的整体质量和可靠性。

# 操作流程

QAC工程构成

```
project
	├─prqa
    │	├─configs
   	│	│  └─Initial
    │	│      ├─cip
    │	│      ├─config
    │	│      ├─logs
    │	│      ├─output
    │	│      └─reports
    │	├─unified
    │	└─upgrade
	├─prqaproject.xml
  	├─src
  	└─lib
```

- prqaproject.xml：文件保存工程相关信息
- config：包含工程各配置文件，如 cct、 acf、 rcf 等。
- output：按照代码层次创建文件夹，保存分析后输出文件。
- reports： 保存生成的报告文件
- src\lib…：相关代码文件夹， 可手动或通过同步导入（保持其结构）

## 新建工程

1. 点击 Create New Project 按钮，或点击 Project 菜单下的 Create New Project，创建新工程

![新建工程](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_14_34_39_202409231434699.png)

2. 在弹出对话框设置工程基本信息 。
   1. 通过 Project Location 右侧按钮选择工程根目录， Project Name 栏以根目录名作为工程名。通过右侧按钮选择工程根目录，以根目录名作为工程名， 选好目录后可以进行修改，修改后重新创建工程目录， 工程名称根据选定的工程路径自动生成。
   
   2. 根据工程需要在下拉菜单选择配置文件。（初始选择 default.acf 和 default.en_US.rcf 即可）
   
      > [!TIP]
      >
      > 注意，这里不能勾选AUTO CTT，否则就不能next
   
   3. 点击 Next， 进行相应编译器的选择，先选择 C，在选择 C++，如果只用是 C 或者只是C++的代码，则另外一种 CCT（Compiler Compatibility Template， 编译器兼容模板） 可以任意选择。
   
      > [!NOTE]
      >
      > 建议选择与实际编译器匹配或接近的 CCT 文件，没有匹配的编译器时选择Helix_Generic_C 或 Helix_Generic_C++ 。

设置完毕后，点击 Finish， 则可在弹出的窗口进行工程详细设定（可暂时不进行详细设定）。  

在`files`卡片中会显示相关工程的组成结构

> [!IMPORTANT]
>
> Reprise License Manager（服务器端） 在每次使用 QAC 前都要启动， 并在QAC 使用期间保持打开。  

## 添加源码

*`手动`导入*

手动添加代码可选择添加单一文件，或添加文件夹。  

可通过右键resource，选择add file来添加源码文件

![添加源码文件](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_15_47_50_202409231547444.png)

*`同步`方式*

同步方式需要跟踪编译过程， 因此 QAC 客户端电脑具备编译环境时才可使用此方式， 主要适用于以下情况：

1. 分析一个已经成功编译的完整工程

2.  Linux 环境下的测试，尤其是使用了 Linux 下的交叉编译环境，由于调用路径纷繁不好追踪， “同步”这种半自动化方式可以协助解决此类问题

步骤如下：

1. 点击 project->synchronize  

2. 在弹出的窗口中填写调用的编译环境，并点击 Synchronize，进行调用。一般可有如下两种模式：  

   1. “Enter Build Command”直接填写编译环境调用的命令，如“.\Keil_v5\UV4\UV4.exe”，则“Optional Working Directory”中无需填写任何路径，该种情况下，点击“Synchronize”按钮之后，会自动调用对应的 keil 编译环境，直接在编译器界面对待测试工程进行build 手动操作，即可实现 QAC 的源码同步

   2.  “Enter Build Command”填写的是跟踪待测工程的编译命令，如“ make clean &&make”， 则必须在“Optional Working Directory”中指明 make 指令执行的 makefile文件的路径，否则无法实现跟踪

      > [!NOTE]
      >
      > 需要注意， 两种模式均需保证工程编译之前是 clean 的状态  

      > 这里以keil为例，在synchronize的页面的entre build command的输入框中输入keil的运行程序exe文件

3. 此时等待 QAC 自动调起 keil。切换到自己需要进行静态分析的工程,对该工程 clean再进行 build 一次，等待 build 完成。在编译环境 build 过程中， QAC 客户端窗口会显示一些文件逐步 File-added  

4. build 完成后， 手动关闭 keil，可以在 QAC 中发现刚刚编译的文件已经添加进来

5. 打开所创建的 QAC 工程目录，一级目录下有文件 prqaproject.xml 文件，文本格式打开可以看到加载进去的所有源码文件,以及源码对应的一些头文件或宏定义信息。  

## 工程配置

1. 点击 Project -> Project Properties，打开工程属性页面。  

   1. Analysis 用于分析配置。从 Component Option 中选择配置选项，在对应 Argument 窗口（界面右下角）中进行对应设置。

      1. -d：进行必要的宏定义添加

      2. -threshold
         度量元检测值定义

         metric{< 或 <= 或 = 或 > 或 >=}value[:msq no]

         eg.STCYC>10.6000

      3. -i：添加需要的头文件路径

      4. -quiet：屏蔽对头文件的扫描

      5. -warncall
         调用函数设置警告

         function[=msq no]

         eq:open=6001

      6. -maxerrors

         指定最大错误数

      ![参数设置](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/23_16_23_25_202409231623694.png)

   2. Rule Configuration 用于消息设定 ：在 Rule Configuration 可查看变更 Message 以及规则组  
   
      ![消息配置](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_9_57_47_202409240957871.png)
      
      1. Analysis 界面中添加 M3CM 组件（弹出的窗口点击 Yes）
      
         ![添加组件](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_10_4_30_202409241004394.png)
      
         点击完确认之后，会显示产生的相关文件
      
         ![产生的文件名](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_10_6_38_202409241006372.png)
      
      2. 根据 QAC 版本选择对应的 m3cm 规则集进行导入。点击保存即可
      
         ![导入相关规则文件](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_10_12_59_202409241012717.png)

## 执行分析

分析方法有两种：

1. 方式一： 通过菜单栏 Analysis->File-Based Analysis，选择对当前工程、所选文件或勾选指定文件进行静态分析。
   1. 也可以直接点击快捷键进行分析  <img src="https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_10_17_53_202409241017018.png#icon" alt="image-20240924101752916" style="zoom:200%;" />

2. 方式二： 可在工程文件列表中， 选择相应文件夹/文件，右键 Analyse selected file(s)，对其进行分析。  

## 分析结果查看

当分析完成文件之后，将会显示源代码，在源代码的左侧会显示行代码是否有问题

![代码检测结果](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_10_25_18_202409241025469.png)

1. 违反消息处有叹号标记
   1. Analysis Results/Diagnostics 中罗列当前选中文件中违反的消息信息。点击具体 ID 查看 Message 帮助文件，点击 Rule，查看规则详细信息，双击消息内容，在代码中高亮显示违反部分代码。可以通过窗口上方叹号标记筛选显示不同级别消息。筛选同时作用于源代码上的标记。
   2. 红色标记： Error message
   3. 黄色标记： Warning message
   4. 绿色标记： User message
   5. 蓝色标记： Information message
2. 点击“Rule” 下规则条目， 可通过悬浮窗可查看每个违反消息的帮助文档
3. 消息索引：Files、 Rule Groups、 Message Levels

## 导出报告

点击*==菜单>Report>选择生成工程或单独文件的报告。==*

可生成 7 种相关报告：

1.  Code Review Report：生成 html类型文件，显示文件相关度量结果和分析状态（未被抑制的违反规则数）
2. Metrics Data Report：点击生成 Metrics Data Report，将生成 xml 文件，存储相关度量指标数据。
3. Rule Compliance Report：生成 html 类型文件，显示文件详细的违反规则数。
4.  Suppression Report：生成 xml 文件，显示被抑制的消息信息。
5. HIS Metrics Report：生成 html 文件，显示 HIS 度量报告。
6.  Standards Compliance Report： 生成 html 文件， 根据项目所选的规则集显示项目关于该规则集的合规性报告。
7.  MISRA Compliance Report： 生成 html 文件， 显示 MISRA 合规性报告。

# Dashboard相关操作

## 上传项目

1. 服务器端 Perforce -> Helix QAC Dashboard server ， 保持 Dashboard server 开启状态

2. 打开 Helix QAC， 点击*==Portals > Dashboard > connect to Dashboard Sever==*，设置 URL、用户名及密码后，连接到相应的 Dashboard Sever。（注意：首行 Dashboard URL 的填写格式为“Dashboard server 端 IP 地址:8080”）

   > 默认登录 Username 和 Password 均为 admin。

   也可以使用快捷按键<img src="https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_11_12_24_202409241112963.png#icon" alt="连接服务端" style="zoom:150%;" />

3. 连接成功后页面右下角显示：  Dashboard Connection Status:Connected to 192.168.3.224:8080

4. 点击 *==Portals > Dashboard > Upload Results==*，选择上传文件或工程分析结果到 Dashboard。  

5. 在弹出窗口填写相关信息：

   > 注意： Upload Source 选择 All， 可以将源码以及诊断消息全部上传到 dashboard 上，默认只上传诊断消息。
   >
   > Snapshot Nam填写当前测试的版本

> [!WARNING]
>
> 注意：这里需要查看是否上传完成
>
> 需要关掉翻墙软件，防止出现ip地址不明的情况

## 查看项目

在打开的 IE 浏览器输入`http://服务器地址：端口号`（比如 http://localhost:8080）

输入用户名和密码，都是 admin，进入 Dashboard 界面：

1) 从主面板进入工程界面，选中上传之后的项目之后可以对项目做以下的操作：
   1)  Web Message Browser
   2) Diagnostic Summary
   3) Measurement Analysis
   4) Reports

> **Web Message Browser 界面  **
>
> 会展示所有的相关消息
>
> ![相关规则组消息](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_14_32_38_202409241432310.png)
>
> 也可以根据消息的严重程度来筛选所要展示的消息
>
> ![根据消息的严重程度选择性的展示](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_14_35_38_202409241435503.png)
>
> 在不同的区域右键会出现不同的相关信息
>
> 1. 在消息上右键出现Message 菜单： 鼠标右键点击某一条消息会弹出此菜单（消息号、消息帮助链接、严重等级、对应行号、所属消息组、抑制选项等信息）  
>
> ![Message菜单](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_14_49_56_202409241449165.png)
>
> 2. 在源码上右键出现Source Code 菜单： 鼠标右键点击某一行代码会弹出此菜单（行号、所在函数名、文件及函数度量信息、抑制选项等）
>
>    ![source code菜单](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_14_54_41_202409241454409.png)
>
> 3. 在消息组上右键出现Message Group 菜单： 鼠标右键点击某一个消息组会弹出此菜单（主要用来设置抑制选项）
>
>    ![Message group菜单](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_14_58_18_202409241458058.png) 

> **Diagnostic Summary 界面  **
>
> 显示了工程代码的结构：选择任意文件夹或者文件可以查看所选粒度的诊断信息， 可生成固定报告模板。  
>
> ![诊断总结页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_15_14_19_202409241514909.png)

> **Measurement Analysis 界面  **
>
> 显示质量度量指标的发展趋势：可以根据快照名称或者快照的日期来查看趋势。  
>
> ![image-20240924152224957](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_15_22_25_202409241522086.png)
>
> > [!IMPORTANT]
> >
> > 以下是度量标准相关释义：
> >
> > 1. **STCAL**（Static Complexity Analysis）：评估代码的整体复杂性，通常基于控制流和数据流分析。
> > 2. **STM29**（Module Complexity Analysis）：分析模块的复杂性，关注模块的功能和结构。
> > 3. **STCYC**（Static Cyclomatic Complexity）：基于控制流图的分支和循环结构，衡量代码的复杂性。
> > 4. **STPFL**（Static Path Length）：评估执行路径的长度，反映程序的复杂性。
> > 5. **STVFL**（Static Variable Flow）：分析变量在程序中的使用和流动，关注数据依赖性。
> > 6. **STFAN**（Static Fan-in/Fan-out Analysis）：评估模块之间的调用关系，关注模块的耦合度。
> > 7. **STCLO**（Static Closure Analysis）：衡量代码的封闭性，反映模块的独立性和可重用性。
> > 8. **STNFC**（Static Nested Function Complexity）：测量嵌套函数的复杂性，关注其对可读性的影响。
> > 9. **STMAX**（Static Maximum Depth）：评估程序中嵌套结构的最大深度，反映复杂性。
> > 10. **STDDM**（Static Data Dependency Measurement）：分析数据依赖关系，关注变量间的影响。
> > 11. **STRED**（Static Redundancy Measurement）：测量代码中冗余部分的复杂度。
> > 12. **STMFC**（Static Module Function Complexity）：评估模块内函数的复杂性。
> > 13. **STDEC**（Static Depth of Execution Complexity）：评估执行路径的深度。
> > 14. **STWMC**（Static Weighted Method Complexity）：考虑权重的函数复杂度度量。
> > 15. **STLCC**（Static Lines of Code Complexity）：基于代码行数的复杂度评估。
> > 16. **STLOP**（Static Logic Operation Complexity）：评估逻辑操作的复杂性。
> > 17. **STCFG**（Static Control Flow Graph Complexity）：分析控制流图的复杂度。
> > 18. **STCAI**（Static Call Instruction Complexity）：评估调用指令的复杂性。
> > 19. **STBFC**（Static Branch Frequency Complexity）：分析分支频率对代码复杂性的影响。
> > 20. **STABI**（Static Abstraction Complexity）：评估代码的抽象层次。
> > 21. **STMPI**（Static Module Performance Indicator）：评估模块的性能指标。
> > 22. **STTDC**（Static Type Dependency Complexity）：分析类型依赖关系的复杂性。
> > 23. **STEXT**（Static Execution Trace Complexity）：评估执行轨迹的复杂度。
> > 24. **STHWC**（Static Hardware Complexity）：评估代码与硬件交互的复杂性。
> >

## 生成报告

自定制报告，点击 Report，进入自定制报告界面  

在界面右上角，点击 Add Report Object，进行报告自定制， 支持 HTML、 PDF 或者XML 格式。  

![生成报告](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_16_53_24_202409241653863.png)

## 用户管理

Dashboard 最多可同时登录 5 个用户，默认用户为 admin。创建新用户的步骤如下：

1) 在 Dashboard 主界面（Project Dashboard），点击“User Management”按钮。

2) 进入 User Management 界面，点击左上角“Add New User”按钮；填写新用户信息（用户名和密码）， 点击“Save”。

![用户管理页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_16_58_16_202409241658612.png)

3. 选中新建用户，使能权限。点击 Edit permissions，在用户权限界面可依实际需求进行选择，点击 Toggle All Tickboxes 可使能全部权限。  

![新建用户](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/09/24_17_6_26_202409241706262.png)

# 可能出现的问题及解决方案

对于rlm软件打开一会自动关闭的情况，解决方法如下：

> 在打开rlm软件的情况下，访问[localhost:5054](http://localhost:5054/)
>
> 在用户和密码位置键入账号和密码（首次登陆的账号和密码均是admin）
>
> 点击change passward，修改密码为yxd12345（自定义）
>
> 重启rlm即可











































































































