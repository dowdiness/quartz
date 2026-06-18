---
title: Canopy開発日誌-6月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-06-18T15:46:29+09:00
---

# Canopy開発日誌-6月

2026年6月のCanopy開発ログ。細かいPR単位の列挙よりも、作業の流れが追いやすいように要点だけを残す。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

## 今月の大きな流れ

6月前半は、Canopyを「構造編集の実験場」から、より明確なプロダクトとライブラリ群へ整理する作業が中心だった。

- **Lambda編集**: 名前の有効範囲を追跡するscope graph、block内だけに効くbinding編集、LoomのCstFoldを使った投影構築、古い`ModuleProjection`の削除、alpha変換に安全なbeta reductionの実験、構造編集後もNodeIdを保つGrove Level 1 hintまで進んだ。
- **Ideal / Canvas**: 見た目を持たないUI部品ライブラリとしてRabbitaを使い、Tailwind v4へ寄せ、CanvasはUI状態ではなくsource textを正として動くsource-backed方式にした。CodeMirror source editorとFFI file I/Oも入った。
- **Loom**: Lambda側の土台に加え、Markdownの変更されたblockだけを再パースする仕組みと、Markdownを安全に別表現へ変換するMarkdownIRの設計が進んだ。
- **incr**: Incremental TEA prototypeが進んだ。これはTEA風のUIをincrの依存グラフで更新する実験で、描画の生存期間、key付き差分、subscription、非表示rootの停止、再有効化方針まで扱った。
- **js_engine**: 責務でファイルを分けるarchitecture refactor、bytecode実行の近道になるfast path、ES2024 Set methods、test262/benchmark基盤の整備が進んだ。
- **MoonDsp**: Canopyとは直接統合せず、共有基盤を見ながらGraph runtimeやschedulerの責務分割を進めた。

## 2026/6/1

### Canopy

CanopyとMoonDspの関係を整理した。Canopyは構造編集の土台、MoonDspは音楽DSL / DSP側の実験、incr / Loomは共有基盤として扱う方針にした。

Lambda projectionでは、`ProjNode`を作る補助関数、source位置を記録するSourceMap token helper、Block / Holeを型付きで見るviewを追加した。そのうえで、module直下の`let`定義を`LetDef`という独立したprojection nodeにした。以前はbinding rowが初期値expressionのIDや仮IDを借りていて、drag & dropやbinding単位のeditで「何を動かしているのか」が曖昧だった。`LetDef`が実nodeになったことで、binding rowを構造編集の対象として素直に扱えるようになった。

性能面では、`to_flat_proj_incremental`が本当に遅いのかを測るBAND 2bの確認をした。1000個の定義では数msまで伸びることは分かったが、現実のCanopy文書ではまだその規模にならない。そのため、すぐ最適化せず、必要になった時点で再開することにした。

### loom / js_engine / 運用

Loomではjson-settings exampleにlast-good semantic projection attachmentを追加した。これはparseやsemantic変換に失敗しても、最後に成功したprojectionを保持してUIを壊さないための仕組みになる。Lambda exampleにはBlock / Hole / LetDef系のtyped viewを入れた。js_engineではrepeat benchmark runnerとArray mutatorのhole / `undefined`まわりを整理した。

運用面では、Codexをpre-PR reviewだけでなく実装計画の書き手として使う流れが固まり始めた。

