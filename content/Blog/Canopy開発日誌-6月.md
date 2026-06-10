---
title: Canopy開発日誌-6月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-06-11T04:43:24+09:00
---

# Canopy開発日誌-6月

2026年6月のCanopy開発ログ。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

6月1日は、5月末に進めたLambdaのscope graphとgo-to-definitionを、実際の構造編集へ近づける作業が中心だった。あわせて、CanopyとMoonDspの関係、性能改善の優先度、エージェントの使い分けも整理した。

6月2日は、午前にLambda projectionの命名とidentityの整理を済ませ、午後からincr graph visualizerとcanvas graph demoを進めた。CIのノイズ対策と、Codexにreviewと実装計画のどちらを任せるかの使い分けも少し固まった。

6月3日は、MoonDsp / Canopyの全体方針をmainへ入れ、source-backed canvas demoをブラウザで触れるところまで進めた。CIではIdeal web E2EをPR gateに載せ、incr側ではpublic event APIの命名をDerived寄りに整理した。

6月4日は、Rabbita headless UIがCanopyで本当に使えるのかを見極める日だった。Disclosure PoCとdialog spikeで方向を確認しつつ、MoonDspではGraph runtimeのfacade / internal boundaryを切り始めた。

6月5日は、Rabbitaのpatched更新を取り込み、IdealとCanvasからheadless UI primitiveを実際に使い始めた。Action Menu、ContextMenu、Tabs、TreeViewまで進み、incrではtyped spreadsheetとIncremental TEAの実験も一気に進んだ。

6月6日は、IdealのUI基盤を大きく整理した。Resizable、Statusのlive-region、CSSのde-dupを入れたあとTailwind v4への段階的移行を始め、overlay、toolbar、bottom tabs、panel、inspector、outline resize handleまで小さなslice単位で進めた。

6月7日は、Idealの残タスクとprotocolの曖昧さを潰しながら、Canvasを次のsource-backed段階へ戻した。outline E2E、bridgeのpartial batch、cursor intentの単位名、protocol coordinateのdocs、MoonBit registry cacheを入れ、その後`lib/canvas-graph`の抽出まで進めた。

6月8日は、Canvasのsource-backed graphを本格的にCodeMirrorのsource editorへ寄せた。source-backed inspector edit、selection remap、CM6のchange delta lowering、CodeMirror source panelのmountを順に入れ、夜にはstable canvas node identityのPR2の設計と実装に入り始めた。

6月9日は、日付が変わった時点でPR2が進行中。`NodeId` / `EdgeId`を文字列identityへ寄せ、Loomの`GraphDoc::find_node_by_id`を使う方向でbinding remap shimを消す作業を続けている。

6月10日は、incrのIncremental TEAを一気に仕上げた日。renderer lifecycle、keyed VDOM diff、ベンチマーク、subscriptionsが順にmergeされ、prototypeの主要issueがすべて閉じた。MoonDspはloomパーサー置き換えcampaignのPhase 2 parityを完走してADR-0016をAcceptedにし、js_engineはv0.3.0をリリースした。

6月11日は、loomのseparated-list（#279）とgroup shape helpers（#196）を畳んだあと、Canopyのアーキテクチャ再設計に着手し、S0のproposal + API boundary ADRとS1のprotocol/wire抽出を同じ日にmergeした。js_engineではCIのcache系改善が効かないことを実測で確認し、test262のshardingへ方針を切り替えた。

## 2026/6/1

### Canopy

