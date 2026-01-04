---
title: Structural Editingとは何か？
publish: true
tags: [compiler]
aliases: [What is Structural Editing？]
created: 2026-01-03T14:08:04+09:00
modified: 2026-01-04T19:48:19+09:00
---

# Structural Editingとは何か？

## 「テキスト」という制約からの解放

伝統的にプログラムは「テキスト」で記述されてきました。しかし、テキストはプログラムを表現するための一つのインターフェース（UI）に過ぎません。プログラミングの本質とは、文字を打ち込むことではなく、**「プログラムの木構造（論理構造）」を構築・変更する行為**です。

Structural Editingとは、この本質に立ち返り、テキストを介さずにプログラムの構造（AST/CST）を直接操作する手法を指します。

## Projectional Editing との関わり

Martin Fowlerが提唱した**Projectional Editing（投影型編集）**[^1]は、この考えをさらに拡張したものです。

- **ソースコード（真のモデル）：** 永続化されたデータ構造（例：XML, JSON, 独自のバイナリ）。
- **投影（プロジェクション）：** 開発者が目にする画面（テキスト形式、フローチャート形式、数式形式など）。

普通のプログラムでもプログラムはそのままの形では実行できません。コンパイラによって意味論的に同じ実行可能な形へと変換するステップが存在します。プログラムには少なくとも編集のためのインターフェースとしての表現形態と、実行のための表現形態が存在するのです。

![普通のプログラミング言語でのコンパイル](https://www.martinfowler.com/articles/compile.gif "普通のプログラミング言語でのコンパイル")

Projectional Editingにおいては、モデルを表現するためのインターフェースは複数存在します。ユーザーは好みのUIを必要によって選んで編集できますが、捜査は常に背後にある共通のモデルに対して行われます。

![Projectional Editingでのコンパイル](https://www.martinfowler.com/articles/workbench.gif "Projectional Editingでのコンパイル")

## なぜ「双模倣性（Bisimulation）」が必要なのか？

異なる表現形式（例：テキスト形式 A と グラフィカル形式 B）の間で編集を行う際、「形式 A での変更が、形式 B においても意味的に等価であること」を保証しなければなりません。ここで登場するのが双模倣性（そうもほうせい、Bisimulation）です。双模倣性とはシステムの状態遷移系におけるシミュレーションの特殊な形です。

- **シミュレーション：** システムAの動きをシステムBが正確にトレースできること。
- **双模倣性：** システムAとBが互いにお互いをシミュレートでき、どのステップにおいても「意味論的に等価な状態」を維持し続けていること。

Structural Editingにおいては、編集中の「未完成のコード」であっても、モデルとUIの間でこの整合性が保たれる（あるいは回復可能である）ことが理想とされます。

## プログラムの編集過程における課題

構造的編集を実現しようとすると、従来のテキストエディタでは発生しなかった（あるいは無視されていた）以下の問題に直面します。[^2]

1. **Problem 1: Syntactically Malformed Edit States（構文的に不正な編集状態）**
    - 編集の途中で「木構造」として成立しない瞬間をどう扱うか。（例：括弧が閉じられていない状態をASTでどう表現するか）
2. **Problem 2: Statically Meaningless Edit States（静的に無意味な状態）**
    - 構文は正しいが、型チェックを通らない状態（未定義変数の参照など）の許容範囲。
3. **Problem 3: Dynamically Meaningless Edit States（動的に無意味な状態）**
    - 実行した瞬間に必ずエラーになるようなコードへの変更をどう防ぐ、あるいは通知するか。
4. **Problem 4: A Calculus of Edit Actions（編集アクションの計算体系）**
    - 「1文字消す」ではなく「ノードを入れ替える」「関数を抽出する」といった操作を数学的に定義する必要性。
5. **Problem 5: Meaningful Suggestion Generation and Ranking（有意義な提案とランキング）**
    - 構造を直接いじる際、次に可能な操作をどう提示し、文脈から優先順位をつけるか。

## CST と AST の定義の再整理

構造的編集において、これらの使い分けは重要です。

- **CST (Concrete Syntax Tree / 具体構文木):**
    - テキスト表現における空白、コメント、セミコロン、あるいはグラフィカル表現における「色」や「座標」など、**具体的な表示情報**を保持した木。
    - 表示形式（投影）ごとに複数存在する可能性がある。
- **AST (Abstract Syntax Tree / 抽象構文木):**
    - 意味に関係のない装飾を削ぎ落とした、**プログラムの論理構造そのもの**。
    - 一つのプログラムに対し、本質的には一つに収束する。

## 関連リンク

[[Structure editor]]

[^1]: [Projectional Editing](https://www.martinfowler.com/bliki/ProjectionalEditing.html) の記事
[^2]: [Toward Semantic Foundations for Program Editors](https://arxiv.org/pdf/1703.08694) より引用