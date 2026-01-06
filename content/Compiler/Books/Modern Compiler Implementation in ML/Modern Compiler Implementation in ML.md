---
title: Modern Compiler Implementation in ML
publish: false
tags: [compiler]
aliases: [最新コンパイラ構成技法]
created: 2025-05-07T23:34:35+09:00
modified: 2025-05-08T00:58:45+09:00
---

# Modern Compiler Implementation in ML
日本語: 最新コンパイラ構成技法

## 6. 駆動レコード(Activation Records)

### 6.1 スタックフレーム

### 6.2 Tiger コンパイラのフレーム

### 6.3 
LispとAlgolから変数と戻り番地を連続的なStackへと持たせる手法は始まった。
[Recursive Functions of Symbolic Expressions Their Computation by Machine, Part I](https://dl.acm.org/doi/pdf/10.1145/367177.367199)
[THE DESIGN OF THE GIER ALGOL COMPILER](https://www.cs.tufts.edu/comp/150FP/archive/peter-naur/gier-compiler.pdf)

1960,70年代の殆どのコンパイラは変数をメモリに持たせていたので、変数がエスケープする心配もないため番地の必要性も無かった。

Risc革命
https://pages.cs.wisc.edu/~markhill/restricted/ieeecomputer85_cisc.pdf

G. J. Chaitin. 1982. Register allocation &amp; spilling via graph coloring. SIGPLAN Not. 17, 6 (June 1982), 98–101. https://doi.org/10.1145/872726.806984

G. J. Chaitin. 1982. Register allocation &amp; spilling via graph coloring. In Proceedings of the 1982 SIGPLAN symposium on Compiler construction (SIGPLAN '82). Association for Computing Machinery, New York, NY, USA, 98–105. https://doi.org/10.1145/800230.806984


## 15. 関数型プログラミング言語

純関数型プログラミング言語とは等式推論(equational reasoning)が数学と同じように機能するプログラミング方式です。

高階関数は言語が静的スコープ・字句スコープ(lexical scope)を持つ入れ子になった関数を持つ場合に重要になる。高階関数型言語(higher-order functional language)とは静的スコープと高階関数を有する言語です。

### この章で実装する３種類の関数型言語

- Fun-Tiger: 副作用のない高階関数を有する言語。(impure, higher-order functional language)
- PureFun-Tiger: 副作用のない高階関数を有する正格言語(strict, pure functional language)
- Lazy-Tiger: 遅延評価を有する純関数型言語(non-strict, pure functional language)

### 15.1 Fun-Tiger

Tigerに追加する関数型

```tiger
ty → ty -> ty
   → (ty {, ty}) -> ty
   → () -> ty
```

Call式の変更

```
exp → exp (exp {, exp})
exp → exp ()
```

### 15.2 閉包(Closure)

ヒープに割り付けられた駆動レコード

### 15.3変更不能変数

### 15.4 インライン展開

### 15.5 閉包変換

### 15.6 効率的な末尾再帰

### 15.7 遅延評価
call-by-value
call-by-need

#### 遅延評価関数型プログラムの最適化
deforestation

#### ストリクト解析

## 脚注

[The mechanical evaluation of expressions by P.J. Landin](https://www.cs.cmu.edu/~crary/819-f09/Landin64.pdf)
ヒープに割り当てた閉包を用いて抽象マシン上でλ計算をどのように解釈実行するのか示した。
[One Weird Trick to Untie Landin’s Knot](https://www.williamjbowman.com/resources/hope2023-landins-knot.pdf)
最近の論文これも面白いかも