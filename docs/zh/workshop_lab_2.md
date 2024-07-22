---
chapnum: 2
---

# 实验二：场景重用介绍

## 学习目标 - [自动化测试]

这个实验是以安全为导向的验证循环的第二轮迭代。在这个实验中，你将学习如何定制OSC2验证和验证（V＆V）资产：

- **定制场景**：
  - 添加新的约束条件
  - 使用驱动移动操作
  - 使用并行和串行运算符来扩展场景
  - 重用现有的_cut_in_场景

- **在Foretify中检查运行情况**：
   - 检查具有Foretify日志文件的运行情况
   - 使用跟踪和可视化器检查运行情况
   - 检查间隔

- **覆盖率和性能指标收集**：
   - 覆盖率收集
   - KPI实施
   - 检测器实施


## 定制场景

在本节中，您将学习原则和语言构造，您可以利用它们构建或修改OSC2场景。你将重用上一个实验提出的相同场景，并根据新需求进行调整。

### 添加约束条件

本次实验探讨了三种基本约束条件——另一辆车的颜色和类型，以及切入发生的一侧。根据你想要设定哪些限制条件，你可以将约束条件添加到场景文件（在这种情况下是`$FTX_WORKSHOP/scenarios/cut_in_l01.osc`）或测试文件（在这种情况下是`$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`）。为了保持代码可重用性，在测试文件中添加所需的约束条件是最佳做法，如你将在本次实验中看到。

!!! 示例 "亲身操作"
 打开文件 `$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc` ，其中定义了测试内容。

 编辑 `ts_l01_intro.osc` 文件中所定义的场景，并添加上述提到的约束条件：

```osc linenums="13"
extend top.main:
    do cil : sut.cut_in_l01() with:
        keep (it.car1.category == van)
        keep (it.car1.color in [white, blue])
        keep (it.side == right)
```

重新启动Foretify，将其设置为一个新的工作文件夹并执行10次运行：

```bash
foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --crun 10 --batch
```

在场景运行时，注意新引入的约束条件是如何被执行的。特别是你可以观察到切入动作始终是从右侧进行的，执行切入动作的车辆是白色或蓝色的货车。

### 驾驶动作和场景操作符

在开发基本或复杂的OSC2场景时，有三个重要的要点需要记住：

- 驾驶动作
- 并行操作符
- 串行操作符

通过在重用现有的_cut_in_场景并编写_cut_in_and_slow_场景时实践使用这些要点，你将了解到它们的重要性。

#### 驾驶动作

任何车辆角色都提供了驾驶动作，用于控制车辆的移动。单独使用驾驶动作会使车辆沿着随机路径行驶，该路径会被轻微约束在车道内。在本实验中，你将添加修饰符来约束路径和速度。

对于_sut_角色和另一个类型为_vehicle_（_car1_）的角色，你可以按以下方式调用驾驶动作：

```osc linenums="1"
sut.car.drive()

car1.drive()
```

要为驾驶动作添加修饰符（例如，保持特定的速度或特定的车道），请使用以下语法：

```osc linenums="1"
sut.car.drive() with:
    <修饰符>

car1.drive() with:
    <修饰符>
```


#### 并行运算符

并行运算符提供了执行两个或多个子块的灵活性。您还可以控制不同子块之间执行的重叠。在这里您将看到的示例中，所有在 _parallel_ 运算符下的子块都以相同的持续时间执行，因为 _overlap_ 被设置为 _equal_。在高级实验室中，您将看到如何调整重叠。

考虑下面的基本示例，在这个示例中，sut.car 和另一辆车只是在行驶。您可以在这里找到这个示例：`$FTX_WORKSHOP/scenarios/parallel_example.osc`：

&lt;p align="center">
  &lt;a href="images/workshop_l02_parallel.png" target="_blank">
    &lt;img src="images/workshop_l02_parallel.png">
  &lt;/a>
&lt;/p>

在包含并行运算符的这种情况下，两个角色正在做以下事情：

