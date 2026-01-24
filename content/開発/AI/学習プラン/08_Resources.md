# Part 8: 推薦リソース - 読む順番ガイド

## 📚 このガイドについて

**「何を、いつ、どの順番で読むか」を完全に明確化**しました。迷わず学習を進められます。

## 🎯 読書戦略

### 最小セット（必読）
まずこれだけは読む → 各章で指定

### 標準セット（推奨）  
しっかり理解したい → 推奨マーク付き

### 完全セット（上級）
研究レベルを目指す → 全リソース

---

## 📅 12週間基本パス - 完全読書計画

### Week 1-2: 機械学習の分類（Part 1）

#### 📖 必読（10時間）
1. **Deep Learning Book Chapter 6-10**
   - **URL**: https://www.deeplearningbook.org/
   - **読む範囲**: Ch 6 (2h), Ch 7-8 (3h), Ch 10 (3h)
   - **読み方**: 
     - Ch 6.1-6.3: MLPの基礎（必須）
     - Ch 6.5: Backprop（精読）
     - Ch 10.1-10.2: RNN（必須）
     - Ch 10.10: Attention（流し読み）

2. **Attention Is All You Need (論文)**
   - **URL**: https://arxiv.org/abs/1706.03762
   - **時間**: 2時間
   - **読み方**:
     - Abstract + Introduction（10分）
     - Section 3（Multi-Head Attention）（40分）
     - Figure 1,2 を理解（20分）
     - 実装を見る（50分）

#### 💻 実装リソース（5時間）
1. **Annotated Transformer**
   - **URL**: http://nlp.seas.harvard.edu/annotated-transformer/
   - **時間**: 3時間
   - **やること**: コードを実際に実行

2. **PyTorch Tutorial: Transformer**
   - **URL**: https://pytorch.org/tutorials/beginner/transformer_tutorial.html
   - **時間**: 2時間

#### 📚 オプション（時間があれば）
- Understanding Deep Learning (Prince) - Ch 11-12

---

### Week 3-4: オートマトン理論（Part 2）

#### 📖 必読（12時間）
1. **Automata Theory: An Algorithmic Approach (Blondin)**
   - **URL**: https://michaelblondin.com/automata/
   - **読む順番**:
     - **Day 1-2**: Chapter 1-2（有限オートマトン）- 4時間
     - **Day 3-4**: Chapter 3（正規表現）- 2時間
     - **Day 5-6**: Chapter 5（文脈自由文法）- 3時間
     - **Day 7**: Chapter 7（計算可能性）- 3時間
   - **メモ**: 各章の練習問題を必ず解く

2. **Thinking Like Transformers (論文)**
   - **URL**: https://arxiv.org/abs/2106.06981
   - **時間**: 2時間
   - **読み方**: Transformerの形式言語的解釈

#### 📚 推奨（理解を深める）
1. **Computational Complexity - Chapter 1-3**
   - **URL**: https://theory.cs.princeton.edu/complexity/
   - **時間**: 4時間
   - **範囲**: P, NP, NC の定義

#### 💻 実装（5時間）
- Weighted Automaton の実装
- RNN で {a^n b^n} の学習実験

---

### Week 5-6: 定理証明入門（Part 5前半）

#### 📖 必読（15時間）
1. **Software Foundations Vol.1 - Logical Foundations**
   - **URL**: https://softwarefoundations.cis.upenn.edu/lf-current/
   - **読む順番**（重要！）:
   
   **Week 5 (7.5時間)**:
   - **Day 1**: Basics.v（Coqの基本）- 2h
   - **Day 2**: Induction.v（帰納法）- 2h
   - **Day 3**: Lists.v（リスト）- 1.5h
   - **Day 4**: Poly.v（多相性）- 2h

   **Week 6 (7.5時間)**:
   - **Day 5**: Tactics.v（証明戦術）- 2h
   - **Day 6**: Logic.v（論理学）- 2.5h
   - **Day 7**: IndProp.v（帰納的述語）- 3h

   - **重要**: 各章の練習問題を**必ず**解く
   - **Coqインストール**: `opam install coq`

#### 💻 実装（5時間）
- 簡単な関数の証明（plus, append等）
- リストの性質の証明

#### 📚 オプション
- Theorem Proving in Lean 4 - Chapter 1-3（Lean派の人向け）

---

### Week 7-8: 圏論基礎（Part 3前半）

#### 📖 必読（12時間）
1. **Category Theory for Programmers - Part 1**
   - **URL**: https://github.com/hmemcpy/milewski-ctfp-pdf
   - **読む順番**:
   
   **Week 7 (6時間)**:
   - **Chapter 1**: Category（圏）- 1.5h
   - **Chapter 2**: Types and Functions - 1h
   - **Chapter 3**: Categories Great and Small - 1.5h
   - **Chapter 4**: Kleisli Categories - 1h
   - **Chapter 5**: Products and Coproducts - 1h

   **Week 8 (6時間)**:
   - **Chapter 6**: Simple Algebraic Types - 1h
   - **Chapter 7**: Functors（関手）- 2h
   - **Chapter 8**: Functoriality - 1.5h
   - **Chapter 10**: Natural Transformations - 1.5h

2. **Backprop as Functor (論文)**
   - **URL**: https://arxiv.org/abs/1711.10455
   - **時間**: 2時間
   - **読み方**: Week 8 Day 7に読む

#### 📚 推奨（視覚的理解）
1. **Seven Sketches in Compositionality - Chapter 1-3**
   - **URL**: https://arxiv.org/abs/1803.05316
   - **時間**: 3時間
   - **特徴**: 図が豊富

