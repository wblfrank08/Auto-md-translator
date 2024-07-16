---
chapnum: 2
---
# Lab 2: Introduction to Scenario Reuse

## Learning goals

This lab is the second iteration of the safety-driven verification cycle. In this lab, you will learn how to tailor the OSC2 verification and validation (V&V) assets:

- **Tailoring scenarios**:
    - Adding new constraints
    - Using the drive movement action
    - Using the parallel and serial operators to expand scenarios
    - Reusing the existing _cut_in_ scenario

- **Inspecting runs in Foretify**:
    - Inspecting a run with the Foretify log file
    - Inspecting a run with traces and Visualizer
    - Inspecting the Intervals

- **Coverage and performance metrics collection**:
    - Coverage collection
    - KPI implementation
    - Checkers implementaion

## Tailoring scenarios

In this section, you will learn principles and language constructs that you can leverage to construct or modify OSC2 scenarios. You will reuse the same scenario that was proposed in the previous lab and adapt it for new requirements.

### Adding constraints

This lab explores three basic constraints&mdash;the color and the type of the other car, and the side from where the cut-in happens. Depending on what you would like to constrain, you can either add the constraint in the scenario file (in this case, `$FTX_WORKSHOP/scenarios/cut_in_l01.osc`) or in the test file (in this case, `$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`). To keep the code reusable, it is best practice to add the desired constraints in the test file, as you will see  in this lab.

!!! Example "Hands-on Time"
    Open the file `$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`, where the test is defined.

    Edit the scenario in the `ts_l01_intro.osc` file and add the constraints mentioned above:

    ```osc linenums="13"
    extend top.main:
        do cil : sut.cut_in_l01() with:
            keep (it.car1.category == van)
            keep (it.car1.color in [white, blue])
            keep (it.side == right)
    ```

    Restart Foretify, set it to a new work folder and perform 10 runs:

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
        --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --crun 10 --batch
    ```

    While the scenario is running, note how the new constraints introduced are being enforced. In particular you can observe that the cut_in is always from the right, the vehicle performing the cut_in is a white or blue van.


### The drive movement action and scenario operators  

When developing basic or complex OSC2 scenarios, there are three important pillars to keep in mind:

- The drive movement action
- The parallel operator
- The serial operator

You will learn the importance of these pillars by seeing them in practice when reusing the existing _cut_in_ scenario and writing the _cut_in_and_slow_ scenario.


#### The _drive_ movement action

Any vehicle actor provides the _drive_ movement action which controls the car movement. By itself, _drive_ causes the car to just drive along a random path, which is softly constrained to be in a lane. Later in this lab you will add modifiers to constrain the path and the speed.

For the _sut_ actor and another actor of type _vehicle_ (_car1_), you can call the _drive_ movement action as follows:

```osc linenums="1"
sut.car.drive()

car1.drive()
```

To add modifiers to the drive movement (for example, to keep a certain speed or a certain lane), you use the following:

```osc linenums="1"
sut.car.drive() with:
    <modifiers>

car1.drive() with:
    <modifiers>
```


#### The _parallel_ operator

The parallel operator provides the flexibility to execute two or more sub-blocks simultaneously. You can also control the overlap between the execution of the different sub-blocks. In the example you'll see here, all of the sub-blocks under a _parallel_ operator are executed with the same duration because the _overlap_ is set to _equal_. In the advanced labs, you will see how you can also tune the overlap.

Consider the basic example below, where the sut.car and an additional car are just driving. You can find this example  here: `$FTX_WORKSHOP/scenarios/parallel_example.osc`:

<p align="center">
  <a href="images/workshop_l02_parallel.png" target="_blank">
    <img src="images/workshop_l02_parallel.png">
  </a>
</p>

During this scenario, which contains the parallel operator, the two actors are doing the following:

- The SUT is driving without any explicit constraints on its speed or position.
- The car1 is driving without any explicit constraints on its speed but a relative constraint on its position: at least 10 Meters ahead of the SUT at the beginning of the scenario.

!!! Example "Hands-on Time"
    You can now execute the scenario a few times with the following command:

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02_parallel.osc \
    --batch --seed 5 --crun 3 --work_dir $FTX_FM_WORKDIR/l02_second_iter/work1
    ```

    Observe how the scenario unfolds. Note that the SUT is set by default to be red in this workshop. You should observe that the green car is always ahead of the red car at the beginning of the scenario, independent of the specific seed.

