---
title: Canopy開発日誌-3月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-06-24T00:00:00+09:00
modified: 2026-06-24T00:00:00+09:00
---

# Canopy開発日誌-3月

2026年3月のCanopy開発ログ。3月はCanopyのリポジトリ名確定、parser→loomの分離、Rabbita/ideal editorの急速な成長、構造編集アクション、CRDTの性能改善、framework抽出、block editor、JSON editorまで、幅広い領域で大きな進捗があった。

## 今月の大きな流れ

- **parser → loom**: 2月までCanopy内にあったparserを`dowdiness/loom`として独立させ、APIを移行した。
- **SyncEditor確立**: ParsedEditorからSyncEditorへ統一。編集・undo/redo・同期・presenceを一つのeditor abstractionで扱うようになった。
- **名前解決とgraphviz**: Lambdaの名前解決をSyncEditorに組み込み、ToDot/FromDot traitでgraph可視化と連携。
- **Rabbita/ideal editor**: Rabbitaベースのprojectional editorを立ち上げ、性能問題を潰し、mobile layout、design tokens、tree pane navigation、理想エディタのUI基盤を作った。
- **構造編集アクション**: 16の構造編集アクションを実装。
- **CRDT性能**: FugueTree/traverse_treeのiterative化、order-tree導入、event-graph-walkerの高速化（two-count retreatで17.7×）など。
- **WebSocket協調編集**: transport layer、relay server、sync recovery protocol、ephemeral store v2を実装。
- **Framework抽出**: `ProjNode[T]`のgeneric化、`TreeNode`/`Renderable` traitの導入、framework/coreパッケージの切り出し。
- **新しいeditor**: block editor、JSON editor、AST Zipper、Container Phase 1、pretty-printerの導入。

## 3月第1週: parser→loom分離、SyncEditor、名前解決

月初はparserサブモジュールを`dowdiness/loom`として独立させ、Canopy側を新APIに移行する作業が中心。その後、ParsedEditorをSyncEditorに置き換え、名前解決とgraphviz可視化を追加した。

## 2026/3/2

### Canopy / parser → loom 移行開始

parserサブモジュールのURLとディレクトリ名をloomへ変更。`dowdiness/parser` APIから`dowdiness/loom` APIへの移行を開始した。

主なコミット: rename parser submodule directory to loom, update parser submodule URL after repo rename to loom, migrate from dowdiness/parser to dowdiness/loom API

## 2026/3/5

### Canopy / SyncEditor登場

ParsedEditorを廃止し、SyncEditorを導入した。SyncEditorはtext編集、tree編集、undo/redo、sourceMap/registry管理を一元化するeditor abstraction。同時にImperativeParserのdirty-flag方式から、ReactiveParser（Signal/Memo pipeline）へ移行した。

Edit Bridge Phase 1として、`compute_edit`をloomのTextDelta APIに置き換えた。

