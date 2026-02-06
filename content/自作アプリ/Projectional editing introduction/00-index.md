---
title: "構文エラーのない世界へ"
tags:
  - projectional-editing
  - incremental-computation
  - programming
  - editor
  - series-index
series: "構文エラーのない世界へ"
publish: true
created: 2026-02-06
modified: 2026-02-06
---

# 構文エラーのない世界へ

**— Projectional Editing と Incremental Computation が拓くエディタの未来**

---

プログラマーなら誰でも経験がある。括弧を閉じ忘れて画面が真っ赤になる。セミコロンが足りなくてビルドが通らない。考えている途中なのに、エディタは容赦なく「構文エラー」と叫ぶ。

もしプログラムの構造を直接編集できたら？ 構文エラーという概念自体が存在しない世界を作れたら？

この問いに答えるのが **Projectional Editing** であり、その実用的な課題を解決する鍵が **Incremental Computation** にある。

この記事シリーズでは、この二つのアイデアを段階的に紹介する。

---

## このシリーズの読み方

興味のあるところから読んでもらって構わないが、順番に読むと一番わかりやすい。

### 🟢 入門 — まずここから

**[第1回: なぜ構文エラーは生まれるのか](./01-why-syntax-errors.md)**

普段何気なく使っているテキストエディタの裏で何が起きているのか。「テキストを書いてパーサーに渡す」というモデルの構造的な問題を、プログラミング経験があれば誰でもわかる言葉で説明する。

**[第2回: Projectional Editing — もう一つのやり方](./02-projectional-editing.md)**

テキストではなくプログラムの構造（AST）を直接編集する。「穴」を使って未完成のプログラムにも意味を与える。この発想の良さと、実際に使ってみると浮かび上がる難しさの両面を紹介する。

### 🟡 ちゃんとした理解 — 仕組みがわかる

**[第3回: Gradual Structure Editing — テキストの自然さと構造の安全を両立する](./03-gradual-structure-editing.md)**

Pure Projectional の限界を認めた上で、テキスト入力の自然さを取り戻す Hazel プロジェクトのアプローチ。パーサーの役割が「心臓部」から「アダプタ」へと変わる話。Grove による複数人同時編集まで。

**[第4回: Incremental Computation — 変わったところだけ計算し直す](./04-incremental-computation.md)**

文字を打つたびに全体を再計算していたら使い物にならない。「前回の結果を再利用する」というインクリメンタル計算の本質を、Liu の定式化に沿って、身近な例から説明する。

### 🔴 深い理解 — 全体が繋がる

**[第5回: すべてを繋ぐ — エディタのパイプラインをインクリメンタル化する](./05-putting-it-together.md)**

Projectional Editing と Incremental Computation を統合する。エディタ内部のパイプライン（入力→AST操作→型検査→表示）の各段階をインクリメンタル化し、変更が効率的に伝播する仕組みを具体的に描く。Liu の III メソッドと Chain Rule がどう適用されるかまで。

---

## 全体像: 30秒で掴む

```
  【従来】                          【Projectional + Incremental】

  テキスト入力                       テキスト入力
      ↓                                ↓
  パーサー（失敗 = 構文エラー）        Text Input Adapter（失敗 = 穴を挿入）
      ↓                                ↓
  AST                               AST（常に整合、穴付き）
      ↓                                ↓
  全体を型チェック                    変更箇所だけ型チェック ← Incremental
      ↓                                ↓
  全体を再描画                       変更箇所だけ再描画 ← Incremental
                                       ↓
                                    テキスト / ブロック / 数式 ← 複数の射影
```

左は馴染みのあるモデル。右が本シリーズで紹介する世界だ。

---

## 先行研究マップ

各記事の中で詳しく触れるが、全体の関係を先に示しておく。

```
1981  Cornell Program Synthesizer ─── 構造エディタの先駆け
        │
2010s JetBrains MPS ──────────── 産業実用化、粘性問題の発見 (FSE 2016)
        │
2017  Hazelnut (POPL 2017) ────── 穴＋双方向型付けの形式的基礎
        │
2023  Gradual Structure Editing ── テキスト入力との融合 (VL/HCC 2023)
        │
2024  Hazel Marking (POPL 2024) ── 型エラーの局所化と回復
        │
2025  Grove (POPL 2025) ────────── 構造編集のCRDT、複数人同時編集
      Incr. Bidir. Typing (OOPSLA 2025) ── 型チェックのインクリメンタル化
      Liu "What Is the Essence?" ── インクリメンタル計算の統一理論
```

---

## 想定する読者

- プログラミング経験がある人（言語は問わない）
- 「エディタ」「パーサー」「型チェック」という言葉をなんとなく知っている人
- 学術論文を読んだことがなくても大丈夫

専門用語は初出時に必ず説明する。数式は最小限。具体例を多用する。

では、[第1回](./01-why-syntax-errors.md) から始めよう。