- SUT 在没有关于其速度或位置的明确约束的情况下行驶。
- car1 在没有关于其速度的明确约束但在其位置上有一个相对约束：在场景开始时至少比 SUT 前进 10 米。

!!! 示例 "实践时间"
   

接下来，您将使用_serial_运算符扩展示例场景。正如名称所示，位于_serial_运算符下的两个或多个块将依次执行。这引入了场景的隐含约束，保证了执行的物理连续性。

您也可以在`$FTX_WORKSHOP/scenarios/serial_example.osc`下找到此示例：

<p align="center">
  <a href="images/workshop_l02_serial.png" target="_blank">
    <img src="images/workshop_l02_serial.png">
  </a>
</p>

在此包含串行和并行运算符的场景中，两个参与者正在执行由两个阶段组成的场景：

- **phase1**：这与先前的示例类似，但这次car1被限制在与SUT相同的车道上行驶，并在该阶段期间将其速度降低最多20公里/小时。此外，持续时间现在为2到6秒。
- **phase2**：在此阶段中，仅限制car1将其速度提高最多20公里/小时。

**phase1**和**phase2**是在OSC2中用于轻松引用场景不同部分的标签。您将在以后的实验中了解更多关于标签的内容。

使用串行运算符始终会引入一些隐含约束，其中一些列在下面的示例中：

- _phase2_从_phase1_结束的同时开始。
- _car1_和_sut_在_phase2_开始时的速度等于它们在_phase1_结束时的速度。
- _car1_和_sut_在_phase2_开始时的位置等于它们在_phase1_结束时的位置。

隐含约束的解决由Foretify自动处理，确保在整个场景中保持物理连续性，而无需您进行管理。

!!! Example "操作时间"
    您现在可以使用以下命令执行场景几次：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02_serial.osc \
    --batch --seed 3 --crun 3 --work_dir $FTX_FM_WORKDIR/l02_second_iter/work1
    ```

    观察场景的展开方式。请记住，在本次研讨会中，默认情况下将SUT设置为红色。您应该观察到绿色汽车始终在场景开始时领先于红色汽车。然后它会减速并再次加速，与特定的种子无关。

### 场景重用

在本实验室中，您将开始编写自己的OSC2代码。您将实现的场景是“插入和减速”。一辆汽车在SUT前面插入，然后开始减速。

OSC2的主要优点之一是基础场景的可重用性，作为更复杂场景的构建块，这在创建新的更复杂场景时节省资源。

您正在实现的场景由两个阶段组成：插入阶段和减速阶段。您将根据之前学习的内容开始实现这两个阶段。

#### 创建_cut_in_and_slow_场景和ODD拓扑规范

!!! Example "操作时间"
    首先使用以下命令打开`cut_in_and_slow_l02.osc`：

    ```bash
    code $FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02.osc
    ```

通过查看代码，您可以看到第一行的导入语句导入了在实验室1中使用的相同场景（`cut_in_l01.osc`）。

为了重用插入场景，您首先需要创建_cut_in_and_slow_场景的结构。

```Chinese
!!! 示例 "实操时间"
现在你可以尝试编辑`cut_in_and_slow_l02.osc`文件，添加所需的代码来执行切入和减速操作。
由于我们还没有介绍如何实施减速，请专注于切入和行驶场景，车辆在切入后继续移动。请记住，OSC2是基于缩进的语言，Foretify工具需要使用空格（而不是制表符）作为缩进方法。

!!! 小贴士
代码应首先执行`cut_in_l01.osc`中定义的切入操作，然后进行减速。你可以使用稍后将解释的修饰符来实现减速。目前先尝试构建操作的不同阶段。

??? 解决方案 "解决方案 - 点击此处"
 ```osc linenums="1"
 方案 sut.cut_in_and_slow:
  车辆1: 交通工具
  方向: av_side

  连续执行():
   切入: cut_in_l01(车辆1: 车辆1, 方向: 方向)
   减速: 并行(持续时间:[2..5]秒, 重叠:相等):
    sut.car.drive()
    车辆1.drive()
 ```

