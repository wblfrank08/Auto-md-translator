---
chapnum: 2
---

# 实验室 2：场景重用介绍

## 学习目标 - [自动化测试]

这个实验室是以安全驱动的验证周期的第二次迭代。在这个实验室中，您将学习如何调整OSC2验证和验证（V&V）资产：

- **调整场景**：
    - 添加新的约束条件
    - 使用驱动移动操作
    - 使用并行和串行操作符扩展场景
    - 重用现有的_cut_in_场景

- **在Foretify中检查运行情况**：
    - 使用Foretify日志文件检查运行情况
    - 使用跟踪和可视化器检查运行情况
    - 检查时间间隔

- **覆盖率和性能指标收集**：
    - 收集覆盖率
    - KPI实施
    - 检查器实施

## 调整场景

在这一部分，您将学习可以利用来构建或修改OSC2场景的原则和语言构造。您将重用上一次提出的相同场景，并根据新的要求进行调整。

### 添加约束条件

这个实验室探讨了三种基本约束条件 - 其他汽车的颜色和类型以及插入发生的一侧。根据您想要约束什么，您可以将约束条件添加到场景文件（在本例中为`$FTX_WORKSHOP/scenarios/cut_in_l01.osc`）或测试文件（在本例中为`$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`）。为了保持代码的可重用性，最佳做法是在测试文件中添加所需的约束条件，您将在这个实验室中看到。 

!!! 示例 "亲自操作时间"
    打开文件`$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`，在这里定义了测试。

    编辑`ts_l01_intro.osc`文件中的场景，并添加上述约束条件：

```osc linenums="13"
扩展 top.main:
    进行 cIL：sut.cut_in_l01() 使用：
        保持（it.car1.category == van）
        保持（it.car1.color 在 [white, blue] 内）
        保持（it.side == right）
```

重新启动Foretify，将其设置为新的工作文件夹，并执行10次运行：

```bash
foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --crun 10 --batch
```

在场景运行时，注意新引入的约束如何被执行。特别是你可以观察到cut_in始终来自右侧，执行cut_in的车辆是白色或蓝色的货车。


### 驱动行动及场景操作符

在开发基本或复杂的OSC2场景时，有三个重要的支柱需要牢记：

- 驱动行动
- 并行操作符
- 串行操作符

通过在重用现有的_cut_in_场景并编写_cut_in_and_slow_场景中实践，你将学会这些支柱的重要性。


#### _drive_ 驱动行动

任何车辆角色都提供 _drive_ 驱动行动，用于控制车辆移动。仅凭它本身，_drive_ 会导致车辆沿着随机路径行驶，并受到在车道内的柔性约束。在本实验中，您将添加修改器以约束路径和速度。

对于 _sut_ 角色和另一种 _vehicle_ 类型的角色（_car1_），您可以按如下方式调用 _drive_ 驱动行动：

```osc linenums="1"
sut.car.drive()

car1.drive()
```

要向驾驶运动添加修改器（例如，保持一定速度或在某一车道行驶），使用以下方式：

```osc linenums="1"
sut.car.drive() with:
    <modifiers>

car1.drive() with:
    <modifiers>
```

### 并行运算符

并行运算符提供了执行两个或多个子模块的灵活性。您还可以控制不同子模块的执行之间的重叠。在下面的示例中，所有在 _parallel_ 运算符下的子模块都具有相同的持续时间，因为 _overlap_ 被设置为 _equal_。在高级实验室中，您将看到如何调整重叠。

考虑下面的基本示例，其中 SUT 车辆和另一辆车辆正在行驶。您可以在这里找到此示例：`$FTX_WORKSHOP/scenarios/parallel_example.osc`：

<p align="center">
  <a href="images/workshop_l02_parallel.png" target="_blank">
    <img src="images/workshop_l02_parallel.png">
  </a>
</p>

在包含并行运算符的这个场景中，两个参与者正在执行以下操作：

- SUT 车辆在没有速度或位置的明确限制的情况下行驶。
- car1 车辆在没有速度的明确限制的情况下行驶，但在位置上有一个相对的限制：在场景开始时至少比 SUT 车辆前进 10 米。

