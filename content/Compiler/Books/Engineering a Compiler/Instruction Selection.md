---
title: Instruction Selection
publish: true
tags: [compiler]
created: 2025-12-22T17:18:00+09:00
modified: 2025-12-23T10:46:27+09:00
---

# Instruction Selection

[[Engineering a Compiler|目次]]

## 1.Introduction

Instruction SelectionがすることはIRとISAの間のマッピングを作ること。パターンマッチング。
実装方法は主にPeephole optimizationとTree pattern matching algorithmsの二つがある。

この段階ではまだ無限のレジスターの存在を前提としたアセンブリを生成する。

Selection dictates both the time required for an operation and the functional units on which it can execute.

Schedulerやregister allocatorはターゲットのCPUやISAの知識が不可欠だが、これらの知識をパラメトリック多相として分けて実装することにより汎用的なバックエンドを作れる。コードジェネレータージェネレーターと呼べるかも。

コンパイル先を変える能力のあるInstruction Selectorとは、IRとISAの知識と紐づけることにより、これらの間の写像を作るパターンマッチングエンジンである。

Syntaxを与えると自動でスキャナやパーサーを作れるスキャナジェネレータやパーサージェネレータと同じように、マシンの情報を与えることによりバックエンドを自動で生成するInstruction Selectionも作れる。

実際には不可能だがそれでも出来るだけInstruction SelectorとScheduler、register allocatorにマシン依存のコードは限定させるべきである。それにより複雑性を減らせれる。

## 2.Background

[Addressing mode](https://en.wikipedia.org/wiki/Addressing_mode)

一般的にISAはIR命令を実装する手段を複数用意している

RISCはAddress Modeの数が少なくIR命令の実装手段が限られている。そのためInstruction Selectionの複雑性が減るがRegister Allocationの重要になる。
CISCはISAの命令に沢山の機能を詰め込んでおりIR命令の実装手段が多い。そのためInstruction Selectionの方がRegister Allocationよりも重要になる。