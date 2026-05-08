---
title: Software Design Principles
publish: true
tags: [ai]
aliases: [Software Design Principles]
created: 2026-01-04T20:50:52+09:00
modified: 2026-05-09T00:15:15+09:00
---

# Software Design Principles

## 1. Problem first, then solution

When you know the theory behind a data structure or technique, you start looking for places to apply it. The correct order is the opposite: "what needs to be possible" and "why isn't it possible now" come first, and the choice of solution follows.

A technically superior data structure sometimes provides the same capability as the existing mechanism. When the justification lies in design intent (e.g., preserving the semantics of operations) rather than technical properties (e.g., asymptotic complexity), make the design intent explicit first.

## 2. Question binary framings

When you see a framing like "if A, this technique isn't needed; if B, it is," question the framing itself. A design that achieves both A and B often exists, and prior projects may already have built it.

Binary framings tend to appear when the solution space is only partially seen. Widening the frame reveals designs that neither of the original options contained.

## 3. Map existing responsibilities before designing new ones

When designing a new component, first understand what is processed where in the existing codebase. Reading documentation is not enough — you need to understand each function's responsibility and boundary at the implementation level.

Skipping this step leads to writing logic in the new component that already exists in some function. The correct division is usually that the new component describes *what* to do and delegates *how* to existing functions.

## 4. Once a design is written, try to break it

A design at the moment it is written is not "finished" — it is "ready for review." Problems invisible during writing become visible when you attack it.

Things to check:
- Does this data survive persistence and communication round-trips?
- Does this type align with the types in other components?
- Does this operation lose data as a side effect?
- Does this function already exist somewhere?
- Does this assumption (one-to-one correspondence, matching order, etc.) actually hold?

## 5. Represent operations as data

A function call vanishes the moment it happens. Operations represented as enum variants or records persist. Persistence enables:

- undo becomes "the inverse of the operation" rather than "the inverse of the effect"
- logs record "what was done" rather than "what changed"
- collaborative editing can detect semantic conflicts between operations
- UIs can auto-generate menus from operation values

"Can this operation be represented as a value?" is worth asking early in design.

## 6. A framework's generality is determined by what it excludes

The criterion for placing a component inside or outside the framework is: "would the same type be usable in another use case?" If not, keep it out of the framework and place it externally as a per-use-case wrapper.

Keeping a framework general does not mean including many features — it means eliminating dependencies on specific use cases.

## 7. Distinguish between "deferring" and "reserving space"

When deferring a feature, there are two options:

- Use a temporary substitute and don't think about the future
- Don't implement the feature, but reserve an extension point in the data type

The first requires a breaking change when the feature is eventually added. The second only requires adding a field. The cost of reserving an extension point is far lower than the cost of a later breaking change.

## 8. Design docs and implementation docs answer different questions

Design documents answer "what to build, and why this way." Implementation documents answer "how to build it, and in what order."

Measuring design completeness by "is it implementable" lets implementation details bleed into the design document, blurring its actual purposes — recording judgments and building consensus. Conversely, mixing design decisions into implementation documents buries the procedure under explanations of motivation. Keep them separate.