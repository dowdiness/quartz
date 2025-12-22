---
title: Register allocation
publish: false
tags: [compiler]
aliases: [無題のファイル]
created: 2025-12-11T19:52:24+09:00
modified: 2025-12-22T18:45:53+09:00
---

# Register allocation

## Calling convention

Register上での関数呼び出しの実行のされ方を定める規則

これはx86でのルールです

### *caller-saved registers*

関数の呼び出し先で利用されて中身の変わる可能性のあるレジスタ
関数の呼び出し元は関数呼び出しに先駆けてレジスタの中身を開放する義務がある

`rax rcx rdx rsi rdi r8 r9 r10 r12`

### *callee-saved registers*

関数の呼び出し前後でレジスタの中身が変わる可能性のないレジスタ
関数の呼び出し元で使う場合にはスタックに元の値を保存しておいて関数終了前にレジスタの中身を後から元に戻す必要がある

`rsp rbp rbx r12 r13 r14 r15`

関数の引数と戻り値を使われるレジスタ
引数が7つ以上ある場合には関数の呼び出し先のスタックフレームが使われる
７章では末尾再帰の効率化をするためにタプルを使って渡す方法を紹介している

`rdi rsi rdx rcx r8 r9

*call-live variable*


## Liveness Analysis

## Interference Graph

## Graph Coloring

## Patch Instruction


## Mov Biasing