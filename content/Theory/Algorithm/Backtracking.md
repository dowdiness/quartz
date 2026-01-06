---
title: Backtracking
publish: false
tags: [algorithm, backtracking, search]
aliases: []
created: 2025-07-09T17:46:53+09:00
modified: 2026-01-01T16:07:04+09:00
---

# Backtracking

[[Catastrophic Backtracking]]

[pdf](https://jeffe.cs.illinois.edu/teaching/algorithms/book/02-backtracking.pdf)

> Whenever the algorithm needs to decide between multiple alternatives to the next component of the solution, it recursively evaluates every alternative and then chooses the best one.\

## N Queens

The [**n-queens** puzzle](https://leetcode.com/problems/n-queens/description/) is the problem, proposed by German Chess enthusiast [Max Bezzel](https://en.wikipedia.org/wiki/Max_Bezzel) in 1848 for the standard 8 ⇥ 8 board, of placing `n` queens on an `n x n` chessboard such that no two queens attack each other.
