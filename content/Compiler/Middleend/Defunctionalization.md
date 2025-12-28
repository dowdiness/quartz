---
title: Defunctionalization
publish: true
tags: [compiler]
aliases: [脱関数化, 脱高階関数]
created: 2025-12-28T18:34:11+09:00
modified: 2025-12-28T19:24:57+09:00
---

# Defunctionalization

## 解説

圏論で有名なBartosz Milewskiの解説

[Defunctionalization and Freyd’s Theorem](https://bartoszmilewski.com/2020/08/03/defunctionalization-and-freyds-theorem/)
[Replacing functions with data](https://youtu.be/wppzFzzD4b8?si=ZdwZQpTC_b4bihcG)

## 実装

[Refactoring a lambda calculus interpreter into a stack machine, in Haskell](https://youtu.be/O6s8_4agu4c?si=5NOSrWI4jqG3PZva)
[code](https://gist.github.com/paf31/16f25ef70eaf256fbf301b571db6d533)

λ計算のインタープリタをCPSからDefunctionalizationを使ったものへ、さらにスタックを使ったものへと実際にやりながら解説してくれる神動画

## 論文

[Defunctionalisation as modular closure conversion](https://dl.acm.org/doi/10.1145/3131851.3131868)