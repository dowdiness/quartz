---
title: Refactoring Prompts
publish: true
tags:
  - ai
aliases:
  - Refactoring Prompts
created: 2026-04-09T22:00:34+09:00
modified: 2026-04-09T22:00:34+09:00
---

# Refactoring Prompts

コードの構造改善・設計見直しに使うプロンプト集です。
対象の規模と目的に応じて3種類を使い分けます。

## どれを使うか

| プロンプト | 対象 | 目的 | 典型的なケース |
| --- | --- | --- | --- |
| 基本リファクタリング | 単一モジュール〜小規模プロジェクト | 構造改善・重複排除 | 関数の整理、ファイル分割、責務の明確化 |
| 大規模コードベース | 大規模・複雑なシステム | 安全な段階的改善 | 影響範囲が読めない既存システムの部分改修 |
| 設計再構築 | アーキテクチャレベル | 設計の根本的見直し | レイヤー構成の変更、依存方向の修正、境界の再定義 |

迷ったら「基本」を使う。スコープが大きすぎると感じたら「大規模」に切り替え、コード単位でなく設計そのものに問題があるなら「設計再構築」を使う。

---

## 基本リファクタリング

既存コードの構造を改善するための汎用プロンプト。テスト→現状把握→段階的変更→検証というサイクルで進める。全体を一度に書き直すのではなく、小さな変更を積み重ねるアプローチ。

**向いている場面:**
- 単一のモジュールやファイルの整理
- 明らかな重複やもつれた責務の解消
- 「動くけど読みにくい」コードの改善

```md
Refactor this project's code to improve structural clarity, reduce duplication, and increase maintainability — while preserving both functional correctness and non-functional qualities.

This is not a cosmetic cleanup task. Treat this as a careful, evidence-based redesign with explicit safety guarantees.

---

## 1. Establish a safety net before refactoring

- Identify the **core invariants** of the system (what must always be true).
- Implement tests that enforce these invariants:
    - Prefer **property-based tests** when possible.
    - Otherwise, use well-designed unit/integration tests that cover equivalent conditions.
- Ensure tests fail loudly:
    - Do not silently ignore errors (e.g. avoid patterns like `None => true`)
    - Invalid inputs or generator failures must surface as test failures
- Verify test input coverage:
    - Inspect or log generated/test inputs to confirm important cases are actually exercised

Do not begin structural changes until this safety net is in place and passing.

---

## 2. Understand the current system before changing it

- Map:
    - responsibilities of modules/functions
    - data flow and control flow
    - duplicated logic and coupling points
- Identify:
    - mixed responsibilities
    - hidden dependencies
    - hard-coded behavior
- Note any bugs or inconsistencies discovered during testing

---

## 3. Plan refactoring incrementally

- Prefer **small, high-impact changes first**:
    - split large files
    - extract shared functions
    - remove obvious duplication
- After every change:
    - run tests
    - run compiler / static checks
- For mechanical refactoring (e.g. file splits):
    - verify that each function exists in exactly one place
    - ensure no duplication or missing definitions

Avoid large, unstructured rewrites.

---

## 4. Improve structure without over-engineering

Refactor toward:

- higher cohesion (each unit has a clear responsibility)
- lower coupling (fewer implicit dependencies)
- explicit boundaries and interfaces
- reduced duplication

Rules:

- Do not introduce abstractions unless they clearly improve structure
- Do not force unification if the language or problem does not support it
- If a limitation exists (e.g. type system constraints), document it and adapt

---

## 5. Preserve functional correctness

For every change:

- Explain:
    - what the original behavior was
    - what changed structurally
    - why behavior is preserved
- If behavior might change:
    - explicitly state the risk
    - justify the change

---

## 6. Preserve and evaluate non-functional properties

Ensure that the following are preserved or improved:

- performance (time complexity, hot paths)
- memory usage (allocations, copying, lifetime)
- concurrency safety (shared state, race conditions)
- error handling (no silent failures, correct propagation)
- observability (logging, debugging visibility)
- API/behavioral compatibility

If any of these may degrade:

- explicitly identify the impact
- explain the trade-off
- justify why it is acceptable (or avoid the change)

---

## 7. Address deeper design issues carefully

- Consolidate duplicated logic where safe
- Improve APIs instead of repeating workarounds
- Separate:
    - core logic
    - side effects (I/O, UI, external systems)
- Ensure ownership of state is clear

If duplication remains due to real algorithmic differences, document why.

---

## 8. Scale the process appropriately

- For small changes:
    - keep the process lightweight
- For large systems:
    - use full analysis, testing, and staged refactoring

Avoid unnecessary process overhead, but never skip safety-critical steps.

---

## 9. Reflect and improve

After refactoring:

- Identify:
    - what improvements had the biggest impact
    - what was unnecessary or low-value
    - what limitations (language, design) affected outcomes
- Capture lessons for future refactoring work

---

## Output format

1. Invariants and test strategy
2. Current structure diagnosis
3. Refactoring plan (with risks)
4. Step-by-step changes
5. Functional correctness validation
6. Non-functional impact analysis
7. Remaining issues and limitations
8. Lessons learned

---

## Critical rules

- Do not rely on assumptions not supported by the code
- Do not introduce unnecessary abstractions
- Do not hide uncertainty — explicitly state it
- Prefer simple, robust improvements over complex designs
- Optimize for long-term maintainability and safe evolution
```

