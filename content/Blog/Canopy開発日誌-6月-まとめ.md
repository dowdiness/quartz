---
title: Canopy開発日誌-6月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:50:00+09:00
modified: 2026-07-16T13:10:00+09:00
---

# Canopy開発日誌-6月-まとめ

2分で読める月次まとめ。日々の詳細は[[Canopy開発日誌-6月|通常版の日誌]]を、英語版は[[Canopy-June-2026-Highlights|Highlights (English)]]を参照。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. 構造編集で意味が静かに変わることがなくなった

Lambda編集が一つの節目を迎えた。rename・move・duplicate・extract-to-letが、module直下だけでなくblock内のbindingでも動くようになり、すべての編集が「編集後テキストを再パースして意図したASTになるか」で検証されるようになった。ここに至るには、bindingをlambdaパラメータの前へ動かすと参照先が静かに変わってしまうケースを捕まえる、scope graphベースのshadowingチェックが必要だった。名前ではなくbinder identityで計算するalpha-safe beta reductionの実験も始まった。

## 2. 編集してもNodeIdが残る

`Var(x)`をlambdaで包むだけで、そのノードの識別子が失われ、選択状態や折りたたみ状態も一緒に消えてしまっていた。`IdentityTransform`とhinted reconcileという新しいhint channelにより、編集操作が「これはwrapです」「このunwrapはこの子を残します」といった意図を宣言できるようになり、構造編集をまたいでNodeIdが生き残るようになった。安全なhintを出せない操作は保守的にopt outする。

## 3. Markdownが2番目の構造編集対象言語に

見出しやリスト項目に、壊れたparseや一時的な消失をまたいでも安定したidentityを持たせた（parse validityでgateされたretention/retiredのライフサイクル）。同一リスト内でのitem moveも、安全でない移動をきちんと弾いたうえで動くようになった。SDEG（Structure-Directed Edit Grammar）というidentity側表を使うアプローチは、他の言語へも一般化していく土台になる。

## 4. 外部解析ツールを、CRDTを壊さずに取り込む

ast-grepのような外部ツールの解析結果を、「テキストのスナップショットに紐づく、捨てられるfact」として取り込むanalysis query layerが新設された。UTF-16 rangeへ変換され、エディタのdecorationとして表示され、古くなれば捨てられる。text CRDTだけが唯一の永続状態であり続ける。Phase 1（snapshotモデル、offset変換、decoration）がmainに入った。

## 5. js_engineのtest262適合率向上

MoonBit製のJSインタプリタがv0.3.0をリリースし、体系的なtest262キャンペーンを進めた。JSON.parseが100%（142/142）に到達（Proxy対応のreviver込み）、lexerが非BMP文字を含めてUTF-16正確に、regex/除算の判定を書き直し、Promiseの仕様修正5件、tombstoneベースのSet iteration、timer queueの書き換え（2.15倍高速化）まで進んだ。

## 6. ビルドが「大人」になった

Canopyが所有する全マニフェストがmoon.mod.jsonからmoon.mod（TOML）へ移行し、13 submoduleがworkspace memberになった。長らく残っていた依存解決の回避策も解消された。ベンチマークのリグレッション検知がPR gateに近づき、「skipはgreenではない」という明確なルールも定まった。

## 7. 月末、loomgenの誕生とincrのMemo削除、新しいCanopyエディタ2本

最終週（6/27-6/30）は特に密度が高かった。loomに新しいコード生成器「loomgen」が誕生し、アノテーション付きのToken/Term enumからsyntax kind・step lexer・grammar IRを生成できるようになった。incrでは`Memo`/`HybridMemo`/`MemoMap`から`Derived`/`ReachableDerived`/`DerivedMap`へ、レガシー`Signal`型の削除まで含めて3日間で移行を完了させた。Canopy側では、block-editorのdrag-and-dropと、フルに対話的なJSON tree editorという2本の新しいエディタが、テストとともに一気に立ち上がった。

---

ペース: CanopyのPR番号は月内に#445から#783まで進んだ。5つの周辺リポジトリで並行作業が進み、AIエージェント（Claude CodeとCodex）との協働のもとで人間がレビューする、という開発体制で書かれている。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-6月|6月の日誌]]
