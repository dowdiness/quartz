---
title: Canopy開発日誌-6月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-06-17T23:10:00+09:00
---

# Canopy開発日誌-6月

2026年6月のCanopy開発ログ。細かいPR単位の列挙よりも、作業の流れが追いやすいように要点だけを残す。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

## 今月の大きな流れ

6月前半は、Canopyを「構造編集の実験場」から、より明確なプロダクトとライブラリ群へ整理する作業が中心だった。

- **Lambda編集**: scope graph、block-local binding、CstFold現代化、`ModuleProjection`削除、alpha-safe beta pilot、Grove Level 1 identity hintまで進んだ。
- **Ideal / Canvas**: Rabbita headless UI、Tailwind v4、source-backed canvas、CodeMirror source editor、FFI file I/Oを順に入れた。
- **Loom**: Lambda側の土台に加え、Markdown incremental block reparseとMarkdownIRの設計が進んだ。
- **incr**: Incremental TEA prototypeが、renderer lifecycle、keyed diff、subscriptions、inactive root、activation policyまで進んだ。
- **js_engine**: architecture refactorとbytecode fast path、ES2024 Set methods、test262/benchmark基盤の整備が進んだ。
- **MoonDsp**: Canopyとは直接統合せず、共有基盤を見ながらGraph runtimeやschedulerの責務分割を進めた。

## 2026/6/1

### Canopy

CanopyとMoonDspの関係を整理した。Canopyは構造編集の土台、MoonDspは音楽DSL / DSP側の実験、incr / Loomは共有基盤として扱う方針にした。

Lambda projectionでは、`ProjNode`構築helper、SourceMap token helper、Block / Holeのtyped viewを入れたうえで、module-level bindingをfirst-classな`LetDef` projection nodeとして扱えるようにした。binding rowを構造編集の対象として扱いやすくするための一歩だった。

性能面では、`to_flat_proj_incremental`のBAND 2b evidence gateを実行した。1000 defsで数msまで伸びることは分かったが、実際の文書規模ではまだ問題にならないため、最適化はparkした。

### loom / js_engine / 運用

Loomではjson-settings exampleにlast-good semantic projection attachmentを追加し、Lambda exampleにはBlock / Hole / LetDef系のtyped viewを入れた。js_engineではrepeat benchmark runnerとArray mutatorのhole / `undefined`まわりを整理した。

運用面では、Codexをpre-PR reviewだけでなく実装計画の書き手として使う流れが固まり始めた。

## 2026/6/2

### Canopy

前日の性能調査をJS targetでも確認した。原因は単純なwalkではなく、reuseされたCstNode比較とcache由来のpointer chasingが支配的だった。ここでも最適化は見送り、document scaleが必要になってから扱うことにした。

Lambda projectionでは`FlatProj`を`ModuleProjection`へ改名し、ReuseCursor / ProjectionIdentityTracker / incremental projectionの設計トレードオフをADRに分けて記録した。microbenchmarkも、本番と同じID sourceの進め方に合わせた。

### Incr visualizer / Canvas

IncrGraph panelは、topologyだけでなくrecompute状態とコストも見えるようにした。cellごとの最新eventだけを持つbuffer、最新recompute時間、legendを追加した。

Canvasでは、graph demoをより構造編集寄りに使うための準備を進めた。

### CI / 運用

Ideal web E2EをPR gateへ載せるための準備と、CIノイズをどう扱うかの整理を進めた。

## 2026/6/3

### Canopy

MoonDsp / Canopyの全体方針をmainへ入れた。source-backed canvas demoも、source graphをブラウザで触れるところまで進めた。

CIではIdeal web E2EをPR gateに載せた。CanopyのUI変更を、ローカル手動確認だけでなくCI上のブラウザテストで支える方向へ進んだ。

### MoonDsp / incr

MoonDspでは、Canopy連携を見据えながらGraph runtimeの境界を整理した。audio callbackへeditor側の責務を持ち込まない、という境界が少しずつ明確になった。

incr側ではpublic event APIの命名をDerived寄りに整理した。

## 2026/6/4

### Canopy

