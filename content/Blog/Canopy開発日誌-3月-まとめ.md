---
title: Canopy開発日誌-3月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:35:00+09:00
modified: 2026-07-16T10:35:00+09:00
---

# Canopy開発日誌-3月-まとめ

2分で読める月次まとめ。日々の詳細は[[Canopy開発日誌-3月|通常版の日誌]]を、英語版は[[Canopy-March-2026-Highlights|Highlights (English)]]を参照。

> Canopyは、ソースコードを文字列ではなく構造（IR）として扱うエディタです。文字列を正として保ちつつ、そこから導出したプログラムの意味単位を直接操作することで、安全な構造編集やAI・複数人との協調作業がしやすくなります。詳しくは[[Canopyとは]]。

## 1. parserがloomになる

12月からCanopy内にあったparserが、独立リポジトリ`dowdiness/loom`として切り出された。CanopyはこのタイミングでloomのAPIへ移行している。これが、後に複数のライブラリへと分岐していく最初の一歩になった。

## 2. SyncEditorが編集を統一

`ParsedEditor`が`SyncEditor`へ置き換わり、編集・undo/redo・同期・presenceが一つのeditor abstractionにまとまった。以後のLambda名前解決からJSON/block editorまで、すべてこの上に積み上がっていく。

## 3. RabbitaとIdeal editorが急成長

Rabbitaベースのprojectional editorに、この月ようやく本物のUI基盤ができた。性能問題を潰し、mobile layout・design tokens・tree pane navigationが入った。16の構造編集アクションも実装され、テキストではなくプログラムの構造を直接動かすための「動詞」が初めて出揃った。

## 4. CRDTの性能改善

`FugueTree`/`traverse_tree`をiterative化し、order-treeを導入。event-graph-walkerのtwo-count retreat最適化で走査が17.7倍速くなった。地味だが、構造編集を「気づかないほど速い」状態に保つための作業。

## 5. WebSocketでのリアルタイム協調編集

transport layer、relay server、sync recovery protocol、ephemeral store v2が実装され、複数人が同じ文書をライブ編集するための本格的なインフラが初めて整った。

## 6. framework抽出と新エディタ

`ProjNode[T]`のgeneric化、`TreeNode`/`Renderable` traitの導入で、Canopy固有のコードからframework/coreパッケージが切り出され始めた。その土台の上に、block editor、JSON editor、AST Zipper、Containerの最初のフェーズが立ち上がった。

---

3月は、その後1年を通して引き継がれるほぼすべての流れの起点になった月だった――独立ライブラリとしてのloom、汎用的なprojection framework、Lambdaにとどまらないblockベースのエディタ群。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-3月|3月の日誌]]
