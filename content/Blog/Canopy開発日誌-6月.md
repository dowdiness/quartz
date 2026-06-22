---
title: Canopy開発日誌-6月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-06-23T23:59:00+09:00
---

# Canopy開発日誌-6月

2026年6月のCanopy開発ログ。細かいPR単位の列挙よりも、作業の流れが追いやすいように要点だけを残す。

> Canopyは、テキストとプログラムの構造を同じ空間で直接編集できる、次世代のコードエディタです。ソースコードを文字列ではなく「構造」として扱うことで、安全なリファクタリングやAIとの協調作業、CRDTによる複数人同時編集を実現しています。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

## 今月の大きな流れ

6月前半は、Canopyを「構造編集の実験場」から、より明確なプロダクトとライブラリ群へ整理する作業が中心だった。後半は、Markdownの構造編集（SDEG: Structure-Directed Edit Grammar）、外部解析結果を取り込むanalysis query layer、全リポジトリのmoon.mod.json→moon.mod移行、js_engineのtest262適合率向上へと重心が移った。

- **Lambda編集**: 名前の有効範囲を追跡するscope graph、block内だけに効くbinding編集、LoomのCstFoldを使った投影構築、古い`ModuleProjection`の削除、alpha変換に安全なbeta reductionの実験、構造編集後もNodeIdを保つGrove Level 1 hintまで進んだ。
- **Markdown SDEG**: 見出しやリスト項目を構造編集の対象にするべく、heading identityの安定性調査、リスト項目の同一リスト内move、block move provenanceの追跡、リストmove blockerの敵対的ケース対応が進んだ。
- **Ideal / Canvas**: 見た目を持たないUI部品ライブラリとしてRabbitaを使い、Tailwind v4へ寄せ、CanvasはUI状態ではなくsource textを正として動くsource-backed方式にした。CodeMirror source editorとFFI file I/Oも入った。
- **Analysis query layer**: 外部解析結果（ast-grepなど）をCanopyへ安全に取り込む層の設計・Phase 1実装。解析結果はsnapshotに紐づく捨てられるfactとして扱い、text CRDTの永続性を崩さない。
- **moon.mod移行**: Canopyが所有する全マニフェストをmoon.mod.jsonからmoon.mod（TOML形式）へ移行。13 submoduleをworkspace memberに追加し、MOON_WORK=offの依存解決問題を解消した。
- **Loom**: Lambda側の土台に加え、Markdownの変更されたblockだけを再パースする仕組み、MarkdownIRのmdast exportとposition付け、rewrite / canonical formatterのsource-preserving保証が進んだ。
- **incr**: Incremental TEA prototypeが進んだ。TEA風のUIをincrの依存グラフで更新する実験で、7GUIs stress test、inactive-root activation policyまで扱った。
- **js_engine**: 責務でファイルを分けるarchitecture refactor、bytecode実行の近道になるfast path、ES2024 Set methods、test262/benchmark基盤の整備が進んだ。後半はtest262適合率向上が中心で、JSON.parse 100% pass、Promise spec fixes、regex escapes修正、lexer UTF-16正確化、timer queue高速化など。
- **MoonDsp**: Canopyとは直接統合せず、共有基盤を見ながらGraph runtimeやschedulerの責務分割を進めた。
- **作業運用**: 作業記録をGit log・agent session・project memoryの突き合わせで書く運用が定着しつつある。

## 2026/6/1

### Canopy

CanopyとMoonDspの関係を整理した。Canopyは構造編集の土台、MoonDspは音楽DSL / DSP側の実験、incr / Loomは共有基盤として扱う方針にした。

Lambda projectionでは、`ProjNode`を作る補助関数、source位置を記録するSourceMap token helper、Block / Holeを型付きで見るviewを追加した。そのうえで、module直下の`let`定義を`LetDef`という独立したprojection nodeにした。以前はbinding rowが初期値expressionのIDや仮IDを借りていて、drag & dropやbinding単位のeditで「何を動かしているのか」が曖昧だった。`LetDef`が実nodeになったことで、binding rowを構造編集の対象として素直に扱えるようになった。

性能面では、`to_flat_proj_incremental`が本当に遅いのかを測るBAND 2bの確認をした。1000個の定義では数msまで伸びることは分かったが、現実のCanopy文書ではまだその規模にならない。そのため、すぐ最適化せず、必要になった時点で再開することにした。

### loom / js_engine / 運用

Loomではjson-settings exampleにlast-good semantic projection attachmentを追加した。これはparseやsemantic変換に失敗しても、最後に成功したprojectionを保持してUIを壊さないための仕組みになる。Lambda exampleにはBlock / Hole / LetDef系のtyped viewを入れた。js_engineではrepeat benchmark runnerとArray mutatorのhole / `undefined`まわりを整理した。

運用面では、Codexをpre-PR reviewだけでなく実装計画の書き手として使う流れが固まり始めた。