Rabbita headless UIをCanopyで本当に使えるかを見極める日だった。Disclosure PoCとdialog spikeで、RabbitaをIdeal UIの土台にできる感触を確認した。

### MoonDsp / js_engine

MoonDspではGraph runtimeのfacade / internal boundaryを切り始めた。js_engineではArray method fast path delegationやTest262 runner shadowの準備が進んだ。

## 2026/6/5

### Canopy / Rabbita headless UI

Rabbitaのpatched更新を取り込み、IdealとCanvasからheadless UI primitiveを実際に使い始めた。Action Menu、ContextMenu、Tabs、TreeViewまで進み、Ideal UIの土台がRabbitaへ寄っていった。

### incr / MoonDsp / js_engine

incrではtyped spreadsheetとIncremental TEAの実験が進んだ。MoonDspではeditor / audio runtimeのhandoff contractを文書化し、js_engineではArray mutatorやrunner shadow化を進めた。

## 2026/6/6

### Canopy / Ideal UI foundation

IdealのUI基盤を大きく整理した。Resizable、Status live-region、CSS de-dup、Tailwind v4移行、overlay、toolbar、bottom tabs、panel、inspector、outline resize handleを小さなsliceで進めた。

### 周辺リポジトリ

MoonDspではGraph runtime / scheduler / browser internalsの分割が進んだ。Loom、incr、js_engineでも、それぞれparser runtimeやIncremental TEA、test262 runnerの整備が続いた。

## 2026/6/7

### Canopy / Ideal and protocol

Idealの残タスクとprotocolの曖昧さを潰した。outline E2E、bridgeのpartial batch、cursor intentの単位名、protocol coordinate docs、MoonBit registry cacheを入れた。

### Canvas / loom

Canvasは次のsource-backed段階へ戻した。`lib/canvas-graph`の抽出も進めた。

LoomではMoonBit parser integrationが進み、editorへ渡せるsyntax artifactを出せる方向へ寄っていった。

## 2026/6/8

### Canvas

Canvasのsource-backed graphをCodeMirror source editorへ寄せた。source-backed inspector edit、selection remap、CM6 change delta lowering、CodeMirror source panel mountを順に入れた。

dirty edit recoveryは、rollbackではなく「editor bufferが常に真。parse成功時はcurrent result、失敗時はlast-goodを表示」という形にした。

### loom / incr / js_engine

Loomではparser-backed role span、incrではIncremental TEAのkeyed DOM benchmark、js_engineではArray shift / unshift fast pathやrunner parityが進んだ。

## 2026/6/9

### Canopy

Canvas stable identityのPR2を進めた。`NodeId` / `EdgeId`をsource-backedな文字列identityへ寄せ、binding remap shimを消す方向へ進めた。

Lambda側では、generic projection memosへ向かう前段として、scope graphやprojection identityの整理が続いた。

### loom / MoonDsp

Loomではparser runtime attachmentやdeprecated syntax移行が進んだ。MoonDspではmini parser置き換えcampaignが進み、loomへの置き換え方針が具体化した。

## 2026/6/10

### incr / Incremental TEA

Incremental TEAを一気に仕上げた。renderer lifecycle、keyed VDOM diff、benchmark、subscriptionsがmergeされ、prototypeの主要issueがすべて閉じた。

### MoonDsp / loom / Canopy / js_engine

MoonDspはloom parser置き換えcampaignのPhase 2 parityを完走し、ADR-0016をAcceptedにした。Loomではseparated-listやattachment系が進み、CanopyではCanvasのruntime seam整理が続いた。js_engineはv0.3.0をリリースした。

## 2026/6/11

### loom

separated-list（#279）とgroup shape helpers（#196）を畳んだ。Loom側のparser / syntax helperが、Canopyの各言語実装を支えやすい形になってきた。

### Canopy / アーキテクチャ再設計

Canopyのアーキテクチャ再設計に着手した。S0のproposalとAPI boundary ADRを入れ、S1としてprotocol / wireを抽出した。

### js_engine / 運用

js_engineではCI cache改善が効かないことを実測で確認し、test262 shardingへ方針を切り替えた。作業運用としては、相談・レビュー・実装計画をどのエージェントに任せるかの使い分けも少し固まった。

