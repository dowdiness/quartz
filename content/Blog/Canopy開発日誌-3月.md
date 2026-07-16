---
title: Canopy開発日誌-3月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-06-23T03:36:00+09:00
modified: 2026-07-16T16:15:00+09:00
---

# Canopy開発日誌-3月

2026年3月のCanopy開発ログ（日次記録）。月の要約は[[Canopy開発日誌-3月-まとめ|3月-まとめ]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 今月の大きな流れ

月全体の流れは[[Canopy開発日誌-3月-まとめ|3月-まとめ]]にまとめた。以下は週次ヘッダと日付見出しによる作業記録。月初（第1週）から loom 分離と SyncEditor 移行が始まる。

## 3月第1週: parser→loom分離、SyncEditor、名前解決

月初はparserサブモジュールを`dowdiness/loom`として独立させ、Canopy側を新APIへ移行する作業が中心だった。loomはのちに独立リポジトリとして発展するparser/AST基盤であり、この週が分離の出発点となった。続いてParsedEditorをSyncEditorに置き換え、編集・undo・同期・presenceを一つのeditor abstractionで扱える土台を整えた。週末にはLambdaの名前解決とgraphviz可視化を追加し、構造編集の前段階が揃った。

## 2026/3/2

### Canopy / parser → loom 移行開始

parserサブモジュールのURLとディレクトリ名をloomへ変更した。`dowdiness/parser` APIから`dowdiness/loom` APIへの移行を開始した。

主なコミット: rename parser submodule directory to loom, update parser submodule URL after repo rename to loom, migrate from dowdiness/parser to dowdiness/loom API

## 2026/3/5

### Canopy / SyncEditor登場

ParsedEditorを廃止し、SyncEditorを導入した。SyncEditorはtext編集、tree編集、undo/redo、sourceMap/registry管理を一元化するeditor abstractionである。あわせてImperativeParserのdirty-flag方式から、ReactiveParser（Signal/Memo pipeline）へ移行した。

Edit Bridge Phase 1として、`compute_edit`をloomのTextDelta APIに置き換えた。

主なPR / Issue: canopy [#15](https://github.com/dowdiness/canopy/pull/15), [#16](https://github.com/dowdiness/canopy/pull/16), [#17](https://github.com/dowdiness/canopy/pull/17)

## 2026/3/7

### Canopy / 名前解決とgraphviz

- SyncEditorにname resolutionを追加（[#19](https://github.com/dowdiness/canopy/pull/19)）。
- ToDot/FromDot traitを追加し、Graph AST interchangeを実装（[#18](https://github.com/dowdiness/canopy/pull/18)）。
- Incremental Hylomorphismのアーキテクチャ文書を追加。

主なPR / Issue: canopy [#18](https://github.com/dowdiness/canopy/pull/18), [#19](https://github.com/dowdiness/canopy/pull/19)

## 3月第2週: Rabbita性能、source_file_grammar、flat grammar

Rabbitaベースのprojectional editorを本格化した。tree editorのsubtree再利用（elide/hydrate）で性能を回復し、projectional editorとして実用的な速度を取り戻した。同時に`source_file_grammar`によるO(1)增量編集、flat grammar統合、projection incremental updatesの設計を進め、大規模な構造編集に耐えるパーサー・投影層の構築を始めた。

## 2026/3/10

### Canopy / Rabbita Cloudflare Pages化

Rabbita editorをCloudflare Pagesへデプロイする準備を進めた。Wrangler configの追加、tree edit bridgeのCRDT roundtrip対応を行った。

主なコミット: Make Rabbita Cloudflare Pages ready, Add Wrangler config for Rabbita deploy, Add tree edit bridge for CRDT roundtrip

## 2026/3/11

### Canopy / Rabbita性能回復

Rabbita editorの性能が著しく低下していた問題に対処した。subtreeのelide/hydrate、unchanged subtreeのreuse、deferred selection state、incremental text edit、parser memoからのprojection derivationなどを導入し、Rabbita projection editorのrefresh workを削減した。perf harnessも追加した。

主なPR / Issue: canopy [#20](https://github.com/dowdiness/canopy/pull/20), [#21](https://github.com/dowdiness/canopy/pull/21)

### Canopy / examples整理

web appとdemo-reactを`examples/`ディレクトリへ移動した。

## 2026/3/14

### Canopy / source_file_grammarとEphemeralStore

- `source_file_grammar`へ切り替え、O(1)增量編集を実現（source_file_grammar switch）。
- peer presence awareness用のEphemeralStoreを実装し、SyncEditorとFFI surfaceへ統合。
- Rabbita perf harnessをphase timingとdiagnosticsで再設計。

主なコミット: switch to source_file_grammar for O(1) incremental edits, add source_file_to_proj_node for flat LetDef* structure, integrate EphemeralStore into SyncEditor

## 2026/3/15

### Canopy / flat grammar統合とprojection incremental updates

flat grammar統合（[#32](https://github.com/dowdiness/canopy/pull/32)）を進めた。text_change adapterの削除、shared text change moduleの抽出、CellMeta supertraitの導入を行い、projection incremental updatesの計画を立案してFlatProj設計を確定した。

主なPR / Issue: canopy [#32](https://github.com/dowdiness/canopy/pull/32)

## 3月第3週: ProseMirror/CodeMirror、協調編集、CRDT性能

Projectional editorにCodeMirror 6 / ProseMirrorを統合し、テキストエディタとの橋渡しを強化した。協調編集面ではWebSocket transport、relay server、ephemeral store v2、sync recovery protocolを実装し、複数人編集の基盤が大きく進んだ。CRDT側ではFugueTreeのiterative traverse、order-tree導入、event-graph-walkerの高速化により、大規模文書でも使える性能を目指した。

## 2026/3/18

### Canopy / リポジトリ名をcrdtからcanopyへ

プロジェクト名を`crdt`から`canopy`に変更した。README rewrite、architecture docs修正、各種パス・パッケージ名の更新を行った。同日、FlatProj最適化（[#36](https://github.com/dowdiness/canopy/pull/36)）、RLE sync（[#35](https://github.com/dowdiness/canopy/pull/35)）、RLE Phase 0（[#34](https://github.com/dowdiness/canopy/pull/34)）がmergeされた。

### Canopy / ProseMirror + CodeMirror 6統合

ProseMirrorベースのprojectional editorを実装した。CrdtBridge、leaf edit routing、reconciler、CM6 NodeViews、PM schema、ProjNode→PM変換、static EditorViewを追加した。同日、CodeMirror 6 code editorへの置き換えも並行して進めた。

主なPR / Issue: canopy [#34](https://github.com/dowdiness/canopy/pull/34), [#35](https://github.com/dowdiness/canopy/pull/35), [#36](https://github.com/dowdiness/canopy/pull/36)

## 2026/3/19

### Canopy / Ephemeral Store v2とTransport

- EphemeralHub、namespace-based store routing、presence view types、EphemeralValue serializationを実装（[#39](https://github.com/dowdiness/canopy/pull/39)）。
- SyncTransport traitとSyncMessage wire protocol、InMemoryTransportを追加。
- Projectional editorのtext delta経路を実装（[#37](https://github.com/dowdiness/canopy/pull/37)）。

主なPR / Issue: canopy [#37](https://github.com/dowdiness/canopy/pull/37), [#38](https://github.com/dowdiness/canopy/pull/38), [#39](https://github.com/dowdiness/canopy/pull/39)

## 2026/3/20

### Canopy / lazy tree refreshとWebSocket transport Phase 2

- projectionのlazy structural index buildersを導入し、unchanged subtreeをrefreshでskip（[#42](https://github.com/dowdiness/canopy/pull/42)）。
- WebSocket transport Phase 2: Cloudflare Worker relay + client glueを追加（[#41](https://github.com/dowdiness/canopy/pull/41)）。

主なPR / Issue: canopy [#41](https://github.com/dowdiness/canopy/pull/41), [#42](https://github.com/dowdiness/canopy/pull/42)

## 2026/3/21

### Canopy / 構造編集アクションとincremental parser最適化

- **16の構造編集アクション**を実装（[#48](https://github.com/dowdiness/canopy/pull/48)）。WrapInLambda、ChangeOperator、InsertChildなど、Lambda編集の基本操作が揃った。
- incremental parser最適化: balanced trees + size-threshold（[#46](https://github.com/dowdiness/canopy/pull/46)）。
- Ideal editorのmobile layout、drawer panels、touch targets、design tokens導入。

主なPR / Issue: canopy [#46](https://github.com/dowdiness/canopy/pull/46), [#48](https://github.com/dowdiness/canopy/pull/48)

## 2026/3/22

### Canopy / incremental SourceMap/RegistryとCRDT性能

- SourceMapとRegistryのincremental patch pathを実装（[#51](https://github.com/dowdiness/canopy/pull/51)）。
- WebSocket sync recovery protocol（SyncRequest/SyncResponse）を追加（[#53](https://github.com/dowdiness/canopy/pull/53)）。
- Ideal editorのdocument persistence: localStorage + SQLite + shareable URLs（[#52](https://github.com/dowdiness/canopy/pull/52)）。
- FugueTreeのtraverse_treeをiterative化し、500+ defのdocumentでも動作するように。
- order-treeをgit submoduleとして追加。

主なPR / Issue: canopy [#51](https://github.com/dowdiness/canopy/pull/51), [#52](https://github.com/dowdiness/canopy/pull/52), [#53](https://github.com/dowdiness/canopy/pull/53)

## 2026/3/23

### Canopy / graph library (alga) とWebSocket hardened

- DirectedGraph traitとalgorithmsを持つgraph libraryを追加し、`dowdiness/alga`として切り出した。WebSocket協調編集にはerror recovery、offline queue、persistenceを追加した。`compute_text_edit`はhandler chain with middlewareへ分解した。

主なコミット: add graph library with DirectedGraph trait, harden WebSocket collaboration with error recovery, decompose compute_text_edit into handler chain

## 3月第4週: Framework抽出、block editor、JSON editor

Genericなeditor frameworkを切り出し、CanopyをLambda専用から複数言語に対応できる構造編集フレームワークへ近づけた。block editor、JSON editor、AST Zipper、Container Phase 1、pretty-printerを立ち上げ、前半で整えた基盤をもとに新しいeditor形態を次々と増やした一週間だった。

## 2026/3/24

### Canopy / Framework Extraction Phase 1

- `ProjNode[T]`をgeneric化し、`TreeNode`/`Renderable` traitを導入（Term impl）。
- `SyncEditor[T]`をparser factoryでparameterize。
- framework/coreパッケージへの切り出し準備。

主なPR / Issue: canopy [#57](https://github.com/dowdiness/canopy/pull/57)

## 2026/3/28

### Canopy / Framework Extraction Phase 2/3 + block editor

- framework/coreパッケージを抽出し、`lang/lambda/`へlambda固有コードを移動。
- loom側にTreeNode/Renderable traitを移し、Canopy側はloom定義を使用。
- **block editor**のPhase 1a〜1dを実装: scaffold、BlockDoc CRUD、Markdown export/import、1d JS bridge + TypeScript web shell（[#61](https://github.com/dowdiness/canopy/pull/61), [#63](https://github.com/dowdiness/canopy/pull/63), [#65](https://github.com/dowdiness/canopy/pull/65), [#67](https://github.com/dowdiness/canopy/pull/67)）。
- event-graph-walkerをMovableTree CRDTへbump。

主なPR / Issue: canopy [#58](https://github.com/dowdiness/canopy/pull/58), [#60](https://github.com/dowdiness/canopy/pull/60), [#61](https://github.com/dowdiness/canopy/pull/61), [#62](https://github.com/dowdiness/canopy/pull/62), [#63](https://github.com/dowdiness/canopy/pull/63), [#64](https://github.com/dowdiness/canopy/pull/64), [#65](https://github.com/dowdiness/canopy/pull/65), [#66](https://github.com/dowdiness/canopy/pull/66), [#67](https://github.com/dowdiness/canopy/pull/67)

## 2026/3/29

### Canopy / JSON editor、AST Zipper、framework/core

- **JSON editor**のprojection pipeline、edit handlers、bridge、integration testsを実装。
- **AST Zipper**を追加し、tree pane navigationと構造編集を支える（[#89](https://github.com/dowdiness/canopy/pull/89)）。
- `SpanEdit`と`FocusHint`をframework/coreへ移動。
- `TextDoc`/`TreeDoc`/`TreeDocError`を`TextState`/`TreeState`/`TreeError`へrename。

主なPR / Issue: canopy [#89](https://github.com/dowdiness/canopy/pull/89), [#98](https://github.com/dowdiness/canopy/pull/98), [#99](https://github.com/dowdiness/canopy/pull/99)

## 2026/3/30

### Canopy / JSON editor完成とCloudflare deploy

- JSON editorをmerge（[#100](https://github.com/dowdiness/canopy/pull/100)）。
- 全examplesをCloudflare Pagesへdeployするworkflowを追加。
- GitHub Pages deploy workflowを削除。

主なPR / Issue: canopy [#100](https://github.com/dowdiness/canopy/pull/100)

## 2026/3/31

### Canopy / Container Phase 1とideal keyboard navigation

- **Container Phase 1**: Document structとblock editor migration（[#103](https://github.com/dowdiness/canopy/pull/103)）。
- **Ideal editor**: outline treeでのkeyboard-driven structural navigation（[#105](https://github.com/dowdiness/canopy/pull/105)）。
- JSON editor用のVite multi-page build、JSON CRDT editor web pageとTypeScript bridgeを追加。
- Wadler-Lindig pretty-printer engine design specを追加。
- alga、event-graph-walker、loom等submoduleをbump。

主なPR / Issue: canopy [#103](https://github.com/dowdiness/canopy/pull/103), [#104](https://github.com/dowdiness/canopy/pull/104), [#105](https://github.com/dowdiness/canopy/pull/105)
