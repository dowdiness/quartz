---
title: The Simple Essence of Algebraic Subtyping
aliases: [代数的部分型]
created: 2024-06-27T22:49:26+09:00
modified: 2026-01-11T19:40:49+09:00
tags: [compiler, type-system]
---

# The Simple Essence of Algebraic Subtyping

## これは何？

[The Simple Essence of Algebraic Subtyping](https://lptk.github.io/programming/2020/03/26/demystifying-mlsub.html)を読んだ簡単なまとめです。読んだ時に書いたメモなので内容の正確性や読みやすさへの保証は持てません。詳しくは元論文と[Githubの実装](https://github.com/LPTK/simple-sub)に当たってください。

Algebraic SubtypingのMLsubでは、biunificationというbisubstitutionとpolar typesという新しい概念に依存していて抽象代数に関する深い知識がないと難しいアルゴリズムが使われている。この論文ではMLsub相当の機能を持った代替であるSimple-subというアルゴリズムを提唱している。MLsubに比べSimple-subは500以内のScalaコードで書かれていて抽象代数の知識が無くとも理解がしやすく簡単である。

### この論文の構成

1. MLsubとalgebraic subtypingの理論背景の説明
2. Simple-sub型推論のアルゴリズム
3. Simple-subによって推論された型を単純化する手法
4. 安全性(soundness)と完全性(completeness)の定理を使ったSimple-subの正しさの証明。証明に必要な最低限の部分型関係の形式化。
5. 大量のランダム生成されたプログラムに対して、Simple-subとMLsubの型推論が同じ結果を返すことの検証。

## 1 INTRODUCTION

参考
[Safely typing algebraic effects](http://gallium.inria.fr/blog/safely-typing-algebraic-effects)
Algebraic Effectsのある言語にMLsubの型推論アルゴリズムを組み込む話。
パリでの[MPRI](https://wikimpri.dptinfo.ens-cachan.fr/)というプログラムの中の[Functional programming and type systems](https://gitlab.inria.fr/fpottier/mpri-2.4-public/tree/978160d262d503714e212fc2d28c9451c63fe714)というレクチャーで作った言語。

## 2 ALGEBRAIC SUBTYPING AND MLSUB

### 2.1 Background on Algebraic Subtyping

部分型付けの形式化の手法には少なくとも三種類の派閥があるらしい。
- Syntactic approaches
	- この論文で使われている手法。
	- 直接的な仕様(普通は推論規則)を元に部分型関係を定義する。
	- 型の構文に従う?(closely following the syntax of types)
- Semantic approaches
	- 型を型を持つ値の集合として見る。
	- 部分型関係を集合同士の包含関係として定義する。
	- 参考： [Semantic subtyping: Dealing set-theoretically with function, union, intersection, and negation types](https://dl.acm.org/doi/10.1145/1391289.1391293)
	- https://arxiv.org/pdf/2306.06391
- Algebraic approaches
	- 型を[分配束](https://en.wikipedia.org/wiki/Distributive_lattice)の抽象的な要素として定義する。
	- 型の代数的要素を注意深く選ぶことにより、拡張性や主要型の生成などを実現出来る。
	- 参考： [Polymorphism, subtyping, and type inference in MLsub](https://dl.acm.org/doi/10.1145/3093333.3009882)

#### Syntactic approaches

型システムの設計者に全ての推論規則の結果とインタラクションを考慮させることに強制させることによって、シンプルさを保てる手法。
部分型関係が望まれた代数的要素になっているか手動で確認する必要がある。
#### Semantic approaches

おそらく一番直感的で強力な手法。ただし多相性に関する難しさのせいで、ground types?が含まれているとパラドックスが簡単に生まれてしまい、拡張性が無くなってしまう。詳しくは[Algebraic Subtyping](https://www.cs.tufts.edu/~nr/cs257/archive/stephen-dolan/thesis.pdf)を参照。
#### Algebraic approaches

代数が型システムが
型システムの拡張性というコンセプトを強調していて、新しい型が追加されても既存のプログラムがwell-typedになるべきだとしている。
部分型の束を分配束にすることにより、型を単純化出来て、包摂をチェックするアルゴリズムを効率的に構築出来る。

### 2.2 Basics of MLsub

### 2.3 Informal Semantics of Types

#### 2.3.1 Set-Theoretic Types

input position: `α ⊓ τ`
output position: `α ⊔ τ`

高階型にも適用できる

The beauty of MLsub is that this sort of reasoning scales to arbitrary flows of variables andhigher-order functions; for instance, the previous example can be generalized by passing in afunction f to stand for the · − 1 operation, as in `λ f . λx . { L = f x ; R = x }` whose type can beinferred as `(β → γ ) → α ⊓ β → { L : γ ; R : α }`. Applying this function to argument `(λx . x − 1)` yields the same type (after simplification) as in the example of the previous paragraph.

#### 2.3.2 Recursive Types

`µα . τ`

#### 2.3.3 Typing Surprises

驚くことにMLsubでは従来のMLではill-typedな項をwell-typedな項として型付け出来る。
例えば `λx . x x` のような引数をとってそれ自身を適用するような項は、`∀α, β. (α → β) ⊓ α → β` として型付け出来る。

### 2.4 Expressiveness

#### 2.4.1 Polarity of Type Positions

上記の型は自由に型式として使えるわけではなくpolarityにより分けられて使える場所が決まっている。例えばUnionはpositive typeとして、intersectionはnegative typeとして扱う。
Positive Positionは項のoutputに対応し、Negative Positionは項のinputに対応している。[^1]

`(τ0 → τ1) → τ2` の`τ2`は Positive Position
 `(τ0 → τ1)` Negative Positrion

#### 2.4.2 Consequence of the Polarity Restriction

上記の影響により、MLsubでは `int ⊔ string → τ` のような型は `int ⊔ string` が Negative Position に位置するため書けない。逆に `τ → int ⊔ string` のような型は書ける。これによりMLsubは構造的型付けのJavaと同等の表現力を持つ言語になる。
適切なPolarityで使われたunityとintersectionは間接的な[bounding](https://en.wikipedia.org/wiki/Bounded_quantification)と同じことをしている。

なのでMLsubの

`α ⊓ nat ⊓ odd → { L : int ; R : α ⊔ prime }`

とJava風の

`⟨α super prime extends nat & odd⟩(α) → { L : int ; R : α }`

は同じ意味になる。

### 2.5 Essence of MLsub Type Inference

[Algebraic Subtyping](https://www.cs.tufts.edu/~nr/cs257/archive/stephen-dolan/thesis.pdf) と [Polymorphism, subtyping, and type inference in MLsub](https://dl.acm.org/doi/10.1145/3093333.3009882) を読んだ読者はMLsubの本質はな

- union と intersection を サポートした型での分配束(distributive lattice)
- 従来のアルゴリズムとは違う biunification による型推論
- 型構文での positive type と negative type の分離

と思うかもしれない。

しかし実際にはこれらは理解に必要な本質ではない。

MLsubの本質は

> making the semantics of types simple enough that all inferred subtyping constraints can be reduced to constraints on type variables, which can be recorded efficiently (for instance using mutation, as done in this paper and in MLsub’s actual implementation.

- 部分型制約を型変数への制約だけにまとめることによりシンプルにする。

> using intersection, union, and recursive types to express compact type signatures where type variables are indirectly constrained, avoiding the need for separate constraint specifications

- 型変数への間接的な制約のある場所ではintersection型とunion型、再起型を使うことによって型注釈をコンパクトにする。これにより別の制約を分けて指定する必要がなくす。

ことである。

## 3 THE SIMPLE-SUB TYPE INFERENCE ALGORITHM
### 3.1 Simple-sub Syntax

### 3.2 Basic Type Inference

### 3.3 User-Facing Types Representations
### 3.4 Examples

### 3.5 Let Polymorphism and Recursion
### 3.6 Summary

Simple-subではbiunificationと違い伝統的な型推論アルゴリズムに似たアルゴリズムで同じ結果になる実装を簡単に実現出来る。
MLsubの主要型と部分型を含む型推論にはbisubstitutionとpolar typesは必須ではない。
## 4 SIMPLIFYING TYPES

推論される型は冗長な部分があったり、単一化や除去の出来る型変数を含んでいることがある。コンパクトで理解しやすい型にするためにも、部分型の含む型推論では推論された型の単純化が重要になる。もし単純化をしなければ推論される型の量は線形に増加してしまう。
### 4.1 Type Simplification Tradeoffs
### 4.2 Type Simplification in MLsub

型の集合をfinite-state automataとして表現できる。
automata theoryのtype automataという手法を使ってType Simplificationが実現できるらしい。難しいので今回は使わない。
### 4.3 Type Simplification in Simple-sub

co-occurrence analysis

hash consing

## 5 FORMALIZATION OF SIMPLE-SUB

### 5.1 A Syntax-First Definition of Subtyping

## 6 EXPERIMENTAL EVALUATION
Simple-subとMLsubの実装で、ランダムに生成された1,313,832件の式を評価した結果、515,509件がwell-typedと、798,321件がill-typedと判断されたらしい。

Subsumption checking

Generated expressions

Bugs found in MLsub
MLsubのシャドーイングのバグが見つかった。
## 7 CONCLUSIONS AND FUTURE WORK

[^1]: Polarityについては[The Hidden Data Flow in Types](https://www.youtube.com/watch?v=Uw8ayBv1j7Y)の説明が分かりやすい。
