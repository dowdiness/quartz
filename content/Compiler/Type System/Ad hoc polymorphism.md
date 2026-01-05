---
title: Ad hoc polymorphism
created: 2025-04-15T14:22:55+09:00
modified: 2026-01-06T02:47:57+09:00
tags: [compiler, polymorphism, type-system]
aliases: [アドホック多相]
---

# Ad hoc polymorphism

[How to make ad-hoc polymorphism less ad hoc](https://dl.acm.org/doi/10.1145/75277.75283)

> Ad-hoc polymorphism occurs when a function is defined over several diflerent types, acting in a different way for each type. A typical example is overloaded multiplication: the same symbol may be used to denote multiplication of integers (as in `3*3`) and multiplication of floating point values (as in `3.14*3.14`).

> Parametric polymorphism occurs when a function is defined over a range of types, acting in the same way for each type. A typical example is the length function, which acts in the same way on a list of integers and a list of floating point numbers.

[Ad-hoc polymorphic delimited continuations](https://arxiv.org/abs/2307.16073)

## 関連リンク

[[Row Polymorphism]]