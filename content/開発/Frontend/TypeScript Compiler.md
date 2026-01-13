---
title: TypeScript Compiler
publish: false
tags: [compiler, frontend, typescript]
created: 2025-05-23T15:04:55+09:00
modified: 2026-01-13T18:28:47+09:00
---

# TypeScript Compiler

https://2025.tskaigi.org/talks/kazushikonosu
https://github.com/microsoft/TypeScript/wiki/Using-the-Compiler-API
https://github.com/line/tsr/tree/main
https://github.com/mizchi/mints/tree/main
https://github.com/microsoft/TypeScript-Compiler-Notes

Node


SourceFile
単一ファイル毎のコード

Program
複数ファイル　プロジェクト全体

TypeChecker

LanguageService
Programに時間軸とより高度なAPIを追加


Visitor

Transformer Printer

TextChange
