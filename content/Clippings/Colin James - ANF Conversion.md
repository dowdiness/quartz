---
title: "Colin James - ANF Conversion"
source: "https://compiler.club/anf-conversion/"
author:
  - Colin James
description: anf-conversion implementation clipping
tags: ["clippings"]
aliases: [ANF Conversion]
created: 2025-04-19T16:44:51+09:00
modified: 2025-12-23T11:17:42+09:00
---

# ANF Conversion

A common intermediate representation used in compilers is that of [A-Normal Form](https://en.wikipedia.org/wiki/A-normal_form). The key idea of this representation is that intermediary values produced by expressions become bound to variables. The effective result of the transformation is a linearisation of complex expressions which can help in subsequent lowering passes (such as to a lower, three-address code, scheme). Example

As an example, consider the following expression:

```txt
foo(6 + 8, 10) * bar(2, id(12))
```

Its ANF conversion may yield the following:

```txt
let t0 = 6 + 8 in
let t1 = foo(t0, 10) in
let t2 = id(12) in
let t3 = bar(2, t2) in
let t4 = t1 * t3 in
t4
```

There are important points to note about the above:

- Evaluation order becomes explicit as complex expressions are linearised into their constituent parts.
- Arguments to functions become “trivial” - their evaluation halts immediately (in our case, arguments are either variables or integer literals).


## The Algorithm

The first thing we require is a suitable datatype representation to work with:

```ocaml
(** binary operator *)
type bop = Add | Sub | Mul

(** arithmetic expressions with calls *)
type expr =
  | Lit of int
  | Bop of bop * expr * expr
  | Call of string * expr list

(** "trivial" values/atoms *)
type value =
  | Var of string
  | Int of int

(** type alias *)
type var = string

(** ANF/let expressions *)
type lexpr =
  | Halt of value
  | LetBop of var * bop * value * value * lexpr
  | LetCall of var * string * value list * lexpr
```

We will also need a small utility for producing fresh names:

```ocaml
(** fresh name with given prefix *)
let fresh =
  let c = ref (-1) in
  fun p ->
  incr c;
  p ^ string_of_int !c
```

In order to choose how we implement the conversion, it’s important to notice that a simple, recursive, bottom-up conversion will not suffice. It’s clear that each complex expression is bound to a fresh variable. However, where that variable is used must be nested deeper within the result.

To illustrate this, consider the literate algebraic data type representation of the example from above:

```ocaml
LetBop ("t0", Add, Lit 6, Lit 8,
  LetCall ("t1", "foo", [Var "t0"; Lit 10],
    LetCall ("t2", "id", [Lit 12],
      LetCall ("t3", "bar", [Lit 2; Var "t2"],
        LetBop ("t4", Mul, Var "t1", Var "t3", Halt "t4")))))
```

The computation of `t4` uses variables `t1` and `t3`, which are constructed/bound at a shallower level in the structure.

In order to build this bottom-up (yet outside-in), it is convenient to embrace the functional utility of closures for building up data structures. Effectively, we must recursively propagate an embedding context which tells deeper recursive invocations how to use the values each expression yields.

We implement this idea by means of providing an embedding function (continuation) of type `value -> lexpr` as supplementary argument to our primary conversion function, `go`:

```ocaml
let rec go e k = match e with
  | ...
```
- In the case of integer literals, we simply repackage up the integer value as a value and pass it off to the continuation:
```ocaml
| Lit i -> k (Int i)
```
- In the case of binary arithmetic expressions, `Bop (op, l, r)`, we must collect each intermediary value through nested continuations then propagate the bound variable value to the outermost continuation:
```ocaml
| Bop (op, l, r) ->
    go l (fun l -> go r (fun r -> let t = fresh "t" in LetBop (t, op, l, r, k (Var t))))
```

However, this isn’t very readable so we’ll overload `let*` to allow us to write this continuation code in a more direct style:

```ocaml
| Bop (op, l, r) ->
    let ( let* ) = (@@) in
    let* l = go l in
    let* r = go r in
    let t = fresh "t" in
    LetBop (t, op, l, r, k (Var t))
```

The most complex case is that of calls, `Call (f, es)`. In the previous example, we could just propagate each resultant value as the formal parameter of the continuation provided to go. That works fine for cases where we know how many values we must bind. In the case of calls, however, where the number of arguments could be arbitrary, we must construct a larger embedding context that collects each resultant value into a list.

In order to do this, we will construct an $n$ -ary embedding context of type value `list -> lexp` r and accumulate this closure by folding around a base context:

```ocaml
| Call (f, es) ->
    let accumulate e ctx vs =
      go e (fun v -> ctx (v :: vs))
    in
    let base vs =
      let t = fresh "t" in
      LetCall (t, f, List.rev vs, k (Var t))
    in
    List.fold_right accumulate es base []
```

The reversal of `vs` in the base context ensures left-to-right order is preserved after the arguments are collected in reverse (to avoid use of append for each value, which is a linear time operation).

Finally, we’ll wrap go up inside a function, `anf`, that invokes go with a top-level continuation that produces halt of the provided value:

```ocaml
let anf =
  let rec go e k = match e with
    | Lit i -> k (Int i)
    | Bop (op, l, r) ->
       let ( let* ) = (@@) in
       let* l = go l in
       let* r = go r in
       let t = fresh "t" in
       LetBop (t, op, l, r, k (Var t))
    | Call (f, es) ->
       let accumulate e ctx vs =
         go e (fun v -> ctx (v :: vs))
       in
       let base vs =
         let t = fresh "t" in
         LetCall (t, f, List.rev vs, k (Var t))
       in
       List.fold_right accumulate es base []
  in
  Fun.flip go (fun v -> Halt v)
```


## Final Program

Below, you can play around with this simple conversion algorithm by providing arithmetic expressions as input:

Enter arithmetic expression:

```txt
let t0 = 4 * -7 in
let t1 = bar(t0) in
let t2 = id(12) in
let t3 = 0 - t2 in
let t4 = foo(t1, 3, t3) in
let t5 = 1 + t4 in
t5
```
