---
created: 2025-03-07T10:40:26+09:00
modified: 2025-12-24T12:07:07+09:00
---

[Paper](https://www.jstage.jst.go.jp/article/jssst/31/1/31_1_30/_pdf)

[応用](https://yui-knk.hatenablog.com/entry/2023/12/06/082203)

1. [BNF](https://d.hatena.ne.jp/keyword/BNF)のルールに対応する[オートマトン](https://d.hatena.ne.jp/keyword/%A5%AA%A1%BC%A5%C8%A5%DE%A5%C8%A5%F3)を用意する
2. 1で用意した[オートマトン](https://d.hatena.ne.jp/keyword/%A5%AA%A1%BC%A5%C8%A5%DE%A5%C8%A5%F3)を合成して1つの[オートマトン](https://d.hatena.ne.jp/keyword/%A5%AA%A1%BC%A5%C8%A5%DE%A5%C8%A5%F3)にまとめる
3. 2でつくった[オートマトン](https://d.hatena.ne.jp/keyword/%A5%AA%A1%BC%A5%C8%A5%DE%A5%C8%A5%F3)の遷移を表にまとめると[構文解析](https://d.hatena.ne.jp/keyword/%B9%BD%CA%B8%B2%F2%C0%CF)表になる


$CG = {αβ|S ∗ =⇒rm αAw =⇒rm αβw}$

を考える．集合 CG に対して以下の性質が成り立つ．

定理 1 (LR 構文解析の基本原理 (Knuth)) CG は 正規言語である． 

この定理は，任意の文脈自由文法 G の定義から， CG を受理する (決定性) オートマトン DG を構成で きることを示している

