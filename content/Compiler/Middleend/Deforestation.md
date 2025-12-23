---
title: Deforestation(fusion)
publish: false
tags: [compiler]
aliases: [deforestation, Deforestation(fusion), fusion, 融合変換]
created: 2025-05-07T12:19:44+09:00
modified: 2025-12-23T11:22:15+09:00
---

# Deforestation(fusion)

プログラムを変換して中間データ構造を除去するコンパイラ最適化技術です。非正確評価の言語で主に使われているらしい。日本語では融合変換 (program fusion)と呼ばれている？

[Deforestation: transforming programs to eliminate trees](https://www.sciencedirect.com/science/article/pii/030439759090147A?via%3Dihub)

[A Short Cut to Deforestation](https://users.cs.northwestern.edu/~robby/courses/395-495-2017-winter/deforestation-short-cut.pdf)
これがGHCで使われているアルゴリズムらしい

## 実例

[HaskellerとRustaceanが知恵をあわせてプロダクトを3日で1000倍高速化した話](https://zenn.dev/jij_inc/articles/2025-12-03-haskeller-and-rustacean-1000x-faster#%E6%9C%AC%E4%B8%B8%EF%BC%9A%E9%81%85%E5%BB%B6%E9%85%8D%E5%88%97%E3%82%92%E3%81%A4%E3%81%8B%E3%81%A3%E3%81%A6%E9%85%8D%E5%88%97%E3%81%AE%E5%A4%89%E5%BD%A2%E3%81%AB%E4%BC%B4%E3%81%86%E3%82%A2%E3%83%AD%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E7%AF%80%E7%B4%84)

Rustでの実例

### [The Long Way to Deforestation: A Type Inference and Elaboration Technique for Removing Intermediate Data Structures](https://dl.acm.org/doi/10.1145/3674634)

[Video](https://youtu.be/UGcZKycGeGU?si=sORb3A7OPE4V4z17)

これまで提案されたDeforestationの手法には制限や実用的でなかったり(shortcut fusionにしか対応していなかったり)の問題があった。この論文ではこういった問題を解決したLumberhackという新しい手法を提案している。GHCで採用されているDeforestationはcall-by-needにしか対応していないが、Lumberhackはcall-by-need と call-by-valueに対応している。

下記はこの論文で示されているHaskellでの例:

```haskell
let rec map f xs = case xs of { [] -> [];
                                x :: xs -> f x :: map f xs }
let incr x = x + 1
let double x = x * 2
let main ls = map incr (map double ls)
```

このプログラムは `map incr` に渡される `map double ls`により中間的なリストが作られておりメモリ効率が悪い

```haskell
let rec map2 f g xs = case xs of { [] -> [];
                                   x :: xs -> f (g x) :: map2 f g xs }
let main ls = map2 incr double ls
```

map2を使った `map2 incr double ls` へと変換することにより中間的なリストが作られなくなる。