#### The _serial_ operator

Next, you will expand the example scenario using the _serial_ operator. As the name suggests, two or more blocks located under a _serial_ operator are executed one after the other. This introduces implicit constraints on the scenario, which guarantee physical continuity of the execution.

You can find this example also under `$FTX_WORKSHOP/scenarios/serial_example.osc`:

<p align="center">
  <a href="images/workshop_l02_serial.png" target="_blank">
    <img src="images/workshop_l02_serial.png">
  </a>
</p>

During this scenario, which contains both the serial and parallel operator, the two actors are performing a scenario that consists of two phases:

- **phase1**: This is similar to the previous example, but this time car1 is constrained to drive in the same lane as the SUT and to reduce its speed by up to 20 km/h during the phase. In addition, the duration now is 2 to 6 seconds.
- **phase2**: In this phase, car1 is constrained to increase its speed by up to 20 km/h only.

**phase1** and **phase2** are labels that are used in OSC2 to easily reference different parts of a scenario.
You will learn more about labels in later labs.

Using the serial operator always introduces some implicit constraints, some of which are listed below for this example: 

- _phase2_ starts at the same point in time that _phase1_ ends.
- _car1_'s and _sut_'s speeds at the start of _phase2_ are equal to their respective speeds at the end of _phase1_.
- _car1_'s and _sut_'s positions at the start of _phase2_ are equal to their respective positions at the end of _phase1_.

The implicit constraint solving is handled automatically by Foretify, ensuring that physical continuity is maintained throughout the scenario without you having to manage that.

!!! Example "Hands-on Time"
    You can now execute the scenario a few times with the following command:

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02_serial.osc \
    --batch --seed 3 --crun 3 --work_dir $FTX_FM_WORKDIR/l02_second_iter/work1
    ```

    Observe how the scenario unfolds. Keep in mind that the SUT is set by default to be red in this workshop. You should observe that the green car is always ahead of the red car at the beginning of the scenario. Then it slows down and accelerates again, independent of the specific seed.


### Scenario reuse

In this lab, you will start writing your own OSC2 code. The scenario that you will implement is "cut-in and slow". A car cuts-in in front of the SUT and then starts to slow down.

One of the main benefits of OSC2 is the reusability of base scenarios as building blocks for more complex ones, which saves resources when creating new, more complex, scenarios.

The scenario you are implementing consists of two stages: a cut-in stage and a slow down stage. You will start by implementing the two stages according to what you learned in the previous sections.

#### Creation of the _cut_in_and_slow_ scenario and ODD topology specification

!!! Example "Hands-on Time"
    Start by opening `cut_in_and_slow_l02.osc` with a code editor using the following command:

    ```bash
    code $FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02.osc
    ```

By looking at the code, you can see that the import statement in the first line imports the same scenario that was used  in Lab 1 (`cut_in_l01.osc`).

In order to reuse the cut-in scenario, you first need to create the structure of what will be the _cut_in_and_slow_ scenario.

!!! Example "Hands-on Time"
    You can now try to add the code required to perform the cut-in and slow by editing the `cut_in_and_slow_l02.osc` file.
    As we have not covered how to implement the slowdown yet, focus on a cut-in and drive scenario where the cars continue their movement after the cut-in. Keep in mind that OSC2 is an indentation-based language and that the Foretify tool requires spaces (and not tabs) as the indentation method.    

!!! tip
    The code should first perform the cut-in defined in `cut_in_l01.osc` followed by a slowdown. You can achieve the slowdown using modifiers which will be explained later. For now try to structure the different phases of the maneuver.
    
??? Solution "Solution - Click Here"    
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

The solution code above is responsible for the following:

- The scenario _sut.cut_in_and_slow_ is created.
- The two objects _car1_ and _side_ are instantiated. They are, respectively, of the predefined type _vehicle_ and _av_side_.
- After the _do serial()_ statement there are two phases defined, which will happen serially:
    - A first phase labeled with _cut_in_: This phase simply calls the scenario _cut_in_l01_ that is imported in the first line. Here the objects _car1_ and _side_, instantiated for _cut_in_and_slow_, are passed as arguments to _cut_in_l01_. Note how we call a previously defined scenario as a "building block" to a more complex one. This is a very powerful feature.
    - A second phase labeled with _slow_: Here the parallel operator is used and the duration is set to be in the range 2..5 seconds.

!!! Note
    For now in the second phase, both the SUT and car1 are just driving. You will add additional constraints later to make sure that car1 slows down.

!!! Example "Hands-on Time"
    After saving, run a few executions of what you just created, with the following command:

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --seed 5 \
    --batch --crun 2
    ```

    Observe the scenario. It should be similar to the original cut-in scenario with an additional part where the two actors are just driving.

