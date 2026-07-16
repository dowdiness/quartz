---
title: Canopy in June 2026 — Highlights
publish: true
tags: [blog, canopy, english, projectional-editing]
created: 2026-07-04T23:30:00+09:00
modified: 2026-07-16T10:19:26+09:00
aliases: [Canopy in June 2026 — Highlights]
---

# Canopy in June 2026 — Highlights

A two-minute summary of a month of work. For the day-by-day version with PR links, see the [[Canopy-Devlog-June-2026|full June devlog]] (or the [[Canopy開発日誌-6月|Japanese original]]).

**Canopy** is a structure editor written in [MoonBit](https://www.moonbitlang.com/): it treats source code as structure (IR) rather than strings, with text kept as the source of truth. Concurrent editing is built on a CRDT implementing the [eg-walker paper](https://arxiv.org/abs/2409.14252). Around it sit a few sibling projects: **Loom** (incremental parsing), **incr** (incremental computation), and **js_engine** (a JS interpreter validated against test262).

## 1. Structural edits can no longer silently change meaning

Lambda editing reached a milestone: rename, move, duplicate, and extract-to-let now work on bindings *inside* blocks, not just at the top level — and every edit is validated by re-parsing the resulting text and checking it produces the intended AST. Getting there required scope-graph-based shadowing checks: moving a binding past a lambda parameter can silently redirect references, and ordering checks alone don't catch it. An alpha-safe beta reduction pilot (computing with binder identity instead of names) landed as well.

## 2. Edits keep their identity

Wrapping `Var(x)` in a lambda used to destroy the node's identity — and with it, your selection and fold state. A new hint channel (`IdentityTransform` + hinted reconcile) lets edit operations declare their intent ("this is a wrap," "this unwrap keeps this child") so NodeIds survive structural edits. Operations that can't produce a safe hint conservatively opt out. This is one of the core problems any projectional editor has to solve.

## 3. Markdown became the second structurally-editable language

Headings and list items are now tracked with stable identities that survive broken parses and temporary disappearance (a retention/retired lifecycle, gated on parse validity), and same-list item moves work with unsafe moves properly blocked. The approach — a Structure-Directed Edit Grammar (SDEG) with identity side tables — is what will generalize to further languages.

## 4. External analysis tools, without corrupting the CRDT

A new analysis query layer ingests results from tools like ast-grep as *disposable facts tied to a text snapshot* — converted to UTF-16 ranges, shown as editor decorations, and thrown away when stale. The text CRDT remains the only durable state. Phase 1 (snapshot model, offset conversion, decorations) is implemented and on main.

## 5. js_engine's spec-conformance push

The MoonBit JS interpreter shipped v0.3.0 and went through a systematic test262 campaign: JSON.parse at 100% (142/142) with Proxy-aware revivers, the lexer made UTF-16-accurate for astral characters, a rewritten regex/division disambiguator, five Promise spec fixes, tombstone-based Set iteration, and a timer queue rewrite (2.15× faster).

## 6. The build grew up

Every Canopy-owned manifest migrated from moon.mod.json to moon.mod (TOML), with 13 submodules becoming workspace members — killing a long-standing dependency-resolution workaround. Benchmark regression checks moved close to being a PR gate, with an explicit "skipped is not green" rule.

---

The pace: Canopy's PR numbers went from #445 to #783 over the month, with parallel work across five sibling repositories, developed in close collaboration with AI agents (Claude Code and Codex) under human review.

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- Full log: [[Canopy-Devlog-June-2026|June 2026 devlog (English)]]
