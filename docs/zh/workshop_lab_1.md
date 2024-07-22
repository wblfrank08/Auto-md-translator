---
chapnum: 1
---

# 实验1：使用OpenScenario 2和Foretellix技术的第一步

## 学习目标

本实验介绍以下内容：

- 简单的**ASAM OpenSCENARIO 2.0（OSC2）**测试实现。 （ASAM是自动化和测量系统标准化协会。）

!!! 注意
    在Foretellix文档中，“OSC2”一词始终指“ASAM OpenSCENARIO® DSL版本2.x”。

- **Foretify™**，用于：
    - 编译OSC2源
    - 从抽象场景定义生成具体测试
    - 在测试执行期间控制模拟平台中的参与者
    - 绘制和可视化测试执行结果的场景开发和测试自动化平台

- **Foretify Manager**，主要用于：
    - 收集多个测试执行的关键绩效指标（KPI）、检查器消息和覆盖度指标
    - 可视化测试进度，启用安全驱动验证（SDV）方法论

<p align="center">
  <a href="images/l01_ftx_diagram.png" target="_blank">
    <img src="images/l01_ftx_diagram.png">
  </a>
</p>

!!! 注意
    单击任何图像都会在新标签页中以完整分辨率打开它。

## OSC2语言

在AD和ADAS功能的开发和验证过程中，有必要使用各种*场景*来刺激被测系统（SUT）。场景是一个或多个参与者（如汽车、行人、环境条件和SUT本身）按时间顺序执行的动作序列。 OSC2是一种领域特定语言，专门用于描述参与者在环境中移动的场景。这些场景具有属性，允许您约束参与者类型、它们的移动以及环境（包括场景应发生的地图位置）。

!!! Info
    采用*约束随机*方法，未受约束的每个场景属性都是随机的。例如，如果您不将切入场景的“侧面”属性约束为“右侧”，则它将从可能属性的空间（即“左侧”和“右侧”）中随机选择。

    此外，值是从可能属性的空间中选择的，以满足从场景、参与者和地图中导出的约束条件。例如，如果SUT（EGO）正在驾驶2条车道的最右侧车道上，那么切入“侧面”属性将不会选择为“右侧”。

OSC2的构建块是数据结构，例如：

- **Actors**: 代表现实世界实体。正如名称所示，它们在场景中“扮演角色”。
- **Scenarios** 或 **Actions**: 描述了 actors 的行为。通常，一个场景是一系列行动的长序列，但两者之间没有正式区别。两者都可以通过 *modifiers* 进行修改。
- **Modifiers**: 对场景添加约束条件，有助于控制其在期望范围内的执行。
- **Labels**: 定义了任何标量、结构或 actor 类型的命名数据字段。
- **Simple structs**: 是包含属性、约束等基本实体。