以上解决方案代码负责以下内容：
```

- 场景_sut.cut_in_and_slow_已创建。
- 两个对象_car1_和_side_已实例化。它们分别是预定义类型_vehicle_和_av_side_。
- 在_do serial()_语句之后，定义了两个按顺序发生的阶段：
  - 标记为_cut_in_的第一个阶段：该阶段简单地调用在第一行导入的场景_cut_in_l01_。在这里，为_cut_in_and_slow _实例化的对象_car1 和 _side__作为参数传递给_cut_in_l01__。请注意我们如何将先前定义的场景称为更复杂场景的“构建块”。这是一个非常强大的功能。
  - 标记为_slow 的第二个阶段：这里使用并行操作符，并将持续时间设置在2到5秒之间。

!!! 注意
 目前在第二阶段，SUT和car1都只是驾驶。稍后您将添加额外约束以确保car1减速。

!!! 示例 “动手时间”
 保存后，请运行刚刚创建的内容几次执行以下命令：

 ```bash
 foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
 --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --seed 5 \
 --batch --crun 2
 ```

 观察场景。它应该类似于原始变道情景，并包括两位演员仅仅是在驾驶中的额外部分。

!!! 信息
 能够重用、组合和混合现有情景大大地减少了编写新情景所需的工作量。每个情境成为一个基本单元，您可以与其他现有单元组装起来创建更加复杂且融合了施加约束条件新情境。

可变性场景的结果是其基本元素的可变性产物。这个特性提高了搜索新漏洞的效率，因为所有可能的组合都可以考虑进去。

到目前为止，你所实现的只是一个“插入并驾驶”的场景。现在你将学习如何添加到“慢速”部分的修饰符，以使“car1”减速。

#### “drive()”运动场景的修饰符

接下来，你将进一步塑造场景，并在“慢速”阶段使“car1”减速。为此，你需要在“drive()”运动场景上引入一些额外的修饰符。这些修饰符将作用于“car1”和“sut”的速度和车道。

这并不意味着OSC2允许的唯一修饰符是演员的速度或车道。还有许多其他修饰符可以用于限制，例如，车辆相对于另一个演员的位置或其加速度，就像串行和并行运算符的基本示例一样。

请注意，OSC2中编写的每个阶段都带有预定义的“start”和“end”事件。这就是为什么在定义新的修饰符时可以使用这些事件。

##### “change_speed”修饰符

为了实现插入并减速的场景，我们需要使用“change_speed”修饰符。“change_speed”通过给定的量（可以是正值或负值（分别加速或减速））来改变演员的速度。当你想要控制相对于当前段开始时的车辆速度的速度增加或减少时，这非常有用。

!!! 例子 "实践时间"
    再次使用代码编辑器打开`$FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02.osc`，并按以下方式修改场景：

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

什么被添加了？

该示例在第14行引入了一个_change_speed_修饰符，它将_car1_的速度限制为在名为_slow_的阶段开始和结束之间至少减少15 kph。这种减速完成了切入和减速的情景。

!!! 注意
    你注意到了吗？我们再次使用了_run_mode: best_effort_，但这次作为修饰符的内联参数。我们这样做是因为该场景将被计划为模拟器不可知，这意味着模拟器可能以意外的方式限制车辆的减速。在这种情况下，我们不希望由于不完整的场景而导致运行失败。

!!! 示例 "动手时间"
    保存后，使用以下命令运行几个执行：

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --batch --crun 5
    ```

你可以观察情况是如何发展的，并查看日志消息。它们是否符合你的预期？

检查最后一条日志消息。由于刚刚添加的限制条件，car1的速度应至少比起始速度慢15公里/小时。

通过change_speed修饰符，您只需指定速度增加或减少的数量。

这是一个强大的功能，可以提高给定情景的抽象级别。

### 添加一个新阶段

