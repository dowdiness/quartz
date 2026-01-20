---
title: Moonbitでラムダ計算のコンパイラを作ってみた
publish: false
tags: [compiler, moonbit]
aliases: [はじめに, 無題のファイル]
created: 2025-12-08T23:49:49+09:00
modified: 2026-01-20T14:20:15+09:00
---

# はじめに

手順

クロージャー変換

## まずは目標を決める

コンパイラがしていることは意味的に同じ違う言語への変換です。アセンブリなどの低級な言語へと変換します。

## Compiler Explorer でコンパイラの出力を見てみる

## 簡単な足し算をLLVM IRへと変換するコンパイラ

## ラムダ計算とIRとの距離感

3つしか項がないシンプルな言語ですが、アセンブリへと

高階関数

## クロージャ変換

自由変数を取り除く

## ホイスティング


## 参考文献

[Compiling Lambda Calculus](https://compiler.club/compiling-lambda-calculus/)

[Compiler Explorer](https://godbolt.org/)