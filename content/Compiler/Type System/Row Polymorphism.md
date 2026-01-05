---
title: Row Polymorphism
created: 2024-06-25T20:22:42+09:00
modified: 2026-01-06T03:10:00+09:00
tags: [compiler, polymorphism, type-system]
---

# Row Polymorphism

[wiki](https://en.wikipedia.org/wiki/Row_polymorphism)

[入門記事](https://jadon.io/blog/row-polymorphism/)

[A Polymorphic Type System for Extensible Records and Variants](https://web.cecs.pdx.edu/~mpj/pubs/96-3.pdf)

[Abstracting Extensible Data Types](https://homepage.cs.uiowa.edu/~jgmorrs/pubs/morris-popl2019-rows.pdf)

## 部分型との違いは？

おそらく論理の導入規則か除去規則の違いによる。言い方を変えると値・項の構築か消費なのか。
部分型は値が使われるときのsubstitutionのルール、Row Polymorphismは値を構築するときのルールのはず。

## 関連リンク

[[Ad hoc polymorphism]]