## 大規模コードベース

全体を把握しきれない大規模システム向けです。最大の違いは「スコープを限定してから作業を始める」ことです。基本版は理解してから直しますが、こちらは理解できる範囲を先に決めてから直します。影響分析とチェックポイントを重視しています。

**向いている場面:**
- 触ったことのないレガシーコードの改修
- 変更の影響範囲が事前に読めない
- 複数チーム・複数モジュールにまたがるシステム

```md
Refactor this large-scale codebase with a focus on safety, controlled impact, and long-term maintainability.

Assume that:

- the codebase is too large to fully understand at once
- hidden dependencies exist
- small changes may have wide and non-obvious effects

Your goal is not global redesign. Your goal is safe, incremental improvement with explicit control over impact.

---

## 1. Scope before action

- Do NOT attempt to refactor the entire system.
- Identify a **narrow, well-defined target area**:
    - a module
    - a subsystem
    - or a specific responsibility

Define clearly:

- what is inside the scope
- what is outside the scope

Do not proceed without a bounded scope.

---

## 2. Build a local understanding

Within the selected scope:

- Map:
    - entry points
    - call graph (who calls what)
    - data flow
- Identify:
    - dependencies (incoming and outgoing)
    - shared state
    - external side effects

Explicitly list:

- what this part depends on
- what depends on this part

---

## 3. Establish a safety net (local, not global)

- Identify invariants for this scope
- Write or verify tests that protect:
    - core behavior
    - critical edge cases
- Prefer:
    - property tests if available
    - otherwise focused unit/integration tests

Ensure:

- failures are loud (no silent passes)
- important scenarios are actually covered

Do NOT refactor without a safety net for this scope.

---

## 4. Analyze impact before changing anything

Before any change, explicitly answer:

- What modules might be affected?
- What assumptions might break?
- Are there implicit contracts (not enforced by types)?

Classify impact:

- local (safe)
- cross-module (medium risk)
- system-wide (high risk)

Avoid high-risk changes unless explicitly justified.

---

## 5. Refactor incrementally with checkpoints

- Make **small, reversible changes**
- After each step:
    - run tests
    - verify behavior
    - confirm no unexpected breakage

Prefer:

- extraction over rewriting
- isolation over redesign
- simplification over abstraction

Avoid:

- large rewrites
- multi-module simultaneous changes
- "cleanup everything" approaches

---

## 6. Reduce coupling safely

Focus on:

- removing unnecessary dependencies
- making interfaces more explicit
- isolating side effects

But:

- do NOT break existing contracts
- do NOT introduce new abstractions unless clearly beneficial

If coupling cannot be reduced safely, document why.

---

## 7. Preserve non-functional properties

Verify that changes do not degrade:

- performance (especially hot paths)
- memory usage
- concurrency behavior
- error handling
- observability (logs, debugging)

If any risk exists:

- explicitly identify it
- measure or estimate impact
- justify the change

---

## 8. Respect system constraints

- Do not assume missing abstractions can be added easily
- Verify language/framework limitations before designing changes
- If the system resists a change, prefer adapting to it over forcing a redesign

---

## 9. Document every decision

For each change, record:

- original problem
- change made
- affected scope
- risk level
- why it is safe

Also document:

- what was intentionally NOT changed
- known limitations left in place

---

## 10. Stop early if necessary

Do not over-refactor.

If:

- risk increases
- understanding is insufficient
- impact becomes unclear

Then:

- stop
- report findings
- suggest next safe steps

---

## Output format

1. Scope definition
2. Local architecture understanding
3. Dependency and impact analysis
4. Identified problems (within scope only)
5. Refactoring plan (incremental, low-risk)
6. Step-by-step changes with checkpoints
7. Functional validation
8. Non-functional impact analysis
9. Remaining risks and constraints
10. Recommended next steps

---

## Critical rules

- Never assume global understanding
- Never make large implicit changes
- Always make impact explicit
- Prefer safety over elegance
- Prefer clarity over cleverness
```

## 設計再構築

他の2つがコードの構造を変えつつ動作を保つリファクタリングであるのに対し、このプロンプトはアーキテクチャそのものを再設計します。単なるコード整理ではなく、設計上の問題を根本から解決するためのプロンプトです。

**向いている場面:**
- 機能追加のたびに複数箇所を修正しなければならない
- レイヤー間の責務が混在して変更の影響が広がる
- 依存方向が逆転している、または循環している

