---
title: Canopy開発日誌-8月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-08-04T22:54:40+09:00
modified: 2026-08-04T23:14:11+09:00
---

# Canopy開発日誌-8月

2026年8月のCanopy開発ログ（日次記録）。月の要約は[[Canopy開発日誌-8月-まとめ|8月-まとめ]]（月初4日分、随時更新）。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

## 今月の大きな流れ

進行中の月のため、以下は日付見出しによる作業記録（8/4まで）。PR番号の一覧は文末の[PR索引](#pr索引)にある。7月の記録は[[Canopy開発日誌-7月|7月の日誌]]・[[Canopy開発日誌-7月-まとめ|7月-まとめ]]。

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
