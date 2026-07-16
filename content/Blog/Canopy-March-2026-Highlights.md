---
title: Canopy in March 2026 — Highlights
publish: true
tags: [blog, canopy, english, projectional-editing]
created: 2026-07-16T09:05:00+09:00
modified: 2026-07-16T10:22:44+09:00
aliases: [Canopy in March 2026 — Highlights]
---

# Canopy in March 2026 — Highlights

A two-minute summary of a month of work. For the day-by-day version, see the [[Canopy開発日誌-3月|Japanese original]].

**Canopy** is a structure editor written in [MoonBit](https://www.moonbitlang.com/): it treats source code as structure (IR) rather than strings, with text kept as the source of truth. Concurrent editing is built on a CRDT implementing the [eg-walker paper](https://arxiv.org/abs/2409.14252). Around it sit a few sibling projects: **Loom** (incremental parsing), **incr** (incremental computation), and **js_engine** (a JS interpreter validated against test262).

## 1. The parser becomes loom

The parser that had lived inside Canopy since December moved out to become its own repository, `dowdiness/loom`, with Canopy migrating onto the new API. This is the first of several libraries that would go on to grow independently of Canopy over the following months.

## 2. SyncEditor unifies editing

`ParsedEditor` was replaced by `SyncEditor`, folding editing, undo/redo, sync, and presence into a single editor abstraction — the foundation everything from Lambda name resolution to the later block/JSON editors would build on.

## 3. Rabbita and the Ideal editor take off

A Rabbita-based projectional editor got a real UI foundation this month: performance problems were fixed, and mobile layout, design tokens, and tree-pane navigation landed. Sixteen structural editing actions shipped too — the first broad set of "verbs" for editing a program's structure directly instead of its text.

## 4. CRDT performance gets serious

`FugueTree`/`traverse_tree` went iterative, an order-tree was introduced, and a two-count retreat optimization made event-graph-walker traversal 17.7× faster — the kind of unglamorous work that keeps structural editing fast enough to not notice.

## 5. Real-time collaboration over WebSocket

A transport layer, relay server, sync-recovery protocol, and an ephemeral-presence store (v2) landed, giving Canopy the first real infrastructure for more than one person editing the same document live.

## 6. Framework extraction begins, new editors debut

`ProjNode[T]` went generic, `TreeNode`/`Renderable` traits appeared, and the first framework/core packages were carved out of Canopy-specific code. On top of that groundwork, a block editor, a JSON editor, an AST zipper, and the first phase of a "Container" block-document abstraction all shipped.

---

March set up almost every thread the rest of the year pulls on: loom as an independent parser library, a generic projection framework, and block-based editors reaching beyond Lambda.

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- Full log: [[Canopy開発日誌-3月|March 2026 devlog (Japanese)]]
