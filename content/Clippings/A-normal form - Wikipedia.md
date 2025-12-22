---
title: A-normal form - Wikipedia
source: https://en.wikipedia.org/wiki/A-normal_form
author: anonymous
description: A-normal_form wiki clipping
tags: [clippings, compiler, ir]
created: 2025-04-19T16:45:12+09:00
modified: 2025-12-22T20:37:56+09:00
---

# A-normal form - Wikipedia

[[A-normal form]]

# Wiki

In [computer science](https://en.wikipedia.org/wiki/Computer_science "Computer science"), **A-normal form** (abbreviated **ANF**, sometimes expanded as **administrative normal form** or as **atomic normal form**) is an [intermediate representation](https://en.wikipedia.org/wiki/Intermediate_language "Intermediate language") of [programs](https://en.wikipedia.org/wiki/Program_\(computer_science\) "Program (computer science)") in [functional programming](https://en.wikipedia.org/wiki/Functional_programming "Functional programming") language [compilers](https://en.wikipedia.org/wiki/Compiler "Compiler"). In ANF, all [arguments](https://en.wikipedia.org/wiki/Argument_\(computer_science\) "Argument (computer science)") to a [function](https://en.wikipedia.org/wiki/Function_\(computer_science\) "Function (computer science)") must be trivial (constants or variables). That is, evaluation of each argument must halt immediately.

ANF was introduced by Sabry and [Felleisen](https://en.wikipedia.org/wiki/Matthias_Felleisen "Matthias Felleisen") in 1992 [^1] as a simpler alternative to [continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style "Continuation-passing style") (CPS). Some of the advantages of using CPS as an intermediate representation are that optimizations are easier to perform on programs in CPS than in the source language, and that it is also easier for compilers to generate [machine code](https://en.wikipedia.org/wiki/Machine_code "Machine code") for programs in CPS. Flanagan et al.[^2] showed how compilers could use ANF to achieve those same benefits with one source-level transformation; in contrast, for realistic compilers the CPS transformation typically involves additional phases, for example, to simplify CPS terms.

## Grammar

Consider the pure [λ-calculus](https://en.wikipedia.org/wiki/%CE%9B-calculus "Λ-calculus") with constants and [let-expressions](https://en.wikipedia.org/wiki/Let-expression "Let-expression"). The ANF restriction is enforced by

1. allowing only *values* (variables, constants, and λ-terms), to serve as operands of function applications, and
2. requiring that the result of a non-trivial expression (such as a function application) be immediately captured in a [let-bound](https://en.wikipedia.org/wiki/Let-bound "Let-bound") variable.

The following [BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form "Backus–Naur form") grammars show how one would modify the syntax of λ-expressions to implement the constraints of ANF:

| Original | ANF |
| --- | --- |
| ``` EXP ::= λ VAR . EXP       \| EXP EXP       \| VAR       \| CONST       \| let VAR = EXP in EXP  CONST ::= f \| g \| h ``` | ``` EXP ::= VAL        \| let VAR = VAL in EXP       \| let VAR = VAL VAL in EXP  VAL ::= VAR       \| CONST       \| λ VAR . EXP  CONST ::= f \| g \| h ``` |

Variants of ANF used in compilers or in research often allow records, tuples, multiargument functions, primitive operations and conditional expressions as well.

## Examples

The expression:

```
f(g(x),h(y))
```

is written in ANF as:

```
let v0 = g(x) in
    let v1 = h(y) in
        f(v0,v1)
```

By imagining the sort of assembly this function call would produce:

```
;; let v0 = g(x)
move x into args[0]
call g
move result into temp[0]
;; let v1 = h(y)
move y into args[0]
call h
move result into temp[1]
;; f(v0, v1)
move temp[0] into args[0]
move temp[1] into args[1]
call f
```

One can see the immediate similarities between ANF and the compiled form of a function; this property is a part of what makes ANF a good intermediate representation for optimisations in compilers.

## See also

- [Continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style "Continuation-passing style")
- [Static single assignment form](https://en.wikipedia.org/wiki/Static_single_assignment_form "Static single assignment form")

## References

[^1]: Sabry, Amr; Felleisen, Matthias. "Reasoning about Programs in Continuation-Passing Style". *Proceedings of the 1992 ACM Conference on LISP and Functional Programming, LFP'92*. San Francisco, CA, USA. [CiteSeerX](https://en.wikipedia.org/wiki/CiteSeerX_\(identifier\) "CiteSeerX (identifier)") [10.1.1.22.7509](https://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.22.7509). Sabry92.
[^2]: Flanagan, Cormac; Sabry, Amr; Duba, Bruce F.; Felleisen, Matthias. ["The Essence of Compiling with Continuations"](http://users.soe.ucsc.edu/~cormac/papers/pldi93.pdf) (PDF). *Proceedings ACM SIGPLAN 1993 Conf. on Programming Language Design and Implementation, PLDI'93*. Albuquerque, NM, USA. Flanagan93. Retrieved 2012-11-16.