#### 💻 実装（4時間）
- Haskellで圏論的NN実装
- または Catlab.jl で可視化

---

### Week 9-10: State Space Models（Part 1後半 + Part 2後半）

#### 📖 必読（10時間）
1. **Mamba論文**
   - **URL**: https://arxiv.org/abs/2312.00752
   - **読む順番**:
   
   **Day 1 (2h)**: 
   - Abstract, Introduction
   - Figure 1-3 を理解
   - Section 2（背景）

   **Day 2 (2h)**:
   - Section 3（Selective SSM）
   - Figure 4（アーキテクチャ）
   
   **Day 3 (2h)**:
   - Appendix B（並列スキャン）
   - 数式を追う

   **Day 4-5 (4h)**:
   - 実装を読む
   - 自分で実装してみる

2. **S4論文（背景理解用）**
   - **URL**: https://arxiv.org/abs/2111.00396
   - **時間**: 2時間
   - **読み方**: Mambaの前提知識として

#### 💻 実装（10時間）
- **必須**: 簡易版Mamba実装
- **推奨**: 並列スキャンの実装

#### 📚 オプション
- FlashAttention-2（効率化に興味があれば）

---

### Week 11-12: 統合プロジェクト（Part 7）

#### 📖 必読（5時間）
1. **CompCert論文**
   - **URL**: https://xavierleroy.org/publi/compcert-CACM.pdf
   - **時間**: 3時間
   - **読み方**: 意味保存の概念を理解

2. **Software Foundations Vol.2 - Chapter 1-3**
   - **URL**: https://softwarefoundations.cis.upenn.edu/plf-current/
   - **時間**: 2時間（流し読み）
   - **目的**: Hoare論理の概要

#### 💻 実装（15時間）
- **Week 11**: 検証済みコンパイラ最適化（10h）
- **Week 12**: 統合・テスト・ドキュメント（5h）

---

## 📅 24週間標準パス - 詳細読書計画

### Phase 1: 基礎構築（Week 1-8）

同じく上記だが、各週にオプション課題を追加

### Phase 2: 理論深化（Week 9-16）

#### Week 9-10: 圏論完全版

**追加必読**:
1. **Category Theory for Programmers - Part 2 Chapter 11-14**
   - Monoidal Categories
   - Enriched Categories
   - 時間: 8時間

2. **Categorical Semantics of Neural Networks**
   - **URL**: https://arxiv.org/abs/2101.05084
   - 時間: 3時間

#### Week 11-12: 論理学・型理論（Part 4）

**新規必読**:
1. **Types and Programming Languages - Chapter 1-5**
   - 型理論の基礎
   - 時間: 10時間

2. **Curry-Howard対応の論文**
   - 時間: 2時間

#### Week 13-14: 定理証明応用

**必読**:
1. **Software Foundations Vol.2 - 完全版**
   - Programming Language Foundations
   - 時間: 20時間（2週間）

#### Week 15-16: NN検証

**必読**:
1. **α,β-CROWN論文**
   - **URL**: https://arxiv.org/abs/2011.13824
   - 時間: 4時間

2. **ツール使用**:
   - α,β-CROWN実装
   - 時間: 10時間

### Phase 3: 実践応用（Week 17-24）

#### Week 17-20: コンパイラ最適化（Part 6）

**必読**:
1. **CompCert ソースコード読解**
   - **URL**: https://github.com/AbsInt/CompCert
   - 読む順番:
     - Week 17: common/, lib/（基礎）
     - Week 18: backend/（最適化）
     - Week 19: cfrontend/（フロントエンド）
     - Week 20: driver/（全体統合）

#### Week 21-24: 統合プロジェクト

実装に集中

---

## 📊 レベル別リソースマップ

### 🟢 初級（Week 1-6）

| リソース | 時間 | 優先度 |
|---------|------|--------|
| Deep Learning Book Ch 6-10 | 10h | ★★★★★ |
| Attention Is All You Need | 2h | ★★★★★ |
| Blondin Automata Ch 1-5 | 10h | ★★★★★ |
| Software Foundations Vol.1 Ch 1-7 | 15h | ★★★★★ |
| Category Theory Ch 1-10 | 12h | ★★★★☆ |

**合計**: 約49時間

### 🟡 中級（Week 7-16）

| リソース | 時間 | 優先度 |
|---------|------|--------|
| Mamba論文 | 8h | ★★★★★ |
| Backprop as Functor | 3h | ★★★★★ |
| CompCert論文 | 3h | ★★★★★ |
| Software Foundations Vol.2 | 20h | ★★★★☆ |
| Types and Programming Languages | 10h | ★★★★☆ |
| α,β-CROWN | 4h | ★★★☆☆ |

**合計**: 約48時間

### 🔴 上級（Week 17+）

研究論文中心

---

## 🎯 「これだけは読め」最小セット

### 時間がない人向け（30時間）

1. **Deep Learning Book Ch 6, 10** (5h)
2. **Attention Is All You Need** (2h)
3. **Blondin Automata Ch 1-3** (6h)
4. **Software Foundations Vol.1 Ch 1-4** (8h)
5. **Category Theory for Programmers Ch 1-8** (6h)
6. **Mamba論文** (3h)

**これで全体の60%は理解できる**

---

## 📚 分野別おすすめ順序

### 機械学習エンジニア向け

```
1. Deep Learning Book
2. Attention Is All You Need
3. Mamba
4. FlashAttention-2
5. CompilerGym
→ 実装重視
```

### 理論研究者向け

```
1. Blondin Automata
2. Computational Complexity
3. Software Foundations
4. Category Theory for Programmers
5. Backprop as Functor
→ 数学的厳密性重視
```

