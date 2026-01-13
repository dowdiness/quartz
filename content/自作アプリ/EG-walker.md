---
title: EG-walker
aliases: [event graph walker]
created: 2025-03-15T17:56:20+09:00
modified: 2026-01-13T17:57:48+09:00
tags: [distributed-system]
publish: true
---

# EG-walker

[Moonbitによる自作のライブラリ](https://github.com/dowdiness/til/tree/main/crdt/event-graph-walker)

[Eg-walker (reference implementation)](https://github.com/josephg/eg-walker-reference)

[Paper repo](https://github.com/josephg/egwalker-paper)

## Papers

- [Collaborative Text Editing with Eg-walker: Better, Faster, Smaller](https://arxiv.org/abs/2409.14252)
- [The Art of the Fugue: Minimizing Interleaving in Collaborative Text Editing](https://arxiv.org/abs/2305.00583)
- [Undo and Redo Support for Replicated Registers](https://arxiv.org/abs/2404.11308)
- [Collabs: A Flexible and Performant CRDT Collaboration Framework](https://arxiv.org/abs/2212.02618)

## Videos

[Text CRDTs from scratch, in code!](https://www.youtube.com/watch?v=_lQ2Q4Kzi1I)
[Collaborative Text Editing with Eg-Walker](https://www.youtube.com/watch?v=rjbEG7COj7o)

## Operational transformation

リアルタイム編集を実現するために、[Google Wave](https://ja.wikipedia.org/wiki/Google_Wave) で使われていたシステム。

Jupiter OT system


## **conflict-free replicated data type** (**CRDT**)

https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type


https://loro.dev/

## CRDT Papers

https://crdt.tech/papers_bib.html


配列ではなくB-treeを使うと高速化出来る

[[Merkle Search Trees]]


## 論文メモ書き

### Introduction

### Background

テキストへの挿入と削除をどの位置に対しても出来る編集システムを考える。この編集を `operation` と呼ぶ。
`Insert(i, c)` は文字`c`をindex`i`に挿入する`operation`。 `Delete(i)`はindex`i` の文字を削除する`operation`である。(indexはゼロから始まる)連続したoperationを圧縮して扱うことも出来るが、簡単にするため今回は全て1文字の`operation`として考える。

#### System Model

それぞれのユーザーがドキュメントを編集に使うデバイスのことを `replica` と呼ぶ。それぞれの `replica` はドキュメントの編集履歴を全て持つ。`replica` は non-Byzantineである。

Our algorithm ensures convergence: any two replicas that have seen the same operations have the same document state (i.e., a text consisting of the same sequence of characters), even if the operations arrived in a different order at each replica. If the underlying broadcast protocol ensures that every non-crashed replica eventually receives every operation, the algorithm achieves strong eventual consistency.

このアルゴリズムは `convergence` を保証する。`convergence` とは、同じ`operation` を持つ `replica` は例え `operation` の順番が別々であっても同じドキュメントステート(文字列の順序が同じテキストを持つこと)を持つことである。

ブロードキャストプロトコルが、すべての壊れていない `replica` からのすべての `operation` を受け取ることを保証する場合、 `strong eventual consistency` を実現出来る。

#### Event graphs

##### `Event graph`

ドキュメントの編集履歴を `Event graph` として表す。`Event graph` は全てのNodeが `operation` のイベント、uuid、親のイベントのIDの集合、として構成された `directed acyclic graph (DAG)` である。Nodeが親子の場合、Graphは親から子へのEdgeを持つ。Graphは [Transitive reduced](https://en.wikipedia.org/wiki/Transitive_reduction) に作られる。aからbへの有向パスが存在する場合、*a は b より以前の出来事* だとする。これを[ランポート](https://ja.wikipedia.org/wiki/%E3%83%AC%E3%82%B9%E3%83%AA%E3%83%BC%E3%83%BB%E3%83%A9%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%88)に従い `a → b` と書く。 この二項関係は `strict partial order` である。同じグラフ内の a と b が `𝑎 ≠ 𝑏` であり、どちらも以前の出来事でない: `𝑎 ↛ 𝑏 ∧ 𝑏 ↛ 𝑎`  の場合、イベント a と b を `concurrent` と呼び、これを `𝑎 ∥ 𝑏` と書く。

##### `frontier`

`frontier` は子のいないイベントの集合である。常にユーザーが `operation` を実行した場合に、この `operation` を含んだ新しいイベントがグラフに追加される。`replica` の以前のローカルコピーグラフ内の `frontier`  は新しいイベントの親になる。新しいイベントはネットワークを通してブロードキャストされて、それぞれの `replica` はローカルコピーグラフに新しいイベントを追加する。もし親が見つからない場合(イベントの親IDが既存のグラフ内のイベントIDに見つからない場合)、 `replica` は親IDのイベントが来るまでグラフにそのイベントを追加するのを待つ。これによりシンプルな `causal broadcast protocol` を実現出来る。2つの `replice` のグラフはイベントの集合の union(和集合) を取ることにより merge 出来る。グラフ内のイベントはimmutable である。イベントは常に最初に生成された時と同じであり、いかなるトランスフォーメーションの結果でもない。グラフはmonotonically([単調](https://ja.wikipedia.org/wiki/%E5%8D%98%E8%AA%BF%E5%86%99%E5%83%8F))に増加する（イベントを取り除くことはない）。新しいイベントは常に既存のイベントの子である（既存のイベントに親を加えることはない）。

グラフのイベント数は幾万にも及ぶ可能性がある。しかしこれは連続する挿入や削除イベントとしてコンパクトに保存することが出来る。

#### Document versions


### Algorithm