在_do serial_中添加另一个名为_speed_up_（建议持续时间为3到5秒）的阶段，在此阶段中，_car1_将保持车道并加速至比之前快至少15公里/小时的速度。与降速一样，速度增加有时会受到模拟器和道路曲率等外部因素的限制。在这种情况下，我们不希望情景失败，而是将接受产生的速度变化。我们可以通过将_change_speed_修饰符的_run_mode_设置为_best_effort_来定义这一点。

包括一些日志消息以改善调试。

保存后，使用以下命令运行另一个测试：

```bash
foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
--work_dir $FTX_FM_WORK

执行序列（）：
    记录（“@cut_in.start car1 speed = $（car1.state.speed）”）
    cut_in：cut_in_l01（car1：car1，side：side）
    记录（“@cut_in.end car1 speed = $（car1.state.speed）”）
    记录（“@slow.start car1 speed = $（car1.state.speed）”）
    slow：parallel（duration：[2..5]秒，overlap：equal）：
        sut.car.drive（）with：
            keep_lane（）
        car1.drive（）with：
            change_speed（- [15..]kph）
            keep_lane（）
    记录（“@slow.end car1 speed = $（car1.state.speed）”）
    记录（“@speed_up.start car1 speed = $（car1.state.speed）”）
    speed_up：parallel（duration：[3..5]秒，overlap：equal）：
        sut.car.drive（）with：
            keep_lane（）
        car1.drive（）with：
            cs：change_speed（[15..]kph，run_mode：best_effort）
            keep_lane（）
    记录（“@speed_up.end car1 speed = $（car1.state.speed）”）


### 如何在Foretify中检查运行情况

在开发或审查场景时，您需要能够检查运行情况。Foretify提供了一个集成的场景可视化器，您在之前的实验中已经探索过。现在，您将探索一些更高级的功能，以帮助您进一步。

有许多方法可以检查运行情况。接下来，您将了解一些广泛使用的方法：

- 使用_log（）_消息和Foretify日志文件。
- 与可视化器一起使用跟踪。


#### 使用Foretify _log文件_检查运行情况

您可以通过查看Foretify _log文件_来检查运行情况。

```markdown
!!! Example "实际操作时间"
  使用代码编辑器打开日志文件，并搜索在前几节中介绍过的消息。最近几次运行的日志可以在$FTX_FM_WORKDIR/l02_second_iter/workdir2/runs/```timestamp```/run.log中找到，或者在$FTX_FM_WORKDIR/l02_second_iter/workdir2/logs/```timestamp```中找到。

#### 使用跟踪和可视化工具检查运行

跟踪功能可以让你绘制变量随时间的图表。当与Foretify Visualizer工具结合使用时，这个功能非常有用。你可以在OSC2场景中自定义跟踪，但是在本练习范围内，你将使用为演员预定义的跟踪。

!!! Example "实际操作时间"
 关闭已经打开的Foretify会话后，再次以GUI模式打开Foretify，并加载上次创建的场景，请使用以下命令：

 ```bash
 foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
 --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --gui
 ```

 通过点击右上角的**Terminal**按钮或按下_t_键来启动Foretify终端。

 默认情况下，多个跟踪是激活状态。你可以通过输入```trace```命令来查看激活状态下的跟踪。

 其他有用的跟踪命令包括：

 - ```trace --help```: 查看所有可用的跟踪命令选项
 - ```trace --off```: 关闭所有跟踪
 - ```trace *.*```: 跟踪给定通配符层级下的所有项目

 <p align="center">
  <a href="images/l02_traces.png" target="_blank">
  <img src="images/l02_traces.png">
  </a>
 </p>

 禁用所有跟踪并只激活那些绘制车辆速度图表的轨迹，请使用以下命令：

 ```bash
 trace --off *.*
 trace *.speed
 ```


这应该显示正在跟踪的两辆车的速度，如下所示：

<p>
  <a href="images/l02_trace_speed.png" target="_blank">
    <img src="images/l02_trace_speed.png">
  </a>
</p>