您可以通过参考 Foretellix [OSC2 语言文档]（../osc_lang/osclang_intro.md）和[OSC 领域模型文档]（../osc_dom/oscdomain_intro.md）来进一步了解上述主题，或者访问 [ASAM 类型定义主题](https://www.asam.net/index.php?eID=dumpFile&t=f&f=3460&token=14e7c7fab9c9b75118bb4939c725738fa0521fe9#type-definitions) 或主要的 [ASAM OSC2 网页](https://www.asam.net/project-detail/asam-openscenario-v20-1/)。

!!! Info
Foretellix 工具**原生支持 ASAM OSC2 语言**，带来许多好处之一就是该语言和工具都是**执行平台无关**的。这导致在更改验证环境时减少工作量，并且在选择合适的验证平台时增加了灵活性。

OSC2 支持**抽象场景**和**安全驱动验证流程**。通过本研讨会，您将了解这些功能如何能够优化 V&V 工作，从而减少验证自主系统所需的资源。

### 我们的第一个测试

测试是从中调用场景的 OSC2 代码，在层级上被视为顶层：

1. 它导入了测试执行平台配置（例如模拟器）。

2. 它导入了SUT配置（例如被测试系统，通常称为EGO）。在这种情况下，作为被测试的功能，我们导入了由Foretellix开发的SUT L4堆栈，并配置了EGO车辆的属性。

3. 它设置要在测试中使用的地图。

4. 它定义了场景和指标（检查、覆盖率、KPIs），并调用了场景执行。

在下图中，您可以看到将要运行的测试结构：

&lt;p align="center">
  &lt;a href="images/Test_2.png" target="_blank">
    &lt;img src="images/Test_2.png">
  &lt;/a>
&lt;/p>

!!! 示例 "实践时间"
    现在您已准备开始研讨会，是时候更详细地查看第一个测试了。

    使用Foretify Developer OSC2代码编译器打开第一个测试：

    ```bash
    foretify --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --gui
    ```

Foretify GUI 将出现在您的浏览器中。在本次培训中，您将学习更多功能。现在您将专注于其OSC2代码编译和可视化功能。单击**Source**选项卡，并

我们现在将逐步介绍导入语句的内容，以及以下各节中的其余代码。

### _workshop_config.osc_ 配置文件

`workshop_config.osc` 文件（在测试文件的第3行导入）包含了设置 Foretify 和执行平台连接的所有定义（它可以使用不同的模拟器），以及系统测试（SUT）连接的定义（在本例中是自动驾驶的 Ego）。

### _cut_in_l01.osc_ 场景文件

这个 `cut_in_l01.osc` 文件在第4行导入，并包含了一个切入场景的抽象定义，也就是本实验的主题。通过一组绝对和相对约束的定义，我们正在定义一个车辆应该在 SUT 前面变道的情况。OSC2 是唯一支持抽象的场景描述语言，极大地减少了场景开发人员编写代码所需的时间。
在下面的图片中，您可以看到抽象切入场景定义的两个具体实例。

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_scenario.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_scenario.png">
  </a>
</p>

!!! 示例 "实践时间"
    打开通过点击源代码选项卡中测试文件的第4行导入的 `cut_in_l01.osc` 文件。

下面是实现切入场景的代码：

<p align="center">
  <a href="images/workshop_l01_cut_in_code_vscode.png" target="_blank">
    <img src="images/workshop_l01_cut_in_code_vscode.png">
  </a>
</p>

让我们逐步了解代码：

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
    - 第28行：将速度约束覆盖为非关键的、最佳努力的约束。
    - 第29行：约束car1在场景阶段结束时与SUT在同一车道上。
    - 第30行：car1的速度在整个场景阶段保持恒定。
    - 第31行：再次，这个速度约束被定义为非关键的，但要尽力执行。
    - 第32行：此行禁用了car1的碰撞避免行为，从而使车道变更对SUT具有挑战性。您将在高级实验室中了解更多关于通用汽车角色的碰撞避免行为的内容。

### 行为监控

以下三个部分将简要介绍 _覆盖率（coverage）_、_KPI 或记录（KPI or record）_ 和 _检查器（checker）_ 的概念，并结合一些例子进行说明。它们是“以安全为导向的验证方法”的关键支柱，因为它们支持验证过程并揭示系统下的不正确表现。在下一个实验中，您将深入探讨并详细阐述它们的功能。

#### _ts_l01_intro_cov.osc_ 覆盖率定义

!!! 例子 "亲自动手时间"
    打开 `ts_l01_intro_cov.osc` 文件，通过点击“源代码”选项卡中 `ts_l01_intro.osc` 测试文件的第5行导入该文件。

&lt;p align="center">
  &lt;a href="images/workshop_lab_1_cut_in_lab_1_osc_cover.png" target="_blank">
    &lt;img src="images/workshop_lab_1_cut_in_lab_1_osc_cover.png">
  &lt;/a>
&lt;/p>

上面的代码定义了 **覆盖率收集**：

- 第一个指标（第4行）是切入发生的侧面——左侧还是右侧。
- 第二个指标（第5行）是执行切入操作的车辆类型，例如卡车或轿车。
- 接下来的指标（第6到11行）是车辆在变道场景阶段结束时的行驶速度。
- 最后一个指标（第12到18行）是主要场景开始时两

#### _ts_l01_intro_cov.osc_ KPI定义

KPI在与前一部分的覆盖项相同的文件中定义。

在代码的这一部分，我们定义了一个KPI：

- 首先，在第21到22行声明了一个变量，用于采样SUT和切入车辆之间的距离，该距离是在变道结束时测量的。
- 第25到27行使用_record()_方法，将KPI记录下来，以便稍后进行可视化。

!!! 信息
    现在您已经浏览了一些示例，了解了覆盖度指标和性能指标之间的主要区别是非常重要的：

    - **覆盖度评估**：_我们在“场景空间”的哪个部分测试了我们的AV？_ 这通过覆盖度和总体覆盖度等级来表示。覆盖项的定义是为了支持覆盖度评估。换句话说，覆盖度等级回答了这个问题：_SUT的测试效果如何？_
    - **性能评估**：_SUT在测试中表现如何？_ 这个问题通过性能等级来回答，可以是一个或多个KPI。

#### _ts_l01_intro_checks.osc_ 检查器定义

!!! 示例 "实践时间"
    在源代码选项卡中，点击`ts_l01_intro.osc`测试文件的第6行，打开导入的`ts_l01_intro_checks.osc`文件。

<p align="center">
  <a href="images/l01_kpi_code.png" target="_blank">
    <img src="images/l01_kpi_code.png">
  </a>
</p>

该检查器的目的是评估SUT和切入车辆之间的距离是否在定义的安全距离范围内，在整个场景中进行评估。

- 第1行，issue_kind类型正在使用唯一名称（safety_distance）对新检查器进行扩展。
- 第3到12行扩展了先前定义的场景以编写检查程序如下：
  - 第5至6行：声明了安全距离阈值的变量，并设置为13米
  - 第8至10行在仿真的每个时间步骤（top.clk）上验证以下内容：
    - 两辆车之间的距离是否超过了定义的阈值
    - 两辆车是否在同一车道上
  - 第11至12行：如果上述条件不满足，则用sut_error类型和safety_distance子类型停止测试运行，并显示错误信息。

!!! Info

- 刚刚添加的检查器是用户自定义的，但Foretify带有内置检查器。您可以在[Global checkers for vehicles documentation](../osc_dom/oscdomain_metrics.md#global-checkers-for-vehicles)中查看这些内容。
  
- 行业中还使用evaluator术语来表示检查器。

- 使用检查程序，您可以表示场景的成功或失败标准，并且它们使用与表示动作和参与者相同的语言（OSC2）编写。

- 在使用Foretify时，某些预定义检查程序始终处于活动状态，例如碰撞检测或SUT偏离道路驾驶等默认情况都可进行修改。

- 您可以定义自定义检查程序来捕获特定于您SUT的问题，就像我们示例中所做。我们所执行的检查器使用KPI值和预定义阈值来评估SUT的预期行为，但这只是一种定义方式而已。

### 地图定义

在下面代码（`ts_l01_intro.osc`测试文件第8和9行），我们设置要使用地图，该地图采用OpenDrive格式（即\*.xodr）。

```osc linenums="8"
扩展测试配置：
    设置地图 = "$FTX_WORKSHOP/maps/Town04.xodr"
```

### 场景执行

在`ts_l01_intro.osc`文件的最后两行中，我们最终调用_cut_in_l01_场景：OSC2程序的入口点总是执行_top.main_场景，类似于C/C++的_main()函数：

```osc linenums="11"
扩展 top.main：
    执行 cil : sut.cut_in_l01()
```
您可以在测试文件的第11和12行找到此代码。

## 首次运行和Foretify

### 有约束的随机生成和自适应场景执行

正如其名称所示，*有约束的随机生成* 意味着在指定约束空间内生成随机变量。这是SDV（安全驱动验证）流程的基本前提，也是Foretify构建的核心原则。

从OSC2抽象场景中，Foretify的生成引擎根据指定的约束随机创建具体场景。其中一个最重要的方面是被随机化的是每个场景将展开的地图区域。

一旦为场景制定了计划，运行时自适应场景执行引擎将根据场景计划处理执行。

!!! 信息
    **有约束的随机生成** 引擎是Foretellix解决方案的主要支柱。这是一个非常强大的工具，可以从抽象描述中生成出有意义的场景变化。

    当利用这项技术时

将生成的场景在执行时，有助于挑战测试系统，以便更高效地**发现和解决错误**。

### 第一次运行

**在继续之前，请确保您已关闭浏览器中的Foretify GUI。**

现在您已经浏览了OSC2场景定义，将使用模拟器作为您的测试执行平台。

现在您将更详细地探索Foretify GUI，以加载、启动和分析测试。稍后您将探索Foretify的更多功能。

!!! 示例 "实践时间"
    在GUI模式下启动Foretify，并加载之前检查过的测试：

    ```bash
    foretify --gui --work_dir $FTX_FM_WORKDIR/l01_intro/workdir \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc
    ```

Foretify交互窗口出现：

&lt;p align="center">
  &lt;a href="images/foretify_gui.png" target="_blank">
    &lt;img src="images/foretify_gui.png">
  &lt;/a>
&lt;/p>

Foretify窗口显示以下信息：

- **加载**、**准备测试**和**调试**选项卡（左上角）：
    - **加载**选项卡用于通过图形界面加载`.osc`文件（我们没有使用图形界面，因为我们使用终端命令加载了文件）。
    - **准备测试**选项卡允许您设置运行的不同参数。您将在后面的部分了解更多信息。
    - **调试**选项卡用于在模拟完成后调试运行。
- **状态**（左上角中间）：
    - 您可以看到加载的文件、加载状态以及加载过程中发现的问题数量。
- **地图**、**源代码**和**预览**选项卡（左侧中间）：
    - **地图**选项卡允许您浏览加载的地图，探索地图的不同图层。
    - **源代码**选项卡显示加载的源文件，允许您查看加载的代码并在加载的文件之间切换。
    - **预览**选项卡允许您在**执行控制**区域点击**预览**按钮后预览计划的测试。
- **问题**、**代码检查违规**和**测试信息**选项卡（左下角）：
    - 这些选项卡在加载场景后显示。
- **控制场景的执行**（右上角）：
    - 您可以设置运行的种子，预览运行或运行实际模拟。种子是每个具体测试执行的唯一标识符，根据抽象定义生成。
- **日志**（右侧中间）

!!! 示例 "实际操作时间"
在实际运行测试之前，您可以通过单击**预览**按钮来尝试一下，该按钮会显示SUT和car1在可视化器中的计划路径。请注意，由于SUT的行为，计划路径可能会在运行时发生变化，但这是一个非常有用的调试工具。请注意，计划路径只是为该场景创建的计划的表示形式：真实轨迹是在运行时计算和更新的。

!!! 示例 "实际操作时间"
在运行测试之前，尝试通过单击右上角的灰色**预览**按钮并更改种子编号来预览您的测试的几个种子。

!!! 示例 "实际操作时间"
通过单击右上角的紫色**运行测试**按钮来运行测试。您也可以通过单击右上角的**终端**按钮并输入_run_来运行测试。

您将看到模拟器窗口弹出：

- Foretify约束随机生成引擎根据在场景的OSC2代码中指定的约束和参数生成了特定的场景变体。当预览运行时，演员的计划路径就是您所看到的路径。

- Foretify运行时测试编排引擎确保测试执行平台（模拟器）中的演员能够根据指定的约束移动。

运行时测试编排引擎确保**自适应场景执行**，这意味着SUT偏离计划路径将导致NPC采取相应的对策，以满足OSC2文件中指定的场景意图。

测试完成后，日志区域显示两个额外的选项卡：

将 **Trace Details** 选项卡，让你检查方案跟踪信息以及在模拟过程中收集到的值，比如跟踪类型、执行者、时间和持续时间。

**Log** 选项卡包含测试执行期间生成的日志。

**Metrics** 选项卡与覆盖率相关，稍后将在研讨会上进行详细介绍。

!!! 示例 "实操时间"
 现在，请查看 **Log** 选项卡，并检查是否可以看到我们添加的日志消息，以显示切入发生在哪一侧。

### 调试运行

#### 使用 Foretify Visualizer 进行调试

运行测试之后，在屏幕中央，你将可以访问 **Visualizer** 选项卡。Visualizer 是一种图形后处理工具之一，可以以各种方式进行配置，帮助你分析执行情况。可视化运行与重复执行不同。它不需要模拟器，并且不会消耗重新运行所需的计算资源。

当执行完成时，屏幕左侧的 **Visualizer** 选项卡将自动打开。
随时单击该选项卡即可返回 Visualizer。

<p align="center">
 <a href="images/visualizer.png" target="_blank">
 <img src="images/visualizer.png">
 </a>
</p>

##### 回放测试
首先，在 Visualizer 时间线底部左侧按下播放按钮，并观察场景是如何回放的:

<p align="center">
 <a href="images/visualizer_play_button.png" target="_blank">
 <img src="images/visualizer_play_button.png">
 </a>
</p>

##### 地图视角
您可以通过在可视化器中单击鼠标右键并将光标拖动到不同的方向来更改视角。您可以通过单击可视化器右上角的扳手图标来访问**视图工具**。**视图工具**具有控制视图的选项：

<p align="center">
  <a href="images/visualizer_tools.png" target="_blank">
    <img src="images/visualizer_tools.png">
  </a>
</p>

- 车道方向：启用或禁用驾驶方向箭头的查看。
- 信号：切换交通信号的可见性。
- 速度限制：显示道路的速度限制。
- 碰撞避免：启用碰撞避免激活的可见性。
- 运行时轨迹：显示所选车辆角色的轨迹。
- 计划路径：突出显示场景中存在的车辆的路径。
- 计划目标：显示为所选车辆角色生成的计划目标。
- 计划姿态：突出显示SUT和其他车辆的下一个姿态。
- 驾驶员目标：显示为所选车辆角色生成的驾驶目标。
- 预测姿态：显示车辆的预测位置。

##### 相机设置
要控制视角和相机，请单击相机设置（相机）图标。要使用相机跟踪特定角色，请从左上角（i）下拉列表中选择角色或单击可视化器以将相机设置为固定位置。

<p align="center">
  <a href="images/Camera_settings.png" target="_blank">
    <img src="images/Camera_settings.png">
  </a>
</p>

- 透视视图：将视角更改为俯视图。
- 跟踪所选角色：跟踪所选角色。
- 将相机重置为所选角色：将相机重置为所选角色的位置。

##### 测量距离

- 如果Visualizer处于透视视图中，请在Visualizer右上角选择相机设置图标，并关闭透视视图选项。
<p align="center">
 <a href="images/Measurement_1.png" target="_blank">
 <img src="images/Measurement_1.png">
 </a>
</p>

- 选择测量距离工具图标。
<p align="center">
 <a href="images/Measurement_2.png" target="_blank">
 <img src="images/Measurement_2.png">
 </a>
</p>

- 在Visualizer中点击以设置测量起点，然后移动光标并点击以设置测量终点。
<p align="center">
 <a href="images/Measurement_3.png" target="_blank">
 <img src="images/Measurement_3.png">
 </a>
</p>

测量结果将显示在连接起点和终点的线旁边。

- 若要隐藏测量结果，请切换关闭“测量距离”工具图标。

#### 使用跟踪进行调试

所有跟踪都可以在“跟踪”视图下查看。该视图与时间轴对齐，因此您可以轻松比较不同的跟踪。

**跟踪** 有以下几种不同类型：

- **间隔**: 表示一段时间内收集的值集合。间隔具有名称、开始/结束时间和类型，并与特定的参与者相关联（橙色方框）。

- **数值**: 表示随时间变化的单个值，数值显示为波形图形式。数值跟踪具有名称、数值和单位，并与特定参与者相关联（红色方框）。

<p align="center"> 
<a href = "images / Traces_1 .png" target = "_ blank"> 
<img src = " images / Traces _1 .png ">
</ a> 
</ p> 

为了增强可视化效果，Foretify记录每个场景的开始和结束时间。

##### 查看间隔：
 - 在Foretify中选择“调试运行”选项卡, 然后单击 Visualizer 下面 的 “Traces”选项卡:

```markdown
<p align="center">
 <a href="images/Traces_2.png" target="_blank">
 <img src="images/Traces_2.png">
 </a>
</p>

轨迹显示为时间轴上的间隔，当前时间光标对应于其他基于时间的视图，如 Visualizer 和 Traces 选项卡中 Actor 内部的 Actor 值轨迹。

1. 单击跟踪名称左侧的箭头以展开并查看其子场景。

2. 单击跟踪以查看跟踪详细信息，例如跟踪类型、Actor、时间、持续时间和间隔期间收集的指标。

3. 在跟踪详情下，单击跟踪的开始时间或结束时间将通用时间轴设置为该时刻。

##### 将时间线框定到轨迹：
1. 在 Traces 选项卡上，选择要在 Timeline 中框定的间隔（橙色框）。

2. 单击框架时间线图标（红色框）。

<p align="center">
 <a href="images/Intervals_3.png" target="_blank">
 <img src="images/Intervals_3.png">
 </a>
</p>

- 要重置 Timeline 以使其不再框定该间隔，请单击 Timeline 右侧的取消框架图标。 

<p align="center">
 <a href="images/Intervals_4.png" target="_blank"> 
<img src="images/Intervals_4.png"> 
</a>
</p>

!!! 示例 "实际操作"
 使用 Visualizer 重新运行测试，并检查 SUT 的行为及上述不同选项。 

### 运行不同种子

!!! 示例 "实际操作"
 使用右上角执行控制区域通过设置种子号为 4 并单击 **运行测试** 按钮来运行另一个模拟。这是一个种子，在这个种子下我们引入了最小距离阈值检查器会失败。 

使用自己选择的种子再次进行模拟。
  
再次搜索日志以找到指示割入发生在哪一侧消息。
```

```markdown
现在你可以通过在Foretify终端中输入exit或关闭Foretify窗口来关闭Foretify。

!!! 信息
    **种子**用作对一个具体的执行场景定义进行随机生成的输入。使用相同的**种子**再次根据抽象定义生成具体测试会得到相同的具体变化。这是一个关键特性，一方面让你完全随机化测试，另一方面也可以为调试目的重新创建单个具体执行。

    由于种子的定义和实施，你可以始终**追踪生成的特定具体变化**。场景的可追溯性是一个基本功能，它让你识别导致错误的条件并重现它们。

## Foretify Manager

现在你已经运行了第一批测试，你可以探索达到的覆盖率和结果。我们将在Lab 2中正式定义覆盖率，但现在你可以使用Foretellix工具直观地探索这个概念。

Foretify Manager是Foretellix工具，它让你可视化验证过程的整体状态，导入多个测试执行的结果和KPIs。它有一个客户端-服务器架构，如下图所示：

&lt;p align="center">
  &

### 创建项目

一个 Foretify Manager 项目是一个协作框架，使用户能够为验证和验证数据以及收集的指标设置权限和所有权。

要创建一个项目：

1. 打开 Foretify Manager 并使用 Foretify Manager 凭据登录。

2. 在“选择项目”页面上，点击“创建新项目”。

!!! 信息
    关于您的项目名称，请联系主持人获取更多信息

- 点击“创建”紫色按钮

&lt;p align="center">
  &lt;a href="images/fmanager_project.png" target="_blank">
    &lt;img src="images/fmanager_project.png">
  &lt;/a>
&lt;/p>

&lt;p align="center">
  &lt;a href="images/fmanager_project_2.png" target="_blank">
    &lt;img src="images/fmanager_project_2.png">
  &lt;/a>
&lt;/p>

### 上传测试套件结果

此时，Foretify Manager 数据库为空，所以下一步是上传您执行的少数测试结果。

!!! 示例 "实践时间"
    切换到终端，并通过以下命令上传结果：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX

### 现场操作时间

切换回你浏览器中打开的Foretify Manager网络应用，并点击刷新按钮（确保在"测试套件结果"选项卡上）。

上传后，测试执行应该会加载并显示在你之前创建的项目中：

&lt;p align="center">
  &lt;a href="images/fmanager_regression_2.png" target="_blank">
    &lt;img src="images/fmanager_regression_2.png">
  &lt;/a>
&lt;/p>

### 分析已上传的运行

通过点击刚刚导入的测试套件，你可以查看各个运行情况。

你应该会看到类似下图的内容：

&lt;p align="center">
  &lt;a href="images/fmanager_runs.png" target="_blank">
    &lt;img src="images/fmanager_runs.png">
  &lt;/a>
&lt;/p>

彩色方块表示：

- _黄色_: 运行列表右上角的图标允许你导出和删除运行，以及保留和重置你的选择。使用最右侧的列选择图标来添加和移除运行属性，例如目录、操作系统用户、持续时间等。
- _橙色_: _问题树_按种类对所有问题进行分组展示。
- _蓝色_: _聚合视图_允

```markdown
<p align="center">
 <a href="images/run_summary.png" target="_blank">
 <img src="images/run_summary.png">
 </a>
</p>

!!! Example "Hands-on Time"
 现在通过单击每个运行来探索另外两个运行。

### 什么是工作区以及如何创建它

工作区是一组导入的测试套件，您想要分析覆盖率数据。

!!! Example "Hands-on Time"
 在**测试套件结果**选项卡中选择您的测试套件，然后按照下面所示单击**创建工作区**。

<p align="center">
 <a href="images/fmanager_workspace_2.png" target="_blank">
 <img src="images/fmanager_workspace_2.png">
 </a>
</p>

选择一个工作区名称。然后单击**创建工作区**来创建该工作区。

<p align="center">
 <a href="images/fmanager_workspace_name_2.png" target="_blank">
 <img src="images/fmanager_workspace_name_2.png">
 </a>
</p>

Foretify Manager Web 应用程序切换到 **当前工作空间** 视图：

<p align="center"> 
<a href = "images/fmanager_workspace_after_creation.png" target = "_ blank"> 
<img src = " images / fmanager_workspace_after_creation.png "> 
</ a> 
</ p> 

该工作空间包括以下内容：

- **VGrade**(您将在第四实验了解) 是总体指标等级。
- 除了 **VGrade**(旁边)，还有 **Total Runs**(总运行次数)，显示已通过和未通过的运行次数统计信息。
- 在 **VPlan**(蓝色) 选项卡中，您可以查看指标层次结构。
- 在 **Runs**(绿色) 选项卡中，您可以查看当前 工作空间选择的运行列表。

!!! Example "Hands-on Time"
 探索在 **VPlan** 树和在 **Runs(运行)** 选项卡中的各种指标与检测器表示形式。
```

#### 覆盖率
在我们的 `ts_l01_intro_cov.osc` 覆盖文件中，我们定义了四个覆盖项和一个关键绩效指标（KPI）。例如，cut_in_side 的覆盖等级为100%，而 speed_sut 的覆盖等级仅为20%。这表明需要运行更多的测试来填充 speed_sut 覆盖的空白桶。
在我们的文件中，这两个覆盖项的定义略有不同：

- 注意：百分比可能因使用的种子而不同！

<p align="center">
  <a href="images/lab01_coverage_conclusions.png" target="_blank">
    <img src="images/lab01_coverage_conclusions.png">
  </a>
</p>

对于 speed_sut，我们指定了桶，而对于 cut_in_side，我们没有强加任何规则。这给了 cut_in_side 覆盖项额外的自由度。该项的覆盖等级为100%，因为在测试过程中至少命中了两个可能的方向（左侧和右侧）。

!!! 示例 "实践时间"
    现在检查其他覆盖项及其在 foretify 管理器中的等级，在 VPlan 选项卡下可以找到 _cut_in_side_ 覆盖项。

#### 关键绩效指标（KPI）
正如您记得的那样，我们之前定义了 distance_kpi 关键绩效指标，它测量了 SUT 和 cut-in 车辆在变道结束事件时的距离。指定事件有助于我们更好地捕捉兴趣点的最高峰值。由于事件也被指定了，我们注意到该 KPI 每次运行只有单个值。

<p align="center">
  <a href="images/workspace_kpi_2.png" target="_blank">
    <img src="images/workspace_kpi_2.png">
  </a>
</p>

!!! 示例 "实践时间"
    您得到了哪些 KPI 值？您可以在可视化工具中查看模拟结果，了解值与模拟之间的相关性。

#### 检查器

当查看**Runs**选项卡时，您可以观察到通过了多少次运行和失败了多少次。失败的运行来自代码中定义的检查器。由于检查器在系统未能满足布尔条件时定义了一个失败响应，这立即反映在问题树中。

&lt;p align="center">
  &lt;a href="images/checkers.png" target="_blank">
    &lt;img src="images/checkers.png">
  &lt;/a>
&lt;/p>

!!! 示例 "实践时间"
    分析您的运行。有多少次运行失败了？

### 增加覆盖范围

仅进行几项测试远远不足以实现任何有意义的覆盖范围。Foretellix 技术的强大之处在于运行大量自动生成的测试。因此，在本节中，您将运行更多的测试以增加覆盖范围。为此，您需要根据场景创建不同的测试用例。如上所述，这由创建过程中使用的种子控制。

**在继续之前，请确保关闭浏览器中的 Foretify GUI**。

!!! 示例 "实践时间"
    让我们再运行 15 次测试，设置一个新的工作目录。请注意，我们将不使用 foretify gui，因此您将只会看到模拟器窗口不时弹出。

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir2 \
    --load $FTX_WORKSHOP

请注意，生成引擎生成了15个独特的具体测试，每个测试都位于地图的不同部分，同时保持所有具体测试在抽象场景定义的边界内（例如速度、车道位置等）。

!!! 示例 "实践时间"
    新的运行应该增加覆盖率。您现在可以使用以下命令将这15个额外的运行上传到Foretify Manager：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir2
    ```

    然后，在Foretify Manager的**测试套件结果**选项卡中，选择新上传的测试套件，并将其添加到工作区，如下图所示：

    您现在可以浏览**VPlan**树，看看覆盖率提高了多少。

<p align="center">
  <a href="images/l01_add_ws_1.png" target="_blank">
    <img src="images/l01_add_ws_1.png">
  </a>
</p>

### 在测试套件之间切换

当您将新的测试套件添加到工作区时，您将能够在这些测试套件之间进行切换，通过这样做，您可以分别分析每个测试套件。

!!! 示例 "实践时间"
    转到您的工作区，然后单击**测试套件结果工作区视图**中的向下箭头。

<p align="center">
  <a href="images/l01_switch_ws_1.png" target="_blank">
    <img src="images/l01_switch_ws_1.png">
  </a>
</p>

将会出现一个新窗口，显示可用的测试套件。您可以在它们之间切换以查看差异。

<p align="center">
  <a href="images/l01_switch_ws_2.png" target="_blank">
    <img src="images/l01_switch_ws_2.png" width="50%">
  </a>
</p>

在加载新的测试套件后，您需要点击**计算**按钮。

```markdown
<p align="center">
 <a href="images/l01_switch_ws_3.png" target="_blank">
 <img src="images/l01_switch_ws_3.png">
 </a>
</p>

### 分组测试套件

当您有多个测试套件时，有时候您可能想要将它们分组，以增加工作区的覆盖范围。

!!! 例子 "实践时间"
 前往您的工作区，并点击 **工作区测试套件结果**，选择测试套件，然后点击 **分组** 图标将它们分组。

<p align="center">
 <a href="images/l01_group_ws_1.png" target="_blank">
 <img src="images/l01_group_ws_1.png">
 </a>
</p>

会出现一个新窗口，请为您的分组命名并点击 **分组**。

<p align="center">
 <a href="images/l01_group_ws_2.png" target="_blank">
 <img src="images/l01_group_ws_2.png" width = "50%">
 </a>
</p>

您可以看到运行情况已更新，并且覆盖范围已扩大。

<p align="center">
 <a href = "images / l01_group _ws _3 .png" target = "_blank ">
<img src =" images / l01_group _ws _3 .png ">
 </ a>
</ p>

### 取消分组测试套件

也可以取消创建的分组。

!!! 例子 “实践时间”
 前往您的工作区，并点击 **工作区测试套件结果** ，选择已分组运行，并单击 **群 组信息** 图标以取消其分 组。
 
<p align =" center ">
<a href =“ images / l01_un group ws 1 .png”target=“ _ blank ”> 
<img src=“ images / l 0 ungroup ws 1 png ”> 
</ a> 
</ p> 

会出现一个新窗口，请单击 **取消群 组* * 按 钮。 

<p 对齐=”中心”>
<a href=“图像/ l0_un g roup ws2. P n g” 目标='_空白' >
<Img Src='图像/ L O ungr oupw s 步 ' width ='50%'>
 </A > 
至于地图进行更改

在进行下一步之前，请确保在浏览器中关闭 Foretify。
```

另一种增加覆盖率的方法是改变测试执行时的ODD。OSC2语言的一个强大特性是，为了改变ODD，你只需要改变一行代码。

通过一个新的地图，Foretify将在地图拓扑支持的位置随机生成新的具体测试用例。

!!! 示例 "实践时间"
    打开文件`$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`，其中定义了测试。

    编辑`ts_l01_intro.osc`文件中的场景，并将定义地图的那一行代码更改为以下内容：

    ```osc linenums="8"
    extend test_config:
        set map = "$FTX_WORKSHOP/maps/cloverleaf.xodr"
    ```
    现在，你可以使用以下命令再次运行测试5次：

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 2 --crun 5
    ```

现在你可以观察到场景在另一个地图上执行。

!!! 注意

    如果你为特定场景选择的地图不允许执行该场景，Foretellix工具将指示这是一个矛盾错误。例如，一个插入场景不能在单车道上执行。

!!! 示例 "实践时间"
    现在你已经运行了额外的测试，你可以将它们上传到Foretify Manager，以查看覆盖率是否有所改善。为了做到这一点，你可以运行以下命令：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir3
    ```
    将新的运行结果添加到之前创建的工作空间中。


## 下一步

本实验室的目标是

- 熟悉用于工作坊的云环境
- 浏览一些用于插入场景的基本OSC2代码
- 在交互模式下运行Foretify并加载插入场景
- 熟悉种子的概念
- 熟悉用于测试的OSC2及其主要组件
- 打开Foretify Manager并探索收集的指标

接下来，在实验室2中，您将扩展插入场景，收集更多的覆盖度指标，并介绍在Foretify中调试场景的方法。

> 本文由ChatGPT翻译，如有任何遗漏，请[**反馈**](https://github.com/linyuxuanlin/Wiki_MkDocs/issues/new)。