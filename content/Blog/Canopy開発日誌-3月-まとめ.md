---
title: Canopy開発日誌-3月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:35:00+09:00
modified: 2026-07-16T15:50:00+09:00
---

# Canopy開発日誌-3月-まとめ

2月までにmonorepoの形はできていた。3月は、その中身を外に出し、編集を一本化し、UIを載せる——三つの動きが重なった月だ。日々の記録は[[Canopy開発日誌-3月|通常版の日誌]]、英語版は[[Canopy-March-2026-Highlights|Highlights (English)]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. parserがloomになる

12月からCanopy内にあったparserを、`dowdiness/loom`として独立リポジトリへ切り出した。CanopyはこのタイミングでloomのAPIへ移行している。

複数のライブラリへ分岐していく最初の一歩だ。のちのJSON・Markdown・JSX統合は、すべてこの分離の上に載る。

## 2. SyncEditorが編集を統一

`ParsedEditor`を`SyncEditor`へ置き換え、編集・undo/redo・同期・presenceを一つのeditor abstractionにまとめた。

それまでバラバラだった操作が一箇所に寄った。以後のLambda名前解決からJSON/block editorまで、ここに積み上がっていく。

## 3. RabbitaとIdeal editorが急成長

Rabbitaベースのprojectional editorに、ようやく本物のUI基盤ができた。性能問題を潰し、mobile layout、design tokens、tree pane navigationを入れた。

16の構造編集アクションも実装された。テキストを置換するのではなく、プログラムの構造を直接動かす操作が、初めて揃った月だ。

## 4. CRDTの性能改善

`FugueTree`/`traverse_tree`をiterative化し、order-treeを導入した。event-graph-walkerのtwo-count retreat最適化で、走査は17.7倍速くなった。

地味に見えるが、構造編集は「気づかないほど速い」状態でないと成立しない。遅さが表面化してから直すのは、もう手遅れに近い。

## 5. WebSocketでのリアルタイム協調編集

transport layer、relay server、sync recovery protocol、ephemeral store v2を実装した。1月の協調編集デモから一歩進み、複数人が同じ文書をライブ編集するインフラが本格的に整った。

## 6. framework抽出と新エディタ

`ProjNode[T]`のgeneric化、`TreeNode`/`Renderable` traitの導入で、Canopy固有のコードからframework/coreパッケージの切り出しが始まった。

その土台の上に、block editor、JSON editor、AST Zipper、Container Phase 1が立ち上がった。Lambdaだけのエディタではなく、複数の編集形態を支える骨格が見え始める。

---

独立ライブラリとしてのloom、汎用projection framework、Lambdaにとどまらないblockベースのエディタ群。4月のEditorProtocol統一も、この月の延長線上にある。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-3月|3月の日誌]]
