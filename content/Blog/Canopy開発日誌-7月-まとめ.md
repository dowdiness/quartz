---
title: Canopy開発日誌-7月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:55:00+09:00
modified: 2026-07-16T13:10:00+09:00
---

# Canopy開発日誌-7月-まとめ

2分で読める月次まとめ。日々の詳細は[[Canopy開発日誌-7月|通常版の日誌]]を、英語版は[[Canopy-July-2026-Highlights|Highlights (English)]]を参照。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. incrが破壊的リリースを2連発

0.13.0は、段階的な非推奨化を経ずに互換API surfaceを一括削除した。数日後の0.14.0は、使われていないghost handle型の削除、`InternTable`のinterior-mutationの穴の修正、disposed inputのエラーを正式な`ReadError`チャネルへ移す変更を加えた。どちらのリリースも、1〜2日のうちにloomとCanopyのpin更新として波及した。

## 2. 実バグが明文化された契約になる

MoonDspでのデバッグ中に見つかった再入バグ（reactive computeの中から呼ばれた`Input::set`）が、incrの評価戦略全体を定義するADRへと発展した。どのセル種別が透過的にキャッシュしてよいか、なぜ型ではなくランタイムで境界を強制するのか、なぜpull・push・Datalog fixpointが3つのライブラリに分かれず1つの`Runtime`を共有するのか。ガードの実装、レビューによる訂正、そして「ガードをあえて緩めない」という意図的な判断が、その後2日間で続いた。バグを見つけてからルールを書く、という筋の通ったやり方の一例になっている。

## 3. loomgenが自分のコード生成器を退場させる

loomのコード生成ツールloomgenは、月の前半に新しいEBNF演算子（`~`、`!`、`@until`、`{Sep}`、`@error_node`など）を次々に生成パーサーのパイプラインへ追加していった。ところがベンチマークで、tree-walkingするインタプリタが差分再パースにおいて生成コードに追いついたことが分かり、loomは手書きのコード生成器（`emit_grammar.mbt`）そのものを削除して、interpretベースへ全面的に舵を切った。月の後半は、loomgenの重心がMarkdownの行指向レクサ生成へと移っていった。

## 4. JSXがCanopyの4番目の言語になる

Lambda・JSON・Markdownと同じやり方で、CSTから`ProjNode`への読み取り専用projectionがJSX向けに実装された。既存の仕組みをそのまま再利用しており、新しい統合を手組みする必要はなかった。

## 5. generative UIが現れ、判断日 7/29 を持つ

JSX 投入後数日で、LLM 出力を検証済み JSX としてストリーミングパースし DOM へ reconcile する GenUI 実験が立ち上がった。ただし **構文が正しいだけでは「使える UI」の証拠にならない**——これが実験設計の出発点だ。実プロバイダ接続の前に「セッション所有の 1 つの機能的 projection」を証明する方針で、**2026/7/29** を継続/削除の判断日（kill date）とした。証拠が不十分なら DELETE がデフォルト。詳細は[[Canopy-GenUI実験-2026年7月|GenUI 実験の設計メモ]]。

## 6. js_engineがv0.6.0をリリース

`for await`/非同期iterationがtest262適合率100%に到達し、RegExpのlookbehind assertionが入り、Map・Set・Array・Promise・boxed primitivesを含む全stdlib builtinが単一のatomicなインストール契約へ移行し、埋め込み利用者向けのhost-object APIが実装され、class private field/method/static blockが仕様通り実装された。これらをまとめてv0.6.0としてリリースした。

---

7月を貫く筋: incrのAPI整理が周辺へ波及し、loomgenは元の形が不要だったことを自ら証明して身軽になり、そこで空いた場所を、Canopyのこれまでで最も実験的な新機能がすぐさま埋めていった。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [dowdiness/incr](https://github.com/dowdiness/incr) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-7月|7月の日誌]]
