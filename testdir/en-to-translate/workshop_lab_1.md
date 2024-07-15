---
chapnum: 1
---
# Lab 1: First Steps with OpenScenario 2 and Foretellix Technology

## Learning goals

This lab introduces you to the following:

- A simple **ASAM OpenSCENARIO 2.0 (OSC2)** test implementation. (ASAM is the Association for Standardization of Automation and Measuring Systems.)

!!! Note
    Throughout the Foretellix documentation, the term "OSC2" refers to "ASAM OpenSCENARIO® DSL versions 2.x."

- **Foretify™**, the **scenario development and test automation platform** that is used for:
    - Compiling OSC2 sources
    - Generating concrete tests from an abstract scenario definition
    - Controlling the actors in the simulation platform during the test execution
    - Plotting and visualizing the results of the test execution

- **Foretify Manager**, the **big data analytics platform** that is mainly used for:
    - Collecting Key Performance Indicators (KPIs), checker messages, and coverage metrics of multiple test executions
    - Visualizing the progress in testing, enabling the Safety-Driven Verification (SDV) methodology approach

<p align="center">
  <a href="images/l01_ftx_diagram.png" target="_blank">
    <img src="images/l01_ftx_diagram.png">
  </a>
</p>

!!! Note
    Clicking on any image opens it in full resolution in a new tab.

## The OSC2 language

During both development and verification processes of AD and ADAS functions, it is necessary to stimulate the System Under Test (SUT) with various *scenarios*. A scenario is a timed sequence of actions by one or more actors, such as cars, pedestrians, environmental conditions and the SUT itself. OSC2 is a domain-specific language, specifically designed for describing scenarios where actors move through an environment. These scenarios have attributes that allow you to constrain the actor types, their movements, and the environment (including the location on the map where the scenario should take place).

!!! Info 
    With a *constrained random* approach, every scenario attribute that is not constrained is randomized. As an example, if you do not constrain a cut-in scenario "side" attribute to be "right", it will be randomly chosen from the space of possible attributes (namely, "left" and "right").

    Additionally, the values are chosen from the space of possible attributes so that they satisfy the constraints, deriving from the scenario, the actors, and the map. As an example, if the SUT (EGO) is driving on the outmost right lane of a 2 lanes road, the cut-in "side" attribute will not be chosen to be "right".


The building blocks of OSC2 are data structures, such as:

- **Actors**: represent real world entities. As the name suggests, they are "playing a role" in the scenarios.
- **Scenarios** or **Actions**: describe the behavior of actors. Generally, a scenario is a longer sequence of actions, but there is no formal difference between the two. Both can be modified through *modifiers*.
- **Modifiers**: add constraints to scenarios, helping control their execution within the desired boundaries.
- **Labels**: define a named data field of any scalar, struct or actor type.
- **Simple structs**: are basic entities containing attributes, constraints and so on.