```md
Redesign the architecture of this project to improve long-term maintainability, clarify responsibility boundaries, and enable safer future change.

This is an architecture restructuring task, not a cosmetic refactoring task.

Your goal is to identify where the current architecture no longer supports the needs of the system, and to propose a better structure with clear reasoning, controlled impact, and explicit trade-offs.

---

## 0. Identify change pressure (mandatory starting point)

Before any architectural work, identify the real forces driving change.

Examples include:

- difficulty adding new features
- frequent breakage when modifying code
- performance or scalability limits
- unclear ownership of logic or state
- team friction or coordination issues

Do NOT propose architectural changes unless they are directly linked to concrete change pressures observed in the system.

---

## 1. Diagnose the current architecture

Identify:

- major subsystems
- responsibility of each subsystem
- dependency directions between subsystems
- ownership of state, behavior, and side effects
- architectural bottlenecks and failure points

Look specifically for:

- mixed layers (domain, orchestration, infrastructure, UI)
- hidden or implicit coupling
- circular or unstable dependencies
- shared mutable state with unclear ownership
- modules acting as "gravity wells" (too many responsibilities)
- porous or inconsistent boundaries

Do not redesign before diagnosing.

---

## 2. Identify architectural problems (not code smells)

Distinguish clearly between:

- local code issues
- system-level architectural problems

Focus only on architectural issues such as:

- poor separation of concerns
- unstable or incorrect dependency direction
- inability to change one area without affecting many others
- infrastructure concerns leaking into core logic
- business logic scattered across layers
- duplicated concepts represented inconsistently
- unclear ownership of data lifecycle or transformations

Only include problems that affect system design and evolution.

---

## 3. Define the target architecture

Describe the target structure in terms of:

- subsystems or layers
- clear responsibility boundaries
- dependency rules (what depends on what, and what must not)
- ownership of state and behavior
- interaction patterns between components

Ensure:

- dependency direction is intentional and stable
- higher-level policy does not depend on lower-level implementation details
- core/domain logic is isolated from side effects

Use architectural patterns only if justified by the codebase.

Do not introduce structure without evidence.

---

## 4. Design for changeability, not elegance

Optimize the architecture for:

- safe and localized future changes
- reduced blast radius
- lower cognitive load
- clearer ownership
- testability

Avoid:

- unnecessary layers
- speculative abstractions
- pattern-driven design without real need
- "architecture for its own sake"

Every design decision must trace back to a real problem.

---

## 5. Preserve functional and non-functional requirements

Ensure that redesign preserves or improves:

### Functional correctness

- existing behavior
- invariants
- external contracts

### Non-functional qualities

- performance
- memory usage
- concurrency safety
- error handling
- observability (logging, debugging, tracing)
- operability
- security
- API compatibility

If any of these may degrade:

- explicitly identify the risk
- explain the trade-off
- justify the decision or avoid the change

---

## 6. Plan migration (not just destination)

Do not stop at the ideal architecture.

Define a realistic migration strategy:

- break changes into stages
- prefer reversible, low-risk steps
- isolate transition seams
- define what can be migrated independently
- introduce temporary compatibility layers if necessary

For each stage:

- what changes
- what stays the same
- how correctness is verified
- how risk is controlled

Prefer incremental transformation over replacement.

Do NOT propose a full rewrite unless it is clearly unavoidable.

---

## 7. Define verification and observability

Ensure architectural changes can be validated.

Define:

- invariants to preserve
- required tests
- integration boundaries to verify
- contracts to enforce
- performance or operational checks if needed

Do not introduce architecture that cannot be observed, tested, or verified.

Avoid invisible abstractions or implicit flows.

---

## 8. Respect real-world constraints

Account for:

- language and type system limitations
- framework constraints
- team capabilities
- operational and deployment constraints
- backward compatibility requirements

Do not assume a greenfield rewrite.

Prefer architecture that can emerge from the current system through controlled change.

---

## 9. Make trade-offs explicit

For each major decision, explain:

- what problem it solves
- what it improves
- what complexity or cost it introduces
- alternatives considered
- why this choice is appropriate here

Clearly separate:

- confirmed observations
- design judgments
- uncertainties

Do not present preferences as facts.

---

## 10. Control scope explicitly

Define:

- what is included in the redesign
- what is explicitly excluded

Do NOT expand scope implicitly.

Explicitly identify areas that are intentionally left unchanged.

---

## 11. Stop where understanding ends

If parts of the system are not sufficiently understood:

- state the uncertainty explicitly
- propose investigation steps
- avoid speculative redesign

Prefer partial but correct restructuring over incorrect global redesign.

---

## Output format

1. Change pressures driving redesign
2. Current architecture diagnosis
3. Architectural problems
4. Target architecture
5. Dependency and boundary rules
6. Migration strategy (staged)
7. Verification and observability plan
8. Functional and non-functional risk analysis
9. Trade-offs and alternatives
10. Scope definition (included / excluded)
11. Constraints and unknowns
12. Recommended next steps

---

## Critical rules

- Do not redesign without a clear change pressure
- Do not assume full system understanding
- Do not introduce untestable or invisible architecture
- Do not expand scope implicitly
- Prefer migration over replacement
- Prefer clarity over cleverness
- Prefer evidence over assumption
```
