---
title: Canopy Devlog — June 2026
publish: true
tags: [blog, canopy, projectional-editing, english]
created: 2026-07-04T22:30:00+09:00
modified: 2026-07-04T22:30:00+09:00
---

# Canopy Devlog — June 2026

The Canopy development log for June 2026. Rather than enumerating every PR, this log keeps just the essentials so the flow of the work is easy to follow. (English translation of [[Canopy開発日誌-6月|the Japanese original]].)

> Canopy is an editor that treats source code as structure (IR) rather than as strings. Text remains the source of truth, but by letting you directly manipulate the semantic units of a program derived from it, Canopy makes safe structural editing and collaboration — with AI and with other people — much easier.

Overall direction: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

## The big picture this month

The first half of June centered on reorganizing Canopy from "a playground for structural editing experiments" into a clearer product plus a set of libraries. In the second half, the focus shifted to structural editing of Markdown (SDEG: Structure-Directed Edit Grammar), an analysis query layer for ingesting external analysis results, migrating every repository from moon.mod.json to moon.mod, and pushing js_engine's test262 conformance up.

- **Lambda editing**: a scope graph that tracks where names are visible, binding edits scoped to a single block, projection construction via Loom's CstFold, removal of the old `ModuleProjection`, an experiment in alpha-safe beta reduction, and a Grove Level 1 hint that preserves NodeIds across structural edits.
- **Markdown SDEG**: toward making headings and list items targets of structural editing — investigating heading identity stability, same-list moves for list items, block move provenance tracking, and hardening the list-move blocker against adversarial cases.
- **Ideal / Canvas**: adopted Rabbita as a headless UI component library, moved toward Tailwind v4, and made Canvas source-backed — driven by source text as the source of truth rather than UI state. A CodeMirror source editor and FFI file I/O also landed.
- **Analysis query layer**: designed and implemented Phase 1 of a layer that safely brings external analysis results (ast-grep and the like) into Canopy. Analysis results are treated as disposable facts tied to a snapshot, so the durability of the text CRDT is never compromised.
- **moon.mod migration**: migrated every manifest Canopy owns from moon.mod.json to moon.mod (TOML format). Added 13 submodules as workspace members and resolved the dependency-resolution problems of MOON_WORK=off.
- **Loom**: on top of the foundation for Lambda, progress on reparsing only the changed Markdown blocks, mdast export of MarkdownIR with positions, and source-preserving guarantees for the rewrite / canonical formatter.
- **incr**: the Incremental TEA prototype advanced — an experiment in updating TEA-style UIs through incr's dependency graph, covering the 7GUIs stress test and an inactive-root activation policy.
- **js_engine**: an architecture refactor splitting files by responsibility, a fast path that shortcuts bytecode execution, ES2024 Set methods, and better test262/benchmark infrastructure. The second half of the month centered on test262 conformance: JSON.parse reaching a 100% pass rate, Promise spec fixes, regex escape fixes, making the lexer UTF-16-accurate, and a faster timer queue.
- **MoonDsp**: no direct integration with Canopy yet; instead, split up the responsibilities of the Graph runtime and scheduler while keeping the shared foundation in view.
- **Process**: the habit of writing these work logs by cross-referencing Git logs, agent sessions, and project memory continued to settle in.

## 2026/6/1

### Canopy

Sorted out the relationship between Canopy and MoonDsp. The policy now: Canopy is the foundation for structural editing, MoonDsp is the experiment on the music DSL / DSP side, and incr / Loom are shared infrastructure.

In the Lambda projection, added a helper for building `ProjNode`s, a SourceMap token helper that records source positions, and typed views of Block / Hole. On top of that, `let` definitions directly under a module became independent projection nodes called `LetDef`. Previously, a binding row borrowed the ID of its initializer expression or a provisional ID, which made drag & drop and per-binding edits ambiguous about what exactly was being moved. With `LetDef` as a real node, binding rows can be handled straightforwardly as targets of structural editing.

On the performance side, ran the BAND 2b check to see whether `to_flat_proj_incremental` is actually slow. With 1,000 definitions it does stretch to several milliseconds, but real Canopy documents don't reach that scale yet — so no optimization now; we'll pick it back up when it's needed.

### loom / js_engine / process

In Loom, added last-good semantic projection attachment to the json-settings example. When a parse or semantic transformation fails, this keeps the last successful projection alive so the UI doesn't break. The Lambda example got typed views for Block / Hole / LetDef. In js_engine, cleaned up the repeat benchmark runner and the hole / `undefined` handling around Array mutators.

Process-wise, a workflow started to solidify where Codex is used not just for pre-PR review but as the author of implementation plans.

