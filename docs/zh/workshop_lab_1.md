---
chapnum: 1
---

# 实验 1: 使用 OpenScenario 2 和 Foretellix 技术的第一步

## 学习目标

这个实验向你介绍以下内容：

- 一个简单的 **ASAM OpenSCENARIO 2.0 (OSC2)** 测试实现。（ASAM 是自动化和测试系统标准化协会的缩写。）

!!! 注意
    在 Foretellix 文档中，“OSC2” 一词指的是 “ASAM OpenSCENARIO® DSL 版本 2.x.”

- **Foretify™**，这是一个 **场景开发和测试自动化平台**，用于：
    - 编译 OSC2 源代码
    - 从抽象场景定义生成具体测试
    - 在测试执行期间控制仿真平台中的参与者
    - 绘制和可视化测试执行的结果

- **Foretify Manager**，这是一个**大数据分析平台**，主要用于：
    - 收集多个测试执行的关键性能指标 (KPIs)、检查器消息和覆盖度指标
    - 可视化测试进展，实现基于安全驱动的验证（SDV）方法论

<p align="center">
  <a href="images/l01_ftx_diagram.png" target="_blank">
    <img src="images/l01_ftx_diagram.png">
  </a>
</p>

!!! 注意
    点击任何图片可在新标签页中查看全尺寸图片。

## OSC2 语言

```
在 AD 和 ADAS 功能的开发和验证过程中，有必要使用各种*场景*来刺激系统测试（SUT）。场景是一个或多个行为者（如汽车、行人、环境条件和 SUT 本身）的定时序列。OSC2 是一种特定领域的语言，专门用于描述行为者在环境中移动的场景。这些场景具有属性，可以限制行为者类型、它们的移动方式，以及环境（包括场景应该发生的地图位置）。

!!! Info
    采用*约束随机*方法，对于未受约束的每个场景属性，都会进行随机化。举个例子，如果未约束切入场景的“侧”属性为“右”，那么它将从可能属性的空间中随机选择（即“左”和“右”）。

    此外，这些值是从可能属性的空间中选择，以满足来自场景、行为者和地图的约束。举个例子，如果 SUT（EGO）正在驾驶一条两车道道路的最右侧车道上，“切入”场景的“侧”属性就不会选择为“右”。

OSC2 的构建块是数据结构，比如：
```

- **Actors**: 代表真实世界的实体。正如其名称所暗示的那样，它们在场景中扮演着“角色”。
- **Scenarios** 或 **Actions**: 描述了角色的行为。通常，场景是一系列动作的长序列，但两者之间没有正式区别。两者都可以通过*修饰符*进行修改。
- **Modifiers**: 为场景添加约束，帮助控制其在所需边界内的执行。
- **Labels**: 定义了任何标量、结构或者角色类型的命名数据字段。
- **Simple structs**: 是包含属性、约束等基本实体。

