---
title: Canopy作業日誌
publish: true
tags:
  - projectional-editing
aliases:
  - Canopy
created: 2026-01-04T20:50:52+09:00
modified: 2026-05-19T00:00:00+09:00
---

# Canopy作業日誌

## 2026/5/18

incrのDatalogを使ってUI開発が出来ないかの実験した。
DataScriptみたいなことが出来るかもしれない。
https://github.com/tonsky/datascript
https://github.com/dowdiness/loom/pull/124

RabbitaとCodeMirrorの連携するグルーコードを改善した。
現在はグチャグチャと混沌としたコードになっていてバグが多く、エディターを触っていると途中で動かなくなってしまうことがあったのが直った。
https://claude.ai/share/ed68c575-c459-415d-bd19-b7965dd94e29

## 2026/5/19

### Canopy

canopyでのrabbita_codemirror（CodeMirrorバインディング）の作成を続けた。
途中でバグが見つかって既存のコードを改修する必要が生まれてそれの改修に時間が掛かった。

プログラミング言語の変数の入れ替えをする機能をincrを使って出来るように環境を整えた。

### moondsp

AudioBuffer APIの改修をした。`as_fixed_array`で内部の `fixed_array` に直接アクセスしないといけなかったのをAudioBufferの構造体から`all` / `any`などのメソッドを直接読めるようにした。

### トークンの使用量を減らす工夫