Main PRs / Issues: canopy [#445](https://github.com/dowdiness/canopy/pull/445), [#437](https://github.com/dowdiness/canopy/pull/437), [#447](https://github.com/dowdiness/canopy/pull/447) / loom [#206](https://github.com/dowdiness/loom/pull/206) / js_engine [#186](https://github.com/dowdiness/js_engine/pull/186)

## 2026/6/2

### Canopy

Re-ran the previous day's performance investigation on the JS target. The slowness turned out to be dominated not by the number of tree walks but by pointer chasing when comparing reused CstNodes — the cost of following references inside the cache. Again deferred optimization until we actually need to handle large documents.

In the Lambda projection, renamed `FlatProj` to `ModuleProjection`. More than a rename: the new name makes clear it is "the unit of incremental update for the projection of an entire module." Also recorded the open design questions around ReuseCursor, ProjectionIdentityTracker, and incremental projection as ADRs, and aligned the microbenchmarks to use the same ID numbering as production.

### Incr visualizer / Canvas

The IncrGraph panel now shows not only the shape of the dependency graph but also which cells were recomputed and how long they took. Switched to a buffer that keeps only the latest event per cell so history doesn't grow unboundedly, then added the latest recompute time and a legend. Slow cells now stand out visually — the panel moved a step from a mere structure diagram toward an investigation tool.

In Canvas, prepared to use the graph demo in a more structural-editing-oriented way.

### CI / process

Prepared to put the Ideal web E2E tests on the PR gate — that is, making "Ideal isn't broken in the browser" part of the automated checks on every PR. At the same time, sorted out how to distinguish transient CI failures from real ones.

Main PRs / Issues: canopy [#451](https://github.com/dowdiness/canopy/pull/451), [#452](https://github.com/dowdiness/canopy/pull/452), [#453](https://github.com/dowdiness/canopy/pull/453), [#462](https://github.com/dowdiness/canopy/pull/462), [#465](https://github.com/dowdiness/canopy/pull/465), [#469](https://github.com/dowdiness/canopy/pull/469)

## 2026/6/3

### Canopy

Landed the overall MoonDsp / Canopy direction on main. The source-backed canvas demo also reached the point where a graph built from source text can be manipulated in the browser.

On CI, the Ideal web E2E tests are now on the PR gate. Canopy UI changes are now backed by browser tests in CI, not just manual local checks.

### MoonDsp / incr

In MoonDsp, cleaned up the boundaries of the Graph runtime with future Canopy integration in mind. The rule "don't bring editor-side responsibilities into the audio callback" gradually became clearer.

On the incr side, aligned the public event API naming toward Derived.

Main PRs / Issues: canopy [#445](https://github.com/dowdiness/canopy/pull/445), [#461](https://github.com/dowdiness/canopy/pull/461), [#479](https://github.com/dowdiness/canopy/pull/479)

## 2026/6/4

### Canopy

A day for judging whether Rabbita's headless UI can really be used in Canopy. Through a Disclosure PoC and a dialog spike, confirmed the feel that Rabbita can serve as the foundation of the Ideal UI.

### MoonDsp / js_engine

In MoonDsp, started cutting the facade / internal boundary of the Graph runtime — separating the public API used from outside from the implementation parts that are free to change internally. In js_engine, prepared Array method fast-path delegation and the Test262 runner shadow.

Main PRs / Issues: canopy [#489](https://github.com/dowdiness/canopy/pull/489), [#508](https://github.com/dowdiness/canopy/pull/508), [#511](https://github.com/dowdiness/canopy/pull/511)

## 2026/6/5

### Canopy / Rabbita headless UI

Pulled in Rabbita's patched updates and actually started using headless UI primitives from Ideal and Canvas. A headless UI primitive is a component that provides only state management and keyboard interaction, without imposing any visual CSS. Progress reached Action Menu, ContextMenu, Tabs, and TreeView, and the foundation of the Ideal UI shifted toward Rabbita. ContextMenu in particular used Canvas's right-click menu as a real consumer: verified positioning the menu at the click point, closing on outside click, restoring focus after close, and placement that never overflows the viewport.

### incr / MoonDsp / js_engine

In incr, the typed spreadsheet and Incremental TEA experiments advanced. MoonDsp documented the editor / audio runtime handoff contract, and js_engine pushed forward on Array mutators and runner shadowing.

Main PRs / Issues: canopy [#517](https://github.com/dowdiness/canopy/pull/517), [#523](https://github.com/dowdiness/canopy/pull/523), [#524](https://github.com/dowdiness/canopy/pull/524), [#525](https://github.com/dowdiness/canopy/pull/525), [#526](https://github.com/dowdiness/canopy/pull/526), [#528](https://github.com/dowdiness/canopy/pull/528)

## 2026/6/6

### Canopy / Ideal UI foundation

A major cleanup of Ideal's UI foundation: panel resizing, a live region that communicates state to screen readers, removal of duplicated CSS, the Tailwind v4 migration, plus overlay, toolbar, bottom tabs, panel, inspector, and the outline resize handle — all in small increments.

### Surrounding repositories

In MoonDsp, the split of Graph runtime / scheduler / browser internals advanced. Loom, incr, and js_engine each continued work on their parser runtime, Incremental TEA, and test262 runner respectively.

Main PRs / Issues: canopy [#529](https://github.com/dowdiness/canopy/pull/529), [#532](https://github.com/dowdiness/canopy/pull/532), [#534](https://github.com/dowdiness/canopy/pull/534), [#539](https://github.com/dowdiness/canopy/pull/539), [#541](https://github.com/dowdiness/canopy/pull/541), [#544](https://github.com/dowdiness/canopy/pull/544)

## 2026/6/7

### Canopy / Ideal and protocol

Knocked out Ideal's remaining tasks and the ambiguities in the protocol: outline E2E, how the bridge should handle a batch that only partially succeeded, the unit name for cursor intent, docs on what the coordinates in the protocol actually refer to, and a MoonBit registry cache.

### Canvas / loom

Canvas went back to the next source-backed stage. Source-backed means that instead of treating on-screen nodes/edges as the source of truth, the Graph DSL source behind them is authoritative, and the screen is rebuilt from it. Also progressed on extracting `lib/canvas-graph`.

In Loom, MoonBit parser integration advanced, moving toward emitting syntax artifacts that can be handed to the editor.

Main PRs / Issues: canopy [#553](https://github.com/dowdiness/canopy/pull/553), [#554](https://github.com/dowdiness/canopy/pull/554), [#558](https://github.com/dowdiness/canopy/pull/558), [#560](https://github.com/dowdiness/canopy/pull/560), [#562](https://github.com/dowdiness/canopy/pull/562)

## 2026/6/8

### Canvas

Moved Canvas's source-backed graph toward the CodeMirror source editor: source-backed inspector edits, selection remap, CM6 change-delta lowering, and the CodeMirror source panel mount, in that order. Here the flow became clear: instead of mutating UI state directly, edits are lowered into the Graph DSL source, and the re-parsed source-backed graph is displayed again.

Also settled the recovery policy for parse failures mid-edit. Rather than rolling back input on failure, the editor buffer is always treated as the true input: if the parse succeeds, show the current result; if it fails, show the last-good result.

### loom / incr / js_engine

In Loom, role spans that mark ranges of syntax roles based on parser output; in incr, a keyed DOM benchmark for Incremental TEA; in js_engine, fast paths for Array shift / unshift and runner parity.

Main PRs / Issues: canopy [#569](https://github.com/dowdiness/canopy/pull/569), [#570](https://github.com/dowdiness/canopy/pull/570), [#571](https://github.com/dowdiness/canopy/pull/571), [#576](https://github.com/dowdiness/canopy/pull/576)

## 2026/6/9

### Canopy

Advanced PR 2 of Canvas stable identity. Stable identity here means IDs that let the same node/edge be recognized as the same thing even after the source is edited and the screen is rebuilt. Moved `NodeId` / `EdgeId` toward source-backed string identities and toward removing the temporary shim used for binding remaps.

On the Lambda side, continued cleaning up the scope graph and projection identity as groundwork before heading toward generic projection memos.

### loom / MoonDsp

In Loom, parser runtime attachment and the migration off deprecated syntax advanced. In MoonDsp, the mini-parser replacement campaign progressed and the plan to replace it with Loom became concrete.

Main PRs / Issues: canopy [#571](https://github.com/dowdiness/canopy/pull/571), [#575](https://github.com/dowdiness/canopy/pull/575), [#577](https://github.com/dowdiness/canopy/pull/577)

## 2026/6/10

### incr / Incremental TEA

Pushed Incremental TEA to completion in one stretch. Renderer lifecycle, keyed VDOM diff, benchmarks, and subscriptions were merged, closing all the prototype's major issues.

### MoonDsp / loom / Canopy / js_engine

MoonDsp finished Phase 2 parity of the Loom parser replacement campaign and moved ADR-0016 to Accepted. In Loom, separated-list and attachment work advanced; in Canopy, cleanup of Canvas's runtime seam continued — a seam being the boundary surface, i.e. the work of making explicit where runtime responsibility begins. js_engine released v0.3.0.

Main PRs / Issues: incr [#209](https://github.com/dowdiness/incr/pull/209), [#211](https://github.com/dowdiness/incr/pull/211), [#238](https://github.com/dowdiness/incr/pull/238), [#243](https://github.com/dowdiness/incr/pull/243), [#244](https://github.com/dowdiness/incr/pull/244) / canopy [#611](https://github.com/dowdiness/canopy/pull/611), [#615](https://github.com/dowdiness/canopy/pull/615)

## 2026/6/11

### loom

Landed separated-list (#279) and group shape helpers (#196). Loom's parser / syntax helpers are taking a shape that better supports each of Canopy's language implementations.

### Canopy / architecture redesign

Started the redesign of Canopy's architecture. Landed the S0 proposal and the API boundary ADR, then extracted protocol / wire as S1.

### js_engine / process

In js_engine, measured that the CI cache improvements weren't actually helping, and switched strategy to test262 sharding. On the process side, the division of labor — which agent to use for consultation, review, and implementation planning — firmed up a little more.

Main PRs / Issues: loom [#279](https://github.com/dowdiness/loom/pull/279), [#196](https://github.com/dowdiness/loom/pull/196) / canopy [#587](https://github.com/dowdiness/canopy/pull/587), [#588](https://github.com/dowdiness/canopy/pull/588) / js_engine [#344](https://github.com/dowdiness/js_engine/pull/344), [#349](https://github.com/dowdiness/js_engine/pull/349)

## 2026/6/12

### Canopy / architecture redesign

Pushed the redesign from S2 through S5a in one stretch. Extracted the sync session and transport out of the editor, defined the interface a language runtime provides to the editor (the lang/runtime SPI), fixed where FFI and host capabilities get registered, set responsibility rules for the foundation layer, and locked the boundaries in with an import-graph lint that checks dependency directions. This was not mere file-splitting but institutionalization: making sure the editor, runtime, transport, and host capabilities cannot silently leak into one another.

### Product direction

Decided on a course change: from "a proof of the editor / framework" toward a "write-to-self post product." Post product here doesn't mean a single polished app so much as viewing Canopy as an environment that returns your own thoughts and work logs back to you later. Placed a prototype of the minimal write→surface loop, and moved toward validating the experience of "what you write comes back to you" before building source retrieval.

### Surrounding repositories

In incr, the Incremental TEA renderer and subscriptions advanced further; Loom progressed on parser runtime attachment, js_engine on architecture refactor Stages 0–7, and MoonDsp on splitting responsibilities around the scheduler.

Main PRs / Issues: canopy [#587](https://github.com/dowdiness/canopy/pull/587), [#588](https://github.com/dowdiness/canopy/pull/588), [#589](https://github.com/dowdiness/canopy/pull/589), [#590](https://github.com/dowdiness/canopy/pull/590), [#597](https://github.com/dowdiness/canopy/pull/597), [#599](https://github.com/dowdiness/canopy/pull/599), [#610](https://github.com/dowdiness/canopy/pull/610)

## 2026/6/13

### Canopy / editor-neutral test grammar

To fully decouple the editor from any particular language, introduced a neutral test grammar — a tiny syntax that exists only for testing editor features, not a real language like Lambda — so editor tests don't get dragged around by language specifics. This reduced the problem of editor tests depending too heavily on Lambda-specific syntax.

### Codex integration

Validated the division between the Codex app-server and the MCP wrapper: MCP for the advisory role; the app-server for "building on top of Codex" use cases like streaming and interactive approval.

### Product and Canvas

The write→surface loop gained reordering by resurfacing signal — the score that decides which notes to show again — and same-input ask, re-querying with additional prompts from the same input. In Canvas, cut out the boundary with the runtime and sorted the contracts of who guarantees what.

### loom / js_engine

In Loom, preparation for MarkdownIR and Markdown block reparse advanced; in js_engine, the Stage 0–7 architecture refactor continued.

Main PRs / Issues: canopy [#602](https://github.com/dowdiness/canopy/pull/602), [#604](https://github.com/dowdiness/canopy/pull/604), [#609](https://github.com/dowdiness/canopy/pull/609), [#610](https://github.com/dowdiness/canopy/pull/610), [#611](https://github.com/dowdiness/canopy/pull/611), [#615](https://github.com/dowdiness/canopy/pull/615), [#619](https://github.com/dowdiness/canopy/pull/619)

## 2026/6/14

### Canopy / Lambda CstFold modernization

Advanced the CstFold modernization of the Lambda projection. CstFold is Loom's mechanism for folding a CST (concrete syntax tree) into an AST-like structure; the goal is to reduce hand-written conversions on the Canopy side. Made the differences between existing Canopy semantics and Loom's CstFold explicit, put a compatibility adapter in place, and then moved leaf / if / app / bop and friends over to CstFold step by step. For example, `{ 1 }` and `{ }` are treated differently by Loom's plain CstFold and by Canopy's editor semantics — the call was to absorb that in a Canopy-side adapter rather than changing Loom.

In the same flow, extracted `DefinitionIndex` and began detaching the scope / edit / semantic / companion / ideal consumers from `ModuleProjection`. `DefinitionIndex` is a thin index for looking up which node corresponds to which module definition. Block-local rename now works correctly too.

### Canvas / loom / incr / MoonDsp / js_engine

In Canvas, moved connection-preview compatibility to the MoonBit side and sorted the `defer_sync` ordering of the source-backed demo. Loom progressed on Markdown incremental block reparse, incr on the incr_tea benchmark, MoonDsp on the scheduler facade split, and js_engine on architecture redesign Stages 8–10.

Main PRs / Issues: canopy [#637](https://github.com/dowdiness/canopy/pull/637), [#638](https://github.com/dowdiness/canopy/pull/638), [#640](https://github.com/dowdiness/canopy/pull/640), [#641](https://github.com/dowdiness/canopy/pull/641), [#644](https://github.com/dowdiness/canopy/pull/644), [#647](https://github.com/dowdiness/canopy/pull/647), [#648](https://github.com/dowdiness/canopy/pull/648), [#655](https://github.com/dowdiness/canopy/pull/655), [#639](https://github.com/dowdiness/canopy/pull/639), [#643](https://github.com/dowdiness/canopy/pull/643)

## 2026/6/15

### Canopy / Lambda §20 completion

Finished the block-local support for Lambda editing in one stretch. Block-local support means that not only `let`s directly under the root, but also `let`s inside block expressions are handled correctly as targets of rename / duplicate / move and so on. Landed delete / duplicate / move binding ops, scoping soundness for the move op, the block-shadowing filter, typed `fn` token metadata, and an EditContext cleanup. For move, it turned out that looking only at ordering within the same scope isn't enough — you also have to consider shadowing by lambda params and by outer module defs, or references silently change direction — so the implementation moved to using scope graph resolution.

Finally, deleted the legacy `ModuleProjection`; Lambda has fully migrated to generic projection memos. That means retiring the old Lambda-specific projection cache and building projections with the same generic memo stack as other languages. With this, the binding clause of docs/TODO.md §20 is complete.

### FFI / Ideal

Created `lib/js`: an opaque `Any` handle for holding JS values abstractly on the MoonBit side, a minimal escape hatch for touching JS properties / methods / globals, JSON, and a Promise bridge. The escape hatch is a temporary exit for accessing JS features that don't have a typed API yet. Used it to implement file read/write in `ffi/io`, callable from Ideal's Open / Save toolbar.

On the Ideal side, consolidated `globalThis.__canopy_*` into a single `__canopy_bridge`.

### Surrounding repositories

Loom progressed on the MarkdownIR M0 policy and the M1 heading/paragraph slice; incr on the spreadsheet proof and inactive root; js_engine on ES2024 Set methods and internal-slot cleanup.

Main PRs / Issues: canopy [#660](https://github.com/dowdiness/canopy/pull/660), [#663](https://github.com/dowdiness/canopy/pull/663), [#671](https://github.com/dowdiness/canopy/pull/671), [#673](https://github.com/dowdiness/canopy/pull/673), [#664](https://github.com/dowdiness/canopy/pull/664), [#677](https://github.com/dowdiness/canopy/pull/677), [#666](https://github.com/dowdiness/canopy/pull/666), [#670](https://github.com/dowdiness/canopy/pull/670), [#669](https://github.com/dowdiness/canopy/pull/669), [#668](https://github.com/dowdiness/canopy/pull/668) / loom [#342](https://github.com/dowdiness/loom/pull/342), [#346](https://github.com/dowdiness/loom/pull/346) / incr [#273](https://github.com/dowdiness/incr/pull/273) / js_engine [#356](https://github.com/dowdiness/js_engine/pull/356)

## 2026/6/16

### Canopy / Lambda §20 completion and alpha pilot

Made `ExtractToLet` block-aware. On a block body path it inserts the `let` immediately before the body expression; on a block def path, before the block defs. It distinguishes block scope from lambda scope and adds a capture guard. Rather than forcibly hoisting to the root, the policy is to create the binding in the scope closest to the selection.

Placed the alpha-safe beta pilot for Lambda in `lang/lambda/alpha`. This is an experiment in computing with binder identity so that accidental name collisions can't change a program's meaning: lower the projected term from the `ScopeGraph` into an internal representation, beta-reduce only at the root, and return to a named `Term` in a form where no capture can occur.

Later in the day, landed this alpha-safe core boundary on main and removed the binding-id compatibility fallback — deleting the compatibility path that could still find bindings by old init-ids, so only real LetDef ids are consulted. On the Ideal side, extracted the Tabs UI helper into a small value-derived layer.

### Surrounding repositories

Loom progressed on the MarkdownIR M1 vertical slice, incr on inactive-root cohort measurement, and js_engine on the bytecode call-frame fast path. In js_engine, skipping the env round-trip for param bindings substantially improved the binding step.

Main PRs / Issues: canopy [#674](https://github.com/dowdiness/canopy/pull/674), [#682](https://github.com/dowdiness/canopy/pull/682), [#683](https://github.com/dowdiness/canopy/pull/683), [#684](https://github.com/dowdiness/canopy/pull/684), [#685](https://github.com/dowdiness/canopy/pull/685), issue [#659](https://github.com/dowdiness/canopy/pull/659) / loom [#346](https://github.com/dowdiness/loom/pull/346) / incr [#277](https://github.com/dowdiness/incr/pull/277), [#279](https://github.com/dowdiness/incr/pull/279) / js_engine [#365](https://github.com/dowdiness/js_engine/pull/365), [#366](https://github.com/dowdiness/js_engine/pull/366)

## 2026/6/17

### Canopy / soundness of Lambda editing

Lambda editing shifted from "inspect the generated string" toward "re-parse the post-edit text and check that it produces the intended AST." This came from getting burned by cases like `_copy`, which looks like a trivially small naming difference but in fact can't be read by the lexer and never makes it back into an AST.

- Added adversarial cases for block-local binding edits (move up/down, delete, root/block shadowing).
- Made `ScopeGraph.node_scope` / `node_cutoffs` private, exposed via a query API. Instead of reading internal Maps directly, consumers go through meaningful query functions, making the implementation easier to change later.
- Confirmed via a regression test with re-parsing that the #674 implementation of `ExtractToLet` satisfies the concerns of #659.
- `DuplicateBinding` now uses lexable names like `x1`, `x2`, ... instead of `_copy`, and also avoids candidates that would capture subsequent free references.

With this, [#649](https://github.com/dowdiness/canopy/pull/649) and [#659](https://github.com/dowdiness/canopy/pull/659) are done. [#650](https://github.com/dowdiness/canopy/pull/650) remains, for move/delete indentation.

Main PRs / Issues: canopy [#688](https://github.com/dowdiness/canopy/pull/688), [#689](https://github.com/dowdiness/canopy/pull/689), [#691](https://github.com/dowdiness/canopy/pull/691), [#696](https://github.com/dowdiness/canopy/pull/696)

### Canopy / Grove Level 1 identity hint

To make NodeIds easier to preserve across structural edits, introduced the Grove Level 1 identity hint channel. The aim: even for kind-mismatch edits like wrapping `Var(x)` in `Lam(Param, Var(x))`, don't lose the inner NodeId, so UI state such as selection and folds survives.

On the core side, added `IdentityTransform` and `reconcile_hinted`. `IdentityTransform` is a hint expressing edit intent — "this edit is a wrap," "this unwrap keeps this child" — and `reconcile_hinted` uses that hint to decide the NodeId correspondence between the old and new trees. On the editor side, `SyncEditor` holds the hint corresponding to a span-edit batch, and on the Lambda side, `TreeEditOp` lowers into `IdentityTransform`.

Ops like `WrapInLambda` emit hints usable for NodeId preservation. Ops for which a safe hint is hard to produce — `ExtractToLet`, `ChangeOperator` — conservatively lower to `Opaque`. Finally added E2E coverage for Wrap / Unwrap.

What remains: wrapping the write / read / clear two-ended contract in a `HintChannel` type, and deduplicating the hinted / unhinted reconcile paths.

Main PRs / Issues: canopy [#690](https://github.com/dowdiness/canopy/pull/690), [#697](https://github.com/dowdiness/canopy/pull/697), [#698](https://github.com/dowdiness/canopy/pull/698)

### Canopy / analysis query layer design

Added the design for an analysis query layer that brings external analysis results into Canopy. Rather than mixing the output of ast-grep or `moon ide` directly into editor internals, this is a layer that converts them into a common form Canopy can handle. Analysis results are tied to a snapshot identifying which version of the text they came from, stored as typed facts whose kind is known, and displayed on screen as decorations. Here too, the boundary is preserved: the text CRDT is the only durable state, and analysis results are things to throw away once stale.

Phase 1 only converts ast-grep byte offsets into UTF-16 ranges and highlights them. Byte offsets and editor character positions drift apart easily, so the safe position conversion comes first. No rewrites, no node-id mapping, no protocol changes yet.

Main PRs / Issues: canopy [#687](https://github.com/dowdiness/canopy/pull/687)

### Canopy / Ideal

Moved Ideal's Action Overlay to a form where UI helpers no longer hold Cell / Emit handles directly and receive only values — that is, the helper functions that draw UI don't grab Rabbita's state cells or event emitters; callers pass computed values and callbacks. Also re-split the action overlay's flow and exec.

Main PRs / Issues: canopy [#686](https://github.com/dowdiness/canopy/pull/686)

### loom

MarkdownIR advanced from M1 to recovery / raw node semantics. A raw node here is a node that preserves unsupported or broken input as-is; a recovered node is one the parser produced while recovering. Added raw / recovered diagnostics, mdast export, direct syntax diagnostics, and the recovery adapter contract, and clarified what gets sent to the HTML harness. Notably, the policy became: preserve broken input for the editor, and vary how much canonical rewriting and HTML output normalize, per conversion target.

In the current working tree, a change is in progress — the equivalent of the next [#328](https://github.com/dowdiness/loom/pull/328) — attaching unist `position` to MarkdownIR's mdast export. Seen from the Canopy superproject, the `loom` submodule advanced from `0a827c3` to `3856167` with uncommitted diffs remaining.

Main PRs / Issues: loom [#348](https://github.com/dowdiness/loom/pull/348), [#350](https://github.com/dowdiness/loom/pull/350), [#351](https://github.com/dowdiness/loom/pull/351), [#352](https://github.com/dowdiness/loom/pull/352), [#353](https://github.com/dowdiness/loom/pull/353), [#354](https://github.com/dowdiness/loom/pull/354), [#355](https://github.com/dowdiness/loom/pull/355), [#356](https://github.com/dowdiness/loom/pull/356), [#357](https://github.com/dowdiness/loom/pull/357), [#358](https://github.com/dowdiness/loom/pull/358)

### incr

incr_tea moved from inactive-root measurement to an activation policy. An inactive root is a hidden UI subtree whose DOM stays in place while updates are paused; the activation policy is the rule for when to start it moving again. Re-checked the ratio table, added an activation trigger probe, decided the policy in docs, and implemented it. The `incr` inside the Loom submodule also advanced from `34ac477` to `f7681bc`.

Main PRs / Issues: incr [#281](https://github.com/dowdiness/incr/pull/281), [#282](https://github.com/dowdiness/incr/pull/282), [#284](https://github.com/dowdiness/incr/pull/284), [#285](https://github.com/dowdiness/incr/pull/285)

### js_engine

The `needs_own_env` line of bytecode optimizations continued. This optimization determines ahead of time whether a function call really needs a new Environment, and skips creating one when it doesn't. Skipped `Environment::new` for leaf bytecode functions, avoided the realm-proto wrapper for same-realm callees, and folded the active-override `Ref`s into a single `Ref[FunctionRealmProtos?]`. Finally added an `exec/for_of` row to the benchmark table.

Main PRs / Issues: js_engine [#367](https://github.com/dowdiness/js_engine/pull/367), [#368](https://github.com/dowdiness/js_engine/pull/368), [#369](https://github.com/dowdiness/js_engine/pull/369), [#370](https://github.com/dowdiness/js_engine/pull/370), [#371](https://github.com/dowdiness/js_engine/pull/371), [#372](https://github.com/dowdiness/js_engine/pull/372)

### Process notes

This entry was written by cross-referencing the Git logs of Canopy / Loom / nested incr / js_engine, the current uncommitted diffs, Claude project memory, and Codex memory.

The three points that matter most as of 6/17:

1. Lambda edit validation re-parses the actual post-edit text to verify it.
2. Analysis facts are tied to a snapshot and treated as disposable at any time.
3. The Grove hint channel must be made explicit as a `HintChannel` type before extending it to the next structural-editing language.

## 2026/6/18

### Canopy / analysis query layer implementation

Implemented Phase 1 of the analysis query layer designed the previous day and landed it on main. On the `lib/analysis` side: a `SourceSnapshot` carrying document identity, version, a 32-bit hash, and UTF-16 length; a `PatternMatchFact` carrying a UTF-16 range plus pattern id / captures; and a helper that converts ast-grep byte offsets into the editor's UTF-16 offsets.

In Canopy's `analysis` package, added adapters that convert ast-grep matches into `PatternMatchFact`s and lower them into protocol decorations and match-list entries. An initial ast-grep rule that picks up MoonBit `fn` definitions also landed. Along the way, discovered that judging snapshot identity by version and hash alone can mistake a different document with identical content for the current one — fixed by also checking `doc_id` and `utf16_len`.

Phase 1 has now reached "accept external analysis results as disposable facts tied to a snapshot, and convert them into values for range highlights / match lists." The host-side FFI wiring — actually passing ast-grep results in from JS and displaying them in the UI — remains for the next stage.

Main PRs / Issues: canopy [#699](https://github.com/dowdiness/canopy/pull/699), [#692](https://github.com/dowdiness/canopy/pull/692), [#693](https://github.com/dowdiness/canopy/pull/693), [#694](https://github.com/dowdiness/canopy/pull/694), [#695](https://github.com/dowdiness/canopy/pull/695)

### loom / MarkdownIR

MarkdownIR reached the point of attaching unist `position` to the mdast export. Positions are created at the export boundary from MarkdownIR's internal source origins and a `LineIndex`, so MarkdownIR itself isn't dragged around by mdast position semantics. Position-drift-prone cases — non-BMP characters, raw / recovered nodes, block separators, fenced code blocks, CRLF — are pinned down with tests.

After that, moved on to the rewrite / canonical formatter side of [#333](https://github.com/dowdiness/loom/pull/333), adding source-preserving rewrite smoke coverage for code fences and links. For code fences in particular, verified that rewriting only the content doesn't damage the fence itself or the surrounding source, and that rewrite boundaries hold even for unclosed fences.

Main PRs / Issues: loom [#359](https://github.com/dowdiness/loom/pull/359), [#360](https://github.com/dowdiness/loom/pull/360), [#361](https://github.com/dowdiness/loom/pull/361)

### incr / Incremental TEA

Added the 7GUIs stress test to incr_tea. Counter, Temperature Converter, Flight Booker, Timer, CRUD, Circle Drawer, and Cells each live in their own package, expanding the experimental surface for how far TEA-style UIs can be driven by incr's dependency graph.

Also tidied small APIs around `on_change` and pointer offsets. Following the previous day's inactive-root activation policy, this is now the stage of lining up multiple UI patterns — not single small demos — to see whether incr holds up as an incremental UI substrate distinct from Rabbita. Next up are the remaining TEA follow-ups, especially the local pointer coordinate work.

Main PRs / Issues: incr [#291](https://github.com/dowdiness/incr/pull/291), [#268](https://github.com/dowdiness/incr/pull/268), [#286](https://github.com/dowdiness/incr/pull/286), [#287](https://github.com/dowdiness/incr/pull/287), [#288](https://github.com/dowdiness/incr/pull/288), [#289](https://github.com/dowdiness/incr/pull/289), [#290](https://github.com/dowdiness/incr/pull/290)

### Rabbita / pointer events

In the Rabbita fork that Canopy references, the pointer event binding branch advanced. Added `PointerEvent` bindings to the DOM / HTML / subscription layers, preparing downstream UI to treat input as pointer input rather than mouse-only. A follow-up fix aligned constructors and cast style with the existing mouse event conversions.

In the Canopy parent repository, the `rabbita` submodule pointer still remains as an uncommitted diff.

### js_engine

Fixed a spec bug in Set iteration. If a callback inside `Set.prototype.forEach` deletes the last element and re-adds it, the spec says the re-added value occupies a new unvisited slot and must be visited again. To match, entries are no longer physically removed during an active `forEach` — they become tombstones, compacted after the outer iteration finishes. `clear()`, `values()` / `keys()` / `@@iterator`, and `entries()` were all made tombstone-aware, with a one-shot-guard test to avoid infinite loops.

In parallel, a PR is open to make `Function.prototype.toString` return the original source. It's a larger change threading source text and spans through the parser / AST / runtime; after review it progressed to building spans from the original source. Not yet on main. Another PR reorganized the docs layout for roadmap / design / decisions and refreshed the current architecture target and the test262 snapshot.

Main PRs / Issues: js_engine [#373](https://github.com/dowdiness/js_engine/pull/373), [#374](https://github.com/dowdiness/js_engine/pull/374), [#375](https://github.com/dowdiness/js_engine/pull/375), [#310](https://github.com/dowdiness/js_engine/pull/310), [#357](https://github.com/dowdiness/js_engine/pull/357)

### Process notes

As of today, the Canopy parent repository has dirty submodule pointers for `loom` and `rabbita`. `loom` has advanced to `b26a304`, with its inner `incr` submodule at `7a971ab`. `rabbita` is at `54b3188` on the pointer-event branch, but whether the parent takes it is still unsettled.

The day's overall shape: on Canopy proper, the analysis layer reached a minimal implementation; around it, MarkdownIR's export / rewrite guarantees, incr_tea's UI stress surface, Rabbita's pointer input, and js_engine's Set spec conformance and docs cleanup all advanced in parallel.

## 2026/6/19

### Canopy / Markdown SDEG heading

Advanced the SDEG (Structure-Directed Edit Grammar) investigation into making Markdown headings targets of structural editing.

- Added a heading identity probe exploring how stable a Markdown heading's NodeId stays across edits ([#716](https://github.com/dowdiness/canopy/pull/716)).
- Placed a design sketch of the SDEG NodeId side table as a test ([#717](https://github.com/dowdiness/canopy/pull/717)) — the concept of an auxiliary table for looking up each block's NodeId across the document.
- Added E2E validation of the heading edit path ([#718](https://github.com/dowdiness/canopy/pull/718)) and updated docs carrying the SDEG Phase 0 findings into the Phase 1 plan ([#719](https://github.com/dowdiness/canopy/pull/719)).

Main PRs / Issues: canopy [#716](https://github.com/dowdiness/canopy/pull/716), [#717](https://github.com/dowdiness/canopy/pull/717), [#718](https://github.com/dowdiness/canopy/pull/718), [#719](https://github.com/dowdiness/canopy/pull/719)

### js_engine

- Implemented the shared AsyncGeneratorPrototype chain ([#405](https://github.com/dowdiness/js_engine/pull/405)). Per spec §27.4, every generator created by an `async function*` now shares a common prototype chain.
- Added an `end_offset` field to tokens, removing the parser's need to re-scan the source ([#403](https://github.com/dowdiness/js_engine/pull/403)). Previously the parser walked the source again to learn where a token ends; now the lexer records the end offset in UTF-16 code units up front.
- Added runtime atomics test coverage ([#402](https://github.com/dowdiness/js_engine/pull/402)) and regression tests for the test262 tooling ([#401](https://github.com/dowdiness/js_engine/pull/401)).

Main PRs / Issues: js_engine [#405](https://github.com/dowdiness/js_engine/pull/405), [#403](https://github.com/dowdiness/js_engine/pull/403), [#402](https://github.com/dowdiness/js_engine/pull/402), [#401](https://github.com/dowdiness/js_engine/pull/401)

## 2026/6/20

### Canopy / Markdown list SDEG

Pushed the foundation for structural editing of Markdown lists forward in one stretch.

- Extracted the Markdown SDEG heading side table, making the heading-specific auxiliary table independent ([#722](https://github.com/dowdiness/canopy/pull/722)).
- Added block move provenance ([#723](https://github.com/dowdiness/canopy/pull/723)) — tracking "where a block came from" when it moves, a prerequisite for the list-item moves that follow.
- Hardened the Markdown list move blocker ([#726](https://github.com/dowdiness/canopy/pull/726)), which properly rejects cases where moving a list item isn't safe (moves across lists of different depth or kind, etc.).
- Enabled same-list Markdown item moves ([#731](https://github.com/dowdiness/canopy/pull/731)) and added consumption of list item payloads ([#730](https://github.com/dowdiness/canopy/pull/730)).

Main PRs / Issues: canopy [#722](https://github.com/dowdiness/canopy/pull/722), [#723](https://github.com/dowdiness/canopy/pull/723), [#726](https://github.com/dowdiness/canopy/pull/726), [#730](https://github.com/dowdiness/canopy/pull/730), [#731](https://github.com/dowdiness/canopy/pull/731)

### js_engine / test262 conformance push

A big fix day for js_engine. Systematically knocked down test262 failures, and JSON.parse reached a 100% pass rate.

- **lexer: astral_count tracking** ([#408](https://github.com/dowdiness/js_engine/pull/408)). MoonBit strings are UTF-16-based, but the lexer computed offsets in code points, so any source containing surrogate pairs had every token position shifted. Added an `astral_count` variable, incremented on each non-BMP character, to correct offsets to UTF-16 code-unit counts.
- **Proxy-aware JSON.parse reviver** ([#419](https://github.com/dowdiness/js_engine/pull/419)). Replaced the reviver logic with interpreter-aware abstract operations so Proxy `[[Get]]`/`[[Delete]]`/`[[DefineOwnProperty]]` traps fire correctly. JSON/parse: 36 failures → 0 (100%, 142/142).
- **Five Promise spec fixes** ([#413](https://github.com/dowdiness/js_engine/pull/413)): the `Symbol.toStringTag` descriptor, TypeError on non-object this, the iter-poisoned case, `Promise.any`'s resolve/reject element guard, and honoring newTarget.prototype.
- **`__lookupGetter__`/`__lookupSetter__` and `replaceAll` fixes** ([#412](https://github.com/dowdiness/js_engine/pull/412)): Annex B getter/setter lookup, plus `String.prototype.replaceAll`'s IsRegExp check and `Symbol.replace` override support.
- **Surrogate-safe string slicing** ([#411](https://github.com/dowdiness/js_engine/pull/411)): fixed the `classify_by_edition` tool panicking on emoji and the like.
- **Regex `\u`/`\x` escape support** ([#420](https://github.com/dowdiness/js_engine/pull/420)): fixed a bug where `\uXXXX`/`\xHH`/`\f`/`\v`/`\0`/`\b` inside character classes were all treated as literal characters. Also covers Annex B identity escapes and non-combining surrogate pairs in non-Unicode mode.
- **name/length/property order for anonymous built-ins** ([#410](https://github.com/dowdiness/js_engine/pull/410)) and the **singleton %GeneratorPrototype%** ([#407](https://github.com/dowdiness/js_engine/pull/407)).
- test262: skipped await-dictionary (`Promise.allKeyed`/`allSettledKeyed`) ([#377](https://github.com/dowdiness/js_engine/pull/377)).

Main PRs / Issues: js_engine [#408](https://github.com/dowdiness/js_engine/pull/408), [#419](https://github.com/dowdiness/js_engine/pull/419), [#413](https://github.com/dowdiness/js_engine/pull/413), [#412](https://github.com/dowdiness/js_engine/pull/412), [#411](https://github.com/dowdiness/js_engine/pull/411), [#420](https://github.com/dowdiness/js_engine/pull/420), [#410](https://github.com/dowdiness/js_engine/pull/410), [#407](https://github.com/dowdiness/js_engine/pull/407)

## 2026/6/21

### Canopy / Markdown lists + start of the moon.mod migration

- Continued structural editing of Markdown lists, landing list payload consumption ([#730](https://github.com/dowdiness/canopy/pull/730)) and same-list item moves ([#731](https://github.com/dowdiness/canopy/pull/731)) on main.
- Documented the package map ([#736](https://github.com/dowdiness/canopy/pull/736)), organizing the dependencies and names of all Canopy packages.
- On that basis, converted the first wave of the moon.mod.json→moon.mod migration: the Rabbita UI lib cluster ([#737](https://github.com/dowdiness/canopy/pull/737)) and lib/visualizer ([#738](https://github.com/dowdiness/canopy/pull/738)). Migrating to moon.mod (TOML format) enables dependency resolution via MoonBit's workspace membership and moves us closer to making `NEW_MOON_MOD=0` the default.

Main PRs / Issues: canopy [#736](https://github.com/dowdiness/canopy/pull/736), [#737](https://github.com/dowdiness/canopy/pull/737), [#738](https://github.com/dowdiness/canopy/pull/738)

### js_engine / lexer, spec fixes, perf

- **Regex/division disambiguation after `}`** ([#422](https://github.com/dowdiness/js_engine/pull/422)). Completely rewrote the context tracking that decides whether a `/` after `}` is division or the start of a regex literal. Introduced a brace-is-block stack tracking whether each `}` closes a statement block or an object literal, a ternary colon stack distinguishing ternary colons from label/case colons, and kept the brace-is-block stack sound inside nested class `extends` clauses.
- **bind length via ToIntegerOrInfinity + JS ToString coercion for new Function** ([#431](https://github.com/dowdiness/js_engine/pull/431)). Fixed `Function.prototype.bind`'s length handling to use `ToIntegerOrInfinity` per spec, and made `new Function` evaluate its arguments with the JS `ToString` abstract operation instead of MoonBit's `.to_string()`.
- **Annex B web-compat call-assign** ([#428](https://github.com/dowdiness/js_engine/pull/428)). In non-strict mode, a `CallExpression` on the left of an assignment now produces a runtime `ReferenceError` rather than a parse-time error, per Annex B compatibility.
- **Analysis-family growth convention docs** ([#332](https://github.com/dowdiness/js_engine/pull/332)): documented the file-layout rules for static analysis.
- **Timer queue moved to a priority queue** ([#433](https://github.com/dowdiness/js_engine/pull/433)). Replaced the old `Array[TimerTask] + sort_by + remove(0)` (O(n² log n)) with `@priority_queue.PriorityQueue[TimerTask]` (O(n log n)); draining 200 timers went from 4.19ms to 1.95ms (2.15× faster). Cancellation became lazy deletion.

Main PRs / Issues: js_engine [#422](https://github.com/dowdiness/js_engine/pull/422), [#431](https://github.com/dowdiness/js_engine/pull/431), [#428](https://github.com/dowdiness/js_engine/pull/428), [#332](https://github.com/dowdiness/js_engine/pull/332), [#433](https://github.com/dowdiness/js_engine/pull/433)

## 2026/6/22

### Canopy / moon.mod migration complete

Completed the moon.mod.json→moon.mod migration for every manifest Canopy owns ([#740](https://github.com/dowdiness/canopy/pull/740)) — the biggest change of the week. It includes:

- **Converted 7 Canopy-owned manifests**: root, lib/semantic, examples/resizable, examples/codemirror_demo, examples/block-editor, examples/canvas, examples/ideal.
- **Added 13 submodules as workspace members**: loom/examples/{markdown,json,lambda,graph-dsl}, loom/{loom,seam,pretty,text-change,moji,egglog,egraph}, event-graph-walker, rle, order-tree.
- **Removed every MOON_WORK=off**: moon.mod's `import { }` syntax cannot resolve dependencies without workspace membership, so MOON_WORK=off was stripped from benchmarks, E2E tests, the Cloudflare deploy, and JS build scripts.
- **CI work**: introduced a shared filter suppressing vendored-submodule errors (`scripts/vendored-check-common.sh`). It suppresses the 21 pre-existing errors submodules raise in the workspace-wide check, while a `--keep` option ensures each submodule's own CI doesn't hide its own failures.
- **Cloudflare deploy fix**: workspace builds emit artifacts to `_build/js/release/build/<module>/<pkg>/`, which mismatches the paths vite-plugin-moonbit expects; absorbed the difference with symlinks ([#335](https://github.com/dowdiness/canopy/pull/335)).

In parallel, handled the loom submodule's quickcheck 0.14 Arrow API compat and updated AGENTS.md's submodule guidance.

Main PRs / Issues: canopy [#740](https://github.com/dowdiness/canopy/pull/740), [#335](https://github.com/dowdiness/canopy/pull/335)

## 2026/6/23

### Canopy / submodule bumps and an HTML block fix

Follow-up from Monday's large migration, plus subsequent submodule updates.

- **Submodule bumps**: egraph's moon.mod migration (loom#455), event-graph-walker's trait split fix (#58), pulling in alga v0.4.0, and graphviz's DirectedGraph compatibility fix. Each of these submodules is doing its own moon.mod.json→moon.mod migration, and Canopy followed along.
- **HTML blocks §4.6**: bumped the loom submodule and made Markdown HTML blocks part of projection block children. There was a bug where, in block mode, an HTML block was treated as `text:null`/`editable:false` and disappeared from display; fixed by populating `HtmlBlock` with a proper token span.
- Bumps reflecting the completed moon.mod migrations of the remaining submodules (svg-dsl, rle, order-tree, graphviz) are in progress as [#742](https://github.com/dowdiness/canopy/pull/742).

Main PRs / Issues: canopy [#742](https://github.com/dowdiness/canopy/pull/742)

## 2026/6/24

### Canopy / Markdown SDEG lifecycle

Moved the Markdown SDEG heading side table from a mere "heading-ID correspondence table" toward a structure with parse validity and a lifecycle.

- Gated the heading side table's lifecycle on parse validity ([#763](https://github.com/dowdiness/canopy/pull/763)) — a boundary against carelessly updating the stable-ID table based on a broken parse result.
- Made the SDEG retention threshold configurable: a heading that disappears isn't discarded immediately but transitions to a `Retired` entry after a grace period ([#755](https://github.com/dowdiness/canopy/pull/755), original PR [#746](https://github.com/dowdiness/canopy/pull/746)). Follow-up fixes covered a returning heading dropping its stable id, and duplicate stable ids among retired rows.
- Recorded the loomgen `RawKind` / content-hash identity decision in docs ([#750](https://github.com/dowdiness/canopy/pull/750)) — a record of judgment on the granularity at which Markdown raw / recovered regions get identity.
- Cleaned up warnings around event-graph-walker, loom, alga, and lang/markdown ([#754](https://github.com/dowdiness/canopy/pull/754)).

The day's SDEG work was less about moving headings and list items themselves, and more about pinning down how the table that tracks structural-editing targets withstands broken input and temporary disappearance.

Main PRs / Issues: canopy [#746](https://github.com/dowdiness/canopy/pull/746), [#750](https://github.com/dowdiness/canopy/pull/750), [#754](https://github.com/dowdiness/canopy/pull/754), [#755](https://github.com/dowdiness/canopy/pull/755), [#763](https://github.com/dowdiness/canopy/pull/763)

### Canopy / benchmark CI and projection maps

Moved the benchmark regression workflow closer to being a PR gate. Submodule gitlink and shared vite plugin changes are now included in the benchmark gate, and the workflow itself was parallelized and sped up ([#762](https://github.com/dowdiness/canopy/pull/762)). How far to include pre-existing problems from vendored submodules in the gate remains hard, which later led to the "don't treat skipped checks as green" rule.

On the projection structure side, added a RoseNode map and constructor API ([#761](https://github.com/dowdiness/canopy/pull/761)) — one more small foothold for handling tree projections from outside, toward the ProjNode map that follows.

Main PRs / Issues: canopy [#761](https://github.com/dowdiness/canopy/pull/761), [#762](https://github.com/dowdiness/canopy/pull/762)

### loom / MarkdownIR

In the loom that Canopy references, the Markdown IR implementation files were split up ([#472](https://github.com/dowdiness/loom/pull/472)) and raw kind / Tabs handling was fixed ([#473](https://github.com/dowdiness/loom/pull/473)) — underpinnings for the Markdown SDEG side to handle raw / recovered nodes stably.

Main PRs / Issues: loom [#472](https://github.com/dowdiness/loom/pull/472), [#473](https://github.com/dowdiness/loom/pull/473)

### js_engine

The test262 conformance push continued.

- Separated environment markers into a dedicated map ([#438](https://github.com/dowdiness/js_engine/pull/438)).
- On the call-evaluation side, cleaned up grouping unwrap dispatch and fixed NamedEvaluation for logical assignment operators.
- Brought the `arguments.callee` accessor for sloppy functions with non-simple params in line with the spec ([#440](https://github.com/dowdiness/js_engine/pull/440)).
- Fixed the brand check and done flag of `SetIteratorPrototype.next` ([#441](https://github.com/dowdiness/js_engine/pull/441)).
- Unified `[[OwnPropertyKeys]]` enumeration onto the canonical op ([#442](https://github.com/dowdiness/js_engine/pull/442)).

Main PRs / Issues: js_engine [#438](https://github.com/dowdiness/js_engine/pull/438), [#440](https://github.com/dowdiness/js_engine/pull/440), [#441](https://github.com/dowdiness/js_engine/pull/441), [#442](https://github.com/dowdiness/js_engine/pull/442)

## 2026/6/25

### Canopy / SDEG snapshot validity

Made the Markdown SDEG heading snapshot validity explicit ([#766](https://github.com/dowdiness/canopy/pull/766)). Extending the previous day's parse validity gate, the side table no longer leaves ambiguous which snapshot it is valid against.

Further, review-response work proceeded on the "wire Markdown SDEG snapshot validity" PR (the equivalent of [#767](https://github.com/dowdiness/canopy/pull/767), commit `f4effe5`). Per the agent history, targeted tests for `lang/markdown/proj` and `lang/markdown/companion`, `moon fmt`, `moon info`, and the workspace `moon check` all pass. However, the `Editor Response Benchmark` remained `skipping`, and since repo policy says "skipped is not green," the merge was held. This made it clear that skips in CI need explicit handling.

Main PRs / Issues: canopy [#766](https://github.com/dowdiness/canopy/pull/766), [#767](https://github.com/dowdiness/canopy/pull/767)

### Canopy / ProjNode map and CI cleanup

Following the RoseNode map, added a ProjNode map ([#765](https://github.com/dowdiness/canopy/pull/765)). A direction is emerging where projection trees are handled through common map operations rather than per-language special-casing.

On CI, a Playwright image update landed along with dependabot bumps for Vite / Vitest / React DOM / actions checkout. In the local Canopy worktree, diffs remain for benchmark workflow comment cleanup, removing `alga` from the vendored check filter, and advancing the `loom` submodule pointer to `6d7778b` — the loom side including the Markdown raw kind and Tabs handling fixes.

Main PRs / Issues: canopy [#765](https://github.com/dowdiness/canopy/pull/765), [#727](https://github.com/dowdiness/canopy/pull/727), [#547](https://github.com/dowdiness/canopy/pull/547), [#549](https://github.com/dowdiness/canopy/pull/549), [#550](https://github.com/dowdiness/canopy/pull/550)

### js_engine

Spec conformance work around Map / Set / Promise / Proxy.

- Extended `Reflect.ownKeys` to Map / Set / Promise, and made Proxy's `[[OwnPropertyKeys]]` classify symbol keys correctly ([#445](https://github.com/dowdiness/js_engine/pull/445)).
- Added `test262_failing_diff.js` for viewing per-mode regression diffs in test262 ([#446](https://github.com/dowdiness/js_engine/pull/446)).
- On a branch, fixes continued for Map / Set expando assignment, Promise instance constructor keys, computed Map / Set writes, and stopping Map / Set writes via array own descriptors. In the agent history this is PR [#449](https://github.com/dowdiness/js_engine/pull/449), adding a regression where an array prototype gets inserted into a Map / Set subclass chain — with `moon check`, targeted regressions, `moon test`, `moon info`, `moon fmt`, `moon check --deny-warn`, and release tests all passing.

Main PRs / Issues: js_engine [#445](https://github.com/dowdiness/js_engine/pull/445), [#446](https://github.com/dowdiness/js_engine/pull/446), [#449](https://github.com/dowdiness/js_engine/pull/449)

## 2026/6/26

### Canopy / JSON role spans + CI cleanup

Finished the editor-decoration integration of JSON role spans in one stretch. The role spans Loom emits at parse time (syntax information distinguished per kind of value) now travel across the FFI boundary to the CodeMirror editor and render as decorations (#781, #782, #783).

- #781: integrated Loom's JSON role-span export into the FFI/CodeMirror path, creating the MoonBit→JS data route.
- #782: wrapped role spans in a `Derived::map` reactive cell so they auto-update as the source text changes.
- #783: applied role spans as editor decorations, displaying them as parser-driven syntax coloring.

The vendored error-suppression list cleanup was completed. The entries remaining after the 6/22 moon.mod migration — alga/rle (#773), order-tree (#778), graphviz/svg-dsl (#779), event-graph-walker (#780) — were removed from suppression one by one, reflecting those submodules' completed moon.mod migrations.

The benchmark regression CI also got faster (#777): added Canopy subpackages and loom example benchmarks to the -p flags, bumped the cache key v3→v4, and reworked the execution order of moon-update and moon bench.

Main PRs / Issues: canopy [#781](https://github.com/dowdiness/canopy/pull/781), [#782](https://github.com/dowdiness/canopy/pull/782), [#783](https://github.com/dowdiness/canopy/pull/783), [#773](https://github.com/dowdiness/canopy/pull/773), [#777](https://github.com/dowdiness/canopy/pull/777), [#778](https://github.com/dowdiness/canopy/pull/778), [#779](https://github.com/dowdiness/canopy/pull/779), [#780](https://github.com/dowdiness/canopy/pull/780)

### Loom / arrow lambda + Pratt reuse

Loom centered on the P2 fixes for arrow lambda syntax: regressions around block bodies, soft newlines, typed arrow params (TypeAnnot/TypInt/TypeUnit/TypeArrow), and right-recursive TypeArrow with parenthesized type annotations. Bumped the incr dependency 0.9.0→0.11.0, and landed projection slice trimming for JSON role spans.

Retroactive Pratt reuse groundwork landed (#475) — preparation for reusing Pratt parser state across backtracking. Comparability of the deep nested-lambda benchmark workload's B variant was also restored (#476).

Main PRs / Issues: loom [#475](https://github.com/dowdiness/loom/pull/475), [#476](https://github.com/dowdiness/loom/pull/476)

### js_engine / NFE binding + async fixes + test262 infrastructure

Cluster 11 of the test262 conformance push (NFE self-name binding) is complete (#463). Implemented `FunctionNameBinding` per spec, threaded strict mode all the way through to the bytecode StoreName env assign, and added the has_name_binding control that suppresses it for generator/async methods.

Fixed async function edge cases (#468): parameter TDZ, `this` in non-strict mode, arrow `arguments`, and mapped arguments. On the Array side: TypeError for `ArraySpeciesCreate` with a non-object constructor (#467), handling deletion of `Array.prototype[Symbol.iterator]` (#462), the Int64 migration of array-like lengths (#461), and prototype delegation for `reverse/fill/copyWithin/sort` (#471).

On CI, added a copilot toolchain cache and a Test262 feature-gap comparison tool (#460), plus baseline ratchet and calibration automation.

Main PRs / Issues: js_engine [#463](https://github.com/dowdiness/js_engine/pull/463), [#468](https://github.com/dowdiness/js_engine/pull/468), [#466](https://github.com/dowdiness/js_engine/pull/466), [#467](https://github.com/dowdiness/js_engine/pull/467), [#471](https://github.com/dowdiness/js_engine/pull/471), [#462](https://github.com/dowdiness/js_engine/pull/462), [#461](https://github.com/dowdiness/js_engine/pull/461), [#460](https://github.com/dowdiness/js_engine/pull/460), [#470](https://github.com/dowdiness/js_engine/pull/470)

### Process notes

6/26 marked a milestone for Canopy with the JSON role span editor-decoration integration. The vendored suppression cleanup wrapped up, and the benchmark CI got faster. Loom centered on the P2 arrow lambda fixes, while js_engine saw Cluster 11 completed and a run of async/Array fixes.


