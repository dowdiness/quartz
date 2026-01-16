---
title: Pattern Matching
publish: false
tags: [algorithm, compiler]
aliases: [パターンマッチング]
created: 2025-05-23T15:35:25+09:00
modified: 2026-01-15T15:25:40+09:00
---

# Pattern Matching

実はPattern Matching は [Sum type](https://ncatlab.org/nlab/show/sum+type) の除去規則である。

[A generic algorithm for checking exhaustivity of pattern matching](https://infoscience.epfl.ch/entities/publication/bba4145b-7864-4dd7-87d1-dc3e802ee36f)
[GADTs and Exhaustiveness: Looking for the Impossible](https://arxiv.org/abs/1702.02281)

## 実装方法について

[Compiling Pattern Matching](https://compiler.club/compiling-pattern-matching/)

[juvix](https://github.com/anoma/juvix/issues/1798) より

- Philip Wadler, Efficient compilation of pattern matching, in: Simon Peyton Jones, [The implementation of functional programming languages](https://www.microsoft.com/en-us/research/uploads/prod/1987/01/slpj-book-1987.pdf), Chapter 5.
- Luc Maranget, [Compiling pattern matching to good decision trees](http://moscova.inria.fr/~maranget/papers/ml05e-maranget.pdf)
- Jules Jacobs, [How to compile pattern matching](https://julesjacobs.com/notes/patternmatching/patternmatching.pdf)
- Luc Maranget, [Warnings for pattern matching](http://moscova.inria.fr/~maranget/papers/warn/warn.pdf)