---
title: IR implementation
publish: false
tags: [compiler, ir]
aliases: [IR実装]
created: 2025-12-23T11:16:34+09:00
modified: 2025-12-23T11:58:54+09:00
---

# IR implementation

IRの設計にはIR programのサイズと走査(traverse)のしやすさに気を付けなければいけない

## Greaphical IR

### IRのサイズを決める要因

- データの圧縮:
- 共有で使われるデータのみIRで持つ:node typeとして表現する
- 使われなくなった補助的なデータへの参照を消す: ポインタによって補助的なデータはallocateした後には実際のデータに書き換える。GCによってポインタが生き続けるのを防ぐ効果がある。

### Representing Tree

Tree IRの実装には大きく2つの手法がある:
- Nodeとポインタの組み合わせ
- Nodeの配列Node間のリンクは配列のインデックスかポインタによって実現する

[Free list](https://en.wikipedia.org/wiki/Free_list)

#### Mapping Arbitrary Trees to Binary Trees

#### Representing Arbitrary Graphs

## Linear IR

## 関連リンク

[[Intermediate representation]]
[[A-normal form]]
[[Deforestation|Deforestation(fusion)]]