コーディングエージェントの使用するトークン量を減らすべく [BAML](https://boundaryml.com/) を導入した。
どれくらい効果があるのかまだ分からないので実際に使ってみて確かめる必要がある。

## 2026/5/20

### Canopy / Rabbita CodeMirror

[PR #303](https://github.com/dowdiness/canopy/pull/303)

Canopyの `examples/ideal` で、RabbitaとCodeMirrorのバインディング移行を進めた。
移行が完了するまで既存実装と新しいバインディングを共存させれるようにし、マウント手順と、二重DOM、フラグ読み取り、undo記録まわりの問題を潰した。

CodeMirrorのselectionやextensionなどをCodeMirror専用のAPIとして閉じ込めるずに、標準の[Selection](https://developer.mozilla.org/ja/docs/Web/API/Selection) / [Range](https://developer.mozilla.org/ja/docs/Web/API/Range) APIやCodeMirror本体のAPIを薄いJS FFIとして扱う方向に決定した。

### loom

Lambda exampleのAPIの整理をした

### incr

[PR #58](https://github.com/dowdiness/incr/pull/58)

vizalizationの機能を追加するために内部の整理をして `EventBroadcastPhaseHook` を追加した。
これ専用のpublic APIは増やさず、パフォーマンスを意識してRuntimeのコンストラクタに合わせてイベントをbufferしてから一気にbatch処理する、イベントリスナを使わない形にした。
benchmarkをとって数パーセントほど実行速度が向上していることを確認した。

### moondsp

AudioBufferのwrite-time validationの設計を進めた。
`new` / `filled` / `fill` / `set` を共通のサンプル検証・正規化パスへ通す方針にし、`adopt` はゼロコピー契約上、採用後に外部ハンドルから変更された値まではMoonBit側で検証できないことを明確にした。
-1~+1以外の値を正規化するようにした。

### js_engine

well-known symbolの所有権をrealm側へ移し、[PR#130](https://github.com/dowdiness/js_engine/pull/130)をマージした。`setup_builtins(env, output, symbols, ...)` を単体で呼ぶ場合も、渡された `SymbolState` でwell-known symbolsを割り当てるよう直した。

## 2026/5/21

### Canopy

Canopyの `examples/ideal` で、RabbitaとDOMイベントの境界を整理した。
[PR #312](https://github.com/dowdiness/canopy/pull/312) でRabbitaに依存しないDOM boundary helperを追加し、[PR #313](https://github.com/dowdiness/canopy/pull/313) から [PR #316](https://github.com/dowdiness/canopy/pull/316) でoverlay、sync、structure modeのhidden buttonという名前で設定していた命令的なトリガーをRabbitaのcustom event経由のイベントサブスクリプションに寄せた。

DOMイベントサブスクリプションの失敗をログに出す変更も入れたので、Rabbita側のイベント登録で問題が起きたときに原因を追いやすくなった。

### loom

Loomではincremental parser reuseまわりのTODOを進めた。
削除時の左隣CST reuseや、削除後にoffsetがずれたnodeを再利用するケースをテストで固定し、現在のinvariantはparser-ownedなtoken/subtree identityではなく、チェック済みのCST subtree reuseであることを確認した。

### incr

incrではpublic APIの命名とread helperの整理を続けた。
`read`まわりのpermissiveなhelperをrenameし、理想的なAPIへの移行計画を作った。

### js_engine

js_engineではwell-known symbol lookupの移行を進めた。
[PR #131](https://github.com/dowdiness/js_engine/pull/131) と [PR #132](https://github.com/dowdiness/js_engine/pull/132) で、runtimeやstdlibの残っていたno-argument symbol getter系を明示的なrealm-owned `WellKnownSymbols` accessへ移し、互換用のlegacy pathを削除した。

## 2026/5/22

### Canopy

CanopyでRabbita側のundo/redoショートカットもhidden button経由から外した。
[PR #318](https://github.com/dowdiness/canopy/pull/318) でCodeMirror側が `request-undo` / `request-redo` custom eventをdispatchし、Rabbita側がそれを `Undo` / `Redo` に変換する形になった。

その後、`examples/web` と `examples/ideal` に対する `tsc --noEmit` のCI jobを追加し、Inspectorにincr runtime snapshotを表示する作業も入った。

### incr

incrではtarget API facadeの作業が進んだ。
[PR #68](https://github.com/dowdiness/incr/pull/68) で理想的なAPIのfacadeを追加し、その後runtime read helperのdeprecation、input freshness facade、map relation facadeが続いた。
Canopy側から使うAPIを薄く整えながら、古いread helperに直接依存しない方向へ寄せた。

### js_engine

iterator cacheやprimitive wrapper prototypeの状態を `RealmState` に移した。
[PR #133](https://github.com/dowdiness/js_engine/pull/133) と [PR #134](https://github.com/dowdiness/js_engine/pull/134) がこの流れの中心で、factory/prototype系の状態をmodule globalからrealm-owned stateへ移す作業を続けた。

注: [Realms](https://tc39.es/ecma262/#sec-code-realms)とはEcmaScriptの仕様です。

## 2026/5/23

### Canopy

CanopyではInspectorまわりの作業を続けた。
InspectorにPatch panelを追加し、`view_op_log` / `view_patch` のguard、Op Log label formatの統一、`SourceMap::nodes_at_position` の範囲制約の修正を入れた。

### incr

incrではAPI migrationに向けたドキュメントとexamplesを増やした。
architectureやcookbook、API referenceの例を新しいAPI名に合わせて更新し、[Build Systems à la Carte](https://hackage.haskell.org/package/build)との関係づけもdocsに記録した。

### moondsp

moondspではLoomを使ったmini記法の検証を進めた。
`specs/loom-mini-cst` のgrammar parityに向けて、checked target API examplesやloom-mini-cstのdocsを更新し、Loom側のAPI driftをspecで拾える状態にした。

### js_engine

js_engineではprototype移行を続けた。
object function、Promise、WeakMap / WeakSet、Map / Set、Array prototypeなどのlookupやstorageを順に `RealmState` に寄せ、module globalに残っていたprototype参照を減らした。

## 2026/5/24

### Canopy / loom

[Loom #147](https://github.com/dowdiness/loom/issues/147) の移行をCanopy側まで進めた。
`text_change` と `moji` をCanopy配下ではなくLoom monorepo側のtop-level moduleへ移す方針を決めた。

実装としては [loom PR #149](https://github.com/dowdiness/loom/pull/149) で `text_change` / `moji` がLoom側へ移り、Canopy側では [PR #341](https://github.com/dowdiness/canopy/pull/341) で `./loom/text-change` と `./loom/moji` を参照するようにした。
Canopy内の `lib/text-change` と `lib/moji`、使われていなかった `valtio` submoduleも整理した。

### incr

incrは [v0.6.0](https://github.com/dowdiness/incr/pull/81) のreleaseを行った。
その後、CanopyやLoom側の利用に合わせて、ファイル名やskillの記述を新しいAPIの名前へ寄せた。

### moondsp

moondspではLoom mini CSTの改良を続けた。
[PR #75](https://github.com/dowdiness/moondsp/pull/75) でquickcheckを0.14.0へ上げ、[PR #76](https://github.com/dowdiness/moondsp/pull/76) でloom-mini-cstのgrammarを広げた。

この時点でもproduction parserはLoomへ切り替えておらず、Loomはまだspecと回帰テスト側で使う位置づけ。

### js_engine

RealmState移行をさらに進めた。
runtimeのMap / Set、boxed primitive、Array、WeakMap / WeakSet、ArrayBuffer storageなどを順に `RealmState` 側へ寄せ、CI workflowのNode.js runtime更新とdeprecatedな `moon install` 呼び出しの削除も行った。

## 2026/5/25

### Canopy

Lambda metadataをeditor/workspace/FFIの境界に通す変更を進めた。
`ffi/lambda` のrouting、workspace coordination、Editor側のmetadata受け渡し、typed workflow port handlerが追加され、Lambda exampleをCognition側の流れに接続する準備が進んだ。

### loom

Loomでは `Memo` から `Derived` への用語・API整理に合わせて、`examples/lambda` を更新した。
また、seamにdirect CST query helperを追加し、CST projection guideをdocsに追加した。

### moondsp

moondspではLoom mini CSTからprojection method IRを検証する作業を行った。
[PR #80](https://github.com/dowdiness/moondsp/pull/80) でprojection method IRをvalidateし、apply-editの自動テストやloop expression style guidanceも追加した。

### js_engine

js_engineではconstruct/call contextの明示化を進めた。
ArrayBufferを `RealmState` に移した後の流れとして、construction stateを明示的なcall contextへ移し、ambient interpreter context fallbackを削除した。

## 2026/5/26

### Canopy

CanopyではCognitionまわりの基盤を進めた。
workspace filesを追跡する [PR #357](https://github.com/dowdiness/canopy/pull/357)、minimal incremental reactive layer、context packing API、provider boundaryの計画とdocsを追加し、削除ファイルの依存関係を掃除する修正も入った。

同じ流れで、Lambda側はLoomの `LambdaAnalysis` attachmentを使う形へ寄せた。

### incr

incrではAPIの大きな整理を続けた。
safe incremental refactorとしてtypesとcorrectnessを整理し、pipeline traitsをdeprecateした。
expr formula APIの設計もdocsに残した。

### moondsp

moondspではLoom mini CSTのcoverageを広げた。
slow postfix projection、degrade / euclid projection、dollar stack parity、sub-notation postfix parity、callback method projectionなどを追加し、Loom miniがどこまでproduction miniの構文に追いつけるかを確認した。

### js_engine

js_engineではborrowed built-in realm routingを修正した。
[PR #153](https://github.com/dowdiness/js_engine/pull/153) の変更として、built-in realmの扱いを明示的なroutingへ寄せた。

## 2026/5/27

### Canopy

Canopyではprovider boundaryの設計を実装側へ進めた。
provider boundary domainを追加し、provider boundary planをretargetした。
前日までのrecompute cleanupやcontext packingの作業を受けて、Cognitionが外部providerへ渡す境界を整理している段階になった。

### incr

incrではPhase 3a facade migrationのdocsを入れ、evaluation strategyをkernelから切り出した。
[PR #94](https://github.com/dowdiness/incr/pull/94) では現在のincrのモデルをdocsにまとめている。

### moondsp

moondspではLoom mini CSTのknown edgeをcharacterizeした。
[PR #99](https://github.com/dowdiness/moondsp/pull/99) でmode-incompatibleなmini atomをrejectする挙動や、known edge follow-upの状態をdocsに残した。

### js_engine

js_engineではstartup benchmark stagingとbenchmark summary renderingの整理を行った。
[PR #154](https://github.com/dowdiness/js_engine/pull/154) でJS startup benchmarkの足場を追加し、[PR #155](https://github.com/dowdiness/js_engine/pull/155) でbenchmark dashboardの表示を整理した。

## 2026/5/28

### Canopy

Canopyではprovider planningとLambda semantic側の作業を続けた。
commit履歴ではprovider planningとlambda semanticに関する変更が入り、今日時点の未コミット差分では `lib/cognition/provider_boundary_store.mbt` と `lib/cognition/reactive.mbt` にprovider planning graphを接続し、cancellation、completion、retry classification、driver actionを `@incr` の内部状態として扱う方向の変更が進んでいる。

未コミット差分にはprovider cancellationのidempotency、driver shutdown時にpending requestを観測できること、file removalやbudgeted context変更後のstale completionを拒否するテストも追加されている。

### incr

incrではruntime evaluation event APIまわりを進めた。
internal runtime evaluation eventsを追加し、その後API surfaceを整理するrefactorが入った。

### moondsp

moondspではloom-mini-cstのprovenance matrix coverageを追加し、docsを更新した。
browser telemetry smoke testのflakiness低減も行われている。

### js_engine

js_engineではclosure関係のテスト最適化を行った。
前日のbenchmark stagingの流れに続き、startupやbenchmarkまわりの確認を進めている。
