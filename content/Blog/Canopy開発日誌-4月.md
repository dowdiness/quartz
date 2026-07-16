---
title: Canopy開発日誌-4月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-06-23T03:38:00+09:00
modified: 2026-07-16T16:15:00+09:00
---

# Canopy開発日誌-4月

2026年4月のCanopy開発ログ（日次記録）。月の要約は[[Canopy開発日誌-4月-まとめ|4月-まとめ]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 今月の大きな流れ

月全体の流れは[[Canopy開発日誌-4月-まとめ|4月-まとめ]]にまとめた。以下は週次・日次の作業記録。第1週から EditorProtocol 統一と Markdown block editor の立ち上げが始まる。

## 4月第1週: EditorProtocol、pretty-printer、Markdown block editor

editorとviewの接続方式をEditorProtocolに統一し、ideal editor/prosemirror example/CodeMirror 6を同じprotocolで扱えるようにした。同時にpretty-printerの出力をViewNodeへ橋渡しし、HTML syntax highlightingを実現した。Markdown向けblock editorも立ち上げ、複数言語・複数viewの編集基盤を並行して拡張した一週間だった。

## 2026/4/1

### Canopy / EditorProtocol、pretty-printer、inspector panel

EditorProtocol Phase 3〜5を実装した。CM6Adapter/PMAdapterの追加、Ideal editor protocol migration、prosemirror example migrationを行った。Ideal editorにsource range、text preview、token spansを表示するinspector panelも追加した（[#107](https://github.com/dowdiness/canopy/pull/107)）。3月末に着手した Wadler-Lindig pretty-printer engine も annotation support 付きで実装し、`get_ast_pretty` へ統合した（[#106](https://github.com/dowdiness/canopy/pull/106)）。

主なPR / Issue: canopy [#106](https://github.com/dowdiness/canopy/pull/106), [#107](https://github.com/dowdiness/canopy/pull/107)

## 2026/4/2

### Canopy / Pretty-printer ViewNode bridge

pretty-printer outputをViewNodeへ変換するbridgeを実装した。`get_pretty_view`、`compute_pretty_patches`、HTMLAdapter syntax highlighting、FFI exportsを追加した。projectionをlanguage-agnosticにするrefactorも並行して進めた（[#109](https://github.com/dowdiness/canopy/pull/109)）。

主なPR / Issue: canopy [#109](https://github.com/dowdiness/canopy/pull/109)

## 2026/4/3

### Canopy / Container Phase 2とecho/TinySegmenter

- Container Phase 2（shared global LVs、text sync）を実装（[#112](https://github.com/dowdiness/canopy/pull/112)）。
- echo: コミット履歴や作業ログから意味的に関連する情報を探すAI検索機能。TinySegmenter + bigram blended tokenizerを実装した（[#110](https://github.com/dowdiness/canopy/pull/110)）。

主なPR / Issue: canopy [#110](https://github.com/dowdiness/canopy/pull/110), [#112](https://github.com/dowdiness/canopy/pull/112)

## 2026/4/4

### Canopy / Markdown edit opsとweb editor

Markdown block editor向けに7つのMarkdownEditOp（SplitBlock、MergeBlocks等）を実装し、web editor pageとTypeScript bridgeを追加した（[#113](https://github.com/dowdiness/canopy/pull/113), [#114](https://github.com/dowdiness/canopy/pull/114), [#115](https://github.com/dowdiness/canopy/pull/115)）。

主なPR / Issue: canopy [#113](https://github.com/dowdiness/canopy/pull/113), [#114](https://github.com/dowdiness/canopy/pull/114), [#115](https://github.com/dowdiness/canopy/pull/115)

## 4月第2週: block-input、zipper、B-tree、semantic layer

Markdown block-inputのtextarea overlay、arrow key navigation、backspace mergeなど細かい挙動を整理し、block editorを実用的に磨いた。rose tree zipper（`lib/zipper`）、generic B-tree library（`lib/btree`）、semantic layer（`lib/semantic`）という新しい汎用ライブラリを立ち上げ、MoonBit v0.9 migrationも完了させた。

## 2026/4/5

### Canopy / BlockInputとMarkdownPreview

textarea overlay付きのBlockInput thin input layerと、MarkdownPreview semantic HTML adapterを追加した（[#117](https://github.com/dowdiness/canopy/pull/117)）。

## 2026/4/6

### Canopy / Block editing fixとscope highlighting

block editorのarrow key navigation、backspace merge、ZWSP placeholder、block ID uniquenessなどのbug fixを行った。Ideal editorではtree viewにscope-colored binder highlightingを追加した（[#122](https://github.com/dowdiness/canopy/pull/122)）。

主なPR / Issue: canopy [#121](https://github.com/dowdiness/canopy/pull/121), [#123](https://github.com/dowdiness/canopy/pull/123), [#125](https://github.com/dowdiness/canopy/pull/125), [#126](https://github.com/dowdiness/canopy/pull/126), [#128](https://github.com/dowdiness/canopy/pull/128)

## 2026/4/7

### Canopy / Rose tree zipperとMoonBit v0.9

- rose tree zipper library（`lib/zipper`）を追加（[#130](https://github.com/dowdiness/canopy/pull/130)）。
- Ideal editorのoutline tree keyboard navigationを強化（[#132](https://github.com/dowdiness/canopy/pull/132)）。
- full MoonBit v0.9 migration（[#131](https://github.com/dowdiness/canopy/pull/131)）。

主なPR / Issue: canopy [#130](https://github.com/dowdiness/canopy/pull/130), [#131](https://github.com/dowdiness/canopy/pull/131), [#132](https://github.com/dowdiness/canopy/pull/132)

## 2026/4/8

### Canopy / lib/btreeとlib/semantic

- generic B-tree library（`lib/btree`）を追加（[#137](https://github.com/dowdiness/canopy/pull/137)）。
- `lib/semantic`にConfidence lattice（推論結果の確信度を階層的に扱う数学的構造）とsymbolic annotatorを追加（[#136](https://github.com/dowdiness/canopy/pull/136)）。
- lambda-specific logicを`lang/lambda` packagesへ抽出（[#135](https://github.com/dowdiness/canopy/pull/135)）。
- container block doc sync（[#134](https://github.com/dowdiness/canopy/pull/134)）。

主なPR / Issue: canopy [#134](https://github.com/dowdiness/canopy/pull/134), [#135](https://github.com/dowdiness/canopy/pull/135), [#136](https://github.com/dowdiness/canopy/pull/136), [#137](https://github.com/dowdiness/canopy/pull/137)

## 2026/4/9

### Canopy / B-tree range delete extraction

order-treeからB-tree range delete logicを`lib/btree`へ移行した。BTreeElem trait統合、range delete whitebox testsを追加した（[#138](https://github.com/dowdiness/canopy/pull/138), [#139](https://github.com/dowdiness/canopy/pull/139), [#140](https://github.com/dowdiness/canopy/pull/140)）。

主なPR / Issue: canopy [#138](https://github.com/dowdiness/canopy/pull/138), [#139](https://github.com/dowdiness/canopy/pull/139), [#140](https://github.com/dowdiness/canopy/pull/140)

## 2026/4/10

### Canopy / B-tree defensive fix

B-tree `delete_range`のunderfull boundary repair、property-based tests、API narrowing（walker internals非公開化）を追加した。

主なPR / Issue: canopy [#141](https://github.com/dowdiness/canopy/pull/141)

## 4月第3週: Language decoupling、egraph optimizer、Web E2E、drag-and-drop

SyncEditorをlambda-specific typesから切り離す`LanguageCapabilities[T]`を導入し、Canopyを汎用構造編集フレームワークへ近づけた。Lambda evaluatorにはegraph optimizer Tier 3を統合した。品質面ではWeb E2EテストをCIへ追加し、UI面ではideal editorとblock editorの双方でdrag-and-drop foundation、semantic Before/After drop、grip-only dragを実装した。

## 2026/4/11

### Canopy / LanguageCapabilitiesとegraph optimizer

- `LanguageCapabilities[T]`でSyncEditorをlambda-specific typesからdecouple（[#146](https://github.com/dowdiness/canopy/pull/146)）。
- egraph lambda optimizer Tier 3をcanopyに統合（[#158](https://github.com/dowdiness/canopy/pull/158)）。
- Tier 1 + Tier 2 batch escalation with incremental caching（[#150](https://github.com/dowdiness/canopy/pull/150)）。
- Web E2EテストをCIへ追加（[#145](https://github.com/dowdiness/canopy/pull/145)）。
- lib/btreeのsplice promotion chain repair（[#143](https://github.com/dowdiness/canopy/pull/143)）。

主なPR / Issue: canopy [#142](https://github.com/dowdiness/canopy/pull/142), [#143](https://github.com/dowdiness/canopy/pull/143), [#145](https://github.com/dowdiness/canopy/pull/145), [#146](https://github.com/dowdiness/canopy/pull/146), [#147](https://github.com/dowdiness/canopy/pull/147), [#148](https://github.com/dowdiness/canopy/pull/148), [#150](https://github.com/dowdiness/canopy/pull/150), [#151](https://github.com/dowdiness/canopy/pull/151), [#152](https://github.com/dowdiness/canopy/pull/152), [#158](https://github.com/dowdiness/canopy/pull/158)

## 2026/4/12

### Canopy / Drag-and-drop foundation

Ideal editor向けdrag-and-drop foundationを実装した（[#174](https://github.com/dowdiness/canopy/pull/174)）。Confidence lattice lawsのformal verificationも追加した（[#161](https://github.com/dowdiness/canopy/pull/161)）。

主なPR / Issue: canopy [#161](https://github.com/dowdiness/canopy/pull/161), [#172](https://github.com/dowdiness/canopy/pull/172), [#173](https://github.com/dowdiness/canopy/pull/173), [#174](https://github.com/dowdiness/canopy/pull/174)

## 2026/4/14

### Canopy / Drag-and-drop exchange

drag-drop exchange、grip-only drag、position detection、outline DnDを実装した（[#176](https://github.com/dowdiness/canopy/pull/176)）。

## 2026/4/15

### Canopy / Block editor drag-and-dropとsemantic drop

- block editorでsemantic Before/After positioning付きdrag-and-drop reordering（[#179](https://github.com/dowdiness/canopy/pull/179), [#181](https://github.com/dowdiness/canopy/pull/181)）。
- `move_block`のlegality validation（[#180](https://github.com/dowdiness/canopy/pull/180)）。
- loom Boundary 3 bidirectional type-checkerを統合（[#182](https://github.com/dowdiness/canopy/pull/182)）。

主なPR / Issue: canopy [#179](https://github.com/dowdiness/canopy/pull/179), [#180](https://github.com/dowdiness/canopy/pull/180), [#181](https://github.com/dowdiness/canopy/pull/181), [#182](https://github.com/dowdiness/canopy/pull/182)

## 2026/4/17

### Canopy / Document-level undoとlambda diagnostics

- containerのdocument-level undo grouping（[#187](https://github.com/dowdiness/canopy/pull/187)）。
- lambda type-check diagnosticsをweb editorへ表示（[#186](https://github.com/dowdiness/canopy/pull/186)）。

主なPR / Issue: canopy [#185](https://github.com/dowdiness/canopy/pull/185), [#186](https://github.com/dowdiness/canopy/pull/186), [#187](https://github.com/dowdiness/canopy/pull/187)

## 4月第4週: Unified Parser、FFI split、moon.work

loomのunified `@loom.Parser[T]`へ移行し、ReactiveParserを廃止した。FFIをjson/markdown/lambdaごとにpackage分割して言語ごとの責務を明確にした。最後にMoonBit workspace機構であるmoon.workを導入し、CIでパッケージ間の依存方向ルールを検証する仕組みを整えた。

## 2026/4/18

### Canopy / Unified Parser[T]とFFI split

- SyncEditor + lang companions + FFIをunified `@loom.Parser[T]`へ移行（[#200](https://github.com/dowdiness/canopy/pull/200)）。
- FFIをjson/markdown/lambdaごとにpackage分割（[#195](https://github.com/dowdiness/canopy/pull/195), [#196](https://github.com/dowdiness/canopy/pull/196), [#197](https://github.com/dowdiness/canopy/pull/197)）。
- parse recoveryをError ProjNodesとしてsurface（[#193](https://github.com/dowdiness/canopy/pull/193)）。
- SyncStatus + watchdog for WebSocket sync recovery（[#199](https://github.com/dowdiness/canopy/pull/199)）。

主なPR / Issue: canopy [#191](https://github.com/dowdiness/canopy/pull/191), [#192](https://github.com/dowdiness/canopy/pull/192), [#193](https://github.com/dowdiness/canopy/pull/193), [#194](https://github.com/dowdiness/canopy/pull/194), [#195](https://github.com/dowdiness/canopy/pull/195), [#196](https://github.com/dowdiness/canopy/pull/196), [#197](https://github.com/dowdiness/canopy/pull/197), [#199](https://github.com/dowdiness/canopy/pull/199), [#200](https://github.com/dowdiness/canopy/pull/200), [#201](https://github.com/dowdiness/canopy/pull/201)

## 2026/4/19

### Canopy / loom Stage 5/6 bump

loom Stage 5（remove ReactiveParser）とStage 6にbumpした。MemoMap sweep-on-rebuild、InternId-in-Relation testを取り込んだ。

主なPR / Issue: canopy [#202](https://github.com/dowdiness/canopy/pull/202), [#203](https://github.com/dowdiness/canopy/pull/203)

## 2026/4/22

### Canopy / moon.work workspace導入

MoonBit workspace（moon.work）を導入した。canopy + lib/text-change + lib/zipperをworkspace化し、CIで依存方向ルールを検証する。lib/editor-adapterをnpm package `@canopy/editor-adapter`へ変換した（[#210](https://github.com/dowdiness/canopy/pull/210), [#211](https://github.com/dowdiness/canopy/pull/211), [#212](https://github.com/dowdiness/canopy/pull/212)）。

主なPR / Issue: canopy [#210](https://github.com/dowdiness/canopy/pull/210), [#211](https://github.com/dowdiness/canopy/pull/211), [#212](https://github.com/dowdiness/canopy/pull/212)

## 2026/4/24

### Canopy / btree registry化

`lib/btree`をmooncakes registry経由のrle@0.2.0に切り替え、独立packageとして整理した。

## 2026/4/26

### Canopy / lambda typecheck pipeline evolution

loomのtypecheck range wedge fixをbumpした。lambda typecheck pipeline evolution planをdocsに追加した。
