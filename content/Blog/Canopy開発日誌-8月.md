---
title: Canopy開発日誌-8月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-08-04T22:54:40+09:00
modified: 2026-08-10T18:10:47+09:00
---

# Canopy開発日誌-8月

2026年8月のCanopy開発ログ（日次記録）。月の要約は[[Canopy開発日誌-8月-まとめ|8月-まとめ]]（月初4日分、随時更新）。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

## 今月の大きな流れ

進行中の月のため、以下は日付ごとの作業記録（8/10まで）。PR番号の一覧は文末の[PR索引](#pr索引)にある。7月の記録は[[Canopy開発日誌-7月|7月の日誌]]・[[Canopy開発日誌-7月-まとめ|7月-まとめ]]。

## 2026/8/1

### Canopy / 診断基盤の完成とLoomarkの産声

前日始めた診断パイプラインが、この日のうちに使える機能へ育った。reachable failure——実際に実行してみるまで分からない失敗——をdiagnosticsとして投影する仕組みが入り、監査用の独立プローブ、revisionに紐づけた診断publicationのプロトタイプ、Lambdaの構造検索結果UI、Signal Labのプリセットが朝から立て続けに着地している。ここまでの量が半日で積み上がると、診断は静的なlintの延長というより、実行時に何が起きたかを後から辿るための記録という性格を帯びてくる。Markdown projectionのコンテナalignmentを保つ細かな修正も同じ日に紛れ込んだ。

同じ日、Loomarkという名前が初めて現れた。「Loomark Markdown editorのための純粋なアプリケーションコア」とだけ説明された独立パッケージで、Canopy本体・dom_boundary・js_ffi・Rabbita、そして新顔のUIライブラリruiを土台に、typed Markdown foundationsから着手している。7/15にCanopy本体から切り出したjs_ffi・dom_boundaryが、切り出されて初めて「Canopyを土台にした別プロダクト」を生んだことになる。あの分離が正しかったかどうかは、Loomarkがこの先どこまで育つかで答えが出る話だ。移行に伴うapplication handoffのdocsも同日にまとまった。

### loom / CommonMark実装が完成する

7/31から続いていたCommonMark対応が、この日でほぼ揃った。completion handoffのdocsを書き、監査をcompletion-gradeへ引き上げたところから始まり、block containerの判定の一元化、raw HTMLポリシーの明示、entity参照のデコード、soft/hard line breakの型付け、autolinkとinline HTML、link reference definitionのlowering時収集、コンテナインデントの再パース境界修正、link/image参照の解決、diagnosticsをtoken kindから切り離すリファクタ、link destinationのシリアライズ修正、diagnostic coreの独立抽出、reference definition境界のクローズ修正と、CommonMarkのブロック・インライン構文が1日で並んだ。仕様書の項目を上から潰していくような日になっている。

主なPR / Issue: canopy [#1088](https://github.com/dowdiness/canopy/pull/1088), [#1094](https://github.com/dowdiness/canopy/pull/1094), [#1097](https://github.com/dowdiness/canopy/pull/1097), [#1098](https://github.com/dowdiness/canopy/pull/1098), [#1100](https://github.com/dowdiness/canopy/pull/1100), [#1101](https://github.com/dowdiness/canopy/pull/1101), [#1104](https://github.com/dowdiness/canopy/pull/1104), [#1105](https://github.com/dowdiness/canopy/pull/1105), [#1106](https://github.com/dowdiness/canopy/pull/1106), [#1107](https://github.com/dowdiness/canopy/pull/1107) / loom [#813](https://github.com/dowdiness/loom/pull/813), [#814](https://github.com/dowdiness/loom/pull/814), [#815](https://github.com/dowdiness/loom/pull/815), [#816](https://github.com/dowdiness/loom/pull/816), [#817](https://github.com/dowdiness/loom/pull/817), [#818](https://github.com/dowdiness/loom/pull/818), [#819](https://github.com/dowdiness/loom/pull/819), [#820](https://github.com/dowdiness/loom/pull/820), [#822](https://github.com/dowdiness/loom/pull/822), [#824](https://github.com/dowdiness/loom/pull/824), [#825](https://github.com/dowdiness/loom/pull/825), [#831](https://github.com/dowdiness/loom/pull/831), [#832](https://github.com/dowdiness/loom/pull/832), [#833](https://github.com/dowdiness/loom/pull/833)

## 2026/8/2

### Canopy / Loomarkのdev hostとdom-boundary昇格

Loomとdiagnostic APIの依存更新を挟み、`dom-boundary`パッケージにtyped text controlsが入った。Rabbitaは0.14.1のmodel-firstコールバックへ追従し、canvasのmodule manifestも移行、CM6 diagnostic fixtureをeditor-adapterから分離する変更も続いた。Loomark側では、外部に公開しないテスト駆動用の最小ホスト環境——private single-mount dev host——と、raw previewのapplication trainが加わっている。生まれて1日のプロダクトに、もう専用の開発環境ができた形になる。

### loom / CommonMark完成後の性能作業

CommonMarkが固まったところで、視線が性能へ移った。Unicode表示幅とdiagnosticのアラインメント、link delimiter優先順位の解決漏れ、入れ子image構文の認定test、パーサー性能ステージのプロファイリングと続き、block再パース後もtoken bufferを保持する最適化、reference lookaheadをprefix-linearにする最適化で、1万行文書の性能envelope測定を完了させた。未解決local blockの専用コア追加とUnicode対応のdiagnostic pretty layoutを経て、この日の到達点は**reactive keyed MarkdownIR shell**になる。差分再パースの単位をkeyで安定させ、reactiveな更新に載せるための土台で、正しさを固めた翌日に速さへ向かうという流れが、そのままこの1日に収まっている。

主なPR / Issue: canopy [#1108](https://github.com/dowdiness/canopy/pull/1108), [#1109](https://github.com/dowdiness/canopy/pull/1109), [#1111](https://github.com/dowdiness/canopy/pull/1111), [#1113](https://github.com/dowdiness/canopy/pull/1113), [#1118](https://github.com/dowdiness/canopy/pull/1118), [#1119](https://github.com/dowdiness/canopy/pull/1119), [#1120](https://github.com/dowdiness/canopy/pull/1120) / loom [#834](https://github.com/dowdiness/loom/pull/834), [#835](https://github.com/dowdiness/loom/pull/835), [#837](https://github.com/dowdiness/loom/pull/837), [#840](https://github.com/dowdiness/loom/pull/840), [#841](https://github.com/dowdiness/loom/pull/841), [#842](https://github.com/dowdiness/loom/pull/842), [#844](https://github.com/dowdiness/loom/pull/844), [#849](https://github.com/dowdiness/loom/pull/849), [#850](https://github.com/dowdiness/loom/pull/850), [#851](https://github.com/dowdiness/loom/pull/851)

## 2026/8/3

### Canopy / Loomarkのローカルファースト設計とMarkdown編集の拡張

Loomarkに「ローカルファーストなドキュメント所有権」の設計方針が入った。出発点にあるのは、Loomarkは今、Markdown文書を編集できるが所有はできないという指摘だ。スナップショットが`{version, source, mode}`だけなので、復元すれば文字は戻るが、因果履歴もversionのfrontierも、他レプリカとの連続性も失われる。タブを閉じれば文書は事実上放棄されたことになる。ここから立てた原則は一行で済む——テキストではなく操作履歴を永続化する。スコープは単一文書・ローカルのみ・再起動からの復旧に絞り、ネットワーク同期やワークスペースカタログは明示的に除外した。

この設計はCodex（GPT-5）による3回の独立レビューを経ている。二度目のレビューはover-reductionを突いた。削りすぎていた4つの論点——checkpointがarchive metadataを運べないこと、document_idを今のうちに予約すべきこと、source+Versionだけでは弱すぎるacceptance oracleであること、10万操作のrestore上限が実は荷重を支えていたこと——が、指摘のたびに復元されている。設計を書いて終わりにもしていない。4つの主張は実際に動くprobeで検証し、アサーションを意図的に壊してから戻すという較正を通してから結果を信じ、3つの前提はdrift detector testとして固定した。プロダクトが2日目にして自分自身の設計原則を実測で裏づけている、というのはなかなかない話だ。

Markdown編集機能では、listとfenced codeの編集対応、Markdown façadeへのarchive surfaceの付与が続いた。リポジトリ全体のtopologyをmanifestから導出しissueへ紐づけるdocs整備も同日に入っている。

### loom / performance envelopeの仕上げ

equal-cardinalityなmode relexストレージのローカライズ、フルパース性能contractの修正、コードブロックごとの文書再構築を避ける最適化、custom tracker constructorを使うprojectionリファクタが続き、Incrをv0.15.0へ更新、keyed projection attachmentの追加で締めくくった。8/2に据えたreactive keyed shellが、この日で実用段階に届いている。

### incr / v0.15.0リリース

typed-sheetアプリケーションの所有権を深くする整理と、Watch上のrooted readerを統合する変更を経て、scope-owned post-GC maintenanceを設計・実装し、0.15.0をリリースした。

主なPR / Issue: canopy [#1123](https://github.com/dowdiness/canopy/pull/1123), [#1126](https://github.com/dowdiness/canopy/pull/1126), [#1127](https://github.com/dowdiness/canopy/pull/1127), [#1129](https://github.com/dowdiness/canopy/pull/1129) / incr [#433](https://github.com/dowdiness/incr/pull/433), [#434](https://github.com/dowdiness/incr/pull/434), [#446](https://github.com/dowdiness/incr/pull/446), [#447](https://github.com/dowdiness/incr/pull/447), [#448](https://github.com/dowdiness/incr/pull/448) / loom [#852](https://github.com/dowdiness/loom/pull/852), [#853](https://github.com/dowdiness/loom/pull/853), [#855](https://github.com/dowdiness/loom/pull/855), [#858](https://github.com/dowdiness/loom/pull/858), [#859](https://github.com/dowdiness/loom/pull/859), [#860](https://github.com/dowdiness/loom/pull/860)

## 2026/8/4

### Canopy / Loomarkのブロックエディタ実働とワークスペース大規模再編

Loomarkに、Markdownの履歴を意識したcommit receipt機能が入り、superseded——上書きされた——block focus effectを無視する修正を経て、ブロックエディタのprototypeが完成した。RUIによるMarkdownプレビューのレンダリング、インタラクティブなchromeとtest hookの分離、本番デモに合わせたブラウザchrome、ブロック整形ツールバー、一連のブロックエディタ操作が、この日で一通り揃っている。8/1に生まれてから4日目にして、もう触って動かせるものになった。

同じ日、Canopy本体のワークスペースディレクトリを大規模に再編した。foundation・runtime・Rabbita・domain・editor adapter・application slice・primary Canopyモジュールを、依存境界を強制するテストを添えながら順番に配置し直す、22コミットがかりの移動になっている。7月末のLoomark誕生とdom-boundary・js_ffiの独立が先にあり、モノレポとしての形をあとから追いかけて整え直しているように見える。svg-dsl・graphviz・loom・event-graph-walker（v0.7.1）のsubmodule更新も同日に続いた。

### js_engine / スタックセーフ化キャンペーンが本格化

7/29に決めた「部分トランポリン」方針が、7/31から大規模なキャンペーンへ発展している。GitHub issueごとに`codex/issue-NNN`というブランチが切られ、それぞれ別のコーディングエージェントが担当してmainへマージされていく形で、4日間で約300コミットが積み上がった。狙いは、インタプリタ全体をネイティブ再帰からスタックセーフな明示的継続へ置き換えること——ゲスト側の再帰がどれだけ深くても、ホスト呼び出し深さは一定に保つという不変条件を、静的検証からランタイム実行そのものへ広げている。

対象の広がり方には日ごとの筋がある。7/31は直接呼び出しのactivation深さ制御と、それをgetter・proxy rootにも適用する修正、test262のタイムアウトgate強化。8/1はordinary callのtrampoline化、spread call、receiverアクセス、bytecodeのソース情報保持、bounded evalファサードの受け入れテスト。8/2はconstructor呼び出し（spread込み）のsuspend化、`Function.prototype.call`の反復化、timer・event loopのbounded checkpoint、comma式の直接評価の反復化。8/3はordinary constructorのdispatchと、その根拠になるソースメタデータの検証・確定。8/4は`Array.prototype.map`の直接コールバック呼び出しと`Function.prototype.apply`のsealed forwardingまで届いた。呼び出し経路を1つずつ順番に潰しているのは分かるが、経路の総数がどれだけあり、あと何日でここを抜けるのかは、この4日間の記録だけでは見えてこない。

手順のほうは一貫している。まずsuspend可能にし、先に失敗するテストを書き（RED）、それから直す。境界と契約はそのつど`docs(stack-safety)`で文書化される。

### loom

egglogとevent-graph-walkerがそれぞれincr v0.15.0・v0.7.1へ追従し、MarkdownIRの網羅的なviewとsource-awareなviewが公開された。

主なPR / Issue: canopy [#1142](https://github.com/dowdiness/canopy/pull/1142), [#1143](https://github.com/dowdiness/canopy/pull/1143), [#1144](https://github.com/dowdiness/canopy/pull/1144), [#1146](https://github.com/dowdiness/canopy/pull/1146), [#1147](https://github.com/dowdiness/canopy/pull/1147), [#1148](https://github.com/dowdiness/canopy/pull/1148), [#1149](https://github.com/dowdiness/canopy/pull/1149) / loom [#864](https://github.com/dowdiness/loom/pull/864), [#865](https://github.com/dowdiness/loom/pull/865), [#866](https://github.com/dowdiness/loom/pull/866), [#867](https://github.com/dowdiness/loom/pull/867)

## 2026/8/5

### Canopy / Loomarkのアプリ分割と応答性

8/4に22コミットのワークスペース再編を終えた翌日、Loomarkはアプリ構造の分割へ入った。responsibilitiesを責務ごとに分け、レスポンシブなsplit previewを備えた2画面レイアウトが加わっている。prototypeから1日で、実際のアプリ配置を考え始める段階へ移った形になる。

### js_engine / direct-return call chainの整列

sealed leaf call argumentの束縛（#814）から始まり、exact direct-return recursionのトランポリン化（#810）、sealed leaf経由のdirect return（#812）と、呼び出しチェーンを1本に束ねる作業が続いた。個別の呼び出し経路をsuspend可能にしてから、それらをdirect returnという単一の計画へ降ろしていく——8/4までに`Array.prototype.map`や`Function.prototype.apply`の個別経路を片付け始めた作業が、ここでは「チェーン全体を1つのplanに下ろす」という形に昇華されている。

主なPR / Issue: canopy [#1160](https://github.com/dowdiness/canopy/pull/1160), [#1161](https://github.com/dowdiness/canopy/pull/1161) / js_engine [#810](https://github.com/dowdiness/js_engine/pull/810), [#812](https://github.com/dowdiness/js_engine/pull/812), [#814](https://github.com/dowdiness/js_engine/pull/814)

## 2026/8/6

### Canopy / Loomarkのスタンドアロン化とドキュメントアーカイブ

この日、LoomarkはWarren——Canopyのリポジトリ管理インフラ——を経由してRabbitaのスタンドアロンアプリケーションとしてshipされた（#1177）。8/1に生まれて5日目にして、Canopyのサブパッケージではなく独立したデプロイ対象になった形になる。これに合わせてhandoffのdocsもstandalone方向へ揃え直した。

同時にドキュメントアーカイブの仕組みが入った。versioned envelope（#1174）で保存形式を定義し、explicit open limits（#1175）で開く範囲を制限し、per-admission text history limits（#1173）でCRDT側の履歴サイズに歯止めをかけ、最後にローカルアーカイブの永続化（#1178）で閉じた。8/3に設計した「テキストではなく操作履歴を永続化する」原則が、この日で実際に動く保存機構になった。Cursor Cloud環境のsetup docs（#1031）も同日に追加され、マルチエージェント開発の土台も整えられている。

### js_engine / direct-return chainからresult-fed pipelineへ

ordered direct-return argumentsの所有権（#816）、recursive resultsのsealed ordinary callへの供給（#818）、sealed direct-return wrapper rootsのadmission（#820）と進み、mixed ordered direct-return arguments（#821）、direct-return call chainsを1つのowned planにlowerする変更（#824）を経て、result-fed call pipelineの抽出（#826）で締めくくった。個々の呼び出し経路を片付ける段階から、呼び出しチェーンを1本のpipelineとして扱う段階へ移った日で、8/5のdirect-return整列がpipelineという概念に育っている。

主なPR / Issue: canopy [#1031](https://github.com/dowdiness/canopy/pull/1031), [#1173](https://github.com/dowdiness/canopy/pull/1173), [#1174](https://github.com/dowdiness/canopy/pull/1174), [#1175](https://github.com/dowdiness/canopy/pull/1175), [#1177](https://github.com/dowdiness/canopy/pull/1177), [#1178](https://github.com/dowdiness/canopy/pull/1178) / js_engine [#816](https://github.com/dowdiness/js_engine/pull/816), [#818](https://github.com/dowdiness/js_engine/pull/818), [#820](https://github.com/dowdiness/js_engine/pull/820), [#821](https://github.com/dowdiness/js_engine/pull/821), [#824](https://github.com/dowdiness/js_engine/pull/824), [#826](https://github.com/dowdiness/js_engine/pull/826)

## 2026/8/7

### Canopy / LoomarkのFunctional Core分解とCloudflareデプロイ整備

Loomarkのupdate_modelが、この日で大幅に分解された。commit classificationを純粋な決定コアとして抽出し（#1182）、2つあったcommit shellを1つのtransaction coreに統一し（#1183）、app state nodeから最新のrendered modelを読むようにして（#1184）、deferred-effect決定を純粋関数として切り出し（#1185）、update_model自体をconcern別のsub-dispatcherに分解した（#1186）。editing mode gateとprobe resultの固定（#1187）も同日に入っている。8/3の「Functional Core, Imperative Shell」設計方針が、4日目にして実際のコード構造に反映された日だ。

パフォーマンス面ではRaw input transactionのcoalesce（#1189）が入り、レイテンシの低い入力パスができた。standalone editorのレイアウト整備（#1181）も同じ日。言語面ではgeneric language SPIの深層化（#1190）で、言語プラグインの抽象化境界が広がった。

Cloudflareデプロイは同日中に4コミットかけて整えた。vendored graphvizパスの参照修正、namespaced canvas artifactsのsymlink、prosemirrorからapps/web moonbit pluginへのポインタ変更、loomark wrangler configのリポジトリルート外への退避、そしてデプロイトリガー。独立したRabbitaアプリとしてshipするには、CIとデプロイのパイプ自体も一緒に直す必要があった。

### js_engine / activation受信準備とreceiver dispatch

direct-return programのatomic seal（#829）、changing receiver activationsのdispatch（#832）、Diago restoration readinessのintegration test（#830）、receiver-call registry ownershipのリファクタ（#834）と、activation graphの呼び出し元側に焦点が移った。8/6にresult-fed pipelineを抽出したのが、8/7にはreceiver dispatchと呼び出し元registryの整理に繋がっている。pipelineを作ったあとは、そのpipelineに何を流すかの型付けに入る、という自然な流れだ。

主なPR / Issue: canopy [#1181](https://github.com/dowdiness/canopy/pull/1181), [#1182](https://github.com/dowdiness/canopy/pull/1182), [#1183](https://github.com/dowdiness/canopy/pull/1183), [#1184](https://github.com/dowdiness/canopy/pull/1184), [#1185](https://github.com/dowdiness/canopy/pull/1185), [#1186](https://github.com/dowdiness/canopy/pull/1186), [#1187](https://github.com/dowdiness/canopy/pull/1187), [#1189](https://github.com/dowdiness/canopy/pull/1189), [#1190](https://github.com/dowdiness/canopy/pull/1190) / js_engine [#829](https://github.com/dowdiness/js_engine/pull/829), [#830](https://github.com/dowdiness/js_engine/pull/830), [#832](https://github.com/dowdiness/js_engine/pull/832), [#834](https://github.com/dowdiness/js_engine/pull/834)

## 2026/8/8

### Canopy / Loomarkのレイテンシとエディタ信頼性

LoomarkのRaw Markdown入力パスでtyping latency削減（#1195）が入り、モバイルページのframe简化（#1192）とBlock DOM収束のE2Eテスト（#1193）が続いた。エディタ側ではparser failure時にもMarkdownを保持する修正（#1194）が入り、coreのreconcile adapter3つを1つのfallback realizerに統一するリファクタ（#1197）、edit application runtimeの統一（#1198）と、構造編集の信頼性を高める方向に重心が移った。8/7のFunctional Core分解がコアの信頼性を高め、8/8はその上で入力の速さと壊れにくさを同時に追求している。

### js_engine / tree activation graphの構築

result-fed call-chain admissionの一般化（#835）から始まり、executor child abrupt completionsの配送（#838）を経て、resumable tree activation frame（#840）、executor activation admission evidenceの型付け（#842）、tree admission graphの反復化（#843）、multi-call activation graphのclose（#845）まで辿り着いた。個々の呼び出し経路をsuspend可能にする段階を超えて、呼び出しの「木」をactivation graphとして明示的に構築し、そのgraphを反復的に辿れるようにした。ツリー構造のactivationは、JSの動的dispatchが持つ「どの経路で呼ばれるか分からない」性質を、静的なgraphに落として扱う試みと言える。

主なPR / Issue: canopy [#1192](https://github.com/dowdiness/canopy/pull/1192), [#1193](https://github.com/dowdiness/canopy/pull/1193), [#1194](https://github.com/dowdiness/canopy/pull/1194), [#1195](https://github.com/dowdiness/canopy/pull/1195), [#1197](https://github.com/dowdiness/canopy/pull/1197), [#1198](https://github.com/dowdiness/canopy/pull/1198) / js_engine [#835](https://github.com/dowdiness/js_engine/pull/835), [#838](https://github.com/dowdiness/js_engine/pull/838), [#840](https://github.com/dowdiness/js_engine/pull/840), [#842](https://github.com/dowdiness/js_engine/pull/842), [#843](https://github.com/dowdiness/js_engine/pull/843), [#845](https://github.com/dowdiness/js_engine/pull/845)

## 2026/8/9

### Canopy / LoomarkのRaw選択とPortable Projection / CodeMirror増分適用

Loomarkはstale Block inputのreject（#1199）から1日が始まった。前日の編集信頼性作業が「不正な入力を弾く」という防御に繋がり、empty previous mergeのsentinel削除（#1201）、fatal stateでのBlock input clear（#1202）と防御層が積み上がった。

構造側ではportable projection core（#1203）、portable commit transaction（#1205）、portableなsnapshot/receipt投影（#1208）と、projectionの可搬性を高める3つの変更が入った。commit receiptの仕組みをprojection層に引き下げ、どこからでも同じtransactionを使えるようにした形だ。CRDT text versionのキャッシング（#1200）で編集時の性能も押さえている。

選択（selection）まわりも同日に進んだ。directed Raw selectionsの保持（#1206）に続き、pre-input Raw selectionsのpairing（#1212）が入り、入力の前後でselection状態が途切れないようになった。IME composition inputの保持（#1217）は、日本語入力のようなcomposition stateが必要なケースでもRaw selectionが崩れないことを意味する。

Canopy本体では、CodeMirrorのlocal delta増分適用（#1213）、structure drag-drop E2Eのmove contract整合（#1216）、LetDef行のmoveによる重複排除（#1207）、post-commit parser failureのテスト追加（#1209）、Block input merge rejectionのE2E await（#1215）と、エディタ全体のcontract整備が幅広い範囲で並んだ。

### js_engine / member callのadmissionとv0.8.0リリース

result-fed tree callsのadmission（#847）、static own-data member calls（#848）、literal-computed member callsの正規化（#850）と、オブジェクトのメンバーアクセスに対するactivation graph適用が3段階で進んだ。practical activation frontierの記録（#859）、legacy callからのadmitted callee entry（#860）を経て、**v0.8.0がリリースされた**（#605）。

スタックセーフ化キャンペーン開始から10日。direct-return call chainの1つのleafをsealedにするところから始まった作業が、activation graph、tree activation、member callと段階を経て広がり、この日で1つの区切りをつけた。

主なPR / Issue: canopy [#1199](https://github.com/dowdiness/canopy/pull/1199), [#1200](https://github.com/dowdiness/canopy/pull/1200), [#1201](https://github.com/dowdiness/canopy/pull/1201), [#1202](https://github.com/dowdiness/canopy/pull/1202), [#1203](https://github.com/dowdiness/canopy/pull/1203), [#1205](https://github.com/dowdiness/canopy/pull/1205), [#1206](https://github.com/dowdiness/canopy/pull/1206), [#1207](https://github.com/dowdiness/canopy/pull/1207), [#1208](https://github.com/dowdiness/canopy/pull/1208), [#1209](https://github.com/dowdiness/canopy/pull/1209), [#1212](https://github.com/dowdiness/canopy/pull/1212), [#1213](https://github.com/dowdiness/canopy/pull/1213), [#1215](https://github.com/dowdiness/canopy/pull/1215), [#1216](https://github.com/dowdiness/canopy/pull/1216), [#1217](https://github.com/dowdiness/canopy/pull/1217) / js_engine [#847](https://github.com/dowdiness/js_engine/pull/847), [#848](https://github.com/dowdiness/js_engine/pull/848), [#850](https://github.com/dowdiness/js_engine/pull/850), [#859](https://github.com/dowdiness/js_engine/pull/859), [#860](https://github.com/dowdiness/js_engine/pull/860), [#605](https://github.com/dowdiness/js_engine/pull/605)

## 2026/8/10

### Canopy / LoomarkのRaw入力仕上げ

v0.8.0リリースの翌日、視線は再びLoomarkの入力パスに戻った。CodeMirror deltaをsource versionでガードし（#1218）、render raceを跨いでraw inputを保持し（#1222）、same-character Raw insertionの修正（#1227）、grapheme boundaryのvalidation毎再利用（#1229）と、Raw Markdown入力の細かい信頼性と性能を追い込んだ。Raw frontier e2eの決定性テスト（#1226）と、Raw input phaseのプロファイル記録（#1220）で、測定できる形に固めている。

8/7のFunctional Core分解 → 8/8のレイテンシ・信頼性 → 8/9のportable projectionとRaw選択 → 8/10のRaw入力仕上げと、Loomarkはstandalone化から1週間で「触って使えるエディタ」から「入力に耐えるエディタ」へ段階を上げている。

### js_engine / literal-computed member callのsuspend

v0.8.0の直後、literal computed getter readsのsuspend（#862）とliteral-computed member callsのsuspend（#865）が入った。8/9に正規化まで終えたliteral-computed member callを、今度はsuspend可能にしてactivation graphに載せ直している。releaseしたばかりのコードに即座に次の変更が入るのは、このキャンペーンがまだ進行中であることを示している。

主なPR / Issue: canopy [#1218](https://github.com/dowdiness/canopy/pull/1218), [#1220](https://github.com/dowdiness/canopy/pull/1220), [#1222](https://github.com/dowdiness/canopy/pull/1222), [#1226](https://github.com/dowdiness/canopy/pull/1226), [#1227](https://github.com/dowdiness/canopy/pull/1227), [#1229](https://github.com/dowdiness/canopy/pull/1229) / js_engine [#862](https://github.com/dowdiness/js_engine/pull/862), [#865](https://github.com/dowdiness/js_engine/pull/865)

## PR索引

日付ごとのPR / Issue一覧。GitHub上の詳細への索引。

<details>
<summary>8月第1週（8/1〜8/4）</summary>

### 2026/8/1

**CommonMark完成 / Loomark誕生**

canopy [#1088](https://github.com/dowdiness/canopy/pull/1088), [#1094](https://github.com/dowdiness/canopy/pull/1094), [#1097](https://github.com/dowdiness/canopy/pull/1097), [#1098](https://github.com/dowdiness/canopy/pull/1098), [#1100](https://github.com/dowdiness/canopy/pull/1100), [#1101](https://github.com/dowdiness/canopy/pull/1101), [#1104](https://github.com/dowdiness/canopy/pull/1104), [#1105](https://github.com/dowdiness/canopy/pull/1105), [#1106](https://github.com/dowdiness/canopy/pull/1106), [#1107](https://github.com/dowdiness/canopy/pull/1107) / loom [#813](https://github.com/dowdiness/loom/pull/813), [#814](https://github.com/dowdiness/loom/pull/814), [#815](https://github.com/dowdiness/loom/pull/815), [#816](https://github.com/dowdiness/loom/pull/816), [#817](https://github.com/dowdiness/loom/pull/817), [#818](https://github.com/dowdiness/loom/pull/818), [#819](https://github.com/dowdiness/loom/pull/819), [#820](https://github.com/dowdiness/loom/pull/820), [#822](https://github.com/dowdiness/loom/pull/822), [#824](https://github.com/dowdiness/loom/pull/824), [#825](https://github.com/dowdiness/loom/pull/825), [#831](https://github.com/dowdiness/loom/pull/831), [#832](https://github.com/dowdiness/loom/pull/832), [#833](https://github.com/dowdiness/loom/pull/833)

### 2026/8/2

**Loomark dev host / MarkdownIR性能作業**

canopy [#1108](https://github.com/dowdiness/canopy/pull/1108), [#1109](https://github.com/dowdiness/canopy/pull/1109), [#1111](https://github.com/dowdiness/canopy/pull/1111), [#1113](https://github.com/dowdiness/canopy/pull/1113), [#1118](https://github.com/dowdiness/canopy/pull/1118), [#1119](https://github.com/dowdiness/canopy/pull/1119), [#1120](https://github.com/dowdiness/canopy/pull/1120) / loom [#834](https://github.com/dowdiness/loom/pull/834), [#835](https://github.com/dowdiness/loom/pull/835), [#837](https://github.com/dowdiness/loom/pull/837), [#840](https://github.com/dowdiness/loom/pull/840), [#841](https://github.com/dowdiness/loom/pull/841), [#842](https://github.com/dowdiness/loom/pull/842), [#844](https://github.com/dowdiness/loom/pull/844), [#849](https://github.com/dowdiness/loom/pull/849), [#850](https://github.com/dowdiness/loom/pull/850), [#851](https://github.com/dowdiness/loom/pull/851)

### 2026/8/3

**Loomarkローカルファースト設計 / incr v0.15.0**

canopy [#1123](https://github.com/dowdiness/canopy/pull/1123), [#1126](https://github.com/dowdiness/canopy/pull/1126), [#1127](https://github.com/dowdiness/canopy/pull/1127), [#1129](https://github.com/dowdiness/canopy/pull/1129) / incr [#433](https://github.com/dowdiness/incr/pull/433), [#434](https://github.com/dowdiness/incr/pull/434), [#446](https://github.com/dowdiness/incr/pull/446), [#447](https://github.com/dowdiness/incr/pull/447), [#448](https://github.com/dowdiness/incr/pull/448) / loom [#852](https://github.com/dowdiness/loom/pull/852), [#853](https://github.com/dowdiness/loom/pull/853), [#855](https://github.com/dowdiness/loom/pull/855), [#858](https://github.com/dowdiness/loom/pull/858), [#859](https://github.com/dowdiness/loom/pull/859), [#860](https://github.com/dowdiness/loom/pull/860)

### 2026/8/4

**Loomarkブロックエディタ完成 / ワークスペース再編 / js_engineスタックセーフ化**

canopy [#1142](https://github.com/dowdiness/canopy/pull/1142), [#1143](https://github.com/dowdiness/canopy/pull/1143), [#1144](https://github.com/dowdiness/canopy/pull/1144), [#1146](https://github.com/dowdiness/canopy/pull/1146), [#1147](https://github.com/dowdiness/canopy/pull/1147), [#1148](https://github.com/dowdiness/canopy/pull/1148), [#1149](https://github.com/dowdiness/canopy/pull/1149) / loom [#864](https://github.com/dowdiness/loom/pull/864), [#865](https://github.com/dowdiness/loom/pull/865), [#866](https://github.com/dowdiness/loom/pull/866), [#867](https://github.com/dowdiness/loom/pull/867)

</details>

<details>
<summary>8月第2週（8/5〜8/10）</summary>

### 2026/8/5

**Loomarkアプリ分割 / js_engine direct-return recursion**

canopy [#1160](https://github.com/dowdiness/canopy/pull/1160), [#1161](https://github.com/dowdiness/canopy/pull/1161) / js_engine [#810](https://github.com/dowdiness/js_engine/pull/810), [#812](https://github.com/dowdiness/js_engine/pull/812), [#814](https://github.com/dowdiness/js_engine/pull/814)

### 2026/8/6

**Loomarkスタンドアロン化 / ドキュメントアーカイブ / js_engine direct-return chain**

canopy [#1031](https://github.com/dowdiness/canopy/pull/1031), [#1173](https://github.com/dowdiness/canopy/pull/1173), [#1174](https://github.com/dowdiness/canopy/pull/1174), [#1175](https://github.com/dowdiness/canopy/pull/1175), [#1177](https://github.com/dowdiness/canopy/pull/1177), [#1178](https://github.com/dowdiness/canopy/pull/1178) / js_engine [#816](https://github.com/dowdiness/js_engine/pull/816), [#818](https://github.com/dowdiness/js_engine/pull/818), [#820](https://github.com/dowdiness/js_engine/pull/820), [#821](https://github.com/dowdiness/js_engine/pull/821), [#824](https://github.com/dowdiness/js_engine/pull/824), [#826](https://github.com/dowdiness/js_engine/pull/826)

### 2026/8/7

**Loomark Functional Core分解 / Cloudflareデプロイ整備 / js_engine activation受信**

canopy [#1181](https://github.com/dowdiness/canopy/pull/1181), [#1182](https://github.com/dowdiness/canopy/pull/1182), [#1183](https://github.com/dowdiness/canopy/pull/1183), [#1184](https://github.com/dowdiness/canopy/pull/1184), [#1185](https://github.com/dowdiness/canopy/pull/1185), [#1186](https://github.com/dowdiness/canopy/pull/1186), [#1187](https://github.com/dowdiness/canopy/pull/1187), [#1189](https://github.com/dowdiness/canopy/pull/1189), [#1190](https://github.com/dowdiness/canopy/pull/1190) / js_engine [#829](https://github.com/dowdiness/js_engine/pull/829), [#830](https://github.com/dowdiness/js_engine/pull/830), [#832](https://github.com/dowdiness/js_engine/pull/832), [#834](https://github.com/dowdiness/js_engine/pull/834)

### 2026/8/8

**Loomarkレイテンシ削減 / エディタ信頼性 / js_engine tree activation graph**

canopy [#1192](https://github.com/dowdiness/canopy/pull/1192), [#1193](https://github.com/dowdiness/canopy/pull/1193), [#1194](https://github.com/dowdiness/canopy/pull/1194), [#1195](https://github.com/dowdiness/canopy/pull/1195), [#1197](https://github.com/dowdiness/canopy/pull/1197), [#1198](https://github.com/dowdiness/canopy/pull/1198) / js_engine [#835](https://github.com/dowdiness/js_engine/pull/835), [#838](https://github.com/dowdiness/js_engine/pull/838), [#840](https://github.com/dowdiness/js_engine/pull/840), [#842](https://github.com/dowdiness/js_engine/pull/842), [#843](https://github.com/dowdiness/js_engine/pull/843), [#845](https://github.com/dowdiness/js_engine/pull/845)

### 2026/8/9

**Loomark Raw選択 / Portable Projection / CodeMirror増分 / js_engine v0.8.0リリース**

canopy [#1199](https://github.com/dowdiness/canopy/pull/1199), [#1200](https://github.com/dowdiness/canopy/pull/1200), [#1201](https://github.com/dowdiness/canopy/pull/1201), [#1202](https://github.com/dowdiness/canopy/pull/1202), [#1203](https://github.com/dowdiness/canopy/pull/1203), [#1205](https://github.com/dowdiness/canopy/pull/1205), [#1206](https://github.com/dowdiness/canopy/pull/1206), [#1207](https://github.com/dowdiness/canopy/pull/1207), [#1208](https://github.com/dowdiness/canopy/pull/1208), [#1209](https://github.com/dowdiness/canopy/pull/1209), [#1212](https://github.com/dowdiness/canopy/pull/1212), [#1213](https://github.com/dowdiness/canopy/pull/1213), [#1215](https://github.com/dowdiness/canopy/pull/1215), [#1216](https://github.com/dowdiness/canopy/pull/1216), [#1217](https://github.com/dowdiness/canopy/pull/1217) / js_engine [#847](https://github.com/dowdiness/js_engine/pull/847), [#848](https://github.com/dowdiness/js_engine/pull/848), [#850](https://github.com/dowdiness/js_engine/pull/850), [#859](https://github.com/dowdiness/js_engine/pull/859), [#860](https://github.com/dowdiness/js_engine/pull/860), [#605](https://github.com/dowdiness/js_engine/pull/605)

### 2026/8/10

**Loomark Raw入力仕上げ / js_engine literal-computed suspend**

canopy [#1218](https://github.com/dowdiness/canopy/pull/1218), [#1220](https://github.com/dowdiness/canopy/pull/1220), [#1222](https://github.com/dowdiness/canopy/pull/1222), [#1226](https://github.com/dowdiness/canopy/pull/1226), [#1227](https://github.com/dowdiness/canopy/pull/1227), [#1229](https://github.com/dowdiness/canopy/pull/1229) / js_engine [#862](https://github.com/dowdiness/js_engine/pull/862), [#865](https://github.com/dowdiness/js_engine/pull/865)

</details>
