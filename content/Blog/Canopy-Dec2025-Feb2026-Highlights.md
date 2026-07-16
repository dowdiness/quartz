---
title: Canopy in December 2025 – February 2026 — Highlights
publish: true
tags: [blog, canopy, english, projectional-editing]
created: 2026-07-16T09:00:00+09:00
modified: 2026-07-16T10:24:16+09:00
aliases: [Canopy in December 2025 – February 2026 — Highlights]
---

# Canopy in December 2025 – February 2026 — Highlights

A two-minute summary of Canopy's first three months. For the day-by-day version, see the [[Canopy開発日誌-2025年12月〜2026年2月|Japanese original]].

**Canopy** is a structure editor written in [MoonBit](https://www.moonbitlang.com/): it treats source code as structure (IR) rather than strings, with text kept as the source of truth. Concurrent editing is built on a CRDT implementing the [eg-walker paper](https://arxiv.org/abs/2409.14252). Around it sit a few sibling projects: **Loom** (incremental parsing), **incr** (incremental computation), and **js_engine** (a JS interpreter validated against test262) — all of which trace their roots back to this period.

## 1. A CRDT experiment, not an editor with sync bolted on

Canopy started life on December 26th as a three-day CRDT experiment: a FugueMax merge algorithm, a baseline editor, and a public API, built before there was any "editor" to speak of. The ordering was deliberate — the CRDT came first, and everything else, including projectional editing itself, was built on top of it rather than added to an existing text editor later.

## 2. Structural parsing from day three

By December 28th, incremental reparsing (Lezer-style) and structural validation were already in place. Combined with the CRDT core, this gave the earliest prototype a property it never lost: the tree is always kept in sync with the text as you type, not reconstructed after the fact.

## 3. Branches, version vectors, and the event graph

January's work built out the collaboration machinery: a branch system, version vectors, and an early event-graph-walker (the ancestor of what's now the independent `event-graph-walker` library underpinning Loom). A formal CRDT specification and an RLE (run-length encoding) implementation replaced what had been fast-and-loose merge code.

## 4. Projectional editing gets its first real implementation

The core idea of the whole project — editing a program's meaning rather than its characters — got Phases 1 through 3 this month, alongside a React-based collaborative-editing demo proving the concept could actually run in a browser with more than one cursor in it.

## 5. February: the monorepo split

By the end of February, the codebase had outgrown a single package. Git submodules split it apart, LCS-based AST children reconciliation and UndoManager integration landed, and a quality-verification CI workflow was put in place — the shape that would let loom, incr, and the others grow independently over the following months.

---

The pace: three months from a bare CRDT prototype to a working monorepo, almost entirely solo commits with no PR review process yet — that would come later. js_engine, meanwhile, was already quietly racking up roughly 480 commits in parallel, developing mostly on its own track.

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- Full log: [[Canopy開発日誌-2025年12月〜2026年2月|Dec 2025 – Feb 2026 devlog (Japanese)]]
