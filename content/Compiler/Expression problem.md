---
title: Expression problem
date: 2025-04-26
tags: []
created: 2025-04-26T16:37:36+09:00
modified: 2026-01-04T14:18:10+09:00
---

# Expression problem

[Philip Wadler](https://homepages.inf.ed.ac.uk/wadler/)が作った新しいデータ型と関数の追加の大変さがトレードオフの関係になっていることを示す言葉。おそらくこの[文書](https://homepages.inf.ed.ac.uk/wadler/papers/expression/expression.txt)が初出のはず。

パターンマッチを使うような関数型では新しいデータ型を追加する度にそれに関連する全ての関数に手を加えないといけないが、オブジェクト指向の場合は[Visitor パターン](https://ja.wikipedia.org/wiki/Visitor_%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)を使うと既存のコードに変更を加えずに新しいクラスを追加するだけでデータ型の追加が出来る。ただし新しい関数を追加する場合には関数型では既存のコードを変更せずに済むのに対し、オブジェクト指向では関連する全てのクラスを変更する必要が生じて立場が逆転する。

- [The Expression Problem and its solutions](https://eli.thegreenplace.net/2016/the-expression-problem-and-its-solutions)
- [The Expression Problem, Gracefully](https://pdxscholar.library.pdx.edu/cgi/viewcontent.cgi?article=1141&context=compsci_fac)

## Data types à la carte 

[Functional Pearl: Data Types A La Carte](https://www.cambridge.org/core/services/aop-cambridge-core/content/view/14416CB20C4637164EA9F77097909409/S0956796808006758a.pdf/data-types-a-la-carte.pdf) - Wouter Swierstra

Expression problem のHaskellでの解決方法。
 
[Data types à la carte in TypeScript](https://susisu.hatenablog.com/entry/2024/12/04/210921)

TypeScriptでの実践 