主なPR / Issue: canopy [#15](https://github.com/dowdiness/canopy/issues/15), [#16](https://github.com/dowdiness/canopy/issues/16), [#17](https://github.com/dowdiness/canopy/issues/17)

## 2026/3/7

### Canopy / 名前解決とgraphviz

- SyncEditorにname resolutionを追加（[#19](https://github.com/dowdiness/canopy/issues/19)）。
- ToDot/FromDot traitを追加し、Graph AST interchangeを実装（[#18](https://github.com/dowdiness/canopy/issues/18)）。
- Incremental Hylomorphismのアーキテクチャ文書を追加。

主なPR / Issue: canopy [#18](https://github.com/dowdiness/canopy/issues/18), [#19](https://github.com/dowdiness/canopy/issues/19)

## 3月第2週: Rabbita性能、source_file_grammar、flat grammar

Rabbitaベースのprojectional editorを本格化。tree editorのsubtree再利用で性能を回復し、`source_file_grammar`によるO(1)增量編集、flat grammar統合を進めた。

## 2026/3/10

### Canopy / Rabbita Cloudflare Pages化

Rabbita editorをCloudflare Pagesにデプロイする準備。Wrangler config追加、tree edit bridgeのCRDT roundtrip対応。

主なコミット: Make Rabbita Cloudflare Pages ready, Add Wrangler config for Rabbita deploy, Add tree edit bridge for CRDT roundtrip

## 2026/3/11

### Canopy / Rabbita性能回復

Rabbita editorの性能が著しく落ちていた問題に対処。subtreeのelide/hydrate、unchanged subtreeのreuse、deferred selection state、incremental text edit、parser memoからのprojection derivationなどを導入し、Rabbita projection editorのrefresh workを削減。perf harnessも追加した。

主なPR / Issue: canopy [#20](https://github.com/dowdiness/canopy/issues/20), [#21](https://github.com/dowdiness/canopy/issues/21)

### Canopy / examples整理

web appとdemo-reactを`examples/`ディレクトリへ移動。

## 2026/3/14

### Canopy / source_file_grammarとEphemeralStore

- `source_file_grammar`へ切り替え、O(1)增量編集を実現（source_file_grammar switch）。
- peer presence awareness用のEphemeralStoreを実装し、SyncEditorとFFI surfaceへ統合。
- Rabbita perf harnessをphase timingとdiagnosticsで再設計。

主なコミット: switch to source_file_grammar for O(1) incremental edits, add source_file_to_proj_node for flat LetDef* structure, integrate EphemeralStore into SyncEditor

## 2026/3/15

### Canopy / flat grammar統合とprojection incremental updates

flat grammar統合（[#32](https://github.com/dowdiness/canopy/issues/32)）。text_change adapter削除、shared text change module抽出、CellMeta supertraitの導入。projection incremental updatesの計画を立案し、FlatProj設計を確定。

主なPR / Issue: canopy [#32](https://github.com/dowdiness/canopy/issues/32)

## 3月第3週: ProseMirror/CodeMirror、協調編集、CRDT性能

Projectional editorにCodeMirror 6 / ProseMirrorを統合し、WebSocket協調編集のtransportとephemeral store v2を実装。CRDT側ではFugueTreeの性能改善とorder-tree導入が進んだ。

## 2026/3/18

### Canopy / リポジトリ名をcrdtからcanopyへ

プロジェクト名を`crdt`から`canopy`に変更。README rewrite、architecture docs修正、各種パス・パッケージ名の更新。同日、FlatProj最適化（[#36](https://github.com/dowdiness/canopy/issues/36)）、RLE sync（[#35](https://github.com/dowdiness/canopy/issues/35)）、RLE Phase 0（[#34](https://github.com/dowdiness/canopy/issues/34)）がmergeされた。

### Canopy / ProseMirror + CodeMirror 6統合

ProseMirrorベースのprojectional editorを実装。CrdtBridge、leaf edit routing、reconciler、CM6 NodeViews、PM schema、ProjNode→PM変換、static EditorViewを追加。同日にCodeMirror 6 code editorへ置き換える変更も並行して進んだ。

主なPR / Issue: canopy [#34](https://github.com/dowdiness/canopy/issues/34), [#35](https://github.com/dowdiness/canopy/issues/35), [#36](https://github.com/dowdiness/canopy/issues/36)

## 2026/3/19

### Canopy / Ephemeral Store v2とTransport

- EphemeralHub、namespace-based store routing、presence view types、EphemeralValue serializationを実装（[#39](https://github.com/dowdiness/canopy/issues/39)）。
- SyncTransport traitとSyncMessage wire protocol、InMemoryTransportを追加。
- Projectional editorのtext delta経路を実装（[#37](https://github.com/dowdiness/canopy/issues/37)）。

主なPR / Issue: canopy [#37](https://github.com/dowdiness/canopy/issues/37), [#38](https://github.com/dowdiness/canopy/issues/38), [#39](https://github.com/dowdiness/canopy/issues/39)

## 2026/3/20

### Canopy / lazy tree refreshとWebSocket transport Phase 2

- projectionのlazy structural index buildersを導入し、unchanged subtreeをrefreshでskip（[#42](https://github.com/dowdiness/canopy/issues/42)）。
- WebSocket transport Phase 2: Cloudflare Worker relay + client glueを追加（[#41](https://github.com/dowdiness/canopy/issues/41)）。

主なPR / Issue: canopy [#41](https://github.com/dowdiness/canopy/issues/41), [#42](https://github.com/dowdiness/canopy/issues/42)

## 2026/3/21

### Canopy / 構造編集アクションとincremental parser最適化

- **16の構造編集アクション**を実装（[#48](https://github.com/dowdiness/canopy/issues/48)）。WrapInLambda、ChangeOperator、InsertChildなど、Lambda編集の基本操作が揃った。
- incremental parser最適化: balanced trees + size-threshold（[#46](https://github.com/dowdiness/canopy/issues/46)）。
- ideal editorのmobile layout、drawer panels、touch targets、design tokens導入。

主なPR / Issue: canopy [#46](https://github.com/dowdiness/canopy/issues/46), [#48](https://github.com/dowdiness/canopy/issues/48)

## 2026/3/22

### Canopy / incremental SourceMap/RegistryとCRDT性能

- SourceMapとRegistryのincremental patch pathを実装（[#51](https://github.com/dowdiness/canopy/issues/51)）。
- WebSocket sync recovery protocol（SyncRequest/SyncResponse）を追加（[#53](https://github.com/dowdiness/canopy/issues/53)）。
- ideal editorのdocument persistence: localStorage + SQLite + shareable URLs（[#52](https://github.com/dowdiness/canopy/issues/52)）。
- FugueTreeのtraverse_treeをiterative化し、500+ defのdocumentでも動作するように。
- order-treeをgit submoduleとして追加。

主なPR / Issue: canopy [#51](https://github.com/dowdiness/canopy/issues/51), [#52](https://github.com/dowdiness/canopy/issues/52), [#53](https://github.com/dowdiness/canopy/issues/53)

## 2026/3/23

### Canopy / graph library (alga) とWebSocket hardened

- DirectedGraph traitとalgorithmsを持つgraph libraryを追加し、`dowdiness/alga`として切り出した。
- WebSocket協調編集にerror recovery、offline queue、persistenceを追加。
- `compute_text_edit`をhandler chain with middlewareに分解。

主なコミット: add graph library with DirectedGraph trait, harden WebSocket collaboration with error recovery, decompose compute_text_edit into handler chain

## 3月第4週: Framework抽出、block editor、JSON editor

Genericなeditor frameworkを切り出し、block editor、JSON editor、AST Zipper、Container Phase 1、pretty-printerを立ち上げた。

## 2026/3/24

### Canopy / Framework Extraction Phase 1

- `ProjNode[T]`をgeneric化し、`TreeNode`/`Renderable` traitを導入（Term impl）。
- `SyncEditor[T]`をparser factoryでparameterize。
- framework/coreパッケージへの切り出し準備。

主なPR / Issue: canopy [#57](https://github.com/dowdiness/canopy/issues/57)

## 2026/3/28

### Canopy / Framework Extraction Phase 2/3 + block editor

- framework/coreパッケージを抽出し、`lang/lambda/`へlambda固有コードを移動。
- loom側にTreeNode/Renderable traitを移し、Canopy側はloom定義を使用。
- **block editor**のPhase 1a〜1dを実装: scaffold、BlockDoc CRUD、Markdown export/import、1d JS bridge + TypeScript web shell（[#61](https://github.com/dowdiness/canopy/issues/61), [#63](https://github.com/dowdiness/canopy/issues/63), [#65](https://github.com/dowdiness/canopy/issues/65), [#67](https://github.com/dowdiness/canopy/issues/67)）。
- event-graph-walkerをMovableTree CRDTへbump。

主なPR / Issue: canopy [#58](https://github.com/dowdiness/canopy/issues/58), [#60](https://github.com/dowdiness/canopy/issues/60), [#61](https://github.com/dowdiness/canopy/issues/61), [#62](https://github.com/dowdiness/canopy/issues/62), [#63](https://github.com/dowdiness/canopy/issues/63), [#64](https://github.com/dowdiness/canopy/issues/64), [#65](https://github.com/dowdiness/canopy/issues/65), [#66](https://github.com/dowdiness/canopy/issues/66), [#67](https://github.com/dowdiness/canopy/issues/67)

## 2026/3/29

### Canopy / JSON editor、AST Zipper、framework/core

- **JSON editor**のprojection pipeline、edit handlers、bridge、integration testsを実装。
- **AST Zipper**を追加し、tree pane navigationと構造編集を支える（[#89](https://github.com/dowdiness/canopy/issues/89)）。
- `SpanEdit`と`FocusHint`をframework/coreへ移動。
- `TextDoc`/`TreeDoc`/`TreeDocError`を`TextState`/`TreeState`/`TreeError`へrename。

主なPR / Issue: canopy [#89](https://github.com/dowdiness/canopy/issues/89), [#98](https://github.com/dowdiness/canopy/issues/98), [#99](https://github.com/dowdiness/canopy/issues/99)

## 2026/3/30

### Canopy / JSON editor完成とCloudflare deploy

- JSON editorをmerge（[#100](https://github.com/dowdiness/canopy/issues/100)）。
- 全examplesをCloudflare Pagesへdeployするworkflowを追加。
- GitHub Pages deploy workflowを削除。

主なPR / Issue: canopy [#100](https://github.com/dowdiness/canopy/issues/100)

## 2026/3/31

### Canopy / Container Phase 1とideal keyboard navigation

- **Container Phase 1**: Document structとblock editor migration（[#103](https://github.com/dowdiness/canopy/issues/103)）。
- **ideal editor**: outline treeでのkeyboard-driven structural navigation（[#105](https://github.com/dowdiness/canopy/issues/105)）。
- JSON editor用のVite multi-page build、JSON CRDT editor web pageとTypeScript bridgeを追加。
- Wadler-Lindig pretty-printer engine design specを追加。
- alga、event-graph-walker、loom等submoduleをbump。

主なPR / Issue: canopy [#103](https://github.com/dowdiness/canopy/issues/103), [#104](https://github.com/dowdiness/canopy/issues/104), [#105](https://github.com/dowdiness/canopy/issues/105)

## 2026/4/1

### Canopy / Pretty-printer

Wadler-Lindig pretty-printer engineをannotation support付きで実装し、`get_ast_pretty`へ統合（[#106](https://github.com/dowdiness/canopy/issues/106)）。

## 作業運用メモ

3月はCanopyの"理想エディタ"となるideal editor、Rabbita、構造編集アクション、協調編集基盤が一気に育った月。同時に後の独立リポジトリ化を見越したframework抽出と、CRDT/パーサーの性能改善も並行して進んだ。4月はこの勢いを引き継ぎ、さらにUI/UXと言語実装が深まっていく。
