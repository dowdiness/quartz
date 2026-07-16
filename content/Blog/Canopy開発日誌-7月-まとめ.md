---
title: Canopy開発日誌-7月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:55:00+09:00
modified: 2026-07-16T15:40:00+09:00
---

# Canopy開発日誌-7月-まとめ

incrの整理が一区切りつけば、JSXへ手を広げられるはずだった。0.13.0と0.14.0は、そのあたりの見込みを改めた。日々の記録は[[Canopy開発日誌-7月|通常版の日誌]]、英語版は[[Canopy-July-2026-Highlights|Highlights (English)]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. incrが破壊的リリースを2連発

0.13.0は、段階的な非推奨化を経ずに互換API surfaceを一括削除した。数日後の0.14.0では、ghost handle型の整理、`InternTable`のinterior-mutation修正、disposed inputのエラーを`ReadError`チャネルへ移す変更が続いた。

loomとCanopyのpin更新は、どちらのリリースも1〜2日で終わっている。休む間もなく、次の破壊的変更へ移った。

## 2. 実バグが明文化された契約になる

MoonDspのデバッグで、reactive computeの最中から`Input::set`が呼ばれる再入が見つかった。バグ報告で終わらず、incrの評価戦略全体を定義するADRへ発展した。

どのセルが透過的にキャッシュしてよいか。なぜ型ではなくランタイムで境界を強制するのか。pull・push・Datalog fixpointをなぜ1つの`Runtime`で扱うのか。7/8〜7/10の2日間で、ガード実装とレビュー訂正を経て、「ここはあえて緩めない」という判断まで一気に決まった。

## 3. loomgenが手書きコード生成器を削除する

月の前半は、EBNF演算子（`~`、`!`、`@until`、`{Sep}`、`@error_node`など）をloomgenに足し、生成パーサーのパイプラインを広げる作業が続いていた。ベンチマークを見ると、tree-walkingインタプリタが差分再パースで生成コードに追いついていた。

手書きの`emit_grammar.mbt`を削除し、interpretベースへ移行する判断は、この数字から出ている。月の後半は、Markdown向けの行指向レクサ生成へ重心が移った。

## 4. JSXがCanopyの4番目の言語になる

Lambda・JSON・Markdownと同じ経路で、CSTから`ProjNode`への読み取り専用projectionをJSX向けに足した。新しい統合を手組みする必要はなく、既存の仕組みをそのまま伸ばせた。

## 5. generative UIが現れ、判断日 7/29 を持つ

JSX投入から数日で、LLM出力を検証済みJSXとしてストリーミングパースしDOMへreconcileするGenUI実験が立ち上がった。

**構文が正しいだけでは「使えるUI」の証拠にならない**——実験設計はここから始まる。実プロバイダ接続の前に「セッション所有の1つの機能的projection」を証明する方針で、**2026/7/29**を継続/削除の判断日（kill date）とした。証拠が不十分ならDELETEがデフォルト。[[Canopy-GenUI実験-2026年7月|設計メモ]]に詳しい。

## 6. js_engineがv0.6.0をリリース

月の後半にv0.6.0を出した。変更点は多いが、目につくものだけ挙げる。

- `for await`/非同期iterationのtest262適合率100%
- RegExp lookbehind assertion
- stdlib builtinのインストール契約を一本化
- 埋め込み向けhost-object API
- class private field/method/static block

test262の数字が一段上がり、埋め込み向けAPIも揃った。

---

incrの整理がloomとCanopyへ波及し、loomgenは手書き生成器を削除してinterpretベースへ移った。そのあとJSXとGenUIが入った。GenUIが残るかどうかは、7/29までにはっきりしない。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [dowdiness/incr](https://github.com/dowdiness/incr) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-7月|7月の日誌]]
