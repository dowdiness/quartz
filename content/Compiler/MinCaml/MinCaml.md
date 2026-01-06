---
title: MinCaml
publish: false
tags: []
created: 2025-05-07T18:19:54+09:00
modified: 2025-05-08T00:12:35+09:00
---

# MinCaml

[Github](https://github.com/esumii/min-caml)
[PDF](https://www.jstage.jst.go.jp/article/jssst/25/2/25_2_2_28/_pdf)

超入門
## 参考記事

[MinCaml読解ノート](http://smpl.seesaa.net/article/5456194.html)

[DockerでMinCamlを動かす](https://blog.ojisan.io/min-caml-for-mac/)
MinCamlは32bitのx_86バイナリにコンパイルされるので新しいMacでは動かない。
dockerを使ってCentOS6の上でプログラムを動かす方法について解説されている。

## MiniML

MinCamlとは少し違うがほとんど同じようなコンパイラを作る京都大学の授業
[IoPLMaterials](https://kuis-isle3sw.github.io/IoPLMaterials/): インタプリタ
[コンパイラ](https://www.fos.kuis.kyoto-u.ac.jp/~umatani/le4/index.html)
### K-Normalization

From region inference to von Neumann machines via region representation inference[^1] が初出っぽい？
https://cs.stackexchange.com/questions/100318/what-is-the-difference-between-a-normalization-and-k-normalization-in-compilers

## 参考実装

[ml2wasm](https://github.com/akawashiro/ml2wasm/tree/dev)
[WebAssembly](https://webassembly.org/)へとコンパイルされる[MinCaml](http://esumii.github.io/min-caml/)のHaskell実装
https://a-kawashiro.hatenablog.com/entry/2018/10/31/211424

[min-caml-hs](https://github.com/minoki/min-caml-hs)
AArch64へとコンパイルされるMinCamlのHaskellでのコンパイラ

https://blog.miz-ar.info/2022/06/min-caml-in-haskell/
https://zenn.dev/mod_poppo/scraps/a01e08d7d98b9e

[MinCamlOnline](https://github.com/yuki67/MinCamlOnline)
JavaScriptへとコンパイルMinCaml
https://yuki67.github.io/post/mincaml_online/

## 脚注

[^1]: Lars Birkedal, Mads Tofte, and Magnus Vejlstrup. 1996. From region inference to von Neumann machines via region representation inference. In Proceedings of the 23rd ACM SIGPLAN-SIGACT symposium on Principles of programming languages (POPL '96). Association for Computing Machinery, New York, NY, USA, 171–183. https://doi.org/10.1145/237721.237771