## 2026/6/12

### Canopy / アーキテクチャ再設計

再設計のS2からS5aまでを一気に進めた。editorからsync sessionとtransportを切り出し、lang/runtime SPI、ffi/host registry、substrate governance、import-graph lintで境界を固定した。

### プロダクト方針

Canopyを「editor / frameworkの証明」から、「write-to-selfのpost product」へ寄せる方針転換を決めた。最小のwrite→surfaceループのprototypeを置き、source retrievalより前に「書いたものが自分へ返ってくる」体験を確かめる方向へ進んだ。

### 周辺リポジトリ

incrではIncremental TEAのrendererとsubscriptionがさらに進み、Loomではparser runtime attachment、js_engineではarchitecture refactor Stage 0-7、MoonDspではscheduler周辺の責務分割が進んだ。

## 2026/6/13

### Canopy / editor-neutral test grammar

editorをlanguageから完全に切り離すため、neutral test grammarを入れた。これにより、editorのテストがLambda固有の構文に依存しすぎる問題を減らした。

### Codex連携

Codex app-serverとMCP wrapperの使い分けを検証した。相談役としてはMCP、streamingやinteractive approvalなど「Codexの上に作る」用途ではapp-server、という整理をした。

### プロダクトとCanvas

write→surfaceループには、resurfacing signalでの並べ替えとsame-input askを追加した。Canvasではruntime seam extractionやcontractsの整理が進んだ。

### loom / js_engine

LoomではMarkdownIRやMarkdown block reparseの準備が進み、js_engineではStage 0-7のarchitecture refactorが進んだ。

## 2026/6/14

### Canopy / Lambda CstFold modernization

Lambda投影のCstFold現代化を進めた。既存Canopy semanticsとLoom CstFoldの差分を明示し、互換adapterを置いたうえで、leaf / if / app / bopなどを段階的にCstFold経由へ寄せた。

その流れで`DefinitionIndex`を切り出し、scope / edit / semantic / companion / idealのconsumerを`ModuleProjection`から離し始めた。block-local renameも正しく動くようになった。

### Canvas / loom / incr / MoonDsp / js_engine

Canvasでは接続preview互換性をMoonBit側へ寄せ、source-backed demoの`defer_sync`順序を整理した。LoomではMarkdown incremental block reparse、incrではincr_tea benchmark、MoonDspではscheduler facade分割、js_engineではarchitecture redesign Stage 8-10が進んだ。

## 2026/6/15

### Canopy / Lambda §20 completion

Lambda編集のblock-local対応を一気に仕上げた。delete / duplicate / move binding ops、move opのscoping soundness、block-shadowing filter、typed `fn` token metadata、EditContext整理を入れた。

最後にlegacy `ModuleProjection`を削除し、Lambdaはgeneric projection memosへ完全に移行した。これでdocs/TODO.md §20のbinding clauseは完了した。

### FFI / Ideal

`lib/js`を新設し、opaque `Any` handle、property / method / global escape hatch、JSON、Promise bridgeを置いた。これを使って`ffi/io`にfile read/writeを実装し、IdealのOpen / Save toolbarから呼べるようにした。

Ideal側では`globalThis.__canopy_*`を単一の`__canopy_bridge`へ集約した。

### 周辺リポジトリ

LoomではMarkdownIR M0 policyとM1 heading/paragraph slice、incrではspreadsheet proofとinactive root、js_engineではES2024 Set methodsやinternal slots整理が進んだ。

## 2026/6/16

### Canopy / Lambda §20完了とalpha pilot

`ExtractToLet`をblock-awareにした。block body pathではbody expressionの直前に、block def pathではblock defsの前に`let`を挿入する。block scopeとlambda scopeを区別し、capture guardも追加した。

Lambdaのalpha-safe beta pilotを`lang/lambda/alpha`に置いた。`ScopeGraph`から投影termをlowerし、binder identityでroot beta reductionし、capture無しでnamed `Term`へreifyする実験的packageになる。

夕方以降には、このalpha-safe core boundaryをmainへ入れ、binding-id compatibility fallbackも削った。Ideal側ではTabs UI helperをvalue-derivedな小さな層へ切り出した。

