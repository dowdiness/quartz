---
title: Canopy開発日誌-6月
publish: true
tags: [blogcanopyprojectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-06-06T15:41:03+09:00
---

# Canopy開発日誌-6月

2026年6月のCanopy開発ログ。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

6月1日は、5月末に進めたLambdaのscope graphやgo-to-definitionを、実際の構造編集へ近づける作業が中心だった。
同時に、CanopyとMoonDspの関係、性能改善の優先度、エージェントの使い分けも整理した。

6月2日は、午前にLambda projectionの命名とidentity整理を締め、午後からincr graph visualizerとcanvas graph demoを進めた。
CIのノイズ対策と、Codexによるreview / 実装計画の使い分けも少し固まった。

6月3日は、MoonDsp / Canopy全体方針をmainへ入れ、source-backed canvas demoをブラウザで触れるところまで進めた。
CIではIdeal web E2EをPR gateに載せ、incr側ではpublic event APIの命名をDerived寄りに整理した。

## 2026/6/1

### Canopy

まず、CanopyとMoonDspの長期的な接続構想を整理した。
MoonDspは別リポジトリの音楽DSL / DSPエンジンなので、Canopyへすぐ統合する対象ではない。

今は、Canopyは構造編集の土台、MoonDspは音楽DSL側の実験、incr / Loomは共有基盤として分けて考えることにした。
関連PR: [#445](https://github.com/dowdiness/canopy/pull/445)

Lambda projectionでは、小さな整理を先に進めた。
`ProjNode` の構築helper、SourceMap token helper、Block / Holeのtyped viewなどを使い、projection処理の重複を減らした。

その後、module-level bindingをfirst-classな `LetDef` projection nodeとして扱う変更を入れた。
以前はbinding rowがinit expressionの `NodeId` やsynthetic idを借りていたので、Structure modeのdrag/dropやbinding-level editで責務が曖昧だった。

`LetDef` が実際のnodeになったことで、binding rowを構造編集の対象として扱いやすくなった。
binderの位置取得は、引き続き `@scope.binder_span` / `@scope.go_to_definition` が担当する。
関連PR: [#437](https://github.com/dowdiness/canopy/pull/437)、[#438](https://github.com/dowdiness/canopy/pull/438)、[#439](https://github.com/dowdiness/canopy/pull/439)、[#440](https://github.com/dowdiness/canopy/pull/440)、[#446](https://github.com/dowdiness/canopy/pull/446)、[#448](https://github.com/dowdiness/canopy/pull/448)

性能面では、BAND 2bのevidence gateを走らせた。
ここでは最適化を書かず、まず本当に遅いのかを測った。

`to_flat_proj_incremental` は1000 defsで数msまで伸びることが分かった。
ただし、実際のCanopy内のLambda documentはまだそこまで大きくない。

そのため、この最適化は一旦parkした。
大きなdocumentやimport機能が出てきたら、改めて取り組む。
関連PR: [#447](https://github.com/dowdiness/canopy/pull/447)

依存関係まわりでは、rle submoduleの更新と、examples配下のVite / Playwright / React系の更新も行った。
PlaywrightはpackageだけでなくCI containerも合わせる必要があった。
関連PR: [#435](https://github.com/dowdiness/canopy/pull/435)、[#280](https://github.com/dowdiness/canopy/pull/280)

### loom

Loomでは `examples/json-settings` に、last-good semantic projection attachmentのchecked exampleを追加した。
docsにあったtemplateを、実際にテストできるexampleへ落とした形。

ポイントは、`@incr.Derived` のcomputeをpureに保ち、last-goodの保持を外側の `settle` に任せたこと。
`Watch::read()` の `ReadError` も、parser/projection failureとは分けて `GraphBlocked` として扱うようにした。
関連PR: [loom #206](https://github.com/dowdiness/loom/pull/206)

Lambda example側ではBlock / Holeのtyped syntax viewを追加し、その後 `LetDef` term wrapperを入れた。
Canopyのfirst-class `LetDef` projection nodeはこのLoom側の変更を受けて進んでいる。
関連PR: [loom #207](https://github.com/dowdiness/loom/pull/207)、[loom #209](https://github.com/dowdiness/loom/pull/209)

incr dependencyも整理した。
Loomの3 consumerをregistryの `dowdiness/incr@0.7.0` へ寄せた。

Canopy側の追従も見たが、loom配下のincr submoduleに別作業が残っていたので一旦pauseした。
関連PR: [loom #208](https://github.com/dowdiness/loom/pull/208)

### js_engine

js_engineでは、benchmarkを読みやすくする作業を進めた。
focused repeat benchmark runnerを追加し、複数回の測定からmedianやCVを見られるようにした。

単発の結果ではなく、ノイズを見ながら判断するための道具になっている。
関連PR: [js_engine #186](https://github.com/dowdiness/js_engine/pull/186)、[js_engine #187](https://github.com/dowdiness/js_engine/pull/187)

startup側では、`new_interpreter` の中を分けて測り、final realm stampingを軽くした。
runtimeで作られるiterator callbackやPromise callbackにもrealmを付ける整理をした。
関連PR: [js_engine #188](https://github.com/dowdiness/js_engine/pull/188)、[js_engine #192](https://github.com/dowdiness/js_engine/pull/192)

Arrayまわりでは、`sort` や `splice` などのmutatorで、明示的な `undefined` とholeの扱いがずれる問題を直した。
genericなArray mutatorでは、Proxy receiverを考慮して `HasProperty` 相当の経路を使う必要がある、というメモも残した。
関連PR: [js_engine #191](https://github.com/dowdiness/js_engine/pull/191)

### 作業運用

Claude / Codex / piの使い分けも整理した。

長い実装計画はCodexが書き、Claude/Opusは設計と実行管理を持つ。
Codexはpre-PR reviewだけでなく、実装順序や検証点を含む計画を書く役割も持つ。

この運用はLoomのjson-settings exampleで最初に使い、`GraphBlocked` stateのような設計漏れも拾えた。

## 2026/6/2

### Canopy

まず、BAND 2bのJS target addendumを追加した。
`to_flat_proj_incremental` のcliffはNode上でも再現し、1000 defsのtailで約4msまで伸びた。

一方で、原因の説明は修正した。
当初疑っていたper-positionの `start()` / `cst_node()` walkではなく、reuseされたdef subtree同士の `CstNode==` とcache-boundなpointer chasingが支配的だった。

ここでも最適化コードは入れていない。
cross-parse interningとhash-only reuseというfix leverは残しつつ、実際のdocument scaleが必要になるまではparkする判断にした。
関連PR: [#451](https://github.com/dowdiness/canopy/pull/451)

Lambda projectionでは、`FlatProj` を `ModuleProjection` に改名した。
これは単なる名前の置換ではなく、「Moduleのlet defsとfinal exprをflattenした、incremental projection diffの単位」という責務を表に出すための整理。

同時に、ReuseCursor、ProjectionIdentityTracker、`to_module_projection_incremental` の3つのidentity / reuse mechanismをADRに分けて記録した。
`#396` のsource-span tensionや、BAND 2bのparkした最適化がclarity refactorではなく設計トレードオフを持つこともここに残した。

依存関係は `dowdiness/incr@0.7.1` とLoom側の追従へ寄せた。
review後には、full delete後に古いmodule root `NodeId` を再利用しうるcache reset漏れも直している。
関連PR: [#452](https://github.com/dowdiness/canopy/pull/452)

microbenchmark側では、本番と同じようにID sourceを前へ進めるよう修正した。
古いprojectionを作ったあと、そのhigh-water markからincremental callを再開する形。
これは性能改善ではなく、benchmark setupが本番のID割り当てとずれないようにする修正。
関連PR: [#453](https://github.com/dowdiness/canopy/pull/453)

### Incr visualizer

IdealのIncrGraphは、topologyを見るだけのpanelから、recomputeの状態とcostも見えるpanelへ少し進んだ。

まず、`IncrMemoEventTap` の一時event bufferをappend-onlyではなく「cellごとに最後のeventだけ残す」形にした。
status coloringはもともと各cellの最新eventしか使っていなかったので、bufferをdistinct cell数でboundできる。
関連PR: [#462](https://github.com/dowdiness/canopy/pull/462)

その上で、各cellの最新recompute時間をnode detailへ出すようにした。
`ns` / `us` / `ms` の単位を切り替えて表示するので、遅いcellが周囲から浮いて見える。
Codexのreview履歴でも、この変更はstatus pathやJS targetのInt64 formattingを確認したうえでPASSしていた。
関連PR: [#465](https://github.com/dowdiness/canopy/pull/465)

UI側には、IncrGraph panelの色の意味を示すlegendを追加した。
Idle / Recomputing / Changed / Failedの4つで、SVG rendererと同じ `VisualNodeStatus::stroke` / `::fill` を使う。
関連PR: [#469](https://github.com/dowdiness/canopy/pull/469)

命名も整理した。
tapがMemoだけでなくDerivedのrecomputeも見ているので、`IncrMemoEvent*` を `Recompute*` 系へ改名し、snapshot側も `RecomputeSnapshot*` に揃えた。
夜のCodexメモでは、incr本体側の `MemoEvent` / `DerivedEvent` 命名移行も検討していて、Canopy側のvisualizer名をrecompute中心に寄せる流れとつながっている。
関連PR: [#471](https://github.com/dowdiness/canopy/pull/471)、[#475](https://github.com/dowdiness/canopy/pull/475)

### Canvas

Canvas exampleでは、graph DSLとUIの接続を進めた。

まず、Graph DSL sourceをroundtripするsmoke testを追加した。
ここでLoom submoduleも進め、canvas側がsource-backed graphを扱う準備を始めた。
関連PR: [#466](https://github.com/dowdiness/canopy/pull/466)

次に、canvasのgraph modelを `examples/canvas/graph_model` へ切り出した。
巨大化していた `canvas_state` / `canvas_update` から、graph operationの責務を分けた形。
同時に、web側には `GraphAdapter` lifecycleのstubを置いた。
関連PR: [#474](https://github.com/dowdiness/canopy/pull/474)

その後、graph UI stateをincrで分け、`canvas_runtime` を作った。
状態更新を直接UI stateに詰め込むのではなく、graph model、runtime、adapterの境界を少し見えるようにした。
関連PR: [#477](https://github.com/dowdiness/canopy/pull/477)

最後に、source-backed graph adapterを追加した。
source graphからrealized node idを取り出してlogできるようになり、hard-codedなgraph demoから、sourceを背後に持つdemoへ近づいた。
関連PR: [#476](https://github.com/dowdiness/canopy/pull/476)

### CI / 作業運用

CIでは、editor-response benchmarkと `moon update` の2つを安定化した。

editor-responseは、CI runner上の単発p95 paintがflakeしやすかった。
一度CI-only budgetを175msへ緩めたあと、10% trimmed meanを主なgateにして、keystroke数も30から40へ増やした。
一時的なspikeではなく、全paintが遅くなる本物のregressionを拾うための変更。
関連PR: [#459](https://github.com/dowdiness/canopy/pull/459)、[#463](https://github.com/dowdiness/canopy/pull/463)

`moon update` は、mooncakes CDNの一時的な403で落ちることがあった。
そのため、transientな403 / 429 / 5xx / DNS / connection errorだけをbounded retryする `scripts/moon-update.sh` を追加した。
Codex reviewでは、最初のretry判定が広すぎる点と設定値validation漏れがfindingになり、修正後にPASSしている。
関連PR: [#468](https://github.com/dowdiness/canopy/pull/468)

さらに、残っていたbare `moon update` call siteをwrapper経由へ寄せ、再発防止のguardも追加した。
MakefileやCloudflare build-deploy scriptまでscan対象を広げたことで、PR CIだけでなくdeploy path側も同じretry policyに乗った。
関連PR: [#470](https://github.com/dowdiness/canopy/pull/470)、[#473](https://github.com/dowdiness/canopy/pull/473)

## 2026/6/3

### Canopy

まず、Ideal web E2EをCIのPR gateに乗せた。
これまで `editor-response.perf.spec.ts` はbenchmark workflowで見ていたが、通常のIdeal editor E2E suiteはPR gatingとして独立していなかった。

`scripts/test-ideal-web-e2e.sh` を追加し、非performanceのPlaywright specをまとめて走らせるようにした。
`moon update` は前日のretry wrapperを使い、Vite側のMoonBit JS buildでは既知のworkspace target問題を避けるため `MOON_WORK=off` を明示している。
関連PR: [#478](https://github.com/dowdiness/canopy/pull/478)

次に、MoonDsp + Canopy ecosystem visionと、BAND 1-2 execution specをmainへ入れた。
この日誌冒頭に置いた全体方針リンクがその文書。

内容としては、operations-as-dataを中核に、Canopy、MoonDsp、Loom、incrをどう分担させるかを整理している。
Canopyは構造編集とprojection、MoonDspはDSP runtime、Loomはsource-backed projection、incrは共有のincremental substrateという見取り図。

この文書はmulti-agent verify passとCodex design-reviewを通して、8件の指摘を取り込んだうえで入った。
関連PR: [#445](https://github.com/dowdiness/canopy/pull/445)

Canvas exampleでは、6月2日に追加したsource-backed graph adapterをブラウザdemoとして触れるところまで進めた。
`?source=1` でsource-backed modeに入り、左側のGraph DSL sourceをcanonical backing storeとして使う。

このdemoでは、node dragはlocal layoutだけを変え、source本文は変えない。
一方でhandle同士をつなぐcanvas gestureや、`Connect osc -> meter` / `Insert reverb` の操作はcanonical sourceへlowerされる。

Playwright E2Eでは、dragがsourceを汚さないこと、canvas gestureが `meter = scope(input: osc)` へ反映されること、source editorから4 node / 2 edgeの状態へreparseできることを確認している。
作業ブランチ: `feat/source-backed-canvas-demo`

### MoonDsp

MoonDsp側では、Canopyのような外部editorがaudio previewを渡すときのhandoff contractを文書化した。
parser、projection、lowering、template analysis、compile、hot-swap準備はeditor / control thread側で行い、audio callbackには持ち込まない、という境界を明確にした。

Preview stateも、単なる再生可否ではなく `Idle` / `Analyzing` / `Ready` / `UsingLastGoodTemplate` / `Failed` として扱う。
失敗した最新editがあっても、前のready runtimeがあるならlast-good previewを鳴らし続ける設計。
関連PR: [moondsp #125](https://github.com/dowdiness/moondsp/pull/125)

さらに、external authoring graphをMoonDspへ渡すコストを見るbenchmarkも追加した。
まず基本形を入れ、その後により現実的なauthoring shapeを追加している。

これはCanopy側のsource-backed graph demoと同じ問題を、MoonDsp runtime側の受け口から測るための土台。
関連PR: [moondsp #126](https://github.com/dowdiness/moondsp/pull/126)
作業ブランチ: `issue-127-realistic-external-authoring-benchmarks`

### incr

incrでは、public event APIの名前を `MemoEvent` から `DerivedEvent` へ寄せた。
実際にはMemoだけでなくDerived recompute eventを表していたため、6月2日のCanopy visualizer側の `Recompute*` 命名整理とも方向が揃っている。

古い `MemoEvent` / `Runtime::on_memo_event` はdeprecated alias / forwarding methodとして残し、外部consumerのsource compatibilityは保った。
ただしversion bumpとmooncakes publishはまだ行わず、`0.8.0` 向けの変更としてUnreleasedに置いている。
関連PR: [incr #171](https://github.com/dowdiness/incr/pull/171)

Codex reviewでは、最初にchecked examplesやroadmap docsに古い名前が残っていることを検出していた。
その後、compat testとdocs更新を入れてからmergeされた。

並行して、`Input::id` やruntime identity surfaceの設計レビューも行った。
こちらはまだ実装ではなく、single-cell handleだけにidentityを出す方針と、`RuntimeId` wrapperを切る案を確認した段階。

