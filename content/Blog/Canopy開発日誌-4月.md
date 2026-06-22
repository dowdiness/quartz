---
title: Canopy開発日誌-4月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-06-24T00:00:00+09:00
modified: 2026-06-24T00:00:00+09:00
---

# Canopy開発日誌-4月

2026年4月のCanopy開発ログ。4月はeditor protocolの統一、pretty-printer ViewNode bridge、Markdown block editor、generic B-tree、Confidence lattice、drag-and-drop、Web E2E、loom Parser[T]統一、moon.work workspace導入など、Canopyの土台を次の段階へ引き上げる月になった。

## 今月の大きな流れ

- **EditorProtocol**: ideal editorとprosemirror exampleをEditorProtocol経由に統一。CM6Adapter/PMAdapterを追加し、editorのadapter層を整理した。
- **Pretty-printer bridge**: Wadler-Lindig pretty-printerの出力をViewNodeへ変換し、HTML syntax highlightingと統合。
- **Markdown block editor**: Markdown用のblock editor、7つのMarkdown edit ops、three-mode web editor、block-input textarea overlay、MarkdownPreview semantic HTML adapterを実装。
- **Container**: Container Phase 2（text sync）、Phase 3（block doc sync）、document-level undo groupingを実装。
- **Generic B-tree**: `lib/btree`を汎用B-tree libraryとして切り出し、order-treeと統合。range delete、splice promotion chain repairを実装。
- **Semantic layer**: `lib/semantic`にConfidence latticeとsymbolic annotatorを追加。
- **Language decoupling**: `LanguageCapabilities[T]`でSyncEditorからlambda-specific typesを切り離し、generic tree opを実現。
- **Drag-and-drop**: ideal editorとblock editorでdrag-and-drop foundation、semantic Before/After drop、grip-only drag、outline DnDを実装。
- **Web E2E**: Lambda/JSON editorのE2EテストをCIへ追加。
- **loom Parser[T]統一**: ReactiveParserを廃止し、unified `@loom.Parser[T]`へ移行。
- **moon.work**: MoonBit workspace機構を導入し、依存方向ルールをCIで検証。

## 4月第1週: EditorProtocol、pretty-printer、Markdown block editor

EditorProtocolの統一、pretty-printerのViewNode bridge、Markdown block editorの立ち上げが中心。

## 2026/4/1

### Canopy / EditorProtocolとinspector panel