!!! 示例 "动手时间"
    您现在可以使用以下命令多次执行场景：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02_parallel.osc \
    --batch --seed 5 --crun 3 --work_dir $FTX_FM_WORKDIR/l02_second_iter/work1
    ```

    观察场景的展开方式。请注意，在本研讨会中，默认将 SUT 车辆设置为红色。您应该观察到绿色车辆在场景开始时始终领先于红色车辆，独立于具体的种子。

### 串行运算符

接下来，你将使用 _serial_ 操作符来扩展示例场景。顾名思义，位于 _serial_ 操作符下的两个或多个块将依次执行。这为场景引入了隐含的约束，确保执行的物理连续性。

你也可以在 `$FTX_WORKSHOP/scenarios/serial_example.osc` 中找到这个示例：

<p align="center">
  <a href="images/workshop_l02_serial.png" target="_blank">
    <img src="images/workshop_l02_serial.png">
  </a>
</p>

在这个同时包含了串行和并行操作符的场景中，两个角色正在执行一个包括两个阶段的场景：

- **phase1**：与之前的示例类似，但这次车辆1被限制在与系统下测试（SUT）相同的车道上行驶，并且在这个阶段内最多减速 20 公里/小时。此外，此时的持续时间范围为 2 到 6 秒。
- **phase2**：在这个阶段，车辆1只能在最多增加 20 公里/小时的速度限制下行驶。

**phase1** 和 **phase2** 是 OSC2 中用作轻松引用场景不同部分的标签。
在后续的实验中，你将更多地了解关于标签的内容。

使用串行操作符总是引入一些隐含的约束，以下是针对这个示例列出的部分隐含约束：

- _phase2_ 开始时刻与 _phase1_ 结束时刻相同。
- _car1_ 和 _sut_ 在 _phase2_ 开始时的速度等于它们在 _phase1_ 结束时的速度。
- _car1_ 和 _sut_ 在 _phase2_ 开始时的位置等于它们在 _phase1_ 结束时的位置。

Foretify 会自动处理隐含约束的解决，确保在整个场景中始终保持物理连续性，而无需你进行手动管理。

```markdown
!!! Example "操作时间"
    您现在可以使用以下命令几次执行该场景：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02_serial.osc \
    --batch --seed 3 --crun 3 --work_dir $FTX_FM_WORKDIR/l02_second_iter/work1
    ```

    观察场景的展开过程。请记住，在本次工作坊中，默认将系统单元测试设置为红色。您应该观察到在场景开始时，绿色车辆总是领先于红色车辆。然后它会减速并再次加速，这与特定种子无关。


### 场景复用

在本实验中，您将开始编写自己的 OSC2 代码。您将要实现的场景是“插入并减速”。一辆车在系统单元测试前面插入，然后开始减速。

OSC2 的主要优点之一是可以将基础场景重复使用，作为构建更复杂场景的组成部分，这在创建新的更复杂场景时节省了资源。

您正在实现的场景包括两个阶段：插入阶段和减速阶段。您将根据之前章节学到的内容，首先实现这两个阶段。

#### 创建 _cut_in_and_slow_ 场景和 ODD 拓扑规范

!!! Example "操作时间"
    首先通过以下命令使用代码编辑器打开 `cut_in_and_slow_l02.osc`：

    ```bash
    code $FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02.osc
    ```

通过查看代码，您可以看到第一行的导入语句导入了在实验 1 中使用的相同场景 (`cut_in_l01.osc`)。

为了重复使用插入场景，您首先需要创建 _cut_in_and_slow_ 场景的结构。
```

```markdown
!!! Example "Hands-on Time"
    您现在可以尝试通过编辑 `cut_in_and_slow_l02.osc` 文件来添加执行剪入和减速所需的代码。
    由于我们还没有讲解如何实现减速，请专注于剪入和驾驶场景，其中汽车在剪入后继续移动。请记住，OSC2 是基于缩进的语言，而 Foretify 工具要求使用空格（而非制表符）作为缩进方式。

!!! tip
    代码应首先执行 `cut_in_l01.osc` 中定义的剪入操作，然后再进行减速。您可以使用稍后会解释的修饰符来实现减速。现在尝试结构化机动的不同阶段。

??? Solution "解决方案 - 点击这里"
    ```osc linenums="1"
    scenario sut.cut_in_and_slow:
        car1: vehicle
        side: av_side

        do serial():
            cut_in: cut_in_l01(car1: car1, side: side)
            slow: parallel(duration:[2..5]second, overlap:equal):
                sut.car.drive()
                car1.drive()
    ```

上述解决方案代码负责以下内容：
```

