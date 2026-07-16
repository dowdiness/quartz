---
title: Canopy in July 2026 — Highlights
publish: true
tags: [blog, canopy, english, projectional-editing]
created: 2026-07-16T09:20:00+09:00
modified: 2026-07-16T10:19:27+09:00
aliases: [Canopy in July 2026 — Highlights]
---

# Canopy in July 2026 — Highlights

A two-minute summary of a month of work. For the day-by-day version with PR links, see the [[Canopy開発日誌-7月|Japanese original]].

**Canopy** is a structure editor written in [MoonBit](https://www.moonbitlang.com/): it treats source code as structure (IR) rather than strings, with text kept as the source of truth. Concurrent editing is built on a CRDT implementing the [eg-walker paper](https://arxiv.org/abs/2409.14252). Around it sit a few sibling projects: **Loom** (incremental parsing), **incr** (incremental computation), and **js_engine** (a JS interpreter validated against test262).

## 1. incr ships two breaking releases back to back

0.13.0 stripped incr's entire legacy compatibility API surface in one shot rather than deprecating it gradually; 0.14.0 followed a few days later, deleting unused ghost handle types, closing an interior-mutation hole in `InternTable`, and moving disposed-input errors onto a proper `ReadError` channel. Both releases rippled through loom and Canopy as pin updates within a day or two.

## 2. A real bug becomes a written contract

Debugging a reentrancy bug in MoonDsp (a stray `Input::set` call from inside a reactive compute) turned into an ADR defining incr's full evaluation-strategy contract: which cell kinds may be cached transparently, why the boundary is enforced at runtime rather than in the type system, and why pull, push, and Datalog fixpoint share one `Runtime` instead of being split into three libraries. Guard code, a peer-review correction pass, and a deliberate decision *not* to loosen the guard followed over the next two days — a clean example of writing the rule down after finding the bug, not before.

## 3. loomgen retires its own code generator

loom's code-generation tool, loomgen, spent the first half of the month adding new EBNF operators (`~`, `!`, `@until`, `{Sep}`, `@error_node`, and more) to its generated-parser pipeline. Then a benchmark showed the tree-walking interpreter had caught up on incremental reparses — so loom deleted the hand-written code generator (`emit_grammar.mbt`) outright and went all-in on interpretation. The second half of the month pivoted loomgen toward generating Markdown's line-oriented lexing instead.

## 4. JSX becomes Canopy's fourth language

A read-only CST-to-`ProjNode` projection for JSX landed, built the same way Lambda, JSON, and Markdown were — reusing the same core machinery rather than hand-rolling a new integration.

## 5. Generative UI appears — with a kill date

Within days of JSX landing, Canopy was streaming LLM-shaped output into structurally-validated JSX and reconciling it into the DOM: a stateful session model, duplicate-sibling-identity fixes, a deterministic async driver, and a browser failure-recovery slice all shipped. The most interesting document of the month is the one that refuses to call this finished — a tightly scoped experiment design for actually wiring in a live provider, explicit that a syntactically valid result isn't evidence anyone can use it, with a hard keep-or-delete decision date of **2026-07-29**.

## 6. js_engine ships v0.6.0

`for await`/async iteration reached 100% test262 conformance, RegExp lookbehind assertions landed, every stdlib builtin (Map, Set, Array, Promise, boxed primitives, and more) was migrated onto one atomic install contract, an embedder-facing host-object API arrived, and class private fields/methods/static blocks shipped — all capped off with a v0.6.0 release.

---

The throughline: incr's API cleanup rippled outward, loomgen proved itself unnecessary in its original form and got leaner, and the space that opened up got filled almost immediately by Canopy's newest and most speculative experiment yet.

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [dowdiness/incr](https://github.com/dowdiness/incr) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- Full log: [[Canopy開発日誌-7月|July 2026 devlog (Japanese)]]
