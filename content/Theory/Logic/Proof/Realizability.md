---
title: Realizability
created: 2025-04-11T19:33:29+09:00
modified: 2025-12-28T19:51:56+09:00
tags: [logic, math, proof]
---

# Realizability

https://www.williamjbowman.com/blog/2022/10/05/what-is-realizability/

My best understanding of realizability right now, in programming languages (PL) terms, is:

1. A technique for assigning each _syntactic type_ to a collection of _semantic terms_;
2. By _induction_ over syntactic types;
3. Where the semantic terms that are _realizers_—i.e., included in the collection related to some syntactic type—are a sub-collection of all possible terms in the semantic domain. That is, there are valid semantic terms not associated with any syntactic type.

I use the word “collection” rather than “set” to avoid invoking set theory.

![realizability](https://www.williamjbowman.com/img/realizability.png)

[Functional Pearl: a Classical Realizability Program](https://arxiv.org/pdf/1908.09123)
