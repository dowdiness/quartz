---
title: Call-by-push-value
created: 2024-10-17T03:51:05+09:00
modified: 2026-01-22T14:07:06+09:00
aliases: [Call-by-push-value, CBPV]
tags: [compiler]
publish: true
---

# Call-by-push-value

Call-by-Push-Valueは値呼び戦略と名前呼び戦略を両立させる評価戦略。

従来の理論での項をCBPVでは値と計算に区別する。call-by-value と call-by-name のsemantic primitivesを使うので、それらで表現出来る項はCBPVでも表現出来るし逆にも出来る。意味的な表現力は同じでもCBPVは従来では出来なかった抽象化が可能になる。

call-by-value や call-by-name の場合

```hs
data Type 
    = Int
    | Fun Type Type

data Term
    = Var Text
    | Int Int
    | Fun Text Term
    | App Term Term
```

Call-by-Push-Value の場合

```hs
-- Type of values
data ValType 
    = Int
    | Thunk CompType

-- Type of computations
data CompType 
    = Fun ValType CompType -- !!
    | Return ValType

-- A value term
data Value
    = Int Int
    | Var Text
    | Thunk Comp

-- A computation term
data Comp
    = Fun Text Comp
    | App Comp Value
    | Return Value
```

[Call-by-push-value](https://dl.acm.org/doi/10.1145/3537668.3537670) by Paul Blain Levy

[長いバージョン](https://pblevy.github.io/papers/thesisqmwphd.pdf)

## ブログ

[I'm betting on Call-by-Push-Value](https://thunderseethe.dev/posts/bet-on-cbpv/)

## 実装

[Zydeco](https://github.com/zydeco-lang/zydeco): a proof-of-concept programming language based on call-by-push-value

Zydecoの理論: [Notions of Stack-Manipulating Computation and Relative Monads](https://dl.acm.org/doi/10.1145/3720434)
