---
title: Integers and variables
publish: false
tags: [compiler]
created: 2025-12-09T04:58:11+09:00
modified: 2025-12-23T13:35:00+09:00
aliases: [Assembly]
---

# Integers and variables
## Assembly

`global` directiveにより `main` procedureを外部へと公開してOSにより呼び出せるようにしている。
*program counter* `rip` 次に実行される命令を保持するレジスタ
㍶は命令が実行される度に増加するので次の命令のメモリアドレスを指している

x86の命令はふたつのoperand、integer constant (*immediate value*)か*register*、memory locationを受け取る。

*register* `%rax`
*immediate value* `$n`
*memory location* `n(%r)`

```x86
	.global main
main:
	movq $10, %rax
	addq $32, %rax
	retq
```

(+ 10 32)

`procedure call stack`

`memory address` を含むものを `pointer` と呼ぶ。