- 已创建场景_sut.cut_in_and_slow_。
- 已实例化两个对象_car1_和_side_。它们分别属于预定义类型_车辆_和_av_side_。
- 在_do serial()_语句之后，定义了两个阶段，这两个阶段将按顺序进行：
    - 标记为_cut_in_的第一个阶段：该阶段简单地调用了第一行导入的场景_cut_in_l01_。在这里，为_cut_in_and_slow_实例化的对象_car1_和_side_作为参数传递给_cut_in_l01_。注意如何将先前定义的场景作为更复杂场景的“构建模块”进行调用，这是一个非常强大的功能。
    - 标记为_slow_的第二个阶段：在这里，使用了并行运算符，并且持续时间设置在2至5秒的范围内。

!!! 注意
    目前在第二阶段中，SUT和car1都在行驶。稍后您将添加额外的约束条件，以确保car1减速。

!!! 示例“实际操作时间”
    保存后，运行刚刚创建的几次执行，使用以下命令：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --seed 5 \
    --batch --crun 2
    ```

    观察场景，它应该类似于原始的变道场景，其中包括另一个部分，两个参与者只是在行驶。

!!! 信息
    能够重用、组合和混合新场景中的现有场景，极大地减少了编写新场景所需的工作量。每个场景都成为一个基本单元，您可以将其与其他现有单元组合，以创建更复杂的场景，其中强加的约束条件合并在一起。

结果方案的可变性是其基本元素可变性的产物。这一特点提高了搜索新错误的效率，因为所有可能的组合都可以纳入考虑范围。

到目前为止，你实施的只是一个 _cut_in_and_drive_ 的场景。现在，你将学习如何添加到 _slow_ 部分以使 _car1_ 减速所需的修改器。

#### _drive()_ 运动场景的修改器

接下来，你将进一步调整场景，使 _car1_ 在 _slow_ 阶段减速。为了做到这一点，你需要在 _drive()_ 运动场景中引入一些额外的修改器。这些修改器将影响 _car1_ 和 _sut_ 的速度和车道。

这并不意味着 OSC2 只允许在演员的速度或车道上使用修改器。还有许多其他修改器可用于约束，例如，车辆相对于另一个演员的位置或其加速度，就像串行和并行操作符的基本示例中那样。

请注意，OSC2 中写入的每个阶段都带有预定义的 "_start_" 和 "_end_" 事件。这就是为什么在定义新的修改器时可以使用这些事件。

##### _change_speed_ 修改器

为了实现一个 cut_in_and_slow 场景，我们需要使用 _change_speed_ 修改器。_change_speed_ 通过给定的数值来改变演员的速度，该数值可以是正值或负值（分别表示加速或减速）。当你想要控制与当前段开始时的车辆速度相对的速度增加或减少时，这一功能就非常有用。

!!! 例子 "动手时间"
    再次用代码编辑器打开 `$FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02.osc` 并按以下方式修改场景：

```osc linenums="1"
import "$FTX_WORKSHOP/scenarios/cut_in_l01.osc"

scenario sut.cut_in_and_slow:
    car1: vehicle
    side: av_side

    do serial():
        log("@cut_in.start car1 speed=$(car1.state.speed)")
        cut_in: cut_in_l01(car1: car1, side: side)
        log("@cut_in.end car1 speed=$(car1.state.speed)")
        log("@slow.start car1 speed=$(car1.state.speed)")
        slow: parallel(duration:[2..5]second, overlap:equal):
            sut.car.drive() with:
                keep_lane()
            car1.drive() with:
                change_speed(-[15..]kph, run_mode: best_effort)
                keep_lane()
        log("@slow.end car1 speed=$(car1.state.speed)")
```

新增内容为何？

该示例介绍了在第14行引入的_change_speed_修改器，该修改器限制_车辆1_的速度在_slow_阶段的开始和结束之间至少减少15千米每小时。这种减速完成了切入和减速场景。

!!! 注意
    你注意到了吗？我们再次使用了_run_mode: best_effort_，但这次作为修改器的内联参数。我们这样做是因为该场景将被计划为与模拟器无关，这意味着模拟器可能以意想不到的方式限制车辆的减速。在这种情况下，我们不希望由于不完整的场景而导致运行失败。

!!! 例子 "动手时间"
    保存后，使用以下命令运行几次执行：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --batch --crun 5
    ```