您可以通过参阅Foretellix的【OSC2语言文档】(../osc_lang/osclang_intro.md) 和【OSC领域模型文档】(../osc_dom/oscdomain_intro.md) 或访问[ASAM类型定义主题](https://www.asam.net/index.php?eID=dumpFile&t=f&f=3460&token=14e7c7fab9c9b75118bb4939c725738fa0521fe9#type-definitions) 或ASAM OSC2网页的主要[ASAM OSC2网页](https://www.asam.net/project-detail/asam-openscenario-v20-1/)，了解更多有关上述主题的信息。

!!! Info 
    Foretellix工具**原生支持ASAM OSC2语言**，带来诸多好处，其中之一是语言和工具都是**执行平台无关**的。这导致在更改验证环境时**减少了工作量**，并且在选择适当的验证平台时**增加了灵活性**。

    OSC2支持**抽象场景**和**安全驱动验证**流。通过本研讨会，您将了解这些功能如何**优化V&V工作**，从而**减少了验证自主系统所需的资源**。

### 我们的第一个测试

测试是从中调用场景的OSC2代码，被认为是在层次结构上的最高层次：

1. 它导入测试执行平台配置（如模拟器）

2. 它导入SUT配置（例如，被测试系统，通常称为EGO）。在这种情况下，作为被测试功能，我们导入了Foretellix开发的SUT L4堆栈，并配置EGO车辆的属性。

3. 它设置要在测试中使用的地图。

4. 它定义场景和指标（检查、覆盖率、KPIs），并调用场景执行。

在下图中，你可以看到你将要运行的测试的结构：

<p align="center">
  <a href="images/Test_2.png" target="_blank">
    <img src="images/Test_2.png">
  </a>
</p>

!!! Example "实践时间"
    现在你已经准备好开始研讨会了，是时候更详细地了解第一个测试了。

    用Foretify Developer OSC2代码编译器打开第一个测试：

    ```bash
    foretify --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --gui
    ```

Foretify GUI将在你的浏览器中出现。你将在本次培训中学到更多功能。现在你将专注于它的OSC2代码编译和可视化功能。单击**Source**标签，并折叠如下图像中所示的**Loaded Files**面板：

<p align="center">
  <a href="images/l01_ftx_dev.png" target="_blank">
    <img src="images/l01_ftx_dev.png">
  </a>
</p>

在测试中，请注意_load_语句，它加载其他OSC2文件：

```osc linenums="3"
import "$FTX_WORKSHOP/common/workshop_config.osc"
import "$FTX_WORKSHOP/scenarios/cut_in_l01.osc"
import "ts_l01_intro_cov.osc"
import "ts_l01_intro_checks.osc"
```
所有的代码可以写在一个文件中，但这样会使得代码不易读，不易重用，并且难以管理。

我们接下来要讨论导入语句中的内容，以及后面的代码部分。

### _workshop_config.osc_ 配置文件

`workshop_config.osc` 文件（在测试文件的第3行导入）包含了设置Foretify和执行平台连接的所有定义（它可以使用不同的模拟器），以及系统被测对象（SUT）连接的定义（在这种情况下，是自主驾驶的自车）。

### _cut_in_l01.osc_ 场景文件

这个 `cut_in_l01.osc` 文件在第4行被导入，它包含了切入场景的抽象定义，这是本实验的主题。我们正在定义，通过一组绝对和相对约束，一个车辆应该在SUT的前面变道。OSC2是唯一支持抽象的场景描述语言，显著减少了场景开发人员编写代码所需的时间量。在下面的图片中，你可以看到抽象切入场景定义的两个具体实例。

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_scenario.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_scenario.png">
  </a>
</p>

!!! 举例 "动手操作时间"
    点击测试文件中源代码选项卡的第4行来打开被导入的 `cut_in_l01.osc` 文件。

下面是实现切入场景的代码：

<p align="center">
  <a href="images/workshop_l01_cut_in_code_vscode.png" target="_blank">
    <img src="images/workshop_l01_cut_in_code_vscode.png">
  </a>
</p>

让我们来解读一下这段代码：

- Line 6: 引入场景 sut.cut_in_l01，它是在系统测试环境中声明的（这就是为什么它是 sut.name_of_scenario）。
- Line 7: car1，另一辆车（不是系统测试环境中的车），被实例化为“vehicle”类型的对象。
- Line 8: car1将从哪一侧切入被实例化为枚举类型“av_side”。这意味着它可以持有值“left”或“right”。
- Line 10: 表示接下来的块将依次被执行。在这种情况下，这意味着在 log_info 之后执行名为 approach_phase 的阶段，然后执行 change_lane 阶段。
- Line 12: 将切入的一侧写入日志。日志语句允许您添加行，以便以后可以用于调试目的。
- Line 14: 创建并标记为“approach_phase”的并行场景阶段。这意味着行15和18将并行执行（记住，OSC2是一种基于缩进的语言）。标签允许您从代码的其他部分引用代码的这一部分。
  - Line 15: 触发系统测试环境开始执行 drive() 操作。接下来的行（16和17）对系统测试环境的驾驶行为添加了一些约束，即：
    - Line 16: 约束系统测试环境的速度至少为 30 公里/小时在接近阶段开始时
    - Line 17: 约束系统测试环境在接近阶段保持车道
  - Line 18: 触发 car1 开始执行 drive() 操作。由于 car1 不是系统测试环境，它将在行驶过程中完全由 Foretify 引擎控制。接下来的行（19到22）对 car1 的移动添加了一些约束，以便实现切入操作，即：
    - Line 19: 约束 car1 在整个这个阶段位于系统测试环境旁边的车道。
    - Line 20: 定义了 car1 相对系统测试环境在这个阶段开始时的位置（在系统测试环境的前方 10 到 20 米处）。
    - Line 21: 定义了 car1 相对系统测试环境在这个阶段结束时的位置（在系统测试环境的前方 10 到 20 米处）。
    - Line 22: 定义了 car1 相对系统测试环境的位置为最大努力条件。这意味着，求解器在约束无法满足的情况下不会将该场景标记为失败。在某些情况下，由于模拟器的限制，计划的运行可能无法完全展开，并被标记为不完整的场景。将非关键约束定义为最大努力约束有助于减少此类运行的占比。您将在高级实验室中详细了解此功能，以及“计划”和“运行时”之间的区别。
- Line 23: 创建了第二个并行场景阶段，并标记为“change_lane”。同样这意味着接下来的行24和32是并行执行的。
  - Line 24: 触发系统测试环境开始执行 drive()，然后是几个修改器。
    - Line 25: 对系统测试环境的驾驶行为添加了约束，即在整个场景阶段中保持车道。
  - Line 26: 触发 car1 执行 drive() 操作，并受到行29到34的约束：
    - Line 27: 约束 car1 在阶段开始时的速度比系统测试环境的速度要慢 5 到 15 公里/小时。
    - Line 28: 重写速度约束为非关键、最大努力约束。
    - Line 29: 约束 car1 在场景阶段结束时与系统测试环境位于相同的车道。
    - Line 30: car1 的速度在整个场景阶段内保持恒定。
    - Line 31: 同样，这个速度约束被定义为非关键，但以最大努力执行。
    - Line 32: 该行取消了 car1 的碰撞避免行为，因此使得系统测试环境的车道变更变得具有挑战性。您将在高级实验室中详细了解通用车辆角色的碰撞避免行为。

### 行为监控

以下三个部分将简要介绍 _覆盖率_、_KPI 或记录_ 和 _检查器_ 的概念，并借助一些例子加以说明。它们是安全驱动验证方法论的关键支柱，因为它们支持验证过程并揭示了 SUT 的不正确性能。在下一个实验中，您将深入了解并详细阐述它们的功能。

#### 定义 _ts_l01_intro_cov.osc_ 覆盖率

!!! 例子 "实践时间"
    在 Source 标签页中点击 `ts_l01_intro.osc` 测试文件的第5行导入 `ts_l01_intro_cov.osc` 文件。

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_osc_cover.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_osc_cover.png">
  </a>
</p>

上述代码定义了**覆盖率收集**：

- 第一个指标（第4行）是切入发生的一侧&mdash;左侧或右侧。
- 第二个指标（第5行）是进行切入的车辆类型，例如卡车或轿车。
- 下一个指标（6到11行）是车辆在变道情景阶段结束时的行驶速度。
- 最后一个指标（12到18行）是主情景开始时两辆车之间的纵向距离。

!!! 信息
    - 在第6行中，我们只使用了 OSC2 中一个重要的结构化类型成员：_event_。该事件是 _end_，更精确地表示了在 `$FTX_WORKSHOP/scenarios/cut_in_l01.osc` 情景中定义的变道阶段的结束事件。
    - 事件是瞬时对象，代表一个时间点，并可以触发在情景中定义的动作。您可以在结构体内定义事件，但更典型的是在角色或情景内定义，就像这个例子中一样。

### 定义 _ts_l01_intro_cov.osc_ 关键绩效指标

该关键绩效指标在与前一部分的覆盖项相同的文件中定义。

在代码的这部分中，我们定义了一个关键绩效指标：

- 首先，在第 21 到 22 行声明了一个用于采样变道结束时 SUT 和插入车辆之间距离的变量。
- 第 25 到 27 行使用 _record()_ 方法，记录将要在稍后进行可视化的关键绩效指标。

!!! 信息
    现在您已经经历了一些例子，了解关键覆盖度和性能度量之间的主要区别至关重要：

    - **覆盖评估**：_“场景空间”的哪一部分我们已经在自动驾驶车辆方面进行了测试？_ 这是通过覆盖度和整体覆盖度等级来表达的。定义了支持覆盖评估的覆盖项。换句话说，覆盖度等级回答了问题：_SUT测试得有多好？_
    - **性能评估**：_SUT在测试中表现如何？_ 这个问题由性能等级来回答，可以是一个或多个关键绩效指标。

### 定义 _ts_l01_intro_checks.osc_ 检查器

!!! 例子 "实践时间"
    在 Source 标签中点击 `ts_l01_intro.osc` 测试文件的第 6 行导入的 `ts_l01_intro_checks.osc` 文件。

<p align="center">
  <a href="images/l01_kpi_code.png" target="_blank">
    <img src="images/l01_kpi_code.png">
  </a>
</p>

该检查器的目的是评估 SUT 和插入车辆之间的距离是否在整个场景中保持在定义的安全距离内。

### 1. 扩展 `issue_kind` 类型

- 在第 1 行中，`issue_kind` 类型被扩展，给新检查器分配了一个唯一的名称（`safety_distance`）。

### 2. 编写检查器的场景扩展

- 第 3 行到第 12 行扩展了之前定义的场景以编写检查器，具体如下：
    - 第 5 行和第 6 行：声明了一个安全距离阈值的变量，并将其设置为 13 米。
    - 第 8 行到第 10 行：在每个时间步（`top.clk`）的仿真中，检查以下条件：
      - 两辆车之间的距离是否超过了定义的阈值
      - 两辆车是否在同一车道上
    - 第 11 行到第 12 行：如果不满足上述条件，则以 `sut_error` 类型和 `safety_distance` 子类型停止测试运行。

### 信息

- 刚刚添加的检查器是用户自定义的检查器，但 Foretify 自带了内置检查器。你可以在 [Global checkers for vehicles documentation](../osc_dom/oscdomain_metrics.md#global-checkers-for-vehicles) 中查看这些检查器。

- 行业中另一个用于表示检查器的术语是评估器（evaluator）。

- 使用检查器可以表示场景的成功或失败标准，检查器使用与表示动作和角色相同的语言（OSC2）来编写。

- 一些预定义的检查器在使用 Foretify 时总是处于激活状态，例如碰撞检查器或检查 SUT 是否偏离道路的检查器。你可以随时修改适用于所有场景的默认设置。

- 你可以定义自定义检查器来捕捉特定于你 SUT 的问题，正如我们示例中所展示的那样。我们使用的检查器通过 KPI 值和预定义阈值来评估 SUT 的预期行为，但这只是定义检查器的一种方式。

### 地图定义

在下面的代码中（`ts_l01_intro.osc` 测试文件的第 8 行和第 9 行），我们设置了要使用的地图，该地图采用 OpenDrive 格式（即 \*.xodr）。

```osc linenums="8"
extend test_config:
    set map = "$FTX_WORKSHOP/maps/Town04.xodr"
```

### 场景执行

在`ts_l01_intro.osc`文件的最后两行中，我们终于调用_cut_in_l01_场景：OSC2程序的入口点始终是_top.main_场景，类似于C/C++_main()_函数：

```osc linenums="11"
extend top.main:
    do cil : sut.cut_in_l01()
```
您可以在测试文件的第11行和第12行找到此代码。

## 首次运行和Foretify

### 约束随机生成和自适应场景执行

顾名思义，**约束随机生成**意味着在指定约束空间内生成随机变量。这是SDV（安全驱动验证）流程的基本前提，也是Foretify构建的核心原则。

从OSC2抽象场景中，Foretify的生成引擎根据指定的约束随机创建具体场景。其中最重要的一个方面是随机化的是每个场景将展开的地图区域。

一旦场景的计划被确定，运行时的自适应场景执行引擎会根据场景计划执行。

!!! Info
    **约束随机生成**引擎是Foretellix解决方案的主要支柱。这是一个非常强大的工具，可以根据抽象描述生成并意义重大的场景变化。

    在利用这项技术时，与场景编写团队所需的**工程资源显著减少**，因为根据一个抽象定义生成了数百万种有意义的变化。

```markdown
生成的场景，在执行时，将有助于挑战被测试系统，以便以更有效的方式**发现和解决错误**。

### 初次运行

**在继续之前，请确保已关闭浏览器中的 Foretify GUI。**

现在您已经浏览了 OSC2 场景定义，您将使用模拟器作为您的测试执行平台。

现在您会更详细地探索 Foretify GUI，以加载、启动和分析测试。稍后您将探索 Foretify 的更多功能。

!!! 例子 "实践时间"
    在 GUI 模式下启动 Foretify 并加载之前检查过的测试：

    ```bash
    foretify --gui --work_dir $FTX_FM_WORKDIR/l01_intro/workdir \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc
    ```

Foretify 交互窗口将会出现：

<p align="center">
  <a href="images/foretify_gui.png" target="_blank">
    <img src="images/foretify_gui.png">
  </a>
</p>

Foretify 窗口显示以下信息：
```

- **加载**，**准备测试**和**调试**选项卡（左上角）：
    - **加载**选项卡用于通过GUI加载`.osc`文件（我们没有使用GUI加载文件，而是通过终端命令加载文件）。
    - **准备测试**选项卡允许您设置运行的不同参数。您将在后面的部分了解更多信息。
    - **调试**选项卡用于在模拟完成后调试运行。

- **状态**（中上方左侧）：
    - 您可以看到加载的文件、加载状态以及加载过程中发现的问题数量。

- **地图**、**源代码**和**预览**选项卡（中左侧）：
    - **地图**选项卡允许您浏览加载的地图，探索地图的不同图层。
    - **源代码**选项卡显示加载的源代码文件，允许您查看加载的代码并在加载的文件之间切换。
    - **预览**选项卡允许您在单击**执行控制**区域中的**预览**按钮后预览计划的测试。

- **问题**，**代码审查违规**和**测试信息**选项卡（左下角）：
    - 选项卡在场景加载后显示。

- **控制场景的执行**（右上方）：
    - 您可以设置运行的种子，预览运行或运行实际模拟。种子是根据抽象定义生成的每个具体测试执行的唯一标识符。

- **日志**（中右侧）

```markdown
!!! Example "动手时间"
    在实际运行测试之前，您可以通过单击“预览”按钮来尝试一下，预览 SUT 和 car1 在可视化器中的规划路径。请注意，由于 SUT 的行为，规划路径可能会在运行时发生变化，但这是一个非常有用的工具，可以帮助您调试测试。需要注意的是，规划路径只是对该场景创建的计划的一种表示: 真正的轨迹是在运行时计算和更新的。

!!! Example "动手时间"
    在运行测试之前，尝试通过单击右上角的灰色“预览”按钮，更改种子号，来预览您的测试的几种情况。

!!! Example "动手时间"
    通过单击右上角的紫色“运行测试”按钮来运行测试。您也可以单击右上角的“终端”按钮，然后输入 _run_ 命令来运行测试。

您将看到模拟器窗口弹出:

- Foretify 约束随机生成引擎根据在 OSC2 场景代码中指定的约束和参数，生成了特定的场景变体。 规划的角色路径就是您在预览运行时看到的路径。

- Foretify 运行时测试编排引擎确保测试执行平台（模拟器）中的角色能够按照指定的约束进行移动。

运行时测试编排引擎确保了 **自适应场景执行**，这意味着 SUT 路径的偏离将导致 NPCs 采取相应的对策，以满足 OSC2 文件中指定的场景意图。

测试完成后，日志区域会显示两个额外的选项卡:
```

将以下内容翻译成中文：

- **跟踪详细信息** 选项卡，允许您检查模拟过程中收集的数值，如跟踪类型、参与者、时间和持续时间等场景跟踪信息。
- **日志** 选项卡，包含测试执行期间生成的日志信息。
- **度量** 选项卡，与覆盖率相关，将在研讨会后面详细介绍。

!!! 例子 "实践时间"
    现在，请查看**日志**选项卡，并检查是否看到我们添加的日志信息，以显示切入发生在哪一侧。

### 调试执行

#### 使用Foretify Visualizer进行调试

运行测试后，在屏幕中央，您将可以访问**可视化器**选项卡。可视化器是可配置的图形后处理工具之一，可以以多种方式配置，帮助您分析执行过程。可视化一个执行与重复执行不同。它不需要模拟器，也不会消耗重新运行所需的计算资源。

执行完成后，屏幕左侧的**可视化器**选项卡将自动打开。
您可以随时点击选项卡返回可视化器。

<p align="center">
  <a href="images/visualizer.png" target="_blank">
    <img src="images/visualizer.png">
  </a>
</p>

##### 重播测试
首先，您可以按下可视化器时间轴底部的播放按钮，看场景是如何重播的：

<p align="center">
  <a href="images/visualizer_play_button.png" target="_blank">
    <img src="images/visualizer_play_button.png">
  </a>
</p>

##### 地图视角
在可视化器中，您可以通过点击鼠标右键并将光标拖动到不同的方向来改变视角。您可以通过点击可视化器右上角的扳手图标访问**视图工具**。**视图工具**中有控制视图的选项：

<p align="center">
  <a href="images/visualizer_tools.png" target="_blank">
    <img src="images/visualizer_tools.png">
  </a>
</p>

- 车道方向：启用或禁用驾驶方向箭头的显示。
- 信号：切换交通信号的可见性。
- 速度限制：显示道路的限速。
- 避撞系统激活：启用避撞系统激活的可见性。
- 运行轨迹：显示所选车辆角色的轨迹。
- 计划路径：突出显示场景中现有车辆的路径。
- 计划目标：显示为所选车辆角色生成的计划目标。
- 计划位置：突出显示系统的下一个位置及其他车辆的位置。
- 驾驶目标：显示为所选车辆角色生成的驾驶目标。
- 预测位置：显示车辆的预测位置。

##### 摄像头设置 

要控制视角和摄像头，请单击“摄像头设置”(camera)图标。要使用摄像头追踪特定角色，请从左上角的(i)下拉列表中选择一个角色或者在可视化器中单击以将摄像头设置为固定位置。

<p align="center">
  <a href="images/Camera_settings.png" target="_blank">
    <img src="images/Camera_settings.png">
  </a>
</p>

- 透视视图：将视角改变为俯视图。
- 跟随选择的角色：跟随所选角色。
- 将摄像头重置为所选角色：将摄像头重置到所选角色的位置。

##### 测量距离

- 如果 Visualizer 处于透视视图中，请在 Visualizer 右上角选择摄像机设置图标，并关闭透视视图选项。
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

- 在 Visualizer 中点选来设定您的测量起点，然后移动光标并点击来设定终点。
<p align="center">
  <a href="images/Measurement_3.png" target="_blank">
    <img src="images/Measurement_3.png">
  </a>
</p>

测量结果将显示在连接起点和终点的线旁边。

- 要隐藏测量结果，请关闭“测量距离”工具图标。

#### 通过跟踪信息调试

所有跟踪信息都可以在跟踪视图下进行查看。跟踪视图与时间轴对齐，因此可以轻松比较不同的跟踪信息。

**跟踪** 分为以下几种类型：

- **区间**：代表一段时间内收集的一组值。区间具有名称、起始/结束时间和类型，并且与特定的执行者相关联（橙色框）。

- **数值**：代表随时间变化的单个值，以波形图的形式显示。数值跟踪具有名称、值和单位，并且与特定的执行者相关联（红色框）。

<p align="center">
  <a href="images/Traces_1.png" target="_blank">
    <img src="images/Traces_1.png">
  </a>
</p>

为了增强可视化效果，Foretify 记录了每个场景的开始和结束时间。

##### 查看区间：
- 在 Foretify 中选择“调试运行”选项卡，然后点击 Visualizer 下的“跟踪”选项卡：

```markdown
<p align="center">
  <a href="images/Traces_2.png" target="_blank">
    <img src="images/Traces_2.png">
  </a>
</p>

踪迹呈现为时间轴上的间隔，具有与可视化器和踪迹选项卡中的演员值踪迹相对应的当前时间光标的其他基于时间的视图。

1. 单击踪迹名称左侧的箭头以展开并查看其子场景。

2. 单击踪迹以查看踪迹详情，例如踪迹类型、演员、时间、持续时间以及间隔期间收集的指标。

3. 在踪迹详情下，单击踪迹的开始时间或结束时间以将通用时间轴设置为该时间。

##### 将时间轴对准踪迹：
1. 在踪迹选项卡上，选择要将时间轴对准的间隔（橙色框）。

2. 单击“对准时间轴”图标（红色框）。

<p align="center">
  <a href="images/Intervals_3.png" target="_blank">
    <img src="images/Intervals_3.png">
  </a>
</p>

- 要重置时间轴，使其不再对准该间隔，请单击时间轴右侧的“取消对准时间轴”图标。

<p align="center">
  <a href="images/Intervals_4.png" target="_blank">
    <img src="images/Intervals_4.png">
  </a>
</p>

!!! 示例：“实践时间”
    使用可视化器重新运行测试，并检查系统正在测试的行为以及上述不同选项。

### 运行不同的种子

!!! 示例：“实践时间”
    在右上角的执行控制区域使用种子号将另一个模拟设置成4，并单击**运行测试**按钮来运行另一个模拟。在这个种子中，我们引入的最小距离阈值检查器将失败。

    使用您选择的种子再运行一次模拟。

    再次搜索日志，查找指示切入发生在哪一边的消息。
```

```markdown
现在，您可以通过在Foretify终端中键入`exit`或关闭Foretify窗口来关闭Foretify。

!!! Info
    **种子**作为随机生成具体执行的输入，从抽象场景定义中。使用**相同的种子**再次生成具体测试，会得到**相同的具体变体**。这是一个关键功能，既可以在一定程度上完全随机化测试，又可以为调试目的重新创建单一具体执行。

    多亏了种子的定义和实现，您始终可以**追踪生成的特定具体变体**。场景的可追踪性是一个基本特性，让您可以确定导致错误的条件，并重现它们。

## Foretify Manager

现在您已经运行了第一批测试，可以探索已实现的覆盖范围和结果。我们将在Lab 2中正式定义覆盖率，但现在您可以使用Foretellix工具视觉化地探索这个概念。

Foretify Manager是Foretellix工具，可以让您可视化验证过程的整体状态，导入多次测试执行的结果和关键绩效指标（KPI）。它具有如下图所示的客户端-服务器架构：

<p align="center">
  <a href="images/fmanager_architecture.png" target="_blank">
    <img src="images/fmanager_architecture.png">
  </a>
</p>

客户端可以是Python脚本或Web UI（网页），两者都可以对测试套件结果数据执行操作和查询。服务器管理数据库并执行客户端的命令。这种拓扑结构允许多个用户同时分析验证结果的不同方面。

### 打开Foretify Manager

Foretify Manager是基于浏览器的应用程序。
```

```markdown
!!! Example "实际操作时间"
    通过在终端中输入以下命令启动 Foretify 管理器：

    ```bash
    fmanager
    ```

    第一件事就是看到登录页面：

    <img src="images/fmanager_login.png" alt="image" width="200"/>

!!! Info
    如果需要更多关于指定用户名和密码的信息，请联系演讲者。

### 创建一个项目

Foretify 管理器项目是一个协作框架，使用户能够为验证和验证数据以及收集的指标设置权限和所有权。

创建项目的步骤如下：

1. 打开 Foretify 管理器，并使用 Foretify Manager 凭据登录。

2. 在选择项目页面，单击“创建新项目”。

!!! Info
    如果需要更多关于项目名称的信息，请联系演讲者。

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

此时，Foretify Manager 数据库是空的，所以下一步是上传执行的少数测试的结果。

!!! Example "实际操作时间"
    切换到终端并通过以下方式上传结果：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir
    ```

您可以使用```--run_group_name```参数为您的一组测试命名。详见 [upload_runs 文档](../fman_user/fmanuser_launch_test_suite.md#upload-a-regression)。
```

```markdown
!!! Example "实际操作时间"
    切换回浏览器中打开的 Foretify Manager 网页应用，点击刷新按钮（确保在“测试套件结果”选项卡上）

上传后，测试执行应该会被加载并显示在您之前创建的项目中：

<p align="center">
  <a href="images/fmanager_regression_2.png" target="_blank">
    <img src="images/fmanager_regression_2.png">
  </a>
</p>

### 分析已上传的运行记录

通过点击刚刚导入的测试套件，您可以查看单独运行的情况。

您应该会看到如下面图片中的内容：

<p align="center">
  <a href="images/fmanager_runs.png" target="_blank">
    <img src="images/fmanager_runs.png">
  </a>
</p>

彩色方块的高亮部分表示：

- _黄色_: 运行列表右上角的图标允许您导出和删除运行，以及保留和重置您的选择。使用最右侧的列选择图标添加和移除运行属性，比如目录、操作系统用户、持续时间等。
- _橙色_: _问题树_ 以其种类分组显示所有问题。
- _蓝色_: _聚合视图_ 让您根据运行属性来进行运行的聚合。

在 _运行_ 视图中点击一个运行，将会弹出一个新的 Foretify Manager 窗口。

<p align="center">
  <a href="images/Run_view_2.png" target="_blank">
    <img src="images/Run_view_2.png">
  </a>
</p>

如您所见，有两个主要选项卡：**调试运行**和**运行摘要**。
**调试运行**选项卡与我们之前在 Foretify 中看到的一样，

<p align="center">
  <a href="images/run_source.png" target="_blank">
    <img src="images/run_source.png">
  </a>
</p>

点击**运行摘要**选项卡，您可以查看关于失败运行的更详细信息。
```

```markdown
<p align="center">
  <a href="images/run_summary.png" target="_blank">
    <img src="images/run_summary.png">
  </a>
</p>

!!! 例子 "动手时间"
    点击每个运行，现在就可以查看其它两个运行。

### 工作区是什么以及如何创建它

工作区是一组导入的测试套件，您希望分析覆盖数据。

!!! 例子 "动手时间"
    在 **测试套件结果** 选项卡中选择您的测试套件，然后如下所示单击 **创建工作区**。

<p align="center">
  <a href="images/fmanager_workspace_2.png" target="_blank">
    <img src="images/fmanager_workspace_2.png">
  </a>
</p>

选择一个工作区名称。然后单击 **创建工作区** 创建工作区。

<p align="center">
  <a href="images/fmanager_workspace_name_2.png" target="_blank">
    <img src="images/fmanager_workspace_name_2.png">
  </a>
</p>

Foretify Manager web 应用程序切换到 **当前工作区** 视图：

<p align="center">
  <a href="images/fmanager_workspace_after_creation.png" target="_blank">
    <img src="images/fmanager_workspace_after_creation.png">
  </a>
</p>

工作区包括以下内容：

- **VGrade** 是总体度量等级（您将在实验 4 中了解它）。
- **总运行数**（在 **VGrade** 旁边）是通过和未通过的运行的统计数据。
- 在 **VPlan** 选项卡（蓝色）中，您可以看到度量层次结构。
- 在 **运行** 选项卡（绿色）中，您可以看到当前工作区所选的运行列表。

!!! 例子 "动手时间"
    查看 **VPlan** 树和 **运行** 选项卡中的运行。
    
### 度量和检查器表示
```

#### 覆盖率
在我们的 `ts_l01_intro_cov.osc` 覆盖文件中，我们定义了四个覆盖项和一个KPI指标。
例如，cut_in_side的覆盖等级为100％，而speed_sut的覆盖等级仅为20％。这表明需要运行更多的测试来填补speed_sut覆盖范围的空白区域。
在我们的文件中，这两个覆盖项的定义略有不同：

 - 注意：百分比可能会根据使用的种子而有所不同！

<p align="center">
  <a href="images/lab01_coverage_conclusions.png" target="_blank">
    <img src="images/lab01_coverage_conclusions.png">
  </a>
</p>

对于speed_sut，我们指定了存储桶，而对于cut_in_side，我们没有强加任何规则。这为cut_in_side覆盖项提供了额外的自由度。该项的覆盖等级为100％，因为在测试过程中至少击中了两边（左右）中的一边。

!!! Example "实践时间"
    现在检查其他覆盖项及其等级，可以在foretify manager中，在VPlan选项卡下找到_cut_in_side_覆盖项。

#### KPIs
正如您所记得的，我们之前定义了distance_kpi KPI，它衡量了SUT和变道结束时切入车辆之间的距离。指定事件有助于我们更好地捕捉项目值的最高兴趣点。由于还指定了事件，我们注意到此KPI每次运行都只有单一值。

<p align="center">
  <a href="images/workspace_kpi_2.png" target="_blank">
    <img src="images/workspace_kpi_2.png">
  </a>
</p>

!!! Example "实践时间"
    您得到了哪些KPI值？您可以在Visualizer中查看模拟，并查看值与模拟之间的相关性。

#### 检查员

在查看**Runs**标签页时，您可以观察到通过了多少次运行和失败了多少次运行。失败的运行是来自于代码中定义的检查器。由于检查器在SUT未能满足布尔条件时定义了一个失败响应，这立即反映在了Issues Tree中。

<p align="center">
  <a href="images/checkers.png" target="_blank">
    <img src="images/checkers.png">
  </a>
</p>

!!! 举例 "动手实践时间"
    分析您的运行。您得到了多少次失败的运行？

### 提高覆盖率

少数测试远远不足以实现任何有意义的覆盖率。Foretellix技术的优势在于可以运行大量自动生成的测试。因此，在这一节中，您将运行更多的测试以增加覆盖范围。为此，您需要根据情景创建不同的测试用例。如前所述，这由创建过程中使用的种子来控制。

**在继续之前，请确保关闭浏览器中的Foretify GUI**。

!!! 举例 "动手实践时间"
    让我们运行15个额外的测试，设置一个新的工作目录。请注意，我们将不使用foretify gui，因此您将会看到模拟器窗口不断弹出。

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir2 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 20 --crun 15
    ```

    ```--crun``` 选项将从种子20开始运行15个测试（也就是从种子20到34）。这是扩大测试的一种选项，但在实验室3中，我们将涵盖更多选项。

请注意，生成引擎会在地图的不同区域生成15个独特的具体测试，所有具体测试均在抽象场景定义的边界内（例如，速度、车道位置等）。

!!! Example "动手时间"
    新的运行应该会增加覆盖率。你现在可以使用以下命令将这15个额外的运行上传到Foretify Manager：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir2
    ```

    然后，在Foretify Manager的**测试套件结果**标签中，选择新上传的测试套件，并按照下图所示将其添加到工作区。

    现在你可以探索**VPlan**树，查看覆盖率的提升情况。

<p align="center">
  <a href="images/l01_add_ws_1.png" target="_blank">
    <img src="images/l01_add_ws_1.png">
  </a>
</p>

### 在测试套件之间切换

当你将新的测试套件添加到工作区后，你可以在这些测试套件之间切换，从而单独分析每一个。

!!! Example "动手时间"
    进入你的工作区，并点击**测试套件结果工作区视图**中的向下箭头。

<p align="center">
  <a href="images/l01_switch_ws_1.png" target="_blank">
    <img src="images/l01_switch_ws_1.png">
  </a>
</p>

一个新窗口会出现，显示可用的测试套件。你可以在它们之间切换以查看差异。

<p align="center">
  <a href="images/l01_switch_ws_2.png" target="_blank">
    <img src="images/l01_switch_ws_2.png" width="50%">
  </a>
</p>

加载新测试套件后，你需要点击**计算**按钮。

```markdown
<p align="center">
  <a href="images/l01_switch_ws_3.png" target="_blank">
    <img src="images/l01_switch_ws_3.png">
  </a>
</p>

### 对测试套件进行分组

当您有多个测试套件时，有时希望将它们分组，以扩大工作区的覆盖范围。

!!! 示例 "动手时间"
    转到您的工作区，并单击**Workspace Test Suit Results**，选择测试套件，然后单击**Group**图标以对它们进行分组。

<p align="center">
  <a href="images/l01_group_ws_1.png" target="_blank">
    <img src="images/l01_group_ws_1.png">
  </a>
</p>

会出现一个新窗口，请为您的分组命名，然后单击**Group**。

<p align="center">
  <a href="images/l01_group_ws_2.png" target="_blank">
    <img src="images/l01_group_ws_2.png" width="50%">
  </a>
</p>

您会看到运行结果已更新，覆盖范围也增加了。

<p align="center">
  <a href="images/l01_group_ws_3.png" target="_blank">
    <img src="images/l01_group_ws_3.png">
  </a>
</p>

### 对测试套件进行取消分组

也可以取消您创建的分组。

!!! 示例 "动手时间"
    转到您的工作区，然后单击**Workspace Test Suit Results**，选择已分组的运行结果，然后单击**Group Info**图标以取消分组。

<p align="center">
  <a href="images/l01_ungroup_ws_1.png" target="_blank">
    <img src="images/l01_ungroup_ws_1.png">
  </a>
</p>

会出现一个新窗口，请单击**Ungroup**按钮。

<p align="center">
  <a href="images/l01_ungroup_ws_2.png" target="_blank">
    <img src="images/l01_ungroup_ws_2.png" width="50%">
  </a>
</p>

### 更改地图

**在继续之前，请确保在浏览器中关闭 Foretify**。
```

### 将文本翻译成中文：

另一种增加覆盖率的方法是更改执行测试的ODD。OSC2语言的一个强大特性是，为了更改ODD，你只需改变一行代码。

通过新的地图，Foretify 将在地图拓扑支持场景的位置随机生成新的具体测试案例。

!!! 例子 "实践时间"
    打开文件 `$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`，在该文件中定义了测试。

    编辑 `ts_l01_intro.osc` 文件中的场景，并按以下方式更改定义地图的行：

    ```osc linenums="8"
    extend test_config:
        set map = "$FTX_WORKSHOP/maps/cloverleaf.xodr"
    ```
    现在，您可以使用以下命令再次运行测试5次：

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 2 --crun 5
    ```

    您现在可以观察到场景在另一个地图上执行。

!!! 注意

    如果您为特定场景选择的地图不允许执行该场景，则Foretellix工具将指出这一点作为矛盾错误。例如，无法在单车道上执行变道场景。

!!! 例子 "实践时间"
    现在您已经运行了额外的测试，可以将它们上传到Foretify Manager查看覆盖率是否有所改善。为了做到这一点，您可以运行以下命令：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir3
    ```
    将新的运行结果添加到之前创建的工作空间中。

## 接下来的步骤

本实验室的目标是

- 熟悉用于研讨会的云环境
- 浏览一些适用于渗透场景的基本OSC2代码
- 以交互模式运行Foretify并载入渗透场景
- 熟悉种子的概念
- 熟悉用于测试的OSC2及其主要组件
- 打开Foretify Manager并探索收集到的指标

接下来，在实验室2中，您将扩展渗透场景，收集更多覆盖范围的指标，并介绍在Foretify中调试场景的方法。

> 本文由ChatGPT翻译，如有任何遗漏，请[**反馈**](https://github.com/linyuxuanlin/Wiki_MkDocs/issues/new)。