まず、CanopyとMoonDspの長期的な接続構想を整理した。MoonDspは別リポジトリの音楽DSL / DSPエンジンであり、すぐCanopyへ統合する対象ではない。当面は、Canopyは構造編集の土台、MoonDspは音楽DSL側の実験、incr / Loomは共有基盤、という分け方で考えることにした。
関連PR: [#445](https://github.com/dowdiness/canopy/pull/445)

Lambda projectionでは小さな整理から進めた。`ProjNode`の構築helper、SourceMap token helper、Block / Holeのtyped viewを使ってprojection処理の重複を減らし、その上で、module-level bindingをfirst-classな`LetDef` projection nodeとして扱う変更を入れた。

以前はbinding rowがinit expressionの`NodeId`やsynthetic idを借りていて、Structure modeのdrag & dropやbinding単位のeditで責務が曖昧だった。`LetDef`が実際のnodeになったことで、binding rowを構造編集の対象として素直に扱えるようになった。binderの位置取得は引き続き`@scope.binder_span` / `@scope.go_to_definition`が担当する。
関連PR: [#437](https://github.com/dowdiness/canopy/pull/437)、[#438](https://github.com/dowdiness/canopy/pull/438)、[#439](https://github.com/dowdiness/canopy/pull/439)、[#440](https://github.com/dowdiness/canopy/pull/440)、[#446](https://github.com/dowdiness/canopy/pull/446)、[#448](https://github.com/dowdiness/canopy/pull/448)

性能面では、BAND 2bのevidence gateを走らせた。最適化を書く前に、まず本当に遅いのかを測る段階だ。`to_flat_proj_incremental`は1000 defsで数msまで伸びることが分かったが、実際のCanopy内のLambda documentはまだそこまで大きくない。そのため、この最適化はいったんparkした。大きなdocumentやimport機能が出てきたら改めて取り組む。
関連PR: [#447](https://github.com/dowdiness/canopy/pull/447)

依存関係まわりでは、rle submoduleの更新と、examples配下のVite / Playwright / React系の更新も行った。PlaywrightはpackageだけでなくCI containerも合わせる必要があった。
関連PR: [#435](https://github.com/dowdiness/canopy/pull/435)、[#280](https://github.com/dowdiness/canopy/pull/280)

### loom

`examples/json-settings`に、last-good semantic projection attachmentのchecked exampleを追加した。docsに置いてあったtemplateを、実際にテストできるexampleへ落とし込んだ形だ。ポイントは、`@incr.Derived`のcomputeをpureに保ち、last-good値の保持を外側の`settle`に任せたこと。`Watch::read()`の`ReadError`も、parser / projectionのfailureとは分けて`GraphBlocked`として扱うようにした。
関連PR: [loom #206](https://github.com/dowdiness/loom/pull/206)

Lambda example側ではBlock / Holeのtyped syntax viewを追加し、その後`LetDef` term wrapperを入れた。Canopyのfirst-class `LetDef` projection nodeは、このLoom側の変更を受けて進んでいる。
関連PR: [loom #207](https://github.com/dowdiness/loom/pull/207)、[loom #209](https://github.com/dowdiness/loom/pull/209)

incr依存も整理して、Loomの3つのconsumerをregistryの`dowdiness/incr@0.7.0`へ揃えた。Canopy側の追従も見たが、loom配下のincr submoduleに別作業が残っていたのでいったん保留にした。
関連PR: [loom #208](https://github.com/dowdiness/loom/pull/208)

### js_engine

benchmarkを読みやすくする作業を進めた。focused repeat benchmark runnerを追加し、複数回の測定からmedianやCVを見られるようにした。単発の結果ではなく、ノイズ込みで判断するための道具だ。
関連PR: [js_engine #186](https://github.com/dowdiness/js_engine/pull/186)、[js_engine #187](https://github.com/dowdiness/js_engine/pull/187)

startup側では、`new_interpreter`の中を分けて測り、final realm stampingを軽くした。runtimeで作られるiterator callbackやPromise callbackにもrealmを付ける整理をした。
関連PR: [js_engine #188](https://github.com/dowdiness/js_engine/pull/188)、[js_engine #192](https://github.com/dowdiness/js_engine/pull/192)

Arrayまわりでは、`sort`や`splice`などのmutatorで、明示的な`undefined`とholeの扱いがずれる問題を直した。genericなArray mutatorではProxy receiverを考慮して`HasProperty`相当の経路を使う必要がある、というメモも残した。
関連PR: [js_engine #191](https://github.com/dowdiness/js_engine/pull/191)

### 作業運用

Claude / Codex / piの使い分けも整理した。長い実装計画はCodexが書き、Claude（Opus）が設計と実行管理を持つ。つまりCodexはpre-PR reviewだけでなく、実装順序や検証点を含む計画の書き手でもある。この運用はLoomのjson-settings exampleで最初に試し、`GraphBlocked` stateのような設計漏れも拾えた。

## 2026/6/2

### Canopy

まず、BAND 2bのJS target addendumを追加した。`to_flat_proj_incremental`のcliffはNode上でも再現し、1000 defsのtailで約4msまで伸びた。

一方で、原因の説明は修正した。当初疑っていたper-positionの`start()` / `cst_node()` walkではなく、reuseされたdef subtree同士の`CstNode==`比較と、cacheに縛られたpointer chasingが支配的だった。

ここでも最適化コードは入れていない。cross-parse interningとhash-only reuseという改善手段は残しつつ、実際にそのdocument scaleが必要になるまでparkする判断にした。
関連PR: [#451](https://github.com/dowdiness/canopy/pull/451)

Lambda projectionでは、`FlatProj`を`ModuleProjection`に改名した。単なる置換ではなく、「Moduleのlet defsとfinal exprをflattenした、incremental projection diffの単位」という責務を名前に出すための整理だ。

同時に、ReuseCursor、ProjectionIdentityTracker、`to_module_projection_incremental`という3つのidentity / reuse機構をADRに分けて記録した。[#396](https://github.com/dowdiness/canopy/pull/396)のsource-span tensionや、BAND 2bでparkした最適化が単なるclarity refactorではなく設計トレードオフを持つことも、そこに残してある。

依存関係は`dowdiness/incr@0.7.1`とLoom側の追従へ揃えた。レビュー後には、full delete後に古いmodule rootの`NodeId`を再利用しかねないcache resetの漏れも直した。
関連PR: [#452](https://github.com/dowdiness/canopy/pull/452)

microbenchmark側では、本番と同じようにID sourceを前へ進めるよう修正した。古いprojectionを作ったあと、そのhigh-water markからincremental callを再開する形だ。性能改善ではなく、benchmarkのsetupを本番のID割り当てとずらさないための修正になる。
関連PR: [#453](https://github.com/dowdiness/canopy/pull/453)

### Incr visualizer

IdealのIncrGraph panelは、topologyを見るだけの状態から、recomputeの状態とコストも見える状態へ少し進んだ。

まず、`IncrMemoEventTap`の一時event bufferを、append-onlyではなく「cellごとに最後のeventだけ残す」形にした。status coloringはもともと各cellの最新eventしか使っていないので、bufferのサイズをdistinct cell数で抑えられる。
関連PR: [#462](https://github.com/dowdiness/canopy/pull/462)

その上で、各cellの最新recompute時間をnode detailに表示するようにした。`ns` / `us` / `ms`を切り替えて表示するので、遅いcellが周囲から浮いて見える。Codexのreviewでも、status pathとJS targetのInt64 formattingを確認したうえでPASSになっていた。
関連PR: [#465](https://github.com/dowdiness/canopy/pull/465)

UI側には、IncrGraph panelの色の意味を示すlegendを追加した。Idle / Recomputing / Changed / Failedの4つで、SVG rendererと同じ`VisualNodeStatus::stroke` / `::fill`を使う。
関連PR: [#469](https://github.com/dowdiness/canopy/pull/469)

命名も整理した。tapはMemoだけでなくDerivedのrecomputeも見ているので、`IncrMemoEvent*`を`Recompute*`系へ改名し、snapshot側も`RecomputeSnapshot*`に揃えた。夜のCodexメモではincr本体の`MemoEvent` / `DerivedEvent`の命名移行も検討していて、visualizer名をrecompute中心へ寄せる動きとつながっている。
関連PR: [#471](https://github.com/dowdiness/canopy/pull/471)、[#475](https://github.com/dowdiness/canopy/pull/475)

### Canvas

Canvas exampleでは、graph DSLとUIの接続を進めた。

まず、Graph DSL sourceをroundtripするsmoke testを追加した。あわせてLoom submoduleも進め、canvas側がsource-backed graphを扱う準備を始めた。
関連PR: [#466](https://github.com/dowdiness/canopy/pull/466)

次に、canvasのgraph modelを`examples/canvas/graph_model`へ切り出した。肥大化していた`canvas_state` / `canvas_update`からgraph operationの責務を分離した形で、web側には`GraphAdapter` lifecycleのstubも置いた。
関連PR: [#474](https://github.com/dowdiness/canopy/pull/474)

その後、graph UI stateをincrで分けて`canvas_runtime`を作った。状態更新を直接UI stateへ詰め込むのではなく、graph model・runtime・adapterの境界が見えるようにしている。
関連PR: [#477](https://github.com/dowdiness/canopy/pull/477)

最後に、source-backed graph adapterを追加した。source graphからrealized node idを取り出してログに出せるようになり、ハードコードされたgraph demoから、sourceを背後に持つdemoへ近づいた。
関連PR: [#476](https://github.com/dowdiness/canopy/pull/476)

### CI / 作業運用

CIでは、editor-response benchmarkと`moon update`の2つを安定化した。

editor-responseは、CI runner上で単発のp95 paintがflakeしやすかった。一度CI-only budgetを175msへ緩めたあと、10% trimmed meanを主なgateにして、keystroke数も30から40へ増やした。一時的なspikeではなく、全paintが遅くなる本物のregressionを拾うための変更だ。
関連PR: [#459](https://github.com/dowdiness/canopy/pull/459)、[#463](https://github.com/dowdiness/canopy/pull/463)

`moon update`は、mooncakes CDNの一時的な403で落ちることがあった。そこで、transientな403 / 429 / 5xx / DNS / connection errorに限って回数制限つきでretryする`scripts/moon-update.sh`を追加した。Codex reviewでは、retry判定が広すぎる点と設定値のvalidation漏れが指摘され、修正後にPASSしている。
関連PR: [#468](https://github.com/dowdiness/canopy/pull/468)

さらに、残っていた素の`moon update`呼び出しをwrapper経由に寄せ、再発防止のguardも追加した。MakefileやCloudflareのbuild-deploy scriptまでscan対象を広げたので、PR CIだけでなくdeploy側も同じretry policyに乗っている。
関連PR: [#470](https://github.com/dowdiness/canopy/pull/470)、[#473](https://github.com/dowdiness/canopy/pull/473)

## 2026/6/3

### Canopy

まず、Ideal web E2EをCIのPR gateに載せた。これまで`editor-response.perf.spec.ts`はbenchmark workflowで見ていたが、通常のIdeal editor E2E suiteはPR gateとして独立していなかった。

`scripts/test-ideal-web-e2e.sh`を追加し、performance以外のPlaywright specをまとめて走らせるようにした。`moon update`には前日のretry wrapperを使い、Vite側のMoonBit JS buildでは既知のworkspace target問題を避けるため`MOON_WORK=off`を明示している。
関連PR: [#478](https://github.com/dowdiness/canopy/pull/478)

次に、MoonDsp + Canopy ecosystem visionとBAND 1-2 execution specをmainへ入れた。この日誌の冒頭に置いた全体方針リンクがその文書だ。

内容は、operations-as-dataを中核に据えて、Canopy・MoonDsp・Loom・incrの分担を整理したもの。Canopyは構造編集とprojection、MoonDspはDSP runtime、Loomはsource-backed projection、incrは共有のincremental substrate、という見取り図になっている。multi-agentによるverify passとCodexのdesign reviewを通し、8件の指摘を取り込んだうえでmainへ入れた。
関連PR: [#445](https://github.com/dowdiness/canopy/pull/445)

Canvas exampleでは、6月2日に追加したsource-backed graph adapterを、ブラウザで触れるdemoまで進めた。`?source=1`でsource-backed modeに入り、左側のGraph DSL sourceをcanonical backing storeとして使う。

このdemoでは、node dragはlocal layoutを変えるだけでsource本文には触れない。一方、handle同士をつなぐcanvasのジェスチャーや、`Connect osc -> meter` / `Insert reverb`の操作はcanonical sourceへ下ろされる。

Playwright E2Eでは、dragがsourceを汚さないこと、canvasジェスチャーが`meter = scope(input: osc)`へ反映されること、source editorから4 node / 2 edgeの状態へreparseできることを確認している。
作業ブランチ: `feat/source-backed-canvas-demo`

### MoonDsp

Canopyのような外部editorがaudio previewを渡すときのhandoff contractを文書化した。parser、projection、lowering、template analysis、compile、hot-swapの準備はeditor / control thread側で行い、audio callbackには持ち込まない、という境界を明確にしている。

Preview stateも、単なる再生可否ではなく`Idle` / `Analyzing` / `Ready` / `UsingLastGoodTemplate` / `Failed`として扱う。最新のeditが失敗していても、前のready runtimeがあるならlast-goodのpreviewを鳴らし続ける設計だ。
関連PR: [moondsp #125](https://github.com/dowdiness/moondsp/pull/125)

また、external authoring graphをMoonDspへ渡すコストを測るbenchmarkも追加した。まず基本形を入れ、続けて、より現実的なauthoring shapeを足している。Canopy側のsource-backed graph demoと同じ問題を、MoonDsp runtime側の受け口から測るための土台だ。
関連PR: [moondsp #126](https://github.com/dowdiness/moondsp/pull/126)
作業ブランチ: `issue-127-realistic-external-authoring-benchmarks`

### incr

public event APIの名前を`MemoEvent`から`DerivedEvent`へ寄せた。実際にはMemoに限らずDerivedのrecompute eventを表していたためで、6月2日のCanopy visualizer側の`Recompute*`命名整理とも方向が揃う。

古い`MemoEvent` / `Runtime::on_memo_event`はdeprecated aliasとforwarding methodとして残し、外部consumerのsource互換は保った。version bumpとmooncakesへのpublishはまだ行わず、`0.8.0`向けの変更としてUnreleasedに置いている。
関連PR: [incr #171](https://github.com/dowdiness/incr/pull/171)

Codex reviewは、checked examplesやroadmap docsに古い名前が残っているのを最初に検出してくれた。compat testとdocsの更新を入れてからmergeしている。

並行して、`Input::id`やruntime identity surfaceの設計レビューも行った。こちらはまだ実装前で、single-cell handleだけにidentityを出す方針と、`RuntimeId` wrapperを切る案を確認した段階だ。

## 2026/6/4

### Canopy

この日は、Rabbita headless UIをCanopyでどう扱うかの見極めが中心だった。まずDisclosureのfeasibilityを文書化し、次にnative dialog spikeの結果を残した。

結論は、Disclosure PoCをそのまま広げるのではなく、Canopy内で実際に需要のあるAction Menu、Tabs、TreeView、Resizable、Status / Live Regionを順に切り出すほうがよい、というもの。「headless UIを増やす」のではなく、実際の利用先があるprimitiveだけを切る方針だ。
関連PR: [#501](https://github.com/dowdiness/canopy/pull/501)、[#502](https://github.com/dowdiness/canopy/pull/502)

小さな整理として、btree側の`QueryCase` shrink witnessをmethodに寄せた。本筋ではないが、後のheadless UIやCanvasの作業で使うテスト表現が少し読みやすくなる。
関連PR: [#500](https://github.com/dowdiness/canopy/pull/500)

### MoonDsp

Graph runtimeの公開面を締める作業を始めた。DSP facadeの再exportをやめ、内部model、template / bindingのinternalsを順に切り出した。

Canopyから見ると、これは外部のeditor / authoring graphを受け取るruntime側の境界整理にあたる。Canopyがsource-backed graphを育てるなら、MoonDsp側も「authoring representation」と「audio callbackに持ち込まないruntime boundary」を分けておく必要がある。
関連PR: [moondsp #143](https://github.com/dowdiness/moondsp/pull/143)、[#144](https://github.com/dowdiness/moondsp/pull/144)、[#145](https://github.com/dowdiness/moondsp/pull/145)

### js_engine

Test262の残りを小さなclusterごとに潰していた。class / super / generator、iterator / for-ofのhead、TypedArray constructor / locale stringまわりの不一致を直した。

このあたりはCanopy本体と直接つながらないが、MoonBitで複雑なruntimeを積み上げるときの検証の進め方としてはかなり近い。
関連PR: [js_engine #211](https://github.com/dowdiness/js_engine/pull/211)、[#212](https://github.com/dowdiness/js_engine/pull/212)、[#213](https://github.com/dowdiness/js_engine/pull/213)、[#214](https://github.com/dowdiness/js_engine/pull/214)

## 2026/6/5

### Canopy / Rabbita headless UI

まず、Rabbitaのpatched更新をCanopyに取り込んだ。`rabbita-v0.12.4`系の差分だけでなく、Canopy側のcompat noteやoutline E2Eの安定化も合わせて確認した。
関連PR: [#503](https://github.com/dowdiness/canopy/pull/503)

その後、headless UI最初の実用primitiveとして`lib/menu`を切り出した。Idealのstructural-edit overlayをconsumerにして、keyboard handling、focus request、panel / itemのattrsをbehaviorとして外に出した。
関連PR: [#509](https://github.com/dowdiness/canopy/pull/509)

Canvasが2つ目のconsumerになれるかも確かめた。Canvasのcontext menuはTypeScriptのDOM実装だったので、いきなり`@menu`を入れるのではなく、まずMoonBit側のRabbita islandへmenu surfaceを寄せてから使った。これで`lib/menu`はIdeal専用の抽象ではなくなった。
関連PR: [#513](https://github.com/dowdiness/canopy/pull/513)

続いて、ContextMenuをMenuの上の別primitiveとして切った。Canvasの右クリックメニューを実consumerにして、trigger anchor、dismissable layer、close時のfocus、viewport positioningを順に足していった。途中、整数に丸めていたtrigger座標はfractionalなanchorを保持する形に直した。
関連PR: [#516](https://github.com/dowdiness/canopy/pull/516)、[#522](https://github.com/dowdiness/canopy/pull/522)、[#523](https://github.com/dowdiness/canopy/pull/523)、[#524](https://github.com/dowdiness/canopy/pull/524)、[#525](https://github.com/dowdiness/canopy/pull/525)

TabsとTreeViewもこの日に入った。TabsはIdealのbottom tabsをconsumerにし、TreeViewはWAI-ARIAのtree behaviorとして分けた。
関連PR: [#517](https://github.com/dowdiness/canopy/pull/517)、[#526](https://github.com/dowdiness/canopy/pull/526)

### incr

typed spreadsheet exampleが大きく進んだ。user-facing snapshotにlast dynamic dependenciesを出し、formula factsをevaluatorから分離して、formulaのdependency shapeを分類できるようにした。
関連PR: [incr #192](https://github.com/dowdiness/incr/pull/192)、[#193](https://github.com/dowdiness/incr/pull/193)、[#194](https://github.com/dowdiness/incr/pull/194)

そのうえで、bounded traceのbaseline benchmarkと実APIを入れた。global traceではなく、観測範囲を絞ったtraceをUIやdemoから説明できるようにするための土台だ。後続で、evidence copy、performance workflow、event-traceのfeasibilityも整理した。
関連PR: [incr #195](https://github.com/dowdiness/incr/pull/195)、[#196](https://github.com/dowdiness/incr/pull/196)、[#202](https://github.com/dowdiness/incr/pull/202)、[#203](https://github.com/dowdiness/incr/pull/203)、[#204](https://github.com/dowdiness/incr/pull/204)

さらにIncremental TEA exampleも始めた。skeleton、lifecycle、Cmd scheduler、watched Html renderer prototypeの順で進め、`Runtime::batch`と`Watch`を使ってTEA風のmessage loopをincr上でどう表現するかを試している。
関連PR: [incr #205](https://github.com/dowdiness/incr/pull/205)、[#206](https://github.com/dowdiness/incr/pull/206)、[#207](https://github.com/dowdiness/incr/pull/207)、[#208](https://github.com/dowdiness/incr/pull/208)

### MoonDsp

前日のboundary splitをさらに進めた。Graph runtime、scheduler、browser internalsを切り出し、browser / workletのABI contractとparse resultのerror transportを文書化した。

Canopyとの連携の観点では、editor側の失敗やparse resultをどうruntimeへ渡すか、どこから先をaudio callbackに持ち込まないか、という境界が少しずつ具体化している。
関連PR: [moondsp #146](https://github.com/dowdiness/moondsp/pull/146)、[#147](https://github.com/dowdiness/moondsp/pull/147)、[#148](https://github.com/dowdiness/moondsp/pull/148)、[#149](https://github.com/dowdiness/moondsp/pull/149)、[#153](https://github.com/dowdiness/moondsp/pull/153)、[#154](https://github.com/dowdiness/moondsp/pull/154)、[#155](https://github.com/dowdiness/moondsp/pull/155)、[#159](https://github.com/dowdiness/moondsp/pull/159)、[#160](https://github.com/dowdiness/moondsp/pull/160)、[#161](https://github.com/dowdiness/moondsp/pull/161)

### js_engine

ES2015の残りからTypedArray / Mapまわりまで、Test262 conformanceをかなり進めた。ArrayBuffer / DataView / TypedArrayのconstructor、integer-index internals、arguments objectのArrayIterator、Map iteratorのreceiver / exhaustion semanticsなど。
関連PR: [js_engine #215](https://github.com/dowdiness/js_engine/pull/215)、[#216](https://github.com/dowdiness/js_engine/pull/216)、[#217](https://github.com/dowdiness/js_engine/pull/217)、[#218](https://github.com/dowdiness/js_engine/pull/218)、[#219](https://github.com/dowdiness/js_engine/pull/219)、[#220](https://github.com/dowdiness/js_engine/pull/220)、[#221](https://github.com/dowdiness/js_engine/pull/221)、[#222](https://github.com/dowdiness/js_engine/pull/222)

## 2026/6/6

### Canopy / Ideal UI foundation

この日はIdealのUI基盤を大きく進めた。まずMenuのfocusをscope-awareにし、Idealのoutline panelに`lib/resizable`を統合した。Resizableはdemo用ではなく、実際のeditor layoutで使われるprimitiveになった。
関連PR: [#528](https://github.com/dowdiness/canopy/pull/528)、[#529](https://github.com/dowdiness/canopy/pull/529)

Canvas向けには、再利用可能なStatus / Live Region behaviorを切り出した。成功 / 失敗 / infoのtoneとpolite announcementをbehaviorとして外に出し、source-backed Applyのstatus表示を最初のconsumerにしている。
関連PR: [#531](https://github.com/dowdiness/canopy/pull/531)

CSS面では、Ideal shadow CSSの重複をなくすfoundationを入れた。`editor-shadow.css`を`adoptedStyleSheets`でsingle-source化し、JSのstring constantとして持っていた`SHADOW_STYLES`を削除した。これで、後続のTailwind移行を判断できる状態になった。
関連PR: [#532](https://github.com/dowdiness/canopy/pull/532)

その後、Tailwind v4への段階的な移行を始めた。最初はaction overlayとname prompt、続いてstyle management docs、toolbar recipe、bottom tabs、panel chrome、inspector detail、outline resize handleへと進めた。

一気に全部をTailwind化するのではなく、既存のsemantic hookを残しながら、light-DOMの小さなchrome sliceだけを移す方針にした。`@apply`は使わず、Idealローカルのrecipe helperとclass bundleで管理する。
関連PR: [#534](https://github.com/dowdiness/canopy/pull/534)、[#535](https://github.com/dowdiness/canopy/pull/535)、[#539](https://github.com/dowdiness/canopy/pull/539)、[#540](https://github.com/dowdiness/canopy/pull/540)、[#541](https://github.com/dowdiness/canopy/pull/541)、[#542](https://github.com/dowdiness/canopy/pull/542)、[#543](https://github.com/dowdiness/canopy/pull/543)、[#544](https://github.com/dowdiness/canopy/pull/544)、[#545](https://github.com/dowdiness/canopy/pull/545)

### MoonDsp

Graph facadeのparity guardとruntime boundary inventoryを追加したあと、native CLAP prototypeに入った。公式のCLAP headerをvendorし、MoonBit bridge headerの生成を安定化させ、CLAP prototypeのCI gateとWindows hostでのload修正まで進んだ。

MoonDspは、Webのauthoring graphだけでなく、native plugin hostとの境界も現実の検証対象になってきた。
関連PR: [moondsp #165](https://github.com/dowdiness/moondsp/pull/165)、[#166](https://github.com/dowdiness/moondsp/pull/166)、[#167](https://github.com/dowdiness/moondsp/pull/167)、[#168](https://github.com/dowdiness/moondsp/pull/168)、[#169](https://github.com/dowdiness/moondsp/pull/169)、[#170](https://github.com/dowdiness/moondsp/pull/170)、[#175](https://github.com/dowdiness/moondsp/pull/175)、[#176](https://github.com/dowdiness/moondsp/pull/176)

### loom / incr / js_engine

LoomにはMoonBit parser skeleton exampleが入った。6月7日のMoonBit parser integrationラッシュへつながる最初の足場だ。
関連PR: [loom #220](https://github.com/dowdiness/loom/pull/220)

incrではAcceptedDerived系の設計が進んだ。まずdesign specを置き、state-machine spikeで受け入れ済みの値をどう保持するかを検証している。
関連PR: [incr #213](https://github.com/dowdiness/incr/pull/213)、[#214](https://github.com/dowdiness/incr/pull/214)

js_engineではMap / module graph / tooling migrationの整理が進んだ。Map.groupByやMap.prototype.forEachの残りを直し、ES module graphのlive bindingsを改善して、Test262 runnerやstdlib descriptor helperの整理も進めた。
関連PR: [js_engine #223](https://github.com/dowdiness/js_engine/pull/223)、[#224](https://github.com/dowdiness/js_engine/pull/224)、[#225](https://github.com/dowdiness/js_engine/pull/225)、[#226](https://github.com/dowdiness/js_engine/pull/226)、[#227](https://github.com/dowdiness/js_engine/pull/227)、[#246](https://github.com/dowdiness/js_engine/pull/246)、[#247](https://github.com/dowdiness/js_engine/pull/247)、[#259](https://github.com/dowdiness/js_engine/pull/259)

## 2026/6/7

### Canopy / Ideal and protocol

Ideal Tailwind側では、outlineのpeer / empty-state chromeを移した。6月6日から続いていたTailwind migrationの最初の山は、これで一段落した。
関連PR: [#546](https://github.com/dowdiness/canopy/pull/546)

その後はTailwind以外の残タスクに戻り、outline tree E2Eの穴を埋め、`bridge.ts::applySpliceChanges`のpartial batch broadcast semanticsを直した。
関連PR: [#553](https://github.com/dowdiness/canopy/pull/553)、[#554](https://github.com/dowdiness/canopy/pull/554)

protocol側では、`UserIntent.SetCursor(position)`の曖昧さを解消した。PMのtree positionとCMのdoc code-unit offsetが、どちらも`position`という名前で流れていたので、`SetPmCursor(pm_tree_position)`と`SetDocCursor(doc_code_unit_offset)`に分けた。

続けて、protocolのcoordinate unitsとwire-format変更の規律も文書化した。この種の単位名の混同は後から効いてくるので、早めに潰せてよかった。
関連PR: [#555](https://github.com/dowdiness/canopy/pull/555)、[#558](https://github.com/dowdiness/canopy/pull/558)

CIでは、MoonBitのregistry stateをcacheし、`moon-update.sh`のdiagnosticsを改善した。CIのflakeを「再実行すればよい」で済ませず、観測しやすい形へ寄せている。
関連PR: [#560](https://github.com/dowdiness/canopy/pull/560)

### Canvas

夕方からCanvasのgraph model抽出に戻った。`examples/canvas/main`のUI stateやupdateに混ざっていたdurableなgraph operationを、`lib/canvas-graph`へ切り出した。

`WorkflowAction`、`GraphOperation`、`CanvasState`、reducer APIがlibrary側へ寄ったことで、source-backed loweringと手組みのcanvasが同じmodelを使いやすくなった。6月8日のsource-backed editとCodeMirror source panelの作業は、これが前提になっている。
関連PR: [#562](https://github.com/dowdiness/canopy/pull/562)

### loom

Loomはこの日大きく進んだ。まず、zero-widthなlexer tokenのreuse boundaryを固めた。located token adapter、token provenance offsets、zero-width boundaryのdocs、synthetic zero-width hookのparser-owned化、property test、benchmark、RepeatGroup canonicalizationの修正と文書化まで一気に進んでいる。
関連PR: [loom #221](https://github.com/dowdiness/loom/pull/221)、[#229](https://github.com/dowdiness/loom/pull/229)、[#230](https://github.com/dowdiness/loom/pull/230)、[#231](https://github.com/dowdiness/loom/pull/231)、[#233](https://github.com/dowdiness/loom/pull/233)、[#234](https://github.com/dowdiness/loom/pull/234)、[#235](https://github.com/dowdiness/loom/pull/235)、[#237](https://github.com/dowdiness/loom/pull/237)、[#240](https://github.com/dowdiness/loom/pull/240)、[#242](https://github.com/dowdiness/loom/pull/242)、[#246](https://github.com/dowdiness/loom/pull/246)

MoonBit parser integrationも進み、full MoonBit token syntax kinds、top-level headers、differential fixtures、syntax-onlyのreactive parser、ParserContext grammar helpersまで入った。

Canopy側のCodeMirrorやlanguage toolingを考えると、Loomは単なるparser runtimeではなく、editorに向けたsyntax artifactを出せる方向へ寄ってきている。
関連PR: [loom #236](https://github.com/dowdiness/loom/pull/236)、[#239](https://github.com/dowdiness/loom/pull/239)、[#241](https://github.com/dowdiness/loom/pull/241)、[#247](https://github.com/dowdiness/loom/pull/247)、[#248](https://github.com/dowdiness/loom/pull/248)、[#249](https://github.com/dowdiness/loom/pull/249)、[#250](https://github.com/dowdiness/loom/pull/250)

### MoonDsp / js_engine / incr

MoonDspでは、CLAP audio callbackのallocation auditを行い、callbackでのallocationを除去した。real hostでのcoverage checklistも追加して、CLAP prototypeは「作った」段階から「hostの上でどう検証するか」へ進んでいる。
関連PR: [moondsp #177](https://github.com/dowdiness/moondsp/pull/177)、[#178](https://github.com/dowdiness/moondsp/pull/178)、[#179](https://github.com/dowdiness/moondsp/pull/179)、[#181](https://github.com/dowdiness/moondsp/pull/181)

js_engineでは、Array method fast path delegationとMoonBit Test262 runner shadowが進んだ。iterator / copy / push-pop系のfast pathを分け、Python toolingからMoonBit shadowへ移す準備も入っている。
関連PR: [js_engine #264](https://github.com/dowdiness/js_engine/pull/264)、[#265](https://github.com/dowdiness/js_engine/pull/265)、[#266](https://github.com/dowdiness/js_engine/pull/266)、[#267](https://github.com/dowdiness/js_engine/pull/267)、[#268](https://github.com/dowdiness/js_engine/pull/268)、[#271](https://github.com/dowdiness/js_engine/pull/271)、[#273](https://github.com/dowdiness/js_engine/pull/273)、[#274](https://github.com/dowdiness/js_engine/pull/274)、[#275](https://github.com/dowdiness/js_engine/pull/275)、[#276](https://github.com/dowdiness/js_engine/pull/276)、[#277](https://github.com/dowdiness/js_engine/pull/277)、[#278](https://github.com/dowdiness/js_engine/pull/278)

incrでは、typed spreadsheetのAI / tool支援と、AD / RAD方向のissueを整理した。実装としてはformula AST accessorが入り、次のAI context exportにつながる準備ができた。
関連PR: [incr #224](https://github.com/dowdiness/incr/pull/224)

## 2026/6/8

### Canvas

日付が変わったあと、source-backed canvasのnode renameと数値パラメータの編集をGraph UIから行えるようにした。renameや数値の編集はUI stateを直接いじるのではなく、Graph DSL sourceへ下ろして、reparseされたsource-backed graphを表示し直す。
関連PR: [#563](https://github.com/dowdiness/canopy/pull/563)

続けて、source reorder後にもsource-backed selectionを保つ暫定fixを入れた。この時点では`NodeId`がまだdoc-order由来なので、bindingで選択をrecoverする方式だ。stable identityへ進むまでの橋渡しになる。
関連PR: [#564](https://github.com/dowdiness/canopy/pull/564)

次に、CodeMirrorのchange deltaをLoomのedit列へ下ろす経路を作った。`lib/rabbita_codemirror`の`listen()`にadditiveな`on_change?: Emit[Array[ChangeDelta]]`を足し、JS側の`iterChanges`をMoonBitのdelta payloadへ変換する。Canvas側はそれをordered Loom editsにして`apply_edit`へ渡す。
関連PR: [#569](https://github.com/dowdiness/canopy/pull/569)

最後に、Canvasのsource panelをtextareaからCodeMirrorに置き換えた。これで`?source=1`のsource panelは、CodeMirror bufferをauthoritative sourceとして扱う。

途中でdirty edit recoveryの設計を見直し、rollbackではなく「editor bufferが常に真。parse成功時はcurrent_result、失敗時はlast_goodを表示」という形に寄せた。`/code-review`で出たV5 / V8 / V10の3件は同じPRにfoldし、CIがgreenになってからmergeした。
関連PR: [#570](https://github.com/dowdiness/canopy/pull/570)

この流れで、stable canvas node identityの実行specも固まった。PR1はCodeMirror source mountまで、PR2で`NodeId` / `EdgeId`をstableなsemantic identityへ切り替える、という分割だ。
作業ブランチ: `feat/565-pr2-stable-canvas-node-identity`

### loom

Monogramに着想を得た、parser-backed role span方向の最初のsliceが進んだ。JSON syntax role spanのprojection、parser-backed `@incr` attachment、editor-neutralなexportを順に追加した。

まだ共有の`LanguageModel` APIを切る段階ではなく、JSONローカルに閉じたsliceで、CanopyやCodeMirrorが消費できるrole spanをどう出すかを見ている。
関連PR: [loom #263](https://github.com/dowdiness/loom/pull/263)、[#265](https://github.com/dowdiness/loom/pull/265)、[#266](https://github.com/dowdiness/loom/pull/266)

夜には、CST traversal idioms guideとmetadata-trust invariantのproperty testも入った。今後agentや人間がCST metadataを雑に信用しすぎないためのguardになる。
関連PR: [loom #270](https://github.com/dowdiness/loom/pull/270)

### incr

AcceptedDerivedまわりが一気に進んだ。public facade、checked examples、BackdateEq acceptance tierを入れ、CIもlibraryとdocs examplesの両方を走らせる形に強化した。
関連PR: [incr #215](https://github.com/dowdiness/incr/pull/215)、[#226](https://github.com/dowdiness/incr/pull/226)、[#228](https://github.com/dowdiness/incr/pull/228)、[#231](https://github.com/dowdiness/incr/pull/231)、[#232](https://github.com/dowdiness/incr/pull/232)

typed spreadsheet側には、AI context exportが入った。selected cell、dependencies、bounded trace、formula ASTなどをdeterministicに取り出すためのsurfaceだ。
関連PR: [incr #225](https://github.com/dowdiness/incr/pull/225)

さらに、diamond dependencyで`push_reachable_count`がずれる問題を直し、Salsaとのengine comparison noteも追加した。
関連PR: [incr #233](https://github.com/dowdiness/incr/pull/233)、[#234](https://github.com/dowdiness/incr/pull/234)

### js_engine / MoonDsp

js_engineでは、Test262 runnerのMoonBit shadow化をさらに進めた。Array shift / unshiftのfast path delegation、runner parity、shadow artifact parity、CI artifactsまでつなげている。
関連PR: [js_engine #279](https://github.com/dowdiness/js_engine/pull/279)、[#280](https://github.com/dowdiness/js_engine/pull/280)、[#281](https://github.com/dowdiness/js_engine/pull/281)、[#282](https://github.com/dowdiness/js_engine/pull/282)、[#283](https://github.com/dowdiness/js_engine/pull/283)

MoonDspでは、PatternDocの`BackdateEq`と、loom-mini-cstのlast-good projectionの実験を始めた。parse errorが起きても、前回成功したprojected docを出せるようにする方向だ。
関連ブランチ: `feat/pattern-backdate-eq`、`feat/loom-mini-cst-last-good`

## 2026/6/9

### Canopy

日付が変わった時点では、stable canvas node identityのPR2を進めている。source-backed canvasの`NodeId`を、doc-order由来の`Int`から、Loomの`ProjectionIdentityTracker`が出す`GraphNode::id()`文字列へ寄せる作業だ。

実行addendumで、tokenのformatが`node#<occurrence>:<binding>`であることを確認した。`set_source` / `apply_edit`で別のbinding lineをdeleteした場合、生き残ったtokenはstableで、atomicな`set_source`によるreorderのときだけtokenがchurnする。

この結果から、deleteを無理にdelta変換する必要はないと判断した。user gestureとしてのCodeMirror reorderはdeltaのdelete + insertとして入ってくるので、既存のdelta pathでidentityを保てる。`set_source`はreset / init寄りのpathとして扱い、reorderでchurnしたときはselectionをremapせず自然にdropさせる。

PR2の中心は、binding-recovery shimを消すことだ。`source_selected_node_bindings`、`source_selection_from_bindings`、`remap_selection_to_bindings`のようなcapture / replayをやめて、`prune_selection_to_doc(doc)`で「今のdocに存在するtokenだけ残す」形に寄せている。

renameだけはbindingがtokenに含まれるため、古いtokenから新しいtokenへselectionを書き換える局所的なhookを残す。`action_log`は過去の表示用なのでrewriteしない。

`EdgeId`もallocatorではなく、`(source_token, source_port, target_token, target_port)`の4-tupleから導出する方針だ。手組みのcanvasだけは`next_node_id`を文字列化したmint counterとして残すが、source-backed canvasとは`CanvasState`を共有しないので衝突しない。

現在のworktreeはまだmerge前の作業中。`lib/canvas-graph`、`graph_dsl_adapter`、Canvas runtime / TS adapterまで`Int` → `String`の変更が波及していて、source-backed selectionのテストを仕上げているところだ。
作業ブランチ: `feat/565-pr2-stable-canvas-node-identity`

### loom / MoonDsp

このPR2のために、Loom側には`GraphDoc::find_node_by_id(String)`を追加した。Canopy側の`graph_node_for_canvas_id`は、`nodes[raw - 1]`のようなpositional lookupではなく、このtoken lookupを使う予定だ。
関連コミット: `feat(graph-dsl): add GraphDoc::find_node_by_id for token-identity lookup`

MoonDsp側では、loom-mini-cstのlast-good projectionを進めた。parse error時にlast-goodなprojected docを返す実装を入れ、value-collisionの前提やempty-acceptのケースもテストで固定した。その後、BackdateEq / last-goodのテストをmatch / guard / map寄りに整理している。
関連ブランチ: `feat/loom-mini-cst-last-good`

## 2026/6/10

### incr / Incremental TEA

Incremental TEAの残りのissueを、この日でほぼすべて畳んだ。

まず#209のrenderer unmount / dispose lifecycle。`BrowserRenderer`がすべてのrootを所有する形にして、mounted（毎rAFでflush）とdetached（DOMからは外すがProgram / Scope / Watchは生かしたまま）の2バケツで管理する。detach / reattach / destroy / disposeを揃え、Programを共有しているrootが残っていないか数えてから破棄するので、共有viewを壊さない。
関連PR: [incr #239](https://github.com/dowdiness/incr/pull/239)

次に#211のkeyed VDOM diffとpureなevent payload。`EventBinding`が`Msg`を直接持つのをやめて、`Const(Msg) | Input(tag~)`というpureなdescriptorにした。text入力の解決はrenderer boundaryのresolver（mount時の`on_input`）に置いたので、`Html`値はEqでcacheできるpure dataのまま保たれる。keyed childrenは、DOMに触らないpureなplanner `plan_keyed_diff`がreuse計画を立て、applierがkeyごとにnodeとlistenerを再利用する。
関連PR: [incr #240](https://github.com/dowdiness/incr/pull/240)

#189のベンチマークも取った。毎回viewを再構築してEqでdiffするdirty-cellなbaselineと比べると、viewが読んでいないフィールドのmutationをincrはO(1)（約0.5µs）でskipし、N=16 / 64 / 256で9.5× / 36× / 159×速い。skipの効きがviewサイズに比例する、というのが見たかった数字だ。一方、読まれるmutationではbatchとpropagationのオーバーヘッドでほぼ同等。plannerはO(n·m)なので、key-map + LIS化はfollow-up（#241）に残した。
関連PR: [incr #243](https://github.com/dowdiness/incr/pull/243)

最後に#188のsubscriptions。望ましいsubscription集合をtrackedな`Derived[Subscriptions[Msg]]`として持ち、graph外の`SubscriptionsManager`がtimerなどのside-effect handleを管理する。`SubKey(namespace_id, identity)`で安定したreconciliationを行い、同じkeyのhandlerは作り直さずin-placeで更新する。仕上げにwarning掃除も入れた。
関連PR: [incr #244](https://github.com/dowdiness/incr/pull/244)、[#245](https://github.com/dowdiness/incr/pull/245)

これでTEA trackの主要issueは、ベンチマークまで含めてすべて閉じた。

### MoonDsp

前日の夜、loomで本番のminiパーサー（runtimeの`@mini.parse`とauthoring pipelineの両方）を置き換えるcampaignを立ち上げていた。Phase 0のfeasibility gate（3 backendでのbuild、依存の隔離、error adapterの形、publish bundleの分離）はGO（[moondsp #189](https://github.com/dowdiness/moondsp/pull/189)）。この日はPhase 2のauthoring parityを完走した。

Piece 1はvalue-levelのdifferential parity。40-inputのcorpusでloom版とhand-written版のPatternDocを突き合わせ、value-levelのdivergenceはゼロだった。
関連PR: [moondsp #190](https://github.com/dowdiness/moondsp/pull/190)

Piece 2はsong-levelの対応。songのgrammarとprojectionを足し、bpm、sectionのlayout、occurrence id、sectionごとのbody eventsをoracleと突き合わせるparity harnessを入れた。song keywordは専用tokenではなく`Ident` + `LParen`の隣接で判定する。
関連PR: [moondsp #192](https://github.com/dowdiness/moondsp/pull/192)

Pieces 4+3でcorpusを40から84へ広げた。本物のdivergenceは1件だけ — oracle側のtrailing-junk leniencyで、`s("bd] sd")`を黙って`bd`として解釈してsdを落とす。loomは拒否する。loomの方が正しい挙動なので、characterization testで両側の現状を固定した。
関連PR: [moondsp #194](https://github.com/dowdiness/moondsp/pull/194)

そのうえで、Phase 3（runtime swap）のgateとなるADR-0016をAcceptedにした。`@mini`のsurface / ABI / error contractを凍結したままloomへ差し替える方針で、error adapterはraw CST diagnosticsからposition付きのメッセージを組み立てる。corpus parse latency ≤1.3msなどのacceptance gateもここで決めた。
関連PR: [moondsp #196](https://github.com/dowdiness/moondsp/pull/196)

細かいものでは、ADR-0010のcompile-seam reuse invariantのテスト固定、agent guideのportable化、shipping packageへのcore Iter / Array / Stringメソッドの適用、宣言的iterator chainへのsweep（日付が変わった直後にmerge）も入った。
関連PR: [moondsp #191](https://github.com/dowdiness/moondsp/pull/191)、[#193](https://github.com/dowdiness/moondsp/pull/193)、[#195](https://github.com/dowdiness/moondsp/pull/195)、[#197](https://github.com/dowdiness/moondsp/pull/197)、[#198](https://github.com/dowdiness/moondsp/pull/198)

### loom

Phase 2のsong projectionを作る過程で出てきたupstream要求のうち、#280のtoken adjacency primitivesを実装した。`ParserContext::at_adjacent` / `expect_adjacent`で、trivia無しの連接を判定する。Codex reviewが「index-onlyの隣接判定はunsound」という本質的な穴を見つけた — zero-widthのtokenが間に挟まるケースと、sourceに現れないgapのケースがあるため、indexのcheckとoffsetのcheckの両方が要る。merge後、moondsp側の`expect_song_keyword`は手書きのoffset比較からこれに移行した（上の[moondsp #197](https://github.com/dowdiness/moondsp/pull/197)）。
関連PR: [loom #284](https://github.com/dowdiness/loom/pull/284)

#281の`SyntaxNode::text()`（covered-source accessor）と、seam helperのpositioned iterator chains化も入れた。`token_at_offset`のdeclarative化はbenchmarkで悪化が出たため棄却し、imperativeのまま残している。
関連PR: [loom #282](https://github.com/dowdiness/loom/pull/282)、[#283](https://github.com/dowdiness/loom/pull/283)

### Canopy

日付が変わってすぐ、Canvasのsource panelのpollingをincrのreactive invalidationへ置き換えるPRをmergeした（#565のfollow-up）。settle pingを1つの`SyncFromGraph`にまとめるperf修正も含む。なお前日の日中には、stable canvas node identityのPR2自体が[#571](https://github.com/dowdiness/canopy/pull/571)としてmerge済みで、source-backed canvasの`NodeId`はprojection tokenの文字列になっている。
関連PR: [#576](https://github.com/dowdiness/canopy/pull/576)

前日の夜にはincr 0.9.0のbump train（incrのpublish → loom [#276](https://github.com/dowdiness/loom/pull/276) → canopy [#572](https://github.com/dowdiness/canopy/pull/572)）が完走していて、canopy側はMoonBit 0.10.0 toolchainのpin、incr minor driftのCI check（[#574](https://github.com/dowdiness/canopy/pull/574)）、loom submoduleの追従（[#575](https://github.com/dowdiness/canopy/pull/575)）まで済んでいた。この日はそれを制度として固定するshared-substrate incr version-lockのADRを入れ、issue #441を閉じた。canopyとMoonDspが同じincr minorに揃っていることと、bottom-up（incr → egglog → loom → canopy）のpaired-bump protocolを文書として残した形だ。
関連PR: [#577](https://github.com/dowdiness/canopy/pull/577)

AGENTS.mdの整理も行った。workspace / submodule記述のdrift修正、重複セクションの削減、実装ポリシーの追記。
関連PR: [#580](https://github.com/dowdiness/canopy/pull/580)

### js_engine

v0.3.0をリリースした。リリース後に、report_test262がghのnumericなdatabaseIdをdecodeできない問題を直し、RELEASING.mdに具体的なmooncakes publishチェックリストを足した。
関連PR: [js_engine #287](https://github.com/dowdiness/js_engine/pull/287)、[#288](https://github.com/dowdiness/js_engine/pull/288)、[#289](https://github.com/dowdiness/js_engine/pull/289)

## 2026/6/11

### loom

separated-list parsing（#279）を、独立した2本のPRで仕上げた。

[#285](https://github.com/dowdiness/loom/pull/285)はseam側の`SyntaxNode::direct_elements_grouped_by(separator)`。N個のseparatorをN+1個のgroupに分け、empty groupも保持する。[#286](https://github.com/dowdiness/loom/pull/286)はcore側の`ParserContext::separated_list` combinator。markerによるretroactive wrapで実装し、separatorに隣接するemptyなslotにはzero-widthのerror placeholderと「expected element」診断を入れる。slotごとにreuse-awareで、reuse round-tripのテストで全slotが再利用されることも確認した。

[#287](https://github.com/dowdiness/loom/pull/287)でplan docsをarchiveしてboundary-modelのADRを追加し、#279をclose。続けて#196（group-level shape helpers）も、span-carrying `DirectElementGroup`のAPIとして実装してcloseした。
関連PR: [#288](https://github.com/dowdiness/loom/pull/288)

Codexのpre-PR reviewは2本ともFAIL判定で、lone-separatorケースなどのtest gapを拾った。実装のdefectはゼロで、指摘はすべてテスト追加で対応している。

### Canopy / アーキテクチャ再設計

repo構造の再設計に着手し、S0とS1を同じ日にmergeした。

S0はdocs-only（[#581](https://github.com/dowdiness/canopy/pull/581)）。S0からS6までのstaged migrationを定めたredesign proposalと、3-tierのAPI boundary ADRを入れた。Tier 1がlibrary surface（core / projection / editor / protocol / 将来のprotocol/wire / ephemeralはmodelのみ）、Tier 2がlanguage SPI（lang/*）、Tier 3がinternal（ffi/*、workspace/*、relay、llm、examplesなど）という区分だ。

設計にあたっては、まずExplore agent 4本の並列調査（editorの構造、ideal / ffiの構造、docsの意図、lang-familyの対称性）で現状を把握してから、Codexのdesign reviewを通した。Codexの指摘でmigrationの進め方にはいくつか修正が入った。たとえばS5のde-dup方向は逆転して、loom側をregistryのegglogへ寄せ、canopyはegw submoduleを直接keepする — 外すとegwの変更ごとにloom PRのgateが1つ増えるからだ。target architecture自体は変わっていない。

S1は最初の実装ステージ（[#582](https://github.com/dowdiness/canopy/pull/582)）。sync wire protocolのcodec、ProtocolError、envelope定数、EphemeralNamespaceのframe APIを`protocol/wire`へbyte-equivalentに移した。frozen fixturesは無修正のまま通り、editor側は`#deprecated`のtransparent aliasとforwarding shimで互換を保つ。wire → ephemeral → editorとchainしたpub usingが機能して、`.mbti`に`@wire`起源が出ることも確認した。`SyncTransport`はS2のscopeなのでeditorに残している。

拾った罠が2つ。bare cross-package variant spelling（`@editor.CrdtOps`のような書き方）はtransparentなenum aliasを生き残らないので、該当のcall siteは新package側へ寄せる必要がある。そして`moon.pkg`の`import {`へのsedはmain / test / wbtestの3つのblock全部に当たる — こちらは/code-reviewが拾ってくれた。

### js_engine / CI

CIの高速化に取り組んだ。まずcache系を3つ入れた。`_build` artifact cache（#291）、MoonBit toolchainのcache（#292 → [#295](https://github.com/dowdiness/js_engine/pull/295)）、unit-testの2分割並列（#293 → [#296](https://github.com/dowdiness/js_engine/pull/296)）。

ところが実測すると、workflow全体には意味のある短縮が出なかった。unit-testはもともと1分弱で、test262は実行そのもの（30k超のテスト、1 modeあたり約45分）が支配的。しかもrun間の分散が18分もあり、cacheの改善はその中に埋もれてしまう。

そこでshardingへ切り替えた。`--shard N/M`はrunner側に実装済みだったので、CI matrixにshard次元を足して2 modes × 4 shards = 8並列にする。机上の推定では45分 → 11分だったが、実測では約18分。推定には届かなかったものの、半分以下にはなった。report_test262側もshard artifactをmergeできるようにした。このPRはまだopen。
関連PR: [js_engine #297](https://github.com/dowdiness/js_engine/pull/297)

また、toolingのMoonBit移行が完了してv0.3.0のbakeも済んだため、Phase 4として移行用のPythonスクリプト26本を削除した。
関連PR: [js_engine #290](https://github.com/dowdiness/js_engine/pull/290)

### 作業運用

「長い実装計画はCodexが書く」運用が、この2日でも機能した。loom #279の2-PR分割計画とS1のprotocol/wire移動計画はどちらもCodexが書き、orchestrator側の修正は少数（S1で3件）。pre-PR reviewも全PRで回っていて、loomの2本ではtest gap、S1では最初の返答にevidenceがない問題（probeを要求して解消）を拾った。

一方で、調査をfan-outしたExplore agentは4本とも語数上限を超えて返してきた（350-400語の指定に対して500-700語）。boundの指定方法には改善の余地がある。
