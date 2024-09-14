---
chapnum: 1
---

# Lab 1: OpenScenario 2とForetellixテクノロジーの最初のステップ

## 学習目標

この実習では、以下の内容を紹介します:

- 簡単な**ASAM OpenSCENARIO 2.0 (OSC2)** テスト実装 (ASAMは自動化および計測システムの標準化協会の略称です).

!!! Note
    Foretellixのドキュメント全体で、用語「OSC2」は「ASAM OpenSCENARIO® DSL バージョン 2.x」を指します.

- **Foretify™**、**シナリオ開発およびテスト自動化プラットフォーム**、以下の目的で使用されます:
    - OSC2ソースのコンパイル
    - 抽象シナリオ定義から具体的なテストの生成
    - テスト実行中のシミュレーションプラットフォーム内のアクターの制御
    - テスト実行の結果をプロットして可視化

- **Foretify Manager**、**ビッグデータ分析プラットフォーム**、主に以下の目的で使用されます:
    - 複数のテスト実行のキーパフォーマンスインジケーター（KPI）、チェッカーメッセージ、およびカバレッジメトリクスを収集
    - テストの進捗を可視化し、Safety-Driven Verification (SDV) の方法論アプローチを実現

<p align="center">
  <a href="images/l01_ftx_diagram.png" target="_blank">
    <img src="images/l01_ftx_diagram.png">
  </a>
</p>

!!! Note
    任意の画像をクリックすると、新しいタブで高解像度で開きます.

## OSC2言語

開発と検証の両プロセスにおいて、ADおよびADAS機能については、さまざまな*シナリオ*を用いて System Under Test (SUT) を刺激する必要があります。シナリオとは、車両、歩行者、環境条件、およびSUT自体など、1つ以上のアクターによるアクションの時間的な連続です。OSC2は、アクターが環境を移動するシナリオを記述するために特に設計されたドメイン固有の言語です。これらのシナリオには属性があり、アクタータイプ、彼らの移動、および環境（シナリオがどこで発生すべきかを含むマップ上の位置など）を制約することができます。

!!! Info
    *制約ランダム*のアプローチを用いると、制約がないシナリオ属性はランダムになります。例えば、カットインシナリオの"side"属性を"right"に制約しない場合、可能な属性の空間（すなわち、"left" および "right"）からランダムに選択されます。

    さらに、値は可能な属性の空間から選択されて、シナリオ、アクター、およびマップから派生する制約を満たします。例えば、SUT (EGO) が2車線道路の最も右側のレーンを走行している場合、カットインの "side" 属性は "right" と選択されません。

OSC2の構成要素は、データ構造などです：

- **Actors**: 実世界のエンティティを表します。その名前が示す通り、シナリオで"役割を演じて"います。
- **シナリオ**または**アクション**: アクターの振る舞いを記述します。一般的に、シナリオはアクションの長いシーケンスですが、両者の間に形式上の違いはありません。両方とも*修飾子*を介して修正できます。
- **修飾子**: シナリオに制約を追加し、望ましい範囲内での実行を支援します。
- **ラベル**: スカラー、構造体、アクタータイプの名前付きデータフィールドを定義します。
- **単純な構造体**: 属性、制約などを含む基本的なエンティティです。