运行测试（可以在Foretify终端中使用“run”命令或GUI按钮）。完成测试后，您可以切换到下图所示的**Traces**选项卡：

<p>
  <a href="images/l02_show_traces.png" target="_blank">
    <img src="images/l02_show_traces.png">
  </a>
</p>

现在，您可以通过单击和拖动时间导航器（用橙色圈圈标出）来选择跟踪上的不同时间戳。这将使可视化器移动到相应的时间戳，以便您可以看到在该时间戳模拟中发生了什么。

<p>
  <a href="images/l02_trace_select_time.png" target="_blank">
    <img src="images/l02_trace_select_time.png">
  </a>
</p>

#### 使用跟踪间隔检查运行

现在，您有一个具有更多连续发生的阶段的场景，您将欣赏到Foretify GUI中显示的间隔的好处，这些间隔告诉您每个阶段何时开始和结束，如下所示：

<p align="center">
  <a href="images/l02_intervals.png" target="_blank">
    <img src="images/l02_intervals.png">
  </a>
</p>

## 覆盖率收集

度量驱动的验证和验证（V＆V）是一种定期使用度量衡量验证结果的过程，以便可以根据收集的度量衡量做出决策。这带来了许多好处。度量驱动的V＆V：

- 消除了V&V状态评估中的人为错误，使V&V状态客观化。
- 允许工程师调整决策并将精力集中在客观度量上。
- 通过从一次迭代到下一次的即时反馈加速了V&V过程。
- 提供了决策层次和细节层次的透明度，从工程师到CEO，从SUT速度到整体指标得分。
- 使V&V过程对SUT的行为不敏感，指标反映了真实发生的事情，而不是工程师预期发生的事情。
- 对规模不敏感，V&V随着工作量的增加自然扩展。
- 允许评估SUT的质量，即可以基于客观统计数据做出GO/NO-GO决策。

正如您可能已经注意到的那样，基于指标的验证和验证与基于测试的验证和验证不同，其中进展是通过通过测试的数量来衡量的。可以说，由于上述原因，与基于测试的V&V相比，使用基于指标的V&V带来了许多优势。

Foretellix解决方案通过覆盖度指标来补充传统的KPI，这是本节的主题。这意味着您仍然可以使用传统的方法来评估质量，同时添加新的指标以增加验证目标定义的精度。

### 什么是覆盖率？

工程师定义验证指标以可衡量的方式描述验证目标。覆盖率定义指定必须执行哪些属性/参数值。例如，假设为了证明SUT在切入场景中的行为正确，您需要确保在给定时间点内执行以下两个参数的给定范围：

| 覆盖项目   | 范围      | 桶   | 单位 | 何时         |
|--------------|-------|--------|----|-------------|
| sut_speed    | [0..200] 公里/小时 | 每10 | 公里/小时 | 切入结束    |
| car1_distance| [0..50] 米       | 每5  | 米       | 切入结束    |

为什么我们选择了范围[0..200]和[0..50]？答案是这些范围是根据捕捉系统运行方式以及其优势和劣势的工程规范定义的。举例来说，如果一个覆盖项目被建模为整数，它将有2^32个值的值空间。并非所有这些值都是从验证角度来看有效或有用的。因此，上面的表格指定了所谓的“有效值空间”，将样本收集限制在仅相关值上。限制空间确保覆盖有趣值的范围，而不会浪费验证资源在无关或无效空间上，比如NPC以超音速速度驾驶。

将覆盖项目的值空间限制为指定的有效值是V&V工程师的决定。即使在合法值范围内，您也可以细化并创建称为桶的较小范围（在本例中分别为每10公里/小时和每5米）。桶是在假设对于桶中所有值的行为和质量影响相似的情况下创建的。_when_参数指定采样点&mdash;参数的值何时保存到覆盖数据库中。

OSC2

