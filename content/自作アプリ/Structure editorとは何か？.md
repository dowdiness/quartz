---
title: Structure editorとは何か？
publish: false
tags: [" "]
aliases: [無題のファイル]
created: 2026-01-03T14:08:04+09:00
modified: 2026-01-03T14:24:48+09:00
---

# Structure editorとは何か？

プログラムの表現は伝統的にテキストが用いられてきた。
テキスト表現はシンタックスを通して表現されたＵＩでしかないが、コードの編集とは本質的にはプログラムの木構造を編集する行為である。
CSTではASTを直接いじるような編集行為。

プログラムが持ちうる本質的なモデルは1つしかないが、それを表現する形は複数あるようなDSLを想像してみましょう。

![](https://www.martinfowler.com/articles/workbench.gif)


ここで大事になってくるのが双模倣性（そうもほうせい、Bisimulation）という概念です。双模倣性について理解するためには、まずはシステムの状態遷移系のシミュレーションについて知る必要があります。何故ならば双模倣性とはシミュレーションの特殊な形だからです。システムの状態遷移系が



そのために必要なこと。

プログラムの編集過程で起こる様々な問題点たち。

Problem 1: Syntactically Malformed Edit States

Problem 2: Statically Meaningless Edit States

Problem 3: Dynamically Meaningless Edit States

Problem 4: A Calculus of Edit Actions

Problem 5: Meaningful Suggestion Generation and Ranking