以上のトピックについては、Foretellixの[OSC2言語ドキュメント](../osc_lang/osclang_intro.md)や[OSCドメインモデルドキュメント](../osc_dom/oscdomain_intro.md)を参照するか、[ASAM型定義トピック](https://www.asam.net/index.php?eID=dumpFile&t=f&f=3460&token=14e7c7fab9c9b75118bb4939c725738fa0521fe9#type-definitions)、または[ASAM OSC2ウェブページ](https://www.asam.net/project-detail/asam-openscenario-v20-1/)をご覧ください。


!!! Info
    Foretellixツールは**ASAM OSC2言語をネイティブサポート**しており、その結果、多くの利点があります。その1つは、言語とツールが**実行プラットフォームに依存しない**ことです。これにより、検証環境を変更する際の**作業量が削減**され、適切な検証プラットフォームを選択する際に**柔軟性が向上**します。

    OSC2は**抽象シナリオ**と**セーフティドリブン検証**フローをサポートしています。このワークショップを通じて、これらの機能がどのようにV&V作業を**効率化**し、自律システムを検証する際に必要な**リソースを減らす**のかを理解します。

### 初めてのテスト

テストはシナリオが呼び出されるOSC2コードであり、ヒエラルキ的には最上位層と見なされます。

1. テスト実行プラットフォーム構成（例：シミュレーター）をインポートします。

2. SUT（例：テスト対象システム、通常はEGOとも呼ばれる）の構成をインポートします。この場合、テストされる機能として、Foretellixによって開発されたSUT L4スタックをインポートし、EGO車両の属性を構成します。

3. テストに使用する地図を設定します。

4. シナリオとメトリクス（チェック、カバレッジ、KPIなど）を定義し、シナリオの実行を呼び出します。

次の画像では、実行するテストの構造が確認できます：

<p align="center">
  <a href="images/Test_2.png" target="_blank">
    <img src="images/Test_2.png">
  </a>
</p>

!!! Example "ハンズオン時間"
    ワークショップを始める準備が整ったので、最初のテストを詳しく見てみる時がきました。

    Foretify Developer OSC2コードコンパイラーを使用して最初のテストを開きます：

    ```bash
    foretify --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --gui
    ```

ブラウザにForetify GUIが表示されます。このトレーニング中にさらなる機能を学びます。今のところOSC2コードのコンパイルと視覚化機能に焦点を当てます。**Source**タブをクリックして、以下の画像のように**Loaded Files**ペインを折りたたんでください：

<p align="center">
  <a href="images/l01_ftx_dev.png" target="_blank">
    <img src="images/l01_ftx_dev.png">
  </a>
</p>

テストでは、他のOSC2ファイルをロードする_import_ステートメントに注目してください：

```osc linenums="3"
import "$FTX_WORKSHOP/common/workshop_config.osc"
import "$FTX_WORKSHOP/scenarios/cut_in_l01.osc"
import "ts_l01_intro_cov.osc"
import "ts_l01_intro_checks.osc"
```
全てのコードを1つのファイルにすることもできますが、それでは読みやすさが損なわれ、再利用性が低下し、管理が難しくなります。

### ファイルのインポート文に進みます。次のセクションでは、importステートメントの内容と、コードの残りの部分を確認します。

### _workshop_config.osc_ 設定ファイル

`workshop_config.osc`ファイル（テストファイルの3行目でインポートされます）には、Foretifyの設定および実行プラットフォーム接続（異なるシミュレータを使用できます）のすべての定義が含まれています。また、System Under Test（SUT）接続（この場合、自律走行するエゴ）も含まれています。

### _cut_in_l01.osc_ シナリオファイル

`cut_in_l01.osc`ファイルは4行目でインポートされ、カットインシナリオの抽象的な定義を含んでいます。ここでは、絶対的および相対的な制約のセットを通じて、車両がSUTの前で車線変更を行うべきであると定義しています。OSC2は抽象化をサポートする唯一のシナリオ記述言語であり、シナリオ開発者がコードを書くのに費やす時間を大幅に削減します。

以下の画像では、抽象的なカットインシナリオの具体的なインスタンスを2つ見ることができます。

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_scenario.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_scenario.png">
  </a>
</p>

!!! 例 "実践時間"
    Sourceタブのテストファイルの4行目をクリックしてインポートされた`cut_in_l01.osc`ファイルを開いてください。

以下に示すコードは、カットインシナリオを実装しています:

<p align="center">
  <a href="images/workshop_l01_cut_in_code_vscode.png" target="_blank">
    <img src="images/workshop_l01_cut_in_code_vscode.png">
  </a>
</p>

コードを見ていきましょう:

- Line 6: sut.cut_in_l01というシナリオがSUTのコンテキストで宣言されています（だからsut.name_of_scenarioですね）
- Line 7: SUTではない別の車car1が"vehicle"型のオブジェクトとしてインスタンス化されます
- Line 8: car1が切り込む側が"av_side"という列挙型としてインスタンス化されます。これは"left"または"right"という値を保持できることを意味します
- Line 10: これ以降のブロックが順次実行されることを示します。この場合、log_infoの後にapproach_phaseというフェーズが実行され、それに続いてchange_laneフェーズが実行されます
- Line 12: 切り込み側をログに書き込みます。ログステートメントを使用すると、デバッグ目的で後で使用できる行を追加できます
- Line 14: 並列シナリオフェーズが作成され、"approach_phase"とラベルが付けられます。これはライン15と18が並行して実行されることを意味します（OSC2はインデントベースの言語であることを覚えておいてください）。ラベルを使用すると、コードの他の部分からその部分を参照できます
  - Line 15: SUTにdrive()アクションを開始するようトリガーがかかります。次のライン（16と17）では、SUTのdriveアクションにいくつかの制約が追加されます：
    - Line 16: approach_phaseの開始時点でSUTの速度を少なくとも30 km/hに制約します
    - Line 17: SUTにアプローチフェーズ中に車線を維持するよう制約を加えます
  - Line 18: car1にdrive()アクションを開始するようトリガーがかかります。car1はSUTではないため、運転中は完全にForetifyエンジンによって制御されます。次のライン（19から22）では、car1の移動にいくつかの制約が追加されます。これにより、切り込みマニューバが実現されるようになります：
    - Line 19: このフェーズ全体でcar1がSUTに隣接した車線に制約されます
    - Line 20: このフェーズの開始時のcar1の位置はSUTから10から20メートル前方に定義されます
    - Line 21: このフェーズの終了時のcar1の位置はSUTから10から20メートル前方に定義されます
    - Line 22: car1の位置はベストエフォート条件に定義されます。これは、制約が満たされない場合でもソルバーがこのシナリオを失敗としてラベル付けしないことを意味します。シミュレータの制限により、予定された実行が完全に展開できない場合があり、そのような実行を_incomplete scenarios_としてラベル付けされます。重要でない制約をベストエフォート制約として定義することで、そのような実行の割合を減らすのに役立ちます。この機能や「計画」と「実行時」の違いについては、上級ラボで詳しく学ぶことができます
- Line 23: "change_lane"というラベルが付けられた2番目の並列シナリオフェーズが作成されます。これはライン24と32の後続のアクションが並行して実行されることを意味します
  - Line 24: SUTにdrive()アクションを開始するようトリガーがかかり、複数の修飾子が続きます
    - Line 25: SUTのdriveアクションに制約を追加し、シナリオフェーズ全体で車線を維持するよう求めます
  - Line 26: car1にdrive()アクションを実行するようトリガーがかかり、ライン29から34で制約が追加されます
    - Line 27: フェーズの開始時にcar1の速度をSUTの速度より5〜15 km/h遅くするように制約を加えます
    - Line 28: 速度制約を非重要なベストエフォート制約にオーバーライドします
    - Line 29: フェーズの終了時にcar1がSUTと同じ車線に制約されるようにします
    - Line 30: car1の速度はシナリオフェーズ全体で一定に保たれます
    - Line 31: この速度制約も非重要であり、最善の努力で実行されるよう定義されます
    - Line 32: この行はcar1の衝突回避行動を非アクティブ化し、SUTにとって車線変更が難しくなります。汎用車両アクターの衝突回避行動については、上級ラボで詳しく学ぶことになります

### 行動モニタリング

次の3つのセクションでは、一部の例を使って、_カバレッジ_、_KPIまたはレコード_、_チェッカー_の概念を簡単に紹介します。これらは安全を重視した検証手法の重要な要素です。なぜならば、これらは検証プロセスをサポートし、システムアンダーテストの誤った動作を明らかにするからです。次回の実習では、これらの機能について深く掘り下げて説明します。

#### _ts_l01_intro_cov.osc_のカバレッジ定義

!!! 例 "実践時間"
    ソースタブの`ts_l01_intro.osc`テストファイルの5行目をクリックしてインポートされた`ts_l01_intro_cov.osc`ファイルを開いてください。

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_osc_cover.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_osc_cover.png">
  </a>
</p>

上記のコードは**カバレッジ収集**を定義しています：

- 最初のメトリック（4行目）はカットインが発生した側、左か右かです。
- 2つ目のメトリック（5行目）はカットインを行う車のタイプ、例えばトラックやセダンです。
- 次のメトリック（6から11行目）は、車がレーンチェンジシナリオフェーズの終わりにどの速度で走行しているかです。
- 最後のメトリック（12から18行目）は、メインシナリオが開始する時点で2台の車の間の縦方向の距離です。

!!!情報
    - 6行目では、OSC2の重要な構造化タイプメンバーの1つ、_event_を使用しました。このイベントは_end_であり、具体的には`$FTX_WORKSHOP/scenarios/cut_in_l01.osc`シナリオで定義されたchange_laneフェーズの終了イベントを表します。
    - イベントは一時的なオブジェクトで、時間の特定のポイントを表し、シナリオで定義されたアクションをトリガーできます。イベントは構造体内に定義することもできますが、一般的にはアクターまたはシナリオ内に定義します。

### _ts_l01_intro_cov.osc_ の KPI 定義

この KPI は、前のセクションのカバレッジ項目と同じファイル内で定義されています。

このコードのこの部分では、次のように KPI を定義します:

- まず、SUT とカットイン車との距離をサンプリングするための変数が、21行から22行で宣言されています。
- 25行から27行では、後で視覚化するための KPI を記録するために、_record()_ メソッドが使用されています。

!!! 情報
    いくつかの例を通過したので、カバレッジメトリックとパフォーマンスメトリックの主な違いを理解することが重要です：

    - **カバレッジ評価**：_どの部分の “シナリオスペース” を AV で試験しましたか？_ これはカバレッジと総合的なカバレッジ評価の形で表現されます。カバレッジ項目がカバレッジ評価をサポートするために定義されています。つまり、カバレッジ評価は次の質問に答えます：_SUT のテストはどれほど良かったですか？_
    - **パフォーマンス評価**：_テストで SUT がどれほど優れて機能しましたか？_ この質問には、1つまたは複数の KPI によって答えられます。

### _ts_l01_intro_checks.osc_ のチェッカーの定義

!!! 例 "ハンズオンタイム"
    ソースタブの`ts_l01_intro.osc` テストファイルの6行目をクリックしてインポートされた`ts_l01_intro_checks.osc` ファイルを開いてください。

<p align="center">
  <a href="images/l01_kpi_code.png" target="_blank">
    <img src="images/l01_kpi_code.png">
  </a>
</p>

このチェッカーの目的は、シナリオ全体を通じて SUT とカットイン車との距離が定義された安全距離内かどうかを評価することです。

- In line 1, the `issue_kind` type is being extended with a unique name (`safety_distance`) for the new checker.
- Lines 3 to 12 extend the previously defined scenario for writing the checker as follows:
    - Lines 5 - 6: A variable for the safety distance threshold is declared, and set to a value of 13 meters
    - Lines 8 to 10 verify at each time step (`top.clk`) of the simulation, whether:
      - the distance between the two cars is above the defined threshold
      - both vehicles are in the same lane
    - Lines 11 to 12: If the condition above is not met, then the test run is stopped with an error of the type `sut_error`, and subtype `safety_distance`.

!!! Info

    - The checker just added is a user-defined checker, but Foretify comes with built-in checkers. You can review those in the [Global checkers for vehicles documentation](../osc_dom/oscdomain_metrics.md#global-checkers-for-vehicles).

    - Another term used in the industry for checker is evaluator.

    - With checkers you can represent the success or failure criteria for scenarios, and they are written with the same language (OSC2) used to represent actions and actors.

    - Some predefined checkers are always active when using Foretify, such as the collision check or a check for the SUT driving off-road. You can always modify the defaults that apply for all scenarios.

    - You can define custom checkers to capture issues specific to your SUT, as in our example. The checker we have exercised uses the KPI value and a predefined threshold, in order to evaluate the SUT's expected behavior, but this is just one way of defining it. 

### Map definition

In the code below (lines #8 and #9 of the test file `ts_l01_intro.osc`), we set the map to be used, a map that is in OpenDrive format (i.e. `*.xodr`).

```osc linenums="8"
extend test_config:
    set map = "$FTX_WORKSHOP/maps/Town04.xodr"
```

### シナリオ実行

`ts_l01_intro.osc`ファイルの最後の2行では、ついに_cut_in_l01_シナリオを呼び出します：つねに実行されるOSC2プログラムのエントリーポイントは_top.main_シナリオで、これはC/C++の_main()_関数と同様です：

```osc linenums="11"
extend top.main:
    do cil : sut.cut_in_l01()
```
このコードは、テストファイルの11行目と12行目にあります。

## 初回実行とForetify

### 制約付きランダム生成と適応シナリオ実行

その名前が示すように、*制約付きランダム生成*とは、指定された制約内のランダム変数を生成することを意味します。これはSDV（Safety-Driven Verification）フローの基本的な前提条件であり、Foretifyが構築されている中心的な原則です。

OSC2の抽象的なシナリオから、Foretifyの生成エンジンは指定された制約に基づいて具体的なシナリオをランダムに生成します。ランダム化される最も重要な側面の1つは、各シナリオが展開される地図の領域です。

シナリオの計画が立てられたら、ランタイム適応型シナリオ実行エンジンがシナリオプランに従って実行を処理します。

!!! Info
    **制約付きランダム生成**エンジンはForetellixソリューションの主要な柱です。これは、抽象的な記述から意味のあるシナリオの変化を数百万生成する非常に強力なツールです。

    このテクノロジーを活用すると、シナリオ作成チームに必要な**エンジニアリングリソース**が**著しく削減**されるため、抽象的な定義から多くの意味のあるバリエーションが生成されます。

生成されたシナリオは実行されると、システムをテストする際に**バグを見つけて解決するのに効率的な方法**を提供します。


### 最初の実行

**次に進む前に、ブラウザでForetify GUIを閉じていることを確認してください。**

OSC2シナリオ定義を確認した後、シミュレータをテスト実行プラットフォームとして使用します。

次に、Foretify GUIを詳細に探索し、テストの読み込み、開始、および分析を行います。後に、Foretifyの機能をさらに探索します。

!!! 例 "実地体験"
    ForetifyをGUIモードで起動し、以前に調査したテストを読み込む：

    ```bash
    foretify --gui --work_dir $FTX_FM_WORKDIR/l01_intro/workdir \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc
    ```

Foretifyの対話型ウィンドウが表示されます：

<p align="center">
  <a href="images/foretify_gui.png" target="_blank">
    <img src="images/foretify_gui.png">
  </a>
</p>

Foretifyウィンドウには以下の情報が表示されます：

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
    テストを実行する前に、VisualizerでSUTとcar1の計画された経路を表示する**プレビュー**ボタンで実験できます。計画された経路は、SUTの挙動によって実行時に変更される場合がありますが、テストのデバッグに非常に役立つツールです。計画された経路はそのシナリオのために作成された計画の表現に過ぎないことに注意してください。実際の軌跡はランタイム中に計算および更新されます。

!!! Example "Hands-on Time"
    実行前に、グレーの**プレビュー**ボタンをクリックしていくつかのテストシードをプレビューしてみてください。右上隅にあります。また、シード番号を変更することもできます。

!!! Example "Hands-on Time"
    右上隅にある紫の**実行**ボタンをクリックしてテストを実行してください。また、右上隅にある**ターミナル**ボタンをクリックして_run_と入力することでもテストを実行できます。

シミュレーターウィンドウが表示されます:

- Foretifyの制約ランダム生成エンジンは、シナリオの特定のバリアントを生成しました。これは、シナリオ用のOSC2コードで指定された制約とパラメータに基づいています。アクターの計画された経路は、プレビュー時に見たものです。

- Foretifyのランタイムテストオーケストレーションエンジンは、テスト実行プラットフォーム（シミュレーター）内のアクターが指定された制約に従って移動できるようにしました。

ランタイムテストオーケストレーションエンジンは、**適応型シナリオ実行**を確実にします。つまり、SUTの計画された経路からの逸脱は、OSC2ファイルで指定されたシナリオの意図が達成されるようにNPCから対応策が取られます。

テストが完了すると、ログエリアに2つの追加タブが表示されます:

- **トレース詳細**タブを使用すると、シミュレーション中に収集された値とともにシナリオトレース情報を検査できます。トレースの種類、アクター、時間、および期間などが含まれます。
- **ログ**タブには、テスト実行中に生成されたログが含まれています。
- **メトリックス**タブは、カバレッジに関連しており、ワークショップで後日詳細に説明します。

!!! 例 "実践時間"
    今の段階では、**ログ**タブを確認し、カットインがどちらの側で発生するかを示すために追加したログメッセージが表示されているかどうかを確認してください。

### ランのデバッグ

#### Foretify Visualizerを使用したデバッグ

テストを実行した後、画面の中央にある**Visualizer**タブにアクセスできます。Visualizerは、実行を分析するのに役立つさまざまな方法で構成できるグラフィカルな事後処理ツールの1つです。実行を視覚化することは、実行を繰り返すこととは異なります。シミュレータを必要とせず、再実行が消費する計算リソースを必要としません。

実行が完了すると、画面の左側に**Visualizer**タブが自動的に開きます。
いつでもタブをクリックしてVisualizerに戻ることができます。

<p align="center">
  <a href="images/visualizer.png" target="_blank">
    <img src="images/visualizer.png">
  </a>
</p>

##### テストの再生

まず、Visualizerタイムラインの左下にある再生ボタンを押して、シナリオが再生される様子を確認できます。

<p align="center">
  <a href="images/visualizer_play_button.png" target="_blank">
    <img src="images/visualizer_play_button.png">
  </a>
</p>

##### 地図パースペクティブ
Visualizer内で右マウスボタンをクリックし、カーソルを異なる方向にドラッグすることでパースペクティブを変更できます。Visualizerの右上にあるレンチアイコンをクリックして**ビューツール**にアクセスできます。**ビューツール**にはビューを制御するオプションがあります：

<p align="center">
  <a href="images/visualizer_tools.png" target="_blank">
    <img src="images/visualizer_tools.png">
  </a>
</p>

- レーン方向: 運転方向の矢印の表示を有効または無効にします。
- 信号: 信号の表示を切り替えます。
- 速度制限: 道路の速度制限を表示します。
- 衝突回避: 衝突回避の有効性を表示します。
- ランタイム軌跡: 選択した車両の軌跡を表示します。
- 予定された経路: シナリオに存在する車両の経路を強調表示します。
- 予定された目的: 選択した車両に生成された予定目的を表示します。
- 予定されたポーズ: SUTと他の車両の次のポーズを強調表示します。
- ドライバー目的: 選択した車両に生成された運転目的を表示します。
- 予測ポーズ: 車両の予測位置を表示します。

##### カメラ設定

パースペクティブおよびカメラを制御するには、カメラ設定（カメラ）アイコンをクリックしてください。カメラで特定のアクターを追跡するには、左上隅の（i）ドロップダウンリストからアクターを選択するか、Visualizer内でクリックしてカメラを固定ポジションに設定します。

<p align="center">
  <a href="images/Camera_settings.png" target="_blank">
    <img src="images/Camera_settings.png">
  </a>
</p>

- パースペクティブビュー: パースペクティブを上から見たビューに変更します。
- 選択したアクターにフォロー: 選択したアクターを追跡します。
- 選択したアクターにカメラをリセット: カメラを選択したアクターの位置にリセットします。

##### 距離の計測

- もしVisualizerがパースペクティブビューにある場合は、Visualizerの右上にあるカメラ設定アイコンを選択し、パースペクティブビューオプションをオフにします。
<p align="center">
  <a href="images/Measurement_1.png" target="_blank">
    <img src="images/Measurement_1.png">
  </a>
</p>

- 「距離測定ツール」アイコンを選択します。
<p align="center">
  <a href="images/Measurement_2.png" target="_blank">
    <img src="images/Measurement_2.png">
  </a>
</p>

- 測定を開始する点を設定するためにVisualizerでクリックし、次にカーソルを移動させて測定の終点を設定するためにクリックします。
<p align="center">
  <a href="images/Measurement_3.png" target="_blank">
    <img src="images/Measurement_3.png">
  </a>
</p>

測定は、開始点と終点をつなぐ線の隣に表示されます。

- 測定を非表示にするには、距離測定ツールアイコンをオフに切り替えます。

#### トレースを使用したデバッグ

すべてのトレースはトレースビューの下で表示できます。トレースビューはタイムラインに整列しているため、異なるトレースを簡単に比較することができます。

**トレース**は次の異なるタイプとして表現されます：

- **インターバル**: 一定期間にわたって収集された値のコレクションを表します。インターバルには、名前、開始/終了時刻、種類があり、特定のアクター（オレンジのボックス）と関連付けられています。

- **値**: 時間経過とともに変化する単一の値を表します。値は波形グラフとして表示され、値トレースには名前、値、単位があり、特定のアクター（赤いボックス）と関連付けられています。

<p align="center">
  <a href="images/Traces_1.png" target="_blank">
    <img src="images/Traces_1.png">
  </a>
</p>

視覚化を向上させるために、Foretifyはすべてのシナリオの開始時刻と終了時刻を記録します。

##### インターバルを表示するには：
- Foretifyの「Debug Run」タブを選択し、「Visualizer」の下にある「Traces」タブをクリックします。

```markdown
<p align="center">
  <a href="images/Traces_2.png" target="_blank">
    <img src="images/Traces_2.png">
  </a>
</p>

トレースは、現在の時刻カーソルを持つタイムライン上のインターバルとして表示され、VisualizerやTracesタブ内のアクターの値トレースなどの他の時間ベースのビューと対応しています。

1. トレース名の左側の矢印をクリックして展開し、その子シナリオを表示します。

2. トレースをクリックして、トレースの詳細（トレースタイプ、アクター、時間、期間、およびインターバル中に収集されたメトリクスなど）を表示します。

3. トレースの詳細の下で、トレースの開始時刻または終了時刻をクリックして、その時間にユニバーサルタイムラインを設定します。

##### タイムラインをトレースに合わせるには：
1. Tracesタブで、タイムラインにフレームしたいインターバル（オレンジボックス）を選択します。

2. フレームタイムラインアイコン（赤いボックス）をクリックします。

<p align="center">
  <a href="images/Intervals_3.png" target="_blank">
    <img src="images/Intervals_3.png">
  </a>
</p>

- タイムラインをインターバルにフレームしないようにリセットするには、タイムラインの右側にあるアンフレームタイムラインアイコンをクリックします。

<p align="center">
  <a href="images/Intervals_4.png" target="_blank">
    <img src="images/Intervals_4.png">
  </a>
</p>

!!! Example "Hands-on Time"
    Visualizerを使用してテストをリプレイし、SUTの動作や上記で議論した異なるオプションを調べます。

### 異なるシードで実行

!!! Example "Hands-on Time"
    右上の実行制御エリアを使用して、シード番号を4に設定して**Run Test**ボタンをクリックして別のシミュレーションを実行します。これは、最小距離閾値に導入されたチェッカーが失敗する1つのシードです。

    お好きなシードで別のシミュレーションを実行します。

    再度、カットインがどちら側で起きたかを示すメッセージを検索します。
```

```markdown
## Foretify Manager

そろそろForetifyを閉じることができます。Foretify端末でexitを入力するか、Foretifyウィンドウを閉じることで閉じることができます。

!!! Info
    **Seed**は、具象的な実行のランダム生成のための入力として機能します。抽象的なシナリオ定義から具体的なテストを生成する際、同じシードを使用すると**同じ具体的なバリエーション**が生じます。これは、テストを完全にランダム化する一方で、デバッグ目的で単一の具体的な実行を再現することができる重要な機能です。

    シードの定義と実装により、いつでも生成された特定の具体的なバリエーションを**追跡**することができます。シナリオのトレーサビリティは、バグが発生した条件を特定し再珺することができるため、重要な機能です。

Foretify Managerを実行したら、達成したカバレッジと結果を見ることができます。カバレッジはLab 2で正式に定義しますが、今はForetellixツールを使用してこのコンセプトを視覚的に探索できます。

Foretify Managerは、検証プロセス全体の状態を可視化し、多くのテスト実行の結果やKPIをインポートするForetellixツールです。以下のイメージに示すように、クライアントサーバーアーキテクチャを持っています:

<p align="center">
  <a href="images/fmanager_architecture.png" target="_blank">
    <img src="images/fmanager_architecture.png">
  </a>
</p>

クライアントはPythonスクリプトまたはWeb UI（Webページ）のいずれかであり、どちらもテストスイートの結果データに対して操作やクエリを実行できます。サーバーはデータベースを管理し、クライアントのコマンドを実行します。このトポロジーにより、複数のユーザーが同時に検証結果の異なる側面を分析できます。

###  Foretify Managerを開く

Foretify Managerはブラウザベースのアプリケーションです。
```

!!! Example "Hands-on Time"
    ターミナルから以下のコマンドを呼び出すことでForetify Managerを起動します:

    ```bash
    fmanager
    ```

    最初に表示されるのはログインページです:

    <img src="images/fmanager_login.png" alt="image" width="200"/>

!!! Info
    担当者に連絡して、アサインされたユーザー名とパスワードについて詳細を確認してください。



### プロジェクトの作成

Foretify Managerのプロジェクトは、検証データや収集されたメトリクスに対して権限と所有権を設定するための協力フレームワークです。

プロジェクトを作成するには:

1. Foretify Managerを開いて、Foretify Managerの資格情報でログインします。

2. プロジェクトを作成するために、「新規プロジェクトを作成」をクリックします。

!!! Info
    プロジェクト名に関する詳細情報は、担当者にお問い合わせください。

- 紫の"Create"ボタンをクリックします 

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

### テストスイートの結果をアップロードする

この時点で、Foretify Managerデータベースは空なので、次のステップは実行したテストの結果をアップロードすることです。

!!! Example "Hands-on Time"
    ターミナルに移動して以下のコマンドを呼び出して結果をアップロードします:

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir
    ```

``` --run_group_name ``` パラメータを使用して、テストグループに特定の名前を付けることができます。[upload_runsのドキュメント](../fman_user/fmanuser_launch_test_suite.md#upload-a-regression)を参照してください。

!!! Example "Hands-on Time"
    ブラウザで開いたForetify Managerウェブアプリに切り替えて、リフレッシュボタンをクリックしてください（「Test Suite Results」タブにいることを確認してください）

アップロード後、テストの実行が読み込まれ、以前に作成したプロジェクトに表示されます：

<p align="center">
  <a href="images/fmanager_regression_2.png" target="_blank">
    <img src="images/fmanager_regression_2.png">
  </a>
</p>

### アップロードされた実行の分析

今インポートしたテストスイートをクリックすれば、個々の実行を確認できます。

以下の画像のようなものが表示されるはずです：

<p align="center">
  <a href="images/fmanager_runs.png" target="_blank">
    <img src="images/fmanager_runs.png">
  </a>
</p>

色付きの四角が示すものは次の通りです：

- _yellow_: 実行リストの右上にあるアイコンで実行のエクスポートや削除、選択の保持やリセットができます。右端の列選択アイコンを使用して、実行属性（ディレクトリ、OSユーザ、実行時間など）を追加または削除できます。
- _orange_: _Issues Tree_ は、種類ごとにグループ化されたすべての問題を表示します。
- _blue_: _Aggregation View_ では、実行属性に基づいて実行を集計できます。

_実行_ビューで実行をクリックすると、新しいForetify Managerウィンドウが表示されます。

<p align="center">
  <a href="images/Run_view_2.png" target="_blank">
    <img src="images/Run_view_2.png">
  </a>
</p>

ご覧のように、メインタブが2つあります：**Debug Run** と **Run Summary**.
**Debug Run** タブは、Foretifyで以前に見たものと全く同じです。

<p align="center">
  <a href="images/run_source.png" target="_blank">
    <img src="images/run_source.png">
  </a>
</p>

**Run Summary** タブをクリックすると、失敗した実行に関する詳細情報を確認できます。

```markdown
<p align="center">
  <a href="images/run_summary.png" target="_blank">
    <img src="images/run_summary.png">
  </a>
</p>

!!! 例 "実践タイム"
    各ランをクリックして、他の2つのランを今すぐご覧ください。

### ワークスペースとは何か、そしてそれを作成する方法

ワークスペースとは、カバレッジデータを分析したいインポートされたテストスイートのセットです。

!!! 例 "実践タイム"
    **テストスィートの結果** タブでテストスイートを選択し、次に以下に示すように **ワークスペースの作成** をクリックします。

<p align="center">
  <a href="images/fmanager_workspace_2.png" target="_blank">
    <img src="images/fmanager_workspace_2.png">
  </a>
</p>

ワークスペース名を選択します。その後、 **ワークスペースの作成** をクリックして、ワークスペースを作成します。

<p align="center">
  <a href="images/fmanager_workspace_name_2.png" target="_blank">
    <img src="images/fmanager_workspace_name_2.png">
  </a>
</p>

Foretify Manager のウェブアプリケーションは、**現在のワークスペース** ビューに切り替わります：

<p align="center">
  <a href="images/fmanager_workspace_after_creation.png" target="_blank">
    <img src="images/fmanager_workspace_after_creation.png">
  </a>
</p>


ワークスペースには、以下が含まれます：

- **VGrade** は総合メトリクスグレードです（ラボ4で学習します）。
- **Total Runs**（**VGrade** の隣）は、パスしたランと失敗したランの統計です。
- **VPlan** タブ（青）では、メトリクス階層を表示できます。
- **Runs** タブ（緑）では、現在のワークスペースで選択されたランのリストを見ることができます。

!!! 例 "実践タイム"
    **VPlan** ツリーと **Runs** タブ内のランを探索してください。

### メトリクスとチェッカーの表現
```

#### カバレッジ
 当社の `ts_l01_intro_cov.osc` カバレッジファイルには、4つのカバレッジアイテムと1つのKPIメトリックが定義されています。
 例えば、`cut_in_side` はカバレッジグレードが100％であり、一方で `speed_sut` は20％しかカバレッジグレードがありません。これは、`speed_sut` のカバレッジの空きバケツを埋めるためにさらにテストを実行する必要があることを示しています。
 当社のファイルでは、これら2つのカバレッジアイテムがわずかに異なる方法で定義されていました：

 - 注意：使用されるシードによってパーセンテージが異なる場合があります！

<p align="center">
  <a href="images/lab01_coverage_conclusions.png" target="_blank">
    <img src="images/lab01_coverage_conclusions.png">
  </a>
</p>

`speed_sut` では、バケットを指定しましたが、`cut_in_side` ではどんなルールも課しませんでした。これにより、`cut_in_side` のカバレッジアイテムには余分な自由度が与えられました。このアイテムは、テスト中に可能な両側（左および右）が少なくとも1回はヒットされたため、カバレッジグレードが100％です。

!!! 例 "実習時間"
    今、foretifyマネージャーで他のカバレッジアイテムとそのグレードを検査して、VPlanタブの下に_cut_in_side_カバレッジアイテムを見つけるはずです。

#### KPIs
 記憶に新しいかと思いますが、以前に `distance_kpi` KPI を定義し、それはSUTと切り込み車両との距離を、レーン変更の終了イベント時に測定します。イベントを指定することで、最も関心を持つアイテムの値をよりよく捉えることができます。また、イベントが指定されているため、このKPIについては実行ごとに単一の値があることに気づきます。

<p align="center">
  <a href="images/workspace_kpi_2.png" target="_blank">
    <img src="images/workspace_kpi_2.png">
  </a>
</p>

!!! 例 "実習時間"
    どのKPI値を取得しましたか？Visualizerでシミュレーションを確認し、その値とシミュレーションの関連性を見てください。

#### チェッカー

**Runs**タブを見ると、合格した実行数と失敗した実行数がわかります。失敗した実行はコードで定義されたチェッカーから来ています。チェッカーはSUTがブール条件を満たさなかった場合に失敗応答を定義するため、これはIssues Treeに即座に反映されます。

<p align="center">
  <a href="images/checkers.png" target="_blank">
    <img src="images/checkers.png">
  </a>
</p>

!!! 例 "実践時間"
    実行を分析してください。何件の失敗した実行がありましたか？

### カバレッジを拡大

わずかなテストでは、意味のあるカバレッジを達成するには十分ではありません。Foretellixテクノロジーの強みは、大量の自動生成テストを実行することにあります。したがって、このセクションでは、カバレッジを増やすためにさらに多くのテストを実行します。これを行うには、シナリオから異なるテストケースを作成する必要があります。前述のように、これは作成時に使用されるシードによって制御されます。

**次に進む前に、ブラウザでForetify GUIを閉じてください**。

!!! 例 "実践時間"
    さらに15件のテストを実行し、新しい作業ディレクトリを設定します。Foretify GUIを使用しないことに注意してください。そのため、シミュレータウィンドウが定期的に表示されることになります。

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir2 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 20 --crun 15
    ```

    ```--crun```オプションは、シード20から（つまりシード20から34まで）15回のテストを実行します。 
    これはテストをスケーリングアップするための1つのオプションですが、Lab 3ではさらに多くのオプションをカバーします。

### 日本語への翻訳

ジェネレーションエンジンが15個のユニークな具体的なテストを生成し、それぞれが地図の異なるセクションに配置されることに注目してください。すべての具体的なテストを抽象シナリオで定義された境界内に保持します（例：速度、車線位置など）。

!!! 例 「ハンズオンタイム」
    新しいランではカバレッジが向上するはずです。次のコマンドを使用して、これらの追加の15つのランをForetify Managerにアップロードできます。

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir2
    ```

    次に、Foretify Managerの**テストスイート結果**タブで、新しくアップロードしたテストスイートを選択し、以下の画像に示すようにワークスペースに追加します。

    今、**VPlan**ツリーを探索し、カバレッジがどれだけ向上したかを確認できます。

<p align="center">
  <a href="images/l01_add_ws_1.png" target="_blank">
    <img src="images/l01_add_ws_1.png">
  </a>
</p>

### テストスイート間の切り替え

新しいテストスイートをワークスペースに追加すると、それらのテストスイート間を切り替えることができます。これにより、それぞれを個別に分析できます。

!!! 例 「ハンズオンタイム」
    ワークスペースに移動し、**テストスイート結果ワークスペースビュー**で下矢印をクリックします。

<p align="center">
  <a href="images/l01_switch_ws_1.png" target="_blank">
    <img src="images/l01_switch_ws_1.png">
  </a>
</p>

利用可能なテストスイートが表示される新しいウィンドウが表示されます。これらを切り替えて違いを確認できます。

<p align="center">
  <a href="images/l01_switch_ws_2.png" target="_blank">
    <img src="images/l01_switch_ws_2.png" width="50%">
  </a>
</p>

新しいテストスイートを読み込んだ後に**計算**ボタンをクリックする必要があります。

```html
<p align="center">
  <a href="images/l01_switch_ws_3.png" target="_blank">
    <img src="images/l01_switch_ws_3.png">
  </a>
</p>

### テストスイートのグループ化

複数のテストスイートを持っていると、ワークスペースのカバレッジを増やすためにそれらをグループ化したくなることがあります。

!!! 例 "実践時間"
    ワークスペースに移動して、**ワークスペースのテストスイート結果**をクリックし、テストスイートを選択して**グループ**アイコンをクリックしてグループ化します。

<p align="center">
  <a href="images/l01_group_ws_1.png" target="_blank">
    <img src="images/l01_group_ws_1.png">
  </a>
</p>

新しいウィンドウが表示され、グループに名前を付けて**グループ**をクリックします。

<p align="center">
  <a href="images/l01_group_ws_2.png" target="_blank">
    <img src="images/l01_group_ws_2.png" width="50%">
  </a>
</p>

実行が更新され、カバレッジが増加したことがわかります。

<p align="center">
  <a href="images/l01_group_ws_3.png" target="_blank">
    <img src="images/l01_group_ws_3.png">
  </a>
</p>

### テストスイートのグループ解除

作成したグループを解除することもできます。

!!! 例 "実践時間"
    ワークスペースに移動して、**ワークスペースのテストスイート結果**をクリックし、グループ化された実行を選択して**グループ情報**アイコンをクリックして解除します。

<p align="center">
  <a href="images/l01_ungroup_ws_1.png" target="_blank">
    <img src="images/l01_ungroup_ws_1.png">
  </a>
</p>

新しいウィンドウが表示され、**解除**ボタンをクリックします。

<p align="center">
  <a href="images/l01_ungroup_ws_2.png" target="_blank">
    <img src="images/l01_ungroup_ws_2.png" width="50%">
  </a>
</p>

### マップの変更

**次に進む前に、ブラウザで Foretify を閉じてください**。
```

別の方法でカバレッジを増やすには、テストが実行されるODDを変更することです。OSC2言語の強力な機能の1つは、ODDを変更するにはコードの1行だけを変更すればよいということです。

新しいマップを使用すると、Foretifyは、マップのトポロジがシナリオをサポートできる場所にランダムに生成された具体的なテストケースを見つけます。

!!! 例 "実践時間"
    `ts_l01_intro.osc` ファイルを開きます。ここにはテストが定義されています。

    `ts_l01_intro.osc` ファイル内のシナリオを編集し、マップが定義されている行を次のように変更します:

    ```osc linenums="8"
    extend test_config:
        set map = "$FTX_WORKSHOP/maps/cloverleaf.xodr"
    ```
    このコマンドを使用して、テストをさらに5回実行できます:

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 2 --crun 5
    ```

これにより、別のマップでシナリオが実行されることがわかります。

!!! 注意

    特定のシナリオに選択したマップがそのシナリオの実行を許可しない場合、Foretellixツールはこれを矛盾エラーとして示します。たとえば、1車線の道路ではカットインシナリオを実行できません。

!!! 例 "実践時間"
    追加のテストを実行したので、カバレッジが改善されたかどうかをForetify Managerにアップロードして確認できます。これを行うには、次のコマンドを実行できます:

    ```bash
    upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
    --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir3
    ```
    以前に作成したワークスペースに新しい実行を追加します。


## 次のステップ

このラボの目標は以下の通りでした:

- ワークショップで使用するクラウド環境に慣れる
- カットインシナリオ用のいくつかの基本OSC2コードを実行
- Foretifyをインタラクティブモードで実行し、カットインシナリオを読み込む
- シードの概念に慣れる
- テスト用のOSC2とその主要部品に慣れる
- Foretify Managerを開いて収集したメトリクスを調査する

次に、Lab 2では、カットインシナリオを拡張し、より多くのカバレッジメトリクスを収集し、Foretifyでシナリオのデバッグ方法を紹介します。

> この投稿は ChatGPT を使用して翻訳されています。何か抜けている部分があれば、[**フィードバック**](https://github.com/linyuxuanlin/Wiki_MkDocs/issues/new) をお願いします。