---
title: Pretty Printer
publish: false
tags: [compiler]
aliases: [無題のファイル]
created: 2026-01-04T20:50:52+09:00
modified: 2026-01-13T18:26:08+09:00
---

# Pretty Printer

[Philip Wadler](https://homepages.inf.ed.ac.uk/wadler/papers/prettier/prettier.pdf)
[A Pretty Expressive Printer (with Appendices)](https://arxiv.org/abs/2310.01530)
[Strictly Pretty](https://lindig.github.io/papers/strictly-pretty-2000.pdf)
[Reddit](https://www.reddit.com/r/ProgrammingLanguages/comments/vzp7td/pretty_printing_which_paper/)

## Functional Pearl

- [A Pretty But Not Greedy Printer](https://jyp.github.io/pdf/Prettiest.pdf)
- [Linear, Bounded, Functional Pretty-Printing](https://www.cs.kent.ac.uk/pubs/2009/2847/content.pdf)

# Geminiによる調査

https://gemini.google.com/share/7977505b9bf3

## 質問

Pretty printerの歴史と仕組み、現代に作る場合のベストプラクティスなどを教えてください。

## 答え

Pretty Printer（コード整形ツール）の歴史、仕組み、そして現代における実装のベストプラクティスについて、提示された資料と技術的な背景をもとに解説します。

## 1. 歴史的背景と進化

Pretty Printerのアルゴリズムは、数十年にわたり「表現力（柔軟なレイアウト）」と「計算量（速度）」のトレードオフと戦ってきました。

- **1980年: Oppenのアルゴリズム (The Imperative approach)**
    
    - 手続き型言語（Lisp/Pascal環境）向けに開発された初期のアルゴリズムです。
        
    - 「可能な限り行を埋め、溢れたら改行する」という貪欲法（Greedy）に近いアプローチで、線形時間 $O(n)$ で動作するのが特徴です。
        
    - 現在のRustコンパイラやGoのフォーマッタなど、高速性が求められるシステムの一部はこの系譜の影響を受けています。
        
- **1995年: John Hughesの設計 (The Functional approach)**
    
    - 関数型プログラミングの文脈で「ドキュメント代数（Algebra）」という概念を導入しました。
        
    - コードを単なる文字列ではなく、`Text`（文字）、`Nest`（インデント）、`Union`（複数のレイアウト候補の集合）といった構成要素の組み合わせとして扱います。
        
- **2003年: Philip Wadler "A Prettier Printer" (提示されたPDF)**
    
    - Hughesの理論を洗練させ、Haskellの遅延評価（Lazy Evaluation）を活かしたシンプルで強力なライブラリを提案しました。
        
    - **仕組み:** 文書を「一行で表示する場合」と「改行する場合」の候補の木構造として持ち、ページ幅に収まる「最良のレイアウト」を探索します。
        
    - **現代への影響:** 今日の多くのPretty Printer（JavaScriptの`Prettier`など）の基礎理論となっています。
        
- **2023年: "A Pretty Expressive Printer" (提示されたArXiv論文)**
    
    - **課題:** 既存のアルゴリズム（Wadlerなど）には、「表現できないレイアウトがある」あるいは「最適なレイアウトを見つけるために指数関数的な時間がかかる場合がある」という欠点がありました。
        
    - **解決策:** この論文では、これまでの手法よりも厳密に「表現力が高い（Expressive）」かつ「最適性（Optimality）」を保証しつつ、実用的な速度で動作する新しいアルゴリズム（$\Pi_e$）を提案しています。これはPretty Printerの理論における最新の成果の一つです。
        

---

## 2. 仕組み（メカニズム）

現代的なPretty Printerは、通常以下の3段階のパイプラインで動作します。

1. **AST/CST解析 (Parse)**
    
    - ソースコードを解析し、抽象構文木（AST）または具象構文木（CST）を作成します。
        
2. **中間表現への変換 (Doc IR Generation)**
    
    - 構文木を、Pretty Printer専用の「中間表現（Intermediate Representation: IR）」、しばしば**Doc**と呼ばれる代数データ型に変換します。
        
    - **主な構成要素:**
        
        - `text(string)`: 文字列そのもの。
            
        - `line`: 文脈によって「スペース」にも「改行」にもなりうる区切り。
            
        - `group(doc)`: 「可能なら一行に収め、無理なら内部の`line`で改行する」という指示。
            
        - `nest(n, doc)`: インデントレベルを上げる指示。
            
3. **レンダリング (Solving/Printing)**
    
    - 指定されたページ幅（例: 80文字）に基づき、`group`や`line`を実際にどう配置するかを決定し、最終的な文字列を出力します。
        

---

## 3. 現代に作る場合のベストプラクティス

提示されたRedditのスレッドや近年の開発トレンド（Prettier, Tree-sitter, Rust製ツールなど）に基づくと、現代的なPretty Printerを作成する際は以下が重要になります。

### A. コメントの保存 (The "Comments" Problem)

- **課題:** コンパイラの教科書通りにAST（抽象構文木）を作ると、コメントや空白が捨てられてしまいます。フォーマッタにおいてコメントの消失は許されません。
    
- **対策:**
    
    - **CST (Concrete Syntax Tree) を使う:** `Tree-sitter`や`Roslyn`のように、コメントや空白も含めて木構造にするパーサを使用します。
        
    - **ASTへのアタッチ:** コメントを「直前のノード」や「直後のノード」に関連付けてAST上に保持する手法です（Prettierはこの方式ですが、移動制御が複雑になります）。
        

### B. Wadlerのアルゴリズムの「厳格な（Strict）」実装

- Wadlerのオリジナル論文はHaskellの遅延評価を前提としています。これをRust, Go, TypeScriptなどの**先行評価（Strict）言語**でそのまま実装すると、計算量が指数関数的に爆発する可能性があります。
    
- **対策:** Christian Lindigの論文 "Strictly Pretty" の手法などを参考に、先行評価言語向けに計算量を $O(n)$ に抑える工夫が入った実装を採用すべきです。
    

### C. 「Opinionated（独断的）」であること

- 現代のツール（Goの`gofmt`やJSの`Prettier`）の成功要因は、設定項目を極限まで減らしたことにあります。
    
- **ベストプラクティス:** ユーザーに細かい設定をさせず、ツールが唯一の正解を出力するように設計します。これによりチーム内のスタイル論争を終わらせることができます。
    

### D. 最新研究の採用 (ArXiv論文の示唆)

- もしあなたが新しい汎用言語のフォーマッタを作るなら、提示されたArXiv論文（A Pretty Expressive Printer）にあるような、**「最適レイアウトのコスト関数」を最小化するアプローチ**を検討する価値があります。
    
- 従来の貪欲法（最初に見つけた改行場所で改行する）では、「全体で見ると不格好」な出力になる場合がありますが、コストベースのアプローチ（TeXの改行アルゴリズムに近い発想）を取り入れることで、より人間にとって読みやすい出力を生成できます。
    

## まとめ

現代でPretty Printerを作るなら、**Wadlerの代数モデル**を基礎にしつつ、**CSTを用いてコメントを完全に維持**し、言語仕様（先行評価）に合わせて**線形時間で動作するように最適化**されたアルゴリズムを採用するのが定石です。