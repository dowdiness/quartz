---
title: α-conversion
aliases: [アルファ変換]
created: 2025-04-24T01:01:12+09:00
modified: 2026-01-07T23:15:47+09:00
tags: [compiler]
---

# α-conversion

https://en.wikipedia.org/wiki/Lambda_calculus#%CE%B1-conversion
https://people.cs.nott.ac.uk/psztxa/publ/alpha-draft.pdf

https://en.wikipedia.org/wiki/De_Bruijn_index

https://en.wikipedia.org/wiki/De_Bruijn_index#Alternatives_to_de_Bruijn_indices

## [Alpha-conversion is inevitable](https://okmij.org/ftp/Computation/lambda-calc.html#alpha-conv)

最初に変数名を固有なものに変換(alpha-conversion)しておけば、その後のフェーズでのalpha-conversionは不要になるのか解説した記事。結論としては、もちろん不要になる場合もあるが基本的にはalpha-conversionは避けられないらしい。

`modulo alpha-conversion, the renaming of bound identifiers` が存在する場合はalpha-conversionが必要になりそう。

[Hashing Modulo Alpha-Equivalence](https://arxiv.org/pdf/2105.02856)を使えば大丈夫？この問題は[[Common subexpression elimination]]にも関わっているらしい。