主なPR / Issue: canopy [#445](https://github.com/dowdiness/canopy/issues/445), [#437](https://github.com/dowdiness/canopy/issues/437), [#447](https://github.com/dowdiness/canopy/issues/447) / loom [#206](https://github.com/dowdiness/loom/issues/206) / js_engine [#186](https://github.com/dowdiness/js_engine/issues/186)

## 2026/6/2

### Canopy

前日の性能調査をJS targetでも確認した。遅さの原因は単純に木を歩く回数ではなく、再利用されたCstNode同士を比較するときのpointer chasing、つまりcache内の参照をたどるコストが支配的だった。ここでも最適化は見送り、大きなdocumentを扱う必要が出てから手を付けることにした。

Lambda projectionでは`FlatProj`を`ModuleProjection`へ改名した。単なる名前変更ではなく、「module全体の投影結果を差分更新する単位」だと分かる名前にした。ReuseCursor、ProjectionIdentityTracker、incremental projectionの設計上の迷いもADRに分けて記録し、microbenchmarkも本番と同じID採番になるよう合わせた。

### Incr visualizer / Canvas

IncrGraph panelは、依存関係の形だけでなく、どのcellが再計算され、どれくらい時間がかかったかも見えるようにした。cellごとの最新eventだけを持つbufferにして、履歴が増え続けないようにしたうえで、最新recompute時間とlegendを追加した。遅いcellが視覚的に浮くようになり、単なる構造図から調査用のpanelへ少し近づいた。

Canvasでは、graph demoをより構造編集寄りに使うための準備を進めた。

### CI / 運用

Ideal web E2EをPR gateへ載せる準備を進めた。つまり、ブラウザ上でIdealが壊れていないことをPRごとの自動チェックに含める方向へ進めた。同時に、CIの一時的な失敗と本当の失敗をどう見分けるかも整理した。

主なPR / Issue: canopy [#451](https://github.com/dowdiness/canopy/issues/451), [#452](https://github.com/dowdiness/canopy/issues/452), [#453](https://github.com/dowdiness/canopy/issues/453), [#462](https://github.com/dowdiness/canopy/issues/462), [#465](https://github.com/dowdiness/canopy/issues/465), [#469](https://github.com/dowdiness/canopy/issues/469)

## 2026/6/3

### Canopy

MoonDsp / Canopyの全体方針をmainへ入れた。source-backed canvas demoも、source textから作ったgraphをブラウザで触れるところまで進めた。

CIではIdeal web E2EをPR gateに載せた。CanopyのUI変更を、ローカル手動確認だけでなくCI上のブラウザテストで支える方向へ進んだ。

### MoonDsp / incr

MoonDspでは、Canopy連携を見据えながらGraph runtimeの境界を整理した。audio callbackへeditor側の責務を持ち込まない、という境界が少しずつ明確になった。

incr側ではpublic event APIの命名をDerived寄りに整理した。

主なPR / Issue: canopy [#445](https://github.com/dowdiness/canopy/issues/445), [#461](https://github.com/dowdiness/canopy/issues/461), [#479](https://github.com/dowdiness/canopy/issues/479)

## 2026/6/4

### Canopy

Rabbita headless UIをCanopyで本当に使えるかを見極める日だった。Disclosure PoCとdialog spikeで、RabbitaをIdeal UIの土台にできる感触を確認した。

### MoonDsp / js_engine

MoonDspではGraph runtimeのfacade / internal boundaryを切り始めた。外から使う公開APIと、内部だけで変えてよい実装部分を分ける作業になる。js_engineではArray method fast path delegationやTest262 runner shadowの準備が進んだ。

主なPR / Issue: canopy [#489](https://github.com/dowdiness/canopy/issues/489), [#508](https://github.com/dowdiness/canopy/issues/508), [#511](https://github.com/dowdiness/canopy/issues/511)

## 2026/6/5

### Canopy / Rabbita headless UI

Rabbitaのpatched更新を取り込み、IdealとCanvasからheadless UI primitiveを実際に使い始めた。headless UI primitiveは、見た目のCSSを押し付けず、状態管理やキーボード操作だけを提供する部品のこと。Action Menu、ContextMenu、Tabs、TreeViewまで進み、Ideal UIの土台がRabbitaへ寄っていった。特にContextMenuは、Canvasの右クリックメニューを実際の利用側にして、右クリック位置を基準にメニューを出すこと、外側クリックで閉じること、閉じた後にfocusを戻すこと、画面外へはみ出さない配置まで確認した。

### incr / MoonDsp / js_engine

incrではtyped spreadsheetとIncremental TEAの実験が進んだ。MoonDspではeditor / audio runtimeのhandoff contractを文書化し、js_engineではArray mutatorやrunner shadow化を進めた。

主なPR / Issue: canopy [#517](https://github.com/dowdiness/canopy/issues/517), [#523](https://github.com/dowdiness/canopy/issues/523), [#524](https://github.com/dowdiness/canopy/issues/524), [#525](https://github.com/dowdiness/canopy/issues/525), [#526](https://github.com/dowdiness/canopy/issues/526), [#528](https://github.com/dowdiness/canopy/issues/528)

## 2026/6/6

### Canopy / Ideal UI foundation

IdealのUI基盤を大きく整理した。パネルのリサイズ、スクリーンリーダーへ状態を伝えるlive-region、重複CSSの削除、Tailwind v4移行、overlay、toolbar、bottom tabs、panel、inspector、outline resize handleを小さな単位で進めた。

### 周辺リポジトリ

MoonDspではGraph runtime / scheduler / browser internalsの分割が進んだ。Loom、incr、js_engineでも、それぞれparser runtimeやIncremental TEA、test262 runnerの整備が続いた。

主なPR / Issue: canopy [#529](https://github.com/dowdiness/canopy/issues/529), [#532](https://github.com/dowdiness/canopy/issues/532), [#534](https://github.com/dowdiness/canopy/issues/534), [#539](https://github.com/dowdiness/canopy/issues/539), [#541](https://github.com/dowdiness/canopy/issues/541), [#544](https://github.com/dowdiness/canopy/issues/544)

## 2026/6/7

### Canopy / Ideal and protocol

Idealの残タスクとprotocolの曖昧さを潰した。outline E2E、bridgeが一部だけ成功したbatchをどう扱うか、cursor intentの単位名、protocol上の座標が何を指すかのdocs、MoonBit registry cacheを入れた。

### Canvas / loom

Canvasは次のsource-backed段階へ戻した。source-backedとは、画面上のnode/edgeを直接正とするのではなく、裏にあるGraph DSL sourceを正として、そこから画面を作り直す方式のこと。`lib/canvas-graph`の抽出も進めた。

LoomではMoonBit parser integrationが進み、editorへ渡せるsyntax artifactを出せる方向へ寄っていった。

主なPR / Issue: canopy [#553](https://github.com/dowdiness/canopy/issues/553), [#554](https://github.com/dowdiness/canopy/issues/554), [#558](https://github.com/dowdiness/canopy/issues/558), [#560](https://github.com/dowdiness/canopy/issues/560), [#562](https://github.com/dowdiness/canopy/issues/562)

## 2026/6/8

### Canvas

Canvasのsource-backed graphをCodeMirror source editorへ寄せた。source-backed inspector edit、selection remap、CM6 change delta lowering、CodeMirror source panel mountを順に入れた。ここで、UI stateを直接いじるのではなくGraph DSL sourceへ下ろし、再パースされたsource-backed graphを表示し直す流れがはっきりしてきた。

編集途中でparseに失敗したときの回復方針も決めた。失敗時に入力を巻き戻すのではなく、editor bufferを常に正しい入力として扱う。parseに成功していればcurrent resultを表示し、失敗していれば最後に成功したlast-goodを表示する。

### loom / incr / js_engine

Loomではparser結果に基づいてsyntax roleの範囲を出すrole span、incrではIncremental TEAのkeyed DOM benchmark、js_engineではArray shift / unshiftのfast pathやrunner parityが進んだ。

主なPR / Issue: canopy [#569](https://github.com/dowdiness/canopy/issues/569), [#570](https://github.com/dowdiness/canopy/issues/570), [#571](https://github.com/dowdiness/canopy/issues/571), [#576](https://github.com/dowdiness/canopy/issues/576)

## 2026/6/9

### Canopy

Canvas stable identityのPR2を進めた。ここでのstable identityは、sourceを編集して画面を作り直しても同じnode/edgeを同じものとして扱えるIDのこと。`NodeId` / `EdgeId`をsource-backedな文字列identityへ寄せ、binding remap用の一時的なshimを消す方向へ進めた。

Lambda側では、generic projection memosへ向かう前段として、scope graphやprojection identityの整理が続いた。

### loom / MoonDsp

Loomではparser runtime attachmentやdeprecated syntax移行が進んだ。MoonDspではmini parser置き換えcampaignが進み、loomへの置き換え方針が具体化した。

主なPR / Issue: canopy [#571](https://github.com/dowdiness/canopy/issues/571), [#575](https://github.com/dowdiness/canopy/issues/575), [#577](https://github.com/dowdiness/canopy/issues/577)

## 2026/6/10

### incr / Incremental TEA

Incremental TEAを一気に仕上げた。renderer lifecycle、keyed VDOM diff、benchmark、subscriptionsがmergeされ、prototypeの主要issueがすべて閉じた。

### MoonDsp / loom / Canopy / js_engine

MoonDspはloom parser置き換えcampaignのPhase 2 parityを完走し、ADR-0016をAcceptedにした。Loomではseparated-listやattachment系が進み、CanopyではCanvasのruntime seam整理が続いた。seamは境界面のことで、どこから先をruntime責務にするかを明確にする作業だった。js_engineはv0.3.0をリリースした。

主なPR / Issue: incr [#209](https://github.com/dowdiness/incr/issues/209), [#211](https://github.com/dowdiness/incr/issues/211), [#238](https://github.com/dowdiness/incr/issues/238), [#243](https://github.com/dowdiness/incr/issues/243), [#244](https://github.com/dowdiness/incr/issues/244) / canopy [#611](https://github.com/dowdiness/canopy/issues/611), [#615](https://github.com/dowdiness/canopy/issues/615)

## 2026/6/11

### loom

separated-list（#279）とgroup shape helpers（#196）を畳んだ。Loom側のparser / syntax helperが、Canopyの各言語実装を支えやすい形になってきた。

### Canopy / アーキテクチャ再設計

Canopyのアーキテクチャ再設計に着手した。S0のproposalとAPI boundary ADRを入れ、S1としてprotocol / wireを抽出した。

### js_engine / 運用

js_engineではCI cache改善が効かないことを実測で確認し、test262 shardingへ方針を切り替えた。作業運用としては、相談・レビュー・実装計画をどのエージェントに任せるかの使い分けも少し固まった。

主なPR / Issue: loom [#279](https://github.com/dowdiness/loom/issues/279), [#196](https://github.com/dowdiness/loom/issues/196) / canopy [#587](https://github.com/dowdiness/canopy/issues/587), [#588](https://github.com/dowdiness/canopy/issues/588) / js_engine [#344](https://github.com/dowdiness/js_engine/issues/344), [#349](https://github.com/dowdiness/js_engine/issues/349)

## 2026/6/12

### Canopy / アーキテクチャ再設計

再設計のS2からS5aまでを一気に進めた。editorからsync sessionとtransportを切り出し、言語runtimeがeditorに提供するインターフェース（lang/runtime SPI）、FFIやhost機能の登録場所、基盤層の責務ルール、依存方向をチェックするimport-graph lintで境界を固定した。単なるファイル分割ではなく、editor、runtime、transport、host機能が互いに勝手に漏れないようにするための制度化だった。

### プロダクト方針

Canopyを「editor / frameworkの証明」から、「write-to-selfのpost product」へ寄せる方針転換を決めた。ここでのpost productは、完成された一つのアプリというより、自分の思考や作業ログを後から自分へ返すための環境としてCanopyを見る、という意味で使っている。最小のwrite→surfaceループのprototypeを置き、source retrievalより前に「書いたものが自分へ返ってくる」体験を確かめる方向へ進んだ。

### 周辺リポジトリ

incrではIncremental TEAのrendererとsubscriptionがさらに進み、Loomではparser runtime attachment、js_engineではarchitecture refactor Stage 0-7、MoonDspではscheduler周辺の責務分割が進んだ。

主なPR / Issue: canopy [#587](https://github.com/dowdiness/canopy/issues/587), [#588](https://github.com/dowdiness/canopy/issues/588), [#589](https://github.com/dowdiness/canopy/issues/589), [#590](https://github.com/dowdiness/canopy/issues/590), [#597](https://github.com/dowdiness/canopy/issues/597), [#599](https://github.com/dowdiness/canopy/issues/599), [#610](https://github.com/dowdiness/canopy/issues/610)

## 2026/6/13

### Canopy / editor-neutral test grammar

editorをlanguageから完全に切り離すため、neutral test grammarを入れた。これはLambdaのような実言語ではなく、editor機能をテストするためだけの小さな構文で、editorテストが特定言語に引きずられないようにするためのもの。これにより、editorのテストがLambda固有の構文に依存しすぎる問題を減らした。

### Codex連携

Codex app-serverとMCP wrapperの使い分けを検証した。相談役としてはMCP、streamingやinteractive approvalなど「Codexの上に作る」用途ではapp-server、という整理をした。

### プロダクトとCanvas

write→surfaceループには、どのメモを再表示するかを決めるresurfacing signalでの並べ替えと、同じ入力から追加で問い直すsame-input askを追加した。Canvasではruntimeとの境界を切り出し、どちらが何を保証するかのcontractsも整理した。

### loom / js_engine

LoomではMarkdownIRやMarkdown block reparseの準備が進み、js_engineではStage 0-7のarchitecture refactorが進んだ。

主なPR / Issue: canopy [#602](https://github.com/dowdiness/canopy/issues/602), [#604](https://github.com/dowdiness/canopy/issues/604), [#609](https://github.com/dowdiness/canopy/issues/609), [#610](https://github.com/dowdiness/canopy/issues/610), [#611](https://github.com/dowdiness/canopy/issues/611), [#615](https://github.com/dowdiness/canopy/issues/615), [#619](https://github.com/dowdiness/canopy/issues/619)

## 2026/6/14

### Canopy / Lambda CstFold modernization

Lambda投影のCstFold現代化を進めた。CstFoldはLoom側のCST（具象構文木）を畳み込んでAST相当の構造へ変換する仕組みで、Canopy側の手組み変換を減らす狙いがある。既存Canopy semanticsとLoom CstFoldの差分を明示し、互換adapterを置いたうえで、leaf / if / app / bopなどを段階的にCstFold経由へ寄せた。たとえば`{ 1 }`や`{ }`の扱いはLoomの素のCstFoldとCanopy editor semanticsで異なるため、Loom側を変えずにCanopy側adapterで吸収する判断にした。

その流れで`DefinitionIndex`を切り出し、scope / edit / semantic / companion / idealの各利用側を`ModuleProjection`から離し始めた。`DefinitionIndex`は、どのnodeがどのmodule定義に対応するかを引くための薄い索引になる。block-local renameも正しく動くようになった。

### Canvas / loom / incr / MoonDsp / js_engine

Canvasでは接続preview互換性をMoonBit側へ寄せ、source-backed demoの`defer_sync`順序を整理した。LoomではMarkdown incremental block reparse、incrではincr_tea benchmark、MoonDspではscheduler facade分割、js_engineではarchitecture redesign Stage 8-10が進んだ。

主なPR / Issue: canopy [#637](https://github.com/dowdiness/canopy/issues/637), [#638](https://github.com/dowdiness/canopy/issues/638), [#640](https://github.com/dowdiness/canopy/issues/640), [#641](https://github.com/dowdiness/canopy/issues/641), [#644](https://github.com/dowdiness/canopy/issues/644), [#647](https://github.com/dowdiness/canopy/issues/647), [#648](https://github.com/dowdiness/canopy/issues/648), [#655](https://github.com/dowdiness/canopy/issues/655), [#639](https://github.com/dowdiness/canopy/issues/639), [#643](https://github.com/dowdiness/canopy/issues/643)

## 2026/6/15

### Canopy / Lambda §20 completion

Lambda編集のblock-local対応を一気に仕上げた。block-local対応とは、root直下の`let`だけでなく、block式の中にある`let`もrename / duplicate / moveなどの対象として正しく扱うこと。delete / duplicate / move binding ops、move opのscoping soundness、block-shadowing filter、typed `fn` token metadata、EditContext整理を入れた。moveでは、同scopeの前後関係だけでなくlambda paramや外側のmodule defによるshadowingも見ないと、参照の向きが静かに変わることが分かり、scope graph resolutionを使う形へ寄せた。

最後にlegacy `ModuleProjection`を削除し、Lambdaはgeneric projection memosへ完全に移行した。これはLambda専用の古いprojection cacheをやめ、他言語と同じ汎用memo stackでprojectionを作るという意味になる。これでdocs/TODO.md §20のbinding clauseは完了した。

### FFI / Ideal

`lib/js`を新設し、JS値をMoonBit側で抽象的に持つopaque `Any` handle、JS property / method / globalへ触る最低限のescape hatch、JSON、Promise bridgeを置いた。escape hatchは、型付きAPIがまだないJS機能へ一時的にアクセスするための出口になる。これを使って`ffi/io`にfile read/writeを実装し、IdealのOpen / Save toolbarから呼べるようにした。

Ideal側では`globalThis.__canopy_*`を単一の`__canopy_bridge`へ集約した。

### 周辺リポジトリ

LoomではMarkdownIR M0 policyとM1 heading/paragraph slice、incrではspreadsheet proofとinactive root、js_engineではES2024 Set methodsやinternal slots整理が進んだ。

主なPR / Issue: canopy [#660](https://github.com/dowdiness/canopy/issues/660), [#663](https://github.com/dowdiness/canopy/issues/663), [#671](https://github.com/dowdiness/canopy/issues/671), [#673](https://github.com/dowdiness/canopy/issues/673), [#664](https://github.com/dowdiness/canopy/issues/664), [#677](https://github.com/dowdiness/canopy/issues/677), [#666](https://github.com/dowdiness/canopy/issues/666), [#670](https://github.com/dowdiness/canopy/issues/670), [#669](https://github.com/dowdiness/canopy/issues/669), [#668](https://github.com/dowdiness/canopy/issues/668) / loom [#342](https://github.com/dowdiness/loom/issues/342), [#346](https://github.com/dowdiness/loom/issues/346) / incr [#273](https://github.com/dowdiness/incr/issues/273) / js_engine [#356](https://github.com/dowdiness/js_engine/issues/356)

## 2026/6/16

### Canopy / Lambda §20完了とalpha pilot

`ExtractToLet`をblock-awareにした。block body pathではbody expressionの直前に、block def pathではblock defsの前に`let`を挿入する。block scopeとlambda scopeを区別し、capture guardも追加した。rootへ無理にhoistするのではなく、選択位置のscopeに近い場所へbindingを作る方針にした。

Lambdaのalpha-safe beta pilotを`lang/lambda/alpha`に置いた。これは、変数名の偶然の一致で意味が変わらないようにbinder identityで計算する実験になる。`ScopeGraph`から投影termを内部表現へlowerし、rootだけbeta reductionし、captureが起きない形でnamed `Term`へ戻す。

夕方以降には、このalpha-safe core boundaryをmainへ入れ、binding-id compatibility fallbackも削った。古いinit-idでもbindingを見つける互換経路を消し、実際のLetDef idだけを見るようにした。Ideal側ではTabs UI helperをvalue-derivedな小さな層へ切り出した。

### 周辺リポジトリ

LoomではMarkdownIR M1 vertical slice、incrではinactive-root cohort測定、js_engineではbytecode call-frame fast pathが進んだ。js_engineではparam bindingのenv round-tripをskipし、binding stepが大きく改善した。

主なPR / Issue: canopy [#674](https://github.com/dowdiness/canopy/issues/674), [#682](https://github.com/dowdiness/canopy/issues/682), [#683](https://github.com/dowdiness/canopy/issues/683), [#684](https://github.com/dowdiness/canopy/issues/684), [#685](https://github.com/dowdiness/canopy/issues/685), issue [#659](https://github.com/dowdiness/canopy/issues/659) / loom [#346](https://github.com/dowdiness/loom/issues/346) / incr [#277](https://github.com/dowdiness/incr/issues/277), [#279](https://github.com/dowdiness/incr/issues/279) / js_engine [#365](https://github.com/dowdiness/js_engine/issues/365), [#366](https://github.com/dowdiness/js_engine/issues/366)

## 2026/6/17

### Canopy / Lambda編集の健全性

Lambda編集は「生成された文字列を見る」よりも、「編集後テキストを再パースして意図したASTになるかを見る」方針へ寄せた。これは、`_copy`のように見た目では小さな命名差に見えても、実際にはlexerで読めずASTに戻らないケースを踏んだため。

- block-local binding editの敵対的ケースを追加した（move up/down、delete、root/block shadowing）。
- `ScopeGraph.node_scope` / `node_cutoffs`をprivateにし、query API経由にした。内部Mapを直接読むのではなく、意味のある問い合わせ関数を通す形にして、後で実装を変えやすくした。
- `ExtractToLet`は#674の実装で#659の懸念を満たしていることを、再パース付きregression testで確認した。
- `DuplicateBinding`は`_copy`ではなく`x1`, `x2`, ... のようなlexableな名前を使うようにした。後続のfree referenceを捕まえる候補も避ける。

これで[#649](https://github.com/dowdiness/canopy/issues/649)と[#659](https://github.com/dowdiness/canopy/issues/659)は完了。[#650](https://github.com/dowdiness/canopy/issues/650)はmove/deleteのindentationとして残る。

主なPR / Issue: canopy [#688](https://github.com/dowdiness/canopy/issues/688), [#689](https://github.com/dowdiness/canopy/issues/689), [#691](https://github.com/dowdiness/canopy/issues/691), [#696](https://github.com/dowdiness/canopy/issues/696)

### Canopy / Grove Level 1 identity hint

構造編集後もNodeIdを保ちやすくするため、Grove Level 1のidentity hint channelを入れた。狙いは、`Var(x)`を`Lam(Param, Var(x))`で包むようなkind mismatchの編集でも、内側のNodeIdを失わず、selectionやfoldなどのUI状態を保つこと。

core側では`IdentityTransform`と`reconcile_hinted`を追加した。`IdentityTransform`は「この編集はwrapです」「この子を残してunwrapします」のような編集意図を表すhintで、`reconcile_hinted`はそのhintを使って古い木と新しい木のNodeId対応を決める。editor側では`SyncEditor`がspan edit batchに対応するhintを保持し、Lambda側では`TreeEditOp`を`IdentityTransform`へ落とす。

`WrapInLambda`などはNodeId維持に使えるhintを出す。一方、`ExtractToLet`や`ChangeOperator`のように安全なhintを出しづらいものは、保守的に`Opaque`へ落とす。最後にWrap / UnwrapのE2E coverageも追加した。

残りは、write / read / clearの2端contractを`HintChannel`型で包むことと、hintあり / なしreconcileの重複整理。

主なPR / Issue: canopy [#690](https://github.com/dowdiness/canopy/issues/690), [#697](https://github.com/dowdiness/canopy/issues/697), [#698](https://github.com/dowdiness/canopy/issues/698)

### Canopy / analysis query layer設計

外部解析結果をCanopyへ入れるためのanalysis query layer設計を追加した。これは、ast-grepや`moon ide`の結果をそのままeditor内部へ混ぜるのではなく、Canopy側で扱いやすい共通形式へ変換する層になる。解析結果は、どの版のtextから得たものかを表すsnapshotに紐づけ、種類の分かるtyped factとして保存し、画面上ではdecorationsとして表示する。ここでもtext CRDTが唯一の永続状態で、analysis resultは古くなれば捨てるもの、という境界を崩さない。

Phase 1は、ast-grepのbyte offsetをUTF-16 rangeへ変換してrange highlightするだけ。byte offsetとエディタの文字位置はずれやすいので、まず位置変換を安全にするところから始める。rewrite、node-id mapping、protocol変更はまだしない。

主なPR / Issue: canopy [#687](https://github.com/dowdiness/canopy/issues/687)

### Canopy / Ideal

IdealのAction Overlayを、UI helperがCell / Emit handleを直接持たず、値だけを受け取る形へ寄せた。つまり、UIを描く補助関数がRabbitaの状態セルやイベント送信口を握らず、呼び出し側が計算済みの値とcallbackを渡す形に近づけた。action overlayのflowとexecも分け直した。

主なPR / Issue: canopy [#686](https://github.com/dowdiness/canopy/issues/686)

### loom

MarkdownIRはM1からrecovery / raw node semanticsへ進んだ。ここでのraw nodeは未対応・壊れた入力をそのまま保持するnodeで、recovered nodeはparserが回復しながら作ったnodeを指す。raw / recovered diagnostics、mdast export、direct syntax diagnostics、recovery adapter contractを追加し、HTML harnessへ送る範囲も明確にした。特に、editor向けには壊れた入力も保持し、canonical rewriteやHTML出力ではどこまで正規化するかを変換先ごとに分ける方針になった。

現在の作業ツリーでは、次の[#328](https://github.com/dowdiness/loom/issues/328)相当としてMarkdownIR mdast exportにunist `position`を付ける変更が進行中。Canopy superprojectから見ると、`loom` submoduleは`0a827c3`から`3856167`へ進んだうえで未コミット差分が残っている。

主なPR / Issue: loom [#348](https://github.com/dowdiness/loom/issues/348), [#350](https://github.com/dowdiness/loom/issues/350), [#351](https://github.com/dowdiness/loom/issues/351), [#352](https://github.com/dowdiness/loom/issues/352), [#353](https://github.com/dowdiness/loom/issues/353), [#354](https://github.com/dowdiness/loom/issues/354), [#355](https://github.com/dowdiness/loom/issues/355), [#356](https://github.com/dowdiness/loom/issues/356), [#357](https://github.com/dowdiness/loom/issues/357), [#358](https://github.com/dowdiness/loom/issues/358)

### incr

incr_teaはinactive-root測定からactivation policyへ進んだ。inactive-rootは、DOMは残したまま更新を止めておく非表示UI subtreeのことで、activation policyはそれをいつ再び動かすかの方針になる。ratio tableを再確認し、activation trigger probeを追加し、policyをdocsで決めて実装まで入れた。Loom submodule内の`incr`も`34ac477`から`f7681bc`へ進んでいる。

主なPR / Issue: incr [#281](https://github.com/dowdiness/incr/issues/281), [#282](https://github.com/dowdiness/incr/issues/282), [#284](https://github.com/dowdiness/incr/issues/284), [#285](https://github.com/dowdiness/incr/issues/285)

### js_engine

`needs_own_env`系列のbytecode最適化が続いた。これは、関数呼び出し時に新しいEnvironmentを本当に作る必要があるかを事前に判定し、不要なら生成をskipする最適化になる。leaf bytecode functionで`Environment::new`をskipし、same-realm calleeではrealm-proto wrapperを避け、active-override `Ref`も単一の`Ref[FunctionRealmProtos?]`へ畳んだ。最後にbenchmark tableへ`exec/for_of` rowを追加した。

主なPR / Issue: js_engine [#367](https://github.com/dowdiness/js_engine/issues/367), [#368](https://github.com/dowdiness/js_engine/issues/368), [#369](https://github.com/dowdiness/js_engine/issues/369), [#370](https://github.com/dowdiness/js_engine/issues/370), [#371](https://github.com/dowdiness/js_engine/issues/371), [#372](https://github.com/dowdiness/js_engine/issues/372)

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

主なPR / Issue: canopy [#699](https://github.com/dowdiness/canopy/issues/699), [#692](https://github.com/dowdiness/canopy/issues/692), [#693](https://github.com/dowdiness/canopy/issues/693), [#694](https://github.com/dowdiness/canopy/issues/694), [#695](https://github.com/dowdiness/canopy/issues/695)

### loom / MarkdownIR

MarkdownIRはmdast exportにunist `position`を付けるところまで進んだ。positionはMarkdownIR内部のsource originと`LineIndex`からexport境界で作るもので、MarkdownIR自体をmdast位置情報に引きずられないようにしている。non-BMP文字、raw / recovered node、block separator、fenced code block、CRLFなど、位置がずれやすいケースもtestで押さえた。

その後は[#333](https://github.com/dowdiness/loom/issues/333)のrewrite / canonical formatter側へ進み、code fenceとlinkのsource-preserving rewrite smoke coverageを追加した。特にcode fenceでは、contentだけを書き換える場合にfenceそのものや周辺sourceを壊さないこと、unclosed fenceでもrewriteの境界を守ることを確認している。

主なPR / Issue: loom [#359](https://github.com/dowdiness/loom/issues/359), [#360](https://github.com/dowdiness/loom/issues/360), [#361](https://github.com/dowdiness/loom/issues/361)

### incr / Incremental TEA

incr_teaでは7GUIs stress testを追加した。Counter、Temperature Converter、Flight Booker、Timer、CRUD、Circle Drawer、Cellsを別packageとして置き、TEA風UIをincrの依存グラフでどこまで扱えるかを見るための実験面が広がった。

同時に、`on_change`やpointer offsetまわりの小さなAPIも整えた。前日のinactive-root activation policyに続き、単体の小さなdemoではなく、複数のUIパターンを並べて「Rabbitaとは別のincremental UI substrateとして成立するか」を見る段階に入った。次は残っているTEA follow-up、特にlocal pointer coordinateまわりの整理が候補になる。

主なPR / Issue: incr [#291](https://github.com/dowdiness/incr/issues/291), [#268](https://github.com/dowdiness/incr/issues/268), [#286](https://github.com/dowdiness/incr/issues/286), [#287](https://github.com/dowdiness/incr/issues/287), [#288](https://github.com/dowdiness/incr/issues/288), [#289](https://github.com/dowdiness/incr/issues/289), [#290](https://github.com/dowdiness/incr/issues/290)

### Rabbita / pointer events

Canopyが参照しているRabbita forkでは、pointer event bindingのbranchが進んだ。DOM / HTML / subscription層へ`PointerEvent`系のbindingを足し、後続のUIでmouse専用ではなくpointer入力として扱えるようにする準備になる。実装後にconstructorやcast styleを既存のmouse event conversionに揃える修正も入った。

まだCanopy親リポジトリでは、`rabbita` submodule pointerが未コミット差分として残っている。

### js_engine

js_engineでは、Set iterationの仕様バグを直した。`Set.prototype.forEach`中にcallbackが最後の要素をdeleteしてaddし直すと、仕様上は再追加された値が未訪問の新しいslotとして再度訪問される。これに合わせて、activeな`forEach`中は物理削除せずtombstoneとして残し、外側のiterationが終わってからcompactするモデルにした。`clear()`、`values()` / `keys()` / `@@iterator`、`entries()`もtombstoneを考慮するようにして、無限ループを避けるone-shot guard付きtestも追加した。

並行して、`Function.prototype.toString`が元sourceを返せるようにするPRが開かれている。parser / AST / runtimeへsource textやspanを通す大きめの変更で、review後にoriginal sourceからspanを作る修正まで進んだ。こちらはmainにはまだ入っていない。別PRではdocsのroadmap / design / decisions配置を整理し、現在のarchitecture targetやtest262 snapshotを更新した。

主なPR / Issue: js_engine [#373](https://github.com/dowdiness/js_engine/issues/373), [#374](https://github.com/dowdiness/js_engine/issues/374), [#375](https://github.com/dowdiness/js_engine/issues/375), [#310](https://github.com/dowdiness/js_engine/issues/310), [#357](https://github.com/dowdiness/js_engine/issues/357)

### 作業運用メモ

今日の時点で、Canopy親リポジトリは`loom`と`rabbita` submodule pointerがdirtyになっている。`loom`は`b26a304`まで進み、その中の`incr` submoduleも`7a971ab`まで進んでいる。`rabbita`はpointer event branchの`54b3188`まで進んでいるが、親側で取り込むかどうかはまだ未整理。

今日の大きな流れは、Canopy本体ではanalysis layerを最小実装まで進め、周辺ではMarkdownIRのexport / rewrite保証、incr_teaのUI stress surface、Rabbitaのpointer入力、js_engineのSet仕様適合とdocs整理が並行して進んだ、という感じだった。
