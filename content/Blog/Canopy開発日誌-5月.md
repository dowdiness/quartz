---
title: Canopy開発日誌-5月
publish: true
tags: [blog, canopy, projectional-editing]
aliases: [Canopy作業日誌]
created: 2026-01-04T20:50:52+09:00
modified: 2026-08-04T23:15:02+09:00
---

# Canopy開発日誌-5月

2026年5月のCanopy開発ログ（日次記録）。月の要約は[[Canopy開発日誌-5月-まとめ|5月-まとめ]]。活動は5/7から。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 今月の大きな流れ

月全体の流れは[[Canopy開発日誌-5月-まとめ|5月-まとめ]]にまとめた。以下は週次・日次の作業記録。PR番号の一覧は文末の[PR索引](#pr索引)にある。前半（5/7〜5/17）は Unicode・履歴・moji、後半（5/18〜）は Rabbita 接続・Cognition・scope graph が中心になる。

## 5月第1週: Unicode 正しさと編集履歴（5/7〜5/9）

月初のCanopy活動は5/7から。SyncEditorにcausal snapshotを足し、Idealでは編集の因果関係をGraphvizで見るhistory viewの土台を作った。並行してissue #216のUnicode正しさ監査が始まり、MarkdownのZWSP処理修正やbridgeのposition単位整理が進んだ。

## 2026/5/7

### Canopy / SyncEditor causal snapshot

SyncEditor に `causal_snapshot` と construction identity（Phase 0）を追加した。編集操作の因果関係を後から追える前提を整えた（[#225](https://github.com/dowdiness/canopy/pull/225)）。

### loom

deprecated MoonBit API の置き換えや seam まわりの整理を進めた（[#100](https://github.com/dowdiness/loom/pull/100), [#101](https://github.com/dowdiness/loom/pull/101), [#102](https://github.com/dowdiness/loom/pull/102)）。

主なPR / Issue: canopy [#225](https://github.com/dowdiness/canopy/pull/225) / loom [#100](https://github.com/dowdiness/loom/pull/100), [#101](https://github.com/dowdiness/loom/pull/101), [#102](https://github.com/dowdiness/loom/pull/102)

## 2026/5/8

### Canopy / causal-graph history と editor-adapter

Ideal に causal-graph history view の Phase 1a（DOT 生成）と Phase 1b（UI 配線）を実装した（[#229](https://github.com/dowdiness/canopy/pull/229), [#232](https://github.com/dowdiness/canopy/pull/232)）。Graphviz/History SVG のフィット、bottom panel の render cache とキーボードナビも整えた（[#234](https://github.com/dowdiness/canopy/pull/234), [#235](https://github.com/dowdiness/canopy/pull/235)）。`editor-adapter` では CM6Adapter への `SetDiagnostics` 実装と strict TypeScript 向けの型修正を入れた（[#227](https://github.com/dowdiness/canopy/pull/227), [#230](https://github.com/dowdiness/canopy/pull/230)）。

### loom

egraph / egglog submodule の整理、cst-transform 研究サンドボックスの削除、pub using facade の拡張を行った（[#103](https://github.com/dowdiness/loom/pull/103)〜[#107](https://github.com/dowdiness/loom/pull/107)）。

主なPR / Issue: canopy [#227](https://github.com/dowdiness/canopy/pull/227), [#228](https://github.com/dowdiness/canopy/pull/228), [#229](https://github.com/dowdiness/canopy/pull/229), [#230](https://github.com/dowdiness/canopy/pull/230), [#231](https://github.com/dowdiness/canopy/pull/231), [#232](https://github.com/dowdiness/canopy/pull/232), [#233](https://github.com/dowdiness/canopy/pull/233), [#234](https://github.com/dowdiness/canopy/pull/234), [#235](https://github.com/dowdiness/canopy/pull/235)

## 2026/5/9

### Canopy / Unicode 正しさ監査の開始

issue #216 に沿った Unicode 正しさ作業が本格化した。非 ASCII の現状挙動をテストで固定し（[#239](https://github.com/dowdiness/canopy/pull/239)）、Markdown export の ZWSP sentinel 除去を修正した（[#238](https://github.com/dowdiness/canopy/pull/238)）。API docs で position 単位を明文化し（[#241](https://github.com/dowdiness/canopy/pull/241)）、event-graph-walker を surrogate-split 修正込みで bump した（[#240](https://github.com/dowdiness/canopy/pull/240)）。あわせてコードベース全体の cohesion audit を実施した（[#236](https://github.com/dowdiness/canopy/pull/236)）。

主なPR / Issue: canopy [#236](https://github.com/dowdiness/canopy/pull/236), [#238](https://github.com/dowdiness/canopy/pull/238), [#239](https://github.com/dowdiness/canopy/pull/239), [#240](https://github.com/dowdiness/canopy/pull/240), [#241](https://github.com/dowdiness/canopy/pull/241)

## 5月第2週: moji・ベンチマーク・Canvas（5/10〜5/16）

UAX #29 ベースの `moji` ライブラリとエディタ統合が入り、Unicode 監査は ideal-bridge の refactor まで進んだ。編集応答の realistic benchmark、Canvas の handles/edges、ZWSP sentinel の三層整理、Inspector traceability の docs 整備が続いた。

## 2026/5/10

### Canopy / moji と ideal-bridge

`moji` ライブラリに UAX #29 grapheme / word segmentation を実装し、エディタへ統合した（[#251](https://github.com/dowdiness/canopy/pull/251)）。ideal-bridge の per-char ループを `handle_text_intent` へ移し（[#246](https://github.com/dowdiness/canopy/pull/246)）、#216 監査 docs を更新した（[#242](https://github.com/dowdiness/canopy/pull/242)〜[#249](https://github.com/dowdiness/canopy/pull/249)）。

### loom

Unicode-safe な lexer offset helper、step lexer、parser diagnostics まわりを Codex 主導で強化した（[#108](https://github.com/dowdiness/loom/pull/108)〜[#120](https://github.com/dowdiness/loom/pull/120)）。

主なPR / Issue: canopy [#242](https://github.com/dowdiness/canopy/pull/242), [#243](https://github.com/dowdiness/canopy/pull/243), [#245](https://github.com/dowdiness/canopy/pull/245), [#246](https://github.com/dowdiness/canopy/pull/246), [#247](https://github.com/dowdiness/canopy/pull/247), [#248](https://github.com/dowdiness/canopy/pull/248), [#249](https://github.com/dowdiness/canopy/pull/249), [#251](https://github.com/dowdiness/canopy/pull/251), [#252](https://github.com/dowdiness/canopy/pull/252) / loom [#108](https://github.com/dowdiness/loom/pull/108)〜[#120](https://github.com/dowdiness/loom/pull/120)

## 2026/5/13

### Canopy / ベンチマークと Unicode 追従

realistic editor response benchmark と phase timing 分割を追加した（[#259](https://github.com/dowdiness/canopy/pull/259), [#260](https://github.com/dowdiness/canopy/pull/260)）。event-graph-walker を `lv_to_position` 最適化と non-BMP 対応込みで更新した（[#256](https://github.com/dowdiness/canopy/pull/256), [#257](https://github.com/dowdiness/canopy/pull/257), [#258](https://github.com/dowdiness/canopy/pull/258)）。`editor-adapter` を npm 0.1.0-alpha.0 として publish した（[#223](https://github.com/dowdiness/canopy/pull/223)）。

主なPR / Issue: canopy [#223](https://github.com/dowdiness/canopy/pull/223), [#256](https://github.com/dowdiness/canopy/pull/256), [#257](https://github.com/dowdiness/canopy/pull/257), [#258](https://github.com/dowdiness/canopy/pull/258), [#259](https://github.com/dowdiness/canopy/pull/259), [#260](https://github.com/dowdiness/canopy/pull/260)

## 2026/5/14

### Canopy / Canvas handles

Canvas に handles と edges を追加した（[#262](https://github.com/dowdiness/canopy/pull/262)）。

主なPR / Issue: canopy [#262](https://github.com/dowdiness/canopy/pull/262)

## 2026/5/15

### Canopy / Canvas smoke test と docs

Canvas の Playwright smoke test を追加した（[#264](https://github.com/dowdiness/canopy/pull/264)）。package README の刷新と aggregator re-export の整理も行った（[#265](https://github.com/dowdiness/canopy/pull/265), [#266](https://github.com/dowdiness/canopy/pull/266)）。

主なPR / Issue: canopy [#264](https://github.com/dowdiness/canopy/pull/264), [#265](https://github.com/dowdiness/canopy/pull/265), [#266](https://github.com/dowdiness/canopy/pull/266)

## 2026/5/16

### Canopy / ZWSP 整理と Inspector traceability

moji に `ZERO_WIDTH_SPACE` と ignorable code point 判定を追加し（[#269](https://github.com/dowdiness/canopy/pull/269)）、Markdown の ZWSP sentinel を三層 split に整理した（[#270](https://github.com/dowdiness/canopy/pull/270)）。core/ の stub 注釈や Inspector traceability TODO など、§7 docs を段階的に進めた（[#268](https://github.com/dowdiness/canopy/pull/268)〜[#275](https://github.com/dowdiness/canopy/pull/275)）。editor-infrastructure 型への `Show` 実装と Inspector kind chip の Renderable 経由ルーティングも入った（[#277](https://github.com/dowdiness/canopy/pull/277), [#278](https://github.com/dowdiness/canopy/pull/278)）。

### incr

v0.9.2 migration と information-structure rebuild、push fanout の lazy allocation 最適化を行った（[#49](https://github.com/dowdiness/incr/pull/49), [#50](https://github.com/dowdiness/incr/pull/50), [#51](https://github.com/dowdiness/incr/pull/51)）。

主なPR / Issue: canopy [#268](https://github.com/dowdiness/canopy/pull/268)〜[#278](https://github.com/dowdiness/canopy/pull/278) / incr [#49](https://github.com/dowdiness/incr/pull/49), [#50](https://github.com/dowdiness/incr/pull/50), [#51](https://github.com/dowdiness/incr/pull/51)

## 5月第3週: Intent panel（5/17）

loomの`pretty_unparse`とCanonical traitを取り込み、IdealのIntent panelを仕上げた。翌週からはRabbitaとCodeMirrorの接続安定化が中心になる。

## 2026/5/17

### Canopy / Intent panel

Ideal に Intent panel を追加し、History/Graphviz の Rabbita 修正と undo Bool API を入れた（[#293](https://github.com/dowdiness/canopy/pull/293)）。Patch / label-unify / op-log gate の follow-up docs も整えた（[#294](https://github.com/dowdiness/canopy/pull/294)）。

### loom

`pretty_unparse`、Canonical companion trait、plan archive などを landing し、Canopy 側で submodule を順次 bump した（[#121](https://github.com/dowdiness/loom/pull/121)〜[#123](https://github.com/dowdiness/loom/pull/123)）。

主なPR / Issue: canopy [#288](https://github.com/dowdiness/canopy/pull/288)〜[#294](https://github.com/dowdiness/canopy/pull/294) / loom [#121](https://github.com/dowdiness/loom/pull/121), [#122](https://github.com/dowdiness/loom/pull/122), [#123](https://github.com/dowdiness/loom/pull/123)

## 5月第4週: Rabbita/CodeMirror 安定化と Inspector（5/18〜5/24）

RabbitaとCodeMirrorの接続安定化、hidden buttonからイベント購読への移行、InspectorとOp logの整備が中心だった。

## 2026/5/18

incr の Datalog を使った UI 開発を試した。[DataScript](https://github.com/tonsky/datascript) と同様のことができるかもしれない。

Rabbita と CodeMirror をつなぐグルーコードも改善した。これまで混沌としていたコードではバグが多く、エディタ操作中に途中で動かなくなることがあったが、それが解消された。
https://claude.ai/share/ed68c575-c459-415d-bd19-b7965dd94e29

主なPR / Issue: loom [#124](https://github.com/dowdiness/loom/pull/124) / canopy [#293](https://github.com/dowdiness/canopy/pull/293), [#296](https://github.com/dowdiness/canopy/pull/296)

## 2026/5/19

### Canopy

rabbita_codemirror（CodeMirror バインディング）の作成を続けた。途中で既存コードのバグが見つかり、その修正に時間を取られた。

また、プログラミング言語の変数入れ替えを incr で実現できるよう環境を整えた。

### moondsp

AudioBuffer API を改修した。これまで `as_fixed_array` で内部の `fixed_array` に直接アクセスする必要があったが、AudioBuffer 構造体から `all` / `any` などのメソッドを直接呼べるようにした。

### トークンの使用量を減らす工夫

コーディングエージェントのトークン使用量を減らすため、[BAML](https://boundaryml.com/) を導入した。効果はまだ不明で、実際に使って確かめる予定。

主なPR / Issue: canopy [#297](https://github.com/dowdiness/canopy/pull/297), [#299](https://github.com/dowdiness/canopy/pull/299), [#300](https://github.com/dowdiness/canopy/pull/300), [#301](https://github.com/dowdiness/canopy/pull/301), [#302](https://github.com/dowdiness/canopy/pull/302) / loom [#126](https://github.com/dowdiness/loom/pull/126), [#129](https://github.com/dowdiness/loom/pull/129) / moondsp [#60](https://github.com/dowdiness/moondsp/pull/60), [#62](https://github.com/dowdiness/moondsp/pull/62)

## 2026/5/20

### Canopy / Rabbita CodeMirror

Canopy の `examples/ideal` で Rabbita と CodeMirror のバインディング移行を進めた。移行完了まで既存実装と新バインディングを共存させ、マウント手順や二重 DOM、フラグ読み取り、undo 記録まわりの問題を潰した。

selection や extension を CodeMirror 専用 API に閉じ込めるのではなく、標準の [Selection](https://developer.mozilla.org/ja/docs/Web/API/Selection) / [Range](https://developer.mozilla.org/ja/docs/Web/API/Range) API や CodeMirror 本体の API を薄い JS FFI として扱う方針に決めた。

### loom

Lambda exampleのAPIを整理した。

### incr

可視化機能追加に向けて内部を整理し、`EventBroadcastPhaseHook` を追加した。専用の public API は増やしていない。パフォーマンスを意識し、イベントリスナではなく Runtime コンストラクタでイベントを buffer してから一括 batch 処理する形にした。ベンチマークでは実行速度が数パーセント向上した。

### moondsp

AudioBuffer の write-time validation 設計を進めた。`new` / `filled` / `fill` / `set` は共通のサンプル検証・正規化パスを通し、-1〜+1 の範囲外の値は正規化する方針にした。`adopt` については、ゼロコピー契約上、採用後に外部ハンドルから変更された値までは MoonBit 側で検証できないことを明記した。

### js_engine

well-known symbol の所有権を realm 側へ移し、[js_engine PR #130](https://github.com/dowdiness/js_engine/pull/130) をマージした。`setup_builtins(env, output, symbols, ...)` を単体で呼ぶ場合も、渡された `SymbolState` で well-known symbol を割り当てるよう修正した。

主なPR / Issue: canopy [#303](https://github.com/dowdiness/canopy/pull/303), [#305](https://github.com/dowdiness/canopy/pull/305), [#306](https://github.com/dowdiness/canopy/pull/306), [#307](https://github.com/dowdiness/canopy/pull/307) / loom [#131](https://github.com/dowdiness/loom/pull/131), [#132](https://github.com/dowdiness/loom/pull/132) / incr [#58](https://github.com/dowdiness/incr/pull/58), [#59](https://github.com/dowdiness/incr/pull/59), [#60](https://github.com/dowdiness/incr/pull/60) / moondsp [#63](https://github.com/dowdiness/moondsp/pull/63) / js_engine [#130](https://github.com/dowdiness/js_engine/pull/130)

## 2026/5/21

### Canopy

`examples/ideal` で Rabbita と DOM イベントの境界を整理した。[Canopy PR #312](https://github.com/dowdiness/canopy/pull/312) で Rabbita に依存しない DOM boundary helper を追加し、[Canopy PR #313](https://github.com/dowdiness/canopy/pull/313) から [Canopy PR #316](https://github.com/dowdiness/canopy/pull/316) では、overlay・sync・structure mode の hidden button として実装していた命令的トリガーを Rabbita の custom event 購読へ置き換えた。

イベント購読の失敗をログ出力する変更も入れたため、Rabbita 側のイベント登録で問題が起きたときに原因を追いやすくなった。UI 操作を DOM の隠しボタンに頼るより、Rabbita 側のイベント購読として扱うほうが、後から読んだときに責務の境界が分かりやすい。

### loom

incremental parser reuse まわりの TODO を進めた。削除時に左隣 CST を再利用するケースや、削除で offset がずれた node を再利用するケースをテストで固定し、現在の invariant が parser-owned な token / subtree identity ではなく、検証済み CST subtree reuse であることを確認した。

### incr

public API の命名と read helper を整理した。`read` まわりの permissive な helper をリネームし、理想形 API への移行計画を立てた。

### js_engine

well-known symbol lookup の移行を進めた。[js_engine PR #131](https://github.com/dowdiness/js_engine/pull/131) と [js_engine PR #132](https://github.com/dowdiness/js_engine/pull/132) で、runtime や stdlib に残っていた引数なし symbol getter を明示的な realm-owned `WellKnownSymbols` アクセスへ移し、互換用 legacy path を削除した。

主なPR / Issue: canopy [#312](https://github.com/dowdiness/canopy/pull/312), [#313](https://github.com/dowdiness/canopy/pull/313), [#316](https://github.com/dowdiness/canopy/pull/316) / loom [#134](https://github.com/dowdiness/loom/pull/134), [#135](https://github.com/dowdiness/loom/pull/135), [#136](https://github.com/dowdiness/loom/pull/136) / incr [#61](https://github.com/dowdiness/incr/pull/61), [#62](https://github.com/dowdiness/incr/pull/62), [#63](https://github.com/dowdiness/incr/pull/63) / js_engine [#131](https://github.com/dowdiness/js_engine/pull/131), [#132](https://github.com/dowdiness/js_engine/pull/132)

## 2026/5/22

### Canopy

Rabbita 側の undo / redo ショートカットも hidden button 経由から外した。[Canopy PR #318](https://github.com/dowdiness/canopy/pull/318) で、CodeMirror 側が `request-undo` / `request-redo` の custom event を dispatch し、Rabbita 側がそれを `Undo` / `Redo` に変換する形になった。前日から続けていた hidden button トリガーの削除は、これで一段落。

その後、`examples/web` と `examples/ideal` への `tsc --noEmit` CI job 追加と、Inspector への incr runtime snapshot 表示も入れた。

### incr

target API facade の作業を進めた。[incr PR #68](https://github.com/dowdiness/incr/pull/68) で理想形 API の facade を追加し、runtime read helper の deprecation、input freshness facade、map relation facade と続けた。Canopy 側から使う API を薄く整えつつ、古い read helper への直接依存を減らしている。

### js_engine

iterator cache や primitive wrapper prototype の状態を `RealmState` へ移した。中心は [js_engine PR #133](https://github.com/dowdiness/js_engine/pull/133) と [js_engine PR #134](https://github.com/dowdiness/js_engine/pull/134) で、factory / prototype 系の状態を module global から realm-owned state へ移す作業を続けている。

注: [Realm](https://tc39.es/ecma262/#sec-code-realms)はECMAScript仕様の概念。

主なPR / Issue: canopy [#318](https://github.com/dowdiness/canopy/pull/318), [#320](https://github.com/dowdiness/canopy/pull/320), [#321](https://github.com/dowdiness/canopy/pull/321) / incr [#68](https://github.com/dowdiness/incr/pull/68), [#69](https://github.com/dowdiness/incr/pull/69), [#70](https://github.com/dowdiness/incr/pull/70), [#71](https://github.com/dowdiness/incr/pull/71), [#72](https://github.com/dowdiness/incr/pull/72) / js_engine [#133](https://github.com/dowdiness/js_engine/pull/133), [#134](https://github.com/dowdiness/js_engine/pull/134)

## 2026/5/23

### Canopy

Inspector まわりの作業を続けた。Patch panel を追加し、`view_op_log` / `view_patch` の guard、Op Log の label format 統一、`SourceMap::nodes_at_position` の範囲制約修正を入れた。Ideal を触りながら内部状態を確認する道具が増え、op log や patch から原因を追いやすくなった。

### incr

API migration に向けてドキュメントと example を増やした。architecture、cookbook、API reference の例を新 API 名に合わせて更新し、[Build Systems à la Carte](https://hackage.haskell.org/package/build) を読みながら、自分の実装がどの評価戦略・依存関係モデルに近いかを docs に記録した。

### moondsp

Loom を使った mini 記法の検証を進めた。`specs/loom-mini-cst` の grammar parity に向けて checked target API examples と loom-mini-cst の docs を更新し、Loom 側の API drift を spec で検出できる状態にした。production parser をすぐ切り替えるのではなく、まず spec 側で Loom の挙動を固定していく方針だ。

### js_engine

prototype 移行を続けた。object function、Promise、WeakMap / WeakSet、Map / Set、Array prototype などの lookup や storage を順に `RealmState` へ寄せ、module global に残っていた prototype 参照を減らした。

主なPR / Issue: canopy [#323](https://github.com/dowdiness/canopy/pull/323), [#324](https://github.com/dowdiness/canopy/pull/324), [#327](https://github.com/dowdiness/canopy/pull/327), [#329](https://github.com/dowdiness/canopy/pull/329) / incr [#73](https://github.com/dowdiness/incr/pull/73), [#74](https://github.com/dowdiness/incr/pull/74), [#75](https://github.com/dowdiness/incr/pull/75), [#76](https://github.com/dowdiness/incr/pull/76), [#77](https://github.com/dowdiness/incr/pull/77), [#78](https://github.com/dowdiness/incr/pull/78), [#79](https://github.com/dowdiness/incr/pull/79) / js_engine [#135](https://github.com/dowdiness/js_engine/pull/135), [#136](https://github.com/dowdiness/js_engine/pull/136), [#137](https://github.com/dowdiness/js_engine/pull/137), [#138](https://github.com/dowdiness/js_engine/pull/138), [#139](https://github.com/dowdiness/js_engine/pull/139)

## 2026/5/24

### Canopy / loom

[Loom issue #147](https://github.com/dowdiness/loom/pull/147) の移行を Canopy 側まで進めた。`text_change` と `moji` は Canopy 配下ではなく Loom monorepo の top-level module に置く方針とし、[loom PR #149](https://github.com/dowdiness/loom/pull/149) で両者を Loom 側へ移し、Canopy 側は [Canopy PR #341](https://github.com/dowdiness/canopy/pull/341) で `./loom/text-change` と `./loom/moji` を参照するようにした。

Canopy 内の `lib/text-change` と `lib/moji`、未使用だった `valtio` submodule も整理した。Loom を単体でビルドしやすくするため、Canopy 側に置かれていた共通部品を Loom の責任範囲へ戻した。

### incr

[incr PR #81](https://github.com/dowdiness/incr/pull/81) で v0.6.0 をリリースした。その後、Canopy や Loom 側の利用に合わせてファイル名や skill の記述を新 API 名に揃えた。

### moondsp

Loom mini CST の改良を続けた。[moondsp PR #75](https://github.com/dowdiness/moondsp/pull/75) で quickcheck を 0.14.0 へ上げ、[moondsp PR #76](https://github.com/dowdiness/moondsp/pull/76) で loom-mini-cst の grammar を広げた。この時点でも production parser は Loom へ切り替えておらず、Loom は spec と回帰テストで使う位置づけのままだ。

### js_engine

`RealmState` 移行をさらに進めた。runtime の Map / Set、boxed primitive、Array、WeakMap / WeakSet、ArrayBuffer storage などを順に `RealmState` 側へ寄せ、CI workflow の Node.js 更新と deprecated な `moon install` 呼び出しの削除も行った。

主なPR / Issue: loom [#149](https://github.com/dowdiness/loom/pull/149) / canopy [#341](https://github.com/dowdiness/canopy/pull/341) / incr [#81](https://github.com/dowdiness/incr/pull/81), [#82](https://github.com/dowdiness/incr/pull/82), [#83](https://github.com/dowdiness/incr/pull/83) / moondsp [#71](https://github.com/dowdiness/moondsp/pull/71), [#73](https://github.com/dowdiness/moondsp/pull/73), [#74](https://github.com/dowdiness/moondsp/pull/74), [#75](https://github.com/dowdiness/moondsp/pull/75), [#76](https://github.com/dowdiness/moondsp/pull/76), [#79](https://github.com/dowdiness/moondsp/pull/79) / js_engine [#140](https://github.com/dowdiness/js_engine/pull/140), [#142](https://github.com/dowdiness/js_engine/pull/142), [#143](https://github.com/dowdiness/js_engine/pull/143), [#144](https://github.com/dowdiness/js_engine/pull/144), [#146](https://github.com/dowdiness/js_engine/pull/146), [#147](https://github.com/dowdiness/js_engine/pull/147), [#148](https://github.com/dowdiness/js_engine/pull/148), [#149](https://github.com/dowdiness/js_engine/pull/149), [#151](https://github.com/dowdiness/js_engine/pull/151)
## 5月第5週: Cognition と scope graph（5/25〜5/31）

Cognition のワークスペースと provider boundary、Lambda scope graph と go-to-definition、ephemeral / byte-codec の切り出し、incr typed spreadsheet demo と js_engine bytecode benchmark が並行して進んだ。

## 2026/5/25

### Canopy

Lambda metadata を editor・ワークスペース・FFI の境界に通す変更を進めた。`ffi/lambda` の routing、ワークスペースの coordination、Editor 側の metadata 受け渡し、typed workflow port handler を追加し、Lambda example を Cognition 側へ接続する準備が整ってきた。Lambda example を単なるサンプルではなく、ワークスペースや Cognition の実験台として使えるようにするための作業だ。

### loom

`Memo` から `Derived` への用語・API 整理に合わせて `examples/lambda` を更新した。seam への direct CST query helper と docs への CST projection guide も追加した。

### moondsp

Loom mini CST から projection method IR を検証した。[moondsp PR #80](https://github.com/dowdiness/moondsp/pull/80) で projection method IR を validate し、apply-edit の自動テストや loop expression の style guidance も追加した。

### js_engine

construct / call コンテキストの明示化を進めた。ArrayBuffer の `RealmState` 移行に続き、construction state を明示的な call コンテキストへ移し、ambient interpreter context への fallback を削除した。

主なPR / Issue: canopy [#345](https://github.com/dowdiness/canopy/pull/345), [#347](https://github.com/dowdiness/canopy/pull/347), [#348](https://github.com/dowdiness/canopy/pull/348), [#349](https://github.com/dowdiness/canopy/pull/349), [#350](https://github.com/dowdiness/canopy/pull/350) / loom [#152](https://github.com/dowdiness/loom/pull/152), [#154](https://github.com/dowdiness/loom/pull/154), [#155](https://github.com/dowdiness/loom/pull/155), [#156](https://github.com/dowdiness/loom/pull/156) / moondsp [#80](https://github.com/dowdiness/moondsp/pull/80), [#81](https://github.com/dowdiness/moondsp/pull/81), [#83](https://github.com/dowdiness/moondsp/pull/83), [#84](https://github.com/dowdiness/moondsp/pull/84), [#85](https://github.com/dowdiness/moondsp/pull/85), [#87](https://github.com/dowdiness/moondsp/pull/87) / js_engine [#152](https://github.com/dowdiness/js_engine/pull/152)

## 2026/5/26

### Canopy

Cognition の基盤を進めた。ワークスペース file の追跡 [Canopy PR #357](https://github.com/dowdiness/canopy/pull/357) に加え、minimal な incremental reactive layer、コンテキスト packing API、provider boundary の計画と docs を追加し、削除済みファイルの依存関係を掃除する修正も入れた。

あわせて Lambda 側は Loom の `LambdaAnalysis` attachment を使う形に寄せた。Cognition が参照するファイル・依存関係・コンテキストを明示的に扱えるようにし、後続のプロバイダ連携へ進む土台を作っている。

### incr

API の大きな整理を続けた。safe incremental refactor として types と correctness を整理し、pipeline traits を deprecated 扱いにした。expr formula API の設計も docs に残している。

### moondsp

Loom mini CST の coverage を広げた。slow postfix projection、degrade / euclid projection、dollar stack parity、sub-notation postfix parity、callback method projection などを追加し、Loom mini が production mini の構文にどこまで追いつけるかを確認した。

### js_engine

borrowed built-in realm の routing を修正した。[js_engine PR #153](https://github.com/dowdiness/js_engine/pull/153) で、built-in realm の扱いを明示的な routing に寄せている。

主なPR / Issue: canopy [#355](https://github.com/dowdiness/canopy/pull/355), [#357](https://github.com/dowdiness/canopy/pull/357), [#358](https://github.com/dowdiness/canopy/pull/358), [#359](https://github.com/dowdiness/canopy/pull/359), [#360](https://github.com/dowdiness/canopy/pull/360), [#362](https://github.com/dowdiness/canopy/pull/362), [#363](https://github.com/dowdiness/canopy/pull/363), [#364](https://github.com/dowdiness/canopy/pull/364) / incr [#87](https://github.com/dowdiness/incr/pull/87), [#89](https://github.com/dowdiness/incr/pull/89) / moondsp [#88](https://github.com/dowdiness/moondsp/pull/88), [#89](https://github.com/dowdiness/moondsp/pull/89), [#90](https://github.com/dowdiness/moondsp/pull/90), [#91](https://github.com/dowdiness/moondsp/pull/91), [#92](https://github.com/dowdiness/moondsp/pull/92), [#93](https://github.com/dowdiness/moondsp/pull/93), [#94](https://github.com/dowdiness/moondsp/pull/94), [#95](https://github.com/dowdiness/moondsp/pull/95) / js_engine [#153](https://github.com/dowdiness/js_engine/pull/153)

## 2026/5/27

### Canopy

provider boundary の設計を実装へ進めた。provider boundary domain を追加し、provider boundary plan を retarget した。前日までの recompute cleanup やコンテキスト packing を受け、Cognition が外部プロバイダへ渡す境界を整理する段階に入っている。プロバイダの結果をそのまま受け入れるのではなく、どの入力とコンテキストに対する結果かを追えるようにしておく必要がある。

### incr

Phase 3a facade migration の docs を入れ、evaluation strategy を kernel から切り出した。[incr PR #94](https://github.com/dowdiness/incr/pull/94) では現在の incr モデルを docs にまとめている。

### moondsp

Loom mini CST の known edge を characterize した。[moondsp PR #99](https://github.com/dowdiness/moondsp/pull/99) で、mode-incompatible な mini atom を reject する挙動や known edge の follow-up 状況を docs に残した。

### js_engine

startup benchmark の staging と benchmark summary 表示を整理した。[js_engine PR #154](https://github.com/dowdiness/js_engine/pull/154) で JS startup benchmark の足場を追加し、[js_engine PR #155](https://github.com/dowdiness/js_engine/pull/155) で benchmark dashboard の表示を整えた。

主なPR / Issue: canopy [#365](https://github.com/dowdiness/canopy/pull/365) / incr [#90](https://github.com/dowdiness/incr/pull/90), [#91](https://github.com/dowdiness/incr/pull/91), [#92](https://github.com/dowdiness/incr/pull/92), [#93](https://github.com/dowdiness/incr/pull/93), [#94](https://github.com/dowdiness/incr/pull/94), [#96](https://github.com/dowdiness/incr/pull/96) / moondsp [#96](https://github.com/dowdiness/moondsp/pull/96), [#97](https://github.com/dowdiness/moondsp/pull/97), [#99](https://github.com/dowdiness/moondsp/pull/99), [#100](https://github.com/dowdiness/moondsp/pull/100), [#102](https://github.com/dowdiness/moondsp/pull/102) / js_engine [#154](https://github.com/dowdiness/js_engine/pull/154), [#155](https://github.com/dowdiness/js_engine/pull/155)

## 2026/5/28

### Canopy

provider planning と Lambda semantic 側の作業を続けた。provider planning と lambda semantic overlay を追加し、ワークスペース memo の smoke test、memo lifecycle API、reactive provider boundary driver まで進んだ。`lib/cognition/provider_boundary_store.mbt` と `lib/cognition/reactive.mbt` に provider planning graph を接続し、cancellation・completion・retry classification・driver action を `@incr` の内部状態として扱う方向が固まってきた。

テストも増やした。provider cancellation の idempotency、driver shutdown 時に pending request を観測できること、file removal や budgeted context 変更後に stale な completion を拒否すること。プロバイダ応答が遅れて返ってきたときに、古いコンテキストの結果を現在の状態へ混ぜないための整理だ。

Lambda・JSON・Markdown の FFI read accessor も coordinator 経由の protected read へ寄せた。ワークスペース更新中に外側から半端な状態を読まれないようにする変更で、プロバイダ連携を進める前の境界固めにあたる。

### incr

runtime evaluation event API まわりを進めた。internal runtime evaluation events と evaluation strategy bundle を追加し、static derived fast path の benchmark も取った。honest read-error ownership の設計を docs に残し、`Derived::fallible` / `DerivedMap::fallible` を追加した。

### moondsp

loom-mini-cst の provenance matrix coverage と control method projection parity を追加した。Loom 移行に向け、upstream へ要求する挙動と recovery state を docs に分け、回復処理の evidence も増やした。

### js_engine

closure-converted block bodies を最適化した。続けて opt-in の bytecode prototype を追加し、既存 interpreter を残したまま bytecode 実行経路を育てる準備に入った。

主なPR / Issue: canopy [#367](https://github.com/dowdiness/canopy/pull/367), [#368](https://github.com/dowdiness/canopy/pull/368), [#370](https://github.com/dowdiness/canopy/pull/370), [#372](https://github.com/dowdiness/canopy/pull/372), [#374](https://github.com/dowdiness/canopy/pull/374), [#375](https://github.com/dowdiness/canopy/pull/375), [#376](https://github.com/dowdiness/canopy/pull/376), [#377](https://github.com/dowdiness/canopy/pull/377), [#378](https://github.com/dowdiness/canopy/pull/378), [#379](https://github.com/dowdiness/canopy/pull/379) / incr [#95](https://github.com/dowdiness/incr/pull/95), [#97](https://github.com/dowdiness/incr/pull/97), [#98](https://github.com/dowdiness/incr/pull/98) / moondsp [#101](https://github.com/dowdiness/moondsp/pull/101), [#104](https://github.com/dowdiness/moondsp/pull/104), [#106](https://github.com/dowdiness/moondsp/pull/106), [#107](https://github.com/dowdiness/moondsp/pull/107), [#108](https://github.com/dowdiness/moondsp/pull/108) / js_engine [#156](https://github.com/dowdiness/js_engine/pull/156), [#157](https://github.com/dowdiness/js_engine/pull/157)

## 2026/5/29

### Canopy

前日まで進めていた Cognition から少し離れ、repo 全体の再利用性と整理に手を入れた。agent reuse protocol の docs を追加し、レビュー指摘を受けて API map、PR template、package overview の型まわりも直した。

MoonBit の idiom sweep として、core / projection、lang/json、lang/lambda、editor まわりで guard、pattern matching、loop idiom、`ProjNode::id()` の使い方を整理した。tree-editor の file split も入れ、後続の変更で触る範囲を読みやすくした。

大きめの変更として、editor 内にあった ephemeral presence subsystem を `dowdiness/canopy/ephemeral` へ切り出した。さらに wire primitive を汎用の `lib/byte-codec` として抽出し、relay の wire codec もそこへ移した。presence と relay がそれぞれ似た wire 処理を持つのではなく、低レベルの byte 列変換を共通部品として扱う形になった。

Canvas 側では connection drag 中の preview port compatibility を追加した。接続を引いている途中でも port の互換性を確認しながら preview でき、グラフ編集の手触りが良くなった。

### moondsp

Loom mini CST projection での helper 利用を進めた。projection identity helper と optional-edit projection helper を使うようにし、Loom 側に寄せた projection API で spec を保てるか確認している。

### js_engine

bytecode 実行経路を広げた。short-circuit operator と comma expression の bytecode 対応を追加し、sloppy arguments formal binding、double super initialization、async generator function の name / length、destructuring rest parameter の扱いを順に修正した。

bytecode はまだ opt-in 段階だが、式や関数境界の細かい仕様ケースを通しながら interpreter との差分を潰している。

主なPR / Issue: canopy [#381](https://github.com/dowdiness/canopy/pull/381), [#382](https://github.com/dowdiness/canopy/pull/382), [#383](https://github.com/dowdiness/canopy/pull/383), [#385](https://github.com/dowdiness/canopy/pull/385), [#387](https://github.com/dowdiness/canopy/pull/387), [#388](https://github.com/dowdiness/canopy/pull/388), [#390](https://github.com/dowdiness/canopy/pull/390), [#391](https://github.com/dowdiness/canopy/pull/391), [#392](https://github.com/dowdiness/canopy/pull/392), [#394](https://github.com/dowdiness/canopy/pull/394) / moondsp [#109](https://github.com/dowdiness/moondsp/pull/109), [#110](https://github.com/dowdiness/moondsp/pull/110) / js_engine [#158](https://github.com/dowdiness/js_engine/pull/158), [#159](https://github.com/dowdiness/js_engine/pull/159), [#160](https://github.com/dowdiness/js_engine/pull/160), [#161](https://github.com/dowdiness/js_engine/pull/161), [#162](https://github.com/dowdiness/js_engine/pull/162), [#163](https://github.com/dowdiness/js_engine/pull/163)

## 2026/5/30

### Canopy

Lambda の scope graph を本格的に実装へ落とし始めた。NodeId をキーにした binding index を追加し、rename の binder lookup を古い `resolve_binder` から `@scope.declaration` へ移した。残っていた呼び出し側も `@scope.declaration` へ移し、module binder の `Decl.node_id` に関する production contract と cross-pipeline resolution equivalence をテストで固定した。

cross-pipeline の PBT を通す中で、module binder の `node_id` が実際の projection node を指していない問題もはっきりした。go-to-definition を作るときに邪魔になるため、既存 SourceMap token span から binder location を引ける Option D の設計に整理した。

### loom

Canopy 側の scope graph と projection identity を支える変更を進めた。CST token を source span として保持し、parser-owned reuse の rebase を取り戻し、source-span reuse API を固めた。さらに `ProjectionIdentityTracker` を追加し、projection identity を単発 helper ではなく、編集列をまたいで追跡できる部品にした。

### incr

typed spreadsheet demo を実際に触れる UI へ育てた。セル編集に始まり、50x50 の fullscreen sheet、inline edit、trace / evidence overlay、night theme を備えた Rabbita demo まで広げた。式の評価は MoonBit 側に残し、Rabbita は表示と操作の層に留める方針のままだ。

### moondsp

Loom 側で増えた projection identity helper を spec へ取り込んだ。Loom mini CST projection を production parser へすぐ置き換えるのではなく、まず spec の projection identity を上流 API に寄せ、移行時の前提を揃えている。

### js_engine

opt-in bytecode / VM prototype の coverage を大きく広げた。演算子、property access、call / construct、destructuring、eval、`super` などの実行経路を既存 runtime helper に寄せながら bytecode へ通し、未対応構文は明示的な unsupported 診断で落とすようにした。その後、bytecode performance microbenchmark の追加と、不要な arguments object setup を避ける最適化も入れた。

主なPR / Issue: canopy [#396](https://github.com/dowdiness/canopy/pull/396), [#397](https://github.com/dowdiness/canopy/pull/397), [#398](https://github.com/dowdiness/canopy/pull/398), [#399](https://github.com/dowdiness/canopy/pull/399), [#400](https://github.com/dowdiness/canopy/pull/400), [#401](https://github.com/dowdiness/canopy/pull/401), [#402](https://github.com/dowdiness/canopy/pull/402), [#403](https://github.com/dowdiness/canopy/pull/403) / loom [#188](https://github.com/dowdiness/loom/pull/188), [#189](https://github.com/dowdiness/loom/pull/189), [#190](https://github.com/dowdiness/loom/pull/190), [#191](https://github.com/dowdiness/loom/pull/191), [#192](https://github.com/dowdiness/loom/pull/192) / incr [#117](https://github.com/dowdiness/incr/pull/117), [#118](https://github.com/dowdiness/incr/pull/118) / moondsp [#111](https://github.com/dowdiness/moondsp/pull/111) / js_engine [#164](https://github.com/dowdiness/js_engine/pull/164), [#171](https://github.com/dowdiness/js_engine/pull/171), [#172](https://github.com/dowdiness/js_engine/pull/172)

## 2026/5/31

### Canopy

前日に設計した scope graph の binder location を実装した。`@scope.binder_span` と `@scope.go_to_definition` を追加し、`references` を `DeclId` キーに移して、module binder の synthetic `node_id` に依存しない形にした。incremental と full pipeline の差分テストも追加し、FlatProj reuse や `@incr` memo stack を通しても同じ解決結果になることを確認している。

incr の read channel が `ReadError` を返すようになったのに合わせ、coordinator 側でも ReadError を伝播するようにした。scope graph 側では module edit の reference を identity ベースにし、edit capture check や Ideal の scope annotation も canonical な `@scope` graph から導出する形へ寄せた。UI の highlight と scope graph の解決結果が別 resolver を持つ状態から、これで一歩抜け出せた。

docs 側では repository responsibility map、GUI layer integration report、module 一覧と `.gitmodules` / `moon.mod.json` の整合性を整理した。Structure mode の fallback document も schema valid に直している。

### loom

前日の `ProjectionIdentityTracker` を、失敗した edit や malformed damage をまたいで compose できるようにした。また、incr 側の typed spreadsheet demo、ReadError、accumulator ReadError に合わせて submodule を更新し、Canopy や moondsp が同じ基盤を参照できるようにした。

### incr

honest read-error ownership の Tier 2 として、public read channel を `CycleError` から `ReadError` へ広げた。これで dispose された cell の読み取りを catch できない abort ではなく `Err(Disposed(_))` として扱える。`Derived::fallible` の recipe と ReachableDerived の ADR も docs に追加し、typed spreadsheet demo 側では GC rooting と per-edit evidence snapshot の上限も直した。

その後、static `Derived` の private path を通常経路へ昇格させ、disposed cell id の dependent guard、Datalog relation の net change publish、accumulator read の `ReadError` 対応も入れた。typed spreadsheet demo には Cloudflare Pages への deploy workflow を追加し、Node 24 actions にも合わせた。

### moondsp

Loom の tracker edit composition を spec 側で消費し、Loom promotion notes も現状に合わせて更新した。web 側では live UI から song playback を触れるようにし、multiline song の help や global BPM の説明も補った。

### js_engine

bytecode prototype の性能を測る入口を整えた。PR ごとに base-vs-head の benchmark を出せるようにし、live benchmark dashboard を再設計して commit date や scan control を見やすくした。さらに plain object property helper と bytecode environment lookup の hot path を最適化し、startup Hyperfine workflow、startup decomposition helper、startup phase breakdown benchmark を追加した。

主なPR / Issue: canopy [#404](https://github.com/dowdiness/canopy/pull/404), [#405](https://github.com/dowdiness/canopy/pull/405), [#406](https://github.com/dowdiness/canopy/pull/406), [#407](https://github.com/dowdiness/canopy/pull/407), [#408](https://github.com/dowdiness/canopy/pull/408), [#409](https://github.com/dowdiness/canopy/pull/409), [#410](https://github.com/dowdiness/canopy/pull/410), [#411](https://github.com/dowdiness/canopy/pull/411), [#412](https://github.com/dowdiness/canopy/pull/412), [#420](https://github.com/dowdiness/canopy/pull/420), [#421](https://github.com/dowdiness/canopy/pull/421), [#426](https://github.com/dowdiness/canopy/pull/426), [#427](https://github.com/dowdiness/canopy/pull/427), [#431](https://github.com/dowdiness/canopy/pull/431), [#432](https://github.com/dowdiness/canopy/pull/432), [#433](https://github.com/dowdiness/canopy/pull/433) / loom [#197](https://github.com/dowdiness/loom/pull/197), [#198](https://github.com/dowdiness/loom/pull/198), [#199](https://github.com/dowdiness/loom/pull/199), [#200](https://github.com/dowdiness/loom/pull/200), [#201](https://github.com/dowdiness/loom/pull/201), [#204](https://github.com/dowdiness/loom/pull/204) / incr [#119](https://github.com/dowdiness/incr/pull/119), [#120](https://github.com/dowdiness/incr/pull/120), [#125](https://github.com/dowdiness/incr/pull/125), [#126](https://github.com/dowdiness/incr/pull/126), [#127](https://github.com/dowdiness/incr/pull/127), [#132](https://github.com/dowdiness/incr/pull/132), [#133](https://github.com/dowdiness/incr/pull/133), [#134](https://github.com/dowdiness/incr/pull/134), [#135](https://github.com/dowdiness/incr/pull/135), [#136](https://github.com/dowdiness/incr/pull/136), [#137](https://github.com/dowdiness/incr/pull/137), [#141](https://github.com/dowdiness/incr/pull/141), [#142](https://github.com/dowdiness/incr/pull/142), [#144](https://github.com/dowdiness/incr/pull/144), [#145](https://github.com/dowdiness/incr/pull/145) / moondsp [#112](https://github.com/dowdiness/moondsp/pull/112), [#113](https://github.com/dowdiness/moondsp/pull/113), [#115](https://github.com/dowdiness/moondsp/pull/115), [#116](https://github.com/dowdiness/moondsp/pull/116) / js_engine [#173](https://github.com/dowdiness/js_engine/pull/173), [#174](https://github.com/dowdiness/js_engine/pull/174), [#175](https://github.com/dowdiness/js_engine/pull/175), [#176](https://github.com/dowdiness/js_engine/pull/176), [#177](https://github.com/dowdiness/js_engine/pull/177), [#178](https://github.com/dowdiness/js_engine/pull/178), [#182](https://github.com/dowdiness/js_engine/pull/182), [#183](https://github.com/dowdiness/js_engine/pull/183), [#184](https://github.com/dowdiness/js_engine/pull/184)

## PR索引

週ごとに折りたたんだ PR / Issue 一覧。GitHub 上の詳細への索引。

<details>
<summary>5月第1週（5/7〜5/9）</summary>

### 2026/5/7

**Canopy / loom**

canopy [#225](https://github.com/dowdiness/canopy/pull/225) / loom [#100](https://github.com/dowdiness/loom/pull/100), [#101](https://github.com/dowdiness/loom/pull/101), [#102](https://github.com/dowdiness/loom/pull/102)

### 2026/5/8

**Canopy / loom**

canopy [#227](https://github.com/dowdiness/canopy/pull/227), [#228](https://github.com/dowdiness/canopy/pull/228), [#229](https://github.com/dowdiness/canopy/pull/229), [#230](https://github.com/dowdiness/canopy/pull/230), [#231](https://github.com/dowdiness/canopy/pull/231), [#232](https://github.com/dowdiness/canopy/pull/232), [#233](https://github.com/dowdiness/canopy/pull/233), [#234](https://github.com/dowdiness/canopy/pull/234), [#235](https://github.com/dowdiness/canopy/pull/235)

### 2026/5/9

**Unicode監査**

canopy [#236](https://github.com/dowdiness/canopy/pull/236), [#238](https://github.com/dowdiness/canopy/pull/238), [#239](https://github.com/dowdiness/canopy/pull/239), [#240](https://github.com/dowdiness/canopy/pull/240), [#241](https://github.com/dowdiness/canopy/pull/241)

</details>

<details>
<summary>5月第2週（5/10〜5/16）</summary>

### 2026/5/10

**moji / loom**

canopy [#242](https://github.com/dowdiness/canopy/pull/242), [#243](https://github.com/dowdiness/canopy/pull/243), [#245](https://github.com/dowdiness/canopy/pull/245), [#246](https://github.com/dowdiness/canopy/pull/246), [#247](https://github.com/dowdiness/canopy/pull/247), [#248](https://github.com/dowdiness/canopy/pull/248), [#249](https://github.com/dowdiness/canopy/pull/249), [#251](https://github.com/dowdiness/canopy/pull/251), [#252](https://github.com/dowdiness/canopy/pull/252) / loom [#108](https://github.com/dowdiness/loom/pull/108)〜[#120](https://github.com/dowdiness/loom/pull/120)

### 2026/5/13

**ベンチマーク**

canopy [#223](https://github.com/dowdiness/canopy/pull/223), [#256](https://github.com/dowdiness/canopy/pull/256), [#257](https://github.com/dowdiness/canopy/pull/257), [#258](https://github.com/dowdiness/canopy/pull/258), [#259](https://github.com/dowdiness/canopy/pull/259), [#260](https://github.com/dowdiness/canopy/pull/260)

### 2026/5/14

**Canvas handles**

canopy [#262](https://github.com/dowdiness/canopy/pull/262)

### 2026/5/15

**Canvas smoke test**

canopy [#264](https://github.com/dowdiness/canopy/pull/264), [#265](https://github.com/dowdiness/canopy/pull/265), [#266](https://github.com/dowdiness/canopy/pull/266)

### 2026/5/16

**ZWSP / incr**

canopy [#268](https://github.com/dowdiness/canopy/pull/268)〜[#278](https://github.com/dowdiness/canopy/pull/278) / incr [#49](https://github.com/dowdiness/incr/pull/49), [#50](https://github.com/dowdiness/incr/pull/50), [#51](https://github.com/dowdiness/incr/pull/51)

</details>

<details>
<summary>5月第3週（5/17）</summary>

### 2026/5/17

**Intent panel**

canopy [#288](https://github.com/dowdiness/canopy/pull/288)〜[#294](https://github.com/dowdiness/canopy/pull/294) / loom [#121](https://github.com/dowdiness/loom/pull/121), [#122](https://github.com/dowdiness/loom/pull/122), [#123](https://github.com/dowdiness/loom/pull/123)

</details>

<details>
<summary>5月第4週（5/18〜5/24）</summary>

### 2026/5/18

**Rabbita / loom**

loom [#124](https://github.com/dowdiness/loom/pull/124) / canopy [#293](https://github.com/dowdiness/canopy/pull/293), [#296](https://github.com/dowdiness/canopy/pull/296)

### 2026/5/19

**Canopy / moondsp**

canopy [#297](https://github.com/dowdiness/canopy/pull/297), [#299](https://github.com/dowdiness/canopy/pull/299), [#300](https://github.com/dowdiness/canopy/pull/300), [#301](https://github.com/dowdiness/canopy/pull/301), [#302](https://github.com/dowdiness/canopy/pull/302) / loom [#126](https://github.com/dowdiness/loom/pull/126), [#129](https://github.com/dowdiness/loom/pull/129) / moondsp [#60](https://github.com/dowdiness/moondsp/pull/60), [#62](https://github.com/dowdiness/moondsp/pull/62)

### 2026/5/20

**Rabbita CodeMirror**

canopy [#303](https://github.com/dowdiness/canopy/pull/303), [#305](https://github.com/dowdiness/canopy/pull/305), [#306](https://github.com/dowdiness/canopy/pull/306), [#307](https://github.com/dowdiness/canopy/pull/307) / loom [#131](https://github.com/dowdiness/loom/pull/131), [#132](https://github.com/dowdiness/loom/pull/132) / incr [#58](https://github.com/dowdiness/incr/pull/58), [#59](https://github.com/dowdiness/incr/pull/59), [#60](https://github.com/dowdiness/incr/pull/60) / moondsp [#63](https://github.com/dowdiness/moondsp/pull/63) / js_engine [#130](https://github.com/dowdiness/js_engine/pull/130)

### 2026/5/21

**DOM boundary / loom**

canopy [#312](https://github.com/dowdiness/canopy/pull/312), [#313](https://github.com/dowdiness/canopy/pull/313), [#316](https://github.com/dowdiness/canopy/pull/316) / loom [#134](https://github.com/dowdiness/loom/pull/134), [#135](https://github.com/dowdiness/loom/pull/135), [#136](https://github.com/dowdiness/loom/pull/136) / incr [#61](https://github.com/dowdiness/incr/pull/61), [#62](https://github.com/dowdiness/incr/pull/62), [#63](https://github.com/dowdiness/incr/pull/63) / js_engine [#131](https://github.com/dowdiness/js_engine/pull/131), [#132](https://github.com/dowdiness/js_engine/pull/132)

### 2026/5/22

**Inspector / incr**

canopy [#318](https://github.com/dowdiness/canopy/pull/318), [#320](https://github.com/dowdiness/canopy/pull/320), [#321](https://github.com/dowdiness/canopy/pull/321) / incr [#68](https://github.com/dowdiness/incr/pull/68), [#69](https://github.com/dowdiness/incr/pull/69), [#70](https://github.com/dowdiness/incr/pull/70), [#71](https://github.com/dowdiness/incr/pull/71), [#72](https://github.com/dowdiness/incr/pull/72) / js_engine [#133](https://github.com/dowdiness/js_engine/pull/133), [#134](https://github.com/dowdiness/js_engine/pull/134)

### 2026/5/23

**Op log / incr**

canopy [#323](https://github.com/dowdiness/canopy/pull/323), [#324](https://github.com/dowdiness/canopy/pull/324), [#327](https://github.com/dowdiness/canopy/pull/327), [#329](https://github.com/dowdiness/canopy/pull/329) / incr [#73](https://github.com/dowdiness/incr/pull/73), [#74](https://github.com/dowdiness/incr/pull/74), [#75](https://github.com/dowdiness/incr/pull/75), [#76](https://github.com/dowdiness/incr/pull/76), [#77](https://github.com/dowdiness/incr/pull/77), [#78](https://github.com/dowdiness/incr/pull/78), [#79](https://github.com/dowdiness/incr/pull/79) / js_engine [#135](https://github.com/dowdiness/js_engine/pull/135), [#136](https://github.com/dowdiness/js_engine/pull/136), [#137](https://github.com/dowdiness/js_engine/pull/137), [#138](https://github.com/dowdiness/js_engine/pull/138), [#139](https://github.com/dowdiness/js_engine/pull/139)

### 2026/5/24

**text-change移行**

loom [#149](https://github.com/dowdiness/loom/pull/149) / canopy [#341](https://github.com/dowdiness/canopy/pull/341) / incr [#81](https://github.com/dowdiness/incr/pull/81), [#82](https://github.com/dowdiness/incr/pull/82), [#83](https://github.com/dowdiness/incr/pull/83) / moondsp [#71](https://github.com/dowdiness/moondsp/pull/71), [#73](https://github.com/dowdiness/moondsp/pull/73), [#74](https://github.com/dowdiness/moondsp/pull/74), [#75](https://github.com/dowdiness/moondsp/pull/75), [#76](https://github.com/dowdiness/moondsp/pull/76), [#79](https://github.com/dowdiness/moondsp/pull/79) / js_engine [#140](https://github.com/dowdiness/js_engine/pull/140), [#142](https://github.com/dowdiness/js_engine/pull/142), [#143](https://github.com/dowdiness/js_engine/pull/143), [#144](https://github.com/dowdiness/js_engine/pull/144), [#146](https://github.com/dowdiness/js_engine/pull/146), [#147](https://github.com/dowdiness/js_engine/pull/147), [#148](https://github.com/dowdiness/js_engine/pull/148), [#149](https://github.com/dowdiness/js_engine/pull/149), [#151](https://github.com/dowdiness/js_engine/pull/151)

</details>

<details>
<summary>5月第5週（5/25〜5/31）</summary>

### 2026/5/25

**Cognition / loom**

canopy [#345](https://github.com/dowdiness/canopy/pull/345), [#347](https://github.com/dowdiness/canopy/pull/347), [#348](https://github.com/dowdiness/canopy/pull/348), [#349](https://github.com/dowdiness/canopy/pull/349), [#350](https://github.com/dowdiness/canopy/pull/350) / loom [#152](https://github.com/dowdiness/loom/pull/152), [#154](https://github.com/dowdiness/loom/pull/154), [#155](https://github.com/dowdiness/loom/pull/155), [#156](https://github.com/dowdiness/loom/pull/156) / moondsp [#80](https://github.com/dowdiness/moondsp/pull/80), [#81](https://github.com/dowdiness/moondsp/pull/81), [#83](https://github.com/dowdiness/moondsp/pull/83), [#84](https://github.com/dowdiness/moondsp/pull/84), [#85](https://github.com/dowdiness/moondsp/pull/85), [#87](https://github.com/dowdiness/moondsp/pull/87) / js_engine [#152](https://github.com/dowdiness/js_engine/pull/152)

### 2026/5/26

**Cognition基盤**

canopy [#355](https://github.com/dowdiness/canopy/pull/355), [#357](https://github.com/dowdiness/canopy/pull/357), [#358](https://github.com/dowdiness/canopy/pull/358), [#359](https://github.com/dowdiness/canopy/pull/359), [#360](https://github.com/dowdiness/canopy/pull/360), [#362](https://github.com/dowdiness/canopy/pull/362), [#363](https://github.com/dowdiness/canopy/pull/363), [#364](https://github.com/dowdiness/canopy/pull/364) / incr [#87](https://github.com/dowdiness/incr/pull/87), [#89](https://github.com/dowdiness/incr/pull/89) / moondsp [#88](https://github.com/dowdiness/moondsp/pull/88), [#89](https://github.com/dowdiness/moondsp/pull/89), [#90](https://github.com/dowdiness/moondsp/pull/90), [#91](https://github.com/dowdiness/moondsp/pull/91), [#92](https://github.com/dowdiness/moondsp/pull/92), [#93](https://github.com/dowdiness/moondsp/pull/93), [#94](https://github.com/dowdiness/moondsp/pull/94), [#95](https://github.com/dowdiness/moondsp/pull/95) / js_engine [#153](https://github.com/dowdiness/js_engine/pull/153)

### 2026/5/27

**provider boundary**

canopy [#365](https://github.com/dowdiness/canopy/pull/365) / incr [#90](https://github.com/dowdiness/incr/pull/90), [#91](https://github.com/dowdiness/incr/pull/91), [#92](https://github.com/dowdiness/incr/pull/92), [#93](https://github.com/dowdiness/incr/pull/93), [#94](https://github.com/dowdiness/incr/pull/94), [#96](https://github.com/dowdiness/incr/pull/96) / moondsp [#96](https://github.com/dowdiness/moondsp/pull/96), [#97](https://github.com/dowdiness/moondsp/pull/97), [#99](https://github.com/dowdiness/moondsp/pull/99), [#100](https://github.com/dowdiness/moondsp/pull/100), [#102](https://github.com/dowdiness/moondsp/pull/102) / js_engine [#154](https://github.com/dowdiness/js_engine/pull/154), [#155](https://github.com/dowdiness/js_engine/pull/155)

### 2026/5/28

**provider planning**

canopy [#367](https://github.com/dowdiness/canopy/pull/367), [#368](https://github.com/dowdiness/canopy/pull/368), [#370](https://github.com/dowdiness/canopy/pull/370), [#372](https://github.com/dowdiness/canopy/pull/372), [#374](https://github.com/dowdiness/canopy/pull/374), [#375](https://github.com/dowdiness/canopy/pull/375), [#376](https://github.com/dowdiness/canopy/pull/376), [#377](https://github.com/dowdiness/canopy/pull/377), [#378](https://github.com/dowdiness/canopy/pull/378), [#379](https://github.com/dowdiness/canopy/pull/379) / incr [#95](https://github.com/dowdiness/incr/pull/95), [#97](https://github.com/dowdiness/incr/pull/97), [#98](https://github.com/dowdiness/incr/pull/98) / moondsp [#101](https://github.com/dowdiness/moondsp/pull/101), [#104](https://github.com/dowdiness/moondsp/pull/104), [#106](https://github.com/dowdiness/moondsp/pull/106), [#107](https://github.com/dowdiness/moondsp/pull/107), [#108](https://github.com/dowdiness/moondsp/pull/108) / js_engine [#156](https://github.com/dowdiness/js_engine/pull/156), [#157](https://github.com/dowdiness/js_engine/pull/157)

### 2026/5/29

**ephemeral / byte-codec**

canopy [#381](https://github.com/dowdiness/canopy/pull/381), [#382](https://github.com/dowdiness/canopy/pull/382), [#383](https://github.com/dowdiness/canopy/pull/383), [#385](https://github.com/dowdiness/canopy/pull/385), [#387](https://github.com/dowdiness/canopy/pull/387), [#388](https://github.com/dowdiness/canopy/pull/388), [#390](https://github.com/dowdiness/canopy/pull/390), [#391](https://github.com/dowdiness/canopy/pull/391), [#392](https://github.com/dowdiness/canopy/pull/392), [#394](https://github.com/dowdiness/canopy/pull/394) / moondsp [#109](https://github.com/dowdiness/moondsp/pull/109), [#110](https://github.com/dowdiness/moondsp/pull/110) / js_engine [#158](https://github.com/dowdiness/js_engine/pull/158), [#159](https://github.com/dowdiness/js_engine/pull/159), [#160](https://github.com/dowdiness/js_engine/pull/160), [#161](https://github.com/dowdiness/js_engine/pull/161), [#162](https://github.com/dowdiness/js_engine/pull/162), [#163](https://github.com/dowdiness/js_engine/pull/163)

### 2026/5/30

**scope graph**

canopy [#396](https://github.com/dowdiness/canopy/pull/396), [#397](https://github.com/dowdiness/canopy/pull/397), [#398](https://github.com/dowdiness/canopy/pull/398), [#399](https://github.com/dowdiness/canopy/pull/399), [#400](https://github.com/dowdiness/canopy/pull/400), [#401](https://github.com/dowdiness/canopy/pull/401), [#402](https://github.com/dowdiness/canopy/pull/402), [#403](https://github.com/dowdiness/canopy/pull/403) / loom [#188](https://github.com/dowdiness/loom/pull/188), [#189](https://github.com/dowdiness/loom/pull/189), [#190](https://github.com/dowdiness/loom/pull/190), [#191](https://github.com/dowdiness/loom/pull/191), [#192](https://github.com/dowdiness/loom/pull/192) / incr [#117](https://github.com/dowdiness/incr/pull/117), [#118](https://github.com/dowdiness/incr/pull/118) / moondsp [#111](https://github.com/dowdiness/moondsp/pull/111) / js_engine [#164](https://github.com/dowdiness/js_engine/pull/164), [#171](https://github.com/dowdiness/js_engine/pull/171), [#172](https://github.com/dowdiness/js_engine/pull/172)

### 2026/5/31

**go-to-definition / ReadError**

canopy [#404](https://github.com/dowdiness/canopy/pull/404), [#405](https://github.com/dowdiness/canopy/pull/405), [#406](https://github.com/dowdiness/canopy/pull/406), [#407](https://github.com/dowdiness/canopy/pull/407), [#408](https://github.com/dowdiness/canopy/pull/408), [#409](https://github.com/dowdiness/canopy/pull/409), [#410](https://github.com/dowdiness/canopy/pull/410), [#411](https://github.com/dowdiness/canopy/pull/411), [#412](https://github.com/dowdiness/canopy/pull/412), [#420](https://github.com/dowdiness/canopy/pull/420), [#421](https://github.com/dowdiness/canopy/pull/421), [#426](https://github.com/dowdiness/canopy/pull/426), [#427](https://github.com/dowdiness/canopy/pull/427), [#431](https://github.com/dowdiness/canopy/pull/431), [#432](https://github.com/dowdiness/canopy/pull/432), [#433](https://github.com/dowdiness/canopy/pull/433) / loom [#197](https://github.com/dowdiness/loom/pull/197), [#198](https://github.com/dowdiness/loom/pull/198), [#199](https://github.com/dowdiness/loom/pull/199), [#200](https://github.com/dowdiness/loom/pull/200), [#201](https://github.com/dowdiness/loom/pull/201), [#204](https://github.com/dowdiness/loom/pull/204) / incr [#119](https://github.com/dowdiness/incr/pull/119), [#120](https://github.com/dowdiness/incr/pull/120), [#125](https://github.com/dowdiness/incr/pull/125), [#126](https://github.com/dowdiness/incr/pull/126), [#127](https://github.com/dowdiness/incr/pull/127), [#132](https://github.com/dowdiness/incr/pull/132), [#133](https://github.com/dowdiness/incr/pull/133), [#134](https://github.com/dowdiness/incr/pull/134), [#135](https://github.com/dowdiness/incr/pull/135), [#136](https://github.com/dowdiness/incr/pull/136), [#137](https://github.com/dowdiness/incr/pull/137), [#141](https://github.com/dowdiness/incr/pull/141), [#142](https://github.com/dowdiness/incr/pull/142), [#144](https://github.com/dowdiness/incr/pull/144), [#145](https://github.com/dowdiness/incr/pull/145) / moondsp [#112](https://github.com/dowdiness/moondsp/pull/112), [#113](https://github.com/dowdiness/moondsp/pull/113), [#115](https://github.com/dowdiness/moondsp/pull/115), [#116](https://github.com/dowdiness/moondsp/pull/116) / js_engine [#173](https://github.com/dowdiness/js_engine/pull/173), [#174](https://github.com/dowdiness/js_engine/pull/174), [#175](https://github.com/dowdiness/js_engine/pull/175), [#176](https://github.com/dowdiness/js_engine/pull/176), [#177](https://github.com/dowdiness/js_engine/pull/177), [#178](https://github.com/dowdiness/js_engine/pull/178), [#182](https://github.com/dowdiness/js_engine/pull/182), [#183](https://github.com/dowdiness/js_engine/pull/183), [#184](https://github.com/dowdiness/js_engine/pull/184)

</details>

