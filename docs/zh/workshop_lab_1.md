---
chapnum: 1
---

# Lab 1: 使用 OpenScenario 2 和 Foretellix 技术的第一步

## 学习目标

本实验室将介绍以下内容：

- 一个简单的 **ASAM OpenSCENARIO 2.0 (OSC2)** 测试实现。 (ASAM 是自动化和测量系统标准化协会。)

!!! 注意
    在 Foretellix 文档中，“OSC2” 一词指的是 “ASAM OpenSCENARIO® DSL 版本 2.x”。

- **Foretify™**，这是一个**场景开发和测试自动化平台**，用于：
    - 编译 OSC2 源
    - 从抽象场景定义中生成具体测试
    - 在测试执行期间控制仿真平台中的参与者
    - 绘图和可视化测试执行结果

- **Foretify Manager**，这是一个**主要用于大数据分析的平台**，主要用于：
    - 收集多个测试执行的关键绩效指标 (KPIs)、检查器消息和覆盖度指标
    - 通过可视化在测试中的进展，实现面向安全的验证（SDV）方法论

<p align = "center">
  <a href = "images/l01_ftx_diagram.png" target = "_blank">
    <img src = "images/l01_ftx_diagram.png">
  </a>
</p>

!!! 注意
    单击任何图像会在新标签页中以完整分辨率打开它。

## OSC2 语言

在 AD 和 ADAS 功能的开发和验证过程中，有必要用各种*场景*激发被测试系统 (SUT)。场景是一个或多个参与者（如汽车、行人、环境条件和 SUT 本身）按时间顺序执行的一系列动作。OSC2 是一种特定领域的语言，专门用于描述参与者在环境中移动的场景。这些场景具有属性，可以约束参与者类型、他们的移动方式以及环境（包括场景所发生的地图位置）。

!!! 信息
    采用*约束随机*方式时，每个未约束的场景属性都会被随机化。例如，如果未约束切入场景中的“side”属性为“right”，则它将从可能属性（即“left”和“right”）的空间中随机选择。

    此外，值是从可能属性的空间中选择的，以满足源自场景、参与者和地图的约束条件。举个例子，如果 SUT（EGO）正在沿着二车道道路的最右车道行驶，切入的“side”属性不会被选择为“right”。

OSC2 的基本构建模块是数据结构，例如：

- **演员**：代表现实世界的实体。顾名思义，他们在情景中“扮演角色”。
- **情景**或**动作**：描述演员的行为。一般情况下，情景是一系列动作的较长序列，但两者之间没有正式区别。两者都可以通过*修饰符*进行修改。
- **修饰符**：为情景增加约束条件，帮助控制其在所需范围内的执行。
- **标签**：定义任何标量、结构或演员类型的命名数据字段。
- **简单结构**：是包含属性、约束等基本实体。