主なPR / Issue: canopy [#445](https://github.com/dowdiness/canopy/pull/445), [#437](https://github.com/dowdiness/canopy/pull/437), [#447](https://github.com/dowdiness/canopy/pull/447) / loom [#206](https://github.com/dowdiness/loom/pull/206) / js_engine [#186](https://github.com/dowdiness/js_engine/pull/186)

## 2026/6/2

### Canopy

前日の性能調査をJS targetでも確認した。遅さの原因は単純に木を歩く回数ではなく、再利用されたCstNode同士を比較するときのpointer chasing、つまりcache内の参照をたどるコストが支配的だった。ここでも最適化は見送り、大きなdocumentを扱う必要が出てから手を付けることにした。

Lambda projectionでは`FlatProj`を`ModuleProjection`へ改名した。単なる名前変更ではなく、「module全体の投影結果を差分更新する単位」だと分かる名前にした。ReuseCursor、ProjectionIdentityTracker、incremental projectionの設計上の迷いもADRに分けて記録し、microbenchmarkも本番と同じID採番になるよう合わせた。

### Incr visualizer / Canvas

IncrGraph panelは、依存関係の形だけでなく、どのcellが再計算され、どれくらい時間がかかったかも見えるようにした。cellごとの最新eventだけを持つbufferにして、履歴が増え続けないようにしたうえで、最新recompute時間とlegendを追加した。遅いcellが視覚的に浮くようになり、単なる構造図から調査用のpanelへ少し近づいた。

Canvasでは、graph demoをより構造編集寄りに使うための準備を進めた。

### CI / 運用

Ideal web E2EをPR gateへ載せる準備を進めた。つまり、ブラウザ上でIdealが壊れていないことをPRごとの自動チェックに含める方向へ進めた。同時に、CIの一時的な失敗と本当の失敗をどう見分けるかも整理した。

主なPR / Issue: canopy [#451](https://github.com/dowdiness/canopy/pull/451), [#452](https://github.com/dowdiness/canopy/pull/452), [#453](https://github.com/dowdiness/canopy/pull/453), [#462](https://github.com/dowdiness/canopy/pull/462), [#465](https://github.com/dowdiness/canopy/pull/465), [#469](https://github.com/dowdiness/canopy/pull/469)

## 2026/6/3

### Canopy

MoonDsp / Canopyの全体方針をmainへ入れた。source-backed canvas demoも、source textから作ったgraphをブラウザで触れるところまで進めた。

CIではIdeal web E2EをPR gateに載せた。CanopyのUI変更を、ローカル手動確認だけでなくCI上のブラウザテストで支える方向へ進んだ。

### MoonDsp / incr

MoonDspでは、Canopy連携を見据えながらGraph runtimeの境界を整理した。audio callbackへeditor側の責務を持ち込まない、という境界が少しずつ明確になった。

incr側ではpublic event APIの命名をDerived寄りに整理した。

主なPR / Issue: canopy [#445](https://github.com/dowdiness/canopy/pull/445), [#461](https://github.com/dowdiness/canopy/pull/461), [#479](https://github.com/dowdiness/canopy/pull/479)

## 2026/6/4

### Canopy

Rabbita headless UIをCanopyで本当に使えるかを見極める日だった。Disclosure PoCとdialog spikeで、RabbitaをIdeal UIの土台にできる感触を確認した。

### MoonDsp / js_engine

MoonDspではGraph runtimeのfacade / internal boundaryを切り始めた。外から使う公開APIと、内部だけで変えてよい実装部分を分ける作業になる。js_engineではArray method fast path delegationやTest262 runner shadowの準備が進んだ。

主なPR / Issue: canopy [#489](https://github.com/dowdiness/canopy/pull/489), [#508](https://github.com/dowdiness/canopy/pull/508), [#511](https://github.com/dowdiness/canopy/pull/511)

## 2026/6/5

### Canopy / Rabbita headless UI

Rabbitaのpatched更新を取り込み、IdealとCanvasからheadless UI primitiveを実際に使い始めた。headless UI primitiveは、見た目のCSSを押し付けず、状態管理やキーボード操作だけを提供する部品のこと。Action Menu、ContextMenu、Tabs、TreeViewまで進み、Ideal UIの土台がRabbitaへ寄っていった。特にContextMenuは、Canvasの右クリックメニューを実際の利用側にして、右クリック位置を基準にメニューを出すこと、外側クリックで閉じること、閉じた後にfocusを戻すこと、画面外へはみ出さない配置まで確認した。

### incr / MoonDsp / js_engine

incrではtyped spreadsheetとIncremental TEAの実験が進んだ。MoonDspではeditor / audio runtimeのhandoff contractを文書化し、js_engineではArray mutatorやrunner shadow化を進めた。

主なPR / Issue: canopy [#517](https://github.com/dowdiness/canopy/pull/517), [#523](https://github.com/dowdiness/canopy/pull/523), [#524](https://github.com/dowdiness/canopy/pull/524), [#525](https://github.com/dowdiness/canopy/pull/525), [#526](https://github.com/dowdiness/canopy/pull/526), [#528](https://github.com/dowdiness/canopy/pull/528)

## 2026/6/6

### Canopy / Ideal UI foundation

IdealのUI基盤を大きく整理した。パネルのリサイズ、スクリーンリーダーへ状態を伝えるlive-region、重複CSSの削除、Tailwind v4移行、overlay、toolbar、bottom tabs、panel、inspector、outline resize handleを小さな単位で進めた。

### 周辺リポジトリ

MoonDspではGraph runtime / scheduler / browser internalsの分割が進んだ。Loom、incr、js_engineでも、それぞれparser runtimeやIncremental TEA、test262 runnerの整備が続いた。

主なPR / Issue: canopy [#529](https://github.com/dowdiness/canopy/pull/529), [#532](https://github.com/dowdiness/canopy/pull/532), [#534](https://github.com/dowdiness/canopy/pull/534), [#539](https://github.com/dowdiness/canopy/pull/539), [#541](https://github.com/dowdiness/canopy/pull/541), [#544](https://github.com/dowdiness/canopy/pull/544)

## 2026/6/7

### Canopy / Ideal and protocol

Idealの残タスクとprotocolの曖昧さを潰した。outline E2E、bridgeが一部だけ成功したbatchをどう扱うか、cursor intentの単位名、protocol上の座標が何を指すかのdocs、MoonBit registry cacheを入れた。

### Canvas / loom

Canvasは次のsource-backed段階へ戻した。source-backedとは、画面上のnode/edgeを直接正とするのではなく、裏にあるGraph DSL sourceを正として、そこから画面を作り直す方式のこと。`lib/canvas-graph`の抽出も進めた。

LoomではMoonBit parser integrationが進み、editorへ渡せるsyntax artifactを出せる方向へ寄っていった。

主なPR / Issue: canopy [#553](https://github.com/dowdiness/canopy/pull/553), [#554](https://github.com/dowdiness/canopy/pull/554), [#558](https://github.com/dowdiness/canopy/pull/558), [#560](https://github.com/dowdiness/canopy/pull/560), [#562](https://github.com/dowdiness/canopy/pull/562)

## 2026/6/8

### Canvas

Canvasのsource-backed graphをCodeMirror source editorへ寄せた。source-backed inspector edit、selection remap、CM6 change delta lowering、CodeMirror source panel mountを順に入れた。ここで、UI stateを直接いじるのではなくGraph DSL sourceへ下ろし、再パースされたsource-backed graphを表示し直す流れがはっきりしてきた。

編集途中でparseに失敗したときの回復方針も決めた。失敗時に入力を巻き戻すのではなく、editor bufferを常に正しい入力として扱う。parseに成功していればcurrent resultを表示し、失敗していれば最後に成功したlast-goodを表示する。

### loom / incr / js_engine

Loomではparser結果に基づいてsyntax roleの範囲を出すrole span、incrではIncremental TEAのkeyed DOM benchmark、js_engineではArray shift / unshiftのfast pathやrunner parityが進んだ。

主なPR / Issue: canopy [#569](https://github.com/dowdiness/canopy/pull/569), [#570](https://github.com/dowdiness/canopy/pull/570), [#571](https://github.com/dowdiness/canopy/pull/571), [#576](https://github.com/dowdiness/canopy/pull/576)

## 2026/6/9

### Canopy

Canvas stable identityのPR2を進めた。ここでのstable identityは、sourceを編集して画面を作り直しても同じnode/edgeを同じものとして扱えるIDのこと。`NodeId` / `EdgeId`をsource-backedな文字列identityへ寄せ、binding remap用の一時的なshimを消す方向へ進めた。

Lambda側では、generic projection memosへ向かう前段として、scope graphやprojection identityの整理が続いた。

### loom / MoonDsp

Loomではparser runtime attachmentやdeprecated syntax移行が進んだ。MoonDspではmini parser置き換えcampaignが進み、loomへの置き換え方針が具体化した。

主なPR / Issue: canopy [#571](https://github.com/dowdiness/canopy/pull/571), [#575](https://github.com/dowdiness/canopy/pull/575), [#577](https://github.com/dowdiness/canopy/pull/577)

## 2026/6/10

### incr / Incremental TEA

Incremental TEAを一気に仕上げた。renderer lifecycle、keyed VDOM diff、benchmark、subscriptionsがmergeされ、prototypeの主要issueがすべて閉じた。

### MoonDsp / loom / Canopy / js_engine

MoonDspはloom parser置き換えcampaignのPhase 2 parityを完走し、ADR-0016をAcceptedにした。Loomではseparated-listやattachment系が進み、CanopyではCanvasのruntime seam整理が続いた。seamは境界面のことで、どこから先をruntime責務にするかを明確にする作業だった。js_engineはv0.3.0をリリースした。

主なPR / Issue: incr [#209](https://github.com/dowdiness/incr/pull/209), [#211](https://github.com/dowdiness/incr/pull/211), [#238](https://github.com/dowdiness/incr/pull/238), [#243](https://github.com/dowdiness/incr/pull/243), [#244](https://github.com/dowdiness/incr/pull/244) / canopy [#611](https://github.com/dowdiness/canopy/pull/611), [#615](https://github.com/dowdiness/canopy/pull/615)

## 2026/6/11

### loom

separated-list（#279）とgroup shape helpers（#196）を畳んだ。Loom側のparser / syntax helperが、Canopyの各言語実装を支えやすい形になってきた。

### Canopy / アーキテクチャ再設計

Canopyのアーキテクチャ再設計に着手した。S0のproposalとAPI boundary ADRを入れ、S1としてprotocol / wireを抽出した。

### js_engine / 運用

js_engineではCI cache改善が効かないことを実測で確認し、test262 shardingへ方針を切り替えた。作業運用としては、相談・レビュー・実装計画をどのエージェントに任せるかの使い分けも少し固まった。

主なPR / Issue: loom [#279](https://github.com/dowdiness/loom/pull/279), [#196](https://github.com/dowdiness/loom/pull/196) / canopy [#587](https://github.com/dowdiness/canopy/pull/587), [#588](https://github.com/dowdiness/canopy/pull/588) / js_engine [#344](https://github.com/dowdiness/js_engine/pull/344), [#349](https://github.com/dowdiness/js_engine/pull/349)

## 2026/6/12

### Canopy / アーキテクチャ再設計

再設計のS2からS5aまでを一気に進めた。editorからsync sessionとtransportを切り出し、言語runtimeがeditorに提供するインターフェース（lang/runtime SPI）、FFIやhost機能の登録場所、基盤層の責務ルール、依存方向をチェックするimport-graph lintで境界を固定した。単なるファイル分割ではなく、editor、runtime、transport、host機能が互いに勝手に漏れないようにするための制度化だった。

### プロダクト方針

Canopyを「editor / frameworkの証明」から、「write-to-selfのpost product」へ寄せる方針転換を決めた。ここでのpost productは、完成された一つのアプリというより、自分の思考や作業ログを後から自分へ返すための環境としてCanopyを見る、という意味で使っている。最小のwrite→surfaceループのprototypeを置き、source retrievalより前に「書いたものが自分へ返ってくる」体験を確かめる方向へ進んだ。

### 周辺リポジトリ

incrではIncremental TEAのrendererとsubscriptionがさらに進み、Loomではparser runtime attachment、js_engineではarchitecture refactor Stage 0-7、MoonDspではscheduler周辺の責務分割が進んだ。

主なPR / Issue: canopy [#587](https://github.com/dowdiness/canopy/pull/587), [#588](https://github.com/dowdiness/canopy/pull/588), [#589](https://github.com/dowdiness/canopy/pull/589), [#590](https://github.com/dowdiness/canopy/pull/590), [#597](https://github.com/dowdiness/canopy/pull/597), [#599](https://github.com/dowdiness/canopy/pull/599), [#610](https://github.com/dowdiness/canopy/pull/610)

## 2026/6/13

### Canopy / editor-neutral test grammar

editorをlanguageから完全に切り離すため、neutral test grammarを入れた。これはLambdaのような実言語ではなく、editor機能をテストするためだけの小さな構文で、editorテストが特定言語に引きずられないようにするためのもの。これにより、editorのテストがLambda固有の構文に依存しすぎる問題を減らした。

### Codex連携

Codex app-serverとMCP wrapperの使い分けを検証した。相談役としてはMCP、streamingやinteractive approvalなど「Codexの上に作る」用途ではapp-server、という整理をした。

### プロダクトとCanvas

write→surfaceループには、どのメモを再表示するかを決めるresurfacing signalでの並べ替えと、同じ入力から追加で問い直すsame-input askを追加した。Canvasではruntimeとの境界を切り出し、どちらが何を保証するかのcontractsも整理した。

### loom / js_engine

LoomではMarkdownIRやMarkdown block reparseの準備が進み、js_engineではStage 0-7のarchitecture refactorが進んだ。

主なPR / Issue: canopy [#602](https://github.com/dowdiness/canopy/pull/602), [#604](https://github.com/dowdiness/canopy/pull/604), [#609](https://github.com/dowdiness/canopy/pull/609), [#610](https://github.com/dowdiness/canopy/pull/610), [#611](https://github.com/dowdiness/canopy/pull/611), [#615](https://github.com/dowdiness/canopy/pull/615), [#619](https://github.com/dowdiness/canopy/pull/619)

## 2026/6/14

### Canopy / Lambda CstFold modernization

Lambda投影のCstFold現代化を進めた。CstFoldはLoom側のCST（具象構文木）を畳み込んでAST相当の構造へ変換する仕組みで、Canopy側の手組み変換を減らす狙いがある。既存Canopy semanticsとLoom CstFoldの差分を明示し、互換adapterを置いたうえで、leaf / if / app / bopなどを段階的にCstFold経由へ寄せた。たとえば`{ 1 }`や`{ }`の扱いはLoomの素のCstFoldとCanopy editor semanticsで異なるため、Loom側を変えずにCanopy側adapterで吸収する判断にした。

その流れで`DefinitionIndex`を切り出し、scope / edit / semantic / companion / idealの各利用側を`ModuleProjection`から離し始めた。`DefinitionIndex`は、どのnodeがどのmodule定義に対応するかを引くための薄い索引になる。block-local renameも正しく動くようになった。

### Canvas / loom / incr / MoonDsp / js_engine

Canvasでは接続preview互換性をMoonBit側へ寄せ、source-backed demoの`defer_sync`順序を整理した。LoomではMarkdown incremental block reparse、incrではincr_tea benchmark、MoonDspではscheduler facade分割、js_engineではarchitecture redesign Stage 8-10が進んだ。

主なPR / Issue: canopy [#637](https://github.com/dowdiness/canopy/pull/637), [#638](https://github.com/dowdiness/canopy/pull/638), [#640](https://github.com/dowdiness/canopy/pull/640), [#641](https://github.com/dowdiness/canopy/pull/641), [#644](https://github.com/dowdiness/canopy/pull/644), [#647](https://github.com/dowdiness/canopy/pull/647), [#648](https://github.com/dowdiness/canopy/pull/648), [#655](https://github.com/dowdiness/canopy/pull/655), [#639](https://github.com/dowdiness/canopy/pull/639), [#643](https://github.com/dowdiness/canopy/pull/643)

## 2026/6/15

### Canopy / Lambda §20 completion

Lambda編集のblock-local対応を一気に仕上げた。block-local対応とは、root直下の`let`だけでなく、block式の中にある`let`もrename / duplicate / moveなどの対象として正しく扱うこと。delete / duplicate / move binding ops、move opのscoping soundness、block-shadowing filter、typed `fn` token metadata、EditContext整理を入れた。moveでは、同scopeの前後関係だけでなくlambda paramや外側のmodule defによるshadowingも見ないと、参照の向きが静かに変わることが分かり、scope graph resolutionを使う形へ寄せた。

最後にlegacy `ModuleProjection`を削除し、Lambdaはgeneric projection memosへ完全に移行した。これはLambda専用の古いprojection cacheをやめ、他言語と同じ汎用memo stackでprojectionを作るという意味になる。これでdocs/TODO.md §20のbinding clauseは完了した。

### FFI / Ideal

`lib/js`を新設し、JS値をMoonBit側で抽象的に持つopaque `Any` handle、JS property / method / globalへ触る最低限のescape hatch、JSON、Promise bridgeを置いた。escape hatchは、型付きAPIがまだないJS機能へ一時的にアクセスするための出口になる。これを使って`ffi/io`にfile read/writeを実装し、IdealのOpen / Save toolbarから呼べるようにした。

Ideal側では`globalThis.__canopy_*`を単一の`__canopy_bridge`へ集約した。

### 周辺リポジトリ

LoomではMarkdownIR M0 policyとM1 heading/paragraph slice、incrではspreadsheet proofとinactive root、js_engineではES2024 Set methodsやinternal slots整理が進んだ。

主なPR / Issue: canopy [#660](https://github.com/dowdiness/canopy/pull/660), [#663](https://github.com/dowdiness/canopy/pull/663), [#671](https://github.com/dowdiness/canopy/pull/671), [#673](https://github.com/dowdiness/canopy/pull/673), [#664](https://github.com/dowdiness/canopy/pull/664), [#677](https://github.com/dowdiness/canopy/pull/677), [#666](https://github.com/dowdiness/canopy/pull/666), [#670](https://github.com/dowdiness/canopy/pull/670), [#669](https://github.com/dowdiness/canopy/pull/669), [#668](https://github.com/dowdiness/canopy/pull/668) / loom [#342](https://github.com/dowdiness/loom/pull/342), [#346](https://github.com/dowdiness/loom/pull/346) / incr [#273](https://github.com/dowdiness/incr/pull/273) / js_engine [#356](https://github.com/dowdiness/js_engine/pull/356)

## 2026/6/16

### Canopy / Lambda §20完了とalpha pilot

`ExtractToLet`をblock-awareにした。block body pathではbody expressionの直前に、block def pathではblock defsの前に`let`を挿入する。block scopeとlambda scopeを区別し、capture guardも追加した。rootへ無理にhoistするのではなく、選択位置のscopeに近い場所へbindingを作る方針にした。

Lambdaのalpha-safe beta pilotを`lang/lambda/alpha`に置いた。これは、変数名の偶然の一致で意味が変わらないようにbinder identityで計算する実験になる。`ScopeGraph`から投影termを内部表現へlowerし、rootだけbeta reductionし、captureが起きない形でnamed `Term`へ戻す。

夕方以降には、このalpha-safe core boundaryをmainへ入れ、binding-id compatibility fallbackも削った。古いinit-idでもbindingを見つける互換経路を消し、実際のLetDef idだけを見るようにした。Ideal側ではTabs UI helperをvalue-derivedな小さな層へ切り出した。

### 周辺リポジトリ

LoomではMarkdownIR M1 vertical slice、incrではinactive-root cohort測定、js_engineではbytecode call-frame fast pathが進んだ。js_engineではparam bindingのenv round-tripをskipし、binding stepが大きく改善した。

主なPR / Issue: canopy [#674](https://github.com/dowdiness/canopy/pull/674), [#682](https://github.com/dowdiness/canopy/pull/682), [#683](https://github.com/dowdiness/canopy/pull/683), [#684](https://github.com/dowdiness/canopy/pull/684), [#685](https://github.com/dowdiness/canopy/pull/685), issue [#659](https://github.com/dowdiness/canopy/pull/659) / loom [#346](https://github.com/dowdiness/loom/pull/346) / incr [#277](https://github.com/dowdiness/incr/pull/277), [#279](https://github.com/dowdiness/incr/pull/279) / js_engine [#365](https://github.com/dowdiness/js_engine/pull/365), [#366](https://github.com/dowdiness/js_engine/pull/366)

## 2026/6/17

### Canopy / Lambda編集の健全性

Lambda編集は「生成された文字列を見る」よりも、「編集後テキストを再パースして意図したASTになるかを見る」方針へ寄せた。これは、`_copy`のように見た目では小さな命名差に見えても、実際にはlexerで読めずASTに戻らないケースを踏んだため。

- block-local binding editの敵対的ケースを追加した（move up/down、delete、root/block shadowing）。
- `ScopeGraph.node_scope` / `node_cutoffs`をprivateにし、query API経由にした。内部Mapを直接読むのではなく、意味のある問い合わせ関数を通す形にして、後で実装を変えやすくした。
- `ExtractToLet`は#674の実装で#659の懸念を満たしていることを、再パース付きregression testで確認した。
- `DuplicateBinding`は`_copy`ではなく`x1`, `x2`, ... のようなlexableな名前を使うようにした。後続のfree referenceを捕まえる候補も避ける。

これで[#649](https://github.com/dowdiness/canopy/pull/649)と[#659](https://github.com/dowdiness/canopy/pull/659)は完了。[#650](https://github.com/dowdiness/canopy/pull/650)はmove/deleteのindentationとして残る。

主なPR / Issue: canopy [#688](https://github.com/dowdiness/canopy/pull/688), [#689](https://github.com/dowdiness/canopy/pull/689), [#691](https://github.com/dowdiness/canopy/pull/691), [#696](https://github.com/dowdiness/canopy/pull/696)

### Canopy / Grove Level 1 identity hint

構造編集後もNodeIdを保ちやすくするため、Grove Level 1のidentity hint channelを入れた。狙いは、`Var(x)`を`Lam(Param, Var(x))`で包むようなkind mismatchの編集でも、内側のNodeIdを失わず、selectionやfoldなどのUI状態を保つこと。

core側では`IdentityTransform`と`reconcile_hinted`を追加した。`IdentityTransform`は「この編集はwrapです」「この子を残してunwrapします」のような編集意図を表すhintで、`reconcile_hinted`はそのhintを使って古い木と新しい木のNodeId対応を決める。editor側では`SyncEditor`がspan edit batchに対応するhintを保持し、Lambda側では`TreeEditOp`を`IdentityTransform`へ落とす。

`WrapInLambda`などはNodeId維持に使えるhintを出す。一方、`ExtractToLet`や`ChangeOperator`のように安全なhintを出しづらいものは、保守的に`Opaque`へ落とす。最後にWrap / UnwrapのE2E coverageも追加した。

残りは、write / read / clearの2端contractを`HintChannel`型で包むことと、hintあり / なしreconcileの重複整理。

主なPR / Issue: canopy [#690](https://github.com/dowdiness/canopy/pull/690), [#697](https://github.com/dowdiness/canopy/pull/697), [#698](https://github.com/dowdiness/canopy/pull/698)

### Canopy / analysis query layer設計

外部解析結果をCanopyへ入れるためのanalysis query layer設計を追加した。これは、ast-grepや`moon ide`の結果をそのままeditor内部へ混ぜるのではなく、Canopy側で扱いやすい共通形式へ変換する層になる。解析結果は、どの版のtextから得たものかを表すsnapshotに紐づけ、種類の分かるtyped factとして保存し、画面上ではdecorationsとして表示する。ここでもtext CRDTが唯一の永続状態で、analysis resultは古くなれば捨てるもの、という境界を崩さない。

Phase 1は、ast-grepのbyte offsetをUTF-16 rangeへ変換してrange highlightするだけ。byte offsetとエディタの文字位置はずれやすいので、まず位置変換を安全にするところから始める。rewrite、node-id mapping、protocol変更はまだしない。

主なPR / Issue: canopy [#687](https://github.com/dowdiness/canopy/pull/687)

### Canopy / Ideal

IdealのAction Overlayを、UI helperがCell / Emit handleを直接持たず、値だけを受け取る形へ寄せた。つまり、UIを描く補助関数がRabbitaの状態セルやイベント送信口を握らず、呼び出し側が計算済みの値とcallbackを渡す形に近づけた。action overlayのflowとexecも分け直した。

主なPR / Issue: canopy [#686](https://github.com/dowdiness/canopy/pull/686)

### loom

MarkdownIRはM1からrecovery / raw node semanticsへ進んだ。ここでのraw nodeは未対応・壊れた入力をそのまま保持するnodeで、recovered nodeはparserが回復しながら作ったnodeを指す。raw / recovered diagnostics、mdast export、direct syntax diagnostics、recovery adapter contractを追加し、HTML harnessへ送る範囲も明確にした。特に、editor向けには壊れた入力も保持し、canonical rewriteやHTML出力ではどこまで正規化するかを変換先ごとに分ける方針になった。

現在の作業ツリーでは、次の[#328](https://github.com/dowdiness/loom/pull/328)相当としてMarkdownIR mdast exportにunist `position`を付ける変更が進行中。Canopy superprojectから見ると、`loom` submoduleは`0a827c3`から`3856167`へ進んだうえで未コミット差分が残っている。

主なPR / Issue: loom [#348](https://github.com/dowdiness/loom/pull/348), [#350](https://github.com/dowdiness/loom/pull/350), [#351](https://github.com/dowdiness/loom/pull/351), [#352](https://github.com/dowdiness/loom/pull/352), [#353](https://github.com/dowdiness/loom/pull/353), [#354](https://github.com/dowdiness/loom/pull/354), [#355](https://github.com/dowdiness/loom/pull/355), [#356](https://github.com/dowdiness/loom/pull/356), [#357](https://github.com/dowdiness/loom/pull/357), [#358](https://github.com/dowdiness/loom/pull/358)

### incr

incr_teaはinactive-root測定からactivation policyへ進んだ。inactive-rootは、DOMは残したまま更新を止めておく非表示UI subtreeのことで、activation policyはそれをいつ再び動かすかの方針になる。ratio tableを再確認し、activation trigger probeを追加し、policyをdocsで決めて実装まで入れた。Loom submodule内の`incr`も`34ac477`から`f7681bc`へ進んでいる。

主なPR / Issue: incr [#281](https://github.com/dowdiness/incr/pull/281), [#282](https://github.com/dowdiness/incr/pull/282), [#284](https://github.com/dowdiness/incr/pull/284), [#285](https://github.com/dowdiness/incr/pull/285)

### js_engine

`needs_own_env`系列のbytecode最適化が続いた。これは、関数呼び出し時に新しいEnvironmentを本当に作る必要があるかを事前に判定し、不要なら生成をskipする最適化になる。leaf bytecode functionで`Environment::new`をskipし、same-realm calleeではrealm-proto wrapperを避け、active-override `Ref`も単一の`Ref[FunctionRealmProtos?]`へ畳んだ。最後にbenchmark tableへ`exec/for_of` rowを追加した。

主なPR / Issue: js_engine [#367](https://github.com/dowdiness/js_engine/pull/367), [#368](https://github.com/dowdiness/js_engine/pull/368), [#369](https://github.com/dowdiness/js_engine/pull/369), [#370](https://github.com/dowdiness/js_engine/pull/370), [#371](https://github.com/dowdiness/js_engine/pull/371), [#372](https://github.com/dowdiness/js_engine/pull/372)

### 作業運用メモ

今回の記録は、Canopy / Loom / nested incr / js_engineのGit log、現在の未コミット差分、Claude project memory、Codex memoryを突き合わせて書いた。

6/17時点で特に重要なのは次の3点。

1. Lambda editの検証は、実際の編集後テキストを再パースして確認する。
2. analysis factはsnapshotに紐づき、いつでも捨てられるものとして扱う。
3. Grove hint channelは、次の構造編集言語へ広げる前に`HintChannel`型で明示する。

## 2026/6/18

### Canopy / analysis query layer実装

前日に設計したanalysis query layerのPhase 1を実装してmainへ入れた。`lib/analysis`側には、document identity・version・32bit hash・UTF-16長を持つ`SourceSnapshot`、UTF-16 rangeとpattern id / capturesを持つ`PatternMatchFact`、ast-grepのbyte offsetをエディタ側のUTF-16 offsetへ変換するhelperを置いた。

Canopy側の`analysis` packageでは、ast-grep由来のmatchを`PatternMatchFact`へ変換し、それをprotocol decorationやmatch-list entryへ落とすadapterを追加した。初期ルールとしてMoonBitの`fn`定義を拾うast-grep ruleも入った。途中で、snapshotの同一性判定がversionとhashだけだと、同じ内容の別documentを誤ってcurrent扱いしてしまう問題が見つかり、`doc_id`と`utf16_len`も見るように直した。

これでPhase 1は「外部解析結果をsnapshotに紐づく捨てられるfactとして受け、range highlight / match list用の値へ変換する」ところまで到達した。まだhost-side FFI wiring、つまりJSから実際にast-grep結果を渡してUIへ表示する部分は次の段階に残っている。

主なPR / Issue: canopy [#699](https://github.com/dowdiness/canopy/pull/699), [#692](https://github.com/dowdiness/canopy/pull/692), [#693](https://github.com/dowdiness/canopy/pull/693), [#694](https://github.com/dowdiness/canopy/pull/694), [#695](https://github.com/dowdiness/canopy/pull/695)

### loom / MarkdownIR

MarkdownIRはmdast exportにunist `position`を付けるところまで進んだ。positionはMarkdownIR内部のsource originと`LineIndex`からexport境界で作るもので、MarkdownIR自体をmdast位置情報に引きずられないようにしている。non-BMP文字、raw / recovered node、block separator、fenced code block、CRLFなど、位置がずれやすいケースもtestで押さえた。

その後は[#333](https://github.com/dowdiness/loom/pull/333)のrewrite / canonical formatter側へ進み、code fenceとlinkのsource-preserving rewrite smoke coverageを追加した。特にcode fenceでは、contentだけを書き換える場合にfenceそのものや周辺sourceを壊さないこと、unclosed fenceでもrewriteの境界を守ることを確認している。

主なPR / Issue: loom [#359](https://github.com/dowdiness/loom/pull/359), [#360](https://github.com/dowdiness/loom/pull/360), [#361](https://github.com/dowdiness/loom/pull/361)

### incr / Incremental TEA

incr_teaでは7GUIs stress testを追加した。Counter、Temperature Converter、Flight Booker、Timer、CRUD、Circle Drawer、Cellsを別packageとして置き、TEA風UIをincrの依存グラフでどこまで扱えるかを見るための実験面が広がった。

同時に、`on_change`やpointer offsetまわりの小さなAPIも整えた。前日のinactive-root activation policyに続き、単体の小さなdemoではなく、複数のUIパターンを並べて「Rabbitaとは別のincremental UI substrateとして成立するか」を見る段階に入った。次は残っているTEA follow-up、特にlocal pointer coordinateまわりの整理が候補になる。

主なPR / Issue: incr [#291](https://github.com/dowdiness/incr/pull/291), [#268](https://github.com/dowdiness/incr/pull/268), [#286](https://github.com/dowdiness/incr/pull/286), [#287](https://github.com/dowdiness/incr/pull/287), [#288](https://github.com/dowdiness/incr/pull/288), [#289](https://github.com/dowdiness/incr/pull/289), [#290](https://github.com/dowdiness/incr/pull/290)

### Rabbita / pointer events

Canopyが参照しているRabbita forkでは、pointer event bindingのbranchが進んだ。DOM / HTML / subscription層へ`PointerEvent`系のbindingを足し、後続のUIでmouse専用ではなくpointer入力として扱えるようにする準備になる。実装後にconstructorやcast styleを既存のmouse event conversionに揃える修正も入った。

まだCanopy親リポジトリでは、`rabbita` submodule pointerが未コミット差分として残っている。

### js_engine

js_engineでは、Set iterationの仕様バグを直した。`Set.prototype.forEach`中にcallbackが最後の要素をdeleteしてaddし直すと、仕様上は再追加された値が未訪問の新しいslotとして再度訪問される。これに合わせて、activeな`forEach`中は物理削除せずtombstoneとして残し、外側のiterationが終わってからcompactするモデルにした。`clear()`、`values()` / `keys()` / `@@iterator`、`entries()`もtombstoneを考慮するようにして、無限ループを避けるone-shot guard付きtestも追加した。

並行して、`Function.prototype.toString`が元sourceを返せるようにするPRが開かれている。parser / AST / runtimeへsource textやspanを通す大きめの変更で、review後にoriginal sourceからspanを作る修正まで進んだ。こちらはmainにはまだ入っていない。別PRではdocsのroadmap / design / decisions配置を整理し、現在のarchitecture targetやtest262 snapshotを更新した。

主なPR / Issue: js_engine [#373](https://github.com/dowdiness/js_engine/pull/373), [#374](https://github.com/dowdiness/js_engine/pull/374), [#375](https://github.com/dowdiness/js_engine/pull/375), [#310](https://github.com/dowdiness/js_engine/pull/310), [#357](https://github.com/dowdiness/js_engine/pull/357)

### 作業運用メモ

今日の時点で、Canopy親リポジトリは`loom`と`rabbita` submodule pointerがdirtyになっている。`loom`は`b26a304`まで進み、その中の`incr` submoduleも`7a971ab`まで進んでいる。`rabbita`はpointer event branchの`54b3188`まで進んでいるが、親側で取り込むかどうかはまだ未整理。

今日の大きな流れは、Canopy本体ではanalysis layerを最小実装まで進め、周辺ではMarkdownIRのexport / rewrite保証、incr_teaのUI stress surface、Rabbitaのpointer入力、js_engineのSet仕様適合とdocs整理が並行して進んだ、という感じだった。

## 2026/6/19

### Canopy / Markdown SDEG heading

Markdown見出しを構造編集の対象にするSDEG（Structure-Directed Edit Grammar）の調査を進めた。

- Markdown headingのNodeIdが編集をまたいでどのくらい安定するかを探るheading identity probeを追加した（[#716](https://github.com/dowdiness/canopy/pull/716)）。
- SDEG NodeId side tableの設計スケッチをtestとして置いた（[#717](https://github.com/dowdiness/canopy/pull/717)）。これは、各blockのNodeIdを横断的に引ける補助表の構想になる。
- heading edit pathのE2E validationを加え（[#718](https://github.com/dowdiness/canopy/pull/718)）、SDEG Phase 0での発見をPhase 1計画へ持ち越すdocsも更新した（[#719](https://github.com/dowdiness/canopy/pull/719)）。

主なPR / Issue: canopy [#716](https://github.com/dowdiness/canopy/pull/716), [#717](https://github.com/dowdiness/canopy/pull/717), [#718](https://github.com/dowdiness/canopy/pull/718), [#719](https://github.com/dowdiness/canopy/pull/719)

### js_engine

- shared AsyncGeneratorPrototype chainを実装した（[#405](https://github.com/dowdiness/js_engine/pull/405)）。ES仕様 §27.4に従い、`async function*` で作られる各generatorが共通のprototype chainを共有するようにした。
- tokenに`end_offset` fieldを追加し、parserがsourceを再スキャンする必要をなくした（[#403](https://github.com/dowdiness/js_engine/pull/403)）。これまではparserがトークンの終端位置を知るためにsource内を再度走査していたが、lexer時点でUTF-16コードユニット単位のend_offsetを記録するようにした。
- runtime atomicsのtest coverage（[#402](https://github.com/dowdiness/js_engine/pull/402)）とtest262 toolingの回帰テスト（[#401](https://github.com/dowdiness/js_engine/pull/401)）を追加した。

主なPR / Issue: js_engine [#405](https://github.com/dowdiness/js_engine/pull/405), [#403](https://github.com/dowdiness/js_engine/pull/403), [#402](https://github.com/dowdiness/js_engine/pull/402), [#401](https://github.com/dowdiness/js_engine/pull/401)

## 2026/6/20

### Canopy / Markdown list SDEG

Markdownリストの構造編集向けの基盤を一気に進めた。

- Markdown SDEG heading side tableを抽出し、見出し用の補助表を独立させた（[#722](https://github.com/dowdiness/canopy/pull/722)）。
- block move provenanceを追加した（[#723](https://github.com/dowdiness/canopy/pull/723)）。これはblockを移動したときに「どこから来たか」を追跡する仕組みで、後続のリスト項目moveに必要な前提になる。
- Markdown list move blockerをhardeningした（[#726](https://github.com/dowdiness/canopy/pull/726)）。これは、リスト項目の移動が安全にできないケース（異なる階層や種類のリスト間の移動など）をきちんと弾くためのもの。
- same-list Markdown item movesを有効化し（[#731](https://github.com/dowdiness/canopy/pull/731)）、リスト項目payloadの消費も追加した（[#730](https://github.com/dowdiness/canopy/pull/730)）。

主なPR / Issue: canopy [#722](https://github.com/dowdiness/canopy/pull/722), [#723](https://github.com/dowdiness/canopy/pull/723), [#726](https://github.com/dowdiness/canopy/pull/726), [#730](https://github.com/dowdiness/canopy/pull/730), [#731](https://github.com/dowdiness/canopy/pull/731)

### js_engine / test262適合率向上

この日はjs_engineにとって大きなfixデーになった。test262の失敗を系統的に潰し、JSON.parseが100% passに到達した。

- **lexer: astral_count追跡**（[#408](https://github.com/dowdiness/js_engine/pull/408)）。MoonBitのStringはUTF-16ベースだが、lexerのoffset計算がコードポイント単位だったため、サロゲートペアを含むソースでは全トークンの位置がずれていた。`astral_count`変数を追加し、非BMP文字を読むたびに+1してUTF-16コードユニット数へ補正するようにした。
- **JSON.parse reviver Proxy-aware**（[#419](https://github.com/dowdiness/js_engine/pull/419)）。`JSON.parse`のreviver処理をinterpreter-awareな抽象操作経由に置き換え、Proxyの`[[Get]]`/`[[Delete]]`/`[[DefineOwnProperty]]` trapが正しく発火するようにした。JSON/parse: 36 failures → 0（100%, 142/142）。
- **Promise spec fixes 5件**（[#413](https://github.com/dowdiness/js_engine/pull/413)）。`Symbol.toStringTag`のdescriptor、non-object thisでのTypeError、iter-poisonedケース、`Promise.any`のresolve/reject element guard、newTarget.prototypeの反映。
- **`__lookupGetter__`/`__lookupSetter__`と`replaceAll`修正**（[#412](https://github.com/dowdiness/js_engine/pull/412)）。Annex Bのgetter/setter lookupと、`String.prototype.replaceAll`のIsRegExp判定・`Symbol.replace` override対応。
- **surrogate-safe string slicing**（[#411](https://github.com/dowdiness/js_engine/pull/411)）。`classify_by_edition`ツールが絵文字などでpanicするのを修正。
- **regex `\u`/`\x` escape対応**（[#420](https://github.com/dowdiness/js_engine/pull/420)）。文字クラス内の`\uXXXX`/`\xHH`/`\f`/`\v`/`\0`/`\b`が全てリテラル文字扱いされていたバグを修正。Annex Bのidentity escapeやnon-Unicodeモードのサロゲートペア非結合もカバー。
- **匿名built-in関数のname/length/property order**（[#410](https://github.com/dowdiness/js_engine/pull/410)）と**singleton %GeneratorPrototype%**（[#407](https://github.com/dowdiness/js_engine/pull/407)）。
- test262: await-dictionary（`Promise.allKeyed`/`allSettledKeyed`）をskip（[#377](https://github.com/dowdiness/js_engine/pull/377)）。

主なPR / Issue: js_engine [#408](https://github.com/dowdiness/js_engine/pull/408), [#419](https://github.com/dowdiness/js_engine/pull/419), [#413](https://github.com/dowdiness/js_engine/pull/413), [#412](https://github.com/dowdiness/js_engine/pull/412), [#411](https://github.com/dowdiness/js_engine/pull/411), [#420](https://github.com/dowdiness/js_engine/pull/420), [#410](https://github.com/dowdiness/js_engine/pull/410), [#407](https://github.com/dowdiness/js_engine/pull/407)

## 2026/6/21

### Canopy / Markdown list + moon.mod移行開始

- Markdown listの構造編集を引き続き進め、list payloadの消費（[#730](https://github.com/dowdiness/canopy/pull/730)）とsame-list item move（[#731](https://github.com/dowdiness/canopy/pull/731)）をmainへ入れた。
- package mapの文書化（[#736](https://github.com/dowdiness/canopy/pull/736)）を行い、全Canopyパッケージの依存関係と名称を整理した。
- これを受けて、moon.mod.json→moon.mod移行の第一弾としてRabbita UI lib cluster（[#737](https://github.com/dowdiness/canopy/pull/737)）とlib/visualizer（[#738](https://github.com/dowdiness/canopy/pull/738)）を変換した。moon.mod（TOML形式）へ移行することで、MoonBitのworkspace membershipを使った依存解決が可能になり、`NEW_MOON_MOD=0`のデフォルト化へ近づく。

主なPR / Issue: canopy [#736](https://github.com/dowdiness/canopy/pull/736), [#737](https://github.com/dowdiness/canopy/pull/737), [#738](https://github.com/dowdiness/canopy/pull/738)

### js_engine / lexer・spec fixes・perf

- **regex/division disambiguation after `}`**（[#422](https://github.com/dowdiness/js_engine/pull/422)）。`}`の後に`/`が来たとき、それが除算か正規表現リテラルの開始かを判定するcontext trackingを全面的に書き直した。`}`がstatement blockを閉じるかobject literalを閉じるかを追跡するbrace-is-block stack、ternary colonとlabel/case colonを区別するternary colon stack、nested classのextends内で状態が壊れないようにするbrace-is-block stackを導入した。
- **bind length ToIntegerOrInfinity + new FunctionのJS ToString coercion**（[#431](https://github.com/dowdiness/js_engine/pull/431)）。`Function.prototype.bind`のlength引数処理を仕様通り`ToIntegerOrInfinity`へ修正し、`new Function`の引数をMoonBitの`.to_string()`ではなくJSの`ToString`抽象操作で評価するようにした。
- **Annex B web-compat call-assign**（[#428](https://github.com/dowdiness/js_engine/pull/428)）。non-strict modeで`CallExpression`が代入の左辺に来たとき、parse時エラーではなくruntime `ReferenceError`にするAnnex B互換動作を実装した。
- **analysis-family growth convention docs**（[#332](https://github.com/dowdiness/js_engine/pull/332)）。静的解析のファイル構成ルールを文書化した。
- **timer queueをpriority_queueへ移行**（[#433](https://github.com/dowdiness/js_engine/pull/433)）。それまでの`Array[TimerTask] + sort_by + remove(0)`（O(n² log n)）を`@priority_queue.PriorityQueue[TimerTask]`（O(n log n)）へ置き換え、200タイマーのdrainが4.19ms→1.95ms（2.15×高速化）。キャンセルは遅延削除方式にした。

主なPR / Issue: js_engine [#422](https://github.com/dowdiness/js_engine/pull/422), [#431](https://github.com/dowdiness/js_engine/pull/431), [#428](https://github.com/dowdiness/js_engine/pull/428), [#332](https://github.com/dowdiness/js_engine/pull/332), [#433](https://github.com/dowdiness/js_engine/pull/433)

## 2026/6/22

### Canopy / moon.mod移行完了

全Canopy-ownedマニフェストのmoon.mod.json→moon.mod移行を完了した（[#740](https://github.com/dowdiness/canopy/pull/740)）。これが今週最大の変更で、以下の内容を含む。

- **7つのCanopy-ownedマニフェストを変換**: ルート、lib/semantic、examples/resizable、examples/codemirror_demo、examples/block-editor、examples/canvas、examples/ideal。
- **13 submoduleをworkspace memberに追加**: loom/examples/{markdown,json,lambda,graph-dsl}、loom/{loom,seam,pretty,text-change,moji,egglog,egraph}、event-graph-walker、rle、order-tree。
- **全MOON_WORK=offを削除**: moon.modの`import { }`構文はworkspace membershipなしでは依存解決できないため、benchmark、E2Eテスト、Cloudflare deploy、JS build scriptからMOON_WORK=offを全て取り除いた。
- **CI整備**: vendored-submoduleのエラーを抑止する共通filterを導入（`scripts/vendored-check-common.sh`）。workspace全体のcheckでsubmoduleが起こす21件のpre-existing errorを抑制しつつ、submodule自身のCIでは自前の失敗を隠さないよう`--keep`オプションを付ける設計にした。
- **Cloudflare deploy修正**: workspace buildの成果物が`_build/js/release/build/<module>/<pkg>/`へ出力されるのに対し、vite-plugin-moonbitが期待するパスとのずれをsymlinkで吸収した（[#335](https://github.com/dowdiness/canopy/pull/335)）。

並行して、loom submoduleのquickcheck 0.14 Arrow API compat対応や、AGENTS.mdのsubmodule guidance更新も行った。

主なPR / Issue: canopy [#740](https://github.com/dowdiness/canopy/pull/740), [#335](https://github.com/dowdiness/canopy/pull/335)

## 2026/6/23

### Canopy / submodule bumpとHTML block修正

月曜の大規模移行の後始末と後続のsubmodule更新を進めた。

- **submodule bump**: egraphのmoon.mod移行（loom#455）、event-graph-walkerのtrait split修正（#58）、alga v0.4.0の取り込み、graphvizのDirectedGraph互換修正。これらのsubmoduleはmoon.mod.jsonからmoon.modへの移行をそれぞれ進めており、Canopy側で追従した。
- **HTML blocks §4.6**: loom submoduleをbumpし、MarkdownのHTML blockをprojection block childrenに含める対応を入れた。block modeでHTML blockが`text:null`/`editable:false`として扱われてしまい、表示から消えるバグがあった。`HtmlBlock`に適切なtoken spanをpopulateすることで修正した。
- 残っているsubmodule（svg-dsl、rle、order-tree、graphviz）のmoon.mod移行完了に伴うbumpが[#742](https://github.com/dowdiness/canopy/pull/742)として進行中。

主なPR / Issue: canopy [#742](https://github.com/dowdiness/canopy/pull/742)

### 作業運用メモ

6/19〜23の期間は、CanopyではMarkdown SDEGの基盤整備とmoon.mod移行が2本柱だった。js_engineではtest262適合率の系統的な向上が続き、JSON.parse 100% pass到達がハイライト。

今日時点でCanopyは、loom submoduleが`3856167`からHTML blocks §4.6を含む先まで進み、alga、event-graph-walker、graphvizもそれぞれ新バージョンへ進んでいる。js_engineはtest262適合率の改善が一段落し、次はbytecode最適化やPromise.allKeyedなどの残件へ向かう流れ。