!!! Info
    Being able to reuse, combine and mix existing scenarios within new scenarios drastically reduces the work required to write a new scenario. Each scenario becomes a basic unit that you can assemble with other existing units to create even more complex scenarios where the imposed constraints are merged.

    The variability of the resulting scenario is a product of the variability of its base elements. This feature boosts the efficiency in searching for new bugs since all the possible combinations can be taken into account.

What you implemented so far is just a _cut_in_and_drive_ scenario. You will now learn about the modifiers needed to add to the _slow_ section in order to make _car1_ slow down.


#### Modifiers of the _drive()_ movement scenario

Next you'll further shape the scenario and make _car1_ slow down during the _slow_ phase. To do that, you will need to introduce some additional modifiers on the _drive()_ movement scenario. The modifiers will act on the speed and lane of both _car1_ and _sut_.

This does not imply that the only modifiers allowed by OSC2 are on the speed or lane of the actor. There are many more modifiers that can be used to constrain, for example, a vehicle's position relative to another actor or its acceleration as in the basic examples of the serial and parallel operator.

Note that each of the phases written in OSC2 comes with a predefined "_start_" and "_end_" event. This is why you are able to use those events when defining new modifiers.

##### The _change_speed_ modifier

In order to implement a cut_in and slow scenario, we will need to use the _change_speed_ modifier. _change_speed_ changes the speed of the actor by the given amount, which can be a positive or negative value (accelerate or decelerate, respectively). This is useful when you want to control the speed increase or decrease relative to the vehicle´s speed at the start of the current segment.

!!! Example "Hands-on Time"
    Open `$FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02.osc` with a code editor again and modify the scenario as follows:

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

What was added?

The example introduces a _change_speed_ modifier in line 14 that constrains the speed of _car1_ to decrease at least by 15 kph between the beginning and the end of the phase called _slow_. This slowdown completes the cut-in and slow scenario.

!!! Note
    Did you notice? We used _run_mode: best_effort_ again, but this time as an inline parameter of the modifier. We did this, because the scenario will be planned simulator agnostic, which means that the simulator might limit the deceleration of the vehicle in an unexpected way. In this case, we don't want the run to fail due to an incomplete scenario.

!!! Example "Hands-on Time"
    After saving, run a few executions with the following command:

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --batch --crun 5
    ```

    You can observe how the scenario unfolds and review the log messages. Are they as you would expect? 
    
    Check the last log message. The speed of car1 should be at least 15 kph less than its speed at the beginning, due to the constraint just added.

!!! Info
    With the change_speed modifier, you can specify just the amount of speed increase or decrease.

    This is a powerful feature that lets you increase the abstraction level of a given scenario.

### Adding a new phase

!!! Example "Hands-on Time"
    Add another stage called _speed_up_ (suggested duration 3 to 5 seconds) to the _do serial_, where _car1_ will keep its lane and accelerate to a speed at least 15 kph faster than before. As with speed decreases, speed increases sometimes are limited by the simulator and external factors such as road curvature. In that case, we don't want the scenario to fail, but will accept the resulting speed change. We can define that by setting the _run_mode_ of the _change_speed_ modifier to _best_effort_.

    Include some log messages to improve debugging.

    After saving, run one more tests with the following command:

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2  --run --gui
    ```
