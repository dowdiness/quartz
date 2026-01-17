---
title: Incremental computation
publish: false
tags: [compiler]
aliases: [self-adjusting computation, インクリメンタル計算]
created: 2026-01-02T02:35:09+09:00
modified: 2026-01-18T02:29:39+09:00
---

# Incremental computation

## What Is the Essence?

[Incremental Computation: What Is the Essence? (Invited Contribution)](https://dl.acm.org/doi/10.1145/3635800.3637447)
[Video](https://www.youtube.com/watch?v=jRgL41Tn-xs)

### Incremental algorithms

Algorithms for computing particular functions, such as shortest paths, under particular kinds of input changes, such as adding and deleting edges. This includes algorithms known as dynamic algorithms, online algorithms, and other variants.

### Incremental program-evaluation frameworks

Frameworks for evaluating general classes of programs expressed in the framework and handling input changes. This includes frameworks known as memoization, caching,  tabling, change propagation, and other variants.

### Incremental-algorithm-and-program derivation methods

Methods for deriving algorithms and programs that handle input changes from given algorithms or programs and given kinds of input changes. This includes methods known as finite differencing, strengthening and maintaining loop invariants, incrementalization, and other variants.

[adaption](https://github.com/Adapton/adapton.rust)

[Seven Implementations of Incremental](https://www.youtube.com/watch?v=G6a5G5i4gQU)
[online algorithms](https://www.cs.cmu.edu/~avrim/451f13/lectures/lect1107.pdf)の紹介動画

[Deriving Incremental Programs](https://www.cs.cornell.edu/info/people/tt/research/incremental-computation/derivation.html)

[Incremental computation with names](https://dl.acm.org/doi/10.1145/2814270.2814305)

[Incremental Computing by Differential Execution](https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ECOOP.2025.20)

## Blog

[Rado's Incremental cmputaion](https://rkirov.github.io/posts/incremental-computation/)

## Video

[A journey through incremental computation - Raph Levien](https://youtu.be/DSuX-LIAU-I?si=ujPo1l5KibwxFJBo)

## Implementation

[Adaptive](https://hackage.haskell.org/package/Adaptive): Library for incremental computing.
[Adaptive (Self-Adjusting) Computations in JS](https://github.com/rkirov/adapt-comp)


## Type Checker

[A Systematic Approach to Deriving Incremental Type Checkers](https://dl.acm.org/doi/10.1145/3428195)
[Incremental type-checking for free](https://dl.acm.org/doi/10.1145/3563303)

## Self-Adjusting Computation

[Incremental](https://github.com/janestreet/incremental?tab=readme-ov-file)
OCaml library