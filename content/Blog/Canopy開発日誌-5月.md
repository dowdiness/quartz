---
title: Canopy開発日誌-5月
publish: true
tags: [blog, canopy, projectional-editing]
aliases: [Canopy作業日誌]
created: 2026-01-04T20:50:52+09:00
modified: 2026-06-11T04:43:46+09:00
---

# Canopy作業日誌

2026年5月18日から31日ごろまでの作業ログ。

中心はCanopyというprojectional editor（構造編集エディタ）の実験で、周辺ライブラリのloom、incr、moondsp、js_engineにも並行して手を入れている。Canopyでやりたいのは、テキストとしての編集とプログラムの構造を直接触る編集を同じワークスペースで扱えるようにすること。さらに、そのワークスペースの状態をAIへ渡せるコンテキストとして整理して、編集作業とAI支援をつなげることを目指している。

生の作業メモに近いので、細かいPRリンクもそのまま残してある。大まかな流れとしては、前半はRabbitaとCodeMirrorの接続、途中からInspectorやLoom連携、後半はCognitionというAI用ナレッジベース機能の土台づくり。終盤はlambdaのscope graphやgo-to-definition、typed spreadsheet demo、bytecode benchmarkなど、構造編集とその基盤を実際のUIや性能測定へつなぐ作業に広がっていった。incrについては、[Build Systems à la Carte](https://hackage.haskell.org/package/build)を読みながら自分のライブラリのAPIや評価モデルを整理していった時期でもある。

この記事に出てくる主なリポジトリ:

- [Canopy](https://github.com/dowdiness/canopy): projectional editor（構造編集エディタ）とAI用ナレッジベース機能まわりの実験をしている中心リポジトリ。
- [loom](https://github.com/dowdiness/loom): incremental parserやCST projectionを扱うライブラリ。
- [incr](https://github.com/dowdiness/incr): incremental computation用の小さなライブラリ。
- [moondsp](https://github.com/dowdiness/moondsp): MoonBitでDSPや音楽記述用DSLの実験をしているリポジトリ。
- [js_engine](https://github.com/dowdiness/js_engine): MoonBitでのJavaScriptエンジンの実装。

本文でよく出てくる言葉:

- Rabbita: Canopyで使っているUI / editor側の実験的な仕組み。
- CodeMirror: テキストエディタ部分に使っている既存のエディタライブラリ。
- Cognition: Canopy上でAIに渡すコンテキストやプロバイダとの境界を扱うための実験。
- FFI: MoonBit側のコードとJavaScript / DOM側のコードをつなぐ境界。
- provider boundary: AIプロバイダへ渡す入力、返ってきた結果、キャンセルやretryを追跡するための境界。

この期間の大きな流れ:

1. RabbitaとCodeMirrorの接続を安定させる。
2. DOMの隠しボタン経由だった操作を、イベント購読や明示的な境界に寄せる。
3. Inspectorやop logを整えて、内部状態を追いやすくする。
4. [Build Systems à la Carte](https://hackage.haskell.org/package/build)を参照しながら、incrのAPIと評価モデルを整理する。
5. Cognitionのワークスペース、コンテキストpacking、provider boundaryの土台を作る。
6. ephemeral presenceやbyte codecを切り出して、共有部品として扱えるようにする。
7. Lambdaのscope graphとgo-to-definitionを整え、Idealのscope annotationを同じ解決結果へ寄せる。
8. incrのtyped spreadsheet demoとjs_engineのbytecode benchmarkを育て、実験用UIや性能確認の入口を増やす。

## 2026/5/18

incrのDatalogを使ってUI開発ができないか実験した。[DataScript](https://github.com/tonsky/datascript)みたいなことができるかもしれない。
関連PR: [loom PR #124](https://github.com/dowdiness/loom/pull/124)

RabbitaとCodeMirrorをつなぐグルーコードも改善した。これまでは混沌としたコードでバグが多く、エディターを触っていると途中で動かなくなることがあったが、それが直った。
関連PR: [Canopy PR #293](https://github.com/dowdiness/canopy/pull/293)、[Canopy PR #296](https://github.com/dowdiness/canopy/pull/296)
https://claude.ai/share/ed68c575-c459-415d-bd19-b7965dd94e29

## 2026/5/19

### Canopy

rabbita_codemirror（CodeMirrorバインディング）の作成を続けた。途中で既存コードのバグが見つかり、その改修に時間を取られた。
関連PR: [Canopy PR #297](https://github.com/dowdiness/canopy/pull/297)、[Canopy PR #299](https://github.com/dowdiness/canopy/pull/299)、[Canopy PR #300](https://github.com/dowdiness/canopy/pull/300)、[Canopy PR #301](https://github.com/dowdiness/canopy/pull/301)、[Canopy PR #302](https://github.com/dowdiness/canopy/pull/302)

また、プログラミング言語の変数を入れ替える機能をincrで実現できるよう環境を整えた。
関連PR: [loom PR #126](https://github.com/dowdiness/loom/pull/126)、[loom PR #129](https://github.com/dowdiness/loom/pull/129)

### moondsp

AudioBuffer APIを改修した。これまで`as_fixed_array`で内部の`fixed_array`に直接アクセスする必要があったのを、AudioBufferの構造体から`all` / `any`などのメソッドを直接呼べるようにした。
関連PR: [moondsp PR #60](https://github.com/dowdiness/moondsp/pull/60)、[moondsp PR #62](https://github.com/dowdiness/moondsp/pull/62)

### トークンの使用量を減らす工夫

コーディングエージェントの使うトークン量を減らすため、[BAML](https://boundaryml.com/)を導入した。効果のほどはまだ分からないので、実際に使って確かめるつもりだ。

## 2026/5/20

### Canopy / Rabbita CodeMirror

Canopyの`examples/ideal`で、RabbitaとCodeMirrorのバインディング移行を進めた。移行が終わるまで既存実装と新しいバインディングを共存させられるようにして、マウント手順や二重DOM、フラグ読み取り、undo記録まわりの問題を潰した。
関連PR: [Canopy PR #303](https://github.com/dowdiness/canopy/pull/303)、[Canopy PR #305](https://github.com/dowdiness/canopy/pull/305)、[Canopy PR #306](https://github.com/dowdiness/canopy/pull/306)、[Canopy PR #307](https://github.com/dowdiness/canopy/pull/307)

selectionやextensionをCodeMirror専用のAPIに閉じ込めるのではなく、標準の[Selection](https://developer.mozilla.org/ja/docs/Web/API/Selection) / [Range](https://developer.mozilla.org/ja/docs/Web/API/Range) APIやCodeMirror本体のAPIを薄いJS FFIとして扱う方針に決めた。

### loom

Lambda exampleのAPIを整理した。
関連PR: [loom PR #131](https://github.com/dowdiness/loom/pull/131)、[loom PR #132](https://github.com/dowdiness/loom/pull/132)

### incr

可視化機能の追加に向けて内部を整理し、`EventBroadcastPhaseHook`を追加した。専用のpublic APIは増やしていない。パフォーマンスを意識して、イベントリスナではなくRuntimeのコンストラクタに合わせてイベントをbufferしてから一気にbatch処理する形にした。ベンチマークでは実行速度が数パーセント向上していた。
関連PR: [incr PR #58](https://github.com/dowdiness/incr/pull/58)、[incr PR #59](https://github.com/dowdiness/incr/pull/59)、[incr PR #60](https://github.com/dowdiness/incr/pull/60)

### moondsp

AudioBufferのwrite-time validationの設計を進めた。`new` / `filled` / `fill` / `set`は共通のサンプル検証・正規化パスを通す方針にして、-1〜+1の範囲外の値は正規化するようにした。`adopt`については、ゼロコピー契約上、採用後に外部ハンドルから変更された値まではMoonBit側で検証できないことを明確にした。
関連PR: [moondsp PR #63](https://github.com/dowdiness/moondsp/pull/63)

### js_engine

well-known symbolの所有権をrealm側へ移し、[js_engine PR #130](https://github.com/dowdiness/js_engine/pull/130)をマージした。`setup_builtins(env, output, symbols, ...)`を単体で呼ぶ場合も、渡された`SymbolState`でwell-known symbolを割り当てるように直した。

## 2026/5/21

### Canopy

`examples/ideal`でRabbitaとDOMイベントの境界を整理した。[Canopy PR #312](https://github.com/dowdiness/canopy/pull/312)でRabbitaに依存しないDOM boundary helperを追加し、[Canopy PR #313](https://github.com/dowdiness/canopy/pull/313)から[Canopy PR #316](https://github.com/dowdiness/canopy/pull/316)では、overlay・sync・structure modeのhidden buttonとして実装していた命令的なトリガーを、Rabbitaのcustom event経由のイベント購読に置き換えた。

イベント購読の失敗をログに出す変更も入れたので、Rabbita側のイベント登録で問題が起きたときに原因を追いやすくなった。UI操作をDOMの隠しボタンに依存させるより、Rabbita側のイベント購読として扱うほうが、後から読んだときに責務の境界が分かりやすい。

### loom

incremental parser reuseまわりのTODOを進めた。削除時に左隣のCSTを再利用するケースや、削除でoffsetがずれたnodeを再利用するケースをテストで固定し、現在のinvariantがparser-ownedなtoken / subtree identityではなく、チェック済みのCST subtree reuseであることを確認した。
関連PR: [loom PR #134](https://github.com/dowdiness/loom/pull/134)、[loom PR #135](https://github.com/dowdiness/loom/pull/135)、[loom PR #136](https://github.com/dowdiness/loom/pull/136)

### incr

public APIの命名とread helperの整理を続けた。`read`まわりのpermissiveなhelperをリネームし、理想形のAPIへ移行する計画を作った。
関連PR: [incr PR #61](https://github.com/dowdiness/incr/pull/61)、[incr PR #62](https://github.com/dowdiness/incr/pull/62)、[incr PR #63](https://github.com/dowdiness/incr/pull/63)

### js_engine

well-known symbol lookupの移行を進めた。[js_engine PR #131](https://github.com/dowdiness/js_engine/pull/131)と[js_engine PR #132](https://github.com/dowdiness/js_engine/pull/132)で、runtimeやstdlibに残っていた引数なしのsymbol getterを明示的なrealm-owned `WellKnownSymbols`アクセスへ移し、互換用のlegacy pathを削除した。

## 2026/5/22

### Canopy

Rabbita側のundo / redoショートカットもhidden button経由から外した。[Canopy PR #318](https://github.com/dowdiness/canopy/pull/318)で、CodeMirror側が`request-undo` / `request-redo`のcustom eventをdispatchし、Rabbita側がそれを`Undo` / `Redo`に変換する形になった。前日から続けていたhidden buttonトリガーの削除は、これで一段落。

その後、`examples/web`と`examples/ideal`への`tsc --noEmit` CI jobの追加と、Inspectorにincr runtime snapshotを表示する作業も入れた。
関連PR: [Canopy PR #320](https://github.com/dowdiness/canopy/pull/320)、[Canopy PR #321](https://github.com/dowdiness/canopy/pull/321)

### incr

target API facadeの作業を進めた。[incr PR #68](https://github.com/dowdiness/incr/pull/68)で理想形APIのfacadeを追加し、runtime read helperのdeprecation、input freshness facade、map relation facadeと続けた。Canopy側から使うAPIを薄く整えつつ、古いread helperへの直接依存を減らしている。
関連PR: [incr PR #69](https://github.com/dowdiness/incr/pull/69)、[incr PR #70](https://github.com/dowdiness/incr/pull/70)、[incr PR #71](https://github.com/dowdiness/incr/pull/71)、[incr PR #72](https://github.com/dowdiness/incr/pull/72)

### js_engine

iterator cacheやprimitive wrapper prototypeの状態を`RealmState`へ移した。中心は[js_engine PR #133](https://github.com/dowdiness/js_engine/pull/133)と[js_engine PR #134](https://github.com/dowdiness/js_engine/pull/134)で、factory / prototype系の状態をmodule globalからrealm-owned stateへ移す作業を続けている。

注: [Realm](https://tc39.es/ecma262/#sec-code-realms)はECMAScript仕様の概念。

## 2026/5/23

### Canopy

Inspectorまわりの作業を続けた。Patch panelを追加し、`view_op_log` / `view_patch`のguard、Op Logのlabel format統一、`SourceMap::nodes_at_position`の範囲制約の修正を入れた。Idealを触りながら内部状態を確認する道具が増えて、op logやpatchから原因を追いやすくなった。
関連PR: [Canopy PR #323](https://github.com/dowdiness/canopy/pull/323)、[Canopy PR #324](https://github.com/dowdiness/canopy/pull/324)、[Canopy PR #327](https://github.com/dowdiness/canopy/pull/327)、[Canopy PR #329](https://github.com/dowdiness/canopy/pull/329)

### incr

API migrationに向けてドキュメントとexampleを増やした。architecture、cookbook、API referenceの例を新しいAPI名に合わせて更新し、[Build Systems à la Carte](https://hackage.haskell.org/package/build)を読みながら、自分の実装がどの評価戦略・依存関係モデルに近いのかをdocsに記録した。
関連PR: [incr PR #73](https://github.com/dowdiness/incr/pull/73)、[incr PR #74](https://github.com/dowdiness/incr/pull/74)、[incr PR #75](https://github.com/dowdiness/incr/pull/75)、[incr PR #76](https://github.com/dowdiness/incr/pull/76)、[incr PR #77](https://github.com/dowdiness/incr/pull/77)、[incr PR #78](https://github.com/dowdiness/incr/pull/78)、[incr PR #79](https://github.com/dowdiness/incr/pull/79)

### moondsp

Loomを使ったmini記法の検証を進めた。`specs/loom-mini-cst`のgrammar parityに向けて、checked target API examplesとloom-mini-cstのdocsを更新し、Loom側のAPI driftをspecで検出できる状態にした。production parserをすぐ切り替えるのではなく、まずspec側でLoomの挙動を固定していく方針だ。

### js_engine

prototype移行を続けた。object function、Promise、WeakMap / WeakSet、Map / Set、Array prototypeなどのlookupやstorageを順に`RealmState`へ寄せ、module globalに残っていたprototype参照を減らした。
関連PR: [js_engine PR #135](https://github.com/dowdiness/js_engine/pull/135)、[js_engine PR #136](https://github.com/dowdiness/js_engine/pull/136)、[js_engine PR #137](https://github.com/dowdiness/js_engine/pull/137)、[js_engine PR #138](https://github.com/dowdiness/js_engine/pull/138)、[js_engine PR #139](https://github.com/dowdiness/js_engine/pull/139)

## 2026/5/24

### Canopy / loom

[Loom issue #147](https://github.com/dowdiness/loom/issues/147)の移行をCanopy側まで進めた。`text_change`と`moji`はCanopy配下ではなくLoom monorepoのtop-level moduleに置く方針にして、[loom PR #149](https://github.com/dowdiness/loom/pull/149)で両者をLoom側へ移し、Canopy側は[Canopy PR #341](https://github.com/dowdiness/canopy/pull/341)で`./loom/text-change`と`./loom/moji`を参照するようにした。

Canopy内の`lib/text-change`と`lib/moji`、使われていなかった`valtio` submoduleも整理した。Loomを単体でビルドしやすくするための移行で、Canopy側に置かれていた共通部品をLoomの責任範囲へ戻した形だ。

### incr

[incr PR #81](https://github.com/dowdiness/incr/pull/81)でv0.6.0をリリースした。その後、CanopyやLoom側での利用に合わせて、ファイル名やskillの記述を新しいAPI名に揃えた。
関連PR: [incr PR #82](https://github.com/dowdiness/incr/pull/82)、[incr PR #83](https://github.com/dowdiness/incr/pull/83)

### moondsp

Loom mini CSTの改良を続けた。[moondsp PR #75](https://github.com/dowdiness/moondsp/pull/75)でquickcheckを0.14.0へ上げ、[moondsp PR #76](https://github.com/dowdiness/moondsp/pull/76)でloom-mini-cstのgrammarを広げた。この時点でもproduction parserはLoomへ切り替えておらず、Loomはspecと回帰テストで使う位置づけのままだ。
関連PR: [moondsp PR #71](https://github.com/dowdiness/moondsp/pull/71)、[moondsp PR #73](https://github.com/dowdiness/moondsp/pull/73)、[moondsp PR #74](https://github.com/dowdiness/moondsp/pull/74)、[moondsp PR #79](https://github.com/dowdiness/moondsp/pull/79)

### js_engine

`RealmState`移行をさらに進めた。runtimeのMap / Set、boxed primitive、Array、WeakMap / WeakSet、ArrayBuffer storageなどを順に`RealmState`側へ寄せ、CI workflowのNode.js更新とdeprecatedな`moon install`呼び出しの削除も行った。
関連PR: [js_engine PR #140](https://github.com/dowdiness/js_engine/pull/140)、[js_engine PR #142](https://github.com/dowdiness/js_engine/pull/142)、[js_engine PR #143](https://github.com/dowdiness/js_engine/pull/143)、[js_engine PR #144](https://github.com/dowdiness/js_engine/pull/144)、[js_engine PR #146](https://github.com/dowdiness/js_engine/pull/146)、[js_engine PR #147](https://github.com/dowdiness/js_engine/pull/147)、[js_engine PR #148](https://github.com/dowdiness/js_engine/pull/148)、[js_engine PR #149](https://github.com/dowdiness/js_engine/pull/149)、[js_engine PR #151](https://github.com/dowdiness/js_engine/pull/151)

## 2026/5/25

### Canopy

Lambda metadataをeditor・ワークスペース・FFIの境界に通す変更を進めた。`ffi/lambda`のrouting、ワークスペースのcoordination、Editor側のmetadata受け渡し、typed workflow port handlerを追加して、Lambda exampleをCognition側へ接続する準備が整ってきた。Lambda exampleを単なるサンプルではなく、ワークスペースやCognitionの実験台として使えるようにするための作業だ。
関連PR: [Canopy PR #345](https://github.com/dowdiness/canopy/pull/345)、[Canopy PR #347](https://github.com/dowdiness/canopy/pull/347)、[Canopy PR #348](https://github.com/dowdiness/canopy/pull/348)、[Canopy PR #349](https://github.com/dowdiness/canopy/pull/349)、[Canopy PR #350](https://github.com/dowdiness/canopy/pull/350)

### loom

`Memo`から`Derived`への用語・API整理に合わせて`examples/lambda`を更新した。seamへのdirect CST query helperと、docsへのCST projection guideも追加した。
関連PR: [loom PR #152](https://github.com/dowdiness/loom/pull/152)、[loom PR #154](https://github.com/dowdiness/loom/pull/154)、[loom PR #155](https://github.com/dowdiness/loom/pull/155)、[loom PR #156](https://github.com/dowdiness/loom/pull/156)

### moondsp

Loom mini CSTからprojection method IRを検証した。[moondsp PR #80](https://github.com/dowdiness/moondsp/pull/80)でprojection method IRをvalidateし、apply-editの自動テストやloop expressionのstyle guidanceも追加した。
関連PR: [moondsp PR #81](https://github.com/dowdiness/moondsp/pull/81)、[moondsp PR #83](https://github.com/dowdiness/moondsp/pull/83)、[moondsp PR #84](https://github.com/dowdiness/moondsp/pull/84)、[moondsp PR #85](https://github.com/dowdiness/moondsp/pull/85)、[moondsp PR #87](https://github.com/dowdiness/moondsp/pull/87)

### js_engine

construct / callコンテキストの明示化を進めた。ArrayBufferの`RealmState`移行に続けて、construction stateを明示的なcallコンテキストへ移し、ambient interpreter contextへのfallbackを削除した。
関連PR: [js_engine PR #152](https://github.com/dowdiness/js_engine/pull/152)

## 2026/5/26

### Canopy

Cognitionの基盤を進めた。ワークスペースのfileを追跡する[Canopy PR #357](https://github.com/dowdiness/canopy/pull/357)に加えて、minimalなincremental reactive layer、コンテキストpacking API、provider boundaryの計画とdocsを追加し、削除済みファイルの依存関係を掃除する修正も入れた。
関連PR: [Canopy PR #355](https://github.com/dowdiness/canopy/pull/355)、[Canopy PR #358](https://github.com/dowdiness/canopy/pull/358)、[Canopy PR #359](https://github.com/dowdiness/canopy/pull/359)、[Canopy PR #360](https://github.com/dowdiness/canopy/pull/360)、[Canopy PR #363](https://github.com/dowdiness/canopy/pull/363)、[Canopy PR #364](https://github.com/dowdiness/canopy/pull/364)

あわせて、Lambda側はLoomの`LambdaAnalysis` attachmentを使う形に寄せた。Cognitionが参照するファイル・依存関係・コンテキストを明示的に扱えるようにして、後続のプロバイダ連携へ進む土台を作っている。
関連PR: [Canopy PR #362](https://github.com/dowdiness/canopy/pull/362)

### incr

APIの大きな整理を続けた。safe incremental refactorとしてtypesとcorrectnessを整理し、pipeline traitsをdeprecated扱いにした。expr formula APIの設計もdocsに残している。
関連PR: [incr PR #87](https://github.com/dowdiness/incr/pull/87)、[incr PR #89](https://github.com/dowdiness/incr/pull/89)

### moondsp

Loom mini CSTのcoverageを広げた。slow postfix projection、degrade / euclid projection、dollar stack parity、sub-notation postfix parity、callback method projectionなどを追加し、Loom miniがproduction miniの構文にどこまで追いつけるかを確認した。
関連PR: [moondsp PR #88](https://github.com/dowdiness/moondsp/pull/88)、[moondsp PR #89](https://github.com/dowdiness/moondsp/pull/89)、[moondsp PR #90](https://github.com/dowdiness/moondsp/pull/90)、[moondsp PR #91](https://github.com/dowdiness/moondsp/pull/91)、[moondsp PR #92](https://github.com/dowdiness/moondsp/pull/92)、[moondsp PR #93](https://github.com/dowdiness/moondsp/pull/93)、[moondsp PR #94](https://github.com/dowdiness/moondsp/pull/94)、[moondsp PR #95](https://github.com/dowdiness/moondsp/pull/95)

### js_engine

borrowed built-in realmのroutingを修正した。[js_engine PR #153](https://github.com/dowdiness/js_engine/pull/153)で、built-in realmの扱いを明示的なroutingに寄せている。

## 2026/5/27

### Canopy

provider boundaryの設計を実装に進めた。provider boundary domainを追加し、provider boundary planをretargetした。前日までのrecompute cleanupやコンテキストpackingを受けて、Cognitionが外部プロバイダへ渡す境界を整理する段階に入っている。プロバイダの結果をそのまま受け入れるのではなく、どの入力とコンテキストに対する結果なのかを追えるようにしておく必要があるからだ。
関連PR: [Canopy PR #365](https://github.com/dowdiness/canopy/pull/365)

### incr

Phase 3a facade migrationのdocsを入れ、evaluation strategyをkernelから切り出した。[incr PR #94](https://github.com/dowdiness/incr/pull/94)では現在のincrのモデルをdocsにまとめている。
関連PR: [incr PR #90](https://github.com/dowdiness/incr/pull/90)、[incr PR #91](https://github.com/dowdiness/incr/pull/91)、[incr PR #92](https://github.com/dowdiness/incr/pull/92)、[incr PR #93](https://github.com/dowdiness/incr/pull/93)、[incr PR #96](https://github.com/dowdiness/incr/pull/96)

### moondsp

Loom mini CSTのknown edgeをcharacterizeした。[moondsp PR #99](https://github.com/dowdiness/moondsp/pull/99)で、mode-incompatibleなmini atomをrejectする挙動や、known edgeのfollow-up状況をdocsに残した。
関連PR: [moondsp PR #96](https://github.com/dowdiness/moondsp/pull/96)、[moondsp PR #97](https://github.com/dowdiness/moondsp/pull/97)、[moondsp PR #100](https://github.com/dowdiness/moondsp/pull/100)、[moondsp PR #102](https://github.com/dowdiness/moondsp/pull/102)

### js_engine

startup benchmarkのstagingとbenchmark summaryの表示を整理した。[js_engine PR #154](https://github.com/dowdiness/js_engine/pull/154)でJS startup benchmarkの足場を追加し、[js_engine PR #155](https://github.com/dowdiness/js_engine/pull/155)でbenchmark dashboardの表示を整えた。

## 2026/5/28

### Canopy

provider planningとLambda semantic側の作業を続けた。provider planningとlambda semantic overlayを追加し、ワークスペースmemoのsmoke test、memo lifecycle API、reactive provider boundary driverまで進んだ。`lib/cognition/provider_boundary_store.mbt`と`lib/cognition/reactive.mbt`にprovider planning graphを接続し、cancellation・completion・retry classification・driver actionを`@incr`の内部状態として扱う方向が固まってきた。
関連PR: [Canopy PR #367](https://github.com/dowdiness/canopy/pull/367)、[Canopy PR #368](https://github.com/dowdiness/canopy/pull/368)、[Canopy PR #372](https://github.com/dowdiness/canopy/pull/372)、[Canopy PR #379](https://github.com/dowdiness/canopy/pull/379)

テストも増やした。provider cancellationのidempotency、driver shutdown時にpending requestを観測できること、file removalやbudgeted contextの変更後にstaleなcompletionを拒否すること。プロバイダの応答が遅れて返ってきたときに、古いコンテキストの結果を現在の状態へ混ぜないための整理だ。

Lambda・JSON・MarkdownのFFI read accessorもcoordinator経由のprotected readへ寄せた。ワークスペースの更新中に外側から半端な状態を読まれないようにする変更で、プロバイダ連携を進める前の境界固めにあたる。
関連PR: [Canopy PR #370](https://github.com/dowdiness/canopy/pull/370)、[Canopy PR #374](https://github.com/dowdiness/canopy/pull/374)、[Canopy PR #375](https://github.com/dowdiness/canopy/pull/375)、[Canopy PR #376](https://github.com/dowdiness/canopy/pull/376)、[Canopy PR #377](https://github.com/dowdiness/canopy/pull/377)、[Canopy PR #378](https://github.com/dowdiness/canopy/pull/378)

### incr

runtime evaluation event APIまわりを進めた。internal runtime evaluation eventsとevaluation strategy bundleを追加し、static derived fast pathのbenchmarkも取った。honest read-error ownershipの設計をdocsに残し、`Derived::fallible` / `DerivedMap::fallible`を追加した。
関連PR: [incr PR #95](https://github.com/dowdiness/incr/pull/95)、[incr PR #97](https://github.com/dowdiness/incr/pull/97)、[incr PR #98](https://github.com/dowdiness/incr/pull/98)

### moondsp

loom-mini-cstのprovenance matrix coverageとcontrol method projection parityを追加した。Loom移行に向けて、upstreamへ要求する挙動とrecovery stateをdocsに分け、回復処理のevidenceも増やした。
関連PR: [moondsp PR #101](https://github.com/dowdiness/moondsp/pull/101)、[moondsp PR #104](https://github.com/dowdiness/moondsp/pull/104)、[moondsp PR #106](https://github.com/dowdiness/moondsp/pull/106)、[moondsp PR #107](https://github.com/dowdiness/moondsp/pull/107)、[moondsp PR #108](https://github.com/dowdiness/moondsp/pull/108)

### js_engine

closure-converted block bodiesを最適化した。続けてopt-inのbytecode prototypeを追加し、既存interpreterを残したままbytecode実行経路を育てる準備に入った。
関連PR: [js_engine PR #156](https://github.com/dowdiness/js_engine/pull/156)、[js_engine PR #157](https://github.com/dowdiness/js_engine/pull/157)

## 2026/5/29

### Canopy

前日まで進めていたCognitionから少し離れて、repo全体の再利用性と整理に手を入れた。agent reuse protocolのdocsを追加し、レビュー指摘を受けてAPI map、PR template、package overviewの型まわりも直した。

MoonBitのidiom sweepとして、core / projection、lang/json、lang/lambda、editorまわりでguard、pattern matching、loop idiom、`ProjNode::id()`の使い方を整理した。tree-editorのfile splitも入れて、後続の変更で触る範囲を読みやすくした。
関連PR: [Canopy PR #381](https://github.com/dowdiness/canopy/pull/381)、[Canopy PR #382](https://github.com/dowdiness/canopy/pull/382)、[Canopy PR #383](https://github.com/dowdiness/canopy/pull/383)、[Canopy PR #385](https://github.com/dowdiness/canopy/pull/385)

大きめの変更としては、editor内にあったephemeral presence subsystemを`dowdiness/canopy/ephemeral`へ切り出した。さらにwire primitiveを汎用の`lib/byte-codec`として抽出し、relayのwire codecもそこへ移した。presenceとrelayがそれぞれ似たようなwire処理を持つのではなく、低レベルのbyte列変換を共通部品として扱う形になった。
関連PR: [Canopy PR #387](https://github.com/dowdiness/canopy/pull/387)、[Canopy PR #388](https://github.com/dowdiness/canopy/pull/388)、[Canopy PR #390](https://github.com/dowdiness/canopy/pull/390)、[Canopy PR #391](https://github.com/dowdiness/canopy/pull/391)、[Canopy PR #392](https://github.com/dowdiness/canopy/pull/392)

Canvas側では、connection drag中のpreview port compatibilityを追加した。接続を引いている途中でもportの互換性を確認しながらpreviewできるようになり、グラフ編集の手触りが良くなった。
関連PR: [Canopy PR #394](https://github.com/dowdiness/canopy/pull/394)

### moondsp

Loom mini CST projectionでのhelper利用を進めた。projection identity helperとoptional-edit projection helperを使うようにし、Loom側に寄せたprojection APIでspecを保てるか確認している。
関連PR: [moondsp PR #109](https://github.com/dowdiness/moondsp/pull/109)、[moondsp PR #110](https://github.com/dowdiness/moondsp/pull/110)

### js_engine

bytecode実行経路を広げた。short-circuit operatorとcomma expressionのbytecode対応を追加し、sloppy arguments formal binding、double super initialization、async generator functionのname / length、destructuring rest parameterの扱いを順に修正した。
関連PR: [js_engine PR #158](https://github.com/dowdiness/js_engine/pull/158)、[js_engine PR #159](https://github.com/dowdiness/js_engine/pull/159)、[js_engine PR #160](https://github.com/dowdiness/js_engine/pull/160)、[js_engine PR #161](https://github.com/dowdiness/js_engine/pull/161)、[js_engine PR #162](https://github.com/dowdiness/js_engine/pull/162)、[js_engine PR #163](https://github.com/dowdiness/js_engine/pull/163)

bytecodeはまだopt-inの段階だが、式や関数境界の細かい仕様ケースを通しながらinterpreterとの差分を潰している。

## 2026/5/30

### Canopy

Lambdaのscope graphを本格的に実装へ落とし始めた。NodeIdをキーにしたbinding indexを追加し、renameのbinder lookupを古い`resolve_binder`から`@scope.declaration`へ移した。残っていた呼び出し側も`@scope.declaration`へ移し、module binderの`Decl.node_id`に関するproduction contractとcross-pipeline resolution equivalenceをテストで固定した。
関連PR: [Canopy PR #396](https://github.com/dowdiness/canopy/pull/396)、[Canopy PR #397](https://github.com/dowdiness/canopy/pull/397)、[Canopy PR #398](https://github.com/dowdiness/canopy/pull/398)、[Canopy PR #399](https://github.com/dowdiness/canopy/pull/399)、[Canopy PR #400](https://github.com/dowdiness/canopy/pull/400)、[Canopy PR #401](https://github.com/dowdiness/canopy/pull/401)、[Canopy PR #402](https://github.com/dowdiness/canopy/pull/402)

cross-pipelineのPBTを通す中で、module binderの`node_id`が実際のprojection nodeを指していない問題もはっきりした。go-to-definitionを作るときに邪魔になるので、既存のSourceMap token spanからbinder locationを引けるようにするOption Dの設計に整理した。
関連PR: [Canopy PR #403](https://github.com/dowdiness/canopy/pull/403)

### loom

Canopy側のscope graphとprojection identityを支える変更を進めた。CST tokenをsource spanとして保持し、parser-owned reuseのrebaseを取り戻し、source-span reuse APIを固めた。さらに`ProjectionIdentityTracker`を追加して、projection identityを単発のhelperではなく、編集列をまたいで追跡できる部品にした。
関連PR: [loom PR #188](https://github.com/dowdiness/loom/pull/188)、[loom PR #189](https://github.com/dowdiness/loom/pull/189)、[loom PR #190](https://github.com/dowdiness/loom/pull/190)、[loom PR #191](https://github.com/dowdiness/loom/pull/191)、[loom PR #192](https://github.com/dowdiness/loom/pull/192)

### incr

typed spreadsheet demoを実際に触れるUIへ育てた。セル編集に始まり、50x50のfullscreen sheet、inline edit、trace / evidence overlay、night themeを備えたRabbita demoまで広げた。式の評価はMoonBit側に残し、Rabbitaは表示と操作の層に留める方針のままだ。
関連PR: [incr PR #117](https://github.com/dowdiness/incr/pull/117)、[incr PR #118](https://github.com/dowdiness/incr/pull/118)

### moondsp

Loom側で増えたprojection identity helperをspecへ取り込んだ。Loom mini CST projectionをproduction parserへすぐ置き換えるのではなく、まずspecのprojection identityを上流APIに寄せて、移行時の前提を揃えている。
関連PR: [moondsp PR #111](https://github.com/dowdiness/moondsp/pull/111)

### js_engine

opt-in bytecode / VM prototypeのcoverageを大きく広げた。演算子、property access、call / construct、destructuring、eval、`super`などの実行経路を既存のruntime helperに寄せながらbytecodeへ通し、未対応の構文は明示的なunsupported診断で落とすようにした。その後、bytecode performance microbenchmarkの追加と、不要なarguments object setupを避ける最適化も入れた。
関連PR: [js_engine PR #164](https://github.com/dowdiness/js_engine/pull/164)、[js_engine PR #171](https://github.com/dowdiness/js_engine/pull/171)、[js_engine PR #172](https://github.com/dowdiness/js_engine/pull/172)

## 2026/5/31

### Canopy

前日に設計したscope graphのbinder locationを実装した。`@scope.binder_span`と`@scope.go_to_definition`を追加し、`references`を`DeclId`キーに移して、module binderのsynthetic `node_id`に依存しない形にした。incrementalとfull pipelineの差分テストも追加し、FlatProj reuseや`@incr` memo stackを通しても同じ解決結果になることを確認している。
関連PR: [Canopy PR #404](https://github.com/dowdiness/canopy/pull/404)、[Canopy PR #405](https://github.com/dowdiness/canopy/pull/405)、[Canopy PR #406](https://github.com/dowdiness/canopy/pull/406)、[Canopy PR #407](https://github.com/dowdiness/canopy/pull/407)、[Canopy PR #408](https://github.com/dowdiness/canopy/pull/408)、[Canopy PR #411](https://github.com/dowdiness/canopy/pull/411)

incrのread channelが`ReadError`を返すようになったのに合わせて、coordinator側でもReadErrorを伝播するようにした。scope graph側ではmodule editのreferenceをidentityベースにし、edit capture checkやIdealのscope annotationもcanonicalな`@scope` graphから導出する形へ寄せた。UIのhighlightとscope graphの解決結果が別々のresolverを持つ状態から、これで一歩抜け出せた。
関連PR: [Canopy PR #409](https://github.com/dowdiness/canopy/pull/409)、[Canopy PR #410](https://github.com/dowdiness/canopy/pull/410)、[Canopy PR #412](https://github.com/dowdiness/canopy/pull/412)、[Canopy PR #420](https://github.com/dowdiness/canopy/pull/420)、[Canopy PR #426](https://github.com/dowdiness/canopy/pull/426)、[Canopy PR #427](https://github.com/dowdiness/canopy/pull/427)

docs側では、repository responsibility map、GUI layer integration report、module一覧と`.gitmodules` / `moon.mod.json`の整合性を整理した。Structure modeのfallback documentもschema validに直している。
関連PR: [Canopy PR #421](https://github.com/dowdiness/canopy/pull/421)、[Canopy PR #431](https://github.com/dowdiness/canopy/pull/431)、[Canopy PR #432](https://github.com/dowdiness/canopy/pull/432)、[Canopy PR #433](https://github.com/dowdiness/canopy/pull/433)

### loom

前日の`ProjectionIdentityTracker`を、失敗したeditやmalformed damageをまたいでcomposeできるようにした。また、incr側のtyped spreadsheet demo、ReadError、accumulator ReadErrorに合わせてsubmoduleを更新し、Canopyやmoondspが同じ基盤を参照できるようにした。
関連PR: [loom PR #197](https://github.com/dowdiness/loom/pull/197)、[loom PR #198](https://github.com/dowdiness/loom/pull/198)、[loom PR #199](https://github.com/dowdiness/loom/pull/199)、[loom PR #200](https://github.com/dowdiness/loom/pull/200)、[loom PR #201](https://github.com/dowdiness/loom/pull/201)、[loom PR #204](https://github.com/dowdiness/loom/pull/204)

### incr

honest read-error ownershipのTier 2として、public read channelを`CycleError`から`ReadError`へ広げた。これで、disposeされたcellの読み取りをcatchできないabortではなく`Err(Disposed(_))`として扱える。`Derived::fallible`のrecipeとReachableDerivedのADRもdocsに追加し、typed spreadsheet demo側ではGC rootingとper-edit evidence snapshotの上限も直した。
関連PR: [incr PR #119](https://github.com/dowdiness/incr/pull/119)、[incr PR #120](https://github.com/dowdiness/incr/pull/120)、[incr PR #125](https://github.com/dowdiness/incr/pull/125)、[incr PR #126](https://github.com/dowdiness/incr/pull/126)、[incr PR #127](https://github.com/dowdiness/incr/pull/127)、[incr PR #132](https://github.com/dowdiness/incr/pull/132)、[incr PR #133](https://github.com/dowdiness/incr/pull/133)、[incr PR #134](https://github.com/dowdiness/incr/pull/134)

その後、static `Derived`のprivate pathを通常経路へ昇格させ、disposed cell idのdependent guard、Datalog relationのnet change publish、accumulator readの`ReadError`対応も入れた。typed spreadsheet demoにはCloudflare Pagesへのdeploy workflowを追加し、Node 24 actionsにも合わせた。
関連PR: [incr PR #135](https://github.com/dowdiness/incr/pull/135)、[incr PR #136](https://github.com/dowdiness/incr/pull/136)、[incr PR #137](https://github.com/dowdiness/incr/pull/137)、[incr PR #141](https://github.com/dowdiness/incr/pull/141)、[incr PR #142](https://github.com/dowdiness/incr/pull/142)、[incr PR #144](https://github.com/dowdiness/incr/pull/144)、[incr PR #145](https://github.com/dowdiness/incr/pull/145)

### moondsp

Loomのtracker edit compositionをspec側で消費し、Loom promotion notesも現状に合わせて更新した。web側ではlive UIからsong playbackを触れるようにし、multiline songのhelpやglobal BPMの説明も補った。
関連PR: [moondsp PR #112](https://github.com/dowdiness/moondsp/pull/112)、[moondsp PR #113](https://github.com/dowdiness/moondsp/pull/113)、[moondsp PR #115](https://github.com/dowdiness/moondsp/pull/115)、[moondsp PR #116](https://github.com/dowdiness/moondsp/pull/116)

### js_engine

bytecode prototypeの性能を測る入口を整えた。PRごとにbase-vs-headのbenchmarkを出せるようにし、live benchmark dashboardを再設計してcommit dateやscan controlを見やすくした。さらにplain object property helperとbytecode environment lookupのhot pathを最適化し、startup Hyperfine workflow、startup decomposition helper、startup phase breakdown benchmarkを追加した。
関連PR: [js_engine PR #173](https://github.com/dowdiness/js_engine/pull/173)、[js_engine PR #174](https://github.com/dowdiness/js_engine/pull/174)、[js_engine PR #175](https://github.com/dowdiness/js_engine/pull/175)、[js_engine PR #176](https://github.com/dowdiness/js_engine/pull/176)、[js_engine PR #177](https://github.com/dowdiness/js_engine/pull/177)、[js_engine PR #178](https://github.com/dowdiness/js_engine/pull/178)、[js_engine PR #182](https://github.com/dowdiness/js_engine/pull/182)、[js_engine PR #183](https://github.com/dowdiness/js_engine/pull/183)、[js_engine PR #184](https://github.com/dowdiness/js_engine/pull/184)