### コンパイラ開発者向け

```
1. Software Foundations Vol.1-2
2. CompCert論文
3. CompCert ソースコード
4. Alive2
5. Blondin Automata
→ 検証重視
```

---

## 🔄 依存関係グラフ

```
Deep Learning Book
    │
    ├─→ Attention Is All You Need
    │       │
    │       └─→ Mamba
    │
    └─→ Blondin Automata
            │
            └─→ Computational Complexity

Software Foundations Vol.1
    │
    ├─→ Software Foundations Vol.2
    │       │
    │       └─→ CompCert
    │
    └─→ Category Theory
            │
            ├─→ Backprop as Functor
            └─→ Types and Programming Languages
```

**読む順序**: 上から下、左から右

---

## 📖 効果的な読み方

### 論文の読み方（3パス法）

#### Pass 1: スキミング（10分）
- Abstract
- Introduction の最初と最後
- 全 Figure
- Conclusion

#### Pass 2: 詳細（1時間）
- Method セクション
- 重要な数式
- 実験結果

#### Pass 3: 実装（2-4時間）
- コードを書く
- 再現実験
- 自分の問題に適用

### 教科書の読み方

#### 初回（速読モード）
- 定義と定理をマーク
- 証明はスキップ
- 例題だけ解く

#### 2回目（精読モード）
- 重要な証明を追う
- 練習問題を全て解く
- ノートにまとめる

---

## 🎓 学習追跡シート

### チェックリスト

```markdown
## Week 1-2
- [ ] Deep Learning Book Ch 6 読了
- [ ] Deep Learning Book Ch 10 読了
- [ ] Attention Is All You Need 読了
- [ ] Transformer 実装完了

## Week 3-4
- [ ] Blondin Ch 1-2 読了
- [ ] Blondin Ch 3 読了
- [ ] Blondin Ch 5 読了
- [ ] Weighted Automaton 実装完了

(続く...)
```

### 進捗記録テンプレート

```python
progress = {
    "week": 1,
    "hours_spent": 0,
    "resources_completed": [],
    "implementations": [],
    "questions": []
}
```

---

## 💡 学習のコツ

### 1. 積極的読書
```python
def active_reading(paper):
    while reading:
        if confused:
            implement_concept()  # コードを書く
            draw_diagram()       # 図を描く
            ask_question()       # 質問する
```

### 2. スパイラル学習
- 1周目: 全体像（速読）
- 2周目: 詳細理解（精読）
- 3周目: 実装・応用

### 3. アウトプット
- ブログに書く
- 勉強会で発表
- GitHubで公開

---

## 🆘 困ったときは

### Q: 理解できない数式がある
**A**: 
1. まず具体例で計算
2. NumPyで実装
3. それでも分からなければスキップ（後で戻る）

### Q: Coqの証明が進まない
**A**:
1. `SearchAbout` で関連定理を探す
2. 既存の証明を真似る
3. コミュニティで質問

### Q: 論文が難しすぎる
**A**:
1. 関連するブログ記事を先に読む
2. 実装から理解する（逆アプローチ）
3. 1-2週間後にもう一度挑戦

---

## 🌟 次のステップ

### 全て読み終わったら

1. **研究**: 未解決問題に挑戦
2. **実装**: 大きなプロジェクト
3. **貢献**: オープンソース
4. **教育**: 知識を共有

---

**この順番で読めば、確実に理解が深まります！**

---

# 📚 完全リソースリスト

## 1. 教科書（分野別・難易度順）

### 1.1 機械学習・深層学習

#### ⭐⭐⭐⭐⭐ 必読
1. **Deep Learning** (Goodfellow, Bengio, Courville, 2016)
   - **URL**: https://www.deeplearningbook.org/
   - **無料**: 完全無料オンライン版
   - **日本語版**: あり
   - **難易度**: 初級〜中級
   - **おすすめ章**: 6, 7, 8, 10, 11

2. **Understanding Deep Learning** (Prince, 2023)
   - **URL**: https://udlbook.github.io/udlbook/
   - **無料**: 完全無料
   - **特徴**: 最新内容、図が豊富
   - **難易度**: 初級

#### ⭐⭐⭐⭐ 推奨
3. **Pattern Recognition and Machine Learning** (Bishop, 2006)
   - **特徴**: 数学的に厳密
   - **難易度**: 中級〜上級

### 1.2 オートマトン理論・計算理論

#### ⭐⭐⭐⭐⭐ 必読
1. **Automata Theory: An Algorithmic Approach** (Blondin, 2024)
   - **URL**: https://michaelblondin.com/automata/
   - **無料**: 完全無料
   - **特徴**: 実装重視、Weighted Automata
   - **難易度**: 初級〜中級

#### ⭐⭐⭐⭐ 推奨
2. **Introduction to Automata Theory, Languages, and Computation** (Hopcroft, Ullman, 2006)
   - **特徴**: 古典的名著
   - **難易度**: 中級

3. **Computational Complexity: A Modern Approach** (Arora, Barak, 2009)
   - **URL**: https://theory.cs.princeton.edu/complexity/
   - **無料**: PDF版あり
   - **難易度**: 上級

### 1.3 圏論

#### ⭐⭐⭐⭐⭐ 必読
1. **Category Theory for Programmers** (Milewski, 2018)
   - **URL**: https://github.com/hmemcpy/milewski-ctfp-pdf
   - **無料**: 完全無料
   - **特徴**: プログラマ向け、Haskell実装
   - **難易度**: 初級〜中級
   - **動画**: https://www.youtube.com/playlist?list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_

