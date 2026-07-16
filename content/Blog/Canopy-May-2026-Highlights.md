---
title: Canopy in May 2026 — Highlights
publish: true
tags: [blog, canopy, english, projectional-editing]
created: 2026-07-16T09:15:00+09:00
modified: 2026-07-16T10:19:29+09:00
aliases: [Canopy in May 2026 — Highlights]
---

# Canopy in May 2026 — Highlights

A two-minute summary of a month of work. For the day-by-day version, see the [[Canopy開発日誌-5月|Japanese original]].

**Canopy** is a structure editor written in [MoonBit](https://www.moonbitlang.com/): it treats source code as structure (IR) rather than strings, with text kept as the source of truth. Concurrent editing is built on a CRDT implementing the [eg-walker paper](https://arxiv.org/abs/2409.14252). Around it sit a few sibling projects: **Loom** (incremental parsing), **incr** (incremental computation), **moondsp** (a MoonBit DSP/music DSL experiment), and **js_engine** (a JS interpreter validated against test262).

## 1. Rabbita and CodeMirror finally get along

The glue code connecting Rabbita (Canopy's UI layer) to CodeMirror had been fragile enough that the editor could just stop responding mid-session. That class of bug got fixed early in the month, which mattered more than any single feature — nothing else is worth building on top of an editor that randomly locks up.

## 2. Hidden buttons become explicit boundaries

Interactions that had been routed through invisible DOM controls moved to explicit event subscriptions and boundaries instead — a small change that made the system's behavior something you could reason about rather than something you had to reverse-engineer.

## 3. Inspector and op-log tooling

Internal editor state became something you could actually watch happen: an Inspector panel and an operation log turned "what is the editor doing right now" from a debugging guess into something visible on screen.

## 4. incr's API gets rethought

Working through [*Build Systems à la Carte*](https://hackage.haskell.org/package/build), the incr library's evaluation model and public API got a deliberate design pass — groundwork that quietly pays off two months later, in July's 0.13.0 and 0.14.0 API-boundary cleanups.

## 5. Cognition: the first AI-context scaffolding

The first pieces of "Cognition" appeared: a workspace concept, context packing, and a provider boundary that tracks what goes to an AI provider, what comes back, and how cancellation/retry are handled. The goal — letting the editor's own state become context an AI can use — doesn't pay off yet this month, but the scaffolding is where it starts.

## 6. Lambda gets real navigation

A scope graph and go-to-definition landed for Lambda, with Ideal's own scope annotations converging on the same resolution results — the first sign of the name-resolution consolidation that would fully land in July. Shared infrastructure work (an ephemeral-presence store, a byte codec) and the first performance/demo surfaces (incr's typed-spreadsheet demo, js_engine's bytecode benchmark) rounded out the month.

---

May is the quiet month where the AI-context (Cognition) and performance-measurement threads both start — neither pays off yet, but both become central later.

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/incr](https://github.com/dowdiness/incr) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- Full log: [[Canopy開発日誌-5月|May 2026 devlog (Japanese)]]
