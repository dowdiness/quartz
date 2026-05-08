---
title: Finding Common Structures
publish: true
tags: [ai]
aliases: [Finding Common Structures]
created: 2026-01-04T20:50:52+09:00
modified: 2026-05-09T00:15:36+09:00
---

# Finding Common Structures

Finding shared structure across different things enables design judgments that go beyond understanding each thing in isolation. Below are five activities for finding common structures.

## 1. Restate in the same vocabulary

When explaining different things, you naturally use different vocabulary for each. Try restating those explanations in the same vocabulary.

If the restatement is natural, the structure is genuinely shared. If forced abstraction is required, it isn't. The vocabulary that emerges from the restatement becomes the name of the underlying principle.

Example: "register an event listener to receive notifications" and "poll the database for changes and retrieve the diff" can both be restated as "register dependents with the source of change and propagate changes."

## 2. Ask what you have to discard for them to become the same

When two things are similar but different, instead of enumerating the differences, ask: "what do I have to discard for them to become the same?"

The less you have to discard, the deeper the common structure. If discarding only implementation details makes them identical, the common structure is essential. If nothing you discard makes them identical, the similarity is superficial.

This is the criterion that distinguishes essential common structure from incidental resemblance.

## 3. Turn differences into named axes

Don't stop at enumerating "how do A and B differ." Name the difference and turn it into an axis.

When two axes appear, a lattice forms. When a lattice forms, blank cells become visible. Blank cells generate the question "what would this combination look like?" Filling the blanks sometimes reveals structures you didn't know existed.

## 4. Trace effects back to their cause

When multiple properties or benefits hold simultaneously, instead of counting them individually, ask: "why do they all hold at once?"

If the answer converges on a single cause, that cause is the deep structural property. When multiple consequences necessarily follow from a single choice, that choice has power beyond any individual consequence.

The reverse direction also works. Considering what the negation of that cause would demand sometimes reveals a structure where a single uncertainty generates several costs.

## 5. Ask under what conditions it holds

After finding a common structure, ask "why does this hold?" and "when does it stop holding?"

Knowing the conditions clarifies the principle's range of application. Knowing the boundary of failure clarifies the principle's limits. Knowing both lets you use the principle safely.