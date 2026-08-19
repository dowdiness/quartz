---
title: Canopy開発日誌-8月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-08-04T23:10:00+09:00
modified: 2026-08-19T15:47:35+09:00
---

# Canopy開発日誌-8月-まとめ

7月末に完成した診断基盤とCommonMark実装を土台に、新しいプロダクトが生まれ、js_engineは大規模な書き換えキャンペーンに入り、Incr Nextという新しいカーネルが姿を現した。日々の記録は[[Canopy開発日誌-8月|通常版の日誌]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. loomがCommonMarkを完成させ、source-bound semantic documentへ進む

月初、7月から続いていたMarkdown実装が一気に完成した。raw HTMLポリシー、entity参照のデコード、autolinkとinline HTML、link/image参照の解決——CommonMarkのブロック・インライン構文がほぼ1日（8/1）で揃っている。

完成後は間を置かず性能作業へ移った。1万行文書の性能envelope測定、block再パース後のtoken buffer保持、reference lookaheadのprefix-linear化を経て、差分再パースの単位をkeyで安定させる「reactive keyed MarkdownIR shell」まで到達した。月の前半で「正しく動く」から「差分更新に載る」への切り替えを終えている。

8/18にはsource-bound semantic documentの確立（#914）で`MarkdownDocument`のseamとcanonical `parse_document`エントリポイントが追加され、8/19にはCommonMark適合の新しい項目——indented code、fenced-code indent strip、backtick fence info制約、info first word、over-indented ATX lazy continuation——が5つ同日に進んだ。8/1の「完成」から3週間経っても仕様書からの適合項目が続いており、CommonMark実装がまだ深まっている。

## 2. Loomarkという新しいプロダクトが生まれる

8/1、「Loomark Markdown editorのための純粋なアプリケーションコア」という説明を持つ独立パッケージが立ち上がった。7/15にCanopy本体から切り出した`js_ffi`・`dom_boundary`が、ここで初めて「Canopyを土台にした別プロダクト」を生む形で使われている。

その後の3日間で、typed Markdown foundations → private dev host → **ローカルファーストなドキュメント所有権の設計** → ブロックエディタprototypeと、驚くほどの速さで育った。所有権の設計は特に丁寧で、「Loomarkは今、Markdown文書を編集できるが所有はできない」という問題意識から出発し、「テキストではなく操作履歴を永続化する」という1行の原則を立て、Codex（GPT-5）による3回の独立レビューと、動くprobeによる検証（アサーションを意図的に壊してから戻す較正手順込み）を経て固めている。8/4にはRUIによるMarkdownプレビュー、ブロック整形ツールバー、ブロックエディタの一連のインタラクションが揃い、prototypeと呼べる段階に達した。

## 3. Loomarkが2週間で「触って使える」から「測定できる」エディタへ

8/5のアプリ分割から8/19まで、Loomarkはstandalone化から2週間で急速に成熟した。8/6にWarrenを経由したRabbitaスタンドアロンアプリとしてshipされ、同日にドキュメントアーカイブの保存機構が実装された。8/7にはupdate_modelのFunctional Core分解が入り、commit classification、deferred-effect、sub-dispatcherと純粋関数に切り分けた。8/8にはRaw Markdown入力のtyping latency削減とエディタ信頼性の向上。8/9にはportable projection coreとRaw選択の保持、IME composition inputの対応、CodeMirrorの増分delta適用。8/10にはRaw入力パスの性能仕上げとrender race対策。

8/11以降は性能測定の深度が増した。applied-edit回帰テストの実測固定（8/11）、startup corpus benchmarkの再現可能性確保（8/12）、projection placement rejectionの実測比較（8/15）、pre-frame response bottleneckのChromium main-thread calibrated特定（8/16）、post-commit persistenceのarchive/JSON準備への帰属（8/16）、P3 archive reopenの実測改善記録（8/18）と、「速い」ではなく「どこが遅いか分かる」エディタになっている。

## 4. CanvasがTypeScriptからMoonBit/Rabbitaへauthorityを移管する

8/12から8/18にかけて、Canvasのインタラクション入力がほぼすべてTypeScriptからMoonBit/Rabbitaへ移った。spatial coreの抽出（8/12）→ geometry boundaryの硬化（8/12）→ pointer ownership移管（8/15）→ wheel admission移管（8/16）→ edge selection authority移管（8/16）→ context-menu authority移管（8/16）→ edge render projection派生（8/17）→ edge layer Rabbitaレンダリング（8/17）→ edge keyboard activation整合（8/18）と、1週間でCanvasの全authorityがMoonBit/Rabbitaに集中した。8/19には`dowdiness/skyline`というgeneric Bottom-Left integer packerが独立パッケージとして追加され、Canvasの「Arrange compactly」機能に使われている。

## 5. EGW（event-graph-walker）がP1→P2→P3と段階的に統合される

8/12のEGW Gate A計画とCausal Authority residency ADRを起点に、P1 typed admission transition（8/16）、P2 Document admission projection計画（8/16）、P3 Text admission characterization（8/17）→ pending-limit policy boundary明確化（8/17）→ **P3 Text admission cutover統合**（8/17）と、5日で文書→実装の全段階を通過した。P3 cutoverはCanopy側のproduction code変更ゼロのsubmodule bump統合で、段階的に準備してきた成果がそのまま統合コストの低さに表れている。8/18にはpost-admission version expansionの性能帰属特定とremote admission phasesのattributingが進み、P3統合後の性能特性が実測で把握された。

## 6. incrがIncr Next K1を完成させ、pre-1.0 sibling productとして採択する

8/13のK0 product contract確定から8/17のK1.6 product-quality conformance完成まで、**5日でK1の全マイルストーンを完了**した。K1.1 no-memo kernel（8/13）→ K1.2 typed memo verification（8/14）→ K1.3 cycle detection（8/15）→ K1.4 typed cutoff and backdating（8/17）→ K1.5 proof loss（8/17）→ K1.6 product-quality conformance（8/17）→ K1 kernelのpre-1.0 sibling product採択（8/18）。各ステップは「commission（docs-only）→ implement → accept → squash merge → 次のcommission」という厳密な交互手順で進み、実装と文書が1日の中で交互に並ぶ日もある。`dowdiness/incr_next`は既存`dowdiness/incr`とは独立したモジュールとして、別のプロダクトラインで行くことが確定した。

## 7. js_engineがbytecode VMを本格的に構築し、runtime操作をresumeする

v0.8.0リリース（8/9）後のbytecode VMは、2つの方向に同時に進んだ。

1つ目は**静的検証とアーキテクチャ整理**。property mutation activation設計（8/11）→ indexed operands検証・reachable frame shapes検証（8/12）→ activation eligibility分類（8/12）→ environment slotカプセル化（8/13）→ lexical setup準備（8/13）→ destructuringのverified plan lowering（8/13）→ source AST所有権剥奪（8/14）→ CFG observation coverage証明（8/14）→ semantic lifecycle multigraph audit（8/14）→ executor candidates prepare・route（8/15）。bytecode programが「sourceに依存しない独立した実行計画」として完全に閉じるまでの道筋を、2週間で辿り切っている。

2つ目は**runtime操作のresume**。addition・property deletion・iterable spread・CopyDataProperties（8/16）→ with binding resolution・forEach・Promise reactions・subtraction・<= comparison（8/17）→ remaining relational operators・resolved binding reference retain（8/18）→ name update expressions（8/19）と、JS言語仕様の操作を1つずつruntime-owned managed activation seam経由に載せ替えている。8/18のFibonacci graduation evidenceで、fully-Bytecode Fibonacci runtimeが指数関数的elapsed timeに対しRSS平坦であることを実測し、VMの性能特性が測定可能な段階に入った。

Web Playground（8/13）もこの期間に登場し、CodeMirror diagnostics・API hover・seeded dungeon workload（8/13–14）と、ブラウザでbytecode VMを直接試せる環境ができた。

## 8. Canopy本体の開発基盤が整備される

justとlefthookのツールチェーン導入（8/12）、Lefthook pre-commitのpath-awareルーティング（8/13）、submodule reachabilityのpre-push強制（8/15）、MoonBit registry bootstrapのcache-aware化（8/16）、repository scriptsの簡素化（8/16）、release workflowのchangelog range明示化（8/14）と、開発の反復速度を支える基盤が同じ期間に整備された。

---

CommonMarkの完成とLoomarkの誕生が月初に連鎖し、js_engineがv0.8.0をリリースしてbytecode VMの静的検証とruntime resumeが並行して進み、Incr NextがK1を5日で完成させてsibling productとして採択された。Canvasは1週間で全authorityをMoonBit/Rabbitaへ移管し、EGWはP3 Text admission cutoverに到達した。Loomarkはstandalone化から2週間で「どこが遅いか分かる」エディタになっている。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [dowdiness/incr](https://github.com/dowdiness/incr) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-8月|8月の日誌]]
