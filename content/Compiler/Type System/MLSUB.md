---
aliases: ["**Lionel Parreaux**"]
created: 2024-06-25T01:09:16+09:00
modified: 2025-12-28T20:04:02+09:00
---

[Stephen Dolan, "Algebraic Subtyping"](https://www.bcs.org/media/2128/algebraic-subtyping.pdf)

# [**Lionel Parreaux**](https://lptk.github.io/)

香港科技大学で[TACO Lab](https://cse.hkust.edu.hk/~parreaux/)を指導しているAssistant Professor
MLsubの延長線のようなアイデイアの論文と実装を沢山出していてとても興味深い
## Paper

[The Simple Essence of Algebraic Subtyping](https://lptk.github.io/programming/2020/03/26/demystifying-mlsub.html)
Algebraic Subtypingの実装である**MLsub**をシンプルにした**Simple-sub**の論文。500行以内で実装出来て理解しやすい。

[Simpler-sub Algorithm for Type Inference with Subtyping](https://github.com/LPTK/simpler-sub)
Simple-subの機能を制限してさらに分かりやすくした実装。Recursive typesとNested let polymorphism、Precise type-variable-to-type-variable constraintsの機能を削っている。

 [Let Should not be Generalised](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tldi10-vytiniotis.pdf) 
 implicit let-generalisationは殆ど使われないわりに複雑さを招く機能なので無い方がいいんではないかという提案論文。Simpler-subが参考にしている。

[MLstruct: Principal Type Inference in a Boolean Algebra of Structural Types](https://2022.splashcon.org/details/splash-2022-oopsla/41/MLstruct-Principal-Type-Inference-in-a-Boolean-Algebra-of-Structural-Types)

[Github](https://github.com/hkust-taco/mlstruct)

[YouTube](https://www.youtube.com/watch?v=HdppREjvz5k)

[MLscript](https://github.com/hkust-taco/mlscript)

[super-Charging Object-Oriented Programming Through Precise Typing of Open Recursion](https://drops.dagstuhl.de/storage/00lipics/lipics-vol263-ecoop2023/LIPIcs.ECOOP.2023.11/LIPIcs.ECOOP.2023.11.pdf)


# Distributing intersection and union types with splits and duality (functional pearl)

https://dl.acm.org/doi/10.1145/3473594