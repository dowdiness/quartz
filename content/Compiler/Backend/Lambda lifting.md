---
title: Lambda lifting
publish: false
tags: [closure, compiler]
aliases: [ラムダリフティング]
created: 2025-05-08T00:59:54+09:00
modified: 2025-05-08T01:24:48+09:00
---

# Lambda lifting

[Wiki](https://en.wikipedia.org/wiki/Lambda_lifting)

ローカル関数をグローバル関数へと変換する手法

> たまにラムダリフティングを関数のトップレベルへの持ち上げに対する用語として扱っている例が見受けられるが、これは誤用だと個人的には思う。関数の持ち上げはしばしば Hoisting（これに対する通訳は知らない）と呼ばれる。ラムダリフティングの「リフティング」はあくまで自由変数の「持ち上げ」であって、関数の「持ち上げ」ではないと考えるべきだろう。

似た手法に[[Closure conversion]]がある

# Selective Lambda Lifting

Graf, Sebastian and Simon L. Peyton Jones. “Selective Lambda Lifting.” _ArXiv_ abs/1910.11717 (2019): https://arxiv.org/abs/1910.11717v2

[[Closure conversion]]が優先的に使われているため現在ではLambda liftingはあまり使われていないが、この論文ではコード生成の最適化のためがLambda Lifting有効なことを示している。