### 周辺リポジトリ

LoomではMarkdownIR M1 vertical slice、incrではinactive-root cohort測定、js_engineではbytecode call-frame fast pathが進んだ。js_engineではparam bindingのenv round-tripをskipし、binding stepが大きく改善した。

## 2026/6/17

### Canopy / Lambda編集の健全性

Lambda編集は「生成された文字列を見る」よりも、「編集後テキストを再パースして意図したASTになるかを見る」方針へ寄せた。

- block-local binding editの敵対的ケースを追加した（move up/down、delete、root/block shadowing）。
- `ScopeGraph.node_scope` / `node_cutoffs`をprivateにし、query API経由にした。
- `ExtractToLet`は#674の実装で#659の懸念を満たしていることを、再パース付きregression testで確認した。
- `DuplicateBinding`は`_copy`ではなく`x1`, `x2`, ... のようなlexableな名前を使うようにした。後続のfree referenceを捕まえる候補も避ける。

これで#649と#659は完了。#650はmove/deleteのindentationとして残る。

### Canopy / Grove Level 1 identity hint

構造編集後もNodeIdを保ちやすくするため、Grove Level 1のidentity hint channelを入れた。

core側では`IdentityTransform`と`reconcile_hinted`を追加した。editor側では`SyncEditor`がspan edit batchに対応するhintを保持し、Lambda側では`TreeEditOp`を`IdentityTransform`へ落とす。

`WrapInLambda`などはNodeId維持に使えるhintを出す。一方、`ExtractToLet`や`ChangeOperator`のように安全なhintを出しづらいものは、保守的に`Opaque`へ落とす。最後にWrap / UnwrapのE2E coverageも追加した。

残りは、write / read / clearの2端contractを`HintChannel`型で包むことと、hintあり / なしreconcileの重複整理。

### Canopy / analysis query layer設計

外部解析結果をCanopyへ入れるためのanalysis query layer設計を追加した。ast-grepや将来の`moon ide`結果を、snapshotに紐づいたtyped factとして正規化し、decorationsとして表示する方針にした。

Phase 1は、ast-grepのbyte offsetをUTF-16 rangeへ変換してrange highlightするだけ。rewrite、node-id mapping、protocol変更はまだしない。

### Canopy / Ideal

IdealのAction Overlayを値所有の境界へ寄せた。Cell / Emit handleをUI helperの外へ漏らさず、action overlayのflowとexecを分け直した。

### loom

MarkdownIRはM1からrecovery / raw node semanticsへ進んだ。raw / recovered diagnostics、mdast export、direct syntax diagnostics、recovery adapter contractを追加し、HTML harnessへ送る範囲も明確にした。

現在の作業ツリーでは、次の#328相当としてMarkdownIR mdast exportにunist `position`を付ける変更が進行中。Canopy superprojectから見ると、`loom` submoduleは`0a827c3`から`3856167`へ進んだうえで未コミット差分が残っている。

### incr

incr_teaはinactive-root測定からactivation policyへ進んだ。ratio tableを再確認し、activation trigger probeを追加し、policyをdocsで決めて実装まで入れた。Loom submodule内の`incr`も`34ac477`から`f7681bc`へ進んでいる。

### js_engine

`needs_own_env`系列のbytecode最適化が続いた。leaf bytecode functionで`Environment::new`をskipし、same-realm calleeではrealm-proto wrapperを避け、active-override `Ref`も単一の`Ref[FunctionRealmProtos?]`へ畳んだ。最後にbenchmark tableへ`exec/for_of` rowを追加した。

### 作業運用メモ

今回の記録は、Canopy / Loom / nested incr / js_engineのGit log、現在の未コミット差分、Claude project memory、Codex memoryを突き合わせて書いた。

6/17時点で特に重要なのは次の3点。

1. Lambda editの検証は、実際の編集後テキストを再パースして確認する。
2. analysis factはsnapshotに紐づき、いつでも捨てられるものとして扱う。
3. Grove hint channelは、次の構造編集言語へ広げる前に`HintChannel`型で明示する。
