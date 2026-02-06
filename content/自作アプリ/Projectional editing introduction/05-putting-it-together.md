---
title: "第5回: すべてを繋ぐ — エディタのパイプラインをインクリメンタル化する"
tags:
  - projectional-editing
  - incremental-computation
  - programming
  - editor
  - pipeline
  - chain-rule
series: "構文エラーのない世界へ"
publish: true
created: 2026-02-06
modified: 2026-02-06
---

# 第5回: すべてを繋ぐ — エディタのパイプラインをインクリメンタル化する

*シリーズ「構文エラーのない世界へ」 — [目次に戻る](./00-index.md) / [前回](./04-incremental-computation.md)*

---

## これまでのおさらい

4回にわたって、二つの大きなアイデアを見てきた。

**Projectional Editing 側:**
- プログラムの本体はテキストではなくAST（第1回・第2回）
- 穴（Hole）で未完成を表現。構文エラーが原理的に消える
- Gradual Structure Editing で、テキスト入力の自然さを維持（第3回）
- Marking で型エラーを局所化。Grove で複数人同時編集

**Incremental Computation 側:**
- 前回の結果を再利用して、変更箇所だけ計算し直す（第4回）
- P1-P3: 結果の再利用、中間結果のキャッシュ、補助情報の発見
- III メソッド: Iterate, Incrementalize, Implement の体系的手順

今回は、この二つを具体的に接続する。

## エディタの中身はパイプラインである

Gradual Projectional Editor の内部を関数の連鎖として描くと、こうなる:

```
キー入力
  ↓
Text Input Adapter    ── テキスト断片をAST操作に変換
  ↓
Grove (グラフ操作)     ── AST操作をグラフに適用
  ↓
decompose             ── グラフを穴付きASTに変換
  ↓
mark                  ── ASTに型チェック・エラー注釈をつける
  ↓
project               ── 注釈付きASTを画面表示に変換
```

各段階は関数だ。入力を受け取り、出力を返す:

| 関数 | やること | 入力 | 出力 |
|------|---------|------|------|
| translate | テキスト → AST操作 | キー入力 + 文脈 | AST操作の列 |
| apply | グラフ更新 | グラフ + AST操作 | 更新されたグラフ |
| decompose | グラフ → AST | グラフ | 穴付きAST |
| mark | 型チェック | AST + 型環境 | 注釈付きAST |
| project_text | テキスト表示 | 注釈付きAST | 画面用テキスト |

キーを一つ打つと、この5つの関数が順に実行される。プログラムが大きくなると、特に mark（型チェック）と project（表示生成）のコストが問題になる。

## 各段階をインクリメンタル化する

Liu の III メソッドを各段階に適用する。面白いのは、**段階ごとに性質が違うので、適用する III ステップも異なる**ことだ。

### decompose: 集合操作のインクリメンタル化

Grove のグラフは、頂点と辺の**集合**だ。decompose はこの集合から穴付きASTを構築する。

集合に対するインクリメンタル化は、Liu の分類では I2 + I3 が必要:

```
I2 (Incrementalize): 辺が1本追加されたとき、AST全体を再構築しない。
    追加された辺の親ノードの該当スロットだけを更新する。

I3 (Implement): 「この頂点の親は誰か」「このスロットに繋がる辺は何か」を
    O(1) で引けるようにする HashMap を用意する。
```

これは Liu が「有限差分法 (finite differencing)」と呼ぶ技法の直接適用だ。集合に要素を追加/削除するとき、出力のどこが変わるかを機械的に導出できる。

```
辺 e を追加 (InsertEdge):
  → e の親ノードのスロットを更新するだけ。他は触らない。O(1)。

辺 e を削除 (DeleteEdge):
  → e を除去し、孤立した子サブツリーを穴に変換。O(1)。
```

全体を再構築すると O(ノード数) かかるところを、O(1) にできる。

### mark: 再帰関数のインクリメンタル化

型チェック（Marking）はAST上の**再帰関数**だ。

```
mark(ctx, expr) = match expr {
  Var(x)    → ctx で x の型を引く
  App(f, a) → mark(ctx, f) と mark(ctx, a) を再帰的に型チェック
  Let(x, e, body) → e を型チェックし、ctx に x の型を追加して body を型チェック
  Hole      → "ここに式が必要" と注釈をつける
  ...
}
```

