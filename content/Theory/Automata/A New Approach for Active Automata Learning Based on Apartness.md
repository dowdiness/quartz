---
aliases: [1 Introduction]
created: 2025-04-11T23:40:05+09:00
modified: 2026-01-12T23:44:40+09:00
---

https://link.springer.com/chapter/10.1007/978-3-030-99524-9_12

[最新のオートマトン学習アルゴリズムL#について](https://makenowjust-labs.github.io/blog/post/2024-08-17-lsharp/#ref-vaandrager-et-al-2022)
日本語での解説記事

$L ∗$ アルゴリズムを改良した $L^{\#}$を提案する論文です
Apartnessという概念を新しく導入しているみたい
# 1 Introduction

In 1987, Dana Angluin showed that the class of regular languages can be learned efficiently using queries.
This new approach is called ***minimally adequate teacher (MAT)***, learning is viewed as a game in which a learner has to infer a deterministic finite automaton (DFA) for an unknown regular language $L$ by asking queries to a teacher.

The learner may pose two types of queries:
1. “Is the word w in L?”(membership queries)
2. “Is the language recognized by DFA H equal to L?”(equivalence queries)

The L ∗ algorithm proposed by Angluin [5] is able to learn L by asking a polynomial number of membership and equivalence queries (polynomial in the size of the corresponding canonical DFA).

# 2 Partial Mealy Machines and Apartness

# 3 Learning Algorithm
# References

[Angluin, D.: Learning regular sets from queries and counterexamples. Inf. Comput. **75**(2), 87–106 (1987)](https://www.sciencedirect.com/science/article/pii/0890540187900526)
