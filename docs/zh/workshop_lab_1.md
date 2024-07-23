---
chapnum: 1
---

# 实验室 1: 使用 OpenScenario 2 和 Foretellix 技术的第一步

## 学习目标

本实验室将向您介绍以下内容：

- 简单的 **ASAM OpenSCENARIO 2.0 (OSC2)** 测试实现。 (ASAM 是自动化和测量系统标准化协会。)

!!! 注意
 在 Foretellix 文档中，“OSC2”一词指的是“ASAM OpenSCENARIO® DSL 版本 2.x”。

- **Foretify™**，用于：
 - 编译 OSC2 源码
 - 从抽象场景定义生成具体测试
 - 在测试执行过程中控制模拟平台上的参与者
 - 绘制和可视化测试执行结果 的场景开发和测试自动化平台

- **Foretify Manager**，主要用于：
 - 收集多个测试执行的关键性能指标（KPI）、检查器消息和覆盖率指标
 - 可视化测试进展，支持基于安全驱动验证（SDV）方法论的方法

<p align="center">
 <a href="images/l01_ftx_diagram.png" target="_blank">
 <img src="images/l01_ftx_diagram.png">
 </a>
</p>

!!! 注意
 单击任何图像都会在新选项卡中以全分辨率打开。 

## OSC2语言

在AD和ADAS功能的开发和验证过程中，有必要用各种*场景*来激励系统测试（SUT）。场景是一个或多个行动者（如汽车、行人、环境条件和SUT本身）的定时行动序列。OSC2是一种特定领域的语言，专门用于描述行动者在环境中移动的场景。这些场景具有属性，使您能够限制行动者类型、它们的移动方式以及环境（包括场景应该发生的地图位置）。

!!! 信息
    采用*受限随机*方法，未受限制的每个场景属性都将被随机化。例如，如果您不将切入场景的“侧”属性约束为“右侧”，它将从可能属性空间（即“左侧”和“右侧”）中随机选择。

    此外，这些值是从可能属性空间中选择的，以满足源自场景、行动者和地图的约束条件。例如，如果SUT（EGO）正在两车道道路的最右车道上行驶，则切入“侧”属性不会选择为“右侧”。

OSC2的构建模块是数据结构，例如：

- **演员**：代表真实世界的实体。顾名思义，它们在场景中“扮演角色”。
- **场景**或**动作**：描述演员的行为。通常，场景是一系列较长的动作，但两者之间没有正式区别。两者都可以通过*修饰符*进行修改。
- **修饰符**：为场景添加约束，帮助控制其在期望范围内的执行。
- **标签**：定义任何标量、结构或演员类型的命名数据字段。
- **简单结构**：是包含属性、约束等基本实体。