#### ⭐⭐⭐⭐ 推奨
2. **Seven Sketches in Compositionality** (Fong & Spivak, 2018)
   - **URL**: https://arxiv.org/abs/1803.05316
   - **特徴**: 応用圏論、図式豊富
   - **難易度**: 初級

#### ⭐⭐⭐ 上級者向け
3. **Categories for the Working Mathematician** (Mac Lane, 1978)
   - **特徴**: 標準的教科書
   - **難易度**: 上級（数学科向け）

### 1.4 型理論・論理学

#### ⭐⭐⭐⭐⭐ 必読
1. **Types and Programming Languages** (Pierce, 2002)
   - **特徴**: 型理論の古典
   - **難易度**: 中級
   - **内容**: Curry-Howard対応

#### ⭐⭐⭐⭐ 推奨
2. **Homotopy Type Theory** (Univalent Foundations, 2013)
   - **URL**: https://homotopytypetheory.org/book/
   - **無料**: 完全無料
   - **難易度**: 上級

### 1.5 定理証明支援系

#### ⭐⭐⭐⭐⭐ 必読
1. **Software Foundations** (Pierce et al.)
   - **URL**: https://softwarefoundations.cis.upenn.edu/
   - **無料**: 完全無料
   - **難易度**: 初級〜上級
   - **Volume 1**: Logical Foundations（Coq入門）
   - **Volume 2**: Programming Language Foundations
   - **Volume 3**: Verified Functional Algorithms

2. **Theorem Proving in Lean 4**
   - **URL**: https://leanprover.github.io/theorem_proving_in_lean4/
   - **無料**: 完全無料
   - **難易度**: 初級〜中級

#### ⭐⭐⭐⭐ 推奨
3. **Concrete Semantics** (Nipkow & Klein, 2014)
   - **URL**: http://concrete-semantics.org/
   - **ツール**: Isabelle/HOL
   - **難易度**: 中級

---

## 2. 重要論文（分野別・年代順）

### 2.1 Transformer関連

#### 2017: ⭐⭐⭐⭐⭐ Attention Is All You Need
- **URL**: https://arxiv.org/abs/1706.03762
- **著者**: Vaswani et al.
- **読む時期**: Week 1-2
- **重要度**: 最重要

#### 2018: ⭐⭐⭐⭐ BERT
- **URL**: https://arxiv.org/abs/1810.04805
- **著者**: Devlin et al.
- **読む時期**: Week 1-2（オプション）

#### 2020: ⭐⭐⭐⭐ Vision Transformer (ViT)
- **URL**: https://arxiv.org/abs/2010.11929
- **読む時期**: Week 9-10（オプション）

### 2.2 効率的Attention

#### 2022: ⭐⭐⭐⭐ FlashAttention
- **URL**: https://arxiv.org/abs/2205.14135
- **著者**: Dao, Fu, Ermon, Rudra, Ré
- **読む時期**: Week 9-10（推奨）

#### 2023: ⭐⭐⭐⭐⭐ FlashAttention-2
- **URL**: https://arxiv.org/abs/2307.08691
- **読む時期**: Week 9-10（推奨）

#### 2023: ⭐⭐⭐ Grouped-Query Attention
- **URL**: https://arxiv.org/abs/2305.13245
- **読む時期**: Week 9-10（オプション）

### 2.3 State Space Models

#### 2022: ⭐⭐⭐⭐ S4: Efficiently Modeling Long Sequences
- **URL**: https://arxiv.org/abs/2111.00396
- **著者**: Gu, Goel, Ré
- **読む時期**: Week 9（Mambaの前に）

#### 2023: ⭐⭐⭐⭐⭐ Mamba
- **URL**: https://arxiv.org/abs/2312.00752
- **著者**: Gu & Dao
- **読む時期**: Week 9-10
- **重要度**: 最重要

#### 2024: ⭐⭐⭐⭐ Mamba-2
- **URL**: https://arxiv.org/abs/2405.21060
- **読む時期**: Week 17+（上級）

### 2.4 圏論 × 機械学習

#### 2019: ⭐⭐⭐⭐⭐ Backprop as Functor
- **URL**: https://arxiv.org/abs/1711.10455
- **著者**: Fong, Spivak, Tuyéras
- **読む時期**: Week 7-8
- **重要度**: 最重要

#### 2021: ⭐⭐⭐⭐ Categorical Semantics of Neural Networks
- **URL**: https://arxiv.org/abs/2101.05084
- **著者**: Shiebler, Gavranović, Wilson
- **読む時期**: Week 9-10（標準パス）

#### 2022: ⭐⭐⭐ Categorical Foundations of Gradient-Based Learning
- **URL**: https://arxiv.org/abs/2103.01931
- **著者**: Cruttwell et al.
- **読む時期**: Week 11-12（標準パス）

### 2.5 形式言語理論 × NN

#### 1995: ⭐⭐⭐⭐ On the Computational Power of Neural Nets
- **著者**: Siegelmann & Sontag
- **読む時期**: Week 3-4

#### 2021: ⭐⭐⭐⭐ Thinking Like Transformers
- **URL**: https://arxiv.org/abs/2106.06981
- **著者**: Weiss et al.
- **読む時期**: Week 3-4

### 2.6 形式検証

#### 2009: ⭐⭐⭐⭐⭐ CompCert
- **URL**: https://xavierleroy.org/publi/compcert-CACM.pdf
- **著者**: Leroy
- **読む時期**: Week 11-12

#### 2017: ⭐⭐⭐⭐ Reluplex: An Efficient SMT Solver for NN Verification
- **URL**: https://arxiv.org/abs/1702.01135
- **著者**: Katz et al.
- **読む時期**: Week 13-14（標準パス）

