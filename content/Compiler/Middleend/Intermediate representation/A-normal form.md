---
tags: [compiler, ir]
aliases: [ANF]
created: 2025-04-18T13:52:21+09:00
modified: 2025-12-23T11:24:54+09:00
---

# A-normal form

## Intro

[[A-normal form - Wikipedia]]

A-normal form(これ以降はANFと書きます)は[[Continuation-passing style]]の代わりの[[Intermediate representation]]として1991年に[Amr Sabry](https://scholar.google.com/citations?user=dYGSG4EAAAAJ)と[Matthias Felleisen](https://en.wikipedia.org/wiki/Matthias_Felleisen)により発明された手法。[Reasoning about programs in continuation-passing style](https://dl.acm.org/doi/10.1145/141471.141563)が初出っぽい。


```link-bookmark
url:https://matt.might.net/articles/a-normalization/
title:A-Normalization: Why and How
description: A-Normal Form is a sweet spot during program compilation.
coverImg:http://matt.might.net/articles/a-normalization/images/anormal-complexity.png
logo:
```

## [ANF Conversion](https://compiler.club/anf-conversion/)

プログラム中の式を中間値の変数へと変換すること。複雑な式から線形的な（実行順序の明確な）式に変換することで以降のパスでの処理が簡単になる。例:[[Three-address code]]やSchemeなど

例えば下記のコードの場合

```txt
foo(6 + 8, 10) * bar(2, id(12))
```

ANFへと変換すればこのようになる

```txt
let t0 = 6 + 8 in
let t1 = foo(t0, 10) in
let t2 = id(12) in
let t3 = bar(2, t2) in
let t4 = t1 * t3 in
t4
```

ANFのメリット

- 複雑な式であっても線形になりことにより実行順序が明確になる
- 関数の引数の処理が簡単になる。上記の場合は変数とIntに変換されている評価の仕方を考えなくても良い。


```txt
Example 1:
Source:  (λx. (x + 1)) (42)
ANF:  let x1 = x2 42 in let x2 = λx. let x3 = x + 1 in x3 in x1

Example 2:
Source:  let x = 10 in (x + 5)
ANF:  let x = λ_. 10 in let x4 = x + 5 in x4

Example 3:
Source:  if (1 + 2) then 3 else 4
ANF:  let x5 = 1 + 2 in if x5 then 3 else 4

Example 4:
Source:  ((λx. λy. (x + y)) (1)) (2)
ANF:  let x6 = x7 2 in let x7 = x8 1 in let x8 = λx. let x9 = λy. let x10 = x + y in x10 in x9 in x6
```


