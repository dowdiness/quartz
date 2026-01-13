---
title: LLVM
publish: false
tags: [compiler]
aliases: ["LLVM IR"]
created: 2025-12-22T19:51:40+09:00
modified: 2026-01-13T18:05:03+09:00
---

# LLVM

[公式サイト](https://llvm.org/)
[Reference](https://llvm.org/docs/LangRef.html)

[The Architecture of Open Source Applications (Volume 1)LLVM](https://aosabook.org/en/v1/llvm.html)
LLVM の解説記事


## Reference

リファレンスの中でも特に重要そうなページ
- [The Often Misunderstood GEP Instruction](https://llvm.org/docs/GetElementPtr.html): LLVM IRでの [GetElementPtr](https://llvm.org/docs/LangRef.html#getelementptr-instruction) (GEP) instruction は特に難しいと言われているらしい
- [LLVM Loop Terminology (and Canonical Forms)](https://llvm.org/docs/LoopTerminology.html): LLVMでのループの実現方法について
- [LLVM Atomic Instructions and Concurrency Guide](https://llvm.org/docs/Atomics.html): LLVMでのAtomicについて
## Reading

[How LLVM Optimizes a Function](https://blog.regehr.org/archives/1603)


## リンク

[[quartz/content/Blog/Moonbitで作るラムダ計算のコンパイラ|Moonbitで作るラムダ計算のコンパイラ]]