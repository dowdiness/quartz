---
title: EG-walker
aliases: [event graph walker]
created: 2025-03-15T17:56:20+09:00
modified: 2026-01-22T13:29:18+09:00
tags: [distributed-system]
publish: true
---

# EG-walker

[Moonbitによる自作のライブラリ](https://github.com/dowdiness/til/tree/main/crdt/event-graph-walker)

[Eg-walker (reference implementation)](https://github.com/josephg/eg-walker-reference)

[Paper repo](https://github.com/josephg/egwalker-paper)

[Braid](https://braid.org/)

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

リアルタイム共同編集についての歴史や既存手法の紹介
大きく2つに分類される

CRDTには**Operation-Based**と**State-Based**の2種類のやり方がある。

#### Operational Transformation (OT)

Commutativeな操作に限定することで順番が入れ替わってても最終的に同じデータへと収束させれる。(Weak Eventual consistency)

In this paper we propose Event Graph Walker (Eg-walker), a collaborative editing algorithm that overcomes this tradeoff. Like OT, Eg-walker uses integer indexes to identify insertion and deletion positions, and transforms those indexes to merge concurrent operations. When two users concurrently perform 𝑛 operations each, Eg-walker can merge them at a cost of 𝑂(𝑛 log𝑛), much faster than OT’s cost of 𝑂(𝑛 2 ) or worse. The example document that takes 1 hour to merge using OT is merged in just 24 ms using Eg-walker (Figure 8). Eg-walker merges concurrent edits using a CRDT algorithm we designed. Unlike existing algorithms, we invoke the CRDT only to perform merges of concurrent operations and we discard its state as soon as the merge is complete. We never write the CRDT state to disk and never send it over the network. While a document is being edited, we only hold the document text in memory, but no CRDT metadata. Most of the time, Eg-walker therefore uses 1–2 orders of magnitude less memory than the best CRDTs. During merging, when Eg-walker temporarily uses more memory, its peak memory use is comparable to the best known CRDT implementations. Eg-walker assumes no central server, so it can be used over a peer-to-peer network. Although all existing CRDTs and a few OT algorithms can be used peer-to-peer, most of them have poor performance compared to the centralised OT commonly used in production software. In contrast, Eg-walker’s performance matches or surpasses that of centralised algorithms. It therefore paves the way towards more collaboration software working peer-to-peer, for example in environments where co-located devices can communicate via local radio links, but not reach the Internet or any cloud services. This setting is important e.g. for devices onboard the same aircraft, in a military context, or for scientists conducting fieldwork in remote locations.

#### EG-walker

「Eg-walker」はOTとCRDTを複合したアプローチにより、これまでの「OTでの大幅に分岐したブランチをマージするコスト」と「CRDTでのメタデータによるメモリ消費の多さ」という課題を解決する新しいアルゴリズムです。

既存のアルゴリズムではオフライン編集により分岐の大きくなったブランチのマージに時間が掛かるOTか、メタデータのr use OT and accept that offline editing and long-running branches are slow, or pick a CRDT and accept a much higher memory use

sequentialな編集ではindex-basedなOTなアプローチを行い、concurrentな編集ではCRDTのアプローチを一時的な

最大の特徴は、膨大な同時編集が発生しても瞬時に処理できる圧倒的なスピードです。従来の手法では計算に1時間もかかっていたようなケースでも、このアルゴリズムならわずか0.024秒ほどで完了します。また、メモリの管理も非常にスマートで、計算に必要なデータは処理が終わった瞬間に破棄する仕組みを取り入れています。その結果、従来の優れた手法と比べても、使うメモリ量を10分の1から100分の1程度にまで抑えることに成功しました。

𝑂(𝑛 log𝑛)
�(𝑛 2 )

中央サーバーがなくてもデバイス同士でP2Pのやり取りができるため、インターネットやクラウドが使えない飛行機の中や、電波の届かない僻地での調査といった特殊な環境でも、ストレスなく共同作業を行うことが可能になります。

#### Conflict-free Replicated Data Types (CRDTs)

#### Contributions

- CRDTとOTを組み合わせたアルゴリズムであるEG-walkerの紹介。Eg-walkerは既存のCRDTよりも速くてメモリの使用量が少ない。
- Collaborative text editingには確立されたベンチマークがない。この論文は編集履歴データによるベンチマークを提案しており、今回は実際の記録からsequential な編集から concurrent な編集まで多様な編集履歴データを使っている。[^1]
- 既存のCRDTとOTの実装でも同様のベンチマークを行い、Eg-walkerの優位性を示している。
- Eg-walkerアルゴリズムの正しさ

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

#### 2.4 Replaying editing history

#### Implementing OT using a CRDT

### The Event Graph Walker Algorithm

`replica`

- Event Graph
- Document State
- Internal State

#### Characteristic

Eg-walkerは最終的なドキュメントの状態が strong list specification[^2] (replica が同じ状態に合流し正しい位置で操作が適用されること)と、maximally non-interleaving[^3] (concurrent sequences of insertionsが同じ位置で続けて行われた際にinterleavedされないこと) になることを保証します。

Eg-walker ensures that the resulting document state is consistent with Attiya et al.’s strong list specification [8] (in essence, replicas converge to the same state and apply operations in the right place), and it is maximally non-interleaving [60] (i.e., concurrent sequences of insertions at the same position are placed one after another, and not interleaved).

DAGsが。限られた状況下でのみOTのアルゴリズムが使われます。サーバーベースのOTは1本のメインブランチを持つevent graphでの操作で行われます。このメインブランチはOTにおけるサーバーとみなせます。他のブランチはこのメインブランチとの間でのみマージ可能であり、ブランチ同士が直接マージされることはありません。

#### Walking the event graph

- apply
- retreat
- advance

#### Representing prepare and effect versions

#### Mapping indexes to character IDs

#### Clearing the internal state

#### Partial event graph replay

#### 3.7 Algorithm complexity

#### Storing the event graph

EG-walker では Automerge が column-oriented database からインスピレーションを受けたstorage format と、Yjsの bit-packing トリックを使います。

まず最初にグラフのイベントにトポロジカルソートします。replica によりソートの仕方は違うかもしれません。それでもローカルにはソートされた順番によりイベントをインデックスで指定出来るようになります。次にcolumn毎に

- event type
- Inserted content
- Parents
- Event IDs
### Evaluation

[^1]: [Real world text editing traces for benchmarking CRDT and Rope data structures](https://github.com/josephg/editing-traces )
[^2]: [Specification and Complexity of Collaborative Text Editing](https://dl.acm.org/doi/10.1145/2933057.2933090) より
[^3]: [The Art of the Fugue: Minimizing Interleaving in Collaborative Text Editing](https://arxiv.org/abs/2305.00583) より