You can learn more about the topics above by referring to the Foretellix [OSC2 language documentation](../osc_lang/osclang_intro.md) and [OSC domain model documentation](../osc_dom/oscdomain_intro.md) or by visiting the [ASAM Type Definitions topic](https://www.asam.net/index.php?eID=dumpFile&t=f&f=3460&token=14e7c7fab9c9b75118bb4939c725738fa0521fe9#type-definitions) or the main [ASAM OSC2 web page](https://www.asam.net/project-detail/asam-openscenario-v20-1/).


!!! Info 
    Foretellix tools **natively support the ASAM OSC2 language**, resulting in many benefits, one of which is that the language and tools are **execution platform agnostic**. This results in **reduced effort** when changing the verification environment and **increased flexibility** in choosing the appropriate verification platform.

    OSC2 supports **abstract scenarios** and a **Safety Driven Verification** flow. Through this workshop, you will understand how these features can **streamline V&V efforts**, resulting in **fewer resources needed** to verify autonomous systems.

### Our first test

A test is OSC2 code from which the scenario is invoked, it is considered to be the top layer hierarchically:

1. It imports the test execution platform configuration (e.g., the simulator)

2. It imports the SUT configuration (e.g., the system under test, also often known as EGO). In this case, as function being tested, we are importing an SUT L4 stack developed by Foretellix and configuring the attributes of the EGO vehicle.

3. It sets the map to be used in the test.

4. It defines the scenario and metrics (Checks, Coverage, KPIs), and invokes the scenario execution.

In the following image, you can see the structure of the test you will run:

<p align="center">
  <a href="images/Test_2.png" target="_blank">
    <img src="images/Test_2.png">
  </a>
</p>

!!! Example "Hands-on time"
    Now that you are set to start the workshop, it's time to look into the first test in more detail.

    Open the first test with the Foretify Developer OSC2 code compiler:

    ```bash
    foretify --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --gui
    ```

The Foretify GUI will appear in your browser. You will learn more functionality during this training. For now you will focus on its OSC2 code compilation and visualization features. Click the **Source** tab and collapse the **Loaded Files** pane as shown in the image below:

<p align="center">
  <a href="images/l01_ftx_dev.png" target="_blank">
    <img src="images/l01_ftx_dev.png">
  </a>
</p>

In the test, notice the _import_ statements which load other OSC2 files:

```osc linenums="3"
import "$FTX_WORKSHOP/common/workshop_config.osc"
import "$FTX_WORKSHOP/scenarios/cut_in_l01.osc"
import "ts_l01_intro_cov.osc"
import "ts_l01_intro_checks.osc"
```
All the code could be in one file, but that would make it less readable, less reusable, and difficult to manage. 

We will now go through the content of the import statements, as well as the rest of the code in the following sections.

### The _workshop_config.osc_ configuration file

The `workshop_config.osc` file (imported in line #3 of the test file) contains all definitions for setting up Foretify and the execution platform connection (it can use differents simulators), as well as the System Under Test (SUT) connection (in this case, the autonomously driving Ego).

### The _cut_in_l01.osc_ scenario file

This `cut_in_l01.osc` file is imported in line #4 and contains the abstract definition of a cut-in scenario, the subject of this lab. What we are defining, through a set of absolute and relative constraints, is that a vehicle should change lane in front of the SUT. OSC2 is the only scenario description language that supports abstraction, significantly reducing the amount of time scenario developers need to spend in writing code.
In the image below you can see two concrete instances of the abstract cut-in scenario definition.

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_scenario.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_scenario.png">
  </a>
</p>

!!! Example "Hands-on Time"
    Open the `cut_in_l01.osc` file that is imported by clicking on line #4 of the test file in the Source tab.


The code implementing the cut_in scenario shown below:

<p align="center">
  <a href="images/workshop_l01_cut_in_code_vscode.png" target="_blank">
    <img src="images/workshop_l01_cut_in_code_vscode.png">
  </a>
</p>

Let’s walk through the code:

- Line 6: The scenario called sut.cut_in_l01 is declared, in the context of the SUT (that is why it is sut.name_of_scenario)
- Line 7: car1, the other car (not the SUT) is instantiated as an object of type "vehicle".
- Line 8: The side from which car1 will cut in from is instantiated as an enumerated type “av_side”. This means that it can hold the value "left" or "right"
- Line 10: Indicates that the subsequent blocks will be executed one after the other. In this case it means that the phase called approach_phase will be executed after log_info, followed by the phase change_lane.
- Line 12: Writes the cut-in side to the log. The log statements allow you to add lines that can later be used for debugging purposes.
- Line 14: A parallel scenario phase is created and labeled with "approach_phase". This means that the lines 15 and 18 are going to be executed in parallel (remember, OSC2 is an indentation-based language). A label allows you to refer to that part of the code from other areas of the code.
  - Line 15: Triggers the SUT to start the action drive(). The following lines (16 and 17) add some constraints on the drive actions of the SUT, namely:
    - Line 16: Constrains the speed of the SUT to be at least 30 km/h at the start of the approach phase
    - Line 17: Constrains the SUT to keep its lane trough the approach phase
  - Line 18: Triggers car1 to start the action drive(). Since car1 is not the SUT, it will be fully controlled by the Foretify engine while driving. The following lines (19 to 22) add some constraints on the movement of car1, so that it will result in a cut_in maneuver, namely:
    - Line 19: car1 is constrained to be in the lane adjacent to the SUT throughout this phase.
    - Line 20: The position of car1 relative to the SUT is defined for the start of this phase (10 to 20 meters ahead of the SUT).
    - Line 21: The position of car1 relative to the SUT is defined for the end of this phase (10 to 20 meters ahead of the SUT).
    - Line 22: The position of car1 relative to the SUT is defined to be a best effort condition. This means, that the solver will not label this scenario as failed in case the constraint can not be fulfilled. In some cases, the planned run can not unfold completely due to limitations of the simulator and are labeled as _incomplete scenarios_. Defining non-critical constraints as best effort constraints can help decrease the share of such runs. You will learn more about this feature, and about the difference between the "the plan" and "runtime" in the advanced labs.
- Line 23: A second parallel scenario phase is created and labeled with "change_lane". Again this means that the subsequent actions in lines 24 and 32 are executed in parallel.
  - Line 24: Triggers the SUT to start the action drive(), followed by several modifiers.
    - Line 25: Adds a constraint on the drive action of the SUT, namely to keep its lane throughout the scenario phase.
  - Line 26: Triggers car1 to execute the drive() action, constrained by lines 29 to 34:
    - Line 27: Constrains the speed of car1 to be 5 to 15 km/h slower than the speed of the SUT at the beginning of the phase.
    - Line 28: Overrides the speed constraint to be a non-critical, best effort constraint.
    - Line 29: Constrains car1 to be in the same lane as the SUT at the end of the scenario phase.
    - Line 30: The speed of car1 is to be kept constant over the course of the scenario phase.
    - Line 31: Again, this speed constraint is defined to be non-critical, but to be executed at best effort.
    - Line 32: This line is deactivating the collision avoidance behavior for car1, thus making the lane change challenging for the SUT. You will learn more about the collision avoidance behavior of the generic car actor in the advanced labs. 

### Behavior monitoring

The following three sections will briefly introduce the notions of _coverage_, _KPI or record_ and _checker_, with the help of some examples. They are key pillars of the Safety-Driven Verification methodology, since they support the verification process and reveal incorrect performances of the SUT. In the next lab, you will take a deep dive and elaborate on their functionality. 

#### The _ts_l01_intro_cov.osc_ coverage definition

!!! Example "Hands-on Time"
    Open the `ts_l01_intro_cov.osc` file that is imported by clicking on line #5 of the `ts_l01_intro.osc` test file in the Source tab.


<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_osc_cover.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_osc_cover.png">
  </a>
</p>

The code above defines the **coverage collection**:

- The first metric (line 4) is the side from which the cut-in happened&mdash;left or right.
- The second metric (line 5) is the type of the car that does the cut-in, for example, truck or sedan.
- The next metric (lines 6 to 11) is the speed at which the car is driving at the end of the lane_change scenario phase.
- The last (lines 12 to 18) is the longitudinal distance between the two cars, when the main scenario starts.

!!!Info
    - In line 6 we have just used one important structured type member of OSC2: _event_. The event is _end_ and more precisely it represents the end event of the change_lane phase defined in the `$FTX_WORKSHOP/scenarios/cut_in_l01.osc` scenario. 
    - Events are transient objects that represent a point in time and can trigger actions defined in scenarios. You can define an event within a struct, but more typically within an actor or a scenario, as in the example. 

#### The _ts_l01_intro_cov.osc_ KPI definition

The KPI is defined in the same file as the coverage items from the previous section.

In this part of the code, we define a KPI:

- First, a variable for sampling the distance between the SUT and the cut-in car at the end of change_lane is declared in lines 21 to 22.
- Lines 25 to 27 use the _record()_ method, to record the KPI to be visualized later.

!!! Info
    Now that you have gone through some examples, it is essential to understand the main difference between a coverage metric and a performance metric:

    - **Coverage evaluation**: _In which part of the “scenario space” have we exercised our AV?_ This is expressed via coverage and an overall coverage grade. Coverage items are defined to support the coverage evaluation. In other words, the coverage grade answers the question: _How well was the SUT tested?_
    - **Performance evaluation**: _How well did the SUT perform in the tests?_ This question is answered by a performance grade, that can be one or multiple KPIs

#### The _ts_l01_intro_checks.osc_ checker definition

!!! Example "Hands-on Time"
    Open the `ts_l01_intro_checks.osc` file that is imported by clicking on line #6 of the `ts_l01_intro.osc` test file in the Source tab.

<p align="center">
  <a href="images/l01_kpi_code.png" target="_blank">
    <img src="images/l01_kpi_code.png">
  </a>
</p>

The purpose of this checker is to assess whether the distance between the SUT and the cut-in car is within the defined safety distance or not, throughout the scenario:

- In line 1, the issue_kind type is being extended with a unique name (safety_distance) for the new checker.
- Lines 3 to 12 extend the previously defined scenario for writing the checker as follows:
    - Lines 5 - 6: A variable for the safety distance threshold is declared, and set to a value of 13 meters
    - Lines 8 to 10 verify at each time step (top.clk) of the simulation, whether:
      - the distance between the two cars is above the defined threshold
      - both vehicles are in the same lane
    - Lines 11 to 12: If the condition above is not met, then the test run is stopped with an error of the type sut_error, and subtype safety_distance.

!!! Info

    - The checker just added is a user-defined checker, but Foretify comes with built-in checkers. You can review those in the [Global checkers for vehicles documentation](../osc_dom/oscdomain_metrics.md#global-checkers-for-vehicles).

    - Another term used in the industry for checker is evaluator.

    - With checkers you can represent the success or failure criteria for scenarios, and they are written with the same language (OSC2) used to represent actions and actors.

    - Some predefined checkers are always active when using Foretify, such as the collision check or a check for the SUT driving off-road. You can always modify the defaults that apply for all scenarios.

    - You can define custom checkers to capture issues specific to your SUT, as in our example. The checker we have exercised uses the KPI value and a predefined threshold, in order to evaluate the SUT's expected behavior, but this is just one way of defining it. 

### Map definition

In the code below (lines #8 and #9 of the test file `ts_l01_intro.osc`), we set the map to be used, a map that is in OpenDrive format (i.e. \*.xodr).

```osc linenums="8"
extend test_config:
    set map = "$FTX_WORKSHOP/maps/Town04.xodr"
```

### Scenario execution

In the last two lines of the `ts_l01_intro.osc` file, we finally call the _cut_in_l01_ scenario: the OSC2 program's entry point which is always executed is the _top.main_ scenario, similar to C/C++ _main()_ function: 

```osc linenums="11"
extend top.main:
    do cil : sut.cut_in_l01()
```
You can find this code in lines #11 and #12 of the test file.

## First run and Foretify

### Constrained random generation and adaptive scenario execution

As the name implies, *constrained random generation* means to generate random variables within the space of specified constraints. This is a fundamental premise for the SDV (Safety-Driven Verification) flow and is the core principle around which Foretify is built.

From OSC2 abstract scenarios, Foretify’s generation engine creates concrete scenarios randomly, given the constraints that are specified. One of the most important aspects which is being randomized, is the area of the map where each scenario will unfold.

Once a plan for the scenario has been calculated, the runtime adaptive scenario execution engine takes care of the execution according to the scenario plan.

!!! Info
    The **constraint random generation** engine is the main pillar of the the Foretellix solution. This is an extremely powerful tool that can generate meaningful scenario variations out of abstract descriptions.

    The **engineering resources** needed for the scenario writing team **are significantly reduced** when leveraging this technology, since out of one abstract definition millions of meaningful variations are generated.

    The generated scenarios, when executed, will help in challenging the system under test in order to **spot and solve bugs in a more efficient manner**.


### Your first run

**Before moving on, make sure that you closed the Foretify GUI in your browser.**

Now that you walked through the OSC2 scenario definition, you will be using simulator as your test execution platform. 

Now you will explore the Foretify GUI more in detail, to load, launch, and analyze tests. Later you will explore more functionality of Foretify.

!!! Example "Hands-on Time"
    Launch Foretify in GUI mode and load the test you previously examined:

    ```bash
    foretify --gui --work_dir $FTX_FM_WORKDIR/l01_intro/workdir \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc
    ```

The Foretify interactive window appears:

<p align="center">
  <a href="images/foretify_gui.png" target="_blank">
    <img src="images/foretify_gui.png">
  </a>
</p>

The Foretify window displays the following information:

- **Load**, **Prepare Test** and **Debug** tabs (top left):
    - The **Load** tab is used to load `.osc` files through the GUI (we did not use the GUI for this, since we loaded the file with the terminal command). 
    - The **Prepare Test** tab allows you to set up different parameters of the run. You'll learn more about this in a later section. 
    - The **Debug** tab is used to debug the run after the simulation is completed.
- **Status** (mid-top left):
    - You can see the loaded file, the loading status, as well as the number of issues found during the loading.
- **Map**, **Source** and **Preview** tabs (mid-left): 
    - The **Map** tab allows you to browse the loaded map, exploring different layers of the map. 
    - The **Source** tab shows the loaded source files, allowing you to review the loaded code and move between the loaded files. 
    - The **Preview** tab allows you to preview the planned test, after clicking the **Preview** button in the **Execution Control** area.
- **Issues**, **Linter Violations** and **Test Info** tabs (bottom-left)
    - The tabs display once the scenario is loaded.
- **Control the execution** of the scenarios (upper right). 
    - You can set the seed for the run, preview the run or run the actual simulation. A seed is a unique identifier for each concrete test-execution that will be generated out of an abstract definition.
- **Log** (mid right)

!!! Example "Hands-on Time"
    Before actually running the test, you can experiment with the **Preview** button, which shows the planned routes for the SUT and the car1 in Visualizer. Note that the planned path might change at run time due to the SUT behavior, but this is an extremely useful tool to debug your test. Note that the planned path is just a representation of the plan that has been created for that scenario: the real trajectories are calculated and updated during runtime.

!!! Example "Hands-on Time"
    Try to preview a few seeds of your your test before running it by clicking the grey **Preview** button in the top right corner, and changing the seed number.

!!! Example "Hands-on Time"
    Run a test by clicking the purple **Run Test** button in the top right corner. You can also run the test by clicking the **Terminal** button in the top right corner and typing _run_.

You will see the Simulator window pop up:

- The Foretify constraint random generation engine, generated a specific variant of the scenario, based on the constraints and parameters that are specified in the OSC2 code for the scenario. The planned paths for the actors are the ones that you have seen when previewing the run.

- The Foretify runtime-test orchestration engine, ensured that the actors in the test execution platform (the Simulator) were able to move according to the specified constraints.

The runtime-test orchestration engine ensures an **adaptive scenario execution**, meaning that a deviation from the planned path for the SUT will result in corresponding countermeasures from the NPCs, so that the intent of the scenario specified in the OSC2 file will be met.

After the test is completed, the log area shows two additional tabs:

- The **Trace Details** tab, lets you inspect the scenario Trace information with values collected throughout the simulation, like Trace Type, Actor, Time and Duration.
- The **Log** tab, contains the log generated during the test execution.
- The **Metrics** tab, is related to the coverage and will be detailed later on in the workshop.

!!! Example "Hands-on Time"
    For now, review the **Log** tab and check if you see the log message we added to show on which side the cut-in happens.

### Debug a run

#### Debug with the Foretify Visualizer

After running the test, in the middle of the screen, you will have access to the **Visualizer** tab. Visualizer is one of the graphical post-processing tools that can be configured in various ways to help you analyze the execution. Visualizing a run is different from repeating the execution. It does not require the simulator and it will not consume the compute resources that a rerun would.


Once the execution is finished, the **Visualizer** tab on the left side of the screen will automatically open.
You can click on the tab anytime to return to Visualizer. 

<p align="center">
  <a href="images/visualizer.png" target="_blank">
    <img src="images/visualizer.png">
  </a>
</p>


##### Replay test
First, you can press the play button in the Visualizer timeline located at the bottom left and see how the scenario is replayed:

<p align="center">
  <a href="images/visualizer_play_button.png" target="_blank">
    <img src="images/visualizer_play_button.png">
  </a>
</p>

##### Map perspective
You can change the perspective by clicking the right mouse button in Visualizer and dragging the cursor in a different direction. You can access **View Tools** by clicking the wrench icon in the top right of Visualizer. **View Tools** has options to control the view:

<p align="center">
  <a href="images/visualizer_tools.png" target="_blank">
    <img src="images/visualizer_tools.png">
  </a>
</p>

- Lane Directions: Enables or disables viewing of driving direction's arrow.
- Signals: Toggles visibility of traffic signals.
- Speed limits: Displays the speed limit of a road.
- Collision avoidance: Enables the visibility of collision avoidance activation.
- Runtime Trajectories: Displays the trajectory of the selected vehicle actor.
- Planned path: Highlights the path of the vehicles present in the scenario.
- Planned objectives: Displays the planned objectives generated for the selected vehicle actor.
- Planned pose: Highlights the next pose of the SUT and other vehicles.
- Driver objectives: Displays the driving objectives generated for the selected vehicle actor.
- Projected Pose: Displays the projected position of the vehicle.

##### Camera settings

To control the perspective and the camera, click the Camera Settings (camera) icon. To track a particular actor with the camera, select an actor from the top left corner (i) dropdown list or click in Visualizer to set the camera to a fixed position.

<p align="center">
  <a href="images/Camera_settings.png" target="_blank">
    <img src="images/Camera_settings.png">
  </a>
</p>

- Perspective view: Changes the perspective to top-down.
- Follow selected actor: Follow the actor selected.
- Reset camera to selected actor: Reset the camera to the position of the selected actor.

##### Measure distances

- If Visualizer is in Perspective view, select the Camera Settings icon on the top right of Visualizer and turn off the Perspective view option.
<p align="center">
  <a href="images/Measurement_1.png" target="_blank">
    <img src="images/Measurement_1.png">
  </a>
</p>

- Select the Measure Distance tool icon.
<p align="center">
  <a href="images/Measurement_2.png" target="_blank">
    <img src="images/Measurement_2.png">
  </a>
</p>

- Point and click in Visualizer to set the starting point of your measurement, then move the cursor and click to set the end point of your measurement.
<p align="center">
  <a href="images/Measurement_3.png" target="_blank">
    <img src="images/Measurement_3.png">
  </a>
</p>

The measurement displays next to the line that connects the starting point and end point.

- To hide the measurement, toggle off the Measure Distance tool icon.

#### Debug with Traces

All traces can be viewed under the Traces view. The Traces view is aligned with the timeline, so you can easily compare different traces.

**Traces** are represented as these distinct types:

- **Intervals**: Represent a collection of values collected over a period of time. Intervals have a name, start/end time and type and are associated with a specific actor (Orange box).

- **Values**: Represent a single value that changes over time, values are displayed as waveform graphs, value traces have a name, value and unit and are associated with a specific actor (Red box).

<p align="center">
  <a href="images/Traces_1.png" target="_blank">
    <img src="images/Traces_1.png">
  </a>
</p>

To enhance the visualization, Foretify records the start and end times of every scenario.

##### To view Intervals:
- Select the Debug Run tab in Foretify and click the Traces tab under Visualizer:

<p align="center">
  <a href="images/Traces_2.png" target="_blank">
    <img src="images/Traces_2.png">
  </a>
</p>

Traces are shown as Intervals in a timeline with a current time cursor that corresponds to other time-based views such as Visualizer and the Actor value traces inside Actors in the Traces tab.

1. Click the arrow to the left of a trace name to expand it and see its child scenarios.

2. Click on a trace to view the trace details, such as the Trace Type, Actor, Time, Duration, and metrics collected during the interval.

3. Under Traces Details, click on the Start time or End time of the trace to set the universal timeline to that time.

##### To frame the Timeline to a Trace:
1. On the Traces tab, select the interval you want to frame the Timeline (Orange box).

2. Click the Frame timeline icon (Red box).

<p align="center">
  <a href="images/Intervals_3.png" target="_blank">
    <img src="images/Intervals_3.png">
  </a>
</p>

- To reset the Timeline so that it no longer frames the interval, click the unframe timeline icon to the right of the Timeline.

<p align="center">
  <a href="images/Intervals_4.png" target="_blank">
    <img src="images/Intervals_4.png">
  </a>
</p>

!!! Example "Hands-on Time"
    Replay the test using Visualizer and examine the SUT's behavior and the different options discussed above.

### Running different seeds

!!! Example "Hands-on Time"
    Use the execution control area in the top right to run another simulation by setting the seed number to 4 and clicking the **Run Test** button. This is one seed where the checker that we introduced on the minimum distance threshold will fail.

    Run another simulation with a seed of your choice.

    Again, search the log for the message that indicates on which side the cut-in happened.

    Now you can close Foretify by typing exit in the Foretify terminal or closing the Foretify window.

!!! Info
    The **seed** serves as input for the random generation of one specific concrete execution out of an abstract scenario definition. Generating the concrete test out of the abstract definition again with the **same seed** results in the **same concrete variation**. This is a critical feature that lets you fully randomize the testing on one hand, but also recreate a single concrete execution for debug purposes.

    Thanks to the seed definition and implementation, you can always **trace the particular concrete variation generated.** Traceability of scenarios is a fundamental feature that lets you identify the conditions that led to the bug and to reproduce them.

## Foretify Manager

Now that you've run the first tests, you can explore the achieved coverage and results. We will formally define coverage in the Lab 2, but for now you will visually explore this concept using the Foretellix tools.

Foretify Manager is the Foretellix tool that lets you visualize the overall status of the verification process, importing results and KPIs of numerous test executions. It has a client-server architecture shown in the following image:

<p align="center">
  <a href="images/fmanager_architecture.png" target="_blank">
    <img src="images/fmanager_architecture.png">
  </a>
</p>

The client can be either a Python script or a web UI (a web page) and both of these can perform actions and queries on test suite results data. The server manages the database and executes the client's commands. This topology allows multiple users to concurrently analyze different aspects of the verification results.

###  Open Foretify Manager

Foretify Manager is a browser-based application.

!!! Example "Hands-on Time"
    Start Foretify Manager by calling the following command from the terminal: 

    ```bash
    fmanager
    ```

    The first thing you see is the login page:

    <img src="images/fmanager_login.png" alt="image" width="200"/>

!!! Info
    Reach out to the presenter for more information about your assigned username and password.



### Create a project

A Foretify Manager project is a collaborative framework, enabling the users to set permissions and ownership for the Validation and Verification data and collected metrics.

To create a project:

1. Open Foretify Manager and log in with the Foretify Manager credentials.

2. From the Select a Project page, click Create New Project.

!!! Info
    Reach out to the presenter for more information about the name of your project

- Click the Create purple button 

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

### Upload test suite results

At this point, the Foretify Manager database is empty, so the next step is to upload the results of the few tests that you executed.

!!! Example "Hands-on Time"
    Switch to a terminal and upload the results by calling:

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir
    ```

You can use the ``` --run_group_name ``` parameter to give your group of tests a specific name. See the [upload_runs documentation](../fman_user/fmanuser_launch_test_suite.md#upload-a-regression).

!!! Example "Hands-on Time"
    Switch back to the Foretify Manager web app that opened in your browser and hit the refresh button (make sure to be on the "Test Suite Results" tab)

After upload, the test executions should be loaded and displayed in your previously created Project:

<p align="center">
  <a href="images/fmanager_regression_2.png" target="_blank">
    <img src="images/fmanager_regression_2.png">
  </a>
</p>

### Analyze the uploaded runs

By clicking on the test suite that you just imported, you can have a look at the individual runs.

You should see something like in the image below:

<p align="center">
  <a href="images/fmanager_runs.png" target="_blank">
    <img src="images/fmanager_runs.png">
  </a>
</p>

The colored squares highlight:

- _yellow_: The icons in the top right of the Runs list let you export and delete runs, as well as keep and reset your choices. Use the rightmost Column Selection icon to add and remove run attributes, for example, the directory, the OS user, duration, and many more.
- _orange_: The _Issues Tree_ presents all the issues grouped by their kind.
- _blue_: The _Aggregation View_ lets you aggregate the runs based on a run attribute.

When clicking on a run in the _Runs_ view, a new Foretify Manager window appears.

<p align="center">
  <a href="images/Run_view_2.png" target="_blank">
    <img src="images/Run_view_2.png">
  </a>
</p>

As you can see, there are two main tabs: **Debug Run** and **Run Summary**. 
The **Debug Run** tab is exactly like the one we have previously seen in Foretify,

<p align="center">
  <a href="images/run_source.png" target="_blank">
    <img src="images/run_source.png">
  </a>
</p>

You can see more detailed information about the failed run by clicking on the **Run Summary** tab. 

<p align="center">
  <a href="images/run_summary.png" target="_blank">
    <img src="images/run_summary.png">
  </a>
</p>

!!! Example "Hands-on Time"
    Explore now your two other runs by clicking on each run.

### What is a workspace and how to create it

The workspace is a set of imported test suites, for which you want to analyze coverage data.

!!! Example "Hands-on Time"
    Select your test suite in the **Test Suite Results** tab and then click **Create Workspace** as shown below.

<p align="center">
  <a href="images/fmanager_workspace_2.png" target="_blank">
    <img src="images/fmanager_workspace_2.png">
  </a>
</p>

Choose a workspace name. Click then on **Create Workspace** to create the workspace.

<p align="center">
  <a href="images/fmanager_workspace_name_2.png" target="_blank">
    <img src="images/fmanager_workspace_name_2.png">
  </a>
</p>

The Foretify Manager web app switches to the **Current Workspace** view:

<p align="center">
  <a href="images/fmanager_workspace_after_creation.png" target="_blank">
    <img src="images/fmanager_workspace_after_creation.png">
  </a>
</p>


The workspace includes the following:

- **VGrade** is the overall metrics grade (you'll learn about it in Lab 4).
- **Total Runs** (next to **VGrade**) is a statistic of passed and failed runs.
- In the **VPlan** tab (blue), you can see a metrics hierarchy.
- In the **Runs** tab (green), you can see the list of runs that are selected by the current workspace.

!!! Example "Hands-on Time"
    Explore the **VPlan** tree and the runs in the **Runs** tab.

### Metrics and checkers representation

#### Coverage
In our `ts_l01_intro_cov.osc` coverage file, we have defined four coverage items and one KPI metric. 
For example, the cut_in_side has a coverage grade of 100%, while the speed_sut has a coverage grade of only 20%. This is an indicator that more tests need to be run in order to fill in the empty buckets for the speed_sut coverage.
In our file, these two coverage items were defined slightly differently:

 - Note: Percentages can be different depending of the seeds used!

<p align="center">
  <a href="images/lab01_coverage_conclusions.png" target="_blank">
    <img src="images/lab01_coverage_conclusions.png">
  </a>
</p>

 For the speed_sut, we specified the buckets, while for the cut_in_side, we did not impose any rule. This has given an extra degree of freedom to the cut_in_side coverage item. This item has a 100% coverage grade because both possible sides (left and right) were hit at least once during testing. 


!!! Example "Hands-on Time"
    Inspect now the other coverage items and their grades in foretify manager, under the VPlan tab. You should be able to find the _cut_in_side_ coverage item.

#### KPIs
As you recall, we defined the distance_kpi KPI earlier and it measures the distance between the SUT and the cut-in vehicle, at change_lane end event. Specifying the event helps us to better catch an item value at its highest peak of interest. Since the event is also specified, we notice that we have single values per run for this KPI.

<p align="center">
  <a href="images/workspace_kpi_2.png" target="_blank">
    <img src="images/workspace_kpi_2.png">
  </a>
</p>

!!! Example "Hands-on Time"
    Which KPI values did you get? You can review the simulation in Visualizer and see the correlation between the value and the simulation.

#### Checkers

When looking at the **Runs** tab, you can observe how many of the runs passed and how many failed. The failed runs are coming from the checkers defined in the code. Since the checkers define a failure response if the SUT fails to meet a boolean condition, this is instantly reflected in the Issues Tree.

<p align="center">
  <a href="images/checkers.png" target="_blank">
    <img src="images/checkers.png">
  </a>
</p>

!!! Example "Hands-on Time"
    Analyze your runs. How many failed runs did you get?

### Increase coverage

A few tests are by far not enough to achieve any meaningful coverage. The strength of the Foretellix technology is in running large numbers of automatically generated tests. So in this section, you will run more tests to increase the coverage. To do this, you need to create different test cases out of the scenario. As described above, this is controlled by the seed that is used during creation.

**Before moving on, make sure to close the Foretify GUI in your browser**.

!!! Example "Hands-on Time"
    Let's run 15 more tests, setting up a new work directory. Note that we will not use the foretify gui, so you will just see the simulator window popping up time in time.

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir2 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 20 --crun 15
    ```

    The ```--crun``` option will run 15 tests starting from seed 20 (which means seeds 20 to 34). 
    This is one option to scale-up your testing, but we will cover many more options in Lab 3.

Note how the generation engine generates 15 unique concrete tests, each of them in a different section of the map while keeping all the concrete tests within the boundaries defined by the abstract scenario (e.g., speed, lane position, etc.).

!!! Example "Hands-on Time"
    The new runs should increase the coverage. You can now upload the fifteen additional runs to Foretify Manager using the command:

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir2
    ```

    Then, in the **Test Suite Results** tab of Foretify Manager, select the newly uploaded test suite and add them to the workspace as shown in the image below

    You can now explore the **VPlan** tree and see how much the coverage improved.

<p align="center">
  <a href="images/l01_add_ws_1.png" target="_blank">
    <img src="images/l01_add_ws_1.png">
  </a>
</p>

### Switching between test suites

When you add new test suite to your workspace you will be able to switch between those test suites, by doing that you can analyze each one separately.

!!! Example "Hands-on Time"
    Go to your workspace and click on the down arrow in **Test suite result workspace view**.

<p align="center">
  <a href="images/l01_switch_ws_1.png" target="_blank">
    <img src="images/l01_switch_ws_1.png">
  </a>
</p>

A new window will appear with the available test suites. You can switch between them to see the difference.

<p align="center">
  <a href="images/l01_switch_ws_2.png" target="_blank">
    <img src="images/l01_switch_ws_2.png" width="50%">
  </a>
</p>

You need to click on the **Calculate** button after loading a new test suite.

<p align="center">
  <a href="images/l01_switch_ws_3.png" target="_blank">
    <img src="images/l01_switch_ws_3.png">
  </a>
</p>

### Grouping test suites

When you have more than 1 test suite, sometimes you want to group them to increase the coverage of your workspace.

!!! Example "Hands-on Time"
    Go to your workspace and click on **Workspace Test Suit Results**, select the test suites and click on the **Group** icon to group them.

<p align="center">
  <a href="images/l01_group_ws_1.png" target="_blank">
    <img src="images/l01_group_ws_1.png">
  </a>
</p>

A new window will appear, give a name to your group and click on **Group**.

<p align="center">
  <a href="images/l01_group_ws_2.png" target="_blank">
    <img src="images/l01_group_ws_2.png" width="50%">
  </a>
</p>

You can see that your runs are updated and that the coverage increased.

<p align="center">
  <a href="images/l01_group_ws_3.png" target="_blank">
    <img src="images/l01_group_ws_3.png">
  </a>
</p>

### Ungrouping test suites

It is also possible to ungroup the group that you created.

!!! Example "Hands-on Time"
    Go to your workspace and click on **Workspace Test Suit Results**, select the grouped runs and click on **Group Info** icon to ungroup them.

<p align="center">
  <a href="images/l01_ungroup_ws_1.png" target="_blank">
    <img src="images/l01_ungroup_ws_1.png">
  </a>
</p>

A new window will appear, click on the **Ungroup** button.

<p align="center">
  <a href="images/l01_ungroup_ws_2.png" target="_blank">
    <img src="images/l01_ungroup_ws_2.png" width="50%">
  </a>
</p>

### Changing the map

**Before moving on, make sure to close Foretify in your browser**.

Another way to increase coverage is to change the ODD where the tests are executed. One powerful feature of the OSC2 language is that in order to change the ODD, you only need to change one single line of code. 

With a new map, Foretify will find new concrete test cases randomly generated in locations where the map topology can support the scenario.

!!! Example "Hands-on Time"
    Open the file `$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`, where the test is defined.

    Edit the scenario in the `ts_l01_intro.osc` file and change the line where the map is defined as follows:

    ```osc linenums="8"
    extend test_config:
        set map = "$FTX_WORKSHOP/maps/cloverleaf.xodr"
    ```
    You can now rerun the test another 5 times with the following command:

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 2 --crun 5
    ```

You can now observe that the scenario is executed on another map.

!!! note

    If the map you choose for a specific scenario does not allow the execution of the scenario, the Foretellix tools indicate this as a contradiction error. For example, a cut-in scenario can not be executed on a one lane road.

!!! Example "Hands-on Time"
    Now that you've run additional tests, you can upload them to Foretify Manager to see if the coverage improved. In order to do that, you can run the following command:

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir3
    ```
    Add the new runs to the workspace that you previously created.


## Next steps

The goals of this lab were to

- Become familiar with the Cloud environment used for the workshop
- Go through some basic OSC2 code for the cut-in scenario
- Run Foretify in interactive mode and load the cut-in scenario
- Get familiar with the concept of a seed
- Get familiar with the OSC2 for a test and its main components
- Open Foretify Manager and explore the collected metrics

Next, in Lab 2, you will extend the cut-in scenario, collect more coverage metrics and introduce ways to debug the scenarios in Foretify.