---
chapnum: 1
---

# Lab 1: OpenScenario 2とForetellixテクノロジーの最初のステップ

## 学習目標

この実験では、以下について紹介します:

- 簡単な**ASAM OpenSCENARIO 2.0 (OSC2)** テストの実装。（ASAMは自動化および計測システムの標準化協会です。）

!!! Note
 Foretellixドキュメント全体で、「OSC2」という用語は「ASAM OpenSCENARIO® DSL バージョン2.x」を指します。

- **Foretify™**, **シナリオ開発およびテスト自動化プラットフォーム**:
 - OSC2ソースのコンパイル
 - 抽象的なシナリオ定義から具体的なテストの生成
 - テスト実行中にシミュレーションプラットフォーム内のアクターを制御
 - テスト実行結果をプロットし可視化する

- **Foretify Manager**, **ビッグデータ分析プラットフォーム**:
 - 複数回のテスト実行時にキーコンフィグレーションインディケータ（KPI）、チェッカーメッセージ、カバレッジメトリクスを収集する
 - テスト進捗を可視化し、セーフティ・ドリブン・検証（SDV）方法論アプローチを可能にする

<p align="center">
 <a href="images/l01_ftx_diagram.png" target="_blank">
 <img src="images/l01_ftx_diagram.png">
 </a>
</p>

!!! Note
 画像をクリックすると新しいタブで全解像度で表示されます。

## OSC2言語

During both development and verification processes of AD and ADAS functions, it is necessary to stimulate the System Under Test (SUT) with various *scenarios*. A scenario is a timed sequence of actions by one or more actors, such as cars, pedestrians, environmental conditions and the SUT itself. OSC2 is a domain-specific language, specifically designed for describing scenarios where actors move through an environment. These scenarios have attributes that allow you to constrain the actor types, their movements, and the environment (including the location on the map where the scenario should take place).

!!! Info 
    With a *constrained random* approach, every scenario attribute that is not constrained is randomized. As an example, if you do not constrain a cut-in scenario "side" attribute to be "right", it will be randomly chosen from the space of possible attributes (namely, "left" and "right").

    Additionally, the values are chosen from the space of possible attributes so that they satisfy the constraints, deriving from the scenario, the actors, and the map. As an example, if the SUT (EGO) is driving on the outmost right lane of a 2 lanes road, the cut-in "side" attribute will not be chosen to be "right".




- **Actors**: 実世界のエンティティを表すもの。その名前が示す通り、シナリオで役割を演じている。
- **シナリオ**または**アクション**: アクターの振る舞いを記述する。一般的に、シナリオはアクションの長い連続だが、形式上の違いはない。どちらも*修飾子*を介して変更できる。
- **修飾子**: シナリオに制約を追加し、望ましい境界内で実行を制御するのに役立つ。
- **ラベル**: スカラー型や構造体型、アクター型の名前付きデータフィールドを定義する。
- **単純な構造体**: 属性や制約などを含む基本的なエンティティ。

