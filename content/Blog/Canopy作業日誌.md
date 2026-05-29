---
title: Canopy作業日誌
publish: true
tags: [projectional-editing]
aliases: [Canopy]
created: 2026-01-04T20:50:52+09:00
modified: 2026-05-30T00:48:22+09:00
---

# Canopy作業日誌

2026年5月18日から29日ごろまでの作業ログ。
中心はCanopyというprojectional editorの実験で、周辺ライブラリのloom、incr、moondsp、js_engineも同時に触っている。

かなり生の作業メモに近いので、細かいPRリンクもそのまま残している。
大まかには、前半はRabbitaとCodeMirrorの接続、途中からInspectorやLoom連携、後半はCognitionというAI用ナレッジベース機能の土台づくりに進んでいる。
incrについては、[Build Systems à la Carte](https://hackage.haskell.org/package/build)を読みながら、自分のライブラリのAPIや評価モデルを整理していった時期でもある。

この記事に出てくる主なRepository:

- [Canopy](https://github.com/dowdiness/canopy): projectional editorとAI用ナレッジベース機能まわりの実験をしている中心リポジトリ。
- [loom](https://github.com/dowdiness/loom): incremental parserやCST projectionを扱うライブラリ。
- [incr](https://github.com/dowdiness/incr): incremental computation用の小さなライブラリ。
- [moondsp](https://github.com/dowdiness/moondsp): MoonBitでDSPや音楽記述用DSLの実験をしているリポジトリ。
- [js_engine](https://github.com/dowdiness/js_engine): MoonBitでのJavaScript engineの実装。

本文でよく出てくる言葉:

- Rabbita: Canopyで使っているUI / editor側の実験的な仕組み。
- CodeMirror: テキストエディタ部分に使っている既存のエディタライブラリ。
- Cognition: Canopy上でAIに渡すcontextやproviderとの境界を扱うための実験。
- FFI: MoonBit側のコードとJavaScript / DOM側のコードをつなぐ境界。
- provider boundary: AI providerへ渡す入力、返ってきた結果、キャンセルやretryを追跡するための境界。

この期間の大きな流れ:

1. RabbitaとCodeMirrorの接続を安定させる。
2. DOMの隠しボタン経由だった操作を、イベント購読や明示的な境界に寄せる。
3. Inspectorやop logを整えて、内部状態を追いやすくする。
4. [Build Systems à la Carte](https://hackage.haskell.org/package/build)を参照しながら、incrのAPIと評価モデルを整理する。
5. Cognitionのworkspace、context packing、provider boundaryの土台を作る。
6. 最後にephemeral presenceやbyte codecを切り出して、共有部品として扱えるようにする。

## 2026/5/18

incrのDatalogを使ってUI開発が出来ないかの実験をした。
DataScriptみたいなことが出来るかもしれない。
https://github.com/tonsky/datascript
関連PR: [loom PR #124](https://github.com/dowdiness/loom/pull/124)

RabbitaとCodeMirrorの連携するグルーコードを改善した。
現在はグチャグチャと混沌としたコードになっていてバグが多く、エディターを触っていると途中で動かなくなってしまうことがあったのが直った。
関連PR: [Canopy PR #293](https://github.com/dowdiness/canopy/pull/293)、[Canopy PR #296](https://github.com/dowdiness/canopy/pull/296)
https://claude.ai/share/ed68c575-c459-415d-bd19-b7965dd94e29

## 2026/5/19

### Canopy

canopyでのrabbita_codemirror（CodeMirrorバインディング）の作成を続けた。
途中でバグが見つかって既存のコードを改修する必要が生まれてそれの改修に時間が掛かった。
関連PR: [Canopy PR #297](https://github.com/dowdiness/canopy/pull/297)、[Canopy PR #299](https://github.com/dowdiness/canopy/pull/299)、[Canopy PR #300](https://github.com/dowdiness/canopy/pull/300)、[Canopy PR #301](https://github.com/dowdiness/canopy/pull/301)、[Canopy PR #302](https://github.com/dowdiness/canopy/pull/302)

プログラミング言語の変数の入れ替えをする機能をincrを使って出来るように環境を整えた。
関連PR: [loom PR #126](https://github.com/dowdiness/loom/pull/126)、[loom PR #129](https://github.com/dowdiness/loom/pull/129)

### moondsp

AudioBuffer APIの改修をした。`as_fixed_array`で内部の `fixed_array` に直接アクセスしないといけなかったのをAudioBufferの構造体から`all` / `any`などのメソッドを直接呼べるようにした。
関連PR: [moondsp PR #60](https://github.com/dowdiness/moondsp/pull/60)、[moondsp PR #62](https://github.com/dowdiness/moondsp/pull/62)

### トークンの使用量を減らす工夫

コーディングエージェントの使用するトークン量を減らすべく [BAML](https://boundaryml.com/) を導入した。
どれくらい効果があるのかまだ分からないので実際に使ってみて確かめる必要がある。

## 2026/5/20

### Canopy / Rabbita CodeMirror

関連PR: [Canopy PR #303](https://github.com/dowdiness/canopy/pull/303)、[Canopy PR #305](https://github.com/dowdiness/canopy/pull/305)、[Canopy PR #306](https://github.com/dowdiness/canopy/pull/306)、[Canopy PR #307](https://github.com/dowdiness/canopy/pull/307)

Canopyの `examples/ideal` で、RabbitaとCodeMirrorのバインディング移行を進めた。
移行が完了するまで既存実装と新しいバインディングを共存させられるようにし、マウント手順と、二重DOM、フラグ読み取り、undo記録まわりの問題を潰した。

CodeMirrorのselectionやextensionなどをCodeMirror専用のAPIとして閉じ込めずに、標準の[Selection](https://developer.mozilla.org/ja/docs/Web/API/Selection) / [Range](https://developer.mozilla.org/ja/docs/Web/API/Range) APIやCodeMirror本体のAPIを薄いJS FFIとして扱う方向に決定した。

### loom

Lambda exampleのAPIの整理をした
関連PR: [loom PR #131](https://github.com/dowdiness/loom/pull/131)、[loom PR #132](https://github.com/dowdiness/loom/pull/132)

### incr

関連PR: [incr PR #58](https://github.com/dowdiness/incr/pull/58)、[incr PR #59](https://github.com/dowdiness/incr/pull/59)、[incr PR #60](https://github.com/dowdiness/incr/pull/60)

visualizationの機能を追加するために内部の整理をして `EventBroadcastPhaseHook` を追加した。
これ専用のpublic APIは増やさず、パフォーマンスを意識してRuntimeのコンストラクタに合わせてイベントをbufferしてから一気にbatch処理する、イベントリスナを使わない形にした。
benchmarkをとって数パーセントほど実行速度が向上していることを確認した。

### moondsp

AudioBufferのwrite-time validationの設計を進めた。
`new` / `filled` / `fill` / `set` を共通のサンプル検証・正規化パスへ通す方針にし、`adopt` はゼロコピー契約上、採用後に外部ハンドルから変更された値まではMoonBit側で検証できないことを明確にした。
-1~+1以外の値を正規化するようにした。
関連PR: [moondsp PR #63](https://github.com/dowdiness/moondsp/pull/63)

### js_engine

well-known symbolの所有権をrealm側へ移し、[js_engine PR #130](https://github.com/dowdiness/js_engine/pull/130)をマージした。`setup_builtins(env, output, symbols, ...)` を単体で呼ぶ場合も、渡された `SymbolState` でwell-known symbolsを割り当てるよう直した。

## 2026/5/21

### Canopy

Canopyの `examples/ideal` で、RabbitaとDOMイベントの境界を整理した。
[Canopy PR #312](https://github.com/dowdiness/canopy/pull/312) でRabbitaに依存しないDOM boundary helperを追加し、[Canopy PR #313](https://github.com/dowdiness/canopy/pull/313) から [Canopy PR #316](https://github.com/dowdiness/canopy/pull/316) でoverlay、sync、structure modeのhidden buttonという名前で設定していた命令的なトリガーをRabbitaのcustom event経由のイベントサブスクリプションに寄せた。

DOMイベントサブスクリプションの失敗をログに出す変更も入れたので、Rabbita側のイベント登録で問題が起きたときに原因を追いやすくなった。
IdealのUI操作をDOMの隠しボタンに依存させるより、Rabbita側のイベント購読として扱う方が、あとから読んだときに責務の境界が分かりやすい。

### loom

Loomではincremental parser reuseまわりのTODOを進めた。
削除時の左隣CST reuseや、削除後にoffsetがずれたnodeを再利用するケースをテストで固定し、現在のinvariantはparser-ownedなtoken/subtree identityではなく、チェック済みのCST subtree reuseであることを確認した。
関連PR: [loom PR #134](https://github.com/dowdiness/loom/pull/134)、[loom PR #135](https://github.com/dowdiness/loom/pull/135)、[loom PR #136](https://github.com/dowdiness/loom/pull/136)

### incr

incrではpublic APIの命名とread helperの整理を続けた。
`read`まわりのpermissiveなhelperをrenameし、理想的なAPIへの移行計画を作った。
関連PR: [incr PR #61](https://github.com/dowdiness/incr/pull/61)、[incr PR #62](https://github.com/dowdiness/incr/pull/62)、[incr PR #63](https://github.com/dowdiness/incr/pull/63)

### js_engine

js_engineではwell-known symbol lookupの移行を進めた。
[js_engine PR #131](https://github.com/dowdiness/js_engine/pull/131) と [js_engine PR #132](https://github.com/dowdiness/js_engine/pull/132) で、runtimeやstdlibの残っていたno-argument symbol getter系を明示的なrealm-owned `WellKnownSymbols` accessへ移し、互換用のlegacy pathを削除した。

## 2026/5/22

### Canopy

CanopyでRabbita側のundo/redoショートカットもhidden button経由から外した。
[Canopy PR #318](https://github.com/dowdiness/canopy/pull/318) でCodeMirror側が `request-undo` / `request-redo` custom eventをdispatchし、Rabbita側がそれを `Undo` / `Redo` に変換する形になった。

これで前日から続いていたhidden button経由のトリガー削除が一段落した。
その後、`examples/web` と `examples/ideal` に対する `tsc --noEmit` のCI jobを追加し、Inspectorにincr runtime snapshotを表示する作業も入った。
関連PR: [Canopy PR #320](https://github.com/dowdiness/canopy/pull/320)、[Canopy PR #321](https://github.com/dowdiness/canopy/pull/321)

### incr

incrではtarget API facadeの作業が進んだ。
[incr PR #68](https://github.com/dowdiness/incr/pull/68) で理想的なAPIのfacadeを追加し、その後runtime read helperのdeprecation、input freshness facade、map relation facadeが続いた。
Canopy側から使うAPIを薄く整えながら、古いread helperに直接依存しない方向へ寄せた。
関連PR: [incr PR #69](https://github.com/dowdiness/incr/pull/69)、[incr PR #70](https://github.com/dowdiness/incr/pull/70)、[incr PR #71](https://github.com/dowdiness/incr/pull/71)、[incr PR #72](https://github.com/dowdiness/incr/pull/72)

### js_engine

iterator cacheやprimitive wrapper prototypeの状態を `RealmState` に移した。
[js_engine PR #133](https://github.com/dowdiness/js_engine/pull/133) と [js_engine PR #134](https://github.com/dowdiness/js_engine/pull/134) がこの流れの中心で、factory/prototype系の状態をmodule globalからrealm-owned stateへ移す作業を続けた。

注: [Realms](https://tc39.es/ecma262/#sec-code-realms)とはECMAScriptの仕様です。

## 2026/5/23

### Canopy

CanopyではInspectorまわりの作業を続けた。
InspectorにPatch panelを追加し、`view_op_log` / `view_patch` のguard、Op Log label formatの統一、`SourceMap::nodes_at_position` の範囲制約の修正を入れた。
Idealを触りながら内部状態を確認するための道具立てが増え、op logやpatchを見て原因を追いやすくなった。
関連PR: [Canopy PR #323](https://github.com/dowdiness/canopy/pull/323)、[Canopy PR #324](https://github.com/dowdiness/canopy/pull/324)、[Canopy PR #327](https://github.com/dowdiness/canopy/pull/327)、[Canopy PR #329](https://github.com/dowdiness/canopy/pull/329)

### incr

incrではAPI migrationに向けたドキュメントとexamplesを増やした。
architectureやcookbook、API referenceの例を新しいAPI名に合わせて更新し、[Build Systems à la Carte](https://hackage.haskell.org/package/build)を読みながら、自分の実装がどの評価戦略や依存関係モデルに近いのかをdocsに記録した。
関連PR: [incr PR #73](https://github.com/dowdiness/incr/pull/73)、[incr PR #74](https://github.com/dowdiness/incr/pull/74)、[incr PR #75](https://github.com/dowdiness/incr/pull/75)、[incr PR #76](https://github.com/dowdiness/incr/pull/76)、[incr PR #77](https://github.com/dowdiness/incr/pull/77)、[incr PR #78](https://github.com/dowdiness/incr/pull/78)、[incr PR #79](https://github.com/dowdiness/incr/pull/79)

### moondsp

moondspではLoomを使ったmini記法の検証を進めた。
`specs/loom-mini-cst` のgrammar parityに向けて、checked target API examplesやloom-mini-cstのdocsを更新し、Loom側のAPI driftをspecで拾える状態にした。
production parserへすぐ切り替えるのではなく、まずspec側でLoomの挙動を固定していく方針になっている。

### js_engine

js_engineではprototype移行を続けた。
object function、Promise、WeakMap / WeakSet、Map / Set、Array prototypeなどのlookupやstorageを順に `RealmState` に寄せ、module globalに残っていたprototype参照を減らした。
関連PR: [js_engine PR #135](https://github.com/dowdiness/js_engine/pull/135)、[js_engine PR #136](https://github.com/dowdiness/js_engine/pull/136)、[js_engine PR #137](https://github.com/dowdiness/js_engine/pull/137)、[js_engine PR #138](https://github.com/dowdiness/js_engine/pull/138)、[js_engine PR #139](https://github.com/dowdiness/js_engine/pull/139)

## 2026/5/24

### Canopy / loom

[Loom issue #147](https://github.com/dowdiness/loom/issues/147) の移行をCanopy側まで進めた。
`text_change` と `moji` をCanopy配下ではなくLoom monorepo側のtop-level moduleへ移す方針を決めた。

実装としては [loom PR #149](https://github.com/dowdiness/loom/pull/149) で `text_change` / `moji` がLoom側へ移り、Canopy側では [Canopy PR #341](https://github.com/dowdiness/canopy/pull/341) で `./loom/text-change` と `./loom/moji` を参照するようにした。
Canopy内の `lib/text-change` と `lib/moji`、使われていなかった `valtio` submoduleも整理した。
Loomを単体でbuildしやすくするための移行で、Canopy側に置かれていた共通部品をLoom側の責任範囲へ戻した。

### incr

incrは [incr PR #81](https://github.com/dowdiness/incr/pull/81) でv0.6.0のreleaseを行った。
その後、CanopyやLoom側の利用に合わせて、ファイル名やskillの記述を新しいAPIの名前へ寄せた。
関連PR: [incr PR #82](https://github.com/dowdiness/incr/pull/82)、[incr PR #83](https://github.com/dowdiness/incr/pull/83)

### moondsp

moondspではLoom mini CSTの改良を続けた。
[moondsp PR #75](https://github.com/dowdiness/moondsp/pull/75) でquickcheckを0.14.0へ上げ、[moondsp PR #76](https://github.com/dowdiness/moondsp/pull/76) でloom-mini-cstのgrammarを広げた。

この時点でもproduction parserはLoomへ切り替えておらず、Loomはまだspecと回帰テスト側で使う位置づけ。
関連PR: [moondsp PR #71](https://github.com/dowdiness/moondsp/pull/71)、[moondsp PR #73](https://github.com/dowdiness/moondsp/pull/73)、[moondsp PR #74](https://github.com/dowdiness/moondsp/pull/74)、[moondsp PR #79](https://github.com/dowdiness/moondsp/pull/79)

### js_engine

RealmState移行をさらに進めた。
runtimeのMap / Set、boxed primitive、Array、WeakMap / WeakSet、ArrayBuffer storageなどを順に `RealmState` 側へ寄せ、CI workflowのNode.js runtime更新とdeprecatedな `moon install` 呼び出しの削除も行った。
関連PR: [js_engine PR #140](https://github.com/dowdiness/js_engine/pull/140)、[js_engine PR #142](https://github.com/dowdiness/js_engine/pull/142)、[js_engine PR #143](https://github.com/dowdiness/js_engine/pull/143)、[js_engine PR #144](https://github.com/dowdiness/js_engine/pull/144)、[js_engine PR #146](https://github.com/dowdiness/js_engine/pull/146)、[js_engine PR #147](https://github.com/dowdiness/js_engine/pull/147)、[js_engine PR #148](https://github.com/dowdiness/js_engine/pull/148)、[js_engine PR #149](https://github.com/dowdiness/js_engine/pull/149)、[js_engine PR #151](https://github.com/dowdiness/js_engine/pull/151)

## 2026/5/25

### Canopy

Lambda metadataをeditor/workspace/FFIの境界に通す変更を進めた。
`ffi/lambda` のrouting、workspace coordination、Editor側のmetadata受け渡し、typed workflow port handlerが追加され、Lambda exampleをCognition側の流れに接続する準備が進んだ。
Lambda exampleを単なるサンプルとしてではなく、workspaceやCognitionの実験台として使えるようにする作業だった。
関連PR: [Canopy PR #345](https://github.com/dowdiness/canopy/pull/345)、[Canopy PR #347](https://github.com/dowdiness/canopy/pull/347)、[Canopy PR #348](https://github.com/dowdiness/canopy/pull/348)、[Canopy PR #349](https://github.com/dowdiness/canopy/pull/349)、[Canopy PR #350](https://github.com/dowdiness/canopy/pull/350)

### loom

Loomでは `Memo` から `Derived` への用語・API整理に合わせて、`examples/lambda` を更新した。
また、seamにdirect CST query helperを追加し、CST projection guideをdocsに追加した。
関連PR: [loom PR #152](https://github.com/dowdiness/loom/pull/152)、[loom PR #154](https://github.com/dowdiness/loom/pull/154)、[loom PR #155](https://github.com/dowdiness/loom/pull/155)、[loom PR #156](https://github.com/dowdiness/loom/pull/156)

### moondsp

moondspではLoom mini CSTからprojection method IRを検証する作業を行った。
[moondsp PR #80](https://github.com/dowdiness/moondsp/pull/80) でprojection method IRをvalidateし、apply-editの自動テストやloop expression style guidanceも追加した。
関連PR: [moondsp PR #81](https://github.com/dowdiness/moondsp/pull/81)、[moondsp PR #83](https://github.com/dowdiness/moondsp/pull/83)、[moondsp PR #84](https://github.com/dowdiness/moondsp/pull/84)、[moondsp PR #85](https://github.com/dowdiness/moondsp/pull/85)、[moondsp PR #87](https://github.com/dowdiness/moondsp/pull/87)

### js_engine

js_engineではconstruct/call contextの明示化を進めた。
ArrayBufferを `RealmState` に移した後の流れとして、construction stateを明示的なcall contextへ移し、ambient interpreter context fallbackを削除した。
関連PR: [js_engine PR #152](https://github.com/dowdiness/js_engine/pull/152)

## 2026/5/26

### Canopy

CanopyではCognitionまわりの基盤を進めた。
workspace filesを追跡する [Canopy PR #357](https://github.com/dowdiness/canopy/pull/357)、minimal incremental reactive layer、context packing API、provider boundaryの計画とdocsを追加し、削除ファイルの依存関係を掃除する修正も入った。
関連PR: [Canopy PR #355](https://github.com/dowdiness/canopy/pull/355)、[Canopy PR #358](https://github.com/dowdiness/canopy/pull/358)、[Canopy PR #359](https://github.com/dowdiness/canopy/pull/359)、[Canopy PR #360](https://github.com/dowdiness/canopy/pull/360)、[Canopy PR #363](https://github.com/dowdiness/canopy/pull/363)、[Canopy PR #364](https://github.com/dowdiness/canopy/pull/364)

同じ流れで、Lambda側はLoomの `LambdaAnalysis` attachmentを使う形へ寄せた。
Cognitionが参照するファイル、依存関係、contextを明示的に扱えるようにすることで、後続のprovider連携へ進む土台を作った。
関連PR: [Canopy PR #362](https://github.com/dowdiness/canopy/pull/362)

### incr

incrではAPIの大きな整理を続けた。
safe incremental refactorとしてtypesとcorrectnessを整理し、pipeline traitsをdeprecateした。
expr formula APIの設計もdocsに残した。
関連PR: [incr PR #87](https://github.com/dowdiness/incr/pull/87)、[incr PR #89](https://github.com/dowdiness/incr/pull/89)

### moondsp

moondspではLoom mini CSTのcoverageを広げた。
slow postfix projection、degrade / euclid projection、dollar stack parity、sub-notation postfix parity、callback method projectionなどを追加し、Loom miniがどこまでproduction miniの構文に追いつけるかを確認した。
関連PR: [moondsp PR #88](https://github.com/dowdiness/moondsp/pull/88)、[moondsp PR #89](https://github.com/dowdiness/moondsp/pull/89)、[moondsp PR #90](https://github.com/dowdiness/moondsp/pull/90)、[moondsp PR #91](https://github.com/dowdiness/moondsp/pull/91)、[moondsp PR #92](https://github.com/dowdiness/moondsp/pull/92)、[moondsp PR #93](https://github.com/dowdiness/moondsp/pull/93)、[moondsp PR #94](https://github.com/dowdiness/moondsp/pull/94)、[moondsp PR #95](https://github.com/dowdiness/moondsp/pull/95)

### js_engine

js_engineではborrowed built-in realm routingを修正した。
[js_engine PR #153](https://github.com/dowdiness/js_engine/pull/153) の変更として、built-in realmの扱いを明示的なroutingへ寄せた。

## 2026/5/27

### Canopy

Canopyではprovider boundaryの設計を実装側へ進めた。
provider boundary domainを追加し、provider boundary planをretargetした。
前日までのrecompute cleanupやcontext packingの作業を受けて、Cognitionが外部providerへ渡す境界を整理している段階になった。
外部providerの結果をそのまま受け入れるのではなく、どの入力とcontextに対する結果なのかを追える形にする必要があった。
関連PR: [Canopy PR #365](https://github.com/dowdiness/canopy/pull/365)

### incr

incrではPhase 3a facade migrationのdocsを入れ、evaluation strategyをkernelから切り出した。
[incr PR #94](https://github.com/dowdiness/incr/pull/94) では現在のincrのモデルをdocsにまとめている。
関連PR: [incr PR #90](https://github.com/dowdiness/incr/pull/90)、[incr PR #91](https://github.com/dowdiness/incr/pull/91)、[incr PR #92](https://github.com/dowdiness/incr/pull/92)、[incr PR #93](https://github.com/dowdiness/incr/pull/93)、[incr PR #96](https://github.com/dowdiness/incr/pull/96)

### moondsp

moondspではLoom mini CSTのknown edgeをcharacterizeした。
[moondsp PR #99](https://github.com/dowdiness/moondsp/pull/99) でmode-incompatibleなmini atomをrejectする挙動や、known edge follow-upの状態をdocsに残した。
関連PR: [moondsp PR #96](https://github.com/dowdiness/moondsp/pull/96)、[moondsp PR #97](https://github.com/dowdiness/moondsp/pull/97)、[moondsp PR #100](https://github.com/dowdiness/moondsp/pull/100)、[moondsp PR #102](https://github.com/dowdiness/moondsp/pull/102)

### js_engine

js_engineではstartup benchmark stagingとbenchmark summary renderingの整理を行った。
[js_engine PR #154](https://github.com/dowdiness/js_engine/pull/154) でJS startup benchmarkの足場を追加し、[js_engine PR #155](https://github.com/dowdiness/js_engine/pull/155) でbenchmark dashboardの表示を整理した。

## 2026/5/28

### Canopy

Canopyではprovider planningとLambda semantic側の作業を続けた。
provider planningとlambda semantic overlayを追加し、workspace memoのsmoke test、workspace memo lifecycle API、reactive provider boundary driverまで進めた。
`lib/cognition/provider_boundary_store.mbt` と `lib/cognition/reactive.mbt` にprovider planning graphを接続し、cancellation、completion、retry classification、driver actionを `@incr` の内部状態として扱う方向が固まってきた。
関連PR: [Canopy PR #367](https://github.com/dowdiness/canopy/pull/367)、[Canopy PR #368](https://github.com/dowdiness/canopy/pull/368)、[Canopy PR #372](https://github.com/dowdiness/canopy/pull/372)、[Canopy PR #379](https://github.com/dowdiness/canopy/pull/379)

provider cancellationのidempotency、driver shutdown時にpending requestを観測できること、file removalやbudgeted context変更後のstale completionを拒否するテストも追加されている。
providerの応答が遅れて返ってきたときに、古いcontextの結果を現在の状態へ混ぜないための整理になっている。
Lambda、JSON、MarkdownのFFI read accessorもcoordinator経由のprotected readへ寄せた。
workspaceの更新中に外側から半端な状態を読まないようにするための変更で、Cognitionのprovider連携を進める前に境界を固める作業になった。
関連PR: [Canopy PR #370](https://github.com/dowdiness/canopy/pull/370)、[Canopy PR #374](https://github.com/dowdiness/canopy/pull/374)、[Canopy PR #375](https://github.com/dowdiness/canopy/pull/375)、[Canopy PR #376](https://github.com/dowdiness/canopy/pull/376)、[Canopy PR #377](https://github.com/dowdiness/canopy/pull/377)、[Canopy PR #378](https://github.com/dowdiness/canopy/pull/378)

### incr

incrではruntime evaluation event APIまわりを進めた。
internal runtime evaluation eventsとevaluation strategy bundleを追加し、static derived fast pathのbenchmarkも取った。
honest read-error ownershipの設計をdocsに残し、`Derived::fallible` / `DerivedMap::fallible` を追加した。
関連PR: [incr PR #95](https://github.com/dowdiness/incr/pull/95)、[incr PR #97](https://github.com/dowdiness/incr/pull/97)、[incr PR #98](https://github.com/dowdiness/incr/pull/98)

### moondsp

moondspではloom-mini-cstのprovenance matrix coverageとcontrol method projection parityを追加した。
Loom移行に向けて、upstreamへ要求する挙動やrecovery stateをdocsに分け、回復処理のevidenceも増やした。
関連PR: [moondsp PR #101](https://github.com/dowdiness/moondsp/pull/101)、[moondsp PR #104](https://github.com/dowdiness/moondsp/pull/104)、[moondsp PR #106](https://github.com/dowdiness/moondsp/pull/106)、[moondsp PR #107](https://github.com/dowdiness/moondsp/pull/107)、[moondsp PR #108](https://github.com/dowdiness/moondsp/pull/108)

### js_engine

js_engineではclosure-converted block bodiesの最適化を行った。
その後、opt-in bytecode prototypeを追加し、既存interpreterを残したままbytecode実行経路を育てる準備に入った。
関連PR: [js_engine PR #156](https://github.com/dowdiness/js_engine/pull/156)、[js_engine PR #157](https://github.com/dowdiness/js_engine/pull/157)

## 2026/5/29

### Canopy

Canopyでは前日に固めたCognitionまわりから少し離れて、repo全体の再利用性と整理を進めた。
agent reuse protocolのdocsを追加し、レビュー指摘を受けてAPI mapやPR template、package overviewの型まわりも直した。

MoonBitのidiom sweepとして、core / projection、lang/json、lang/lambda、editorまわりでguardやpattern matching、loop idiom、`ProjNode::id()` の使い方を整理した。
tree-editorのfile splitも入り、後続の変更で触る範囲を読みやすくした。
関連PR: [Canopy PR #381](https://github.com/dowdiness/canopy/pull/381)、[Canopy PR #382](https://github.com/dowdiness/canopy/pull/382)、[Canopy PR #383](https://github.com/dowdiness/canopy/pull/383)、[Canopy PR #385](https://github.com/dowdiness/canopy/pull/385)

大きな変更としては、editor内にあったephemeral presence subsystemを `dowdiness/canopy/ephemeral` へ切り出した。
その後、wire primitiveを汎用の `lib/byte-codec` へ抽出し、relayのwire codecも共有byte codecへ移した。
presenceやrelayがそれぞれ似たようなwire処理を持つのではなく、低レベルのbyte列変換を共通部品として扱う方向になった。
関連PR: [Canopy PR #387](https://github.com/dowdiness/canopy/pull/387)、[Canopy PR #388](https://github.com/dowdiness/canopy/pull/388)、[Canopy PR #390](https://github.com/dowdiness/canopy/pull/390)、[Canopy PR #391](https://github.com/dowdiness/canopy/pull/391)、[Canopy PR #392](https://github.com/dowdiness/canopy/pull/392)

Canvas側ではconnection drag中のpreview port compatibilityを追加した。
接続を引いている途中でもportの互換性を見ながらpreviewできるようになり、グラフ編集時の手触りを改善する変更になっている。
関連PR: [Canopy PR #394](https://github.com/dowdiness/canopy/pull/394)

### moondsp

moondspではLoom mini CST projectionのhelper利用を進めた。
projection identity helperとoptional-edit projection helperを使うようにし、Loom側に寄せたprojection APIでspecを保てるか確認している。
関連PR: [moondsp PR #109](https://github.com/dowdiness/moondsp/pull/109)、[moondsp PR #110](https://github.com/dowdiness/moondsp/pull/110)

### js_engine

js_engineではbytecode実行経路を広げた。
short-circuit operatorとcomma expressionのbytecode対応を追加し、sloppy arguments formal binding、double super initialization、async generator functionのname / length、destructuring rest parameterの扱いを順に修正した。
関連PR: [js_engine PR #158](https://github.com/dowdiness/js_engine/pull/158)、[js_engine PR #159](https://github.com/dowdiness/js_engine/pull/159)、[js_engine PR #160](https://github.com/dowdiness/js_engine/pull/160)、[js_engine PR #161](https://github.com/dowdiness/js_engine/pull/161)、[js_engine PR #162](https://github.com/dowdiness/js_engine/pull/162)、[js_engine PR #163](https://github.com/dowdiness/js_engine/pull/163)

bytecodeはまだopt-inの段階だが、式や関数境界の細かい仕様ケースを通しながら、interpreterとの差分を潰していく流れになっている。
