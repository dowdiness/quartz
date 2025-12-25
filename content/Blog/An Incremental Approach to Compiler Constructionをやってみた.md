---
title: An Incremental Approach to Compiler Construction
publish: false
tags: [compiler]
aliases: [An Incremental Approach to Compiler Construction, 無題のファイル]
created: 2025-12-24T12:11:06+09:00
modified: 2025-12-25T11:56:57+09:00
---

# An Incremental Approach to Compiler Construction

## 1. Integers

`42` などのIntegerを実装する

## 2.Immediate Constants


booleanとcharacters、empty listを実装する

## 3. Unary Primitives

`add1` `sub1` などの引数を1つ受けとるプリミティブを実装する
(add1 e) をコンパイルするには先にeのコードを生成する。

## 4.Binary Primitives

引数を2つ受け取るプリミティブを実装する
これらの操作にはレジスターが2つ以上必要になり、スタックの実装する必要がある。

## 5.Local Variables