上記のトピックについて詳しく知りたい場合は、Foretellix [OSC2言語ドキュメント](../osc_lang/osclang_intro.md) と [OSCドメインモデルドキュメント](../osc_dom/oscdomain_intro.md) を参照するか、[ASAM Type Definitionsトピック](https://www.asam.net/index.php?eID=dumpFile&t=f&f=3460&token=14e7c7fab9c9b75118bb4939c725738fa0521fe9#type-definitions) を訪れたりしてください。または[ASAM OSC2ウェブページ](https://www.asam.net/project-detail/asam-openscenario-v20-1/)でも確認できます。

!!! Info 
 Foretellix ツールは**ASAM OSC2言語をネイティブサポート**しており、その結果多くの利点があります。そのうちの1つが、「言語とツールが**実行プラットフォーム不可知**」であることです。これにより検証環境変更時にかかる労力が減少し、「適切な検証プラットフォーム選択時」に「柔軟性が向上」します。

OSC2では、「抽象的なシナリオ」と「安全駆動検証」フローをサポートしています。「このワークショップではこれら特長がV&V作業効率化」という点から「自律システム検証へ投入される資源数削減」へつながっています。

### 最初のテスト

テストとは「OSC2コードから呼び出されるシナリオ」であり、“階層的” トップレイヤーと見做されています:

1. テスト実行プラットフォームの構成（例：シミュレーター）をインポートします。

2. SUT（システムテスト対象、通常EGOとしても知られています）の構成をインポートします。この場合、テストされる機能として、Foretellixによって開発されたSUT L4スタックをインポートし、EGO車両の属性を構成しています。

3. テストで使用する地図を設定します。

4. シナリオとメトリクス（チェック、カバレッジ、KPI）を定義し、シナリオの実行を呼び出します。

次の画像では、実行するテストの構造が見えます：

&lt;p align="center">
  &lt;a href="images/Test_2.png" target="_blank">
    &lt;img src="images/Test_2.png">
  &lt;/a>
&lt;/p>

!!! 例 "実践時間"
    ワークショップを開始する準備が整ったら、最初のテストを詳しく見てみる時がきました。

    Foretify Developer OSC2コードコンパイラーで最初のテストを開いてください：

    ```bash
    foretify --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --gui
    ```

Foretify GUIがブラウザに表示されます。このトレ

以下のセクションでは、インポート文の内容とコードの残りの部分について説明します。

### _workshop_config.osc_ 設定ファイル

`workshop_config.osc` ファイル（テストファイルの3行目でインポートされています）には、Foretifyと実行プラットフォーム接続の設定（異なるシミュレータを使用できます）およびシステムアンダーテスト（SUT）接続（この場合、自律走行のEgo）のすべての定義が含まれています。

### _cut_in_l01.osc_ シナリオファイル

`cut_in_l01.osc` ファイルは4行目でインポートされ、このラボの対象であるカットインシナリオの抽象的な定義を含んでいます。絶対的および相対的な制約のセットを通じて、車両がSUTの前で車線変更を行うという定義をしています。OSC2は抽象化をサポートする唯一のシナリオ記述言語であり、シナリオ開発者がコードの記述に費やす時間を大幅に削減します。
以下の画像では、抽象的なカットインシナリオの具体的なインスタンスが2つ表示されています。

<p align="center">
  <a href="images/workshop_lab_1_cut_in_lab_1_scenario.png" target="_blank">
    <img src="images/workshop_lab_1_cut_in_lab_1_scenario.png">
  </a>
</p>

!!! 例 "実践時間"
    ソースタブのテストファイルの4行目をクリックしてインポートされる `cut_in_l01.osc` ファイルを開いてください。

以下に示すコードは、カットインシナリオを実装しています。

<p align="center">
  <a href="images/workshop_l01_cut_in_code_vscode.png" target="_blank">
    <img src="images/workshop_l01_cut_in_code_vscode.png">
  </a>
</p>

コードの内容を確認しましょう。

- Line 6: sut.cut_in_l01というシナリオが宣言されています。これはSUTの文脈で宣言されているため、sut.name_of_scenarioと呼ばれています。
- Line 7: car1は、"vehicle"型のオブジェクトとしてインスタンス化されます。これはSUTではないもう一つの車です。
- Line 8: car1がカットインする側は、列挙型の"av_side"としてインスタンス化されます。これは、"left"または"right"の値を保持できることを意味します。
- Line 10: 以降のブロックが順番に実行されることを示しています。この場合、log_infoの後にapproach_phaseというフェーズが実行され、その後にchange_laneフェーズが実行されます。
- Line 12: カットイン側をログに書き込みます。ログステートメントを使用すると、後でデバッグ目的で使用できる行を追加できます。
- Line 14: "approach_phase"というラベルが付けられた並列シナリオフェーズが作成されます。これは、行15と18が並列に実行されることを意味します（OSC2はインデントベースの言語であることを覚えておいてください）。ラベルを使用すると、コードの他の領域からその部分を参照できます。
  - Line 15: SUTにdrive()アクションを開始するようにトリガーをかけます。次の行（16と17）では、SUTのドライブアクションにいくつかの制約が追加されています。
    - Line 16: アプローチフェーズの開始時にSUTの速度が少なくとも30 km/hであるように制約をかけます。
    - Line 17: SUTがアプローチフェーズを通過する間、車線を維持するように制約をかけます。
  - Line 18: car1にdrive()アクションを開始するようにトリガーをかけます。car1はSUTではないため、Foretifyエンジンによって完全に制御されます。次の行（19から22行目）では、car1の移動にいくつかの制約が追加され、カットインマネuーバーにつながるようになります。
    - Line 19: このフェーズ全体で、car1はSUTに隣接するレーンにいるように制約をかけます。
    - Line 20: このフェーズの開始時に、car1のSUTに対する相対位置が定義されます（SUTの10〜20メートル先）。
    - Line 21: このフェーズの終了時に、car1のSUTに対する相対位置が定義されます（SUTの10〜20メートル先）。
    - Line 22: car1のSUTに対する相対位置は、ベストエフォート条件として定義されます。これは、制約を満たすことができない場合でも、ソルバーがこのシナリオを失敗としてラベル付けしないことを意味します。シミュレータの制限により、計画された実行が完全に展開できない場合があり、_incomplete scenarios_としてラベル付けされる場合があります。非重要制約をベストエフォート制約として定義することで、そのような実行の割合を減らすことができます。詳細については、高度なラボで「計画」と「ランタイム」の違いについて学びます。
- Line 23: "change_lane"というラベルが付けられた2番目の並列シナリオフェーズが作成されます。これは、行24と32の後続アクションが並列に実行されることを意味します。
  - Line 24: SUTにdrive()アクションを開始するようにトリガーをかけ、その後にいくつかの修飾子が続きます。
    - Line 25: SUTのドライブアクションに制約を追加し、このシナリオフェーズ全体で車線を維持するようにします。
  - Line 26: car1にdrive()アクションを実行するようにトリガーをかけ、行29から34で制約をかけます。
    - Line 27: このフェーズの開始時に、car1の速度がSUTの速度よりも5〜15 km/h遅くなるように制約をかけます。
    - Line 28: 速度制約を非重要なベストエフォート制約にオーバーライドします。
    - Line 29: このシナリオフェーズの終わりに、car1がSUTと同じレーンにいるように制約をかけます。
    - Line 30: car1の速度は、シナリオフェーズ全体で一定に保たれます。
    - Line 31: 再び、この速度制約は非重要であり、ベストエフォートで実行されるように定義されています。
    - Line 32: この行は、car1の衝突回避行動を無効にし、SUTにとってレーンチェンジが難しくなります。詳細については、高度なラボで一般的な車両アクターの衝突回避行動について学びます。

### 行動モニタリング

以下の3つのセクションでは、いくつかの例を交えながら、_カバレッジ_、_KPIまたはレコード_、そして _チェッカー_ の概念について簡単に紹介します。これらは安全志向の検証方法論の重要な要素であり、検証プロセスをサポートし、SUTの誤ったパフォーマンスを明らかにします。次回の実験では、それぞれの機能について詳しく説明していきます。

#### _ts_l01_intro_cov.osc_ カバレッジ定義

!!! 例 "実践時間"
 `ts_l01_intro.osc` テストファイル内で5行目をクリックして `ts_l01_intro_cov.osc` ファイルを開いてください。

<p align="center">
 <a href="images/workshop_lab_1_cut_in_lab_1_osc_cover.png" target="_blank">
 <img src="images/workshop_lab_1_cut_in_lab_1_osc_cover.png">
 </a>
</p>

上記コードは**カバレッジ収集**を定義しています：

- 最初の指標（4行目）は車線変更が起こる側―左または右です。
- 2番目の指標（5行目）は車線変更する車種です。例えばトラックまたはセダンです。
- 次の指標（6から11行目）はシナリオフェーズ終了時点で車が走行する速度です。
- 最後に（12から18行目）主要なシナリオが開始される際に2台の車両間で生じる縦方向距離です。

!!!情報
 - 6行目では OSC2 の重要な構造化型メンバーである _event_(イベント) を使用しました。このイベントは_end_(終了) とより具体的に言うと `$FTX_WORKSHOP/scenarios/cut_in_l01.osc` シナリオで定義された change_lane フェーズ終了イベントを表しています。
 - イベントは一過的なオブジェクトであり時間的ポイントを表し、シナリオ内で定義されたアクションをトリガーさせることが出来ます。通常そのような例外的操作やアクターや場面内部でもそれ自身もしくわより典型的操作範囲内部でもそのような事象(インスタンス) を確立することが出来ます。

#### _ts_l01_intro_cov.osc_ KPIの定義

KPIは、前のセクションのカバレッジ項目と同じファイルで定義されています。

このコードのこの部分では、以下のようにKPIを定義しています。

- まず、change_laneの終わりでSUTとcut-in carの間の距離をサンプリングするための変数が、21行から22行で宣言されます。
- 25行から27行では、KPIを後で可視化するために_record()_メソッドを使用して、KPIを記録します。

!!! Info
    いくつかの例を見てきたので、カバレッジメトリックとパフォーマンスメトリックの主な違いを理解することが重要です。

    - **カバレッジ評価**: _どの「シナリオスペース」の部分でAVを実行したか？_ これは、カバレッジと全体的なカバレッジグレードによって表されます。カバレッジ項目は、カバレッジ評価をサポートするために定義されます。つまり、カバレッジグレードは、SUTがどの程度テストされたかという問いに答えます。
    - **パフォーマンス評価**: _テストでSUTがどの程度うまく機能したか？_ この質問には、1つまたは複数のKPIで答えることができます。

#### _ts_l01_intro_checks.osc_ チェッカーの定義

!!! Example "Hands-on Time"
    ソースタブの`ts_l01_intro.osc`テストファイルの6行目をクリックしてインポートされた`ts_l01_intro_checks.osc`ファイルを開きます。

<p align="center">
  <a href="images/l01_kpi_code.png" target="_blank">
    <img src="images/l01_kpi_code.png">
  </a>
</p>

このチェッカーの目的は、シナリオ全体でSUTとcut-in carの間の距離が定義された安全距離内にあるかどうかを評価することです。

- Line 1では、新しいチェッカーのためにユニークな名前（safety_distance）でissue_kindタイプが拡張されています。
- 行3から12では、チェッカーを記述するための以前に定義されたシナリオが拡張されています：
    - 行5 - 6：安全距離のしきい値用の変数が宣言され、13メートルの値に設定されています
    - 行8から10：シミュレーションの各タイムステップ（top.clk）で、以下を確認します：
      - 2台の車両間の距離が定義されたしきい値を超えているかどうか
      - 両方の車両が同じレーンにいるかどうか
    - 行11から12：上記の条件を満たさない場合、テストランはsut_errorタイプおよびsubtype safety_distanceのエラーで停止します。

!!! 情報

    - 追加されたチェッカーはユーザー定義のチェッカーですが、Foretifyには組み込みのチェッカーも付属しています。これらは[Global checkers for vehicles documentation](../osc_dom/oscdomain

```osc linenums="8"
extend test_config:
 set map = "$FTX_WORKSHOP/maps/Town04.xodr"
```

### シナリオの実行

`ts_l01_intro.osc` ファイルの最後の2行では、ついに _cut_in_l01_ シナリオを呼び出します。OSC2 プログラムのエントリーポイントは常に _top.main_ シナリオであり、これは C/C++ の _main()_ 関数と似ています。

```osc linenums="11"
extend top.main:
 do cil : sut.cut_in_l01()
```
このコードはテストファイルの11行目と12行目にあります。

## 最初の実行とForetify

### 制約付きランダム生成および適応的シナリオ実行

その名前が示すように、*制約付きランダム生成* とは指定された制約空間内でランダム変数を生成することを意味します。これは SDV（Safety-Driven Verification）フローの基本的な前提条件であり、Foretify の構築原理です。

OSC2 の抽象シナリオから、Foretify の生成エンジンは指定された制約条件内で具体的なシナリオをランダムに作成します。最も重要な側面の1つは、各シナリオが展開される地図上の領域です。

シナリオの計画が立てられると、ランタイム適応型シナリオ実行エンジンがその計画に従って実行を処理します。

!!! Info
**制約付きランダム生成** エンジンは Foretellix ソリューションの主要な支柱です。これは抽象的な記述から意味あるシナリオ変化を発生させる非常に強力なツールです。

この技術を活用することで、**エンジニア資源** を大幅に削減することが可能です。抽象的な定義から何百万もの有意義なバリエーションを生み出すため、シナリオ作成チームへ必要なものが大幅に削減されます。

```markdown
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
```

- **Load**, **Prepare Test** and **Debug** tabs (top left):
    - **Load**タブはGUIを通じて`.osc`ファイルを読み込むために使用されます（この際、私たちはターミナルコマンドでファイルを読み込んだため、GUIは使用しませんでした）。
    - **Prepare Test**タブでは、実行の異なるパラメータを設定することができます。これについては後のセクションで詳しく学びます。
    - **Debug**タブは、シミュレーションが完了した後に実行をデバッグするために使用されます。
- **Status**（中央上部左側）：
    - 読み込まれたファイル、読み込み状況、および読み込み中に見つかった問題の数を確認できます。
- **Map**, **Source** and **Preview** tabs (中央左側)：
    - **Map**タブでは、読み込まれた地図を閲覧し、地図の異なるレイヤーを探索することができます。
    - **Source**タブには、読み込まれたソースファイルが表示され、読み込まれたコードを確認

```markdown
!!! Example "実践時間"
 実際にテストを実行する前に、VisualizerでSUTとcar1の予定ルートを表示する**Preview**ボタンで実験することができます。予定された経路は、SUTの振る舞いによって実行時に変更されるかもしれませんが、これはテストをデバッグするための非常に役立つツールです。計画された経路はシナリオのために作成された計画の表現だけであることに注意してください：実際の軌跡はランタイム中に計算および更新されます。

!!! Example "実践時間"
 テストを実行する前に、右上隅のグレー色の**Preview**ボタンをクリックし、シード番号を変更していくつかのテストパターンをプレビューしてみてください。

!!! Example "実践時間"
 右上隅の紫色の**Run Test**ボタンをクリックしてテストを実行します。また、右上隅の**Terminal**ボタンをクリックし、「_run_」と入力してもテストを実行できます。

シミュレータウィンドウが表示されます：

- Foretify制約ランダム生成エンジンでは、シナリオ内で指定された制約およびパラメーターに基づいて特定のバリアントが生成されました。プレビュー時に見たもう一方アクター用予定経路です。

- Foretifyランタイムテストオーケストレーションエンジンでは、テスト実行プラットフォーム（シミュレータ）内でアクターが指定した制約どおり移動可能であることが確認されました。

ランサイムテストオーケストレーショングエインでは、「適応的なシナリオ実行」が保証されており、SUT用予定ルートから逸脱した場合はNPCから対応策が講じられるためOSC2ファイルで指示したシナリオ意図が達成されます。

```
After the test is completed, the log area shows two additional tabs:

- **トレースの詳細**タブでは、シミュレーション中に収集された値と共にシナリオトレース情報を調査できます。トレースの種類、アクター、時間、および期間などが含まれます。
- **ログ**タブには、テスト実行中に生成されたログが含まれています。
- **メトリクス**タブは、カバレッジに関連しており、ワークショップで後日詳しく説明されます。

!!! 例 "実践時間"
    現時点では、**ログ**タブを確認し、カットインがどちらの側で発生するかを示すために追加したログメッセージが表示されるかどうかを確認してください。

### ランのデバッグ

#### Foretify Visualizerでデバッグ

テストを実行した後、画面の中央に**Visualizer**タブにアクセスできます。Visualizerは、実行を分析するのに役立つさまざまな方法で構成できるグラフィカルな事後処理ツールの1つです。ランを可視化することは実行を繰り返すこととは

##### マップのパースペクティブ
ビジュアライザーで右クリックしてカーソルを別の方向にドラッグすることで、パースペクティブを変更できます。ビジュアライザーの右上にあるレンチアイコンをクリックすると、「表示ツール」にアクセスできます。「表示ツール」には以下のオプションがあります。

<p align="center">
 <a href="images/visualizer_tools.png" target="_blank">
 <img src="images/visualizer_tools.png">
 </a>
</p>

- レーン方向: 運転方向矢印の表示を有効または無効にします。
- 信号: 信号の表示を切り替えます。
- 速度制限: 道路の速度制限を表示します。
- 衝突回避: 衝突回避機能の可視化を有効にします。
- 実行時軌跡: 選択した車両エージェントの軌跡を表示します。
- 計画された経路: シナリオ内に存在する車両の経路を強調表示します。
- 計画された目的地: 選択した車両エージェント用に生成された計画された目的地を表示します。
- 計画されたポーズ: SUTおよび他の車両の次ポーズが強調表示されます。
- ドライバー目標: 選択した車両エージェント用に生成された運転目標が表示されます。
- 予測位置: 車両の予測位置が表示されます。

##### カメラ設定

パースペクティブとカメラを制御する場合は、[Camera Settings (camera)] アイコン をクリックしてください。特定 のアクター をカメラで追跡する場合は、左上隅（i）から アクターリストから アクター を選択し、もしくはビジュアライザ内で クリックしてカメラ を固定位置 へ設定してください。

<p align="center">
 <a href="images/Camera_settings.png" target="_blank">
 <img src="images/Camera_settings.png">
 </a>
</p>

 - パースペクティブビュー：透視図へ変更
 - 選択したアクターに追従：選択したアクター を追従
 - カメラ を選択した アトウォ の位置ぐ位まり直す：カメラン の位置　服装いセつ 天まりE戒て気儒指す 眠キ floわせ

- もしVisualizerがパースペクティブビューにある場合は、Visualizerの右上にあるカメラ設定アイコンを選択し、パースペクティブビューオプションをオフにしてください。

<p align="center">
  <a href="images/Measurement_1.png" target="_blank">
    <img src="images/Measurement_1.png">
  </a>
</p>

- 測定距離ツールアイコンを選択してください。

<p align="center">
  <a href="images/Measurement_2.png" target="_blank">
    <img src="images/Measurement_2.png">
  </a>
</p>

- Visualizer内でポイントしてクリックして、測定の開始点を設定し、カーソルを移動してクリックして、測定の終了点を設定してください。

<p align="center">
  <a href="images/Measurement_3.png" target="_blank">
    <img src="images/Measurement_3.png">
  </a>
</p>

測定値は、開始点と終了点を結ぶ線の隣に表示されます。

- 測定を非表示にするには、測定距離ツールアイコンを切り替えてオフにしてください。

#### トレースでデバッグ

すべてのトレースは、トレースビューの下で表示できます。トレースビューはタイムラインに合わせて配置されているため、異なるトレースを簡単に比較できます。

**トレース**は、次のような異なるタイプで表されます。

- **インターバル**：一定期間にわたって収集された値のコレクションを表します。インターバルには、名前、開始/終了時間、タイプがあり、特定のアクター（オレンジボックス）に関連付けられています。

- **値**：時間の経過とともに変化する単一の値を表します。値は波形グラフとして表示され、値トレースには、名前、値、単位があり、特定のアクター（赤いボックス）に関連付けられています。

<p align="center">
  <a href="images/Traces_1.png" target="_blank">
    <img src="images/Traces_1.png">
  </a>
</p>

視覚化を向上させるために、Foretifyはすべてのシナリオの開始時間と終了時間を記録します。

##### インターバルを表示するには：

- Foretifyのデバッグ実行タブを選択し、Visualizerの下にあるトレースタブをクリックしてください。

```markdown
&lt;p align="center">
  &lt;a href="images/Traces_2.png" target="_blank">
    &lt;img src="images/Traces_2.png">
  &lt;/a>
&lt;/p>

トレースは、現在時刻を示すカーソルを持つタイムライン上のインターバルとして表示され、VisualizerやTracesタブ内のアクター値トレースなど、他の時間ベースのビューに対応しています。

1. トレース名の左側の矢印をクリックして展開し、その子シナリオを表示します。

2. トレースをクリックして、トレースの詳細（トレースタイプ、アクター、時間、期間、およびインターバル中に収集されたメトリクスなど）を表示します。

3. トレースの詳細で、トレースの開始時刻または終了時刻をクリックして、ユニバーサルタイムラインをその時刻に設定します。

##### タイムラインをトレースに合わせるには：
1. Tracesタブで、タイムラインにフレームを設定したいインターバル（オレン

```markdown
Now you can close Foretify by typing exit in the Foretify terminal or closing the Foretify window.

!!! Info
    The **seed** serves as input for the random generation of one specific concrete execution out of an abstract scenario definition. Generating the concrete test out of the abstract definition again with the **same seed** results in the **same concrete variation**. This is a critical feature that lets you fully randomize the testing on one hand, but also recreate a single concrete execution for debug purposes.

    Thanks to the seed definition and implementation, you can always **trace the particular concrete variation generated.** Traceability of scenarios is a fundamental feature that lets you identify the conditions that led to the bug and to reproduce them.

## Foretify Manager

Now that you've run the first tests, you can explore the achieved coverage and results. We will formally define coverage in the Lab 2, but for now you will visually explore this concept using the Foretellix tools.

Foretify Manager is the Foretellix tool that lets

1

```markdown
!!! Example "Hands-on Time"
 Foretify Manager webアプリに戻り、ブラウザで開いたらリフレッシュボタンを押してください（「Test Suite Results」タブにいることを確認してください）。

アップロード後、テスト実行はロードされ、以前に作成したプロジェクトに表示されます：

<p align="center">
 <a href="images/fmanager_regression_2.png" target="_blank">
 <img src="images/fmanager_regression_2.png">
 </a>
</p>

### アップロードされた実行の分析

ただいまインポートしたテストスイートをクリックすると、個々の実行を確認できます。

以下の画像のようなものが表示されるはずです：

<p align="center">
 <a href="images/fmanager_runs.png" target="_blank">
 <img src="images/fmanager_runs.png">
 </a>
</p>

色つきの四角が示す内容は次の通りです：

- _黄色_: 実行一覧上部にあるアイコンでは、実行をエクスポートまたは削除したり、選択肢を保持またはリセットしたりできます。右端の列選択アイコンではディレクトリやOSユーザー、期間などラン属性を追加・削除できます。
- _オレンジ_: _Issues Tree_ では種類別にすべての問題がグループ化されて表示されます。
- _青_: _Aggregation View_ ではラン属性に基づいて実行が集約表示されます。

_Runs_ ビューで任意の実行をクリックすると新しい Foretify Manager ウィンドウが表示されます。

<p align="center">
 <a href="images/Run_view_2.png" target="_blank">
 <img src="images/Run_view_2.png"> 
 </a>
</p>

おわかりいただけるように、「**Debug Run**」と「**Run Summary**」というメインタブがあります。
「**Debug Run**」タブは以前 Foretify で見たものそのままです、

<p align = "center"> 
<a href =" images/run_source.png "target =" blank "> 
<img src =" images/run_source .png "> 
</ a> 
</ p> 

「 **Run Summary **」タブ をクリックすることで失敗した 実 行 の 詳細 情報 を 詳しく 確認 し ること が でき ます。
```

1

#### カバレッジ
当社の `ts_l01_intro_cov.osc` カバレッジファイルには、4つのカバレッジ項目と1つのKPIメトリックが定義されています。
例えば、cut_in_side はカバレッジグレードが100%であり、一方で speed_sut はわずか20%のカバレージグレードです。これは、speed_sut のカバー率を埋めるためにさらにテストを実行する必要があることを示しています。
当社のファイルでは、これら2つのカバレッジアイテムがわずかに異なる方法で定義されていました：

- 注意：使用するシードによってパーセンテージが異なる場合があります！

<p align="center">
 <a href="images/lab01_coverage_conclusions.png" target="_blank">
 <img src="images/lab01_coverage_conclusions.png">
 </a>
</p>

speed_sut では、各バケットを指定しましたが、cut_in_side では任意の規則を課していません。これにより cut_in_side カバレッジアイテムに余分な自由度が与えられました。このアイテムは100% のカバー率を持っています。左右の可能性それぞれ（left and right）が少なくとも1回ずつテスト中にヒットしたためです。

!!! Example "実践時間"
VPlanタブ内のforetify managerで他のカバリエージアイテムとその評価値を確認してください。_cut_in_side_ のカラーレージ項目を見つけることができます。

#### KPI（重要業績評価指標）
ご記憶の通り、私たちは以前 distance_kpi KPI を定義しました。これは変更車線終了時点でSUTと切り込み車両間の距離を測定します。その際に事象を特定することで最も興味深いピーク時点でアイテム値 をより良く捉えることが可能です。また事象も特定されているため、このKPIでは実行ごとに一度だけ単一値が得られていることに気付きます。

<p align="center">
 <a href="images/workspace_kpi_2.png" target="_blank">
 <img src="images/workspace_kpi_2.png">
 </a>
</p>

!!! Example "実践時間"
どんなKPI値を取得しましたか？Visualizer内でもシミューレーションやその値間関係 を確認することが出来ます。

#### チェックポインタ

**実行**タブを見ると、実行がいくつ成功し、いくつ失敗したかがわかります。失敗した実行は、コードで定義されたチェッカーから来ています。チェッカーは、SUTがブール条件を満たさない場合に失敗応答を定義するため、これは即座にIssues Treeに反映されます。

<p align="center">
  <a href="images/checkers.png" target="_blank">
    <img src="images/checkers.png">
  </a>
</p>

!!! 例 "実践時間"
    実行を分析してください。失敗した実行はいくつありましたか？

### カバレッジの増加

わずかなテストでは、意味のあるカバレッジを達成するには十分ではありません。Foretellixの技術の強みは、大量の自動生成テストを実行することにあります。したがって、このセクションでは、カバレッジを増やすためにさらにテストを実行します。これには、シナリオから異なるテストケースを作成する必要があります。上記の説明にあるように、これは作成時に使用されるシードによって制御されます。

**次に進む前に、ブラウザでForetify GUIを閉じてください**。

!!! 例 "実践時間"
    さらに15のテストを実行し、新しい作業ディレクトリを設定します。Foretify GUIは使用しないため、シミュレータウィンドウが時折表示されます。

    ```bash
    foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir2 \
    --load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 20 --crun 15
    ```

    ```--crun```オプションは、シード20から始まる15のテストを実行します（つまり、シード20から34までのシード）。これはテストをスケールアップするための1つのオプションですが、Lab 3ではさらに多くのオプションをカバーします。

```markdown
注[to_be_replace[x]]を見てください。ジェネレーションエンジンが15個のユニークな具体的なテストケースを生成し、それぞれが地図の異なるセクションにありますが、すべての具体的なテストケースを抽象的なシナリオで定義された境界内に保ちます（例：速度、車線位置など）。

!!! 例 "実践時間"
 新しいランはカバレッジを増やすはずです。次のコマンドを使用して、これらの15個の追加ランをForetify Managerにアップロードできます：

 ```bash
 upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
 --runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir2
 ```

 その後、Foretify Managerの**Test Suite Results**タブで新しくアップロードされたテストスイートを選択し、以下の画像に示すようにそれらをワークスペースに追加します。

 今では **VPlan** ツリーを探索してカバレッジがどれだけ改善したか確認できます。

<p align="center">
 <a href="images/l01_add_ws_1.png" target="_blank">
 <img src="images/l01_add_ws_1.png">
 </a>
</p>

### テストスイート間の切り替え

新しいテストスイートをワークスペースに追加すると、それらのテストスイート間で切り替えることができるようになります。これによりそれぞれ por一つずつ分析することが可能です。

!!! 例 "実践時間"
 ワークスペースに移動して **Test suite result workspace view** の下矢印 をクリックします。

<p align="center">
 <a href="images/l01_switch_ws_1.png" target="_blank">
 <img src="images/l01_switch_ws_1.png">
 </a>
</p>

利用可能なテストスイートが表示される新しいウィンドウが表示されます。差分を確認するためにそれらから切り替えることができます。

<p align="center">
 <a href="images/l01_switch_ws_2.png" target="_blank">
 <img src="images/l01_switch_ws_2.png" width="50%">
 </a>
</p>

新しいテストスイート読み込み後は、「Calculate」ボタン をクリックする必要があります。
```

```markdown
### テストスイートのグループ化

複数のテストスイートを持っている場合、ワークスペースのカバレッジを増やすためにそれらをグループ化したいことがあります。

!!! 例 "実践時間"
    ワークスペースに移動し、**ワークスペースのテストスイート結果**をクリックし、テストスイートを選択して**グループ**アイコンをクリックします。

### テストスイートのアングループ化

作成したグループを解除することも可能です。

!!! 例 "実践時間"
    ワークスペースに移動し、**ワークスペースのテストスイート結果**をクリックし、グループ化された実行を選択して**グループ情報**アイコンをクリックして解除します。

### マップの変更

**次に進む前に、ブラウザでForetifyを閉じてください。

別の方法でカバレッジを増やすには、テストが実行されるODDを変更することです。OSC2言語の強力な機能の1つは、ODDを変更するためにコードの1行だけを変更すればよいということです。

新しいマップがあれば、Foretifyはシナリオをサポートできる場所でランダムに生成された新しい具体的なテストケースを見つけ出します。

!!! Example "ハンズオンタイム"
`$FTX_WORKSHOP/l01_intro/ts_l01_intro.osc`ファイルを開きます。そこにテストが定義されています。

`ts_l01_intro.osc`ファイル内のシナリオを編集し、マップが次のように定義されている行を変更します：

```osc linenums="8"
extend test_config:
  set map = "$FTX_WORKSHOP/maps/cloverleaf.xodr"
```
このコマンドでテストを追加で5回実行できます：

```bash
foretify --work_dir $FTX_FM_WORKDIR/l01_intro/workdir3 \
--load $FTX_WORKSHOP/l01_intro/ts_l01_intro.osc --batch --seed 2 --crun 5
```

これで、シナリオが別のマップ上で実行されることがわかります。

!!! note

特定のシナリオに選択したマップではシナリオが実行不可能な場合は、Foretellixツールはこれを矛盾エラーとして示します。例えば、カットインシナリオは片側通行道路上では実行不可能です。

!!! Example "ハンズオンタイム"
追加のテストケースを実行したら、それらをForetify Managerにアップロードしてカバレッジが改善したかどうか確認することができます。そのために次のコマンドを実行します：

```bash
upload_runs ${FTX_FM_SERVER_ARGS} ${FTX_FM_LOGIN_ARGS} \
--runs_top_dir $FTX_FM_WORKDIR/l01_intro/workdir3
```
前もって作成したワークスペースに新しいランスケース を追加してください。


## 次の手順

このラボの目標は

- ワークショップで使用されるクラウド環境に慣れる
- カットインシナリオの基本的なOSC2コードを確認する
- Foretifyをインタラクティブモードで実行し、カットインシナリオを読み込む
- シードの概念に慣れる
- テスト用のOSC2およびその主要なコンポーネントに慣れる
- Foretify Managerを開き、収集されたメトリクスを探索する

次は、Lab 2ではカットインシナリオを拡張し、カバレッジメトリクスをさらに収集し、Foretifyでシナリオのデバッグ方法を紹介します。

> この投稿は ChatGPT を使用して翻訳されています。何か抜けている部分があれば、[**フィードバック**](https://github.com/linyuxuanlin/Wiki_MkDocs/issues/new) をお願いします。