#### 2021: ⭐⭐⭐⭐⭐ α,β-CROWN
- **URL**: https://arxiv.org/abs/2011.13824
- **著者**: Wang et al.
- **読む時期**: Week 15-16（標準パス）

---

## 3. オンラインコース

### 3.1 機械学習（難易度順）

#### ⭐⭐⭐⭐⭐ Stanford CS231n: CNN for Visual Recognition
- **URL**: http://cs231n.stanford.edu/
- **動画**: YouTube（無料）
- **難易度**: 初級〜中級
- **時間**: 約30時間
- **推奨時期**: Week 1-2と並行

#### ⭐⭐⭐⭐⭐ Stanford CS224n: NLP with Deep Learning
- **URL**: http://web.stanford.edu/class/cs224n/
- **動画**: YouTube（無料）
- **難易度**: 中級
- **時間**: 約40時間
- **推奨時期**: Week 3-4と並行

#### ⭐⭐⭐⭐ MIT 6.S191: Introduction to Deep Learning
- **URL**: http://introtodeeplearning.com/
- **動画**: YouTube（無料）
- **難易度**: 初級
- **時間**: 約10時間
- **推奨時期**: Week 1（概要把握）

### 3.2 圏論

#### ⭐⭐⭐⭐⭐ Category Theory for Programmers (動画)
- **URL**: https://www.youtube.com/playlist?list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_
- **講師**: Bartosz Milewski
- **難易度**: 初級〜中級
- **時間**: 約30時間
- **推奨時期**: Week 7-8と並行

#### ⭐⭐⭐ Applied Category Theory (MIT)
- **URL**: https://ocw.mit.edu/courses/mathematics/18-s097-applied-category-theory-january-iap-2019/
- **難易度**: 中級
- **推奨時期**: Week 9-10（標準パス）

### 3.3 定理証明

#### ⭐⭐⭐⭐ Formal Reasoning About Programs (MIT 6.822)
- **URL**: http://adam.chlipala.net/frap/
- **ツール**: Coq
- **難易度**: 上級
- **推奨時期**: Week 17+

---

## 4. ツール・ライブラリ

### 4.1 機械学習フレームワーク

#### ⭐⭐⭐⭐⭐ PyTorch
```bash
pip install torch torchvision
```
- **URL**: https://pytorch.org/
- **特徴**: 研究向け、動的グラフ
- **学習曲線**: 緩やか
- **推奨時期**: Week 1から

#### ⭐⭐⭐⭐ JAX
```bash
pip install jax jaxlib
```
- **URL**: https://github.com/google/jax
- **特徴**: 関数型、JIT、自動微分
- **学習曲線**: やや急
- **推奨時期**: Week 9（SSM実装時）

#### ⭐⭐⭐ MLX (Apple Silicon)
```bash
pip install mlx
```
- **URL**: https://github.com/ml-explore/mlx
- **特徴**: Apple Silicon最適化
- **推奨時期**: Macユーザーは Week 1から

### 4.2 効率的実装

#### ⭐⭐⭐⭐ Flash-Attention
```bash
pip install flash-attn
```
- **URL**: https://github.com/Dao-AILab/flash-attention
- **推奨時期**: Week 9-10

#### ⭐⭐⭐⭐⭐ Mamba
```bash
pip install mamba-ssm
```
- **URL**: https://github.com/state-spaces/mamba
- **推奨時期**: Week 9-10

#### ⭐⭐⭐ Triton
```bash
pip install triton
```
- **URL**: https://github.com/openai/triton
- **用途**: カスタムGPUカーネル
- **推奨時期**: Week 17+（上級）

### 4.3 定理証明支援系

#### ⭐⭐⭐⭐⭐ Coq
```bash
opam install coq
# または
brew install coq  # macOS
```
- **URL**: https://coq.inria.fr/
- **IDE**: CoqIDE, VSCode + VsCoq
- **推奨時期**: Week 5

#### ⭐⭐⭐⭐⭐ Lean 4
```bash
curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh
```
- **URL**: https://leanprover.github.io/
- **IDE**: VSCode + Lean 4 extension
- **推奨時期**: Week 5（Coqの代わりに）

#### ⭐⭐⭐ Dafny
```bash
dotnet tool install --global dafny
```
- **URL**: https://dafny.org/
- **推奨時期**: Week 11-12（実用的検証）

### 4.4 圏論プログラミング

#### ⭐⭐⭐⭐ Catlab.jl
```julia
using Pkg
Pkg.add("Catlab")
```
- **URL**: https://github.com/AlgebraicJulia/Catlab.jl
- **言語**: Julia
- **推奨時期**: Week 7-8

### 4.5 コンパイラ・検証

#### ⭐⭐⭐⭐ CompilerGym
```bash
pip install compiler-gym
```
- **URL**: https://github.com/facebookresearch/CompilerGym
- **用途**: RL による最適化
- **推奨時期**: Week 11-12

#### ⭐⭐⭐ Alive2
- **URL**: https://github.com/AliveToolkit/alive2
- **用途**: LLVM最適化の検証
- **推奨時期**: Week 17+

#### ⭐⭐⭐⭐⭐ CompCert
- **URL**: https://compcert.org/
- **特徴**: 検証済みCコンパイラ
- **推奨時期**: Week 11-12（ソース読解）

### 4.6 NN検証

#### ⭐⭐⭐⭐ α,β-CROWN
```bash
git clone https://github.com/Verified-Intelligence/alpha-beta-CROWN
cd alpha-beta-CROWN
pip install -r requirements.txt
```
- **推奨時期**: Week 15-16（標準パス）

