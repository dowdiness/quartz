---
title: Canopy in May 2026 — Highlights
publish: true
tags: [blog, canopy, english, projectional-editing]
created: 2026-07-16T09:15:00+09:00
modified: 2026-07-16T18:20:00+09:00
aliases: [Canopy in May 2026 — Highlights]
---

# Canopy in May 2026 — Highlights

A two-minute summary of a month of work. For the day-by-day version, see the [[Canopy開発日誌-5月|Japanese original]].

**Canopy** is a structure editor written in [MoonBit](https://www.moonbitlang.com/): it treats source code as structure (IR) rather than strings, with text kept as the source of truth. Concurrent editing is built on a CRDT implementing the [eg-walker paper](https://arxiv.org/abs/2409.14252). Around it sit a few sibling projects: **Loom** (incremental parsing), **incr** (incremental computation), **moondsp** (a MoonBit DSP/music DSL experiment), and **js_engine** (a JS interpreter validated against test262).

## 1. Unicode correctness and edit history

Canopy's activity started on 5/7. SyncEditor gained causal snapshots, and Ideal got the groundwork for a Graphviz history view of edit causality. In parallel, a Unicode correctness audit (issue #216) began: `moji` (UAX #29), Markdown ZWSP handling, and position-unit cleanup on the bridge. Edit-response benchmarks, Canvas, and the Intent panel also landed in this stretch — not flashy features, but the foundation name resolution and external analysis would need later.

## 2. Rabbita and CodeMirror finally get along

The glue code connecting Rabbita (Canopy's UI layer) to CodeMirror had been fragile enough that the editor could just stop responding mid-session. Stabilizing that connection came before new feature work — nothing else is worth building on top of an editor that randomly locks up. Once it held, Inspector work and Cognition experiments could start in earnest.

## 3. From guessing to tracing internal state

Interactions that had been routed through invisible DOM controls moved to explicit event subscriptions and boundaries instead. Internal state that used to require guesswork during debugging became visible through an Inspector panel and operation log — a shift from "touch it and it moves" to "explain why it moved."

## 4. incr's API gets rethought

Working through [*Build Systems à la Carte*](https://hackage.haskell.org/package/build), the incr library's evaluation model and public API got a deliberate design pass — groundwork that quietly pays off two months later, in July's 0.13.0 and 0.14.0 API-boundary cleanups.

## 5. Cognition: the first AI-context scaffolding

The first pieces of "Cognition" appeared: a workspace concept, context packing, and a provider boundary that tracks what goes to an AI provider, what comes back, and how cancellation/retry are handled. The goal — letting the editor's own state become context an AI can use — doesn't pay off yet this month, but the scaffolding is where it starts, and it leads toward July's GenUI experiment.

## 6. Lambda gets real navigation

A scope graph and go-to-definition landed for Lambda, with Ideal's own scope annotations converging on the same resolution results — the first sign of the name-resolution consolidation that would fully land in July. Ephemeral-presence cleanup and js_engine's bytecode benchmark ran in parallel. For the first time, the structure editor had language-server-style "jump" operations.

---

The first half of May was Unicode, moji, and the Intent panel; the second half was Rabbita wiring, Cognition, and the scope graph. June's NodeId preservation and SDEG expansion both rest on this month's groundwork.

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/incr](https://github.com/dowdiness/incr) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- Full log: [[Canopy開発日誌-5月|May 2026 devlog (Japanese)]]