您可以通过参考Foretellix [OSC2语言文档](../osc_lang/osclang_intro.md)和[OSC领域模型文档](../osc_dom/oscdomain_intro.md)来了解上述主题，或者访问[ASAM类型定义主题](https://www.asam.net/index.php?eID=dumpFile&t=f&f=3460&token=14e7c7fab9c9b75118bb4939c725738fa0521fe9#type-definitions)或主[ASAM OSC2网页](https://www.asam.net/project-detail/asam-openscenario-v20-1/)。

!!! 信息
    Foretellix工具 **原生支持ASAM OSC2语言**，带来许多好处，其中之一是语言和工具是**执行平台无关的**。这导致在更改验证环境时**减少了工作量**，同时在选择适当的验证平台时带来**更大的灵活性**。

    OSC2支持**抽象情景**和**安全驱动验证**流程。通过本研讨会，您将了解这些功能如何**简化V&V工作**，从而**减少验证自主系统所需的资源**。

### 我们的第一个测试

测试是从中调用情景的OSC2代码，从层次上看被认为是最高层：

1. 它导入测试执行平台配置（例如模拟器）。

2. 它导入了SUT配置（例如被测试系统，通常称为EGO）。在这种情况下，作为被测试功能，我们导入了由Foretellix开发的SUT L4堆栈，并配置了EGO车辆的属性。

3. 它设置要在测试中使用的地图。

4. 定义场景和指标（检查点、覆盖率、KPIs），并调用场景执行。

在下面的图片中，您可以看到将要运行的测试结构：

<p align="center">
  <a href="images/Test_2.png" target="_blank">
    <img src="images/Test_2.png">
  </a>
</p>

!!! 例子 "实际操作时间"
    现在您已准备开始研讨会，是时候更详细地了解第一个测试了。

    使用Foretify Developer OSC2代码编译器打开第一个测试：

    ```bash
    foretify --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --gui
    ```

Foretify GUI 将会在您的浏览器中打开。在这次培训过程中，您将学习到更多功能。目前您将专注于其OSC2代码编译和可视化功能。点击**Source**选项卡，并像下图中那样折叠**Loaded Files**面板：

<p align="center">
  <a href="images/l01_ftx_dev.png" target="_blank">
    <img src="images/l01_ftx_dev.png">
  </a>
</p>

在测试中，请注意载入其他OSC2文件的_import_语句：

```osc linenums="3"
import "$FTX_WORKSHOP/common/workshop_config.osc"
import "$FTX_WORKSHOP/scenarios/cut_in_l01.osc"
import "ts_l01_intro_cov.osc"
import "ts_l01_intro_checks.osc"
```

所有代码可以放在一个文件中，但这样会使其变得不易阅读、不易重用，并且难以管理。

我们现在将浏览导入语句的内容，以及以下各部分中的其余代码。

### _workshop_config.osc_ 配置文件

`workshop_config.osc` 文件（在测试文件的第3行导入）包含了设置 Foretify 和执行平台连接的所有定义（可以使用不同的模拟器），以及系统测试（SUT）连接的定义（在本案例中，即自主驾驶的自我）。

### _cut_in_l01.osc_ 场景文件

这个 `cut_in_l01.osc` 文件在第4行被导入，包含了插入切入场景的抽象定义，这是此实验室的主题。我们通过一组绝对和相对约束的定义，是要求某辆车应该在 SUT 前面变道。OSC2 是唯一支持抽象描述的场景描述语言，极大地减少了场景开发人员需要花费在编写代码上的时间。在下面的图像中，您可以看到抽象切入场景定义的两个具体实例。

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_scenario.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_scenario.png">
  </a>
</p>

!!! 例子 "实践时间"
    点击在 Source 选项卡的测试文件的第4行导入的 `cut_in_l01.osc` 文件，以打开它。

下面展示了实现切入场景的代码：

<p align="center">
  <a href="images/workshop_l01_cut_in_code_vscode.png" target="_blank">
    <img src="images/workshop_l01_cut_in_code_vscode.png">
  </a>
</p>

让我们来看代码：

- Line 6: 在SUT环境中声明了名为sut.cut_in_l01的场景（这就是为什么它是sut.name_of_scenario）
- Line 7: 另一辆车car1（不是SUT）被实例化为“vehicle”类型的对象。
- Line 8: car1将从哪一侧切入被实例化为“av_side”枚举类型。这意味着它可以取值为“left”或“right”。
- Line 10: 表示后续块将依次执行。在这种情况下，这意味着在log_info之后将执行名为approach_phase的阶段，然后是change_lane阶段。
- Line 12: 将切入侧写入日志。日志语句允许您添加行，以便以后用于调试目的。
- Line 14: 创建并标记为“approach_phase”的并行场景阶段。这意味着行15和18将并行执行（记住，OSC2是一种基于缩进的语言）。标签使您可以引用代码的其他部分。
  - Line 15: 触发SUT启动行驶动作。接下来的行（16和17）对SUT的行驶动作添加了一些约束条件，即：
    - Line 16: 约束SUT的速度在接近阶段开始时至少为30公里/小时
    - Line 17: 约束SUT在接近阶段保持其车道
  - Line 18: 触发car1开始行驶动作。由于car1不是SUT，因此在行驶时将完全由Foretify引擎控制。接下来的行（19至22）对car1的移动添加了一些约束条件，以便进行切入操作，即：
    - Line 19: 约束car1在整个阶段中位于SUT相邻的车道。
    - Line 20: 定义car1相对于SUT在该阶段开始时的位置（SUT前方10到20米）。
    - Line 21: 定义car1相对于SUT在该阶段结束时的位置（SUT前方10到20米）。
    - Line 22: 定义car1相对于SUT的位置为尽力而为的条件。这意味着解算器在无法满足约束条件时不会将该场景标记为失败。在某些情况下，由于模拟器的限制，计划运行无法完全展开并被标记为_不完整的场景_。将非关键约束定义为尽力而为的约束可以帮助减少这样的运行份额。您将在高级实验室中了解更多关于这一特性和“计划”与“运行时”之间区别的信息。
- Line 23: 创建并标记为“change_lane”的第二个并行场景阶段。同样，这意味着行24和32中的后续操作是并行执行的。
  - Line 24: 触发SUT启动行驶动作，并跟随几个修饰符。
    - Line 25: 对SUT的行驶动作添加了一个约束条件，即在整个场景阶段保持其车道。
  - Line 26: 触发car1执行行驶()动作，并受到行29至34的约束：
    - Line 27: 约束car1的速度在阶段开始时比SUT的速度慢5到15公里/小时。
    - Line 28: 将速度约束覆盖为非关键的尽力而为的约束。
    - Line 29: 约束car1在场景阶段结束时与SUT在同一车道。
    - Line 30: car1的速度在整个场景阶段内保持恒定。
    - Line 31: 再次强调，此速度约束被定义为非关键，但会尽最大努力执行。
    - Line 32: 该行停用了car1的碰撞规避行为，从而使车道变更对于SUT来说具有挑战性。您将在高级实验室中了解更多有关通用汽车演员碰撞规避行为的信息。

### 行为监测

以下三个部分将简要介绍 _覆盖率_、_KPI或记录_ 和 _检查器_ 的概念，并举例说明。它们是安全驱动验证方法论的关键支柱，支持验证过程并揭示系统下错误的性能。在接下来的实验中，您将深入了解并详细阐述它们的功能。

#### _ts_l01_intro_cov.osc_ 覆盖率定义

!!! 举例 "实操时间"
    打开 `ts_l01_intro_cov.osc` 文件，点击 `ts_l01_intro.osc` 测试文件中源代码选项卡的第5行导入。

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_osc_cover.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_osc_cover.png">
  </a>
</p>

上面的代码定义了 **覆盖率收集**：

- 第一个指标（第4行）是切入发生的一侧&mdash;左侧或右侧。
- 第二个指标（第5行）是执行切入操作的汽车类型，例如卡车或轿车。
- 下一个指标（第6至11行）是车辆在变道场景阶段末尾行驶的速度。
- 最后一个（第12至18行）是当主场景开始时两辆车之间的纵向距离。

!!! 信息
    - 在第6行，我们仅使用了 OSC2 的一个重要结构类型成员：_event_。该事件是 _end_，更确切地说，代表了在 `$FTX_WORKSHOP/scenarios/cut_in_l01.osc` 场景中定义的变道阶段的结束事件。
    - 事件是瞬态对象，代表着一个时刻，并可以触发场景中定义的操作。您可以在结构体中定义事件，但更典型的做法是在演员或场景中定义，就像这个例子中一样。

#### _ts_l01_intro_cov.osc_ KPI定义

该KPI定义在与上一节覆盖项目相同的文件中。

在这段代码中，我们定义了一个KPI：

- 首先，在第21到22行声明了一个变量，用于采样在变道结束时SUT和插入车辆之间的距离。
- 在第25到27行使用了_record()_方法，用于记录稍后要可视化的KPI。

!!! 信息
    现在你已经经历了一些例子，了解覆盖度量和性能度量之间的主要区别至关重要：

    - **覆盖度评估**：_我们在“场景空间”的哪一部分对我们的自动驾驶车辆进行了测试？_ 通过覆盖度和总体覆盖度等级来表达。定义了覆盖项目以支持覆盖度评估。换句话说，覆盖度等级回答了问题：_SUT经过了多好的测试？_
    - **性能评估**：_SUT在测试中表现得有多好？_ 这个问题通过性能等级来回答，可以是一个或多个KPI

#### _ts_l01_intro_checks.osc_ 检查器定义

!!! 例子 "亲身操作时间"
    打开通过在源选项卡中点击`ts_l01_intro.osc`测试文件的第6行导入的`ts_l01_intro_checks.osc`文件。

<p align="center">
  <a href="images/l01_kpi_code.png" target="_blank">
    <img src="images/l01_kpi_code.png">
  </a>
</p>

该检查器的目的是评估SUT和插入车辆之间的距离在整个场景中是否在定义的安全距离内。

- 在第1行中，issue_kind类型正在使用一个独特的名称（safety_distance）扩展，用于新的检查器。
- 第3到12行扩展了先前定义的编写检查器的情景，具体如下：
    - 第5到6行：声明一个用于安全距离阈值的变量，并将其设置为13米。
    - 第8到10行：在模拟的每个时间步（top.clk）中验证以下内容：
        - 两辆车之间的距离是否超过定义的阈值
        - 两辆车是否在同一车道上
    - 第11到12行：如果上述条件不满足，则以sut_error类型和subtype safety_distance的错误停止测试运行。

!!! 信息

    - 刚添加的检查器是用户定义的检查器，但Foretify带有内置检查器。您可以在[全局车辆检查器文档](../osc_dom/oscdomain_metrics.md#global-checkers-for-vehicles)中查看这些检查器。

    - 行业中另一个用于检查器的术语是评估器。

    - 使用检查器，您可以表示场景的成功或失败标准，它们使用与表示操作和参与者的相同语言（OSC2）编写。

    - 使用Foretify时，一些预定义检查器始终处于活动状态，例如碰撞检查或用于SUT偏离道路的检查。您可以随时修改适用于所有场景的默认设置。

    - 您可以定义自定义检查器来捕捉特定于您的SUT的问题，就像我们的示例一样。我们执行的检查器使用KPI值和预定义阈值，以评估SUT的预期行为，但这只是定义的一种方式。

### 地图定义

在下面的代码中（测试文件`ts_l01_intro.osc`的第8行和第9行），我们设置要使用的地图，这是以OpenDrive格式（即\*.xodr）的地图。

```osc linenums="8"
extend test_config:
    set map = "$FTX_WORKSHOP/maps/Town04.xodr"
```

### 场景执行

在`ts_l01_intro.osc`文件的最后两行中，我们最终调用了_cut_in_l01_场景: OSC2程序的入口点始终是_top.main_场景，类似于C/C++中的_main()_函数:

```osc linenums="11"
extend top.main:
    do cil : sut.cut_in_l01()
```
您可以在测试文件的第11行和第12行找到这段代码。

## 首次运行和Foretify

### 有约束的随机生成和自适应场景执行

正如其名称所示，*有约束的随机生成* 是指在指定约束空间内生成随机变量。这是SDV（安全驱动验证）流程的基本前提，也是Foretify构建的核心原则。

从OSC2抽象场景中，Foretify的生成引擎根据指定的约束随机创建具体场景。其中一个最重要的随机化方面是每个场景将展开的地图区域。

一旦为场景制定了计划，运行时的自适应场景执行引擎会根据场景计划进行执行。

!!! Info
    **约束随机生成** 引擎是Foretellix解决方案的主要支柱。这是一个极其强大的工具，可以从抽象描述中生成出数百万个有意义的场景变化。

    当利用这项技术时，情景编写团队所需的**工程资源显着减少**，因为可以从一个抽象定义中生成出数百万个有意义的变化。

生成的场景在执行时将有助于挑战被测试系统，以更有效地**发现和解决错误**。

### 第一次运行

**在继续之前，请确保在浏览器中关闭了Foretify GUI。**

现在您已经浏览了OSC2场景定义，将使用模拟器作为测试执行平台。

接下来，您将更详细地探索Foretify GUI，以加载、启动和分析测试。随后您将进一步探索Foretify的更多功能。

!!! 例子 "实践时间"
    以GUI模式启动Foretify并加载之前检查过的测试：

    ```bash
    foretify --gui --work_dir $FTX_FM_WORKDIR/l01_intro/workdir \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc
    ```

Foretify交互窗口将显示：

<p align="center">
  <a href="images/foretify_gui.png" target="_blank">
    <img src="images/foretify_gui.png">
  </a>
</p>

Foretify窗口显示以下信息：

- **加载**、**准备测试**和**调试**标签（左上角）：
    - **加载**标签用于通过图形用户界面加载`.osc`文件（我们没有使用GUI进行此操作，因为我们通过终端命令加载了文件）。
    - **准备测试**标签允许您设置运行的不同参数。您将在后面的部分了解更多信息。
    - **调试**标签用于在模拟完成后调试运行。
- **状态**（中上方左侧）：
    - 您可以看到加载的文件、加载状态，以及加载过程中发现的问题数量。
- **地图**、**源代码**和**预览**标签（中左侧）：
    - **地图**标签允许您浏览加载的地图，探索地图的不同图层。
    - **源代码**标签显示加载的源文件，使您能够查看加载的代码并在加载的文件之间切换。
    - **预览**标签允许您在单击**执行控制**区域中的**预览**按钮后预览计划的测试。
- **问题**、**检测违规**和**测试信息**标签（左下方）：
    - 这些标签在场景加载后显示。
- **控制场景的执行**（右上方）:
    - 您可以为运行设置种子，预览运行或运行实际模拟。种子是每个具体测试执行的唯一标识符，将从抽象定义生成。
- **日志**（中右侧）

```markdown
!!! Example "操作时间"
在实际运行测试之前，您可以使用**预览**按钮进行实验，该按钮会显示测试对象（SUT）和小车1在可视化器中的预定路线。请注意，由于SUT的行为，计划路径可能会在运行时发生变化，但这是一个非常有用的调试工具。请注意，计划路径只是对该场景创建的计划的表示：实际轨迹是在运行时计算并更新的。

!!! Example "操作时间"
在运行测试之前，尝试通过点击右上角的灰色**预览**按钮并更改种子号来预览您的测试的几个种子。

!!! Example "操作时间"
通过点击右上角的紫色**运行测试**按钮来运行测试。您也可以通过点击右上角的**终端**按钮，然后输入_run_来运行测试。

您会看到模拟器窗口弹出：

- Foretify约束随机生成引擎根据在OSC2代码中为该场景指定的约束和参数生成了特定的场景变体。在预览运行时看到的计划路径为演员们计划的路径。

- Foretify运行时测试编排引擎确保在测试执行平台（模拟器）中的演员能够根据指定的约束移动。

运行时测试编排引擎确保了**自适应场景执行**，这意味着SUT的计划路径偏离会导致NPC采取相应的对策，以确保OSC2文件中指定的场景意图得以实现。

测试完成后，日志区域显示两个额外的选项卡：
```

- **Trace Details**选项卡可以让你查看在仿真过程中收集的场景跟踪信息，如跟踪类型、执行者、时间和持续时间。
- **日志**选项卡中包含了测试执行过程中生成的日志记录。
- **指标**选项卡与覆盖率有关，将在研讨会中详细介绍。

!!! 举例 "实践时间"
    现在，请查看**日志**选项卡，并检查是否看到我们添加的日志消息，以显示切换发生在哪一侧。

### 调试运行

#### 使用Foretify Visualizer进行调试

在运行测试之后，在屏幕中间，您将可以访问**可视化器**选项卡。可视化器是图形后处理工具之一，可以以各种方式配置，帮助您分析执行过程。可视化运行不同于重复执行。它不需要模拟器，并且不会消耗重新运行所需的计算资源。

执行完成后，屏幕左侧的**可视化器**选项卡将自动打开。您可以随时单击该选项卡返回到可视化器。

<p align="center">
  <a href="images/visualizer.png" target="_blank">
    <img src="images/visualizer.png">
  </a>
</p>

##### 重播测试
首先，您可以按下位于左下角的可视化器时间轴中的播放按钮，查看场景如何重新播放：

<p align="center">
  <a href="images/visualizer_play_button.png" target="_blank">
    <img src="images/visualizer_play_button.png">
  </a>
</p>

##### 地图视角
你可以通过在可视化器中点击鼠标右键并向不同方向拖动光标来改变视角。你可以通过在可视化器右上角点击扳手图标来访问**视图工具**。**视图工具** 包括控制视图的选项:

<p align="center">
  <a href="images/visualizer_tools.png" target="_blank">
    <img src="images/visualizer_tools.png">
  </a>
</p>

- 车道方向：启用或禁用驾驶方向箭头的可视化。
- 信号灯：切换交通信号的可见性。
- 速度限制：显示道路的速度限制。
- 碰撞回避：启用碰撞回避激活的可见性。
- 运行时轨迹：显示所选车辆角色的轨迹。
- 计划路径：突出显示场景中车辆的路径。
- 计划目标：显示为所选车辆角色生成的计划目标。
- 计划姿势：突出显示SUT和其他车辆的下一个姿势。
- 驾驶目标：显示为所选车辆角色生成的驾驶目标。
- 预测姿势：显示车辆的预测位置。

##### 摄像机设置

要控制视角和摄像机，点击摄像机设置（相机）图标。要使用摄像机跟踪特定的角色，从左上角的角色(i)下拉列表中选择一个角色，或者在可视化器中点击设置摄像机到固定位置。

<p align="center">
  <a href="images/Camera_settings.png" target="_blank">
    <img src="images/Camera_settings.png">
  </a>
</p>

- 透视视图：切换到俯视图透视。
- 跟随所选角色：跟随所选角色。
- 重置摄像机到所选角色：将摄像机重置到所选角色位置。

##### 测量距离

- 如果Visualizer处于透视视图，请在Visualizer右上角选择相机设置图标，然后关闭透视视图选项。

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

- 在Visualizer中点选以设置测量起始点，然后移动光标并单击以设置测量终点。

<p align="center">
  <a href="images/Measurement_3.png" target="_blank">
    <img src="images/Measurement_3.png">
  </a>
</p>

测量结果将显示在连接起始点和终点的线旁边。

- 要隐藏测量结果，切换关闭测量距离工具图标。

#### 跟踪调试

可以在跟踪视图下查看所有跟踪。跟踪视图与时间轴对齐，因此可以轻松比较不同的跟踪。

**跟踪**以以下特定类型表示：

- **区间**：代表在一段时间内收集的值的集合。区间具有名称、开始/结束时间和类型，并与特定操作者（橙色框）相关联。

- **数值**：代表随时间变化的单个值，数值以波形图形式显示，数值跟踪具有名称、数值和单位，并与特定操作者（红色框）相关联。

<p align="center">
  <a href="images/Traces_1.png" target="_blank">
    <img src="images/Traces_1.png">
  </a>
</p>

为了增强可视化效果，Foretify记录了每个场景的开始和结束时间。

##### 查看区间：
- 在Foretify中选择调试运行选项卡，然后点击Visualizer下的Traces选项卡：

```markdown
<p align="center">
  <a href="images/Traces_2.png" target="_blank">
    <img src="images/Traces_2.png">
  </a>
</p>

在时间轴中，轨迹被显示为间隔，其中有一个当前时间光标，对应其他基于时间的视图，比如可视化器和轨迹选项卡内Actors中的Actor值轨迹。

1. 点击轨迹名称左侧的箭头展开，查看其子场景。

2. 点击轨迹以查看轨迹详细信息，如轨迹类型、Actor、时间、持续时间以及间隔期间收集的度量数据。

3. 在轨迹详细信息下，点击轨迹的开始时间或结束时间以将通用时间轴设置为该时间。

##### 将时间轴定位到轨迹：
1. 在轨迹选项卡中，选择要将时间轴定位到的间隔（橙框）。

2. 点击帧时间轴图标（红框）。

<p align="center">
  <a href="images/Intervals_3.png" target="_blank">
    <img src="images/Intervals_3.png">
  </a>
</p>

- 要重置时间轴，使其不再定位到间隔，点击时间轴右侧的取消定位图标。

<p align="center">
  <a href="images/Intervals_4.png" target="_blank">
    <img src="images/Intervals_4.png">
  </a>
</p>

!!! 例如"实操时间"
    使用可视化器重新运行测试，检查SUT的行为以及上述不同选项。

### 运行不同种子

!!! 例如"实操时间"
    在右上角的执行控制区域使用另一个种子号运行另一个模拟，将种子号设置为4，然后点击**运行测试**按钮。这是一个种子，我们在最小距离阈值上引入的检查程序会失败。

    使用您选择的种子再次运行另一个模拟。

    再次搜索日志，查看指示切入发生在哪一侧的消息。
```

```markdown
现在您可以通过在Foretify终端输入exit或关闭Foretify窗口来关闭Foretify。

!!! 信息
    **种子**作为具体执行的随机生成的输入，基于一个抽象场景定义。使用**相同的种子**再次根据抽象定义生成具体测试会得到**相同的具体变化**。这是一个重要功能，一方面让您可以完全随机测试，另一方面也可以为调试目的重新创建单个具体执行。

    由于种子的定义和实现，您可以始终**追踪生成的特定具体变化**。场景的可追溯性是一个基本功能，它让您能够确定导致错误的条件并重现它们。

## Foretify Manager

现在您已运行第一批测试，您可以探索达到的覆盖范围和结果。我们将在实验2中正式定义覆盖率，但现在您可以使用Foretellix工具直观地了解这个概念。

Foretify Manager是Foretellix工具，可以让您可视化验证过程的整体状态，导入多次测试执行的结果和关键绩效指标（KPIs）。其客户端-服务器架构如下图所示：

<p align="center">
  <a href="images/fmanager_architecture.png" target="_blank">
    <img src="images/fmanager_architecture.png">
  </a>
</p>

客户端可以是Python脚本或Web用户界面（网页），两者都可以对测试套件结果数据执行操作和查询。服务器管理数据库并执行客户端的命令。这种拓扑结构使多个用户可以同时分析验证结果的不同方面。

### 打开Foretify Manager

Foretify Manager是基于浏览器的应用程序。
```

```Chinese
!!! Example "实践时间"
    通过终端调用以下命令启动Foretify Manager： 

    ```bash
    fmanager
    ```

    第一件看到的是登录页面：

    <img src="images/fmanager_login.png" alt="image" width="200"/>

!!! Info
    联系演示者获取有关已分配用户名和密码的更多信息。



### 创建项目

Foretify Manager项目是一个协作框架，使用户能够为验证和验证数据以及收集的指标设置权限和所有权。

要创建项目：

1. 打开Foretify Manager，并使用Foretify Manager凭据登录。

2. 在“选择项目”页面中，单击“创建新项目”。

!!! Info
    联系演示者获取更多关于项目名称的信息

- 单击“创建”紫色按钮

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

此时，Foretify Manager数据库为空，因此下一步是上传执行的少数测试结果。

!!! Example "实践时间"
    切换到终端，并通过以下调用上传结果：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir
    ```

您可以使用```--run_group_name```参数为您的测试组指定一个特定的名称。请参阅[upload_runs文档](../fman_user/fmanuser_launch_test_suite.md#upload-a-regression)。
```

```markdown
!!! Example "实践时间"
    切换回你浏览器中打开的Foretify Manager Web应用，并点击刷新按钮（确保在“测试套件结果”选项卡上）

上传后，测试执行应该会被加载并显示在你之前创建的项目中：

<p align="center">
  <a href="images/fmanager_regression_2.png" target="_blank">
    <img src="images/fmanager_regression_2.png">
  </a>
</p>

### 分析上传的测试执行

通过点击刚刚导入的测试套件，你可以查看每个执行的情况。

你应该会看到如下图所示的内容：

<p align="center">
  <a href="images/fmanager_runs.png" target="_blank">
    <img src="images/fmanager_runs.png">
  </a>
</p>

不同颜色的方块表示：

- _黄色_：运行列表右上角的图标可用于导出和删除执行，以及保留和重置您的选择。使用最右侧的列选择图标来添加和删除运行属性，例如目录、OS用户、持续时间等等。
- _橙色_：_问题树_按种类分组展示了所有问题。
- _蓝色_：_汇总视图_可让你根据运行属性对运行进行汇总。

在_Runs_视图中点击一个运行时，会弹出一个新的Foretify Manager窗口。

<p align="center">
  <a href="images/Run_view_2.png" target="_blank">
    <img src="images/Run_view_2.png">
  </a>
</p>

你会看到有两个主要选项卡：**调试运行**和**运行摘要**。
**调试运行**标签与我们之前在Foretify中看到的内容相同，

<p align="center">
  <a href="images/run_source.png" target="_blank">
    <img src="images/run_source.png">
  </a>
</p>

点击**运行摘要**选项卡，你可以查看关于失败运行的更详细信息。
```

```html
<p align="center">
  <a href="images/run_summary.png" target="_blank">
    <img src="images/run_summary.png">
  </a>
</p>

!!! Example "现场操作时间"
    点击每个运行来探索您的其他两个运行。

### 什么是工作区以及如何创建它

工作区是一组导入的测试套件，您可以在其中分析覆盖率数据。

!!! Example "现场操作时间"
    在**测试套件结果**选项卡中选择您的测试套件，然后按照下面的示例点击**创建工作区**。

<p align="center">
  <a href="images/fmanager_workspace_2.png" target="_blank">
    <img src="images/fmanager_workspace_2.png">
  </a>
</p>

选择一个工作区名称。然后点击**创建工作区**来创建工作区。

<p align="center">
  <a href="images/fmanager_workspace_name_2.png" target="_blank">
    <img src="images/fmanager_workspace_name_2.png">
  </a>
</p>

Foretify Manager 网站应用程序切换到**当前工作区**视图：

<p align="center">
  <a href="images/fmanager_workspace_after_creation.png" target="_blank">
    <img src="images/fmanager_workspace_after_creation.png">
  </a>
</p>


工作区包括以下内容：

- **VGrade** 是总体指标等级（您将在第四实验室中学习）。
- **总运行数**（旁边是**VGrade**）是一个通过和未通过运行的统计数据。
- 在**VPlan**选项卡（蓝色）中，您可以看到指标层次结构。
- 在**运行**选项卡（绿色）中，您可以看到当前工作区选择的运行列表。

!!! Example "现场操作时间"
    探索**VPlan**树和**运行**选项卡中的运行。

### 指标和检查器表示
```

### 覆盖率
在我们的 `ts_l01_intro_cov.osc` 覆盖文件中，我们定义了四个覆盖项和一个关键绩效指标（KPI metric）。
例如，cut_in_side 的覆盖率达到了100%，而 speed_sut 的覆盖率仅为20%。这表明需要运行更多的测试来填补 speed_sut 覆盖率的空缺。
在我们的文件中，这两个覆盖项的定义稍有不同：

- 注意：百分比可能会根据使用的种子而异！

<p align="center">
  <a href="images/lab01_coverage_conclusions.png" target="_blank">
    <img src="images/lab01_coverage_conclusions.png">
  </a>
</p>

对于 speed_sut，我们详细指定了桶，而对于 cut_in_side，我们没有强加任何规则。这给了 cut_in_side 覆盖项额外的自由度。该项的覆盖率达到100% ，是因为在测试过程中至少击中了可能的两侧（左侧和右侧）。

 !!! 示例 "亲自动手时间"
    现在检查其他覆盖项及其分数，可在 Foretify Manager 中，在 VPlan 选项卡下找到 _cut_in_side_ 覆盖项。

### 关键绩效指标（KPIs）
正如你所记得的，我们先前定义了 distance_kpi KPI，它衡量了 SUT 与变道结束时切入车辆之间的距离。指定事件有助于更好地捕捉该项在兴趣高峰时的值。由于事件也已被指定，我们注意到对于此KPI，每次运行都会有单一数值。

<p align="center">
  <a href="images/workspace_kpi_2.png" target="_blank">
    <img src="images/workspace_kpi_2.png">
  </a>
</p>

!!! 示例 "亲自动手时间"
    你得到了哪些 KPI 值？可以在 Visualizer 中复查模拟，并查看值与模拟之间的关联。

### 检查者

在查看**Runs**标签页时，您可以观察到有多少次运行通过了，有多少次失败了。失败的运行是由代码中定义的检查器引起的。由于检查器定义了如果系统不满足布尔条件就会失败响应，因此这立即反映在问题树中。

<p align="center">
  <a href="images/checkers.png" target="_blank">
    <img src="images/checkers.png">
  </a>
</p>

!!! 示例 "实践时间"
    分析您的运行。您得到了多少次失败的运行？

### 增加覆盖率

少量测试远远不足以达到任何有意义的覆盖范围。Foretellix 技术的强大之处在于运行大量自动生成的测试。因此，在这一部分，您将运行更多的测试以增加覆盖率。为此，您需要根据情景创建不同的测试用例。如上所述，这由创建过程中使用的种子控制。

**在继续之前，请确保关闭浏览器中的 Foretify GUI**。

!!! 示例 "实践时间"
    让我们运行多15个测试，设置一个新的工作目录。请注意，我们不会使用 foretify gui，因此你将会不时看到模拟器窗口弹出。

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir2 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 20 --crun 15
    ```

    选项 ```--crun``` 将从种子 20 开始运行 15 个测试（即种子 20 到 34）。这是一个扩展测试的选项之一，但在第三实验中我们将涵盖更多选项。

请注意生成引擎如何生成15个独特的具体测试，在地图的不同部分，同时将所有具体测试保持在抽象场景定义的边界内（例如，速度，车道位置等）。

!!! 例子 "实践时间"
    新的运行应该增加覆盖率。您现在可以使用以下命令将这15个额外运行上传到 Foretify 管理器：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir2
    ```

    然后，在 Foretify 管理器的 **测试套结果** 选项卡中，选择新上传的测试套，并按照下图中所示将其添加到工作区

    您现在可以查看 **VPlan** 树，看看覆盖率提高了多少。

<p align="center">
  <a href="images/l01_add_ws_1.png" target="_blank">
    <img src="images/l01_add_ws_1.png">
  </a>
</p>

### 在测试套间切换

当您将新的测试套添加到工作区时，您可以通过切换到这些测试套之间，以便分析每个测试套。

!!! 例子 "实践时间"
    转到您的工作区，然后单击 **测试套结果工作区视图** 中的下箭头。

<p align="center">
  <a href="images/l01_switch_ws_1.png" target="_blank">
    <img src="images/l01_switch_ws_1.png">
  </a>
</p>

将会弹出一个新窗口显示可用的测试套。您可以在它们之间切换以查看差异。

<p align="center">
  <a href="images/l01_switch_ws_2.png" target="_blank">
    <img src="images/l01_switch_ws_2.png" width="50%">
  </a>
</p>

在加载新的测试套后，您需要单击 **计算** 按钮。

```markdown
<p align="center">
  <a href="images/l01_switch_ws_3.png" target="_blank">
    <img src="images/l01_switch_ws_3.png">
  </a>
</p>

### 分组测试套件

当你有多个测试套件时，有时你想将它们分组，以增加工作空间的覆盖范围。

!!! 例子 "实践时间"
    进入你的工作空间，点击**工作空间测试套件结果**，选择测试套件，然后点击**分组**图标进行分组。

<p align="center">
  <a href="images/l01_group_ws_1.png" target="_blank">
    <img src="images/l01_group_ws_1.png">
  </a>
</p>

一个新窗口会出现，请为你的分组命名，然后点击**分组**。

<p align="center">
  <a href="images/l01_group_ws_2.png" target="_blank">
    <img src="images/l01_group_ws_2.png" width="50%">
  </a>
</p>

你会看到你的运行情况已更新，覆盖范围也增加了。

<p align="center">
  <a href="images/l01_group_ws_3.png" target="_blank">
    <img src="images/l01_group_ws_3.png">
  </a>
</p>

### 取消分组测试套件

也可以取消你创建的分组。

!!! 例子 "实践时间"
    进入你的工作空间，点击**工作空间测试套件结果**，选择已分组的运行，然后点击**分组信息**图标取消分组。

<p align="center">
  <a href="images/l01_ungroup_ws_1.png" target="_blank">
    <img src="images/l01_ungroup_ws_1.png">
  </a>
</p>

一个新窗口会出现，点击**取消分组**按钮。

<p align="center">
  <a href="images/l01_ungroup_ws_2.png" target="_blank">
    <img src="images/l01_ungroup_ws_2.png" width="50%">
  </a>
</p>

### 更改地图

**在继续之前，请确保在浏览器中关闭 Foretify。**
```

另一种增加覆盖率的方法是修改测试执行的ODD。OSC2语言的一个强大特性是，为了改变ODD，您只需要修改一行代码。

有了新的地图，Foretify将在地图拓扑可以支持的位置随机生成新的具体测试用例。

!!! 示例 "实操时间"
    打开文件 `$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`，其中定义了测试。

    编辑 `ts_l01_intro.osc` 文件中的场景，并按以下方式更改定义地图的那一行：

    ```osc linenums="8"
    extend test_config:
        set map = "$FTX_WORKSHOP/maps/cloverleaf.xodr"
    ```
    您现在可以使用以下命令再次运行测试5次：

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 2 --crun 5
    ```

您现在可以观察到这个场景是在另一个地图上执行的。

!!! 注意

    如果您为特定场景选择的地图不支持场景的执行，Foretellix工具将显示矛盾错误。例如，一个插入场景无法在单车道道路上执行。

!!! 示例 "实操时间"
    现在您已经运行了额外的测试，您可以将它们上传到Foretify Manager，查看覆盖率是否有所提高。为了做到这一点，您可以运行以下命令：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir3
    ```
    将新的运行添加到之前创建的工作空间中。

## 后续步骤

本实验室的目标是

- 熟悉用于研讨会的云环境
- 浏览一些针对插入场景的基本 OSC2 代码
- 以交互模式运行 Foretify 并加载插入场景
- 熟悉“seed”的概念
- 熟悉用于测试的 OSC2 及其主要组件
- 打开 Foretify Manager 并探索收集到的指标

接下来，在实验 2 中，您将扩展插入场景，收集更多覆盖度指标，并介绍在 Foretify 中调试场景的方法。

> 本文由ChatGPT翻译，如有任何遗漏，请[**反馈**](https://github.com/linyuxuanlin/Wiki_MkDocs/issues/new)。