#### ⭐⭐⭐ ERAN
- **URL**: https://github.com/eth-sri/eran
- **推奨時期**: Week 15-16（オプション）

---

## 5. 参考サイト・ブログ

### 5.1 インタラクティブ学習

#### ⭐⭐⭐⭐⭐ Annotated Transformer
- **URL**: http://nlp.seas.harvard.edu/annotated-transformer/
- **特徴**: コード付き解説
- **推奨時期**: Week 1-2

#### ⭐⭐⭐⭐ Neural Network Playground
- **URL**: https://playground.tensorflow.org/
- **特徴**: 視覚的理解
- **推奨時期**: Week 1

#### ⭐⭐⭐⭐⭐ Distill.pub
- **URL**: https://distill.pub/
- **特徴**: 高品質な視覚的説明
- **推奨**: 随時参照

### 5.2 論文検索・管理

#### Papers with Code
- **URL**: https://paperswithcode.com/
- **用途**: 実装付き論文検索

#### Connected Papers
- **URL**: https://www.connectedpapers.com/
- **用途**: 関連論文探索

#### arXiv Vanity
- **URL**: https://www.arxiv-vanity.com/
- **用途**: arXiv論文を読みやすく

---

## 6. コミュニティ

### 6.1 研究コミュニティ

#### Applied Category Theory
- **URL**: https://appliedcategorytheory.org/
- **カンファレンス**: ACT（年次）
- **参加推奨時期**: Week 9以降

#### Topos Institute
- **URL**: https://topos.institute/
- **特徴**: 応用圏論研究所

### 6.2 主要カンファレンス

- **NeurIPS**: 機械学習最大規模
- **ICML**: 理論寄り
- **ICLR**: 深層学習
- **POPL**: プログラミング言語・形式手法
- **CPP**: 定理証明支援系

---

## 7. 学習ツール

### 7.1 ノート・メモ

- **Obsidian**: マークダウンベース
- **Notion**: オールインワン
- **Zotero**: 論文管理

### 7.2 コード管理

- **GitHub**: バージョン管理
- **Google Colab**: GPU環境
- **Kaggle**: データセット・コンペ

---

**このリソースリストを活用して、効率的に学習を進めてください！**

### 1.1 機械学習・深層学習

#### **Deep Learning** (Goodfellow, Bengio, Courville, 2016)
- **URL**: https://www.deeplearningbook.org/
- **無料**: オンライン版完全無料
- **日本語版**: あり
- **推奨章**:
  - Ch 6-9: 基本アーキテクチャ
  - Ch 10: 系列モデル (RNN, LSTM)
  - Ch 11: 実践的手法

#### **Pattern Recognition and Machine Learning** (Bishop, 2006)
- **特徴**: 数学的に厳密
- **推奨**: 統計的機械学習の基礎

#### **Understanding Deep Learning** (Prince, 2023)
- **URL**: https://udlbook.github.io/udlbook/
- **無料**: 完全無料
- **特徴**: 最新の内容、図が豊富

### 1.2 オートマトン理論・計算理論

#### **Automata Theory: An Algorithmic Approach** (Blondin, 2024)
- **URL**: https://michaelblondin.com/automata/
- **特徴**: 実装重視、Weighted Automata
- **推奨度**: ⭐⭐⭐⭐⭐

#### **Introduction to Automata Theory** (Hopcroft, Ullman, 2006)
- **特徴**: 古典的名著
- **内容**: Chomsky階層の完全な説明

#### **Computational Complexity** (Arora, Barak, 2009)
- **URL**: https://theory.cs.princeton.edu/complexity/
- **無料**: PDF版
- **内容**: P, NP, PSPACE等

### 1.3 圏論

#### **Category Theory for Programmers** (Milewski, 2018)
- **URL**: https://github.com/hmemcpy/milewski-ctfp-pdf
- **無料**: 完全無料
- **特徴**: プログラマ向け、Haskell例
- **推奨度**: ⭐⭐⭐⭐⭐

#### **Seven Sketches in Compositionality** (Fong & Spivak, 2018)
- **URL**: https://arxiv.org/abs/1803.05316
- **特徴**: 応用圏論、図式豊富
- **推奨**: 初学者向け

#### **Categories for the Working Mathematician** (Mac Lane, 1978)
- **特徴**: 標準的教科書（難しい）
- **推奨**: 数学科向け

### 1.4 型理論・論理学

#### **Types and Programming Languages** (Pierce, 2002)
- **特徴**: 型理論の古典
- **内容**: Curry-Howard対応

#### **Homotopy Type Theory** (Univalent Foundations, 2013)
- **URL**: https://homotopytypetheory.org/book/
- **無料**: 完全無料
- **特徴**: 最新の型理論

### 1.5 定理証明支援系

#### **Software Foundations** (Pierce et al.)
- **URL**: https://softwarefoundations.cis.upenn.edu/
- **無料**: 完全無料
- **推奨度**: ⭐⭐⭐⭐⭐
- **内容**:
  - Vol 1: Logical Foundations (Coq入門)
  - Vol 2: Programming Language Foundations
  - Vol 3: Verified Functional Algorithms

#### **Theorem Proving in Lean 4**
- **URL**: https://leanprover.github.io/theorem_proving_in_lean4/
- **無料**: 完全無料
- **特徴**: Lean 4公式チュートリアル

#### **Concrete Semantics** (Nipkow & Klein, 2014)
- **URL**: http://concrete-semantics.org/
- **特徴**: Isabelle/HOL, プログラム意味論

---

## 2. 重要論文（年代順）

### 2.1 基礎論文

