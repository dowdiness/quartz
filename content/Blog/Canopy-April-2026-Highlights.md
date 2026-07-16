---
title: Canopy in April 2026 — Highlights
publish: true
tags: [blog, canopy, english, projectional-editing]
created: 2026-07-16T09:10:00+09:00
modified: 2026-07-16T09:10:00+09:00
---

# Canopy in April 2026 — Highlights

A two-minute summary of a month of work. For the day-by-day version, see the [[Canopy開発日誌-4月|Japanese original]].

**Canopy** is a structure editor written in [MoonBit](https://www.moonbitlang.com/): it treats source code as structure (IR) rather than strings, with text kept as the source of truth. Concurrent editing is built on a CRDT implementing the [eg-walker paper](https://arxiv.org/abs/2409.14252). Around it sit a few sibling projects: **Loom** (incremental parsing), **incr** (incremental computation), and **js_engine** (a JS interpreter validated against test262).

## 1. One protocol for every editor surface

Ideal editor and the ProseMirror example both moved onto a single `EditorProtocol`, with CM6Adapter/PMAdapter formalizing the adapter layer. Instead of each editor surface having its own bespoke wiring, they now all speak the same language to the underlying document model.

## 2. The pretty-printer meets the DOM

The Wadler-Lindig pretty-printer's output now bridges to `ViewNode`, wired up to real HTML syntax highlighting — turning what had been a text-formatting tool into part of the rendering pipeline.

## 3. Markdown gets a real block editor

Markdown became a first-class editing target: seven Markdown edit operations, a three-mode web editor, a block-input textarea overlay, and a semantic-HTML preview adapter all shipped, alongside Phase 2 and 3 of a "Container" abstraction (text sync, block-doc sync, document-level undo grouping) that treats documents as block-addressable.

## 4. A generic B-tree and a confidence layer

`lib/btree` was carved out as a standalone generic B-tree library and merged with March's order-tree work, gaining range-delete and splice-promotion chain repair. Separately, `lib/semantic` introduced a confidence lattice and symbolic annotator — the first infrastructure for reasoning about how sure the system is about an inferred result rather than treating everything as certain.

## 5. Language decoupling begins

`LanguageCapabilities[T]` split Lambda-specific types out of `SyncEditor`, opening the door to generic tree operations that don't assume Lambda is the only language in the room — groundwork that pays off later when JSON, Markdown, and eventually JSX need the same editor machinery.

## 6. Drag-and-drop, E2E tests, and workspace tooling

Drag-and-drop foundations landed in both Ideal and the block editor (semantic Before/After drops, grip-only drag, outline DnD), Web E2E tests for Lambda/JSON editors joined CI, loom retired `ReactiveParser` in favor of a unified `@loom.Parser[T]`, and `moon.work` workspace tooling arrived with CI-enforced dependency-direction rules.

---

By the end of April, Canopy had the shape it would keep for months: a language-agnostic core, block-addressable documents, and a growing set of adapters rather than one hardcoded editor.

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- Full log: [[Canopy開発日誌-4月|April 2026 devlog (Japanese)]]