??? Solution "Solution - Click Here"
    ```osc linenums="1"
    import "$FTX_WORKSHOP/scenarios/cut_in_l01.osc"

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

### How to inspect a run in Foretify

When developing or reviewing scenarios, you need to be able to inspect runs. Foretify provides an integrated scenario Visualizer that you explored in the previous lab. Now you will explore some of the more advanced features that can help you further.

There are many ways for you to inspect a run. Next, you will look at some widely used methods:

- Using _log()_ messages and the Foretify log file.
- Using traces together with the Visualizer.


#### Inspecting a run with the Foretify _log file_

You can inspect a run by reviewing the Foretify _log file_.

!!! Example "Hands-on Time"
     Open the log file using a code editor and search the log for the messages that were introduced in previous sections. The logs for the last few runs can be found under $FTX_FM_WORKDIR/l02_second_iter/workdir2/runs/```timestamp```/run.log, or in $FTX_FM_WORKDIR/l02_second_iter/workdir2/logs/```timestamp```

#### Inspecting a run with traces and Visualizer

Tracing enables you to plot variables against time. This feature is extremely useful when combined with the Foretify Visualizer tool. You can define custom traces in the OSC2 scenario itself, but for the scope of this exercise, you will use the predefined traces available for the actors.

!!! Example "Hands-on Time"
    After closing the open Foretify sessions, open Foretify again in GUI mode and load the last scenario that was created, using the following command.

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2 --gui
    ```

    Bring up the Foretify terminal by clicking on the **Terminal** button at the top right corner or by pressing _t_.

    By default, multiple traces are active. You can view the active traces by typing the ```trace``` command.
  
    Additional useful trace commnands are:

    - ```trace --help```: see all trace command options
    - ```trace --off```: deactivate all traces
    - ```trace *.*```: trace all items under given wildcard hierarchy

    <p align="center">
        <a href="images/l02_traces.png" target="_blank">
            <img src="images/l02_traces.png">
        </a>
    </p>

    Disable all the traces and activate just the ones plotting the speed of the vehicles, with the following commands:

    ```bash
    trace --off *.*
    trace *.speed
    ```

    This should display the speed of both vehicles being traced, as below:

    <p>
      <a href="images/l02_trace_speed.png" target="_blank">
        <img src="images/l02_trace_speed.png">
      </a>
    </p>

    Run the test (either with the command "run" in the Foretify terminal or with the GUI button). Once the test is completed, you can switch to the **Traces** tab as shown in the image below:

    <p>
      <a href="images/l02_show_traces.png" target="_blank">
        <img src="images/l02_show_traces.png">
      </a>
    </p>

    Now you can select different timestamps on the trace by clicking and dragging the time navigator(Circled in orange). This will move Visualizer to the corresponding timestamp, so you can see what happened in the simulation at that timestamp.

    <p>
      <a href="images/l02_trace_select_time.png" target="_blank">
        <img src="images/l02_trace_select_time.png">
      </a>
    </p>

#### Inspecting a run with Traces intervals

Now that you have a scenario with more phases happening in serial, you will appreciate the benefits of the intervals shown in the Foretify GUI, which tell you precisely when each phase starts and ends, as you can see in the screenshot below:

<p align="center">
  <a href="images/l02_intervals.png" target="_blank">
    <img src="images/l02_intervals.png">
  </a>
</p>

## Coverage Collection

Metrics-driven verification & validation (V&V) is a process that regularly measures the verification results using metrics so that decisions can be made based on the collected metrics. This leads to a number of benefits. Metrics-drive V&V:

- Eliminates human errors from V&V status estimation&mdash;the V&V status is objective
- Allows engineers to adapt their decisions and focus their efforts on an objective measure
- Accelerates the V&V process with instantaneous feedback from one iteration to the next
- Provides transparency across the decision hierarchy and across detail layers&mdash;from engineer to CEO, from SUT speed to overall metrics scores
- Makes the V&V process insensitive to the SUT's behavior&mdash;the metrics reflect what really happened, not what the engineer expected to happen
- Is insensitive to scale&mdash;V&V scales naturally with the size of the effort
- Allows assessment of the quality of the SUT, that is, GO/NO-GO decisions can be taken based on objective statistics

As you may have noticed, metrics-driven verification and validation is different from test-driven verification and validation, where the progress is measured by the number of passing tests. Arguably because of the points above, using metrics-driven V&V brings many advantages when compared to test-driven V&V.

The Foretellix solution complements traditional KPIs with coverage metrics, which are the subject of this section. This means that you can still use traditional ways to assess quality, while adding new metrics to increase the precision of verification goal definition.

### What is coverage?

Engineers define verification metrics to describe the verification goals in measurable terms. Coverage definitions specify **which** attribute / parameter values have to be exercised. For example, say that in order to prove the SUT behaves correctly in a cut-in scenario you need to make sure the following two parameters are exercised within the given ranges at a given point in time:

|coverage item |range  |buckets |unit|when         |
|--------------|-------|--------|----|-------------|
|sut_speed     |[0..200]kph|every 10| kph|end of cut-in|
|car1_distance |[0..50]m  |every 5 | m  |end of cut-in|


Why did we pick the ranges [0..200] and [0..50]? The answer is that the ranges are defined based on an engineering specification that captures how the system works and what its strengths and weaknesses are. As an example, if a coverage item is modeled as an integer, it would have a value space of 2^32 values. Not all of these values are valid or useful from the verification point of view. So, the table above specifies the so called "valid value space" which restricts sample collection to only the relevant values. Limiting the space ensures that the range of interesting values are covered, without wasting verification resources on irrelevant or invalid spaces, such as an NPC driving at hypersonic speed.

It is the V&V engineer's decision to restrict a coverage item's value space to only specified valid values. Even within the legal value range, you can refine and create smaller ranges called buckets (in this case, in 10kph buckets and in 5m buckets, respectively). The buckets are created on the assumption that the behavior and quality implications are similar for all values in a bucket. The _when_ parameter specifies the sampling point&mdash;when the parameter's value is saved into the coverage database.

OSC2 allows coverage items to be defined as part of a given scenario. For example, to define a coverage item as the first row of the table below for the sut.cut_in scenario, you would write the following code:

```osc linenums="1"
# here we extend the cut_in scenario
extend sut.cut_in:                                                                  
    # here we decide when to sample and what signal
    var sut_speed := sample(sut.car.state.speed, @cut_in.end)                       
    cover(name: sut_speed,
            # a description of the coverage item
            text : "Absolute speed of sut at cut_in end (in km/h)",  
            # the physical unit the coverage item represents               
            unit : kph,
            # the relevant range of values, everything outside of it is ignored                                                            
            range : [0..200],
            # size of the buckets (e.g. [10..20], [20..30] etc)                                                   
            every : 10)                                                             
```
!!! tip
  
    See [cover() and record () parameters](../osc_lang/osclang_behavior_monitoring.md#cover-and-record-parameters) for `cover` syntax and definitions. 

Now assume that both the coverage items above are implemented and 77 simulations are run. The results will be similar to:

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

You can see that not all of the buckets were hit, which means the coverage items are not fully covered. But how much is covered? The grade of a coverage item gives the fill level. The coverage grade is a multi-dimensional representation of all situations encountered during testing.

The grade of a coverage item is a percentage: how many of an item's values were covered out of the total number. In this case, the coverage is:

- sut-npc distance coverage - 23% : 3 out of 13 buckets were hit (i.e., values sampled were within the bucket range)
- sut speed coverage - 38% : 5 out of 13 buckets were hit

This means you would have to continue running simulations until you have a 100% coverage grade. If some of the buckets are not hit, you can decide to change the constraints on the scenario in order to target different values.

!!! Info
    Coverage-driven verification (CDV) is a proven successful methodology, used as the industry standard in various fields where the device under test is highly complex, in particular, the semiconductors industry. This approach helps find bugs continually before the products are deployed to the market, avoiding extremely costly recalls.

    Foretellix experts applied more than 20 years of CDV experience in the semiconductor industry in V&V solutions for the safety of autonomous systems. Thanks to the Foretellix tools enabling the CDV approach to autonomous vehicle verification, V&V engineers can apply a proven methodology in order to efficiently monitor the status of the verification process.

    Coverage-driven verification represents a major part of the safety-driven verification methodology.

### How to define a coverage item

When a coverage item does not require sampling (if, for example, it is an input of the scenario), it can be defined as follows:

```osc linenums="1"
    cover(name: aside, expression:side)
```

This creates a coverage item that uses the _side_ field of the scenario at the start of the _cut in_ segment. The _side_ item will have a grade of 100% (i.e., fully covered) as soon as both values of _left_ and _right_ are hit. If only one of the values is hit, the grade of the item will be 50%.

When a coverage item requires sampling, it can be defined as follows:

```osc linenums="1"
    var rel_speed := sample(car1.state.speed - sut.car.state.speed, @cut_in.end)
    cover(name: rel_speed,
            text : "Relative speed cut_in end (in kph)",
            unit : kph,
            range : [-50..50],
            every : 10)
```

If the parameter is an integer, the coverage space is quite large (i.e., 2^32-1 values). Often times, not all integer values are relevant. OSC2 lets you specify the legal range of an integer as in this example (with _range_ and _every_).

The _rel_speed_ is sampled on the _cut_in.end_ event. Its value at this point in time is matched to the bucket that contains it and saved as such. Once there is a sample in a bucket, the bucket has a grade of 100%. By definition, a bucket has a grade of 100% if at least one sample matches the bucket's interval.

!!! Example "Hands-on Time"
    Take a look at the coverage items that are defined in `$FTX_WORKSHOP/l02_second_iter/cut_in_and_slow_l02_cov.osc`. Can you recognize and explain what the coverage items mean? Note that the function _map.abs_distance_between_positions_ that is used here provides the absolute distance between two positions.

     Since you already conducted a few runs, you can collect and upload them to Foretify Manager using the following command:

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir2
    ```

     After opening Foretify Manager using the command, `fmanager`, create a new workspace called _cut_in_and_slow_ for the test run as you learned in Lab 1. At the end, what you see should be similar to the following image (note that the percentages might change):

    <p align="center">
      <a href="images/l02_vgarde_no_cross.png" target="_blank">
        <img src="images/l02_vgarde_no_cross.png">
      </a>
    </p>

    Inspect the coverage metrics in the workspace. You should have four items: aside, sut_speed, rel_speed and car1_distance. What is the coverage grade for the four coverage items? What is the overall VGrade?

### How to define a coverage cross

A cross is a product of two or more coverage items that highlights a relationship between the items. The buckets of a cross coverage item are the Cartesian product of individual items' buckets. Considering the example above, the cross of the SUT speed and the SUT-to-NPC distance looks like this:

<p align="center">
  <a href="images/workshop_lab_5_cross_samples.png" target="_blank">
    <img src="images/workshop_lab_5_cross_samples.png">
  </a>
</p>

The samples fall within the green cells. Similar to the coverage item, the grade of a coverage cross is a percentage: 10 out of 169 buckets were covered, that is, 5.9%.

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
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir3  \
    --batch --crun 5
    ```

    You can now collect and upload the runs to Foretify Manager using the following command:

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir3
    ```

    After opening Foretify Manager using the command, `fmanager`, create a new workspace called _cut_in_and_slow2_ using the new runs that you imported. Can you recognize the new coverage crosses? Inspect the bins (buckets) of the crosses and note how they correlate to the individual items that create the cross.


## KPI implementation

Key Performance Indicators (KPIs) can be as simple as raw metrics measured to see how well the AV performed. There can be safety-related KPIs such as min-Time-To-Collision (min-TTC), measured in seconds, comfort-related KPIs such as max-deceleration, measured in meter/seconds, and so on.

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

        # Record the result of the collect construct at the end of the scenario to get the overall minimum ttc in this run
        record(min_ttc_kpi,
            expression: min_ttc.computed_result,
            unit: s,
            event: sample_ttc,
            text: "Minimum Time to Collision (TTC)")
    ```
!!! Example "Hands-on Time"
    After saving, run a few executions using the updated code:

    ```bash
    foretify --load $FTX_WORKSHOP/l02_second_iter/ts_l02.osc \
    --work_dir $FTX_FM_WORKDIR/l02_second_iter/workdir4 \
    --batch --crun 3
    ```
    You can now collect the runs and upload to Foretify Manager using the following command:

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l02_second_iter/workdir4
    ```
    Have a look now at the KPI results in Foretify Manager.

## Checkers implementation

### Understanding and resolving issues
One of the main verification challenges is to determine&mdash;in the most efficient manner&mdash;whether the SUT performed correctly. For example, you want to know if the SUT:

- Did not meet the minimum legal highway speed.
- Failed to keep a proper distance from pedestrians.
- Drove off the road.

Checkers are OSC2 entities that define the ways that the SUT should or should not behave. The simplest checkers define a failure response if the SUT fails to meet a boolean condition. 

### How to implement a checker

When working with scenarios, you will likely need to define checkers to capture problematic behavior or conditions specific to your SUT. Now you will try a new example similar to the example from Lab 1. For this example, you will adapt the newly written KPI from the previous exercise: min_ttc. The purpose of the checker will be to ensure that the TTC does not exceed the value of 3 seconds at the start of the slow phase.

The steps to write the checker are the following:

1. Create a new file called _cut_in_and_slow_l02_checker.osc_.
2. Create a new issue_kind and call it safe_ttc
3. Define a new variable for the safe TTC threshold type.
4. Set the safe TTC threshold value to 3 seconds.
5. Transfer the _collect_time_ construct from _cut_in_and_slow_l02_cov.osc_ to _cut_in_and_slow_l02_checker.osc_.
6. Add the following parameters to the _collect_time_ construct:

        - bad_is: in what direction to compare the result to the threshold (here: low)
        - threshold: the threshold to compare with (here: our newly defined threshold)
        - first_failure_kind: the kind of alert that should be raised if the threshold is violated (here: safe_ttc)
        - first_failure_severity: the severity of the alert that should be raised (here: error)

7. Import the new file _cut_in_and_slow_l02_checker.osc_ from the _ts_l02.osc_ file. 

!!! tip 
    
    Brief checker syntax reminder from Lab 1 (the group of words marked with <> needs to be adjusted):
    ```osc
    extend issue_kind: [<checker_name>]
   
    extend sut.<scenario_name>:
    
        var <checker_threshold>: time 
        set <checker_threshold> = 3s 

    ```

??? Solution "Solution - Click Here" 

    ```osc linenums="1"

    extend issue_kind: [safe_ttc]   

    extend sut.cut_in_and_slow: 
    
        var ttc_kpi_threshold: time # Safe TTC threshold type
        set ttc_kpi_threshold = 3s # Safe TTC threshold value

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
            text: "Minimum Time to Collision (TTC)")
                
    ```

## Coverage Practice

The purpose of this practice is to:

- Add a new cross-coverage item.
- Run tests.
- Examine results using Foretify Manager.

!!! Example "Hands-on Time" 
    Add a new cross coverage item defined by the side from which the cut-in happens and the relative speed at the end of the cut-in.

    Run additional tests with Foretify (you can use the crun option to automatically run multiple times). It's a good idea to define a new working directory, for example, $FTX_WORKSHOP/l02_second_iter/workdir4.

    Look at the results in Foretify Manager. What can you do to increase coverage?

## Next Steps

This concludes the second safety-driven verification iteration. In this lab, you:

- Became familiar with the scenario reuse.
- Used scenario operators such as _serial_ and _parallel_.
- Debugged the scenario by using the traces and Visualizer.
- Became familiar with the concepts and definitions of coverage items and KPIs.

Next, in Lab 3, you will learn how to scale up the number of tests.