```markdown
可以观察场景如何展开并查看日志消息。它们是否符合你的预期？

检查最后一条日志消息。由于刚刚添加的限制条件，car1 的速度应该至少比起始速度慢15公里每小时。

!!! Info
    使用 change_speed 修饰器，你可以指定速度增加或减少的具体数值。

    这是一个强大的功能，可以提高给定场景的抽象级别。

### 添加新阶段

!!! Example "实践时间"
    在 _do serial_ 中增加一个名为 _speed_up_ 的阶段（建议持续时间为3到5秒），在这个阶段中，_car1_ 将保持其车道并加速至比之前快至少15公里每小时的速度。与降速一样，速度增加有时会受到模拟器和道路曲率等外部因素的限制。在这种情况下，我们不希望场景失败，但会接受结果速度的变化。可以通过将 _change_speed_ 修饰器的 _run_mode_ 设置为 _best_effort_ 来定义这一点。

    包括一些日志消息以改善调试过程。

    保存后，使用以下命令再次运行测试：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2  --run --gui
    ```
??? Solution "解决方案 - 点击这里"
    ```osc linenums="1"
    import "$FTX_WORKSHOP/scenarios/cut_in_l01.osc"
```

```markdown
Translate into Chinese:

    do serial():
        log("@cut_in.start car1 speed=$(car1.state.speed)")
        cut_in: cut_in_l01(car1: car1, side: side)
        log("@cut_in.end car1 speed=$(car1.state.speed)")
        log("@slow.start car1 speed=$(car1.state.speed)")
        slow: parallel(duration:[2..5]second, overlap:equal):
            sut.car.drive() with:
                keep_lane()
            car1.drive() with:
                change_speed(-[15..]kph)
                keep_lane()
        log("@slow.end car1 speed=$(car1.state.speed)")
        log("@speed_up.start car1 speed=$(car1.state.speed)")
        speed_up: parallel(duration:[3..5]second, overlap:equal):
            sut.car.drive() with:
                keep_lane()
            car1.drive() with:
                cs: change_speed([15..]kph, run_mode: best_effort)
                keep_lane()
        log("@speed_up.end car1 speed=$(car1.state.speed)")
    ```

### 如何使用 Foretify 检查运行情况

在开发或审查场景时，您需要能够检查运行情况。Foretify 提供了一个集成的场景可视化工具，您在上一个实验中已经探索过。现在您将探索一些更高级的功能，进一步帮助您。

有许多方法可以检查运行情况。接下来，您将看到一些广泛使用的方法：

- 使用 _log()_ 消息和 Foretify 日志文件。
- 使用跟踪功能以及可视化工具。


#### 使用 Foretify 日志文件检查运行情况

您可以通过审阅 Foretify 日志文件来检查运行情况。


```markdown
!!! Example "实际操作时间"
     使用代码编辑器打开日志文件，并在日志中搜索在前几节中介绍的消息。最近几次运行的日志可以在 `$FTX_FM_WORKDIR/l02_second_iter/workdir2/runs/```timestamp```/run.log` 下找到，或者在 `$FTX_FM_WORKDIR/l02_second_iter/workdir2/logs/```timestamp```` 中找到。

#### 使用跟踪和可视化工具检查运行状态

跟踪使您能够绘制变量随时间的变化情况。这个功能在与 Foretify 可视化工具相结合时非常有用。您可以在 OSC2 方案本身中定义自定义跟踪，但在本练习的范围内，您将使用为执行者提供的预定义跟踪。

!!! Example "实际操作时间"
   在关闭打开的 Foretify 会话后，以 GUI 模式再次打开 Foretify，并加载创建的最后一个场景，使用以下命令。

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --gui
    ```

    通过点击右上角的 **Terminal** 按钮或按 _t_ 键打开 Foretify 终端。

    默认情况下，多个跟踪是活跃的。您可以通过输入 ```trace``` 命令来查看活跃的跟踪。

    其他有用的跟踪命令包括：

    - ```trace --help```: 查看所有跟踪命令选项
    - ```trace --off```: 停用所有跟踪
    - ```trace *.*```: 跟踪给定通配符层次结构下的所有项目

    <p align="center">
        <a href="images/l02_traces.png" target="_blank">
            <img src="images/l02_traces.png">
        </a>
    </p>

    关闭所有跟踪，并激活仅绘制车辆速度的跟踪，使用以下命令：

    ```bash
    trace --off *.*
    trace *.speed
    ```
