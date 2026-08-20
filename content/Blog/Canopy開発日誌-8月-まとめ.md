---
title: Canopy開発日誌-8月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-08-04T23:10:00+09:00
modified: 2026-08-19T15:30:00+09:00
---

# Canopy開発日誌-8月-まとめ

7月末に完成した診断基盤とCommonMark実装を土台に、Loomarkが生まれ、js_engineはbytecode VMを本格構築し、Incr NextがK1を完了した。日々の記録は[[Canopy開発日誌-8月|通常版の日誌]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. Loomark——誕生から2週間で「どこが遅いか分かる」エディタへ

8/1に「Loomark Markdown editorのための純粋なアプリケーションコア」として立ち上がった。7/15にCanopy本体から切り出した`js_ffi`・`dom_boundary`が、初めて「Canopyを土台にした別プロダクト」を生んだ形になる。

最初の4日でtyped Markdown foundations → private dev host → ローカルファーストなドキュメント所有権の設計 → ブロックエディタprototypeと驚くほどの速さで育った。所有権の設計は「Loomarkは今、Markdown文書を編集できるが所有はできない」という問題意識から「テキストではなく操作履歴を永続化する」という1行の原則を立て、Codex（GPT-5）による3回の独立レビューと動くprobeでの較正検証を経て固めた。8/6にはWarren経由でRabbitaスタンドアロンアプリとしてship。8/7にはupdate_modelをFunctional Core分解で純粋関数に切り分け、8/9にはportable projection coreとIME対応、CodeMirror増分delta適用まで入れた。

8/11以降は性能測定の深度が増した。applied-editの実測固定、projection placementの3経路比較、pre-frame bottleneckのChromium main-thread calibrated測定、post-commit persistenceのarchive/JSON準備への帰属特定、P3 archive reopenの実測改善と、「速い」ではなく「どこが遅いか分かる」エディタになっている。

## 2. Canvas authorityのMoonBit/Rabbita移管とEGW統合

8/12から1週間で、Canvasのインタラクション入力がほぼすべてTypeScriptからMoonBit/Rabbitaへ移った。spatial core抽出を皮切りに、pointer・wheel・edge selection・context-menu・edge renderとauthorityを次々と移管し、8/18にはedge keyboard activationの整合で閉じた。8/19には`dowdiness/skyline`というgeneric Bottom-Left integer packerを独立パッケージとして追加し、Canvasの「Arrange compactly」機能に使用。

同じ期間にEGW（event-graph-walker）の段階的統合も進んだ。8/12のGate A計画とCausal Authority ADRを起点に、P1→P2→P3と5日で文書から実装まで通過。P3 Text admission cutoverはproduction code変更ゼロのsubmodule bump統合で済み、8/18にはpost-admissionの性能帰属を実測で特定した。

## 3. js_engine——bytecode VMの静的検証とruntime resume

v0.8.0リリース（8/9）後、bytecode VMは2つの方向に同時に進んだ。

**静的検証とアーキテクチャ整理**：operand検証、frame shape検証、activation eligibility分類、environment slotカプセル化、destructuringのverified plan lowering、source AST所有権剥奪、CFG observation coverage証明、semantic lifecycle multigraph audit、executor candidates prepare/routeと、bytecode programを「sourceに依存しない独立した実行計画」として完全に閉じる道筋を2週間で辿り切った。

**runtime操作のresume**：addition・property deletion・iterable spread・CopyDataProperties・with binding resolution・forEach・Promise reactions・subtraction・relational operators・resolved binding reference・name update expressionsと、JS言語仕様の操作を1つずつruntime-owned managed activation seam経由に載せ替えた。8/18のFibonacci graduation evidenceでfully-Bytecode Fibonacci runtimeのRSS平坦性を実測し、VMの性能特性が測定可能な段階に入った。

Web Playground（8/13）も登場し、ブラウザでbytecode VMを直接試せる環境ができた。

## 4. incr——Incr Next K1を5日で完成

8/13のK0 product contract確定から8/17のK1.6 product-quality conformance完成まで、5日でK1の全マイルストーンを完了した。各ステップは「commission（docs-only）→ implement → accept → squash merge → 次のcommission」という厳密な交互手順で進んだ。

K1.1 no-memo kernel → K1.2 typed memo verification → K1.3 cycle detection → K1.4 typed cutoff and backdating → K1.5 proof loss → K1.6 product-quality conformanceと積み上げ、8/18に`dowdiness/incr_next`を既存`dowdiness/incr`とは独立したpre-1.0 sibling productとして採択した。

## 5. loomのCommonMark深化と開発基盤整備

8/1のCommonMark「完成」から3週間経っても、8/18にsource-bound semantic documentの確立、8/19にindented code・fenced-code indent strip・backtick fence info制約など5項目の適合追加と、実装はまだ深まっている。

開発基盤ではjust/lefthookのツールチェーン導入、pre-commitのpath-awareルーティング、submodule reachabilityのpre-push強制、MoonBit registry bootstrapのcache-aware化、release workflowのchangelog range明示化など、反復速度を支える整備が同じ期間に進んだ。

---

Loomarkは誕生から2週間で入力に耐え性能を測定できるエディタになり、Canvasは全authorityをMoonBit/Rabbitaへ移管し、js_engineはbytecode VMを独立した実行計画として完成させつつruntime操作を載せ替え、Incr NextはK1を5日で完了してsibling productとして巣立った。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [dowdiness/incr](https://github.com/dowdiness/incr) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-8月|8月の日誌]]