#### **1995: On the Computational Power of Neural Nets**
- **著者**: Siegelmann & Sontag
- **内容**: RNNのチューリング完全性
- **重要度**: ⭐⭐⭐⭐

#### **2017: Attention Is All You Need**
- **URL**: https://arxiv.org/abs/1706.03762
- **著者**: Vaswani et al.
- **内容**: Transformer の原論文
- **重要度**: ⭐⭐⭐⭐⭐

#### **2018: BERT**
- **URL**: https://arxiv.org/abs/1810.04805
- **著者**: Devlin et al.
- **内容**: Non-Autoregressive の重要性

### 2.2 圏論×ML

#### **2019: Backprop as Functor**
- **URL**: https://arxiv.org/abs/1711.10455
- **著者**: Fong, Spivak, Tuyéras
- **内容**: Backpropagation の圏論的定式化
- **重要度**: ⭐⭐⭐⭐⭐

#### **2021: Categorical Semantics of Neural Networks**
- **URL**: https://arxiv.org/abs/2101.05084
- **著者**: Shiebler, Gavranović, Wilson
- **内容**: NNの圏論的意味論

#### **2022: Categorical Foundations of Gradient-Based Learning**
- **URL**: https://arxiv.org/abs/2103.01931
- **著者**: Cruttwell et al.
- **内容**: 微分可能プログラミング

### 2.3 State Space Models

#### **2022: Efficiently Modeling Long Sequences (S4)**
- **URL**: https://arxiv.org/abs/2111.00396
- **著者**: Gu, Goel, Ré
- **内容**: Structured State Space Models
- **重要度**: ⭐⭐⭐⭐

#### **2023: Mamba**
- **URL**: https://arxiv.org/abs/2312.00752
- **著者**: Gu & Dao
- **内容**: Selective State Space Models
- **重要度**: ⭐⭐⭐⭐⭐

#### **2024: Mamba-2**
- **URL**: https://arxiv.org/abs/2405.21060
- **内容**: 構造化状態空間の改良

### 2.4 効率的実装

#### **2022: FlashAttention**
- **URL**: https://arxiv.org/abs/2205.14135
- **著者**: Dao, Fu, Ermon, Rudra, Ré
- **内容**: IO-aware Attention

#### **2023: FlashAttention-2**
- **URL**: https://arxiv.org/abs/2307.08691
- **内容**: さらなる高速化

#### **2023: GQA (Grouped-Query Attention)**
- **URL**: https://arxiv.org/abs/2305.13245
- **著者**: Ainslie et al.
- **内容**: KV cache の効率化

### 2.5 形式検証

#### **2009: CompCert**
- **URL**: https://xavierleroy.org/publi/compcert-CACM.pdf
- **著者**: Leroy
- **内容**: 検証済みCコンパイラ
- **重要度**: ⭐⭐⭐⭐⭐

#### **2017: Formal Verification of Neural Networks**
- **URL**: https://arxiv.org/abs/1702.01135
- **著者**: Katz et al.
- **内容**: Reluplexアルゴリズム

#### **2021: Fast and Complete NN Verification (α,β-CROWN)**
- **URL**: https://arxiv.org/abs/2011.13824
- **著者**: Wang et al.
- **内容**: 最先端のNN検証

---

## 3. オンラインコース

### 3.1 機械学習

#### **Stanford CS231n: CNN for Visual Recognition**
- **URL**: http://cs231n.stanford.edu/
- **講義動画**: YouTube無料
- **課題**: PyTorch実装
- **推奨度**: ⭐⭐⭐⭐⭐

#### **Stanford CS224n: NLP with Deep Learning**
- **URL**: http://web.stanford.edu/class/cs224n/
- **内容**: RNN, LSTM, Transformer
- **推奨度**: ⭐⭐⭐⭐⭐

#### **MIT 6.S191: Introduction to Deep Learning**
- **URL**: http://introtodeeplearning.com/
- **特徴**: 短期集中（1週間）
- **推奨**: 概要把握

### 3.2 圏論

#### **Category Theory for Programmers (動画)**
- **URL**: https://www.youtube.com/playlist?list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_
- **講師**: Bartosz Milewski
- **推奨度**: ⭐⭐⭐⭐⭐

#### **Applied Category Theory (MIT)**
- **URL**: https://ocw.mit.edu/courses/mathematics/18-s097-applied-category-theory-january-iap-2019/
- **特徴**: 応用重視

### 3.3 定理証明

#### **Formal Reasoning About Programs (MIT 6.822)**
- **URL**: http://adam.chlipala.net/frap/
- **ツール**: Coq
- **推奨**: 上級者向け

---

## 4. 実装ツール・ライブラリ

### 4.1 機械学習フレームワーク

#### **PyTorch**
- **URL**: https://pytorch.org/
- **特徴**: 動的グラフ、研究向け
- **推奨度**: ⭐⭐⭐⭐⭐

#### **JAX**
- **URL**: https://github.com/google/jax
- **特徴**: 関数型、自動微分、JIT
- **推奨**: State Space Models

#### **MLX (Apple, 2024)**
- **URL**: https://github.com/ml-explore/mlx
- **特徴**: Apple Silicon最適化

### 4.2 効率的実装

#### **Triton (OpenAI)**
- **URL**: https://github.com/openai/triton
- **用途**: GPU kernel の高レベル記述
- **推奨**: FlashAttention実装

#### **Flash-Attention**
- **URL**: https://github.com/Dao-AILab/flash-attention
- **特徴**: 公式実装

#### **Mamba**
- **URL**: https://github.com/state-spaces/mamba
- **特徴**: 公式実装

### 4.3 定理証明支援系

