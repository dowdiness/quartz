---
title: Bidirectional Typing
created: 2025-04-11T14:19:09+09:00
modified: 2026-01-15T15:38:39+09:00
tags: [compiler, type-system]
aliases: [Bidirectional Type, 双方向型検査]
---

# Bidirectional Typing

[双方向型検査: 検査と構築の融合](https://mizunashi-mana.github.io/blog/posts/2023/02/bidirectional-typing/)

[Bidirectional Typing](https://dl.acm.org/doi/10.1145/3450952)

[A Hands-On Introduction to Bidirectional Type Inference with Elm](https://confengine.com/conferences/functional-conf-2025/proposal/21310/a-hands-on-introduction-to-bidirectional-type-inference-with-elm)

[Incremental Bidirectional Typing via Order Maintenance](https://arxiv.org/abs/2504.08946)

## Polarity

[Polarity and bidirectional typechecking](https://semantic-domain.blogspot.com/2018/08/polarity-and-bidirectional-typechecking.html)
[[Polarity]] とBidirectional Typing の関係について

---

## 1. 標準的な双方向型システム (STLC)

$$\begin{array}{cc} \dfrac{\Gamma, x:A \vdash e \Leftarrow B}{\Gamma \vdash \lambda x. e \Leftarrow A \to B} \text{ (Intro-}\to) & \dfrac{\Gamma \vdash t \Rightarrow A \to B \quad \Gamma \vdash e \Leftarrow A}{\Gamma \vdash t e \Rightarrow B} \text{ (Elim-}\to) \\ \\ \dfrac{x:A \in \Gamma}{\Gamma \vdash x \Rightarrow A} \text{ (Var)} & \dfrac{\Gamma \vdash e \Leftarrow A}{\Gamma \vdash (e : A) \Rightarrow A} \text{ (Anno)} \end{array}$$

---

## 2. 極性付き型理論 (Polarized Type Theory)


### 値の型付け (Values: $\Gamma \vdash v \Leftarrow P$)

$$\begin{array}{cc} \dfrac{}{\Gamma \vdash () \Leftarrow 1} \text{ (Unit)} & \dfrac{\Gamma \vdash v \Leftarrow P \quad \Gamma \vdash v' \Leftarrow Q}{\Gamma \vdash (v, v') \Leftarrow P \times Q} \text{ (Pair)} \\ \\ \dfrac{\Gamma \vdash v \Leftarrow P_i \quad i \in \{1, 2\}}{\Gamma \vdash \mathsf{in}_i(v) \Leftarrow P_1 + P_2} \text{ (Sum)} & \dfrac{\Gamma \rhd t \Leftarrow N}{\Gamma \vdash \{t\} \Leftarrow \downarrow N} \text{ (Down)} \end{array}$$

### Spinesの型付け (Spines: $\Gamma \vdash s : N \gg M$)

$$\begin{array}{c} \dfrac{}{\Gamma \vdash \cdot : N \gg N} \text{ (Nil)} \\ \\ \dfrac{\Gamma \vdash v \Leftarrow P \quad \Gamma \vdash s : N \gg M}{\Gamma \vdash v s : P \to N \gg M} \text{ (Cons)} \end{array}$$

### 項の型付け (Terms: $\Gamma \rhd t \Leftarrow N$)

$$\begin{array}{c} \dfrac{\Gamma \vdash v \Leftarrow P}{\Gamma \rhd \mathsf{return}\, v \Leftarrow \uparrow P} \text{ (Return)} \\ \\ \dfrac{\forall i < n. p_i : P \rightsquigarrow \Delta_i \quad \Gamma, \Delta_i \rhd t_i \Leftarrow N}{\Gamma \rhd \lambda (p_i \to t_i)_{i < n} \Leftarrow P \to N} \text{ (Abs)} \\ \\ \dfrac{x:M \in \Gamma \quad \Gamma \vdash s : M \gg \uparrow Q \quad \forall i < n. p_i : Q \rightsquigarrow \Delta_i \quad \Gamma, \Delta_i \rhd t_i \Leftarrow \uparrow P}{\Gamma \rhd \mathsf{match}\, x \cdot s \text{ of } [p_i \to t_i]_{i < n} \Leftarrow \uparrow P} \text{ (Match)} \end{array}$$

---

## Local Type Inference

Bidirectional Typing と関係の深い型推論。UnificationではなくMatchingを使う。

[Local type inference](https://dl.acm.org/doi/10.1145/345099.345100)

[Simple Type Inference for System F](https://semantic-domain.blogspot.com/2022/03/simple-type-inference-for-system-f.html)
 
[Local Contextual Type Inference](https://dl.acm.org/doi/10.1145/3776653)