```

```markdown
这应该显示被追踪的两辆车的速度，如下所示：

<p>
  <a href="images/l02_trace_speed.png" target="_blank">
    <img src="images/l02_trace_speed.png">
  </a>
</p>

运行测试（可以在Foretify终端中使用“run”命令或使用GUI按钮）。一旦测试完成，您可以切换到下面显示的**Traces**选项卡：

<p>
  <a href="images/l02_show_traces.png" target="_blank">
    <img src="images/l02_show_traces.png">
  </a>
</p>

现在，您可以通过单击和拖动时间导航器（橙色圈出）来选择轨迹上的不同时间戳。这将使可视化器移动到相应的时间戳，以便您可以看到模拟中在那个时间戳发生了什么。

<p>
  <a href="images/l02_trace_select_time.png" target="_blank">
    <img src="images/l02_trace_select_time.png">
  </a>
</p>

#### 使用Traces间隔检查运行

现在您有一个包含连续发生的多个阶段的场景，您将会欣赏到Foretify GUI中显示的间隔带来的益处，这些间隔准确告诉您每个阶段何时开始和结束，您可以在下面的屏幕截图中看到：

<p align="center">
  <a href="images/l02_intervals.png" target="_blank">
    <img src="images/l02_intervals.png">
  </a>
</p>

## 覆盖范围收集

度量驱动的验证与验证（V&V）是一个过程，通过度量定期测量验证结果，以便可以根据收集的度量做出决策。这带来了许多好处。度量驱动的V&V：
```

- 从 V&V 状态评估中消除了人为错误 —— V&V 状态是客观的
- 允许工程师调整决策并将精力集中在客观的度量上
- 通过从一次迭代到下一次的即时反馈加速 V&V 过程
- 提供跨决策层次和细节层次的透明度 —— 从工程师到首席执行官，从 SUT 速度到整体指标分数
- 使 V&V 过程不受 SUT 行为的影响 —— 指标反映了实际发生的情况，而不是工程师期望发生的情况
- 对规模不敏感 —— V&V 与工作量的大小自然匹配
- 允许评估 SUT 的质量，也就是说，可以基于客观统计数据做出 GO/NO-GO 决策

您可能已经注意到，基于度量的验证和验证与基于测试的验证和验证有所不同，在基于测试的验证和验证中，进展是通过通过测试的数量来衡量的。可以说，正是由于以上原因，与基于测试的 V&V 相比，使用基于度量的 V&V 带来了许多优势。

Foretellix 解决方案通过覆盖度量来补充传统的关键绩效指标，这是本节的主题。这意味着您仍然可以使用传统的方式来评估质量，同时添加新的度量标准以增加验证目标定义的精确度。

### 什么是覆盖度？

工程师定义验证度量来以可衡量的方式描述验证目标。覆盖度定义指定必须进行测试的属性/参数值。例如，假设为了证明 SUT 在切入场景中的行为是正确的，您需要确保以下两个参数在给定的时间点内在给定范围内运行：

| 覆盖项       | 范围     | 桶       | 单位 | 时间       |
|--------------|----------|----------|------|------------|
| sut_speed    | [0..200]kph | 每 10    | kph  | 切入结束时 |
| car1_distance| [0..50]m   | 每 5     | m    | 切入结束时 |

我们为什么选择 [0..200] 和 [0..50] 的范围？答案是这些范围是基于工程规范定义的，该规范捕捉了系统的工作方式以及它的优缺点。例如，如果一个覆盖项被建模为整数，那么它会有 2^32 个值的值空间。但从验证的角度来看，并不是所有这些值都是有效或有用的。因此，上表指定了所谓的“有效值空间”，将样本收集限制在相关值内。限制范围确保了涵盖有趣的值，而不浪费验证资源在不相关或无效的空间上，比如一个 NPC 以超音速驾驶。

限制覆盖项的值空间仅限于指定的有效值是 V&V 工程师的决策。即使在合法的值范围内，也可以细化并创建称为桶的小范围（在本例中，分别为 10kph 桶和 5m 桶）。这些桶的创建是基于假设桶内所有值的行为和质量影响相似。_时间_ 参数指定了采样点——即参数值保存到覆盖数据库中的时间点。

OSC2 允许在给定场景中定义覆盖项。例如，要为 sut.cut_in 场景定义表格中的第一个覆盖项，可以编写以下代码：

```osc linenums="1"
# 这里我们扩展了cut_in场景
extend sut.cut_in:
    # 这里我们决定何时取样和取什么信号
    var sut_speed := sample(sut.car.state.speed, @cut_in.end)
    cover(name: sut_speed,
            # 覆盖项的描述
            text : "汽车刚切入时的绝对速度（以km/h为单位）",
            # 覆盖项代表的物理单位
            unit : kph,
            # 相关值的范围，范围外的值会被忽略
            range : [0..200],
            # 桶的大小（例如，[10..20]，[20..30]等）
            every : 10)