```osc linenums="1"
# 这里我们扩展了 cut_in 场景
extend sut.cut_in:           
 # 在这里我们决定何时进行采样以及采集什么信号
 var sut_speed := sample(sut.car.state.speed, @cut_in.end)     
 cover(name: sut_speed,
  # 对覆盖项的描述
  text : "cut_in 结束时sut的绝对速度 (单位：km/h)", 
  # 覆盖项代表的物理单位   
  unit : kph,
  # 相关数值范围，范围之外的所有内容都将被忽略          
  range : [0..200],
  # 桶的大小（例如 [10..20], [20..30] 等）         
  every : 10)           
```
!!! tip
 
 查看[cover()和record()参数](../osc_lang/osclang_behavior_monitoring.md#cover-and-record-parameters)获取`cover`语法和定义。

现在假设上述两个覆盖项已经实现，并且进行了77次模拟。结果如下：

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

你可以看到，并非所有的桶都被命中，这意味着覆盖项并没有完全覆盖。但是覆盖了多少？覆盖项的等级给出了填充水平。覆盖等级是所有测试中遇到的情况的多维表示。

覆盖项的等级是一个百分比：覆盖了多少个项目值占总数的百分比。在这种情况下，覆盖率为：

- sut-npc距离覆盖率-23％：13个桶中有3个被命中（即，采样值在桶范围内）
- sut速度覆盖率-38％：13个桶中有5个被命中

这意味着您必须继续运行模拟，直到您获得100％的覆盖等级。如果某些桶未被命中，则可以决定更改场景的约束条件以针对不同的值。

!!! Info
    覆盖驱动验证（CDV）是一种被证明成功的方法，被用作各种领域中高度复杂的测试设备的行业标准，特别是半导体行业。这种方法有助于在产品部署到市场之前不断发现错误，避免极其昂贵的召回。

    Foretellix专家在半导体行业中应用了20多年的CDV经验，为自主系统的安全提供了V＆V解决方案。由于Foretellix工具使CDV方法适用于自动驾驶车辆验证，因此V＆V工程师可以应用一种经过验证的方法来有效地监视验证过程的状态。

    覆盖驱动验证代表了安全驱动验证方法的重要组成部分。

### 如何定义覆盖项

```markdown
当一个覆盖项不需要抽样（例如，它是场景的输入）时，可以定义如下：

```osc linenums="1"
 cover(name: aside, expression:side)
```

这将创建一个覆盖项，该覆盖项在 _cut in_ 段开始时使用场景的 _side_ 字段。只要击中 _left_ 和 _right_ 的两个值，该覆盖项的等级就会达到 100%（即完全覆盖）。如果仅有一个值被击中，则该项的等级为 50%。

当一个覆盖项需要抽样时，可以定义如下：

```osc linenums="1"
 var rel_speed := sample(car1.state.speed - sut.car.state.speed, @cut_in.end)
 cover(name: rel_speed,
  text : "相对速度 cut_in 结束 (以公里/小时计)",
  unit : kph,
  range : [-50..50],
  every : 10)
```

如果参数是整数，则其覆盖空间很大（即2^32-1个值）。通常，并非所有整数值都相关。OSC2 允许您像这个示例一样指定整数的合法范围（使用 _range_ 和 _every_）。

_rel_speed_ 在事件 _cut_in.end_ 上进行抽样。此时的值将匹配包含它的桶并保存下来。一旦桶中有一个样本，该桶就会达到100% 的等级。根据定义，只要至少有一个样本与桶的区间匹配，则该桶就具有100% 的等级。

!!! 示例 "实践时间"
 查看在 `$FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02_cov.osc` 中所定义的覆盖项。你能认出并解释这些覆盖项代表什么吗？请注意，在此处使用了函数_map.abs_distance_between_positions_ 提供了两个位置之间的绝对距离。
```

由于您已经进行了几次运行，您可以使用以下命令将它们收集并上传到Foretify Manager：

```bash
upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
--runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2
```

在使用命令`fmanager`打开Foretify Manager后，为测试运行创建一个名为_cut_in_and_slow_的新工作区，就像您在实验室1中学到的那样。最后，您应该看到类似于以下图像的内容（请注意，百分比可能会更改）：

<p align="center">
  <a href="images/l02_vgarde_no_cross.png" target="_blank">
    <img src="images/l02_vgarde_no_cross.png">
  </a>
</p>

检查工作区中的覆盖度指标。您应该有四个项目：aside、sut_speed、rel_speed和car1_distance。这四个覆盖项的覆盖度分数是多少？整体VGrade是多少？

### 如何定义覆盖交叉

交叉是两个或多个覆盖项的乘积，突出了这些项之间的关系。交叉覆盖项的桶是各个项桶的笛卡尔积。考虑上面的例子，SUT速度和SUT到NPC距离的交叉看起来像这样：

<p align="center">
  <a href="images/workshop_lab_5_cross_samples.png" target="_blank">
    <img src="images/workshop_lab_5_cross_samples.png">
  </a>
</p>

样本落在绿色单元格内。与覆盖项类似，覆盖交叉的分数是一个百分比：169个桶中有10个被覆盖，即5.9%。

将以下内容翻译成中文：

The cross's physical/legal value space is the Cartesian product of the physical/legal value spaces of the component items. Given that, engineers have to be careful when designing crosses since a cross space can grow rapidly to volumes that are hard to fill.

!!! Example "Hands-on Time"
  Add the following cross coverage items to _$FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02_cov.osc_:

 ```osc linenums="1"
  cover(sut_speed_x_rel_speed, items:[sut_speed, rel_speed])
  cover(sut_speed_x_car1_distance, items:[sut_speed, car1_distance])
 ```

 After saving, run a few executions with the following command:

 ```bash
 foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
 --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir3 \
 --batch --crun 5
 ```

 You can now collect and upload the runs to Foretify Manager using the following command:

 ```bash
 upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
 --runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir3
 ```

 After opening Foretify Manager using the command, `fmanager`, create a new workspace called _cut_in_and_slow2_ using the new runs that you imported. Can you recognize the new coverage crosses? Inspect the bins (buckets) of the crosses and note how they correlate to the individual items that create the cross.


## KPI implementation

Key Performance Indicators (KPIs) can be as simple as raw metrics measured to see how well[sic] well[we need recheck here]	the AV performed. There can be safety-related KPIs such as min-Time-To-Collision (min-TTC), measured in seconds, comfort-related KPIs such as max-deceleration, measured in meter/seconds[sic], and so on.


实验1介绍了KPI的概念，并且您已经练习了一个例子。现在，为了更好地理解如何实施KPI，您将使用_collect_结构编写一个新的KPI。该KPI将在每个时间步骤捕获_碰撞时间_并记录其在整个场景中的最小值。

我们还希望在场景结束时记录最小值。

为了做到这一点，您需要使用以下语法进行collect_time()和record()调用：

    - collect_time
        - exp：在每个时间步骤评估的表达式
        - measure：应用于收集数据的计算步骤（例如，最小值、最大值、平均值）

    - record
        - expression：要记录的变量或公式的结果
        - unit：要记录的值的单位
        - event：在模拟时间的哪个点记录该值
        - text：描述该值含义的消息

collect()的目的是定义性能指标或其他数据收集。

!!! 提示

    - 您可以将新的代码片段添加到现有文件中：
    ```bash
    code $FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02_cov.osc 

    ```
    - 要定义变量，您可以查看之前声明变量的示例。
    - 访问TTC的方法称为_get_ttc()_。
    - 我们想要使用的测量单位是秒（s）。

??? 解决方案 "解决方案-点击这里"

    ```osc linenums="1"
    extend sut.cut_in_and_slow:

        # 在每个时间步骤采样TTC KPI，获取其最小值，并检查是否低于定义的阈值
        min_ttc: collect_time(exp: sut.car.get_ttc(),
            measure: min)

        # 定义一个可以被record使用的事件
        event sample_ttc is @end
    ```

```markdown
# 在场景结束时记录聚合构造的结果，以获取本次运行中的总最小TTC

record(min_ttc_kpi,
  表达式: min_ttc.computed_result,
  单位: 秒,
  事件: 样本TTC,
  文本: "最小碰撞时间（TTC）"
)
```

!!! 示例 "实操时间"
保存后，使用更新后的代码运行几个执行：

```bash
foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
 --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir4 \
 --batch --crun 3
```

现在可以收集这些运行并使用以下命令上传到Foretify Manager：

```bash
upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
 --runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir4
```
现在可以在Foretify Manager中查看KPI结果。

## 检查器实施

### 理解和解决问题
其中一个主要的验证挑战是以最有效的方式确定SUT是否表现正确。例如，您想知道SUT是否：

- 没有达到法定高速公路速度最低要求。
- 没有与行人保持适当距离。
- 开车离开了道路。

检查器是定义SUT应该或不应该如何行为的OSC2实体。最简单的检查器会定义如果SUT未能满足布尔条件，则返回失败响应。

### 如何实施检查器

在处理场景时，您可能需要定义检查器来捕获与SUT特定的问题行为或条件有关的问题。现在，您将尝试一个类似于实验室1中示例的新示例。对于此示例，您将调整先前练习中新编写的KPI：min_ttc。检查器的目的是确保TTC在开始缓慢阶段时不超过3秒钟。

编写检查器的步骤如下：

1. 创建名为_cut_in_and_slow_l02_checker.osc_ 的新文件。
2. 创建一个名为safe_ttc 的新issue_kind。
3. 为安全TTC阈值类型定义一个新变量。
4. 将安全TTC阈值设置为3秒钟。
5. 将_collect_time_ 结构从_cut_in_and_slow_l02_cov.osc_ 转移到_cut_in_and_slow_l02_checker.osc_ 中。
6. 将以下参数添加到_collect_time_ 结构中：

   - bad_is: 比较结果与阈值（这里是：low）的方向
   - threshold: 与之比较的阈值（这里是：我们新定义的阈值）
   - first_failure_kind: 如果违反了阈值，则应引发哪种警报（这里是：safe_ttc）
   - first_failure_severity: 应引发哪种警报严重程度（这里是：error）

7. 从_ts_l02.osc_ 文件导入_new file _cut_in_and_slow_l02_checker.osc_

!!! tip

来自实验室1中简短检查器语法提示(用<>标记单词组需要进行调整)：
```osc
extend issue_kind: [<checker_name>]

extend sut.<scenario_name>:

var <checker_threshold>: time 
set <checker_threshold> = 3s 
```

??? Solution "Solution - Click Here"

```osc linenums="1"
extend issue_kind: [safe_ttc]

```yaml
extend sut.cut_in_and_slow:

  var ttc_kpi_threshold: time # 安全TTC阈值类型
  set ttc_kpi_threshold = 3s # 安全TTC阈值数值

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
    event:sample_ttc, 
    text:"最小碰撞时间（TTC）")

```

## 覆盖率实践

这个实践的目的是：

- 添加一个新的交叉覆盖项。
- 运行测试。
- 使用Foretify Manager检查结果。

!!! Example "Hands-on Time"
添加一个新的交叉覆盖项，由切入发生时的侧面和切入结束时相对速度定义。

使用Foretify运行额外的测试（可以使用crun选项自动运行多次）。最好定义一个新的工作目录，例如$FTX_WORKSHOP/l02_second_iter/workdir4。

在Foretify Manager中查看结果。你能做些什么来增加覆盖率呢？

## 下一步骤

这样就完成了第二次安全驱动验证迭代。在这个实验中，你：

- 熟悉了场景复用。
- 使用了诸如 _serial_ 和 _parallel_ 的场景操作符。
- 利用追踪和可视化工具调试了场景。
- 熟悉了覆盖项目和KPIs的概念和定义。

接下来，在第三节实验中，你将学会如何增加测试数量。

> 本文由ChatGPT翻译，如有任何遗漏，请[**反馈**](https://github.com/linyuxuanlin/Wiki_MkDocs/issues/new)。