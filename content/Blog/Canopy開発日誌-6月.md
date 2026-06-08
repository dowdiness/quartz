---
title: Canopy開発日誌-6月
publish: true
tags: [blogcanopyprojectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-06-09T00:45:56+09:00
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

6月4日は、Rabbita headless UIをCanopyで本当に使えるかを見極める日だった。
Disclosure PoCやdialog spikeで方向を確認し、同時にMoonDspではGraph runtimeのfacade / internal boundaryを切り始めた。

6月5日は、Rabbitaのpatched更新を取り込み、Ideal / Canvasからheadless UI primitiveを実利用し始めた。
Action Menu、ContextMenu、Tabs、TreeViewまで進み、incrではtyped spreadsheetとIncremental TEAの実験も一気に進んだ。

6月6日は、IdealのUI基盤を大きく整理した。
Resizable / Status live-region / CSS de-dupを入れたあと、Tailwind v4への段階的移行を開始し、overlay、toolbar、bottom tabs、panel、inspector、outline resize handleまで小さなsliceで進めた。

6月7日は、Idealの残タスクとprotocolの曖昧さを潰しながら、Canvasを次のsource-backed段階へ戻した。
outline E2E、bridge partial batch、cursor intentの単位名、protocol coordinate docs、MoonBit registry cacheを入れ、その後 `lib/canvas-graph` の抽出まで進めた。

6月8日は、Canvas source-backed graphを本格的にCodeMirror source editorへ寄せた。
source-backed inspector edit、selection remap、CM6 change delta lowering、CodeMirror source panel mountを順に入れ、夜には stable canvas node identity のPR2設計と実装へ入り始めた。

6月9日は、日付が変わった直後の時点ではPR2が進行中。
`NodeId` / `EdgeId` を文字列identityへ寄せ、Loomの `GraphDoc::find_node_by_id` を使う方向で、binding remap shimを消す作業を続けている。

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

## 2026/6/4

### Canopy

この日はRabbita headless UIをCanopy側でどう扱うかの見極めが中心だった。
まずDisclosureのfeasibilityを文書化し、次にnative dialog spikeの結果を残した。

結論としては、Disclosure PoCをそのまま広げるより、実際にCanopy内で需要があるAction Menu、Tabs、TreeView、Resizable、Status / Live Regionを順に切る方がよい、という判断になった。
「headless UIを増やす」ではなく、実利用先があるprimitiveだけを切る方向。
関連PR: [#501](https://github.com/dowdiness/canopy/pull/501)、[#502](https://github.com/dowdiness/canopy/pull/502)

小さな整理として、btree側の `QueryCase` shrink witnessもmethod寄せした。
これは本筋ではないが、後のheadless UIやCanvas workで使うテスト表現を少し読みやすくしている。
関連PR: [#500](https://github.com/dowdiness/canopy/pull/500)

### MoonDsp

MoonDspでは、Graph runtimeの公開面を締める作業を始めた。
DSP facadeの再exportを止め、内部model、template / binding internalsを順に切り出した。

Canopyから見ると、これは外部editor / authoring graphを受けるruntime側の境界整理にあたる。
Canopyがsource-backed graphを育てるなら、MoonDsp側も「authoring representation」と「audio callbackに持ち込まないruntime boundary」を分けておく必要がある。
関連PR: [moondsp #143](https://github.com/dowdiness/moondsp/pull/143)、[#144](https://github.com/dowdiness/moondsp/pull/144)、[#145](https://github.com/dowdiness/moondsp/pull/145)

### js_engine

js_engineではTest262 residualを小さなclusterごとに潰していた。
class / super / generator、iterator / for-of head、TypedArray constructor / locale stringまわりの不一致を直した。

このへんはCanopy本体とは直接つながらないが、MoonBitで複雑なruntimeを積み上げるときの検証運用としてはかなり近い。
関連PR: [js_engine #211](https://github.com/dowdiness/js_engine/pull/211)、[#212](https://github.com/dowdiness/js_engine/pull/212)、[#213](https://github.com/dowdiness/js_engine/pull/213)、[#214](https://github.com/dowdiness/js_engine/pull/214)

## 2026/6/5

### Canopy / Rabbita headless UI

まずRabbitaのpatched更新をCanopyに取り込んだ。
`rabbita-v0.12.4` 系の差分だけでなく、Canopy側のcompat noteやoutline E2Eの安定化も合わせて確認した。
関連PR: [#503](https://github.com/dowdiness/canopy/pull/503)

その後、headless UIの最初の実用primitiveとして `lib/menu` を切った。
Ideal structural-edit overlayをconsumerにして、keyboard handling、focus request、panel / item attrsをbehaviorとして外へ出した。
関連PR: [#509](https://github.com/dowdiness/canopy/pull/509)

Canvasが2つ目のconsumerになれるかも確認した。
最初はCanvas context menuがTypeScript DOM実装だったので、いきなり `@menu` を入れず、MoonBit側のRabbita islandへmenu surfaceを寄せてから利用した。
これで `lib/menu` はIdealだけの抽象ではなくなった。
関連PR: [#513](https://github.com/dowdiness/canopy/pull/513)

続いて、ContextMenuはMenuの上に別primitiveとして切った。
Canvasの右クリックメニューを実consumerにし、trigger anchor、dismissable layer、close focus、viewport positioningを順に足した。
途中で整数丸めしていたtrigger座標は、fractional anchorを保持する形に直した。
関連PR: [#516](https://github.com/dowdiness/canopy/pull/516)、[#522](https://github.com/dowdiness/canopy/pull/522)、[#523](https://github.com/dowdiness/canopy/pull/523)、[#524](https://github.com/dowdiness/canopy/pull/524)、[#525](https://github.com/dowdiness/canopy/pull/525)

TabsとTreeViewもこの日に入った。
TabsはIdeal bottom tabsをconsumerにし、TreeViewはWAI-ARIA tree behaviorとして分けた。
関連PR: [#517](https://github.com/dowdiness/canopy/pull/517)、[#526](https://github.com/dowdiness/canopy/pull/526)

### incr

incrではtyped spreadsheet exampleが大きく進んだ。
まずuser-facing snapshotにlast dynamic dependenciesを出し、formula factsをevaluatorから分け、formula dependency shapeを分類できるようにした。
関連PR: [incr #192](https://github.com/dowdiness/incr/pull/192)、[#193](https://github.com/dowdiness/incr/pull/193)、[#194](https://github.com/dowdiness/incr/pull/194)

そのうえで、bounded traceのbaseline benchmarkと実APIを入れた。
global traceではなく、観測範囲を絞ったtraceをUI / demoから説明できるようにするための土台。
後続でevidence copy、performance workflow、event-trace feasibilityも整理した。
関連PR: [incr #195](https://github.com/dowdiness/incr/pull/195)、[#196](https://github.com/dowdiness/incr/pull/196)、[#202](https://github.com/dowdiness/incr/pull/202)、[#203](https://github.com/dowdiness/incr/pull/203)、[#204](https://github.com/dowdiness/incr/pull/204)

さらにIncremental TEA exampleも始めた。
最初はskeleton、次にlifecycle、Cmd scheduler、最後にwatched Html renderer prototypeまで進めた。
`Runtime::batch` と `Watch` を使い、TEA風のmessage loopをincr上でどう表現するかを試している。
関連PR: [incr #205](https://github.com/dowdiness/incr/pull/205)、[#206](https://github.com/dowdiness/incr/pull/206)、[#207](https://github.com/dowdiness/incr/pull/207)、[#208](https://github.com/dowdiness/incr/pull/208)

### MoonDsp

MoonDspは前日のboundary splitをさらに進めた。
Graph runtime、scheduler、browser internalsを切り出し、browser / worklet ABI contractとparse result error transportを文書化した。

Canopy / MoonDsp連携の観点では、editor側の失敗やparse resultをどうruntimeへ渡すか、どこから先をaudio callbackへ持ち込まないか、という境界が少しずつ具体化している。
関連PR: [moondsp #146](https://github.com/dowdiness/moondsp/pull/146)、[#147](https://github.com/dowdiness/moondsp/pull/147)、[#148](https://github.com/dowdiness/moondsp/pull/148)、[#149](https://github.com/dowdiness/moondsp/pull/149)、[#153](https://github.com/dowdiness/moondsp/pull/153)、[#154](https://github.com/dowdiness/moondsp/pull/154)、[#155](https://github.com/dowdiness/moondsp/pull/155)、[#159](https://github.com/dowdiness/moondsp/pull/159)、[#160](https://github.com/dowdiness/moondsp/pull/160)、[#161](https://github.com/dowdiness/moondsp/pull/161)

### js_engine

js_engineではES2015 residualからTypedArray / Mapまわりまで、Test262 conformanceをかなり進めた。
ArrayBuffer / DataView / TypedArray constructor、integer-index internals、arguments object ArrayIterator、Map iterator receiver / exhaustion semanticsなど。
関連PR: [js_engine #215](https://github.com/dowdiness/js_engine/pull/215)、[#216](https://github.com/dowdiness/js_engine/pull/216)、[#217](https://github.com/dowdiness/js_engine/pull/217)、[#218](https://github.com/dowdiness/js_engine/pull/218)、[#219](https://github.com/dowdiness/js_engine/pull/219)、[#220](https://github.com/dowdiness/js_engine/pull/220)、[#221](https://github.com/dowdiness/js_engine/pull/221)、[#222](https://github.com/dowdiness/js_engine/pull/222)

## 2026/6/6

### Canopy / Ideal UI foundation

この日はIdealのUI基盤をかなり進めた。
まずMenu focusをscope-awareにし、Ideal outline panelへ `lib/resizable` を実統合した。
Resizableはdemoだけでなく、実際のeditor layoutで使われるprimitiveになった。
関連PR: [#528](https://github.com/dowdiness/canopy/pull/528)、[#529](https://github.com/dowdiness/canopy/pull/529)

Canvas向けにはreusableなStatus / Live Region behaviorを切った。
成功 / 失敗 / info toneとpolite announcementをbehaviorとして外へ出し、source-backed Apply statusを最初のconsumerにした。
関連PR: [#531](https://github.com/dowdiness/canopy/pull/531)

CSS面では、Ideal shadow CSSの重複を止めるfoundationを入れた。
`editor-shadow.css` を `adoptedStyleSheets` でsingle-source化し、JS string constantとして持っていた `SHADOW_STYLES` を消した。
この時点で、後続のTailwind移行を判断できる状態になった。
関連PR: [#532](https://github.com/dowdiness/canopy/pull/532)

その後、Tailwind v4への段階的移行を始めた。
最初はaction overlay / name prompt、その後style management docs、toolbar recipe、bottom tabs、panel chrome、inspector detail、outline resize handleへ進めた。

ここでは「一気に全部Tailwind化する」のではなく、既存semantic hookを残しながら、light-DOMの小さなchrome sliceだけを移す方針にした。
`@apply` は使わず、Ideal-localなrecipe helperやclass bundleで管理する。
関連PR: [#534](https://github.com/dowdiness/canopy/pull/534)、[#535](https://github.com/dowdiness/canopy/pull/535)、[#539](https://github.com/dowdiness/canopy/pull/539)、[#540](https://github.com/dowdiness/canopy/pull/540)、[#541](https://github.com/dowdiness/canopy/pull/541)、[#542](https://github.com/dowdiness/canopy/pull/542)、[#543](https://github.com/dowdiness/canopy/pull/543)、[#544](https://github.com/dowdiness/canopy/pull/544)、[#545](https://github.com/dowdiness/canopy/pull/545)

### MoonDsp

MoonDspではGraph facadeのparity guardとruntime boundary inventoryを追加したあと、native CLAP prototypeへ入った。
公式CLAP headerをvendorし、MoonBit bridge header generationを安定化し、CLAP prototypeのCI gateとWindows host load fixまで進んだ。

Canopyから見ると、MoonDspはWeb / authoring graphだけでなく、native plugin hostとの境界も現実の検証対象になってきた。
関連PR: [moondsp #165](https://github.com/dowdiness/moondsp/pull/165)、[#166](https://github.com/dowdiness/moondsp/pull/166)、[#167](https://github.com/dowdiness/moondsp/pull/167)、[#168](https://github.com/dowdiness/moondsp/pull/168)、[#169](https://github.com/dowdiness/moondsp/pull/169)、[#170](https://github.com/dowdiness/moondsp/pull/170)、[#175](https://github.com/dowdiness/moondsp/pull/175)、[#176](https://github.com/dowdiness/moondsp/pull/176)

### loom / incr / js_engine

LoomではMoonBit parser skeleton exampleが入った。
このあと6月7日のMoonBit parser integration連打につながる最初の足場。
関連PR: [loom #220](https://github.com/dowdiness/loom/pull/220)

incrではAcceptedDerived系の設計が進んだ。
まずdesign specを置き、state-machine spikeで受け入れ済み値をどう保持するかを検証している。
関連PR: [incr #213](https://github.com/dowdiness/incr/pull/213)、[#214](https://github.com/dowdiness/incr/pull/214)

js_engineではMap / module graph / tooling migrationの整理が進んだ。
Map.groupByやMap.prototype.forEach residualを直し、ES module graph live bindingsを改善し、Test262 runnerやstdlib descriptor helperの整理も進めた。
関連PR: [js_engine #223](https://github.com/dowdiness/js_engine/pull/223)、[#224](https://github.com/dowdiness/js_engine/pull/224)、[#225](https://github.com/dowdiness/js_engine/pull/225)、[#226](https://github.com/dowdiness/js_engine/pull/226)、[#227](https://github.com/dowdiness/js_engine/pull/227)、[#246](https://github.com/dowdiness/js_engine/pull/246)、[#247](https://github.com/dowdiness/js_engine/pull/247)、[#259](https://github.com/dowdiness/js_engine/pull/259)

## 2026/6/7

### Canopy / Ideal and protocol

Ideal Tailwind側ではoutline peer / empty-state chromeを移した。
これで6月6日から続いていたTailwind migrationの最初のまとまった山は一段落した。
関連PR: [#546](https://github.com/dowdiness/canopy/pull/546)

その後は非Tailwindの残タスクへ戻った。
outline tree E2Eの穴を埋め、`bridge.ts::applySpliceChanges` のpartial batch broadcast semanticsを直した。
関連PR: [#553](https://github.com/dowdiness/canopy/pull/553)、[#554](https://github.com/dowdiness/canopy/pull/554)

protocol側では、`UserIntent.SetCursor(position)` の曖昧さを消した。
PM tree positionとCM doc code-unit offsetがどちらも `position` という名前で流れていたので、`SetPmCursor(pm_tree_position)` と `SetDocCursor(doc_code_unit_offset)` に分けた。

続けて、protocol coordinate unitsとwire-format change disciplineも文書化した。
この種の単位名は後から効いてくるので、早めに潰せてよかった。
関連PR: [#555](https://github.com/dowdiness/canopy/pull/555)、[#558](https://github.com/dowdiness/canopy/pull/558)

CIではMoonBit registry stateをcacheし、`moon-update.sh` のdiagnosticsを改善した。
前日までのCI flakeを「再実行すればよい」ではなく、少し観測しやすい形へ寄せている。
関連PR: [#560](https://github.com/dowdiness/canopy/pull/560)

### Canvas

夕方からCanvasのgraph model extractionへ戻った。
これまで `examples/canvas/main` のUI stateやupdateに混ざっていたdurable graph operationを、`lib/canvas-graph` へ切り出した。

`WorkflowAction`、`GraphOperation`、`CanvasState`、reducer APIがlibrary側に寄ったことで、source-backed loweringやhand-built canvasが同じmodelを使いやすくなった。
これは6月8日のsource-backed edit / CodeMirror source panel workの前提になっている。
関連PR: [#562](https://github.com/dowdiness/canopy/pull/562)

### loom

Loomはこの日かなり大きく進んだ。
まずzero-width lexer token reuse boundariesまわりを固めた。
located token adapter、token provenance offsets、zero-width boundary docs、synthetic zero-width hookのparser-owned化、property test、benchmark、RepeatGroup canonicalizationの修正と文書化まで一気に進んでいる。
関連PR: [loom #221](https://github.com/dowdiness/loom/pull/221)、[#229](https://github.com/dowdiness/loom/pull/229)、[#230](https://github.com/dowdiness/loom/pull/230)、[#231](https://github.com/dowdiness/loom/pull/231)、[#233](https://github.com/dowdiness/loom/pull/233)、[#234](https://github.com/dowdiness/loom/pull/234)、[#235](https://github.com/dowdiness/loom/pull/235)、[#237](https://github.com/dowdiness/loom/pull/237)、[#240](https://github.com/dowdiness/loom/pull/240)、[#242](https://github.com/dowdiness/loom/pull/242)、[#246](https://github.com/dowdiness/loom/pull/246)

MoonBit parser integrationも進んだ。
full MoonBit token syntax kinds、top-level headers、differential fixtures、syntax-only reactive parser、ParserContext grammar helpersまで入った。

Canopy側のCodeMirror / language toolingを考えると、Loomが単なるparser runtimeではなく、editor-facingなsyntax artifactを出せる方向へ寄ってきている。
関連PR: [loom #236](https://github.com/dowdiness/loom/pull/236)、[#239](https://github.com/dowdiness/loom/pull/239)、[#241](https://github.com/dowdiness/loom/pull/241)、[#247](https://github.com/dowdiness/loom/pull/247)、[#248](https://github.com/dowdiness/loom/pull/248)、[#249](https://github.com/dowdiness/loom/pull/249)、[#250](https://github.com/dowdiness/loom/pull/250)

### MoonDsp / js_engine / incr

MoonDspではCLAP audio callbackのallocation auditを行い、その後callback allocationsを除去した。
real host coverage checklistも追加され、CLAP prototypeが「作った」から「host上でどう検証するか」へ進んだ。
関連PR: [moondsp #177](https://github.com/dowdiness/moondsp/pull/177)、[#178](https://github.com/dowdiness/moondsp/pull/178)、[#179](https://github.com/dowdiness/moondsp/pull/179)、[#181](https://github.com/dowdiness/moondsp/pull/181)

js_engineではArray method fast path delegationとMoonBit Test262 runner shadowが進んだ。
iterator / copy / push-pop系のfast pathを分け、Python toolingからMoonBit shadowへ移す準備も入っている。
関連PR: [js_engine #264](https://github.com/dowdiness/js_engine/pull/264)、[#265](https://github.com/dowdiness/js_engine/pull/265)、[#266](https://github.com/dowdiness/js_engine/pull/266)、[#267](https://github.com/dowdiness/js_engine/pull/267)、[#268](https://github.com/dowdiness/js_engine/pull/268)、[#271](https://github.com/dowdiness/js_engine/pull/271)、[#273](https://github.com/dowdiness/js_engine/pull/273)、[#274](https://github.com/dowdiness/js_engine/pull/274)、[#275](https://github.com/dowdiness/js_engine/pull/275)、[#276](https://github.com/dowdiness/js_engine/pull/276)、[#277](https://github.com/dowdiness/js_engine/pull/277)、[#278](https://github.com/dowdiness/js_engine/pull/278)

incrではtyped spreadsheetのAI / tool支援とAD / RAD方向のissue整理を行った。
実装としては、formula AST accessorが入り、次のAI context exportへつながる準備ができた。
関連PR: [incr #224](https://github.com/dowdiness/incr/pull/224)

## 2026/6/8

### Canvas

日付が変わったあと、source-backed canvas node renameとnumeric parameter editsをGraph UIから行えるようにした。
renameや数値param editは、UI stateを直接いじるのではなくGraph DSL sourceへlowerし、reparseされたsource-backed graphを再表示する。
関連PR: [#563](https://github.com/dowdiness/canopy/pull/563)

続けて、source-backed selectionをsource reorder後にも保つ暫定fixを入れた。
この時点では、`NodeId` がまだdoc-order由来なので、bindingを使って選択をrecoverする方向。
後でstable identityへ進むまでの橋渡しになった。
関連PR: [#564](https://github.com/dowdiness/canopy/pull/564)

次に、CodeMirrorのchange deltaをLoom edit列へ下ろす経路を作った。
`lib/rabbita_codemirror` の `listen()` にadditiveな `on_change?: Emit[Array[ChangeDelta]]` を足し、JS側の `iterChanges` をMoonBit delta payloadへ変換する。
Canvas側では、それをordered Loom editsに変換して `apply_edit` へ渡すようにした。
関連PR: [#569](https://github.com/dowdiness/canopy/pull/569)

最後に、Canvas source panelをtextareaではなくCodeMirrorにした。
このPRで `?source=1` のsource panelは、CodeMirror bufferをauthoritative sourceとして扱うようになった。

途中でdirty edit recoveryの設計を直し、rollbackではなく「editor bufferが常に真、parse成功時はcurrent_result、失敗時はlast_goodを表示」という形へ寄せた。
`/code-review` ではV5 / V8 / V10の3件を同じPRにfoldし、CI green後にmergeされた。
関連PR: [#570](https://github.com/dowdiness/canopy/pull/570)

この流れで、stable canvas node identityの実行specも固まった。
PR1はCodeMirror source mountまで、PR2で `NodeId` / `EdgeId` をstable semantic identityへ切り替える、という分割。
作業ブランチ: `feat/565-pr2-stable-canvas-node-identity`

### loom

Loomでは、Monogramから着想を得たparser-backed role span方向の最初のsliceが進んだ。
JSON syntax role span projection、parser-backed `@incr` attachment、editor-neutral exportを順に追加した。

まだ共有 `LanguageModel` APIを切る段階ではなく、JSON-localに閉じたsliceで、Canopy / CodeMirrorが消費できるrole spanをどう出すかを見ている。
関連PR: [loom #263](https://github.com/dowdiness/loom/pull/263)、[#265](https://github.com/dowdiness/loom/pull/265)、[#266](https://github.com/dowdiness/loom/pull/266)

夜にはCST traversal idioms guideとmetadata-trust invariant property testも入った。
これは、今後agentや人間がCST metadataを雑に信じすぎないためのguardとして効く。
関連PR: [loom #270](https://github.com/dowdiness/loom/pull/270)

### incr

incrではAcceptedDerived方向が一気に進んだ。
public facade、checked examples、BackdateEq acceptance tierを入れ、CIもlibrary + docs examplesを走らせる形へ強化した。
関連PR: [incr #215](https://github.com/dowdiness/incr/pull/215)、[#226](https://github.com/dowdiness/incr/pull/226)、[#228](https://github.com/dowdiness/incr/pull/228)、[#231](https://github.com/dowdiness/incr/pull/231)、[#232](https://github.com/dowdiness/incr/pull/232)

typed spreadsheet側では、AI context exportも入った。
selected cell、dependencies、bounded trace、formula ASTなどをdeterministicに取り出すためのsurface。
関連PR: [incr #225](https://github.com/dowdiness/incr/pull/225)

さらに、diamond dependencyで `push_reachable_count` がずれる問題を直し、Salsaとのengine comparison noteも追加した。
関連PR: [incr #233](https://github.com/dowdiness/incr/pull/233)、[#234](https://github.com/dowdiness/incr/pull/234)

### js_engine / MoonDsp

js_engineでは、Test262 runnerのMoonBit shadow化をさらに進めた。
Array shift / unshift fast path delegation、runner parity、shadow artifact parity、CI artifactsまでつなげている。
関連PR: [js_engine #279](https://github.com/dowdiness/js_engine/pull/279)、[#280](https://github.com/dowdiness/js_engine/pull/280)、[#281](https://github.com/dowdiness/js_engine/pull/281)、[#282](https://github.com/dowdiness/js_engine/pull/282)、[#283](https://github.com/dowdiness/js_engine/pull/283)

MoonDspでは、PatternDocの `BackdateEq` と、loom-mini-cstのlast-good projection実験を始めた。
parse error時にも前回成功したprojected docを出せるようにする方向。
関連ブランチ: `feat/pattern-backdate-eq`、`feat/loom-mini-cst-last-good`

## 2026/6/9

### Canopy

日付が変わった時点では、stable canvas node identityのPR2を進めている。
これは、source-backed canvasの `NodeId` をdoc-order由来の `Int` から、Loomの `ProjectionIdentityTracker` が出す `GraphNode::id()` 文字列へ寄せる作業。

実行addendumでは、token formatは `node#<occurrence>:<binding>` と確認した。
`set_source` / `apply_edit` で別binding lineをdeleteした場合、survivor tokenはstable。
一方で、atomicな `set_source` reorderだけはtokenがchurnする。

この結果から、deleteを無理にdelta変換する必要はないと判断した。
user gestureとしてのCodeMirror reorderはdelta delete + insertで入り、既存のdelta pathならidentityを保てる。
`set_source` はreset / init寄りのpathとして扱い、reorder churn時にはselectionをremapせず自然にdropする。

PR2の中心は、binding-recovery shimを消すこと。
`source_selected_node_bindings`、`source_selection_from_bindings`、`remap_selection_to_bindings` のようなcapture / replayではなく、`prune_selection_to_doc(doc)` で「今のdocに存在するtokenだけ残す」形へ寄せている。

renameだけはbindingがtokenに含まれるため、old tokenからnew tokenへselectionを書き換える局所hookを残す。
一方で、`action_log` は過去表示用なのでrewriteしない。

`EdgeId` もallocatorではなく、`(source_token, source_port, target_token, target_port)` の4-tupleから導出する方針。
hand-built canvasだけは `next_node_id` を文字列化したmint counterとして残すが、source-backed canvasとは同じ `CanvasState` を共有しないので衝突しない。

現在のworktreeはまだ未mergeの作業中。
`lib/canvas-graph`、`graph_dsl_adapter`、Canvas runtime / TS adapterまで `Int` → `String` のcascadeが入り、source-backed selection testはまだ仕上げ中。
作業ブランチ: `feat/565-pr2-stable-canvas-node-identity`

### loom / MoonDsp

このPR2のために、Loom側では `GraphDoc::find_node_by_id(String)` を追加した。
Canopy側の `graph_node_for_canvas_id` は、`nodes[raw - 1]` のようなpositional lookupではなく、このtoken lookupを使う予定。
関連コミット: `feat(graph-dsl): add GraphDoc::find_node_by_id for token-identity lookup`

MoonDsp側では、loom-mini-cstのlast-good projectionを進めた。
parse error時にlast-good projected docを返す実装を入れ、value-collision premiseやempty-accept caseもtestで固定した。
その後、BackdateEq / last-good testsをmatch / guard / map寄りに整理している。
関連ブランチ: `feat/loom-mini-cst-last-good`

