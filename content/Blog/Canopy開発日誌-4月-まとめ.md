---
title: Canopy開発日誌-4月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:40:00+09:00
modified: 2026-07-16T13:10:00+09:00
---

# Canopy開発日誌-4月-まとめ

2分で読める月次まとめ。日々の詳細は[[Canopy開発日誌-4月|通常版の日誌]]を、英語版は[[Canopy-April-2026-Highlights|Highlights (English)]]を参照。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. すべてのエディタ面がひとつのprotocolに

Ideal editorとprosemirror exampleが、単一の`EditorProtocol`経由で統一された。CM6Adapter/PMAdapterがadapter層として整理され、各エディタ面が個別の配線を持つのではなく、同じ言葉でドキュメントモデルと話せるようになった。

## 2. pretty-printerがDOMとつながる

Wadler-Lindig pretty-printerの出力を`ViewNode`へ変換するbridgeが実装され、HTML syntax highlightingと統合された。テキスト整形ツールだったものが、描画パイプラインの一部になった。

## 3. Markdownが本格的なblock editorに

Markdown用のblock editor、7つのMarkdown edit ops、three-mode web editor、block-input textarea overlay、MarkdownPreviewのsemantic HTML adapterが実装された。あわせて、文書をブロック単位で扱う「Container」のPhase 2（text sync）・Phase 3（block doc sync）、document-level undo groupingも進み、文書がブロック単位でアドレス可能になった。

## 4. 汎用B-treeとconfidence lattice

`lib/btree`が汎用B-treeライブラリとして切り出され、3月のorder-tree作業と統合された。range delete、splice promotion chain repairが実装されている。並行して`lib/semantic`が新設され、推論結果の確信度を扱うConfidence lattice（確信度の階層構造）とsymbolic annotatorが実装された。「すべてを確実とみなす」のではなく、推論の確からしさそのものを扱う最初の基盤になる。

## 5. 言語の分離が始まる

`LanguageCapabilities[T]`によって、`SyncEditor`からLambda固有の型が切り離された。Lambdaが唯一の言語だという前提を置かない、汎用的なtree opへの道が開かれた。後にJSON・Markdown、そして7月のJSXへとつながっていく下地になる。

## 6. drag-and-drop・E2E・moon.work

Ideal editorとblock editorでdrag-and-dropの土台（semantic Before/After drop、grip-only drag、outline DnD）が実装され、Lambda/JSON editorのWeb E2EテストがCIに加わった。loomでは`ReactiveParser`を廃止し統一`@loom.Parser[T]`へ移行。MoonBitのworkspace機構`moon.work`が導入され、依存方向ルールがCIで検証されるようになった。

---

4月末までに、Canopyはその後何ヶ月も保つ形を得た。言語に依存しないcore、ブロック単位でアドレス可能な文書、そして一つの決め打ちエディタではなく、増え続けるadapterの集合。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-4月|4月の日誌]]