再帰関数には I1 + I2 が必要:

```
I1 (Iterate): 最小増分は「ASTノード1つの変更」。
    再帰の構造から、変更されたノードから根に向かうパスだけを再計算すればよい。

I2 (Incrementalize):
    P1 — 変更されていないサブツリーの型チェック結果を再利用する。
    P2 — 各ノードの (型環境, 推論された型) をキャッシュする。
    P3 — 依存関係グラフを維持する（変数 x の型が変わったら、
         x を使っているノードも再チェックが必要）。
```

1万ノードのASTで1ノードが変わった場合:

```
全体再計算: O(10,000)
インクリメンタル版: O(変更ノードから根までの深さ) ≈ O(log n) 〜 O(depth)
```

Hazel プロジェクト自身も、この問題に OOPSLA 2025 論文 "Incremental Bidirectional Typing via Order Maintenance" で取り組んでいる。AST の走査順序を Order Maintenance データ構造で管理し、変更時に再計算が必要なノードを効率的に特定する。

### project_text: 表示のインクリメンタル化

テキスト表示生成もAST上の再帰関数だ。各ノードを文字列に変換し、連結する。

```
I1: 最小増分は「1ノードの注釈変更」。
I2:
    P1 — 変更されていないサブツリーのテキスト表現を再利用。
    P2 — 各ノードのテキスト表現と位置（offset, length）をキャッシュ。
```

実装としては、**Rope** というデータ構造が自然に対応する。Rope は文字列の平衡二分木で、途中への挿入・削除を O(log n) で処理できる。

```
全体再生成: O(全ノード数) の文字列連結
インクリメンタル版: O(log n) の Rope 更新
```

## Chain Rule: パイプラインを繋ぐ

個々の関数をインクリメンタル化しただけでは不十分だ。パイプラインの各段階を**繋ぐ**必要がある。

ここで Liu が指摘する **連鎖律 (chain rule)** の離散版が効く。

微積分の連鎖律は `(f ∘ g)' = f' · g'`。合成関数の微分は、各関数の微分の積。

インクリメンタル化でも同じ構造がある:

```
g(x) = r_g       f(r_g) = r_f
       ↓                  ↓
g'(x, Δx) = Δr_g  f'(r_g, Δr_g) = Δr_f
```

g のインクリメンタル版が「出力の変更 Δr_g」を返す。これが f のインクリメンタル版の「入力の変更」になる。

エディタのパイプラインで具体的に見ると:

```
1. キー入力 Δkey
     ↓ translate
2. AST操作 Δop を生成
     ↓ apply'
3. グラフの変更 Δgraph を生成（どの辺/頂点が変わったか）
     ↓ decompose'
4. ASTの変更 Δexpr を生成（どのノードが変わったか）
     ↓ mark'
5. 注釈の変更 Δmarked を生成（どの型注釈が変わったか）
     ↓ project_text'
6. テキストの変更 Δtext を生成（画面のどこを書き換えるか）
```

各段階の「Δ出力」が次の段階の「Δ入力」になる。**変更が波のようにパイプラインを伝播していく**が、各段階で波は小さくなる（影響範囲が絞り込まれる）。

最終的に画面更新に届くのは「この行のこの部分を書き換えてください」という最小限の変更だけだ。

## III メソッドで整理する

全体を Liu の III メソッドの対応表としてまとめる:

| コンポーネント | 抽象のレベル | I1 (Iterate) | I2 (Incrementalize) | I3 (Implement) |
|------------|----------|------------|-------------------|--------------|
| decompose | 集合操作 | 不要（集合操作自体が最小増分） | 有限差分法で辺の追加/削除を処理 | HashMap で O(1) lookup |
| mark | 再帰関数 | 最小増分 = 1ノード変更。根に向かうパスを再走査 | キャッシュ (P2) + 依存グラフ (P3) | HashMap + Order Maintenance |
| project_text | 再帰関数 | 最小増分 = 1ノードの注釈変更 | テキスト表現キャッシュ (P2) | Rope |
| translate | 規則 (データ+制御) | 最小増分 = 1文字の入力/削除 | パース候補のキャッシュ (P2) + カーソル文脈 (P3) | Trie |

