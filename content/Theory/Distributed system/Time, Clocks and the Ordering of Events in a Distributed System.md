---
title: Time, Clocks and the Ordering of Events in a Distributed System
publish: false
tags: [distributed-system]
aliases: [Clocks and the Ordering of Events in a Distributed System, Time]
created: 2025-06-30T18:26:10+09:00
modified: 2026-01-13T20:56:36+09:00
---

# Time, Clocks and the Ordering of Events in a Distributed System

## Introduction

Distributed Systemで起こる時系列的なイベントの関係を[Partial ordering](https://en.wikipedia.org/wiki/Partially_ordered_set)な [Happened-before](https://en.wikipedia.org/wiki/Happened-before) 順序集合関係として定義し、これをConsistent [total ordering](https://en.wikipedia.org/wiki/Total_order)な順序集合へと拡張するアルゴリズムについて論じた論文です。

## The Partial Ordering

Happened-before Definition. 

> The relation "→" on the set of events of a system is the smallest relation satisfying the following three conditions:
> - (1) If a and b are events in the same process, and a comes before b, then a ~ b.
> - (2) If a is the sending of a message by one process and b is the receipt of the same message by another process, then a ~ b.
> - (3) If a ~ b and b ~ c then a ---* c. Two distinct events a and b are said to be concurrent if a ~ b and b -/-* a.

## Logical Clocks

Clock C は イベントを受け取り値を割り当てる関数

*Clock Condition.*
`For any events a, b: if a → b then C(a) < C(b).`

このClock Conditionが成り立つには下記の２つの条件を満たさなければならない

- C 1. If a and b are events in process $P_{i}$, and a comes before b, then Ci(a) < Ci(b).
- C 2. If a is the sending of a message by process $P_{i}$ and $b$ is the receipt of that message by process Pi, then Ci(a) < Ci(b).
## Ordering the Events Totally

## Anomalous Behavior

## Physical Clocks

## Conclusion