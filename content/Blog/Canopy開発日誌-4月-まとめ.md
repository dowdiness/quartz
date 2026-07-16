---
title: Canopy開発日誌-4月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:40:00+09:00
modified: 2026-07-16T15:55:00+09:00
---

# Canopy開発日誌-4月-まとめ

3月まで、CanopyはLambda中心のエディタに見えていた。4月に固まったのは、その形ではなく、言語を差し替えられる骨格だった。日々の記録は[[Canopy開発日誌-4月|通常版の日誌]]、英語版は[[Canopy-April-2026-Highlights|Highlights (English)]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. すべてのエディタ面がひとつのprotocolに

Ideal editorもprosemirror exampleも、それぞれ別の配線を持っていた。`EditorProtocol`経由に統一し、CM6Adapter/PMAdapterをadapter層として整理した。

どの面も同じ言葉でドキュメントモデルと話せるようになった。「ひとつの決め打ちエディタ」から「adapterの集合」へ、形が変わった。

## 2. pretty-printerがDOMとつながる

Wadler-Lindig pretty-printerは、それまでテキスト整形の道具だった。出力を`ViewNode`へ渡すbridgeを作り、HTML syntax highlightingとつなげた。

整形結果が、そのまま描画パイプラインに入る。エディタの見た目と、ソースの構造が同じ経路で扱われるようになった。

## 3. Markdownが本格的なblock editorに

Markdown用block editor、7つのedit ops、three-mode web editor、preview adapterを実装した。ContainerのPhase 2（text sync）・Phase 3（block doc sync）も進み、文書をブロック単位で扱えるようになった。

段落をまとめて動かす操作が、ここから日常になる。プログラムだけでなく、ノートやドキュメントも同じ編集基盤で扱える。

## 4. 汎用B-treeとconfidence lattice

`lib/btree`を汎用B-treeライブラリとして切り出し、3月のorder-tree作業と統合した。range delete、splice promotion chain repairもこの時期に入る。

並行して`lib/semantic`を新設し、推論結果の確信度を扱うConfidence latticeとsymbolic annotatorを追加した。すべてを確実とみなさない——推論の確からしさそのものを扱う、最初の基盤だ。

## 5. 言語の分離が始まる

`LanguageCapabilities[T]`により、`SyncEditor`からLambda固有の型を切り離した。汎用tree opへの道が開かれた。

JSON・Markdown、そして7月のJSXへつながる下地は、ここでできている。

## 6. drag-and-drop・E2E・moon.work

触れる範囲が広がった分、「壊れていない」証拠も増やした月だ。

Ideal editorとblock editorでdrag-and-dropの土台を実装し、Lambda/JSON editorのWeb E2EテストをCIへ載せた。loomでは`ReactiveParser`を廃止し、統一`@loom.Parser[T]`へ移行した。MoonBitのworkspace機構`moon.work`も導入し、依存方向ルールをCIで検証するようにした。

---

4月末までに、言語非依存のcore、ブロック単位でアドレス可能な文書、複数adapterの構成が見えた。5月のCognitionもscope graphも、この骨格の上に載る。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-4月|4月の日誌]]