```
!!! tip
  
    请参阅[cover()和record()参数](../osc_lang/osclang_behavior_monitoring.md#cover-and-record-parameters)以获得`cover`的语法和定义。

现在假设上述两个覆盖项目都已经实施，并且运行了77次模拟。结果将类似于：

<p align="center">
  <a href="images/workshop_lab2_61_cover_item_sample_1.png" target="_blank">
    <img src="images/workshop_lab2_61_cover_item_sample_1.png">
  </a>
</p>

<p align="center">
  <a href="images/workshop_lab2_61_cover_item_sample_2.png" target="_blank">
    <img src="images/workshop_lab2_61_cover_item_sample_2.png">
  </a>
</p>

你可以看到，并非所有的桶都被命中，这意味着覆盖项目并未完全覆盖。但覆盖了多少？覆盖项目的等级给出了填充水平。覆盖等级是对测试过程中遇到的所有情况的多维表示。

覆盖项的等级是一个百分比：覆盖了多少个项目值占总数的百分比。在这种情况下，覆盖率如下：

- 信号生成系统（SUT）和NPC距离覆盖 - 23%：13个桶中有3个被命中（即，采样值位于桶范围内）
- SUT速度覆盖 - 38%：13个桶中有5个被命中

这意味着你需要继续运行模拟，直到获得100%的覆盖等级。如果一些桶没有被命中，你可以决定改变场景约束以针对不同值。

!!! 信息
    基于覆盖率的验证（CDV）是一种被证明成功的方法论，被广泛应用作为各种领域的行业标准，在被测试设备非常复杂的情况下，特别是半导体行业。这种方法可以在产品上市前持续发现软件缺陷，避免极其昂贵的召回。

    Foretellix的专家们在半导体行业应用了20多年的CDV经验，为自动系统的V&V安全性提供解决方案。借助Foretellix的工具，使得CDV方法可以应用于自动驾驶车辆验证，V&V工程师可以有效地监控验证过程的状态。

    基于覆盖率的验证方法代表了安全驱动验证方法的一个重要部分。

### 如何定义覆盖项目

```markdown
当覆盖项不需要采样时（例如，它是场景的输入），可以定义如下：

```osc linenums="1"
    cover(name: aside, expression:side)
```

这将创建一个覆盖项，该覆盖项使用场景中的 _side_ 字段来开始 _cut in_ 段。一旦 _left_ 和 _right_ 的两个值均被触发，_side_ 项目的得分将达到100%（即完全覆盖）。如果只有其中一个值被触发，则项目的分数将为50%。

当覆盖项需要采样时，可以定义如下：

```osc linenums="1"
    var rel_speed := sample(car1.state.speed - sut.car.state.speed, @cut_in.end)
    cover(name: rel_speed,
            text : "Relative speed cut_in end (in kph)",
            unit : kph,
            range : [-50..50],
            every : 10)
```

如果参数是整数，覆盖空间就会很大（即2^32-1个值）。往往，并非所有整数值都是相关的。OSC2允许您像这个例子一样指定整数的合法范围（使用 _range_ 和 _every_）。

_rel_speed_ 在 _cut_in.end_ 事件上进行抽样。此时的值与包含它的桶进行匹配，并保存。一旦一个桶中有样本，该桶的得分就达到100%。根据定义，只要至少有一个样本匹配桶的区间，桶的得分就达到100%。

!!! 例子 "实践时间"
    查看 `$FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02_cov.osc` 中定义的覆盖项。您能认出并解释这些覆盖项的含义吗？请注意，在此处使用的 _map.abs_distance_between_positions_ 函数提供了两个位置之间的绝对距离。
```

```markdown
由于您已经运行了几次，您可以使用以下命令将它们收集并上传到 Foretify Manager：