Liu の「高レベル抽象ほどインクリメンタル化が容易」という原則がここでも効く。decompose は集合操作だから有限差分法で機械的にインクリメンタル化できる。mark は再帰関数だからキャッシュ戦略が構造的に決まる。もしこれらを低レベルのループと配列操作で書いていたら、インクリメンタル化の設計ははるかに困難だっただろう。

## まだ存在しない統合

ここまで描いたアーキテクチャは、個々の部品としては研究が進んでいる:

- 穴付きASTと型チェック: Hazel (POPL 2017, 2019, 2024)
- Gradual Structure Editing: Hazel (VL/HCC 2023)
- 構造編集の CRDT: Grove (POPL 2025)
- 型チェックのインクリメンタル化: Hazel (OOPSLA 2025)
- インクリメンタル計算の統一理論: Liu (FnTiPL 2025)

しかし、**これらすべてを統合した実用システムは、まだ存在しない。**

未解決の課題も多い:

**Text Input Adapter の設計**: テキスト入力をAST操作に「自然に」変換する仕組みは、最も難しい問題の一つ。`1 + 2` のあとに `*` を打つと、ASTの構造が根本的に変わる（`+` ノード全体が `*` ノードの子に移動する）。「テキストでは1文字の変更が、木構造では大規模な組み替え」になるケースをどう処理するか。

**任意の言語への一般化**: 特定の言語だけでなく、任意の DSL に対してエディタの各コンポーネント（入力アダプタ、型検査、表示、編集操作）を自動生成できるか。Liu の部分評価との関連でいえば、これは Futamura の射影に対応する問題だ。

**実装言語の選択**: Liu が強調する「高レベル抽象ほどインクリメンタル化が容易」は、実装言語の選択に直結する。代数的データ型とパターンマッチングを持つ言語（ML 系、Haskell、MoonBit 等）でASTを定義すれば、再帰関数の構造が明確になり、I1（反復化）が容易になる。

## シリーズのまとめ

最後に、5回分の内容を一つの流れとして振り返る。

**構文エラーはなぜ生まれるか？** — テキストとASTの間にパーサーがいて、途中状態を受け付けないから（第1回）

**別のやり方はあるか？** — ある。ASTを直接編集する Projectional Editing。穴で未完成を表現。構文エラーが原理的に消える（第2回）

**でもテキスト入力の自然さが失われないか？** — Gradual Structure Editing で両立できる。パーサーの役割が「心臓部」から「アダプタ」に変わる。さらに Grove で複数人同時編集も可能に（第3回）

**でも計算コストが爆発しないか？** — Incremental Computation で解決する。変わったところだけ計算し直す。Liu の III メソッドで体系的にインクリメンタル化できる（第4回）

**具体的にどう統合するのか？** — エディタ内部のパイプラインの各段階をインクリメンタル化し、Chain Rule で繋ぐ。各段階の性質（集合操作、再帰関数、規則）に応じた III の適用（第5回 = 本記事）

根底にある原理は二つだけだ:

1. **プログラムは文字列ではなく構造である。構造を直接編集せよ。**
2. **変更は局所的である。局所的に計算し直せ。**

この二つの原理の組み合わせが、次世代のプログラミング環境の基盤になる。

---

## 参考文献

- Omar, C. et al. "Hazelnut: A Bidirectionally Typed Structure Editor Calculus." POPL 2017.
- Omar, C. et al. "Toward Semantic Foundations for Program Editors." SNAPL 2017.
- Voelter, M. et al. "Efficiency of Projectional Editing: A Controlled Experiment." FSE 2016.
- Moon, D., Blinn, A., Omar, C. "Gradual Structure Editing with Obligations." VL/HCC 2023.
- Adams, M. et al. "Grove: A Collaborative, Structure-Aware, Bidirectionally Typed Structure Editor Calculus." POPL 2025.
- Zhao, E. et al. "Incremental Bidirectional Typing via Order Maintenance." OOPSLA 2025.
- Omar, C. et al. "Total Type Error Localization and Recovery with Holes." POPL 2024.
- Liu, Y.A. "Incremental Computation: What Is the Essence?" Foundations and Trends in Programming Languages, 2025.

---

*[シリーズ目次に戻る](./00-index.md)*