#### **Coq**
```bash
# インストール
opam install coq
```
- **URL**: https://coq.inria.fr/

#### **Lean 4**
```bash
# インストール
curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh
```
- **URL**: https://leanprover.github.io/

#### **Dafny**
```bash
# インストール
dotnet tool install --global dafny
```
- **URL**: https://dafny.org/

### 4.4 圏論プログラミング

#### **Catlab.jl**
- **URL**: https://github.com/AlgebraicJulia/Catlab.jl
- **言語**: Julia
- **用途**: 圏論的プログラミング

### 4.5 コンパイラ関連

#### **CompilerGym**
- **URL**: https://github.com/facebookresearch/CompilerGym
- **用途**: 強化学習によるコンパイラ最適化

#### **Alive2**
- **URL**: https://github.com/AliveToolkit/alive2
- **用途**: LLVM最適化の自動検証

#### **CompCert**
- **URL**: https://compcert.org/
- **特徴**: 検証済みCコンパイラ

### 4.6 NN検証

#### **α,β-CROWN**
- **URL**: https://github.com/Verified-Intelligence/alpha-beta-CROWN
- **用途**: NN頑健性検証

#### **ERAN**
- **URL**: https://github.com/eth-sri/eran
- **用途**: ETH発のNN検証器

---

## 5. コミュニティ・カンファレンス

### 5.1 研究コミュニティ

#### **Applied Category Theory**
- **URL**: https://appliedcategorytheory.org/
- **カンファレンス**: ACT (年次)
- **特徴**: ML応用の発表多数

#### **Topos Institute**
- **URL**: https://topos.institute/
- **特徴**: 応用圏論の研究所

### 5.2 主要カンファレンス

#### **NeurIPS**
- **分野**: 機械学習全般
- **特徴**: 最大規模

#### **ICML**
- **分野**: 機械学習
- **特徴**: 理論寄り

#### **ICLR**
- **分野**: 深層学習
- **特徴**: オープンレビュー

#### **POPL**
- **分野**: プログラミング言語
- **特徴**: 形式手法、型理論

#### **CPP**
- **分野**: 証明支援系
- **特徴**: Coq, Lean等

---

## 6. 論文の読み方

### 6.1 効率的な読み方

```
Phase 1: スキミング (10分)
- Abstract
- Introduction の最初と最後
- Figures (全て)
- Conclusion

Phase 2: 本文 (30分)
- Method セクション
- 重要な Figure の詳細

Phase 3: 深読み (必要なら)
- 数式の導出
- 実験の詳細
- 補足資料
```

### 6.2 実装しながら読む

```python
# 論文を読みながらコードを書く
paper = read("Attention Is All You Need")

# 理解を深める
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, n_heads):
        # 論文の数式を実装
        pass
```

### 6.3 推奨ツール

- **arXiv**: https://arxiv.org/
- **Papers with Code**: https://paperswithcode.com/
- **arXiv Vanity**: https://www.arxiv-vanity.com/ (読みやすい形式)
- **Connected Papers**: https://www.connectedpapers.com/ (関連論文探索)

---

## 7. 学習計画テンプレート

### 7.1 基本パス (12週間)

```
Week 1-2: 機械学習の分類
  - Deep Learning Book Ch 6-9
  - PyTorch Tutorial
  - 実装: 簡単なRNN, Transformer

Week 3-4: オートマトン理論
  - Blondin Automata Theory Ch 1-6
  - 実装: Weighted Automaton
  
Week 5-6: 定理証明入門
  - Software Foundations Vol 1
  - 実装: 簡単なCoq証明

Week 7-8: 圏論基礎
  - Category Theory for Programmers Ch 1-5
  - 実装: Haskellで圏論的NN

Week 9-10: State Space Models
  - Mamba論文
  - 実装: 簡易Mamba

Week 11-12: 統合プロジェクト
  - 検証済みコンパイラ最適化
  - または NN検証
```

### 7.2 完全パス (36週間)

全ファイルを順番に完全学習

---

## 8. よくある質問 (FAQ)

### Q1: どこから始めればいい？

**A**: `01_ML_Classification.md` から始めてください。基礎がない場合は Deep Learning Book を並行して読むと良いです。

### Q2: 数学の前提知識は？

**A**: 
- 必須: 線形代数、微積分
- 推奨: 確率論、集合論
- あると良い: 関数型プログラミング

### Q3: プログラミング言語は？

**A**:
- Python: 必須
- Haskell: 圏論の理解に有用
- Coq/Lean: 証明支援系の学習に必要

### Q4: どれくらい時間がかかる？

**A**:
- 基本パス: 3ヶ月（週10時間）
- 標準パス: 6ヶ月（週10時間）
- 完全パス: 9-12ヶ月（週10時間）

### Q5: 独学可能？

**A**: 可能です。このガイドは独学を前提に設計されています。ただし、コミュニティでの議論を強く推奨します。

---

## 9. 次のステップ

### 9.1 さらなる学習

- **幾何学的深層学習**: Bronstein et al.
- **確率的プログラミング**: Pyro, Stan
- **微分可能プログラミング**: Zygote.jl

### 9.2 研究への道

1. 最新論文を追う (arXiv)
2. 実装を公開 (GitHub)
3. コミュニティに参加
4. 自分の問題を見つける

### 9.3 実務への応用

- ML コンパイラ (TVM, XLA)
- 形式検証
- 自動定理証明
- プログラム合成

---

## 📝 このガイドへの貢献

フィードバック歓迎:
- 誤りの報告
- 追加リソースの提案
- 演習問題の解答例

---

**学習を楽しんでください！**