EditorProtocol Phase 3〜5を実装。CM6Adapter/PMAdapter追加、ideal editor protocol migration、prosemirror example migration。ideal editorにsource range、text preview、token spansを表示するinspector panelを追加（[#107](https://github.com/dowdiness/canopy/issues/107)）。

主なPR / Issue: canopy [#107](https://github.com/dowdiness/canopy/issues/107)

## 2026/4/2

### Canopy / Pretty-printer ViewNode bridge

pretty-printer outputをViewNodeへ変換するbridgeを実装。`get_pretty_view`、`compute_pretty_patches`、HTMLAdapter syntax highlighting、FFI exports追加。projectionをlanguage-agnosticにするrefactorも並行して進行（[#109](https://github.com/dowdiness/canopy/issues/109)）。

主なPR / Issue: canopy [#109](https://github.com/dowdiness/canopy/issues/109)

## 2026/4/3

### Canopy / Container Phase 2とecho/TinySegmenter

- Container Phase 2（shared global LVs、text sync）を実装（[#112](https://github.com/dowdiness/canopy/issues/112)）。
- echo: コミットメッセージなどからのsemantic search向けにTinySegmenter + bigram blended tokenizerを実装（[#110](https://github.com/dowdiness/canopy/issues/110)）。

主なPR / Issue: canopy [#110](https://github.com/dowdiness/canopy/issues/110), [#112](https://github.com/dowdiness/canopy/issues/112)

## 2026/4/4

### Canopy / Markdown edit opsとweb editor

Markdown block editor向けに7つのMarkdownEditOp（SplitBlock、MergeBlocks等）を実装し、web editor pageとTypeScript bridgeを追加（[#113](https://github.com/dowdiness/canopy/issues/113), [#114](https://github.com/dowdiness/canopy/issues/114), [#115](https://github.com/dowdiness/canopy/issues/115)）。

主なPR / Issue: canopy [#113](https://github.com/dowdiness/canopy/issues/113), [#114](https://github.com/dowdiness/canopy/issues/114), [#115](https://github.com/dowdiness/canopy/issues/115)

## 4月第2週: block-input、zipper、B-tree、semantic layer

Markdown block-inputの細かい挙動整理、rose tree zipper、generic B-tree library、semantic annotator、MoonBit v0.9 migrationが進んだ。

## 2026/4/5

### Canopy / BlockInputとMarkdownPreview

textarea overlay付きのBlockInput thin input layerと、MarkdownPreview semantic HTML adapterを追加（[#117](https://github.com/dowdiness/canopy/issues/117)）。

## 2026/4/6

### Canopy / Block editing fixとscope highlighting

block editorのarrow key navigation、backspace merge、ZWSP placeholder、block ID uniquenessなどのbug fix。ideal editorでtree viewにscope-colored binder highlightingを追加（[#122](https://github.com/dowdiness/canopy/issues/122)）。

主なPR / Issue: canopy [#121](https://github.com/dowdiness/canopy/issues/121), [#123](https://github.com/dowdiness/canopy/issues/123), [#125](https://github.com/dowdiness/canopy/issues/125), [#126](https://github.com/dowdiness/canopy/issues/126), [#128](https://github.com/dowdiness/canopy/issues/128)

## 2026/4/7

### Canopy / Rose tree zipperとMoonBit v0.9

- rose tree zipper library（`lib/zipper`）を追加（[#130](https://github.com/dowdiness/canopy/issues/130)）。
- ideal editorのoutline tree keyboard navigationを強化（[#132](https://github.com/dowdiness/canopy/issues/132)）。
- full MoonBit v0.9 migration（[#131](https://github.com/dowdiness/canopy/issues/131)）。

主なPR / Issue: canopy [#130](https://github.com/dowdiness/canopy/issues/130), [#131](https://github.com/dowdiness/canopy/issues/131), [#132](https://github.com/dowdiness/canopy/issues/132)

## 2026/4/8

### Canopy / lib/btreeとlib/semantic

- generic B-tree library（`lib/btree`）を追加（[#137](https://github.com/dowdiness/canopy/issues/137)）。
- `lib/semantic`にConfidence latticeとsymbolic annotatorを追加（[#136](https://github.com/dowdiness/canopy/issues/136)）。
- lambda-specific logicを`lang/lambda` packagesへ抽出（[#135](https://github.com/dowdiness/canopy/issues/135)）。
- container block doc sync（[#134](https://github.com/dowdiness/canopy/issues/134)）。

主なPR / Issue: canopy [#134](https://github.com/dowdiness/canopy/issues/134), [#135](https://github.com/dowdiness/canopy/issues/135), [#136](https://github.com/dowdiness/canopy/issues/136), [#137](https://github.com/dowdiness/canopy/issues/137)

## 2026/4/9

### Canopy / B-tree range delete extraction

order-treeからB-tree range delete logicを`lib/btree`へ移行。BTreeElem trait統合、range delete whitebox tests追加（[#138](https://github.com/dowdiness/canopy/issues/138), [#139](https://github.com/dowdiness/canopy/issues/139), [#140](https://github.com/dowdiness/canopy/issues/140)）。

主なPR / Issue: canopy [#138](https://github.com/dowdiness/canopy/issues/138), [#139](https://github.com/dowdiness/canopy/issues/139), [#140](https://github.com/dowdiness/canopy/issues/140)

## 2026/4/10

### Canopy / B-tree defensive fix

B-tree `delete_range`のunderfull boundary repair、property-based tests、API narrowing（walker internals非公開化）を追加。

主なPR / Issue: canopy [#141](https://github.com/dowdiness/canopy/issues/141)

## 4月第3週: Language decoupling、egraph optimizer、Web E2E、drag-and-drop

SyncEditorをlambdaから切り離すLanguageCapabilities[T]、egraph lambda optimizer（Tier 3）、Web E2Eテスト、drag-and-drop foundationが主な流れ。

## 2026/4/11

### Canopy / LanguageCapabilitiesとegraph optimizer

- `LanguageCapabilities[T]`でSyncEditorをlambda-specific typesからdecouple（[#146](https://github.com/dowdiness/canopy/issues/146)）。
- egraph lambda optimizer Tier 3をcanopyに統合（[#158](https://github.com/dowdiness/canopy/issues/158)）。
- Tier 1 + Tier 2 batch escalation with incremental caching（[#150](https://github.com/dowdiness/canopy/issues/150)）。
- Web E2EテストをCIへ追加（[#145](https://github.com/dowdiness/canopy/issues/145)）。
- lib/btreeのsplice promotion chain repair（[#143](https://github.com/dowdiness/canopy/issues/143)）。

主なPR / Issue: canopy [#142](https://github.com/dowdiness/canopy/issues/142), [#143](https://github.com/dowdiness/canopy/issues/143), [#145](https://github.com/dowdiness/canopy/issues/145), [#146](https://github.com/dowdiness/canopy/issues/146), [#147](https://github.com/dowdiness/canopy/issues/147), [#148](https://github.com/dowdiness/canopy/issues/148), [#150](https://github.com/dowdiness/canopy/issues/150), [#151](https://github.com/dowdiness/canopy/issues/151), [#152](https://github.com/dowdiness/canopy/issues/152), [#158](https://github.com/dowdiness/canopy/issues/158)

## 2026/4/12

### Canopy / Drag-and-drop foundation

ideal editor向けdrag-and-drop foundationを実装（[#174](https://github.com/dowdiness/canopy/issues/174)）。 Confidence lattice lawsのformal verification（[#161](https://github.com/dowdiness/canopy/issues/161)）。

主なPR / Issue: canopy [#161](https://github.com/dowdiness/canopy/issues/161), [#172](https://github.com/dowdiness/canopy/issues/172), [#173](https://github.com/dowdiness/canopy/issues/173), [#174](https://github.com/dowdiness/canopy/issues/174)

## 2026/4/14

### Canopy / Drag-and-drop exchange

drag-drop exchange、grip-only drag、position detection、outline DnDを実装（[#176](https://github.com/dowdiness/canopy/issues/176)）。

## 2026/4/15

### Canopy / Block editor drag-and-dropとsemantic drop

- block editorでsemantic Before/After positioning付きdrag-and-drop reordering（[#179](https://github.com/dowdiness/canopy/issues/179), [#181](https://github.com/dowdiness/canopy/issues/181)）。
- `move_block`のlegality validation（[#180](https://github.com/dowdiness/canopy/issues/180)）。
- loom Boundary 3 bidirectional type-checkerを統合（[#182](https://github.com/dowdiness/canopy/issues/182)）。

主なPR / Issue: canopy [#179](https://github.com/dowdiness/canopy/issues/179), [#180](https://github.com/dowdiness/canopy/issues/180), [#181](https://github.com/dowdiness/canopy/issues/181), [#182](https://github.com/dowdiness/canopy/issues/182)

## 2026/4/17

### Canopy / Document-level undoとlambda diagnostics

- containerのdocument-level undo grouping（[#187](https://github.com/dowdiness/canopy/issues/187)）。
- lambda type-check diagnosticsをweb editorへ表示（[#186](https://github.com/dowdiness/canopy/issues/186)）。

主なPR / Issue: canopy [#185](https://github.com/dowdiness/canopy/issues/185), [#186](https://github.com/dowdiness/canopy/issues/186), [#187](https://github.com/dowdiness/canopy/issues/187)

## 4月第4週: Unified Parser、FFI split、moon.work

loomのunified Parser[T]へ移行し、FFIを言語ごとに分割。最後にmoon.work workspaceを導入した。

## 2026/4/18

### Canopy / Unified Parser[T]とFFI split

- SyncEditor + lang companions + FFIをunified `@loom.Parser[T]`へ移行（[#200](https://github.com/dowdiness/canopy/issues/200)）。
- FFIをjson/markdown/lambdaごとにpackage分割（[#195](https://github.com/dowdiness/canopy/issues/195), [#196](https://github.com/dowdiness/canopy/issues/196), [#197](https://github.com/dowdiness/canopy/issues/197)）。
- parse recoveryをError ProjNodesとしてsurface（[#193](https://github.com/dowdiness/canopy/issues/193)）。
- SyncStatus + watchdog for WebSocket sync recovery（[#199](https://github.com/dowdiness/canopy/issues/199)）。

主なPR / Issue: canopy [#191](https://github.com/dowdiness/canopy/issues/191), [#192](https://github.com/dowdiness/canopy/issues/192), [#193](https://github.com/dowdiness/canopy/issues/193), [#194](https://github.com/dowdiness/canopy/issues/194), [#195](https://github.com/dowdiness/canopy/issues/195), [#196](https://github.com/dowdiness/canopy/issues/196), [#197](https://github.com/dowdiness/canopy/issues/197), [#199](https://github.com/dowdiness/canopy/issues/199), [#200](https://github.com/dowdiness/canopy/issues/200), [#201](https://github.com/dowdiness/canopy/issues/201)

## 2026/4/19

### Canopy / loom Stage 5/6 bump

loom Stage 5（remove ReactiveParser）とStage 6にbump。MemoMap sweep-on-rebuild、InternId-in-Relation testを取り込んだ。

主なPR / Issue: canopy [#202](https://github.com/dowdiness/canopy/issues/202), [#203](https://github.com/dowdiness/canopy/issues/203)

## 2026/4/22

### Canopy / moon.work workspace導入

MoonBit workspace（moon.work）を導入。canopy + lib/text-change + lib/zipperをworkspace化し、CIで依存方向ルールを検証。lib/editor-adapterをnpm package `@canopy/editor-adapter`へ変換（[#210](https://github.com/dowdiness/canopy/issues/210), [#211](https://github.com/dowdiness/canopy/issues/211), [#212](https://github.com/dowdiness/canopy/issues/212)）。

主なPR / Issue: canopy [#210](https://github.com/dowdiness/canopy/issues/210), [#211](https://github.com/dowdiness/canopy/issues/211), [#212](https://github.com/dowdiness/canopy/issues/213)

## 2026/4/24

### Canopy / btree registry化

`lib/btree`をmooncakes registry経由のrle@0.2.0に切り替え、独立packageとして整理。

## 2026/4/26

### Canopy / lambda typecheck pipeline evolution

loomのtypecheck range wedge fixをbump。lambda typecheck pipeline evolution planをdocsに追加。

## 作業運用メモ

4月はCanopyが「Lambda中心のエディタ」から「複数言語・複数editor viewを扱える汎用構造編集フレームワーク」へ近づいた月。EditorProtocol、LanguageCapabilities[T]、FFI分割、moon.workの導入がその象徴。同時にblock editor、Markdown editor、B-tree、semantic layerなど新しい柱も増え、5月以降の大規模な機能展開の土台が整った。