```bash
upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
--runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2
```

在使用命令 `fmanager` 打开 Foretify Manager 后，按照第1实验室学习的内容，为测试运行创建一个名为 _cut_in_and_slow_ 的新工作区。最终，您应该会看到与以下图片类似的内容（请注意，百分比可能会有所变化）：

<p align="center">
  <a href="images/l02_vgarde_no_cross.png" target="_blank">
    <img src="images/l02_vgarde_no_cross.png">
  </a>
</p>

检查工作区中的覆盖度指标。您应该会看到四个项目：aside、sut_speed、rel_speed 和 car1_distance。这四个覆盖度项目的覆盖等级分别是多少？整体 VGrade 是多少？

### 如何定义覆盖度交叉

交叉是两个或多个覆盖度项目的乘积，突出了项目之间的关系。一个交叉覆盖度项目的分桶是单个项目分桶的笛卡尔积。以上面的例子为例，SUT 速度和 SUT 到 NPC 距离的交叉看起来像这样：

<p align="center">
  <a href="images/workshop_lab_5_cross_samples.png" target="_blank">
    <img src="images/workshop_lab_5_cross_samples.png">
  </a>
</p>

样本出现在绿色单元格内。与覆盖度项目类似，交叉覆盖度的等级是一个百分比：共有169个分桶中的10个被覆盖了，即5.9%。
```

将以下文字翻译成中文：

交叉的物理/法律价值空间是构成要素的物理/法律价值空间的笛卡尔积。鉴于此，工程师在设计交叉时必须小心谨慎，因为交叉空间的增长可能迅速达到难以填补的体积。

!!! 例子 "动手时间"
     将以下交叉覆盖项添加到 _$FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02_cov.osc_ 文件中：

    ```osc linenums="1"
        cover(sut_speed_x_rel_speed, items:[sut_speed, rel_speed])
        cover(sut_speed_x_car1_distance, items:[sut_speed, car1_distance])
    ```

    保存后，使用以下命令运行几次执行：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir3  \
    --batch --crun 5
    ```

    现在可以使用以下命令收集并上传运行结果到 Foretify Manager：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir3
    ```

    使用命令 `fmanager` 打开 Foretify Manager 后，创建一个名为 _cut_in_and_slow2_ 的新工作区，利用你导入的新运行结果。你能辨别出新的覆盖交叉吗？检查交叉的分箱并注意它们与创建交叉的各个要素的相关性。

## KPI 实施

关键绩效指标（KPI）可以简单到度量自动驾驶汽车的表现的原始指标。其中可以包括与安全相关的KPI，如最短碰撞时间（min-TTC），以秒计量，与舒适性相关的KPI，例如最大减速度，以米/秒计量，等等。

请将以下内容翻译成中文：

```markdown
Lab 1 introduced the notion of KPI and you exercised one example. Now, to better understand how to implement a KPI, you will write a new KPI using the _collect_ construct. This KPI will capture the _time to collision_ at each time step and record its minimum value over the course of the scenario.

We also want to _record_ the min value at the end of the scenario.

In order to do this, you will need a collect_time() and a record() invocation with the following syntax: 

    - collect_time
        - exp: the expression to evaluate at each time step
        - measure: a calculation step to apply to the collected data (e.g. min, max, mean)

    - record
        - expression: which variable or result of a formula to record
        - unit: the unit of the value to record
        - event: at which point in simulation time to record the value
        - text: a message describing the meaning of the value

The purpose of collect() is to define a performance metric or other data collection.

!!! tip 
    
    - You can add the new piece of code in the existing file: 
    ```bash
    code $FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02_cov.osc 

    ```
    - For defining the variable, you can have a look at the previous examples of declaring a variable.
    - The method to access the TTC is called _get_ttc()_.
    - The unit of measurement we want to use is second (s).

??? Solution "Solution - Click Here"

    ```osc linenums="1"
    extend sut.cut_in_and_slow:

        # Sample the time-to-collision KPI at each time step, get its min value and check if it is lower than the defined threshold
        min_ttc: collect_time(exp: sut.car.get_ttc(),
            measure: min)

        # Define an event that can be used by record
        event sample_ttc is @end

```


