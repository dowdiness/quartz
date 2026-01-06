---
title: Garbage Collection
publish: false
tags: [" "]
aliases: [Theory of Garbage Collection, Untitled]
created: 2025-04-30T16:49:54+09:00
modified: 2025-04-30T20:31:47+09:00
---

# Garbage Collection

[Wiki](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))
## 情報源

[一般教養としてのGarbage Collection](http://matsu-www.is.titech.ac.jp/~endo/gc/gc.pdf)
入門記事

[C Programming and Memory Management - Full Course](https://youtu.be/rJrd2QMVbGM?si=4puguMLETfBX7j_f)
C言語に入門しながらGCの実装もしてしまうチュートリアル

[[26. Garbage Collection]]
[[Crafting Interpreters]]のGCの章です

[The Garbage Collection Handbook](https://gchandbook.org/)
GCについて書かれたおそらく世の中で一般的に(大学の授業などで)使われている教科書
## GCの手法

[[RAII]]: GCでないメモリの管理方法。リソースの確保をオブジェクトの初期化時に行い，リソースの開放をオブジェクトの破棄と同時に行う。

## Theory of Garbage Collection

[Unified Theory of Garbage Collection](https://www.cs.cornell.edu/courses/cs6120/2020fa/blog/unified-theory-gc/)
GCを理解するのに必読の論文。参照カウント方式とトレーシング方式のGCが双対関係になっていることが理解出来る。

## Misc

[コードがリソースの寿命を知っているか、データがリソースの生死を知っているか](https://x.com/mod_poppo/status/1910258436404908103)

## 事例

[A Guide to the Go Garbage Collector](https://go.dev/doc/gc-guide)
[Garbage Collection in Ruby](https://blog.peterzhu.ca/notes-on-ruby-gc/)
[LibJS](https://github.com/SerenityOS/serenity/tree/master/Userland/Libraries/LibJS): [SerenityOS](https://github.com/SerenityOS/serenity)のJS Engineです

## Papers

[Collecting Cyclic Garbage across Foreign Function Interfaces](https://dl.acm.org/doi/10.1145/3591244)