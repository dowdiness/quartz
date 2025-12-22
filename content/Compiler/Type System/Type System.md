---
title: Type System
publish: false
tags: [compiler, logic]
created: 2025-05-07T13:07:34+09:00
modified: 2025-12-22T20:40:42+09:00
aliases: [Type Theory, 型システム, 型理論]
---

# Type System

[The Rise of Type Theory](https://pling.jondgoodwin.com/post/rise-of-type-theory/)

[論理学入門](https://scrapbox.io/sno2wman/%E8%AB%96%E7%90%86%E5%AD%A6%E3%82%92%E7%8B%AC%E7%BF%92%E3%81%97%E3%81%9F%E3%81%84%E3%81%82%E3%81%AA%E3%81%9F%E3%81%AB)

## 資料

[learn-tt](https://github.com/jozefg/learn-tt)
https://github.com/steshaw/plt

[Mathematical Logic as based on the Theory of Types.](https://www.jstor.org/stable/pdf/2369948.pdf)

[ラッセルのパラドックス](https://ja.wikipedia.org/wiki/%E3%83%A9%E3%83%83%E3%82%BB%E3%83%AB%E3%81%AE%E3%83%91%E3%83%A9%E3%83%89%E3%83%83%E3%82%AF%E3%82%B9)を避けるためにラッセルが書いた、初めて型という概念が登場する論文。

https://plato.stanford.edu/entries/type-theory/


[TYPES IN LOGIC AND MATHEMATICS BEFORE 1940](https://www.macs.hw.ac.uk/~fairouz/forest/papers/journals-publications/bsl02/bsl02.pdf)

1940年に単純型付きラムダ計算が登場するまでの論理学上の型理論の歴史についてまとめた論文

[Set-Theoretic and Type-Theoretic Ordinals Coincide](https://arxiv.org/abs/2301.10696)

[Programming Language Theory and its Implementation](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=e94334412e47eb35ebb418037f7279a61021151a)

[Type Inference Logics](https://dl.acm.org/doi/10.1145/3689786)

## Curry-style と Church-style

Curry-styleでは既に存在する項に対してextrinsicな型付けを行い、Church-styleでは型付けされた項しか存在できない、つまりintrinsicな型付けが行われる。

https://www.cannorin.net/blog/2018-07-07-difference