```markdown
# 在场景结束时记录收集构造的结果，以获取本次运行中的总最小 TTC

    记录（min_ttc_kpi,
            表达式: min_ttc.computed_result,
            单位: s,
            事件: sample_ttc,
            文本: "最小碰撞时间 (TTC)")
    ```
!!! 例子 "实践时间"
    保存后，使用更新的代码运行几次执行：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir4 \
    --batch --crun 3
    ```
    现在可以收集运行，并使用以下命令上传到 Foretify Manager：

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir4
    ```
    现在可以在 Foretify Manager 中查看 KPI 结果。

## 检查器实施

### 了解和解决问题
最主要的验证挑战之一是以最有效的方式确定系统正在运行是否正确。比如，您想知道系统是否：

- 未达到法定的最低高速公路速度。
- 未能与行人保持适当距离。
- 偏离道路。

检查器是定义系统应该或不应该如何行为的 OSC2 实体。最简单的检查器定义了如果系统未能满足布尔条件，则会产生失败响应。

### 如何实施检查器
```

当你处理场景时，可能需要定义检查器来捕捉特定于你的SUT的问题行为或条件。现在你将尝试一个与实验1类似的新例子。在这个例子中，你将适应前一个练习中新编写的KPI：min_ttc。检查器的目的是确保在慢速阶段开始时，TTC不超过3秒。

编写检查器的步骤如下：

1. 创建一个名为 _cut_in_and_slow_l02_checker.osc_ 的新文件。
2. 创建一个新的 issue_kind 并称之为 safe_ttc。
3. 为安全的TTC阈值类型定义一个新变量。
4. 将安全的TTC阈值设置为3秒。
5. 将 _cut_in_and_slow_l02_cov.osc_ 文件中的 _collect_time_ 结构转移到 _cut_in_and_slow_l02_checker.osc_ 文件中。
6. 向 _collect_time_ 结构添加以下参数：

        - bad_is: 比较结果与阈值的方向（此处为：low）
        - threshold: 要比较的阈值（此处为：我们新定义的阈值）
        - first_failure_kind: 违反阈值时应引发的警报类型（此处为：safe_ttc）
        - first_failure_severity: 违反阈值时应引发的警报严重性（此处为：error）

7. 从 _ts_l02.osc_ 文件导入新文件 _cut_in_and_slow_l02_checker.osc_。

!!! 提示
    
    实验1中的简要检查器语法提醒（标记有<>的词组需要调整）：
    ```osc
    extend issue_kind: [<checker_name>]
   
    extend sut.<scenario_name>:
    
        var <checker_threshold>: time 
        set <checker_threshold> = 3s 

    ```

??? 解决方案 "解决方案 - 点击此处查看"

    ```osc linenums="1"

    extend issue_kind: [safe_ttc]

```markdown
extend sut.cut_in_and_slow: 

    var ttc_kpi_threshold: time # 安全TTC阈值类型
    set ttc_kpi_threshold = 3s # 安全TTC阈值

    min_ttc: collect_time(exp: sut.car.get_ttc(),
        measure: min,
        bad_is: low,
        threshold: ttc_kpi_threshold,
        first_failure_kind: safe_ttc,
        first_failure_severity: error)

    event sample_ttc is @end

    record(min_ttc_kpi,
        expression: min_ttc.computed_result,
        unit: s,
        event: sample_ttc,
        text: "最小碰撞时间 (TTC)")
            
```

## 覆盖实践

该实践的目的是：

- 添加一个新的交叉覆盖项。
- 运行测试。
- 使用Foretify Manager检查结果。

!!! Example "实践时间" 
    添加一个新的交叉覆盖项，该项由切入发生的侧面和切入结束时的相对速度定义。

    使用Foretify运行额外的测试（你可以使用crun选项自动运行多次）。建议定义一个新的工作目录，例如$FTX_WORKSHOP/l02_second_iter/workdir4。

    在Foretify Manager中查看结果。你可以做些什么来增加覆盖率？

## 下一步

这就结束了第二次安全驱动验证迭代。在本次实验中，你：

- 熟悉了场景重用。
- 使用了_串行_和_并行_等场景操作符。
- 通过使用跟踪和可视化工具调试了场景。
- 熟悉了覆盖项和KPI的概念和定义。

接下来，在实验3中，你将学习如何扩大测试的数量。
```

> 本文由ChatGPT翻译，如有任何遗漏，请[**反馈**](https://github.com/linyuxuanlin/Wiki_MkDocs/issues/new)。