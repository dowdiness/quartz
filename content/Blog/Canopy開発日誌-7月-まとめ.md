---
title: Canopy開発日誌-7月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:55:00+09:00
modified: 2026-08-04T23:07:45+09:00
---

# Canopy開発日誌-7月-まとめ

incrの整理が一区切りつけば、JSXへ手を広げられるはずだった。0.13.0と0.14.0は、そのあたりの見込みを改めた。月の後半は、GenUI実験の決着、web全体のWaku移行、EGW認可によるリアルタイムコラボレーション、そしてCRDTの意図保存能力を実証する調査へと続いた。日々の記録は[[Canopy開発日誌-7月|通常版の日誌]]、英語版は[[Canopy-July-2026-Highlights|Highlights (English)]]。

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

## 7. GenUIは、判断日を待たずに決着した

7/29の判断日を迎えても、mainブランチに「keep」や「delete」を宣言するコミットは出てこない。実際には、7/17に作り込まれたCodex/Ollamaのプロバイダ比較実装（30コミット近く）がマージされないまま残り、代わって"Legible Instrument"——旅程の意思決定を対象にした、判読可能な生成UIという方向——が7/19に生まれた。7/27のWaku移行では、GenUIは実プロバイダとの生きた接続ではなく**記録済みのproduction replayを保持する形**で`/genui`へ着地している。証拠が足りない部分は判断日を待たずに淘汰され、証拠のあった部分だけが製品ルートとして残った。

## 8. web全体がWakuへ移り、EGW認可がリアルタイムコラボレーションを支える

7/24、event-graph-walker（EGW）による認可基盤をCanopyへ導入し、collaboration protocolをv3へ直接切り替えた。incrのtyped spreadsheetもこの認可経由へルーティングし、数日後にはCloudflare Workers + Durable Objectsの上で実際に動くリアルタイムコラボレーションへ育てた（room admission policy、WebSocket hibernation復帰、脅威モデルまで込みで1本のPR）。

並行して、`examples/web`全体をVite単体からWakuへ移行した。7/26に着手し、Posts・JSON・Markdown・Mini-ML・Memo・Session Resumeを次々とネイティブルートへ移し、7/27には全ルートの移行を完走させている。

## 9. loomがgrammarの契約を強制し、CommonMark強調記法を仕上げる

`progress_of`が参照先ルールのsubtreeを毎回丸ごと再展開しており、参照の深さに対して指数関数的にコンパイル時間が増えていた（深さ26で65秒）。原因を突き止め、memo化で解消したうえで、「進まないrepeatは欠陥」という契約をgrammar全体へ明文化した（一部コミットはClaude Opus 5と共著）。

月の後半はMarkdownの強調記法（`*`/`_`のdelimiter run）に集中した。誰がdelimiter runを所有するかという設計判断を経て、CommonMarkの強調解決を統合し、不正な出力をfail-fastで検出するcanonical formatterまで仕上げた。

## 10. js_engineがv0.7.0、そしてbtreeがCRDTの穴を実証する

js_engineはv0.7.0をリリースした（test262: strict 91.3%・non-strict 90.7%、Koji Ishimoto氏がgithub-actions botと共著）。構造化Engine diagnosticsとhost-owned JSONの直接注入が主な追加点になる。

月末、独立した`dowdiness/btree`パッケージ（counted B+ tree）のinvariant強化と並行して、興味深い実証実験が入った。論文"Beyond Text Editing"のFig. 3をCanopyのtext CRDT上で再現したところ、rename操作とmove操作を別々のレプリカで行い両方向でマージすると、**収束はするが意味は変わってしまう**（パースは通るが変数が2つunboundになる）ことが確認された。同じシナリオを実際に出荷されているoperation layerで再生すると、意図通りにマージされ、しかも危険なcapture violationは正しく拒否される。text CRDTだけでは掴めない意味の破壊を、operation layerが捕まえている、という実測になる。月の最後の3日間は、この知見も踏まえて構造化パーサー診断をeditorへ届ける基盤づくりに費やされた。

---

incrの整理がloomとCanopyへ波及し、loomgenは手書き生成器を削除してinterpretベースへ移った。そのあとJSXとGenUIが入り、GenUIは判断日を待たずに縮小・着地した。月の後半はWaku移行とEGW認可によるコラボレーション基盤、そしてCRDTの意味保存に関する実証実験に重心が移った。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [dowdiness/incr](https://github.com/dowdiness/incr) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-7月|7月の日誌]]
