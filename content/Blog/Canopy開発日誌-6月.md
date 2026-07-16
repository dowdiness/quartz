---
title: Canopy開発日誌-6月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-07-16T18:13:35+09:00
---

# Canopy開発日誌-6月

2026年6月のCanopy開発ログ（日次記録）。月の要約は[[Canopy開発日誌-6月-まとめ|6月-まとめ]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

## 今月の大きな流れ

月全体の流れは[[Canopy開発日誌-6月-まとめ|6月-まとめ]]にまとめた。以下は週次ヘッダと日付見出しによる作業記録。PR番号の一覧は文末の[PR索引](#pr索引)にある。


## 6月第1週: Ideal・Rabbita・Canvas の土台（6/1〜6/7）

LetDef導入と性能見送り、Rabbita headless UI、IdealのTailwind v4移行、source-backed Canvasの整備。性能より先に、構成を固めた週だった。

## 2026/6/1

### Canopy

Canopy と MoonDsp の関係を整理した。Canopy は構造編集の土台、MoonDsp は音楽 DSL / DSP 側の実験、incr / Loom は共有基盤として扱う方針にした。

Lambda projection では `ProjNode` 作成補助関数、SourceMap token helper、Block / Hole の型付き view を追加した。そのうえで module 直下の `let` 定義を `LetDef` という独立 projection node にした。以前は binding row が初期値 expression の ID や仮 ID を借りており、drag & drop や binding 単位 edit で「何を動かしているのか」が曖昧だった。`LetDef` が実 node になったことで、binding row を構造編集の対象として素直に扱えるようになった。

性能面では `to_flat_proj_incremental` が本当に遅いかを BAND 2b で確認した。1000 個の定義では数 ms まで伸びるが、現実の Canopy 文書ではまだその規模にならない。そのためすぐ最適化せず、必要になった時点で再開することにした。

### loom / js_engine / 運用

Loom では json-settings example に last-good semantic projection attachment を追加した。parse や semantic 変換に失敗しても、最後に成功した projection を保持して UI を壊さない仕組みになる。Lambda example には Block / Hole / LetDef 系の typed view を入れた。js_engine では repeat benchmark runner と Array mutator の hole / `undefined` まわりを整理した。

運用面では、Codex を pre-PR review だけでなく実装計画の執筆にも使う流れが固まり始めた。


## 2026/6/2

### Canopy

前日の性能調査を JS target でも確認した。遅さの原因は木を歩く回数ではなく、再利用された CstNode 同士を比較するときの pointer chasing、つまり cache 内参照をたどるコストが支配的だった。ここでも最適化は見送り、大きな document を扱う必要が出てから手を付けることにした。

Lambda projection では `FlatProj` を `ModuleProjection` へ改名した。単なる名前変更ではなく、「module 全体の投影結果を差分更新する単位」だと分かる名前にした。ReuseCursor、ProjectionIdentityTracker、incremental projection の設計上の迷いも ADR に分けて記録し、microbenchmark も本番と同じ ID 採番になるよう合わせた。

### Incr visualizer / Canvas

IncrGraph panel は依存関係の形だけでなく、どの cell が再計算され、どれくらい時間がかかったかも見えるようにした。cell ごとの最新 event だけを持つ buffer にし、履歴が増え続けないようにしたうえで、最新 recompute 時間と legend を追加した。遅い cell が視覚的に浮き、単なる構造図から調査用 panel へ近づいた。

Canvas では graph demo をより構造編集寄りに使うための準備を進めた。

### CI / 運用

Ideal web E2E を PR gate へ載せる準備を進めた。ブラウザ上で Ideal が壊れていないことを PR ごとの自動チェックに含める方向へ進めた。同時に、CI の一時的な失敗と本当の失敗の見分け方も整理した。


## 2026/6/3

### Canopy

MoonDsp / Canopy の全体方針を main へ入れた。source-backed canvas demo も、source text から作った graph をブラウザで触れるところまで進めた。

CI では Ideal web E2E を PR gate に載せた。Canopy の UI 変更をローカル手動確認だけでなく CI 上のブラウザテストで支える方向へ進んだ。

### MoonDsp / incr

MoonDsp では Canopy 連携を見据え Graph runtime の境界を整理した。audio callback へ editor 側の責務を持ち込まない、という境界が少しずつ明確になった。

incr 側では public event API の命名を Derived 寄りに整理した。


## 2026/6/4

### Canopy

Rabbita headless UI を Canopy で本当に使えるかを見極める日だった。Disclosure PoC と dialog spike で、Rabbita を Ideal UI の土台にできる感触を確認した。

### MoonDsp / js_engine

MoonDsp では Graph runtime の facade / internal boundary を切り始めた。外から使う公開 API と、内部だけで変えてよい実装部分を分ける作業になる。js_engine では Array method fast path delegation や Test262 runner shadow の準備が進んだ。


## 2026/6/5

### Canopy / Rabbita headless UI

Rabbita の patched 更新を取り込み、Ideal と Canvas から headless UI primitive を実際に使い始めた。headless UI primitive は、見た目の CSS を押し付けず、状態管理やキーボード操作だけを提供する部品のこと。Action Menu、ContextMenu、Tabs、TreeView まで進み、Ideal UI の土台が Rabbita へ寄っていった。特に ContextMenu は Canvas の右クリックメニューを実際の利用側とし、右クリック位置を基準にメニューを出すこと、外側クリックで閉じること、閉じた後に focus を戻すこと、画面外にはみ出さない配置まで確認した。

### incr / MoonDsp / js_engine

incr では typed spreadsheet と Incremental TEA の実験が進んだ。MoonDsp では editor / audio runtime の handoff contract を文書化し、js_engine では Array mutator や runner shadow 化を進めた。


## 2026/6/6

### Canopy / Ideal UI foundation

Ideal の UI 基盤を大きく整理した。パネルリサイズ、スクリーンリーダーへ状態を伝える live-region、重複 CSS 削除、Tailwind v4 移行、overlay、toolbar、bottom tabs、panel、inspector、outline resize handle を小さな単位で進めた。

### 周辺リポジトリ

MoonDsp では Graph runtime / scheduler / browser internals の分割が進んだ。Loom、incr、js_engine でも parser runtime、Incremental TEA、test262 runner の整備が続いた。


## 2026/6/7

### Canopy / Ideal and protocol

Ideal の残タスクと protocol の曖昧さを潰した。outline E2E、bridge が一部だけ成功した batch の扱い、cursor intent の単位名、protocol 上の座標が何を指すかの docs、MoonBit registry cache を入れた。

### Canvas / loom

Canvas は次の source-backed 段階へ戻した。source-backed とは、画面上の node/edge を直接正とするのではなく、裏の Graph DSL source を正としてそこから画面を作り直す方式のこと。`lib/canvas-graph` の抽出も進めた。

Loom では MoonBit parser integration が進み、editor へ渡せる syntax artifact を出せる方向へ寄っていった。


## 6月第2週: アーキテクチャ再設計と Lambda 投影（6/8〜6/14）

source-backed CanvasのCodeMirror化とLambda CstFoldの近代化が進んだ。§20完了の準備まで進めたが、性能最適化は先送りにした。

## 2026/6/8

### Canvas

Canvas の source-backed graph を CodeMirror source editor へ寄せた。source-backed inspector edit、selection remap、CM6 change delta lowering、CodeMirror source panel mount を順に入れた。UI state を直接いじるのではなく Graph DSL source へ下ろし、再パースされた source-backed graph を表示し直す流れがはっきりしてきた。

編集途中で parse に失敗したときの回復方針も決めた。失敗時に入力を巻き戻すのではなく、editor buffer を常に正しい入力として扱う。parse 成功時は current result を表示し、失敗時は最後に成功した last-good を表示する。

### loom / incr / js_engine

Loomではparser結果に基づいてsyntax roleの範囲を出すrole span、incrではIncremental TEAのkeyed DOM benchmark、js_engineではArray shift / unshiftのfast pathやrunner parityが進んだ。


## 2026/6/9

### Canopy

Canvas stable identity の PR2 を進めた。stable identity とは、source を編集して画面を作り直しても同じ node/edge を同じものとして扱える ID のこと。`NodeId` / `EdgeId` を source-backed な文字列 identity へ寄せ、binding remap 用の一時 shim を消す方向へ進めた。

Lambda 側では generic projection memos へ向かう前段として、scope graph や projection identity の整理が続いた。

### loom / MoonDsp

Loomではparser runtime attachmentやdeprecated syntax移行が進んだ。MoonDspではmini parser置き換えcampaignが進み、loomへの置き換え方針が具体化した。


## 2026/6/10

### incr / Incremental TEA

Incremental TEA を一気に仕上げた。renderer lifecycle、keyed VDOM diff、benchmark、subscriptions が merge され、prototype の主要 issue がすべて閉じた。

### MoonDsp / loom / Canopy / js_engine

MoonDsp は loom parser 置き換え campaign の Phase 2 parity を完走し、ADR-0016 を Accepted にした。Loom では separated-list や attachment 系が進み、Canopy では Canvas の runtime seam 整理が続いた。seam は境界面のことで、どこから先を runtime 責務にするかを明確にする作業だった。js_engine は v0.3.0 をリリースした。


## 2026/6/11

### loom

separated-list（#279）と group shape helpers（#196）を畳んだ。Loom 側の parser / syntax helper が Canopy の各言語実装を支えやすい形になってきた。

### Canopy / アーキテクチャ再設計

Canopy のアーキテクチャ再設計に着手した。S0 の proposal と API boundary ADR を入れ、S1 として protocol / wire を抽出した。

### js_engine / 運用

js_engine では CI cache 改善が効かないことを実測で確認し、test262 sharding へ方針を切り替えた。作業運用としては、相談・レビュー・実装計画をどのエージェントに任せるかの使い分けも少し固まった。


## 2026/6/12

### Canopy / アーキテクチャ再設計

再設計の S2 から S5a までを一気に進めた。editor から sync session と transport を切り出し、言語 runtime が editor に提供するインターフェース（lang/runtime SPI）、FFI や host 機能の登録場所、基盤層の責務ルール、依存方向をチェックする import-graph lint で境界を固定した。単なるファイル分割ではなく、editor、runtime、transport、host 機能が互いに勝手に漏れないようにするための制度化だった。

### プロダクト方針

Canopy を「editor / framework の証明」から「write-to-self の post product」へ寄せる方針転換を決めた。post product とは完成された一つのアプリというより、自分の思考や作業ログを後から自分へ返すための環境として Canopy を見る、という意味で使っている。最小の write→surface ループ prototype を置き、source retrieval より前に「書いたものが自分へ返ってくる」体験を確かめる方向へ進んだ。

### 周辺リポジトリ

incrではIncremental TEAのrendererとsubscriptionがさらに進み、Loomではparser runtime attachment、js_engineではarchitecture refactor Stage 0-7、MoonDspではscheduler周辺の責務分割が進んだ。


## 2026/6/13

### Canopy / editor-neutral test grammar

editor を language から完全に切り離すため neutral test grammar を入れた。Lambda のような実言語ではなく、editor 機能をテストするためだけの小さな構文で、editor テストが特定言語に引きずられないようにするもの。これにより editor テストが Lambda 固有構文に依存しすぎる問題を減らした。

### Codex連携

Codex app-server と MCP wrapper の使い分けを検証した。相談役は MCP、streaming や interactive approval など「Codex の上に作る」用途は app-server、という整理をした。

### プロダクトとCanvas

write→surface ループには、どのメモを再表示するかを決める resurfacing signal での並べ替えと、同じ入力から追加で問い直す same-input ask を追加した。Canvas では runtime との境界を切り出し、どちらが何を保証するかの contracts も整理した。

### loom / js_engine

LoomではMarkdownIRやMarkdown block reparseの準備が進み、js_engineではStage 0-7のarchitecture refactorが進んだ。


## 2026/6/14

### Canopy / Lambda CstFold modernization

Lambda 投影の CstFold 現代化を進めた。CstFold は Loom 側 CST（具象構文木）を畳み込んで AST 相当の構造へ変換する仕組みで、Canopy 側の手組み変換を減らす狙いがある。既存 Canopy semantics と Loom CstFold の差分を明示し、互換 adapter を置いたうえで leaf / if / app / bop などを段階的に CstFold 経由へ寄せた。たとえば `{ 1 }` や `{ }` の扱いは Loom 素の CstFold と Canopy editor semantics で異なるため、Loom 側を変えず Canopy 側 adapter で吸収する判断にした。

その流れで `DefinitionIndex` を切り出し、scope / edit / semantic / companion / ideal の各利用側を `ModuleProjection` から離し始めた。`DefinitionIndex` は、どの node がどの module 定義に対応するかを引く薄い索引になる。block-local rename も正しく動くようになった。

### Canvas / loom / incr / MoonDsp / js_engine

Canvasでは接続preview互換性をMoonBit側へ寄せ、source-backed demoの`defer_sync`順序を整理した。LoomではMarkdown incremental block reparse、incrではincr_tea benchmark、MoonDspではscheduler facade分割、js_engineではarchitecture redesign Stage 8-10が進んだ。


## 6月第3週: Lambda 健全性・SDEG・moon.mod 移行（6/15〜6/21）

Lambda編集の検証を「編集後テキストを再パースして確認する」方式に寄せた。Grove identity hint、analysis query layer、moon.mod移行が並行して進んだ。

## 2026/6/15

### Canopy / Lambda §20 completion

Lambda 編集の block-local 対応を一気に仕上げた。block-local 対応とは、root 直下の `let` だけでなく block 式内の `let` も rename / duplicate / move などの対象として正しく扱うこと。delete / duplicate / move binding ops、move op の scoping soundness、block-shadowing filter、typed `fn` token metadata、EditContext 整理を入れた。move では同 scope の前後関係だけでなく lambda param や外側 module def による shadowing も見ないと参照の向きが静かに変わることが分かり、scope graph resolution を使う形へ寄せた。

最後に legacy `ModuleProjection` を削除し、Lambda は generic projection memos へ完全移行した。Lambda 専用の古い projection cache をやめ、他言語と同じ汎用 memo stack で projection を作るという意味になる。これで docs/TODO.md §20 の binding clause は完了した。

### FFI / Ideal

`lib/js` を新設し、JS 値を MoonBit 側で抽象的に持つ opaque `Any` handle、JS property / method / global へ触る最低限の escape hatch、JSON、Promise bridge を置いた。escape hatch は型付き API がまだない JS 機能へ一時的にアクセスするための出口になる。これを使って `ffi/io` に file read/write を実装し、Ideal の Open / Save toolbar から呼べるようにした。

Ideal 側では `globalThis.__canopy_*` を単一の `__canopy_bridge` へ集約した。

### 周辺リポジトリ

LoomではMarkdownIR M0 policyとM1 heading/paragraph slice、incrではspreadsheet proofとinactive root、js_engineではES2024 Set methodsやinternal slots整理が進んだ。


## 2026/6/16

### Canopy / Lambda §20完了とalpha pilot

`ExtractToLet` を block-aware にした。block body path では body expression の直前に、block def path では block defs の前に `let` を挿入する。block scope と lambda scope を区別し、capture guard も追加した。root へ無理に hoist するのではなく、選択位置の scope に近い場所へ binding を作る方針にした。

Lambda の alpha-safe beta pilot を `lang/lambda/alpha` に置いた。変数名の偶然の一致で意味が変わらないよう binder identity で計算する実験になる。`ScopeGraph` から投影 term を内部表現へ lower し、root だけ beta reduction し、capture が起きない形で named `Term` へ戻す。

夕方以降、この alpha-safe core boundary を main へ入れ、binding-id compatibility fallback も削った。古い init-id でも binding を見つける互換経路を消し、実際の LetDef id だけを見るようにした。Ideal 側では Tabs UI helper を value-derived な小さな層へ切り出した。

### 周辺リポジトリ

LoomではMarkdownIR M1 vertical slice、incrではinactive-root cohort測定、js_engineではbytecode call-frame fast pathが進んだ。js_engineではparam bindingのenv round-tripをskipし、binding stepが大きく改善した。


## 2026/6/17

### Canopy / Lambda編集の健全性

Lambda 編集は「生成された文字列を見る」より「編集後テキストを再パースして意図した AST になるかを見る」方針へ寄せた。`_copy` のように見た目では小さな命名差に見えても、lexer で読めず AST に戻らないケースを踏んだため。

- block-local binding edit の敵対的ケースを追加した（move up/down、delete、root/block shadowing）。
- `ScopeGraph.node_scope` / `node_cutoffs` を private にし、query API 経由にした。内部 Map を直接読むのではなく、意味のある問い合わせ関数を通し、後で実装を変えやすくした。
- `ExtractToLet` は #674 の実装で #659 の懸念を満たしていることを、再パース付き regression test で確認した。
- `DuplicateBinding` は `_copy` ではなく `x1`, `x2`, ... のような lexable な名前を使うようにした。後続 free reference を捕まえる候補も避ける。

これで [#649](https://github.com/dowdiness/canopy/pull/649) と [#659](https://github.com/dowdiness/canopy/pull/659) は完了。[#650](https://github.com/dowdiness/canopy/pull/650) は move/delete の indentation として残る。


### Canopy / Grove Level 1 identity hint

構造編集後も NodeId を保ちやすくするため、Grove Level 1 の identity hint channel を入れた。狙いは `Var(x)` を `Lam(Param, Var(x))` で包むような kind mismatch 編集でも、内側 NodeId を失わず selection や fold など UI 状態を保つこと。

core 側では `IdentityTransform` と `reconcile_hinted` を追加した。`IdentityTransform` は「この編集は wrap です」「この子を残して unwrap します」のような編集意図を表す hint で、`reconcile_hinted` はその hint を使って古い木と新しい木の NodeId 対応を決める。editor 側では `SyncEditor` が span edit batch に対応する hint を保持し、Lambda 側では `TreeEditOp` を `IdentityTransform` へ落とす。

`WrapInLambda` などは NodeId 維持に使える hint を出す。一方 `ExtractToLet` や `ChangeOperator` のように安全な hint を出しづらいものは保守的に `Opaque` へ落とす。最後に Wrap / Unwrap の E2E coverage も追加した。

残りは write / read / clear の 2 端 contract を `HintChannel` 型で包むことと、hint あり / なし reconcile の重複整理。


### Canopy / analysis query layer設計

外部解析結果を Canopy へ入れる analysis query layer 設計を追加した。ast-grep や `moon ide` の結果をそのまま editor 内部へ混ぜるのではなく、Canopy 側で扱いやすい共通形式へ変換する層になる。解析結果はどの版の text から得たかを表す snapshot に紐づけ、種類の分かる typed fact として保存し、画面上では decorations として表示する。text CRDT が唯一の永続状態で、analysis result は古くなれば捨てるもの、という境界を崩さない。

Phase 1 は ast-grep の byte offset を UTF-16 range へ変換して range highlight するだけ。byte offset とエディタの文字位置はずれやすいので、まず位置変換を安全にするところから始める。rewrite、node-id mapping、protocol 変更はまだしない。


### Canopy / Ideal

Ideal の Action Overlay を、UI helper が Cell / Emit handle を直接持たず値だけを受け取る形へ寄せた。UI 描画補助関数が Rabbita の状態セルやイベント送信口を握らず、呼び出し側が計算済み値と callback を渡す形に近づけた。action overlay の flow と exec も分け直した。


### loom

MarkdownIR は M1 から recovery / raw node semantics へ進んだ。raw node は未対応・壊れた入力をそのまま保持する node、recovered node は parser が回復しながら作った node を指す。raw / recovered diagnostics、mdast export、direct syntax diagnostics、recovery adapter contract を追加し、HTML harness へ送る範囲も明確にした。editor 向けには壊れた入力も保持し、canonical rewrite や HTML 出力では変換先ごとにどこまで正規化するかを分ける方針になった。

現在の作業ツリーでは、次の [#328](https://github.com/dowdiness/loom/pull/328) 相当として MarkdownIR mdast export に unist `position` を付ける変更が進行中。Canopy superproject から見ると `loom` submodule は `0a827c3` から `3856167` へ進んだうえで未コミット差分が残っている。


### incr

incr_tea は inactive-root 測定から activation policy へ進んだ。inactive-root は DOM を残したまま更新を止めた非表示 UI subtree、activation policy はそれをいつ再び動かすかの方針になる。ratio table を再確認し、activation trigger probe を追加し、policy を docs で決めて実装まで入れた。Loom submodule 内の `incr` も `34ac477` から `f7681bc` へ進んでいる。


### js_engine

`needs_own_env` 系列の bytecode 最適化が続いた。関数呼び出し時に新しい Environment を本当に作る必要があるかを事前判定し、不要なら生成を skip する最適化になる。leaf bytecode function で `Environment::new` を skip し、same-realm callee では realm-proto wrapper を避け、active-override `Ref` も単一の `Ref[FunctionRealmProtos?]` へ畳んだ。最後に benchmark table へ `exec/for_of` row を追加した。


### 作業運用メモ

6/17 時点で、次の三つが月の芯になっていた。

1. Lambda edit の検証は、実際の編集後テキストを再パースして確認する。
2. analysis fact は snapshot に紐づき、いつでも捨てられるものとして扱う。
3. Grove hint channel は、次の構造編集言語へ広げる前に `HintChannel` 型で明示する。

## 2026/6/18

### Canopy / analysis query layer実装

前日に設計した analysis query layer の Phase 1 を実装して main へ入れた。`lib/analysis` 側には document identity・version・32bit hash・UTF-16 長を持つ `SourceSnapshot`、UTF-16 range と pattern id / captures を持つ `PatternMatchFact`、ast-grep の byte offset をエディタ側 UTF-16 offset へ変換する helper を置いた。

Canopy 側 `analysis` package では ast-grep 由来 match を `PatternMatchFact` へ変換し、protocol decoration や match-list entry へ落とす adapter を追加した。初期ルールとして MoonBit の `fn` 定義を拾う ast-grep rule も入った。途中で snapshot の同一性判定が version と hash だけだと同内容の別 document を誤って current 扱いしてしまう問題が見つかり、`doc_id` と `utf16_len` も見るよう修正した。

これで Phase 1 は「外部解析結果を snapshot に紐づく捨てられる fact として受け、range highlight / match list 用の値へ変換する」ところまで到達した。host-side FFI wiring、つまり JS から ast-grep 結果を渡して UI へ表示する部分は次の段階に残っている。


### loom / MarkdownIR

MarkdownIR は mdast export に unist `position` を付けるところまで進んだ。position は MarkdownIR 内部 source origin と `LineIndex` から export 境界で作るもので、MarkdownIR 自体を mdast 位置情報に引きずられないようにしている。non-BMP 文字、raw / recovered node、block separator、fenced code block、CRLF など位置がずれやすいケースも test で押さえた。

その後 [#333](https://github.com/dowdiness/loom/pull/333) の rewrite / canonical formatter 側へ進み、code fence と link の source-preserving rewrite smoke coverage を追加した。code fence では content だけを書き換える場合に fence そのものや周辺 source を壊さないこと、unclosed fence でも rewrite 境界を守ることを確認している。


### incr / Incremental TEA

incr_tea では 7GUIs stress test を追加した。Counter、Temperature Converter、Flight Booker、Timer、CRUD、Circle Drawer、Cells を別 package として置き、TEA 風 UI を incr の依存グラフでどこまで扱えるかを見る実験面が広がった。

同時に `on_change` や pointer offset まわりの小さな API も整えた。前日の inactive-root activation policy に続き、単体 demo ではなく複数 UI パターンを並べて「Rabbita とは別の incremental UI substrate として成立するか」を見る段階に入った。次は残る TEA follow-up、特に local pointer coordinate まわりの整理が候補になる。


### Rabbita / pointer events

Canopy が参照している Rabbita fork では pointer event binding の branch が進んだ。DOM / HTML / subscription 層へ `PointerEvent` 系 binding を足し、後続 UI で mouse 専用ではなく pointer 入力として扱えるようにする準備になる。実装後に constructor や cast style を既存 mouse event conversion に揃える修正も入った。

まだ Canopy 親リポジトリでは `rabbita` submodule pointer が未コミット差分として残っている。

### js_engine

js_engine では Set iteration の仕様バグを直した。`Set.prototype.forEach` 中に callback が最後の要素を delete して add し直すと、仕様上は再追加された値が未訪問の新 slot として再度訪問される。これに合わせ active な `forEach` 中は物理削除せず tombstone として残し、外側 iteration 終了後に compact するモデルにした。`clear()`、`values()` / `keys()` / `@@iterator`、`entries()` も tombstone を考慮し、無限ループを避ける one-shot guard 付き test も追加した。

並行して `Function.prototype.toString` が元 source を返せる PR が開かれている。parser / AST / runtime へ source text や span を通す大きめの変更で、review 後に original source から span を作る修正まで進んだ。こちらは main にはまだ入っていない。別 PR では docs の roadmap / design / decisions 配置を整理し、現在 architecture target や test262 snapshot を更新した。


### 作業運用メモ

今日の時点で Canopy 親リポジトリは `loom` と `rabbita` submodule pointer が dirty になっている。`loom` は `b26a304` まで進み、その中の `incr` submodule も `7a971ab` まで進んでいる。`rabbita` は pointer event branch の `54b3188` まで進んでいるが、親側で取り込むかどうかはまだ未整理。

## 2026/6/19

### Canopy / Markdown SDEG heading

Markdown 見出しを構造編集の対象にする SDEG（Structure-Directed Edit Grammar）の調査を進めた。

- Markdown heading の NodeId が編集をまたいでどの程度安定するかを探る heading identity probe を追加した（[#716](https://github.com/dowdiness/canopy/pull/716)）。
- SDEG NodeId side table の設計スケッチを test として置いた（[#717](https://github.com/dowdiness/canopy/pull/717)）。各 block の NodeId を横断的に引ける補助表の構想になる。
- heading edit path の E2E validation を加え（[#718](https://github.com/dowdiness/canopy/pull/718)）、SDEG Phase 0 での発見を Phase 1 計画へ持ち越す docs も更新した（[#719](https://github.com/dowdiness/canopy/pull/719)）。


### js_engine

- shared AsyncGeneratorPrototype chain を実装した（[#405](https://github.com/dowdiness/js_engine/pull/405)）。ES 仕様 §27.4 に従い、`async function*` で作られる各 generator が共通 prototype chain を共有するようにした。
- token に `end_offset` field を追加し、parser が source を再スキャンする必要をなくした（[#403](https://github.com/dowdiness/js_engine/pull/403)）。これまでは parser がトークン終端位置を知るために source 内を再度走査していたが、lexer 時点で UTF-16 コードユニット単位の end_offset を記録するようにした。
- runtime atomicsのtest coverage（[#402](https://github.com/dowdiness/js_engine/pull/402)）とtest262 toolingのリグレッションテスト（[#401](https://github.com/dowdiness/js_engine/pull/401)）を追加した。


## 2026/6/20

### Canopy / Markdown list SDEG

Markdownリストの構造編集向けの基盤を一気に進めた。

- Markdown SDEG heading side table を抽出し、見出し用補助表を独立させた（[#722](https://github.com/dowdiness/canopy/pull/722)）。
- block move provenance を追加した（[#723](https://github.com/dowdiness/canopy/pull/723)）。block 移動時に「どこから来たか」を追跡する仕組みで、後続リスト項目 move の前提になる。
- Markdown list move blocker を hardening した（[#726](https://github.com/dowdiness/canopy/pull/726)）。異なる階層や種類のリスト間移動など、安全にできないケースをきちんと弾くためのもの。
- same-list Markdown item moves を有効化し（[#731](https://github.com/dowdiness/canopy/pull/731)）、リスト項目 payload の消費も追加した（[#730](https://github.com/dowdiness/canopy/pull/730)）。


### js_engine / test262適合率向上

この日は js_engine にとって大きな fix デーだった。test262 失敗を系統的に潰し、JSON.parse が 100% pass に到達した。

- **lexer: astral_count 追跡**（[#408](https://github.com/dowdiness/js_engine/pull/408)）。MoonBit String は UTF-16 ベースだが lexer offset 計算がコードポイント単位だったため、サロゲートペアを含む source では全トークン位置がずれていた。`astral_count` を追加し、非 BMP 文字を読むたび +1 して UTF-16 コードユニット数へ補正するようにした。
- **JSON.parse reviver Proxy-aware**（[#419](https://github.com/dowdiness/js_engine/pull/419)）。`JSON.parse` reviver 処理を interpreter-aware 抽象操作経由に置き換え、Proxy `[[Get]]`/`[[Delete]]`/`[[DefineOwnProperty]]` trap が正しく発火するようにした。JSON/parse: 36 failures → 0（100%, 142/142）。
- **Promise spec fixes 5件**（[#413](https://github.com/dowdiness/js_engine/pull/413)）。`Symbol.toStringTag`のdescriptor、non-object thisでのTypeError、iter-poisonedケース、`Promise.any`のresolve/reject element guard、newTarget.prototypeの反映。
- **`__lookupGetter__`/`__lookupSetter__`と`replaceAll`修正**（[#412](https://github.com/dowdiness/js_engine/pull/412)）。Annex Bのgetter/setter lookupと、`String.prototype.replaceAll`のIsRegExp判定・`Symbol.replace` override対応。
- **surrogate-safe string slicing**（[#411](https://github.com/dowdiness/js_engine/pull/411)）。`classify_by_edition`ツールが絵文字などでpanicするのを修正。
- **regex `\u`/`\x` escape対応**（[#420](https://github.com/dowdiness/js_engine/pull/420)）。文字クラス内の`\uXXXX`/`\xHH`/`\f`/`\v`/`\0`/`\b`が全てリテラル文字扱いされていたバグを修正。Annex Bのidentity escapeやnon-Unicodeモードのサロゲートペア非結合もカバー。
- **匿名built-in関数のname/length/property order**（[#410](https://github.com/dowdiness/js_engine/pull/410)）と**singleton %GeneratorPrototype%**（[#407](https://github.com/dowdiness/js_engine/pull/407)）。
- test262: await-dictionary（`Promise.allKeyed`/`allSettledKeyed`）をskip（[#377](https://github.com/dowdiness/js_engine/pull/377)）。


## 2026/6/21

### Canopy / Markdown list + moon.mod移行開始

- Markdown listの構造編集を引き続き進め、list payloadの消費（[#730](https://github.com/dowdiness/canopy/pull/730)）とsame-list item move（[#731](https://github.com/dowdiness/canopy/pull/731)）をmainへ入れた。
- package map の文書化（[#736](https://github.com/dowdiness/canopy/pull/736)）を行い、全 Canopy パッケージの依存関係と名称を整理した。
- これを受け moon.mod.json→moon.mod 移行第一弾として Rabbita UI lib cluster（[#737](https://github.com/dowdiness/canopy/pull/737)）と lib/visualizer（[#738](https://github.com/dowdiness/canopy/pull/738)）を変換した。moon.mod（TOML 形式）へ移行することで MoonBit workspace membership を使った依存解決が可能になり、`NEW_MOON_MOD=0` デフォルト化へ近づく。


### js_engine / lexer・spec fixes・perf

- **regex/division disambiguation after `}`**（[#422](https://github.com/dowdiness/js_engine/pull/422)）。`}` 後に `/` が来たとき除算か正規表現リテラル開始かを判定する context tracking を全面的に書き直した。`}` が statement block を閉じるか object literal を閉じるかを追跡する brace-is-block stack、ternary colon と label/case colon を区別する ternary colon stack、nested class extends 内で状態が壊れない brace-is-block stack を導入した。
- **bind length ToIntegerOrInfinity + new Function の JS ToString coercion**（[#431](https://github.com/dowdiness/js_engine/pull/431)）。`Function.prototype.bind` length 引数処理を仕様通り `ToIntegerOrInfinity` へ修正し、`new Function` 引数を MoonBit `.to_string()` ではなく JS `ToString` 抽象操作で評価するようにした。
- **Annex B web-compat call-assign**（[#428](https://github.com/dowdiness/js_engine/pull/428)）。non-strict mode で `CallExpression` が代入左辺に来たとき parse 時エラーではなく runtime `ReferenceError` にする Annex B 互換動作を実装した。
- **analysis-family growth convention docs**（[#332](https://github.com/dowdiness/js_engine/pull/332)）。静的解析ファイル構成ルールを文書化した。
- **timer queue を priority_queue へ移行**（[#433](https://github.com/dowdiness/js_engine/pull/433)）。これまでの `Array[TimerTask] + sort_by + remove(0)`（O(n² log n)）を `@priority_queue.PriorityQueue[TimerTask]`（O(n log n)）へ置き換え、200 タイマー drain が 4.19ms→1.95ms（2.15× 高速化）。キャンセルは遅延削除方式にした。


## 6月第4週: SDEG 成熟と test262（6/22〜6/26）

moon.mod移行を完了し、SDEGのlifecycleとbenchmark CIを整えた。js_engineのtest262適合率もこの週から大きく伸び始めた。

## 2026/6/22

### Canopy / moon.mod移行完了

全 Canopy-owned マニフェストの moon.mod.json→moon.mod 移行を完了した（[#740](https://github.com/dowdiness/canopy/pull/740)）。今週最大の変更で、以下を含む。

- **7 つの Canopy-owned マニフェストを変換**: ルート、lib/semantic、examples/resizable、examples/codemirror_demo、examples/block-editor、examples/canvas、examples/ideal。
- **13 submodule を workspace member に追加**: loom/examples/{markdown,json,lambda,graph-dsl}、loom/{loom,seam,pretty,text-change,moji,egglog,egraph}、event-graph-walker、rle、order-tree。
- **全 MOON_WORK=off を削除**: moon.mod `import { }` 構文は workspace membership なしでは依存解決できないため、benchmark、E2E テスト、Cloudflare deploy、JS build script から MOON_WORK=off をすべて取り除いた。
- **CI 整備**: vendored-submodule エラーを抑止する共通 filter を導入（`scripts/vendored-check-common.sh`）。workspace 全体 check で submodule が起こす 21 件 pre-existing error を抑制しつつ、submodule 自身 CI では自前失敗を隠さない `--keep` オプション設計にした。
- **Cloudflare deploy 修正**: workspace build 成果物が `_build/js/release/build/<module>/<pkg>/` へ出力されるのに対し vite-plugin-moonbit 期待パスとのずれを symlink で吸収した（[#335](https://github.com/dowdiness/canopy/pull/335)）。

並行して、loom submoduleのquickcheck 0.14 Arrow API compat対応や、AGENTS.mdのsubmodule guidance更新も行った。


## 2026/6/23

### Canopy / submodule bumpとHTML block修正

月曜大規模移行の後始末と後続 submodule 更新を進めた。

- **submodule bump**: egraph moon.mod 移行（loom#455）、event-graph-walker trait split 修正（#58）、alga v0.4.0 取り込み、graphviz DirectedGraph 互換修正。各 submodule は moon.mod.json から moon.mod への移行を進めており、Canopy 側で追従した。
- **HTML blocks §4.6**: loom submodule を bump し、Markdown HTML block を projection block children に含める対応を入れた。block mode で HTML block が `text:null`/`editable:false` として扱われ表示から消えるバグがあった。`HtmlBlock` に適切 token span を populate することで修正した。
- 残っているsubmodule（svg-dsl、rle、order-tree、graphviz）のmoon.mod移行完了に伴うbumpが[#742](https://github.com/dowdiness/canopy/pull/742)として進行中。


## 2026/6/24

### Canopy / Markdown SDEG lifecycle

Markdown SDEG heading side table を、単なる「見出し ID 対応表」から parse validity と lifecycle を持つ構造へ寄せた。

- heading side table lifecycle を parse validity で gate した（[#763](https://github.com/dowdiness/canopy/pull/763)）。壊れた parse 結果を見て安定 ID 表を安易に更新しないための境界になる。
- SDEG retention threshold を設定可能にし、消えた heading をすぐ捨てず一定期間後 `Retired` entry へ遷移させる形にした（[#755](https://github.com/dowdiness/canopy/pull/755), 元 PR [#746](https://github.com/dowdiness/canopy/pull/746)）。復帰 heading stable id 落下や retired row stable id 重複も追加修正した。
- loomgen `RawKind` / content-hash identity decision を docs へ残した（[#750](https://github.com/dowdiness/canopy/pull/750)）。Markdown raw / recovered 領域をどの単位で同一性判定するかの判断記録になる。
- event-graph-walker、loom、alga、lang/markdown 周辺の warning を整理した（[#754](https://github.com/dowdiness/canopy/pull/754)）。

この日 SDEG 作業は、見出しやリスト項目 move そのものより「構造編集対象を追跡する表が、壊れた入力や一時的消失にどう耐えるか」を詰める作業だった。


### Canopy / benchmark CIとprojection map

benchmark regression workflow を PR gate へ近づけた。submodule gitlink や shared vite plugin 変更も benchmark gate 対象に含め、workflow 自体も並列化して高速化した（[#762](https://github.com/dowdiness/canopy/pull/762)）。ただし vendored submodule 由来既存問題をどこまで gate に含めるかはまだ難しく、後日「skipped check を green 扱いしない」運用につながった。

投影構造側では RoseNode map と constructor API を追加した（[#761](https://github.com/dowdiness/canopy/pull/761)）。後続 ProjNode map へ向け、tree projection を外から扱う足場が増えた。


### loom / MarkdownIR

Canopy が参照する loom では Markdown IR 実装ファイル分割（[#472](https://github.com/dowdiness/loom/pull/472)）と raw kind / Tabs handling 修正（[#473](https://github.com/dowdiness/loom/pull/473)）が進んだ。Markdown SDEG 側で raw / recovered node を安定して扱う下支えになる。


### js_engine

js_engineではtest262適合率向上の流れが続いた。

- environment markerを専用mapへ分離した（[#438](https://github.com/dowdiness/js_engine/pull/438)）。
- call評価側ではgrouping unwrap dispatchを整理し、logical assignment operatorのNamedEvaluationを修正した。
- sloppy functionのnon-simple paramsにおける`arguments.callee` accessorを仕様へ寄せた（[#440](https://github.com/dowdiness/js_engine/pull/440)）。
- `SetIteratorPrototype.next`のbrand checkとdone flagを修正した（[#441](https://github.com/dowdiness/js_engine/pull/441)）。
- `[[OwnPropertyKeys]]`列挙をcanonical opへ統一した（[#442](https://github.com/dowdiness/js_engine/pull/442)）。


## 2026/6/25

### Canopy / SDEG snapshot validity

Markdown SDEG heading snapshot validity を明示した（[#766](https://github.com/dowdiness/canopy/pull/766)）。前日 parse validity gate をさらに進め、side table がどの snapshot に対して妥当かを曖昧にしない形にした。

さらにレビュー対応として「Markdown SDEG snapshot validity を wire する」PR 作業も進んだ（[#767](https://github.com/dowdiness/canopy/pull/767) 相当、commit `f4effe5`）。agent 履歴上は `lang/markdown/proj` と `lang/markdown/companion` targeted test、`moon fmt`、`moon info`、workspace `moon check` まで通っている。一方 `Editor Response Benchmark` が `skipping` として残り、repo 運用上「skipped は green ではない」ため merge は止めた。CI 上 skip を明示的に扱う必要がはっきりした。


### Canopy / ProjNode mapとCI cleanup

RoseNode map に続き ProjNode map を追加した（[#765](https://github.com/dowdiness/canopy/pull/765)）。projection tree を言語ごと特殊処理だけで扱うのではなく、共通 map 操作へ寄せる流れが見えてきた。

CI 側では Playwright image 更新や dependabot による Vite / Vitest / React DOM / actions checkout 更新が入った。手元 Canopy worktree では benchmark workflow コメント整理、vendored check filter から `alga` を外す調整、`loom` submodule pointer を `6d7778b` へ進める差分が残っている。`loom` 側内容は Markdown raw kind と Tabs handling 修正までを含む。


### js_engine

js_engineではMap / Set / Promise / Proxy周辺の仕様適合を進めた。

- `Reflect.ownKeys`をMap / Set / Promiseにも広げ、Proxyの`[[OwnPropertyKeys]]`でsymbol keyを正しく分類するようにした（[#445](https://github.com/dowdiness/js_engine/pull/445)）。
- test262のper-mode regression diffを見るための`test262_failing_diff.js`を追加した（[#446](https://github.com/dowdiness/js_engine/pull/446)）。
- branch上では、Map / Setのexpando assignment、Promise instance constructor keys、computed Map / Set writes、array own descriptorでMap / Set writesを止める修正が続いた。agent履歴ではPR [#449](https://github.com/dowdiness/js_engine/pull/449)として、Map / Set subclass chainにarray prototypeが挟まるリグレッションを追加し、`moon check`、targeted regression、`moon test`、`moon info`、`moon fmt`、`moon check --deny-warn`、release testまで通している。


## 2026/6/26

### Canopy / JSON role spans + CI cleanup

JSON role span の editor decoration 連携を一気に仕上げた。Loom がパース時に出力する role span（値の種類ごとに区別した構文情報）を FFI 境界を通して CodeMirror Editor へ届け、decoration として表示するまでの流れが通った（#781, #782, #783）。

- #781: Loom JSON role-span export を FFI/CodeMirror path へ統合し、MoonBit→JS データ経路を作った。
- #782: role span を `Derived::map` reactive cell で包み、source text 変更に応じて自動更新されるようにした。
- #783: role span を editor decoration として適用し、parser-driven syntax coloring として表示した。

vendored error-suppression list 整理が完了した。6/22 moon.mod 移行後に残っていた alga/rle（#773）、order-tree（#778）、graphviz/svg-dsl（#779）、event-graph-walker（#780）を順に suppression から外し、各 submodule moon.mod 移行完了を反映した。

benchmark regression CI も高速化した（#777）。Canopy subpackage と loom example benchmark を -p flags に追加し、cache key を v3→v4 へ更新、moon-update と moon bench 実行順序も工夫した。


### Loom / arrow lambda + Pratt reuse

Loom では arrow lambda 構文 P2 fix が中心だった。ブロック body、soft newline 周り、typed arrow param（TypeAnnot/TypInt/TypeUnit/TypeArrow）、右再帰 TypeArrow と括弧付き型注釈など regression を修正した。incr 依存を 0.9.0→0.11.0 へ bump し、JSON role span projection slice trimming も入った。

retroactive Pratt reuse groundwork（#475）が入った。Pratt parser 状態を backtracking 間で再利用する下準備になる。deep nested-lambda benchmark workload B 比較性も回復した（#476）。


### js_engine / NFE binding + async fixes + test262整備

js_engine では test262 適合率向上 Cluster 11（NFE self-name binding）が完了した（#463）。`FunctionNameBinding` を仕様通り実装し、strict mode threading を bytecode StoreName env assign まで通し、generator/async method では抑制する has_name_binding 制御も入れた。

async関数のエッジケースも修正した（#468）：parameter TDZ、non-strictでのthis、arrow arguments、mapped arguments。Array関係では、`ArraySpeciesCreate`のnon-object constructorでのTypeError（#467）、`Array.prototype[Symbol.iterator]`削除への対応（#462）、array-like lengthのInt64移行（#461）、`reverse/fill/copyWithin/sort`のprototype委譲（#471）が進んだ。

CI面ではcopilot toolchain cacheとTest262 feature-gap比較ツールを追加し（#460）、baseline ratchetとcalibration automationも整えた。

## 6月第5週: loomgen・block-editor・JSON editor（6/27〜6/30）

loomgenの追加、incrのMemo→Derived完全移行、block-editorとJSON tree editorの立ち上げが、月末の4日間に集中した。

## 2026/6/27

### Canopy / block-editor drag-dropとcore primitives

block-editor drag-drop を実際に使える機能として仕上げる作業が本格化した。前提として `ProjNode::walk_preorder` というアロケーションなし前順走査 visitor を core に追加し（[#793](https://github.com/dowdiness/canopy/pull/793)）、`SourceMap` build/remove/patch をこれ経由にまとめた。続けて該当 node までのパスだけを複製して書き換える `modify_node_at` を追加した（[#794](https://github.com/dowdiness/canopy/pull/794)）。catamorphism ではなく最初に見つかった箇所で止まって祖先パスだけ再構築する、Clojure `update-in` に近い操作で、`replace_loaded_subtree_node` や `update_node_collapsed` にあった重複コードを大きく削った。

drag-drop 本体では並行ドラッグ収束性を quickcheck で検証する test を追加した（[#798](https://github.com/dowdiness/canopy/pull/798)）。複数箇所から同時 drop しても text・AST・ProjNode 整合性が壊れないこと、concurrent drop 後 undo が正しく収束すること、3 者同時編集でも構造エラーが出ないことをランダム化 test と 7 敵対的ケースで確認した。`Inside` drop は入れ子ではなく `SyncEditor::move_node` による exchange（交換）として扱う、という意味的決定をした。UI 側では `get_render_state()` に depth や child_count、生死フラグ、実 parent_id を含む block metadata を持たせ、TypeScript 側 drop ターゲット判定（自分自身や descendant への drop を弾く）を実装した（[#800](https://github.com/dowdiness/canopy/pull/800)）。

loom submoduleを`454b460`へ更新し、loom側の`build_tree_buffered_with`統合を取り込んだ（[#796](https://github.com/dowdiness/canopy/pull/796)）。§7 aggregator-trimの監査完了もdocsに記録した（[#276](https://github.com/dowdiness/canopy/pull/276)）。


### incr / Memo→Derived facade移行が本格化

incr で `Memo` / `HybridMemo` / `MemoMap` という旧世代型を `Derived` / `ReachableDerived` / `DerivedMap` という新 facade へ完全移す作業が始まった。まず `@incr` から互換 re-export を削除し（[#313](https://github.com/dowdiness/incr/pull/313), [#314](https://github.com/dowdiness/incr/pull/314)）、`Scope::adopt[T: Trackable]` で scope ライフサイクル登録を型横断で統一した（[#315](https://github.com/dowdiness/incr/pull/315)）。

このタイミングで `Derived::map` ファミリー命名を入れ替える意図的決定をした（[#316](https://github.com/dowdiness/incr/pull/316)）。backdate しない危険版を `map_no_backdate`、Eq で backdate する安全版を短い `map` にした。危ないほうに長い名前を割り当て、デフォルトで安全側に倒す設計だと分かる。続けて `Input::derived` / `Input::derived_no_backdate` / `Scope::derived_no_backdate` / `Derived::derived_no_backdate` という一連コンストラクタを揃え（[#317](https://github.com/dowdiness/incr/pull/317), [#318](https://github.com/dowdiness/incr/pull/318), [#320](https://github.com/dowdiness/incr/pull/320)）、pipeline どこからでも一貫した書き方で derived を作れるようにした。午後には wbtest ファイル Memo→Derived 移行が始まった（[#322](https://github.com/dowdiness/incr/pull/322), [#323](https://github.com/dowdiness/incr/pull/323)）。

### js_engine / constructor・TDZ・iteratorの仕様適合

Function / Generator / AsyncFunctionの動的コンストラクタにあった4つの仕様漏れを直した（[#476](https://github.com/dowdiness/js_engine/pull/476)）。coercion順序、generatorパラメータ内のyield検出、`.prototype.constructor`の逆参照、AsyncFunctionの`[[Prototype]]`が対象になる。destructuring中にgeneratorがabrupt resumeした際、前のiteratorを閉じ忘れる問題も直し（[#480](https://github.com/dowdiness/js_engine/pull/480)）、SuperCallを正規のbinder経由にしてrest paramやdestructuringを含むケースを救った（[#479](https://github.com/dowdiness/js_engine/pull/479)）。class constructorのパラメータにも§10.2.11のTDZ事前チェックを追加し、`constructor(x = y, y = 1)`のような前方参照が`undefined`ではなく`ReferenceError`になるよう直した（[#481](https://github.com/dowdiness/js_engine/pull/481)）。test262 baselineは非strict 27650→27686、strict 25800→25923まで伸びた（[#477](https://github.com/dowdiness/js_engine/pull/477), [#482](https://github.com/dowdiness/js_engine/pull/482)）。

### loom / loomgenの誕生

この日 loom にコード生成器「loomgen」が生まれた。`#loom.token` / `#loom.punct` / `#loom.keyword` アノテーション付き MoonBit Token/Term enum を読み、`syntax_kind.g.mbt` や `token_impls.g.mbt` を生成する仕組みで、Phase 1 トークン enum 生成（[#492](https://github.com/dowdiness/loom/pull/492)）、Phase 2 term enum（CST ノード種別）統合生成（[#493](https://github.com/dowdiness/loom/pull/493)）まで進んだ。Lambda 26 Token variant すべてを移行し、手書き Show/IsTrivia/IsEof/ToRawKind 実装を削除できた。生成結果と source ずれを見る CI gate（`check-loomgen`）も入れた。`LanguageSpec` ファクトリ関数生成（[#496](https://github.com/dowdiness/loom/pull/496)）、エラーパス終了コード修正や `main` からの名前付きフェーズ抽出（[#505](https://github.com/dowdiness/loom/pull/505), [#510](https://github.com/dowdiness/loom/pull/510), [#511](https://github.com/dowdiness/loom/pull/511)）も進んだ。

loomgen とは別に 3 つの `build_tree` 亜種（トークン生成・再利用処理・node 構築 callback だけが違う）を 5 callback を取る `build_tree_buffered_with` ひとつへ DRY した（[#494](https://github.com/dowdiness/loom/pull/494)）。293 行が 85 行程度まで減った。


## 2026/6/28

### Canopy / typed error modelとJSON tree editor

Ideal action overlay で `error: String` だった状態を `error: OverlayError?` という閉じた enum（`EmptyName | BuildActionFailed(String) | ApplyActionFailed(String)`）に置き換えた（[#799](https://github.com/dowdiness/canopy/pull/799)）。空文字列を「エラーなし」代用にしていた曖昧さをなくし、`None` だけがエラーなしを意味するようにした。

block-editor drag-drop では抜けていた `Inside` drop ゾーンを追加した（[#802](https://github.com/dowdiness/canopy/pull/802)）。上 40% を Before、中央 20% を Inside、下 40% を After とする三分割ジオメトリと `--depth` CSS 変数によるネスト深さ表示を入れた。MoonBit 4 引数シグネチャに気づかず 5 番目引数が黙って無視されていたバグと、`currentDropTarget` 代入漏れでインジケータが消えないバグも直した。cross-parent Before/After 移動が正しく親を付け替えられるようにした（[#806](https://github.com/dowdiness/canopy/pull/806)、[#801](https://github.com/dowdiness/canopy/pull/801) を閉じる。「Policy A: reparent を許可する」方針）。event-graph-walker 新 `Document::parent`（loom [#518](https://github.com/dowdiness/loom/pull/518) 経由 bump）を使い、ハードコードされていた `root_block_id` を `self.tree.parent(ref_id)` に置き換えた。

同日 JSON tree editor が一気に実用段階へ進んだ。node 種別ごと描画（展開/折りたたみ状態が patch 後も保持、[#810](https://github.com/dowdiness/canopy/pull/810)）、キー・値インライン編集と add/delete/wrap/unwrap ボタン（[#811](https://github.com/dowdiness/canopy/pull/811)）、JSON.parse/stringify 往復 Format ボタン（[#812](https://github.com/dowdiness/canopy/pull/812)）、直近 100 件 patch ログ/履歴パネル（[#813](https://github.com/dowdiness/canopy/pull/813)）の 4 本が続けざまに入り、test 数も 16→24 まで増えた。


### incr / Memo→Derived移行が完了

前日始まった移行がこの日で本体まで完了した。wbtest ファイル残り移行（[#326](https://github.com/dowdiness/incr/pull/326), [#327](https://github.com/dowdiness/incr/pull/327), [#329](https://github.com/dowdiness/incr/pull/329), [#331](https://github.com/dowdiness/incr/pull/331)）に続き docs と trait コメント更新（[#332](https://github.com/dowdiness/incr/pull/332)）を経て、`Memo` / `MemoMap` / `HybridMemo` 構造体そのものを削除する Phase 2 一括切り替えを行った（[#333](https://github.com/dowdiness/incr/pull/333)）。`Derived` / `ReachableDerived` / `DerivedMap` が、それまでラップしていた旧エンジン型フィールドを直接持つようになった。

同日うちにレガシー `Signal[T]` 型も削除した（[#334](https://github.com/dowdiness/incr/pull/334)、breaking change）。`Input[T]` が唯一の入力 cell 型になり、`force_set` / `is_fresh` / `derived` / `derived_no_backdate` を吸収した。約 60 test ファイルが移行対象になり 1123 件 test が通った。2 日足らずで「移行開始→黒箱 test→型削除→エイリアス整理」まで駆け抜けた、規律あるフェーズ分割移行だった。

### loom / loomgenのアノテーション語彙が拡張

loomgen アノテーション語彙が急速に増えた。複数 trivia variant を許すよう単一 trivia 制約を緩め（[#513](https://github.com/dowdiness/loom/pull/513)）、その直後に見つかった実バグ、つまり複数 trivia 対応後は副次 trivia（コメントトークンなど）が通常トークンとして漏れ incremental 再利用判定を誤らせる問題を修正した（[#517](https://github.com/dowdiness/loom/pull/517)）。`#loom.view` で型付き projection アクセサを生成し（[#515](https://github.com/dowdiness/loom/pull/515)）、`#loom.void` / `#loom.rawtext` で HTML 要素プロパティ表を生成し（[#519](https://github.com/dowdiness/loom/pull/519)）、`#loom.lexmode` で LexMode enum と dispatch 関数を生成し（[#525](https://github.com/dowdiness/loom/pull/525)）、`#loom.pattern` 正規表現アノテーションから step lexer を生成できるところまで進んだ（[#528](https://github.com/dowdiness/loom/pull/528)）。パターンは MoonBit コンパイル時正規表現リテラルへコンパイルされ、`a*` のような nullable パターンは生成時に拒否される。Grammar IR から MoonBit source を直接出す source emitter も入った（[#533](https://github.com/dowdiness/loom/pull/533)）。

event-graph-walkerを`Document::parent`が使えるバージョンへbumpし（[#518](https://github.com/dowdiness/loom/pull/518)、canopy側の同日#806が消費した）、pre-push fmt checkのスコープをvendored submoduleの巻き添えを避けるよう絞った（[#527](https://github.com/dowdiness/loom/pull/527)）。


## 2026/6/29

### Canopy

JSON editor が「統一」段階に入った。読み取り専用パネルと同じ tree view を、そのまま編集可能な形で contenteditable テキストエディタの代わりに使うようにした（[#814](https://github.com/dowdiness/canopy/pull/814)）。値クリックで編集、キークリックでリネーム、行ごと add/delete/wrap/unwrap ボタン、折りたたみ状態維持に加え Raw/Structured 表示切り替えも入れた。ツールバーは行ごと操作に置き換えて廃止した。

編集系細かな整理も進んだ。ほぼ同じ処理だった `free_names_would_rebind_at_node` と `_at_module_end` を resolver closure を引数に取る 1 関数へ統合し（[#816](https://github.com/dowdiness/canopy/pull/816)）、block-local binding 上下移動でインデントが二重になったり失われたりする問題を、テキスト継ぎ接ぎからインデント再計算へ変えて直した（[#819](https://github.com/dowdiness/canopy/pull/819)、[#650](https://github.com/dowdiness/canopy/pull/650) を閉じる）。§7 TODO のうち 3 件（Warren symlink、rle、seam build_tree DRY）が完了扱いになった（[#797](https://github.com/dowdiness/canopy/pull/797)）。

### incr

Signal 時代内部名を Input 時代名前へ機械的に置き換えた（[#336](https://github.com/dowdiness/incr/pull/336)、`PullSignalData`→`PullInputData` などファイル名ごと）。前日 `Signal` 削除（#334）の内部側仕上げになる。docs 例も `Derived::map` / `map2` を使う形へ揃えた（[#337](https://github.com/dowdiness/incr/pull/337)）。

`Input::derived2` / `derived3` を追加し、`#alias` による lowercase コンストラクタ（`input()`, `derived()` など）とフリー関数 `Runtime::input` を用意した（[#338](https://github.com/dowdiness/incr/pull/338)）。Memo→Derived 一連移行は、公開 API 命名統一・Eq デフォルト安全化・PascalCase/lowercase 両対応エントリポイントまで含め、実質的に完了した。

### loom

loomgen grammar IR 側が成熟した。`#loom.rule("EBNF")` アノテーションから `@grammar.GrammarIr` を作る emitter を追加し（[#534](https://github.com/dowdiness/loom/pull/534)）、`Seq` / `Choice` / `Ref` / `Star` / `Plus` / `Opt` / `Expect` という素朴 EBNF 部分集合を、未知シンボルや左再帰を検出して fail-closed に倒す形で実装した。`AstView` trait を `@core` から `@seam` へ移し、loomgen 生成 `*Proj` 構造体が直接実装できるようにして（[#535](https://github.com/dowdiness/loom/pull/535)）、当初二層構成（`*Proj`/`*View`）を予定していたのを一層に単純化できた。

loomgen とは別に Lambda example では `free_vars` を既存 tagless-final fold（`TermSym`）の新解釈として実装し（[#536](https://github.com/dowdiness/loom/pull/536)）、新走査を書かずに済ませた。if 式・lambda 式 CST ラッパー node と `*Proj` 構造体を追加し（[#542](https://github.com/dowdiness/loom/pull/542)）、両 printer（`to_source`、`to_layout`）を `interpret` カタモルフィズム経由へ統一した（[#544](https://github.com/dowdiness/loom/pull/544)）。`resolve_walk` wildcard 節にあった網羅性穴も塞いだ（[#545](https://github.com/dowdiness/loom/pull/545)）。


## 2026/6/30

### Canopy / moon.mod移行の仕上げとE2E分離

技術的負債と CI に関する節目 PR をまとめて入れた（[#820](https://github.com/dowdiness/canopy/pull/820)）。`dump-deps.sh` / `test-focus.sh` / `package-release.sh` が旧 `moon.mod.json` と新 TOML 形式 `moon.mod` 両方を読める moon.mod 移行下準備。4 つの E2E CI ジョブ（web-e2e、ideal-web-e2e、demo-react-e2e、canvas-e2e）を、それぞれ個別 MoonBit セットアップ・ビルドする形から単一 `build-js` ジョブ成果物をダウンロードする形へ変え、E2E を MoonBit registry flakiness から切り離した。

### incr / MoonDsp

incr_tea_7guis Playwright DOM test を CI に載せ、cross-root locality 検証を加えた（[#339](https://github.com/dowdiness/incr/pull/339)）。`SubSpec::AnimationFrame(Msg)` を追加し、Timer demo にライブなフレームカウンタとして組み込んだ（[#340](https://github.com/dowdiness/incr/pull/340)、#290 完了）。

### loom

loomgen リグレッション test ハーネス（7 whitebox test）と `--seed` 自動検出を追加し、MoonBit ツールチェーン ICE を避けるため check/test を `--target native` に固定した（[#546](https://github.com/dowdiness/loom/pull/546)）。`roles_match` を 10 節から 5 節へ絞り、CWD 依存だった fixture パスも直した（[#548](https://github.com/dowdiness/loom/pull/548)）。

## PR索引

週ごとに折りたたんだ PR / Issue 一覧。GitHub 上の詳細への索引。

<details>
<summary>6月第1週（6/1〜6/7）</summary>

### 2026/6/1

**loom / js_engine / 運用**

canopy [#445](https://github.com/dowdiness/canopy/pull/445), [#437](https://github.com/dowdiness/canopy/pull/437), [#447](https://github.com/dowdiness/canopy/pull/447) / loom [#206](https://github.com/dowdiness/loom/pull/206) / js_engine [#186](https://github.com/dowdiness/js_engine/pull/186)

### 2026/6/2

**CI / 運用**

canopy [#451](https://github.com/dowdiness/canopy/pull/451), [#452](https://github.com/dowdiness/canopy/pull/452), [#453](https://github.com/dowdiness/canopy/pull/453), [#462](https://github.com/dowdiness/canopy/pull/462), [#465](https://github.com/dowdiness/canopy/pull/465), [#469](https://github.com/dowdiness/canopy/pull/469)

### 2026/6/3

**MoonDsp / incr**

canopy [#445](https://github.com/dowdiness/canopy/pull/445), [#461](https://github.com/dowdiness/canopy/pull/461), [#479](https://github.com/dowdiness/canopy/pull/479)

### 2026/6/4

**MoonDsp / js_engine**

canopy [#489](https://github.com/dowdiness/canopy/pull/489), [#508](https://github.com/dowdiness/canopy/pull/508), [#511](https://github.com/dowdiness/canopy/pull/511)

### 2026/6/5

**incr / MoonDsp / js_engine**

canopy [#517](https://github.com/dowdiness/canopy/pull/517), [#523](https://github.com/dowdiness/canopy/pull/523), [#524](https://github.com/dowdiness/canopy/pull/524), [#525](https://github.com/dowdiness/canopy/pull/525), [#526](https://github.com/dowdiness/canopy/pull/526), [#528](https://github.com/dowdiness/canopy/pull/528)

### 2026/6/6

**周辺リポジトリ**

canopy [#529](https://github.com/dowdiness/canopy/pull/529), [#532](https://github.com/dowdiness/canopy/pull/532), [#534](https://github.com/dowdiness/canopy/pull/534), [#539](https://github.com/dowdiness/canopy/pull/539), [#541](https://github.com/dowdiness/canopy/pull/541), [#544](https://github.com/dowdiness/canopy/pull/544)

### 2026/6/7

**Canvas / loom**

canopy [#553](https://github.com/dowdiness/canopy/pull/553), [#554](https://github.com/dowdiness/canopy/pull/554), [#558](https://github.com/dowdiness/canopy/pull/558), [#560](https://github.com/dowdiness/canopy/pull/560), [#562](https://github.com/dowdiness/canopy/pull/562)

</details>

<details>
<summary>6月第2週（6/8〜6/14）</summary>

### 2026/6/8

**loom / incr / js_engine**

canopy [#569](https://github.com/dowdiness/canopy/pull/569), [#570](https://github.com/dowdiness/canopy/pull/570), [#571](https://github.com/dowdiness/canopy/pull/571), [#576](https://github.com/dowdiness/canopy/pull/576)

### 2026/6/9

**loom / MoonDsp**

canopy [#571](https://github.com/dowdiness/canopy/pull/571), [#575](https://github.com/dowdiness/canopy/pull/575), [#577](https://github.com/dowdiness/canopy/pull/577)

### 2026/6/10

**MoonDsp / loom / Canopy / js_engine**

incr [#209](https://github.com/dowdiness/incr/pull/209), [#211](https://github.com/dowdiness/incr/pull/211), [#238](https://github.com/dowdiness/incr/pull/238), [#243](https://github.com/dowdiness/incr/pull/243), [#244](https://github.com/dowdiness/incr/pull/244) / canopy [#611](https://github.com/dowdiness/canopy/pull/611), [#615](https://github.com/dowdiness/canopy/pull/615)

### 2026/6/11

**js_engine / 運用**

loom [#279](https://github.com/dowdiness/loom/pull/279), [#196](https://github.com/dowdiness/loom/pull/196) / canopy [#587](https://github.com/dowdiness/canopy/pull/587), [#588](https://github.com/dowdiness/canopy/pull/588) / js_engine [#344](https://github.com/dowdiness/js_engine/pull/344), [#349](https://github.com/dowdiness/js_engine/pull/349)

### 2026/6/12

**周辺リポジトリ**

canopy [#587](https://github.com/dowdiness/canopy/pull/587), [#588](https://github.com/dowdiness/canopy/pull/588), [#589](https://github.com/dowdiness/canopy/pull/589), [#590](https://github.com/dowdiness/canopy/pull/590), [#597](https://github.com/dowdiness/canopy/pull/597), [#599](https://github.com/dowdiness/canopy/pull/599), [#610](https://github.com/dowdiness/canopy/pull/610)

### 2026/6/13

**loom / js_engine**

canopy [#602](https://github.com/dowdiness/canopy/pull/602), [#604](https://github.com/dowdiness/canopy/pull/604), [#609](https://github.com/dowdiness/canopy/pull/609), [#610](https://github.com/dowdiness/canopy/pull/610), [#611](https://github.com/dowdiness/canopy/pull/611), [#615](https://github.com/dowdiness/canopy/pull/615), [#619](https://github.com/dowdiness/canopy/pull/619)

### 2026/6/14

**Canvas / loom / incr / MoonDsp / js_engine**

canopy [#637](https://github.com/dowdiness/canopy/pull/637), [#638](https://github.com/dowdiness/canopy/pull/638), [#640](https://github.com/dowdiness/canopy/pull/640), [#641](https://github.com/dowdiness/canopy/pull/641), [#644](https://github.com/dowdiness/canopy/pull/644), [#647](https://github.com/dowdiness/canopy/pull/647), [#648](https://github.com/dowdiness/canopy/pull/648), [#655](https://github.com/dowdiness/canopy/pull/655), [#639](https://github.com/dowdiness/canopy/pull/639), [#643](https://github.com/dowdiness/canopy/pull/643)

</details>

<details>
<summary>6月第3週（6/15〜6/21）</summary>

### 2026/6/15

**周辺リポジトリ**

canopy [#660](https://github.com/dowdiness/canopy/pull/660), [#663](https://github.com/dowdiness/canopy/pull/663), [#671](https://github.com/dowdiness/canopy/pull/671), [#673](https://github.com/dowdiness/canopy/pull/673), [#664](https://github.com/dowdiness/canopy/pull/664), [#677](https://github.com/dowdiness/canopy/pull/677), [#666](https://github.com/dowdiness/canopy/pull/666), [#670](https://github.com/dowdiness/canopy/pull/670), [#669](https://github.com/dowdiness/canopy/pull/669), [#668](https://github.com/dowdiness/canopy/pull/668) / loom [#342](https://github.com/dowdiness/loom/pull/342), [#346](https://github.com/dowdiness/loom/pull/346) / incr [#273](https://github.com/dowdiness/incr/pull/273) / js_engine [#356](https://github.com/dowdiness/js_engine/pull/356)

### 2026/6/16

**周辺リポジトリ**

canopy [#674](https://github.com/dowdiness/canopy/pull/674), [#682](https://github.com/dowdiness/canopy/pull/682), [#683](https://github.com/dowdiness/canopy/pull/683), [#684](https://github.com/dowdiness/canopy/pull/684), [#685](https://github.com/dowdiness/canopy/pull/685), issue [#659](https://github.com/dowdiness/canopy/pull/659) / loom [#346](https://github.com/dowdiness/loom/pull/346) / incr [#277](https://github.com/dowdiness/incr/pull/277), [#279](https://github.com/dowdiness/incr/pull/279) / js_engine [#365](https://github.com/dowdiness/js_engine/pull/365), [#366](https://github.com/dowdiness/js_engine/pull/366)

### 2026/6/17

**Canopy / Lambda編集の健全性**

canopy [#688](https://github.com/dowdiness/canopy/pull/688), [#689](https://github.com/dowdiness/canopy/pull/689), [#691](https://github.com/dowdiness/canopy/pull/691), [#696](https://github.com/dowdiness/canopy/pull/696)

**Canopy / Grove Level 1 identity hint**

canopy [#690](https://github.com/dowdiness/canopy/pull/690), [#697](https://github.com/dowdiness/canopy/pull/697), [#698](https://github.com/dowdiness/canopy/pull/698)

**Canopy / analysis query layer設計**

canopy [#687](https://github.com/dowdiness/canopy/pull/687)

**Canopy / Ideal**

canopy [#686](https://github.com/dowdiness/canopy/pull/686)

**loom**

loom [#348](https://github.com/dowdiness/loom/pull/348), [#350](https://github.com/dowdiness/loom/pull/350), [#351](https://github.com/dowdiness/loom/pull/351), [#352](https://github.com/dowdiness/loom/pull/352), [#353](https://github.com/dowdiness/loom/pull/353), [#354](https://github.com/dowdiness/loom/pull/354), [#355](https://github.com/dowdiness/loom/pull/355), [#356](https://github.com/dowdiness/loom/pull/356), [#357](https://github.com/dowdiness/loom/pull/357), [#358](https://github.com/dowdiness/loom/pull/358)

**incr**

incr [#281](https://github.com/dowdiness/incr/pull/281), [#282](https://github.com/dowdiness/incr/pull/282), [#284](https://github.com/dowdiness/incr/pull/284), [#285](https://github.com/dowdiness/incr/pull/285)

**js_engine**

js_engine [#367](https://github.com/dowdiness/js_engine/pull/367), [#368](https://github.com/dowdiness/js_engine/pull/368), [#369](https://github.com/dowdiness/js_engine/pull/369), [#370](https://github.com/dowdiness/js_engine/pull/370), [#371](https://github.com/dowdiness/js_engine/pull/371), [#372](https://github.com/dowdiness/js_engine/pull/372)

### 2026/6/18

**Canopy / analysis query layer実装**

canopy [#699](https://github.com/dowdiness/canopy/pull/699), [#692](https://github.com/dowdiness/canopy/pull/692), [#693](https://github.com/dowdiness/canopy/pull/693), [#694](https://github.com/dowdiness/canopy/pull/694), [#695](https://github.com/dowdiness/canopy/pull/695)

**loom / MarkdownIR**

loom [#359](https://github.com/dowdiness/loom/pull/359), [#360](https://github.com/dowdiness/loom/pull/360), [#361](https://github.com/dowdiness/loom/pull/361)

**incr / Incremental TEA**

incr [#291](https://github.com/dowdiness/incr/pull/291), [#268](https://github.com/dowdiness/incr/pull/268), [#286](https://github.com/dowdiness/incr/pull/286), [#287](https://github.com/dowdiness/incr/pull/287), [#288](https://github.com/dowdiness/incr/pull/288), [#289](https://github.com/dowdiness/incr/pull/289), [#290](https://github.com/dowdiness/incr/pull/290)

**js_engine**

js_engine [#373](https://github.com/dowdiness/js_engine/pull/373), [#374](https://github.com/dowdiness/js_engine/pull/374), [#375](https://github.com/dowdiness/js_engine/pull/375), [#310](https://github.com/dowdiness/js_engine/pull/310), [#357](https://github.com/dowdiness/js_engine/pull/357)

### 2026/6/19

**Canopy / Markdown SDEG heading**

canopy [#716](https://github.com/dowdiness/canopy/pull/716), [#717](https://github.com/dowdiness/canopy/pull/717), [#718](https://github.com/dowdiness/canopy/pull/718), [#719](https://github.com/dowdiness/canopy/pull/719)

**js_engine**

js_engine [#405](https://github.com/dowdiness/js_engine/pull/405), [#403](https://github.com/dowdiness/js_engine/pull/403), [#402](https://github.com/dowdiness/js_engine/pull/402), [#401](https://github.com/dowdiness/js_engine/pull/401)

### 2026/6/20

**Canopy / Markdown list SDEG**

canopy [#722](https://github.com/dowdiness/canopy/pull/722), [#723](https://github.com/dowdiness/canopy/pull/723), [#726](https://github.com/dowdiness/canopy/pull/726), [#730](https://github.com/dowdiness/canopy/pull/730), [#731](https://github.com/dowdiness/canopy/pull/731)

**js_engine / test262適合率向上**

js_engine [#408](https://github.com/dowdiness/js_engine/pull/408), [#419](https://github.com/dowdiness/js_engine/pull/419), [#413](https://github.com/dowdiness/js_engine/pull/413), [#412](https://github.com/dowdiness/js_engine/pull/412), [#411](https://github.com/dowdiness/js_engine/pull/411), [#420](https://github.com/dowdiness/js_engine/pull/420), [#410](https://github.com/dowdiness/js_engine/pull/410), [#407](https://github.com/dowdiness/js_engine/pull/407)

### 2026/6/21

**Canopy / Markdown list + moon.mod移行開始**

canopy [#736](https://github.com/dowdiness/canopy/pull/736), [#737](https://github.com/dowdiness/canopy/pull/737), [#738](https://github.com/dowdiness/canopy/pull/738)

**js_engine / lexer・spec fixes・perf**

js_engine [#422](https://github.com/dowdiness/js_engine/pull/422), [#431](https://github.com/dowdiness/js_engine/pull/431), [#428](https://github.com/dowdiness/js_engine/pull/428), [#332](https://github.com/dowdiness/js_engine/pull/332), [#433](https://github.com/dowdiness/js_engine/pull/433)

</details>

<details>
<summary>6月第4週（6/22〜6/26）</summary>

### 2026/6/22

**Canopy / moon.mod移行完了**

canopy [#740](https://github.com/dowdiness/canopy/pull/740), [#335](https://github.com/dowdiness/canopy/pull/335)

### 2026/6/23

**Canopy / submodule bumpとHTML block修正**

canopy [#742](https://github.com/dowdiness/canopy/pull/742)

### 2026/6/24

**Canopy / Markdown SDEG lifecycle**

canopy [#746](https://github.com/dowdiness/canopy/pull/746), [#750](https://github.com/dowdiness/canopy/pull/750), [#754](https://github.com/dowdiness/canopy/pull/754), [#755](https://github.com/dowdiness/canopy/pull/755), [#763](https://github.com/dowdiness/canopy/pull/763)

**Canopy / benchmark CIとprojection map**

canopy [#761](https://github.com/dowdiness/canopy/pull/761), [#762](https://github.com/dowdiness/canopy/pull/762)

**loom / MarkdownIR**

loom [#472](https://github.com/dowdiness/loom/pull/472), [#473](https://github.com/dowdiness/loom/pull/473)

**js_engine**

js_engine [#438](https://github.com/dowdiness/js_engine/pull/438), [#440](https://github.com/dowdiness/js_engine/pull/440), [#441](https://github.com/dowdiness/js_engine/pull/441), [#442](https://github.com/dowdiness/js_engine/pull/442)

### 2026/6/25

**Canopy / SDEG snapshot validity**

canopy [#766](https://github.com/dowdiness/canopy/pull/766), [#767](https://github.com/dowdiness/canopy/pull/767)

**Canopy / ProjNode mapとCI cleanup**

canopy [#765](https://github.com/dowdiness/canopy/pull/765), [#727](https://github.com/dowdiness/canopy/pull/727), [#547](https://github.com/dowdiness/canopy/pull/547), [#549](https://github.com/dowdiness/canopy/pull/549), [#550](https://github.com/dowdiness/canopy/pull/550)

**js_engine**

js_engine [#445](https://github.com/dowdiness/js_engine/pull/445), [#446](https://github.com/dowdiness/js_engine/pull/446), [#449](https://github.com/dowdiness/js_engine/pull/449)

### 2026/6/26

**Canopy / JSON role spans + CI cleanup**

canopy [#781](https://github.com/dowdiness/canopy/pull/781), [#782](https://github.com/dowdiness/canopy/pull/782), [#783](https://github.com/dowdiness/canopy/pull/783), [#773](https://github.com/dowdiness/canopy/pull/773), [#777](https://github.com/dowdiness/canopy/pull/777), [#778](https://github.com/dowdiness/canopy/pull/778), [#779](https://github.com/dowdiness/canopy/pull/779), [#780](https://github.com/dowdiness/canopy/pull/780)

**Loom / arrow lambda + Pratt reuse**

loom [#475](https://github.com/dowdiness/loom/pull/475), [#476](https://github.com/dowdiness/loom/pull/476)

**js_engine / NFE binding + async fixes + test262整備**

js_engine [#463](https://github.com/dowdiness/js_engine/pull/463), [#468](https://github.com/dowdiness/js_engine/pull/468), [#466](https://github.com/dowdiness/js_engine/pull/466), [#467](https://github.com/dowdiness/js_engine/pull/467), [#471](https://github.com/dowdiness/js_engine/pull/471), [#462](https://github.com/dowdiness/js_engine/pull/462), [#461](https://github.com/dowdiness/js_engine/pull/461), [#460](https://github.com/dowdiness/js_engine/pull/460), [#470](https://github.com/dowdiness/js_engine/pull/470)

</details>

<details>
<summary>6月第5週（6/27〜6/30）</summary>

### 2026/6/27

**Canopy / block-editor drag-dropとcore primitives**

canopy [#793](https://github.com/dowdiness/canopy/pull/793), [#794](https://github.com/dowdiness/canopy/pull/794), [#796](https://github.com/dowdiness/canopy/pull/796), [#798](https://github.com/dowdiness/canopy/pull/798), [#800](https://github.com/dowdiness/canopy/pull/800), [#276](https://github.com/dowdiness/canopy/pull/276)

**loom / loomgenの誕生**

incr [#313](https://github.com/dowdiness/incr/pull/313), [#314](https://github.com/dowdiness/incr/pull/314), [#315](https://github.com/dowdiness/incr/pull/315), [#316](https://github.com/dowdiness/incr/pull/316), [#317](https://github.com/dowdiness/incr/pull/317), [#318](https://github.com/dowdiness/incr/pull/318), [#320](https://github.com/dowdiness/incr/pull/320), [#322](https://github.com/dowdiness/incr/pull/322), [#323](https://github.com/dowdiness/incr/pull/323) / js_engine [#476](https://github.com/dowdiness/js_engine/pull/476), [#477](https://github.com/dowdiness/js_engine/pull/477), [#478](https://github.com/dowdiness/js_engine/pull/478), [#479](https://github.com/dowdiness/js_engine/pull/479), [#480](https://github.com/dowdiness/js_engine/pull/480), [#481](https://github.com/dowdiness/js_engine/pull/481), [#482](https://github.com/dowdiness/js_engine/pull/482) / loom [#492](https://github.com/dowdiness/loom/pull/492), [#493](https://github.com/dowdiness/loom/pull/493), [#494](https://github.com/dowdiness/loom/pull/494), [#496](https://github.com/dowdiness/loom/pull/496), [#505](https://github.com/dowdiness/loom/pull/505), [#510](https://github.com/dowdiness/loom/pull/510), [#511](https://github.com/dowdiness/loom/pull/511)

### 2026/6/28

**Canopy / typed error modelとJSON tree editor**

canopy [#799](https://github.com/dowdiness/canopy/pull/799), [#802](https://github.com/dowdiness/canopy/pull/802), [#806](https://github.com/dowdiness/canopy/pull/806), [#810](https://github.com/dowdiness/canopy/pull/810), [#811](https://github.com/dowdiness/canopy/pull/811), [#812](https://github.com/dowdiness/canopy/pull/812), [#813](https://github.com/dowdiness/canopy/pull/813)

**loom / loomgenのアノテーション語彙が拡張**

incr [#326](https://github.com/dowdiness/incr/pull/326), [#327](https://github.com/dowdiness/incr/pull/327), [#329](https://github.com/dowdiness/incr/pull/329), [#331](https://github.com/dowdiness/incr/pull/331), [#332](https://github.com/dowdiness/incr/pull/332), [#333](https://github.com/dowdiness/incr/pull/333), [#334](https://github.com/dowdiness/incr/pull/334) / loom [#513](https://github.com/dowdiness/loom/pull/513), [#515](https://github.com/dowdiness/loom/pull/515), [#517](https://github.com/dowdiness/loom/pull/517), [#518](https://github.com/dowdiness/loom/pull/518), [#519](https://github.com/dowdiness/loom/pull/519), [#525](https://github.com/dowdiness/loom/pull/525), [#527](https://github.com/dowdiness/loom/pull/527), [#528](https://github.com/dowdiness/loom/pull/528), [#533](https://github.com/dowdiness/loom/pull/533)

### 2026/6/29

**loom**

canopy [#814](https://github.com/dowdiness/canopy/pull/814), [#816](https://github.com/dowdiness/canopy/pull/816), [#819](https://github.com/dowdiness/canopy/pull/819), [#797](https://github.com/dowdiness/canopy/pull/797) / incr [#336](https://github.com/dowdiness/incr/pull/336), [#337](https://github.com/dowdiness/incr/pull/337), [#338](https://github.com/dowdiness/incr/pull/338) / loom [#534](https://github.com/dowdiness/loom/pull/534), [#535](https://github.com/dowdiness/loom/pull/535), [#536](https://github.com/dowdiness/loom/pull/536), [#542](https://github.com/dowdiness/loom/pull/542), [#544](https://github.com/dowdiness/loom/pull/544), [#545](https://github.com/dowdiness/loom/pull/545)

### 2026/6/30

**loom**

canopy [#820](https://github.com/dowdiness/canopy/pull/820) / incr [#339](https://github.com/dowdiness/incr/pull/339), [#340](https://github.com/dowdiness/incr/pull/340) / loom [#546](https://github.com/dowdiness/loom/pull/546), [#548](https://github.com/dowdiness/loom/pull/548)

</details>