您可以通过参考Foretellix [OSC2语言文档](../osc_lang/osclang_intro.md)和[OSC领域模型文档](../osc_dom/oscdomain_intro.md)，或访问[ASAM类型定义主题](https://www.asam.net/index.php?eID=dumpFile&t=f&f=3460&token=14e7c7fab9c9b75118bb4939c725738fa0521fe9#type-definitions)或主要的[ASAM OSC2网页](https://www.asam.net/project-detail/asam-openscenario-v20-1/)，深入了解上述主题。

!!! 信息
    Foretellix工具**原生支持ASAM OSC2语言**，带来许多好处，其中之一是该语言和工具是**执行平台无关**的。

```markdown
1. 导入测试执行平台配置（例如模拟器）。

2. 导入SUT配置（例如被测系统，通常称为EGO）。在这种情况下，作为被测试的功能，我们正在导入Foretellix开发的SUT L4堆栈，并配置EGO车辆的属性。

3. 设置用于测试的地图。

4. 定义场景和指标（检查、覆盖率、关键绩效指标），并调用场景执行。

在下图中，您可以看到将要运行的测试结构：

<p align="center">
 <a href="images/Test_2.png" target="_blank">
 <img src="images/Test_2.png">
 </a>
</p>

!!! 例 "实践时间"
 现在您已经准备开始研讨会了，是时候更详细地了解第一个测试了。

 使用Foretify Developer OSC2代码编译器打开第一个测试：

 ```bash
 foretify --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --gui
 ```

 Foretify GUI 将出现在您的浏览器中。在培训过程中您将了解更多功能。现在您需要专注于其OSC2代码编译和可视化功能。点击**Source**选项卡并折叠**Loaded Files**窗格，如下图所示：

<p align="center">
 <a href="images/l01_ftx_dev.png" target="_blank">
 <img src="images/l01_ftx_dev.png">
 </a>
</p>

请注意，在该测试中有导入语句用于加载其他OSC2文件：

```osc linenums="3"
import "$FTX_WORKSHOP/common/workshop_config.osc"
import "$FTX_WORKSHOP/scenarios/cut_in_l01.osc"
import "ts_l01_intro_cov.osc"
import "ts_l01_intro_checks.osc"
```
所有代码都可以放在一个文件中，但那样会使其不易阅读、不易重复使用且难以管理。
```

### _workshop_config.osc_ 配置文件

`workshop_config.osc` 文件（在测试文件的第3行导入）包含了设置 Foretify 和执行平台连接的所有定义（可以使用不同的模拟器），以及系统测试对象（SUT）连接的定义（在本例中是自动驾驶的 Ego）。

### _cut_in_l01.osc_ 场景文件

这个 `cut_in_l01.osc` 文件被导入到第4行，并包含了切入场景的抽象定义，也就是本实验的主题。我们通过一组绝对和相对约束来定义，一个车辆应该在 SUT 的前面变道。OSC2 是唯一支持抽象化的场景描述语言，大大减少了场景开发人员编写代码所需的时间。
在下面的图像中，您可以看到抽象切入场景定义的两个具体实例。

&lt;p align="center">
  &lt;a href="images/workshop_lab_1_cut_in_lab_1_scenario.png" target="_blank">
    &lt;img src="images/workshop_lab_1_cut_in_lab_1_scenario.png">
  &lt;/a>
&lt;/p>

!!! 示例 "动手时间"
    打开 `cut_in_l01.osc` 文件，方法是点击源选项卡中测试文件的第4行导入

- 第6行：声明了名为sut.cut_in_l01的场景，在SUT的上下文中（这就是为什么它是sut.name_of_scenario）。
- 第7行：car1，另一辆车（不是SUT），被实例化为“vehicle”类型的对象。
- 第8行：从中car1将插入的一侧被实例化为枚举类型“av_side”。这意味着它可以保存“left”或“right”的值。
- 第10行：表示后续的代码块将依次执行。在这种情况下，它意味着在log_info之后将执行名为approach_phase的阶段，然后是change_lane阶段。
- 第12行：将插入的一侧写入日志。日志语句允许您添加可以用于调试目的的行。
- 第14行：创建并标记为“approach_phase”的并行场景阶段。这意味着第15行和第18行将并行执行（记住，OSC2是一种基于缩进的语言）。标签允许您从代码的其他区域引用该代码部分。
  - 第15行：触发SUT开始执行drive()动作。接下来的行（16和17）对SUT的驾驶行为添加了一些约束，即：
    - 第16行：约束SUT在进入approach阶段时的速度至少为30公里/小时。
    - 第17行：约束SUT在整个approach阶段保持车道。
  - 第18行：触发car1开始执行drive()动作。由于car1不是SUT，它将在驾驶时完全由Foretify引擎控制。接下来的行（19到22行）对car1的移动添加了一些约束，以实现插入操作，即：
    - 第19行：约束car1在整个阶段中处于与SUT相邻的车道。
    - 第20行：定义了car1相对于SUT在该阶段开始时的位置（在SUT前方10到20米）。
    - 第21行：定义了car1相对于SUT在该阶段结束时的位置（在SUT前方10到20米）。
    - 第22行：定义了car1相对于SUT的位置为最佳努力条件。这意味着，如果约束无法满足，求解器不会将此场景标记为失败。在某些情况下，由于模拟器的限制，计划的运行无法完全展开，并被标记为“不完整的场景”。将非关键约束定义为最佳努力约束可以帮助减少此类运行的比例。您将在高级实验室中了解更多关于此功能以及“计划”和“运行时”之间的区别的内容。
- 第23行：创建并标记为“change_lane”的第二个并行场景阶段。同样，这意味着第24行和第32行的后续动作将并行执行。
  - 第24行：触发SUT开始执行drive()动作，然后是几个修饰符。
    - 第25行：对SUT的驾驶行为添加了一个约束，即在整个场景阶段保持车道。
  - 第26行：触发car1执行drive()动作，受到第29到34行的约束：
    - 第27行：约束car1在该阶段开始时的速度比SUT慢5到15公里/小时。
    - 第28行：将速度约束覆盖为非关键的最佳努力约束。
    - 第29行：约束car1在场景阶段结束时与SUT在同一车道上。
    - 第30行：car1的速度在整个场景阶段保持恒定。
    - 第31行：再次，此速度约束被定义为非关键约束，但以最佳努力方式执行。
    - 第32行：此行禁用了car1的碰撞避免行为，从而使车道变更对SUT具有挑战性。您将在高级实验室中了解更多关于通用汽车角色的碰撞避免行为的内容。

### 行为监测

接下来的三个部分将简要介绍 _覆盖率_、_KPI 或记录_ 和 _检查器_ 的概念，并举例加以说明。它们是安全驱动验证方法论的关键支柱，因为它们支持验证过程并揭示系统单元测试的不正确表现。在下一个实验中，您将深入了解并详细阐述它们的功能。

#### _ts_l01_intro_cov.osc_ 覆盖定义

!!! 例子 "实践时间"
 打开 `ts_l01_intro_cov.osc` 文件，该文件通过单击“源代码”选项卡中 `ts_l01_intro.osc` 测试文件的第 5 行而被导入。

<p align="center">
 <a href="images/workshop_lab_1_cut_in_lab_1_osc_cover.png" target="_blank">
 <img src="images/workshop_lab_1_cut_in_lab_1_osc_cover.png">
 </a>
</p>

上面的代码定义了 **覆盖收集**：

- 第一个指标（第 4 行）是切入发生的一侧——左边还是右边。
- 第二个指标（第 5 行）是进行切入操作车辆的类型，例如卡车或轿车。
- 接下来的指标（第 6 到 11 行）是车辆在换道场景阶段结束时行驶的速度。
- 最后一个指标（第 12 到 18 行）是主场景开始时两辆汽车之间纵向距离。

!!!信息
 - 在第六行中，我们只使用了 OSC2 的一个重要结构化类型成员：事件。该事件为 _end_, 更确切地表示了在 `$FTX_WORKSHOP/scenarios/cut_in_l01.osc` 场景中定义瞬变换道阶段结束事件。
 - 事件是表示时间点且可以触发场景中定义操作的瞬态对象。您可以在结构体内定义事件，但更常见地在角色或场景内部进行定义，就像这个例子一样。

### _ts_l01_intro_cov.osc_ KPI定义

KPI的定义与前一部分的覆盖项在同一个文件中进行。

在代码的这一部分，我们定义了一个KPI：

- 首先，在第21到22行声明了一个变量，用于采样SUT和切入车辆之间的距离。
- 第25到27行使用了_record()_方法，记录将来要可视化的KPI。

!!! 信息
 现在你已经通过了一些例子，理解覆盖度指标和性能指标之间的主要区别至关重要:

 - **覆盖度评估**: _我们在“场景空间”的哪个部分进行自动驾驶汽车测试？_ 这是通过覆盖度和整体覆盖度等级来表达。为支持覆盖度评估而定义了覆盖项。换句话说，覆盖度等级回答了这个问题：_SUT测试得有多好？_
 - **性能评估**: _系统正在测试中表现如何？_ 这个问题由性能等级回答，可以是一个或多个KPIs

### _ts_l01_intro_checks.osc_ 检查器定义

!!! 示例 "Hands-on Time"
 点击`ts_l01_intro.osc`测试文件Source选项卡中第6行导入的`ts_l01_intro_checks.osc`文件。

<p align="center">
 <a href="images/l01_kpi_code.png" target="_blank">
 <img src="images/l01_kpi_code.png">
 </a>
</p>

该检查器的目的是评估SUT和切入车辆之间是否始终保持在规定安全距离内。

- 在第1行，正在使用唯一名称（safety_distance）扩展issue_kind类型，用于新的checker。
- 第3到12行扩展了先前定义的场景以编写checker，具体如下：
  - 第5至6行：声明了安全距离阈值的变量，并将其设置为13米
  - 第8至10行：在模拟的每个时间步骤（top.clk），验证以下条件是否成立：
    - 两辆车之间的距离是否超过了定义的阈值
    - 两辆车是否在同一车道上
  - 第11至12行：如果上述条件不满足，则停止测试运行，并报告sut_error类型和subtype safety_distance。

!!! Info

- 刚刚添加的checker是用户自定义checker，但Foretify附带了内置checkers。可以在[全球车辆检查器文档](../osc_dom/oscdomain_metrics.md#global-checkers-for-vehicles)中查看这些内容。

- 行业中另一个用于checker的术语是evaluator。

- 使用checkers可以表示场景成功或失败的标准，它们采用与表示动作和演员相同语言（OSC2）编写。

- 在使用Foretify时，某些预定义checkers始终处于活动状态，例如碰撞检查或SUT偏离道路驾驶检查。您始终可以修改适用于所有场景的默认设置。

- 您可以定义自定义checkers来捕获与您SUT特定问题有关联内容，就像我们示例中所做。我们所执行的checker使用KPI值和预定义阈值来评估SUT期望表现方式，但这只是其中一种确定方式。

### 地图定义

在以下代码（`ts_l01_intro.osc`测试文件第8和9行），我们设置要使用地图，该地图以OpenDrive格式（即\*.xodr）提供。

```osc linenums="8"
extend test_config:
 set map = "$FTX_WORKSHOP/maps/Town04.xodr"
```

### 情景执行

在`ts_l01_intro.osc`文件的最后两行中，我们终于调用了_cut_in_l01_情景：即OSC2程序的入口点，它总是执行_top.main_情景，类似于C/C++中的_main()_函数：

```osc linenums="11"
extend top.main:
 do cil : sut.cut_in_l01()
```
您可以在测试文件的第11和12行找到此代码。

## 首次运行和Foretify

### 有限随机生成和自适应场景执行

顾名思义，*有限随机生成* 是指在指定约束空间内生成随机变量。这是SDV（安全驱动验证）流程的基本前提，并且是Foretify构建的核心原则。

从OSC2抽象场景中，Foretify生成引擎根据指定的约束随机创建具体场景。其中一个最重要的方面是被随机化的地图区域，在每个情景将展开。

一旦计算出情景计划，运行时自适应场景执行引擎会负责按照情境计划进行执行。

!!! Info
 **约束随机生成** 引擎是Foretellix解决方案的主要支柱。这是一个极为强大的工具，可以从抽象描述中生成数百万个有意义变种。

当利用这项技术时，**对于情境编写团队所需工程资源显著减少** ，因为可以从一个抽象定义中产生数百万个有意义变种。

#### 你的首次运行

**在继续之前，请确保你已经关闭了浏览器中的Foretify GUI。**

现在，你已经了解了OSC2场景定义，将使用模拟器作为测试执行平台。

接下来，你将更详细地探索Foretify GUI，以加载、启动和分析测试。稍后你还会探索Foretify的更多功能。

!!! 例子 "实践时间"
 启动GUI模式下的Foretify，并加载之前检查过的测试：

 ```bash
 foretify --gui --work_dir $FTX_FM_WORKDIR/l01_intro/workdir \
 --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc
 ```

Foretify交互窗口弹出：

<p align="center">
 <a href="images/foretify_gui.png" target="_blank">
 <img src="images/foretify_gui.png">
 </a>
</p>

Foretify窗口显示以下信息：

- **加载**，**准备测试**和**调试**选项卡（左上角）：
    - **加载**选项卡用于通过 GUI 加载 `.osc` 文件（我们没有使用 GUI 进行此操作，因为我们通过终端命令加载了文件）。
    - **准备测试**选项卡允许您设置运行的不同参数。您将在后面的部分了解更多信息。
    - **调试**选项卡用于在模拟完成后调试运行。

- **状态**（中上方左侧）：
    - 您可以查看已加载的文件、加载状态以及加载过程中发现的问题数量。

- **地图**，**源码**和**预览**选项卡（中左侧）：
    - **地图**选项卡允许您浏览已加载地图，探索地图的不同层。
    - **源码**选项卡显示已加载的源文件，让您查看加载的代码并在已加载文件之间切换。
    - **预览**选项卡允许您在单击**执行控制**区域中的**预览**按钮后预览计划的测试。

- **问题**，**检查器违规**和**测试信息**选项卡（左下角）：
    - 这些选项卡在场景加载后显示。

- 控制场景的执行（右上方）：
    - 您可以为运行设置种子，预览运行或运行实际模拟。种子是每个具体测试执行生成的唯一标识符，

```markdown
!!! Example "操作时间"
 在实际运行测试之前，您可以通过单击**预览**按钮进行试验，在可视化器中显示出系统下测试 (SUT) 和 car1 的计划路径。请注意，由于 SUT 的行为，计划路径可能会在运行时发生变化，但这是一个非常有用的调试工具。值得注意的是，计划路径只是对该场景创建的计划的一种表现形式：真正的轨迹是在运行时计算和更新的。

!!! Example "操作时间"
 在运行测试之前尝试通过单击右上角灰色**预览**按钮并更改种子号来预览您测试的几个种子。

!!! Example "操作时间"
 通过单击右上角紫色**运行测试**按钮来运行测试。您也可以通过单击右上角的**终端**按钮并输入_run_来运行测试。

随后将看到模拟器窗口弹出：

- Foretify 约束随机生成引擎根据 OSC2 代码中为该场景指定的约束和参数生成了特定变体情景。演员们的计划路径就是你在预览时看到过的那些。

- Foretify 运行时测试编排引擎确保了在执行平台（模拟器）中演员能够根据指定约束移动。

 运行时测试编排引擎确保进行了一次自适应情景执行，这意味着 SUT 计划路径偏离将导致 NPC 对策相应调整以满足 OSC2 文件中指定场景要达到目标意图。

 测试完成后，日志区域显示两个额外标签：
```

- **Trace Details**选项卡可让您检查整个模拟过程中收集的跟踪信息，如跟踪类型、执行者、时间和持续时间。

- **日志**选项卡包含测试执行期间生成的日志。

- **指标**选项卡与覆盖率有关，稍后将在研讨会上详细介绍。

!!! 例子 "实操时间"
 现在，请查看**日志**选项卡，并检查是否看到我们添加的日志消息，以显示割入发生在哪一侧。

### 调试运行

#### 使用Foretify Visualizer调试

运行测试后，在屏幕中央，您可以使用**Visualizer**选项卡。 Visualizer是一种图形后处理工具之一，可以以各种方式配置，帮助您分析执行过程。可视化运行与重复执行不同。它不需要模拟器，并且不会消耗重新运行所需的计算资源。

一旦执行完成，在屏幕左侧将自动打开**Visualizer**选项卡。
您随时都可以单击该选项卡返回到Visualizer。 

<p align="center">
 <a href="images/visualizer.png" target="_blank">
 <img src="images/visualizer.png">
 </a>
</p>

##### 回放测试
首先，您可以按下位于左下角Visualizer时间轴上的播放按钮来观看场景是如何重现的：

<p align="center">
 <a href="images/visualizer_play_button.png" target="_blank">
 <img src="images/visualizer_play_button.png">
 </a>
</p>

##### 地图视角
您可以通过在可视化器中单击鼠标右键并将光标拖动到不同的方向来更改视角。您可以通过单击可视化器右上角的扳手图标来访问**视图工具**。**视图工具**提供了控制视图的选项：

<p align="center">
  <a href="images/visualizer_tools.png" target="_blank">
    <img src="images/visualizer_tools.png">
  </a>
</p>

- 车道方向：启用或禁用驾驶方向箭头的显示。
- 信号：切换交通信号的可见性。
- 限速：显示道路的限速。
- 碰撞避免：启用碰撞避免激活的可见性。
- 运行时轨迹：显示所选车辆角色的轨迹。
- 计划路径：突出显示场景中车辆的路径。
- 计划目标：显示为所选车辆角色生成的计划目标。
- 计划姿态：突出显示系统正在测试的车辆和其他车辆的下一个姿态。
- 驾驶员目标：显示为所选车辆角色生成的驾驶目标。
- 预测姿态：显示车辆的预测位置。

##### 摄像头设置

要控制视角和摄像头，请单击摄像头设置（相机）图标。要使用摄像头跟踪特定的角色，请从左上角的下拉列表中选择一个角色，或者在可视化器中单击以将摄像头设置为固定位置。

<p align="center">
  <a href="images/Camera_settings.png" target="_blank">
    <img src="images/Camera_settings.png">
  </a>
</p>

- 透视视图：将视角改为俯视图。
- 跟踪所选角色：跟踪所选的角色。
- 将摄像头重置为所选角色：将摄像头重置为所选角色的位置。

##### 测量距离

- 如果可视化器处于透视视图中，请在可视化器右上角选择“相机设置”图标，并关闭透视视图选项。
<p align="center">
 <a href="images/Measurement_1.png" target="_blank">
 <img src="images/Measurement_1.png">
 </a>
</p>

- 选择“测量距离”工具图标。
<p align="center">
 <a href="images/Measurement_2.png" target="_blank">
 <img src="images/Measurement_2.png">
 </a>
</p>

- 在可视化器中点击以设置测量的起始点，然后移动光标并点击以设置测量的结束点。
<p align="center">
 <a href="images/Measurement_3.png" target="_blank">
 <img src="images/Measurement_3.png">
 </a>
</p>

测量结果将显示在连接起始点和结束点的线旁边。

- 要隐藏测量结果，请切换关闭“测量距离”工具图标。

#### 使用追踪进行调试

所有追踪可以在追踪视图下查看。追踪视图与时间轴对齐，因此您可以轻松比较不同的追踪。

**追踪** 表示为以下不同类型：

- **间隔**: 代表一段时间内收集的数值集合。间隔有名称、开始/结束时间和类型，并且与特定角色（橙色方框）相关联。

- **数值**: 代表随时间变化的单个数值，数值以波形图形式显示，数值追踪有名称、数值和单位，并且与特定角色（红色方框）相关联。

<p align="center">  
<a href = " images / Traces_1.png " target = "_ blank ">  
<img src = " images / Traces_1.png ">  
</ a>  
</ p>  

为了增强可视效果，Foretify记录每个场景的开始和结束时间。

##### 查看间隔：
- 在Foretify中选择“调试运行”选项卡，并点击Visualizer下的Traces选项卡：

&lt;p align="center">
  &lt;a href="images/Traces_2.png" target="_blank">
    &lt;img src="images/Traces_2.png">
  &lt;/a>
&lt;/p>

在时间轴中，迹象显示为间隔，当前时间光标对应于其他基于时间的视图，如可视化器和迹象选项卡中的Actor值迹象。

1. 单击迹象名称左侧的箭头以展开它，并查看其子场景。

2. 单击迹象以查看迹象详细信息，例如迹象类型、Actor、时间、持续时间以及间隔期间收集的度量标准。

3. 在迹象详细信息下，单击迹象的开始时间或结束时间以将通用时间轴设置为该时间。

##### 将时间轴定位到迹象：
1. 在迹象选项卡上，选择要将时间轴定位到的间隔（橙色框）。

2. 单击框架时间轴图标（红色框）。

&lt;p align="center">
  &lt;a href="images/Intervals_3.png" target="_blank">
    &lt;img src="images/Intervals_3.png">
  &lt;/a>
&lt;/p>

- 要重置时间轴，使其不再框定间隔，请单击时间轴右侧的取消框架时间轴图标。

&lt

现在，您可以通过在Foretify终端中键入“exit”或关闭Foretify窗口来关闭Foretify。

!!! Info
    **种子(seed)** 用作具体执行的随机生成的输入，该执行是从抽象场景定义中生成的。使用相同的种子从抽象定义再次生成具体测试会导致相同的具体变化。这是一个关键功能，一方面可以完全随机化测试，另一方面也可以为调试目的重新创建单个具体执行。

    由于种子定义和实现，您始终可以跟踪生成的特定具体变化。场景的可追溯性是一项基本功能，它使您能够识别导致错误的条件并重现它们。

## Foretify Manager

现在您已经运行了第一批测试，可以探索已实现的覆盖范围和结果。我们将在实验室2中正式定义覆盖范围，但现在您将使用Foretellix工具直观地探索这个概念。

Foretify Manager是Foretellix工具，它允许您可视化验证过程的整体状态，导入众多测试执行的结果和KPI。它具有客户端-服务器架构，如下图所示：

<p align="center">
  <a href="images/fmanager_architecture.png" target="_blank">
    <img src="images/fmanager_architecture.png">
  </a>
</p>

客户端可以是Python脚本或Web UI（Web页面），两者都可以对测试套件结果数据执行操作和查询。服务器管理数据库并执行客户端的命令。此拓扑结构允许多个用户同时分析验证结果的不同方面。

### 打开Foretify Manager

Foretify Manager是基于浏览器的应用程序。

!!! 示例 "操作时间"
    从终端调用以下命令启动 Foretify Manager：

    ```bash
    fmanager
    ```

    首先看到的是登录页面：

    <img src="images/fmanager_login.png" alt="image" width="200"/>

!!! 信息
    请联系演讲者获取有关您分配的用户名和密码的更多信息。



### 创建项目

Foretify Manager 项目是一个协作框架，使用户能够为验证和验证数据以及收集的指标设置权限和所有权。

要创建项目：

1. 打开 Foretify Manager 并使用 Foretify Manager 凭据登录。

2. 在“选择项目”页面中，单击“创建新项目”。

!!! 信息
    请联系演讲者获取有关您项目名称的更多信息。

- 单击创建紫色按钮 

<p align="center">
  <a href="images/fmanager_project.png" target="_blank">
    <img src="images/fmanager_project.png">
  </a>
</p>

<p align="center">
  <a href="images/fmanager_project_2.png" target="_blank">
    <img src="images/fmanager_project_2.png">
  </a>
</p>

### 上传测试套件结果

此时，Foretify Manager 数据库为空，因此下一步是上传您执行的少数测试的结果。

!!! 示例 "操作时间"
    切换到终端并通过调用以下命令上传结果：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir
    ```

您可以使用 ``` --run_group_name ``` 参数为您的测试组指定一个特定的名称。请参阅 [upload_runs 文档](../fman_user/fmanuser_launch_test_suite.md#upload-a-regression)。

```markdown
!!! Example "现场实践时间"
 切换回在浏览器中打开的 Foretify Manager web 应用，并点击刷新按钮（确保在“测试套件结果”选项卡上）

上传后，测试执行应该被加载并显示在你之前创建的项目中：

<p align="center">
 <a href="images/fmanager_regression_2.png" target="_blank">
 <img src="images/fmanager_regression_2.png">
 </a>
</p>

### 分析已上传的运行

通过点击刚导入的测试套件，你可以查看各个运行。

你应该看到类似下图所示的内容：

<p align="center">
 <a href="images/fmanager_runs.png" target="_blank">
 <img src="images/fmanager_runs.png">
 </a>
</p>

彩色方块标注了：

- _黄色_: 运行列表右上角的图标让你能够导出和删除运行，以及保留和重置你的选择。使用最右侧的列选择图标添加和移除运行属性，比如目录、操作系统用户、持续时间等。
- _橙色_: _问题树_ 按其类型将所有问题分组呈现。
- _蓝色_: _汇总视图_ 让您能够基于运行属性对运行进行汇总。

当点击 _Runs_ 视图中的一个运行时，会出现一个新的 Foretify Manager 窗口。

<p align="center">
 <a href="images/Run_view_2.png" target="_blank">
 <img src="images/Run_view_2.png">
 </a>
</p>

正如你所见，在这里有两个主要选项卡：**调试运行** 和 **运行摘要**。 
**调试运行** 选项卡与我们之前在 Foretify 中看到过得一样，

<p align="center">
 <a href="images/run_source.png" target="_blank">  
<img src = " images / run_source . png ">  
  </ a>  
</ p>  

通过单击 **Run Summary** 选项卡，您可以查看有关失败运行更详细信息。  
```

```markdown
<p align="center">
 <a href="images/run_summary.png" target="_blank">
 <img src="images/run_summary.png">
 </a>
</p>

!!! Example "实践时间"
 现在通过单击每个运行来探索您的另外两个运行。

### 什么是工作空间以及如何创建它

工作空间是一组导入的测试套件，您希望分析覆盖数据。

!!! 示例 "实践时间"
 在**测试套件结果**选项卡中选择您的测试套件，然后按如下所示单击**创建工作空间**。

<p align="center">
 <a href="images/fmanager_workspace_2.png" target="_blank">
 <img src="images/fmanager_workspace_2.png">
 </a>
</p>

选择一个工作区名称。然后单击**创建工作空间**以创建该工作区。

<p align="center">
 <a href="images/fmanager_workspace_name_2.png" target="_blank">
 <img src="images/fmanager_workspace_name_2.png">
 </a>
</p>

Foretify Manager Web 应用程序切换到 **当前工作区** 视图：

<p align="center">
 <a href="images/fmanager_workspace_after_creation.png" target="_blank"> 
<img src = " images / fmanager_workspace_after_creation.png "> 
</ a> 
</ p>


该工作区包括以下内容：

- **VGrade **是总体度量标准等级（您将在 Lab 4 中了解）。
- **Total Runs（VGrade 旁边）** 是通过和失败运行的统计数据。
- 在 **VPlan** 选项卡（蓝色）中，您可以看到度量标准层次结构。
- 在 **Runs** 选项卡（绿色）中，可以看到当前工作区所选的运行列表。

!!! 示例 “实践时间”
 探索 **VPlan* * 树和 * * 运行* * 标签中的运行。
   
### 度量和检查器表示
```

#### 覆盖率
在我们的`ts_l01_intro_cov.osc`覆盖文件中，我们定义了四个覆盖项和一个KPI指标。
例如，cut_in_side的覆盖等级为100%，而speed_sut的覆盖等级仅为20%。这表明需要运行更多的测试来填补speed_sut覆盖范围中的空白区域。
在我们的文件中，这两个覆盖项的定义略有不同：

- 注意：百分比可能因使用的种子而异！

&lt;p align="center">
  &lt;a href="images/lab01_coverage_conclusions.png" target="_blank">
    &lt;img src="images/lab01_coverage_conclusions.png">
  &lt;/a>
&lt;/p>

对于speed_sut，我们指定了桶，而对于cut_in_side，我们没有强加任何规则。这为cut_in_side覆盖项提供了额外的自由度。该项的覆盖等级为100%，因为在测试期间至少击中了可能的两侧（左侧和右侧）。

!!! 示例 "实践时间"
    现在检查其他覆盖项及其等级，在Foretify Manager中，在VPlan选项卡下。您应该能够找到_cut_in_side_覆盖项。

#### KPIs
正如您所记得的，我们之前定义了

当查看**Runs**选项卡时，您可以观察到有多少次运行通过了，有多少次失败了。失败的运行是来自代码中定义的检查器。由于检查器在SUT未满足布尔条件时定义了一个失败响应，因此这立即反映在问题树中。

<p align="center">
  <a href="images/checkers.png" target="_blank">
    <img src="images/checkers.png">
  </a>
</p>

!!! 例子 "实践时间"
    分析您的运行。您有多少次失败的运行？

### 增加覆盖率

少数测试远远不足以实现任何有意义的覆盖率。Foretellix技术的优势在于运行大量自动生成的测试。因此，在本节中，您将运行更多的测试以增加覆盖范围。为此，您需要根据场景创建不同的测试用例。如上所述，这由创建过程中使用的种子来控制。

**在继续之前，请确保关闭浏览器中的Foretify GUI**。

!!! 例子 "实践时间"
    让我们运行15个额外的测试，设置一个新的工作目录。请注意，我们将不使用foretify gui，因此您只会看到模拟器窗口不时弹出。

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir2 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 20 --crun 15
    ```

    ```--crun```选项将从种子20开始运行15个测试（即种子20到34）。这是扩大测试的一种选项，但在实验室3中，我们将介绍更多选项。

```markdown
注意一下发生式引擎是如何生成 15 个独特的具体测试，每个测试都在地图的不同部分，同时保持所有具体测试在抽象场景定义的边界内（例如速度、车道位置等）。

!!! 示例 "实践时间"
 新运行应该增加覆盖率。现在，您可以使用以下命令将这 15 个额外运行上传到 Foretify 管理器：

 ```bash
 upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
 --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir2
 ```

 然后，在 Foretify 管理器的**测试套结果**选项卡中，选择新上传的测试套，并按照下图所示将它们添加到工作区

 您现在可以探索 **VPlan** 树并查看覆盖率提高了多少。

<p align="center">
 <a href="images/l01_add_ws_1.png" target="_blank">
 <img src="images/l01_add_ws_1.png">
 </a>
</p>

### 在测试套之间切换

当您向工作区添加新的测试套时，您将能够在这些测试套之间进行切换，通过这样做，您可以分析每一个单独地。

!!! 示例 "实践时间"
 转到您的工作区，并单击 **测试套结果工作区视图** 中的向下箭头。

<p align="center">
 <a href="images/l01_switch_ws_1.png" target="_blank">
 <img src="images/l01_switch_ws_1.png">
 </a>
</p>

会出现一个带有可用测试套的新窗口。 您可以在它们之间进行切换以查看差异。

<p align="center">
 <a href="images/l01_switch_ws_2.png" target="_blank">
 <img src="images/l01_switch_ws_2.png" width = "50%">
 </a>
</p>

加载新的测试套后需要单击 **计算** 按钮。
```

```markdown
<p align="center">
 <a href="images/l01_switch_ws_3.png" target="_blank">
 <img src="images/l01_switch_ws_3.png">
 </a>
</p>

### 测试套件分组

当你有多个测试套件时，有时候你希望将它们分组以增加工作空间的覆盖范围。

!!! 例子 "动手操作"
 前往你的工作空间，并点击 **工作空间测试套件结果**，选择测试套件并点击 **分组** 图标进行分组。

<p align="center">
 <a href="images/l01_group_ws_1.png" target="_blank">
 <img src="images/l01_group_ws_1.png">
 </a>
</p>

会出现一个新窗口，请给你的分组命名，并点击 **分组**。

<p align="center">
 <a href="images/l01_group_ws_2.png" target="_blank">
 <img src="images/l01_group_ws_2.png" width="50%">
 </a>
</p>

你会看到运行已更新，覆盖范围也增加了。

<p align="center">
 <a href="images/l01_group_ws_3.png" target="_blank">
 <img src= "l/images/ l 0 1 _ g r o u p _ w s _ 3 . p n g " >
 </ a >
</ p >

### 取消测试套件的分组

也可以取消创建的分组。

!!! 例子 "动手操作"
 前往您的工作空间，并单击 **工作空间测试套件结果**，选择已分组运行，并单击 **群信息** 图标来取消它们。

<p align=“中心”> 
<a href=“ images / l0l / ungroupws1 . png ”target = "_ blank"> 
<img src =“ images / l0l / ungroupws1 . png”> 
</ a > 
</ p >

会出现一个新窗口，请点击 **取消群体** 按钮。

<自适应内容对齐：中心> 
<a href=“图片/101_ungroupws2。png”target = "_ blank">  
<img src = “ imageSIIA101_un group WSZ. PNG ”宽度=”50%">  
< /一 >  

### 更改地图

在继续之前，请确保在浏览器中关闭Foretify。
```

另一种增加覆盖率的方法是更改测试执行的ODD。 OSC2语言的一个强大功能是，为了更改ODD，您只需要更改一行代码。

通过新地图，Foretify将在地图拓扑支持场景的位置随机生成新的具体测试用例。

!!! 示例 "实践时间"
    打开文件`$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`，其中定义了测试。

    编辑`ts_l01_intro.osc`文件中的场景，并将定义地图的行更改为以下内容：

    ```osc linenums="8"
    extend test_config:
        set map = "$FTX_WORKSHOP/maps/cloverleaf.xodr"
    ```
    现在，您可以使用以下命令再次运行测试5次：

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 2 --crun 5
    ```

    您现在可以观察到场景在另一张地图上执行。

!!! 注意

    如果您为特定场景选择的地图不允许执行该场景，则Foretellix工具将指示此为矛盾错误。例如，插入场景无法在单车道上执行。

!!! 示例 "实践时间"
    现在，您已经运行了额外的测试，可以将它们上传到Foretify Manager以查看覆盖率是否有所提高。为此，您可以运行以下命令：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir3
    ```
    将新运行添加到先前创建的工作区中。


## 下一步

本实验室的目标是

- 熟悉用于研讨会的云环境
- 浏览一些用于插入场景的基本OSC2代码
- 以交互模式运行Foretify并加载插入场景
- 熟悉种子的概念
- 熟悉测试中的OSC2及其主要组件
- 打开Foretify Manager并探索收集到的度量标准

接下来，在实验室2中，您将扩展插入场景，收集更多覆盖度量标准，并介绍在Foretify中调试场景的方法

> 本文由ChatGPT翻译，如有任何遗漏，请[**反馈**](https://github.com/linyuxuanlin/Wiki_MkDocs/issues/new)。