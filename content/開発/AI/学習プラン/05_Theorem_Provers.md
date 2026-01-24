# Part 5: 定理証明支援系

## 📚 この章について

定理証明支援系を使って、ニューラルネットワークとコンパイラ最適化を**形式的に検証**します。理論だけでなく、実用的な検証技術を学びます。

**学習時間**: 6-8週間  
**前提知識**: Part 1-2, プログラミング経験

## 🎯 学習目標

- [ ] Coq で基本的な証明ができる
- [ ] Lean 4 でプログラムを検証できる
- [ ] CompCert の意味保存を理解できる
- [ ] NN検証ツール (α,β-CROWN) を使える
- [ ] 検証済みコンパイラ最適化を実装できる

---

## 1. Coq入門

### 1.1 基本的な証明

```coq
(* 自然数の定義 *)
Inductive nat : Type :=
  | O : nat
  | S : nat -> nat.

(* 加算の定義 *)
Fixpoint plus (n m : nat) : nat :=
  match n with
  | O => m
  | S n' => S (plus n' m)
  end.

(* 結合律の証明 *)
Theorem plus_assoc : forall n m p : nat,
  plus (plus n m) p = plus n (plus m p).
Proof.
  intros n m p.
  induction n as [| n' IHn'].
  - (* n = O *)
    simpl. reflexivity.
  - (* n = S n' *)
    simpl. rewrite IHn'. reflexivity.
Qed.
```

### 1.2 リストと高階関数

```coq
(* リスト *)
Inductive list (A : Type) : Type :=
  | nil : list A
  | cons : A -> list A -> list A.

(* map関数 *)
Fixpoint map {A B : Type} (f : A -> B) (l : list A) : list B :=
  match l with
  | nil _ => nil B
  | cons _ h t => cons B (f h) (map f t)
  end.

(* map の正しさ *)
Theorem map_length : forall (A B : Type) (f : A -> B) (l : list A),
  length (map f l) = length l.
Proof.
  intros A B f l.
  induction l as [| h t IH].
  - (* l = nil *)
    simpl. reflexivity.
  - (* l = cons h t *)
    simpl. rewrite IH. reflexivity.
Qed.
```

### 1.3 演習問題

**Q1.1**: `plus` の交換律を証明せよ

**Q1.2**: `filter` 関数を定義し、`length (filter f l) <= length l` を証明せよ

---

## 2. ニューラルネットワークの検証

### 2.1 層の定義と性質

```coq
Require Import Reals.
Require Import Vector.

(* ベクトル型 *)
Definition Vec (n : nat) := t R n.

(* 行列型 *)
Definition Matrix (m n : nat) := Vec m -> Vec n.

(* 線形層 *)
Record LinearLayer (input output : nat) := {
  weights : Matrix input output;
  bias : Vec output;
}.

(* 順伝播 *)
Definition forward {n m : nat} (layer : LinearLayer n m) (x : Vec n) : Vec m :=
  map2 Rplus (weights layer x) (bias layer).

(* 合成の結合律 *)
Theorem layer_compose_assoc : forall {a b c d : nat}
  (l1 : LinearLayer a b) (l2 : LinearLayer b c) (l3 : LinearLayer c d),
  compose_layers (compose_layers l1 l2) l3 =
  compose_layers l1 (compose_layers l2 l3).
Proof.
  intros.
  unfold compose_layers.
  apply functional_extensionality.
  intro x.
  reflexivity.
Qed.
```

### 2.2 敵対的頑健性の検証

```coq
(* ε-ball *)
Definition epsilon_ball {n : nat} (x : Vec n) (eps : R) : Vec n -> Prop :=
  fun x' => norm (vminus x' x) <= eps.

(* 頑健性の定義 *)
Definition robust {n m : nat} (network : Vec n -> Vec m) 
  (x : Vec n) (eps : R) : Prop :=
  forall x' : Vec n,
  epsilon_ball x eps x' ->
  argmax (network x) = argmax (network x').

(* 抽象解釈による検証 *)
Inductive Interval : Type :=
  | IVal : R -> R -> Interval.  (* [lower, upper] *)

(* 抽象的な順伝播 *)
Fixpoint abstract_forward {n m : nat} 
  (layers : list (LinearLayer n m)) (input : Interval) : Interval :=
  match layers with
  | nil => input
  | l :: rest =>
      let abs_affine := abstract_affine_transform l input in
      let abs_relu := abstract_relu abs_affine in
      abstract_forward rest abs_relu
  end.

(* 健全性定理 *)
Theorem abstract_forward_sound : forall network x abs,
  in_interval x abs ->
  in_interval (network x) (abstract_forward network abs).
Proof.
  (* 抽象解釈の健全性を証明 *)
  admit.
Admitted.
```

### 2.3 演習問題

**Q2.1**: ReLU の抽象実行 `abstract_relu` を実装せよ

**Q2.2**: 2層ネットワークの頑健性を Coq で証明せよ（小さな例で）

---

## 3. CompCert: 検証済みコンパイラ

### 3.1 意味保存の定義

```coq
(* プログラムの振る舞い *)
Inductive behavior :=
  | Terminates : trace -> int -> behavior
  | Diverges : trace -> behavior
  | Goes_wrong : trace -> behavior.

(* Forward simulation *)
Record forward_simulation (L1 L2: semantics) := {
  index: Type;
  order: index -> index -> Prop;
  match_states: index -> state L1 -> state L2 -> Prop;
  
  (* 初期状態の対応 *)
  match_initial_states:
    forall s1, initial_state L1 s1 ->
    exists i s2, initial_state L2 s2 /\ match_states i s1 s2;
  
  (* ステップの対応 *)
  simulation:
    forall i s1 t s1',
    step L1 s1 t s1' ->
    forall s2, match_states i s1 s2 ->
    exists i' s2',
      (plus step L2 s2 t s2' \/ (star step L2 s2 t s2' /\ order i' i))
      /\ match_states i' s1' s2';
}.

(* 意味保存定理 *)
Theorem forward_simulation_behavior_improves:
  forall L1 L2, forward_simulation L1 L2 ->
  forall beh1, program_behaves L1 beh1 ->
  exists beh2, program_behaves L2 beh2 /\ behavior_improves beh1 beh2.
```

### 3.2 最適化パスの検証

```coq
(* 定数畳み込み *)
Fixpoint const_fold (e : expr) : expr :=
  match e with
  | Econst n => Econst n
  | Evar v => Evar v
  | Eadd (Econst n1) (Econst n2) =>
      Econst (n1 + n2)  (* 畳み込み *)
  | Eadd e1 e2 =>
      Eadd (const_fold e1) (const_fold e2)
  | _ => e
  end.

(* 正しさの証明 *)
Lemma const_fold_correct : forall e env,
  eval_expr env (const_fold e) = eval_expr env e.
Proof.
  induction e; intros; simpl.
  - (* Econst *) reflexivity.
  - (* Evar *) reflexivity.
  - (* Eadd *)
    destruct e1, e2; simpl;
    try (rewrite IHe1, IHe2; reflexivity).
    + (* 両方定数 *)
      simpl. reflexivity.
Qed.
```

### 3.3 演習問題

**Q3.1**: 強度軽減 (x * 2 → x + x) を実装し、正しさを証明せよ

**Q3.2**: 2つの最適化パスの合成が意味保存であることを証明せよ

---

## 4. Lean 4: 現代的なアプローチ

### 4.1 型安全なニューラルネットワーク

```lean
-- 型レベルで次元を追跡
def Vec (α : Type) (n : Nat) := { l : List α // l.length = n }

-- 行列
def Matrix (α : Type) (m n : Nat) := Vec (Vec α n) m

-- 層の定義
structure Layer (α : Type) (input output : Nat) where
  weights : Matrix α input output
  bias : Vec α output

-- 型安全な合成
def Layer.compose {α : Type} {a b c : Nat}
  (l1 : Layer α a b) (l2 : Layer α b c) : Layer α a c :=
  { weights := sorry,  -- 行列積
    bias := l2.bias }

-- 型が次元を保証
example : Layer Float 784 128 → Layer Float 128 10 → Layer Float 784 10 :=
  Layer.compose

-- これはコンパイルエラー（次元不一致）
-- example : Layer Float 784 128 → Layer Float 64 10 → Layer Float 784 10 :=
--   Layer.compose
```

### 4.2 Tactics による証明

```lean
-- 合成の結合律
theorem Layer.compose_assoc {α : Type} {a b c d : Nat}
  (l1 : Layer α a b) (l2 : Layer α b c) (l3 : Layer α c d) :
  (l1.compose l2).compose l3 = l1.compose (l2.compose l3) := by
  unfold Layer.compose
  simp
  rfl

-- 畳み込みの正しさ
theorem conv_output_size (input_size kernel_size stride : Nat) :
  (input_size - kernel_size) / stride + 1 > 0 := by
  omega  -- 自動で数値的制約を解決
```

### 4.3 演習問題

**Q4.1**: ResNet の Skip Connection を Lean で型安全に実装せよ

**Q4.2**: Batch Normalization の性質を Lean で証明せよ

---

## 5. Dafny: 実用的な検証

### 5.1 コンパイラ最適化の検証

```dafny
// 式の定義
datatype Expr =
  | Const(n: int)
  | Var(x: string)
  | Add(e1: Expr, e2: Expr)
  | Mul(e1: Expr, e2: Expr)

// 評価関数
function eval(e: Expr, env: map<string, int>): int
  requires forall x :: Var(x) in e ==> x in env
{
  match e
  case Const(n) => n
  case Var(x) => env[x]
  case Add(e1, e2) => eval(e1, env) + eval(e2, env)
  case Mul(e1, e2) => eval(e1, env) * eval(e2, env)
}

// 定数畳み込み
function constFold(e: Expr): Expr
{
  match e
  case Add(Const(n1), Const(n2)) => Const(n1 + n2)
  case Mul(Const(n1), Const(n2)) => Const(n1 * n2)
  case Add(e1, e2) => Add(constFold(e1), constFold(e2))
  case Mul(e1, e2) => Mul(constFold(e1), constFold(e2))
  case _ => e
}

// 正しさの証明（Dafnyが自動証明）
lemma constFoldCorrect(e: Expr, env: map<string, int>)
  requires forall x :: Var(x) in e ==> x in env
  ensures eval(constFold(e), env) == eval(e, env)
{
  // Dafny が自動で証明
}
```

### 5.2 ループ最適化

```dafny
// ループ展開
method unrollLoop(n: nat, body: int -> int) returns (r: int)
  requires n > 0
  ensures r == applyNTimes(body, 0, n)
{
  r := 0;
  var i := 0;
  
  while i < n
    invariant 0 <= i <= n
    invariant r == applyNTimes(body, 0, i)
    decreases n - i
  {
    r := body(r);
    i := i + 1;
  }
}

// n回適用
function applyNTimes(f: int -> int, x: int, n: nat): int
{
  if n == 0 then x
  else applyNTimes(f, f(x), n - 1)
}
```

### 5.3 演習問題

**Q5.1**: デッドコード除去を Dafny で実装し検証せよ

**Q5.2**: ループ融合の正しさを Dafny で証明せよ

---

## 6. NN検証ツール

### 6.1 α,β-CROWN の使用

```python
# α,β-CROWNで頑健性検証
from auto_LiRPA import BoundedModule, BoundedTensor
from auto_LiRPA.perturbations import PerturbationLpNorm

# モデル定義
model = SimpleNN()

# 検証用のラッパー
lirpa_model = BoundedModule(model, torch.zeros(1, 784))

# 摂動の定義
ptb = PerturbationLpNorm(norm=np.inf, eps=0.01)

# 入力の範囲
x = BoundedTensor(input_data, ptb)

# 頑健性検証
lb, ub = lirpa_model.compute_bounds(x=(x,), method='CROWN')

# 結果
if lb.argmax() == ub.argmax():
    print("頑健性が証明された")
else:
    print("頑健性は保証できない")
```

### 6.2 SMTソルバとの統合

```python
from z3 import *

# ニューラルネットワークをSMT制約として符号化
def nn_to_smt(weights, biases, input_var):
    """
    NNをZ3の制約として表現
    """
    constraints = []
    
    x = [Real(f'x_{i}') for i in range(len(input_var))]
    
    # 入力制約
    for i, (lb, ub) in enumerate(input_var):
        constraints.append(And(x[i] >= lb, x[i] <= ub))
    
    # 層ごとの制約
    current = x
    for W, b in zip(weights, biases):
        next_layer = []
        for j in range(W.shape[0]):
            # 線形変換
            z = sum(W[j, i] * current[i] for i in range(len(current))) + b[j]
            # ReLU
            y = Real(f'y_{len(next_layer)}')
            constraints.append(y == If(z > 0, z, 0))
            next_layer.append(y)
        current = next_layer
    
    return And(constraints), current

# 頑健性検証
solver = Solver()
constraints, output = nn_to_smt(weights, biases, input_bounds)
solver.add(constraints)

# 出力クラスが変わるか確認
solver.add(output[true_class] < output[adversarial_class])

if solver.check() == unsat:
    print("頑健性が証明された")
else:
    print("反例あり:", solver.model())
```

### 6.3 演習問題

**Q6.1**: 小さなNN（2層）を α,β-CROWN で検証せよ

**Q6.2**: Z3 で最適化の等価性を検証せよ（x*2 vs x+x）

---

## 7. 統合プロジェクト: 検証済みコンパイラ最適化

### 7.1 Coqでの実装

```coq
(* 最適化パスの型 *)
Definition OptPass := program -> program.

(* 意味論 *)
Definition semantics (p : program) : trace -> Prop := ...

(* 意味保存 *)
Definition preserves_semantics (opt : OptPass) : Prop :=
  forall p t,
  semantics p t <-> semantics (opt p) t.

(* 検証済み最適化パス *)
Record VerifiedOptPass := {
  pass : OptPass;
  proof : preserves_semantics pass;
}.

(* 合成 *)
Definition compose_opt (o1 o2 : VerifiedOptPass) : VerifiedOptPass.
Proof.
  refine {|
    pass := fun p => o2.(pass) (o1.(pass) p);
    proof := _;
  |}.
  unfold preserves_semantics. intros.
  rewrite <- o2.(proof).
  rewrite <- o1.(proof).
  reflexivity.
Defined.

(* パイプライン *)
Definition pipeline : VerifiedOptPass :=
  compose_opt const_fold_pass
    (compose_opt dead_code_pass
      identity_pass).
```

### 7.2 実行可能なコード抽出

```coq
(* Coqから実行可能なOCamlコードを抽出 *)
Extraction Language OCaml.
Extract Constant string => "string".

Extraction "verified_optimizer.ml" pipeline.
```

```ocaml
(* 抽出されたOCamlコード *)
let pipeline program =
  let p1 = const_fold program in
  let p2 = dead_code_elim p1 in
  p2

(* 使用 *)
let optimized = pipeline my_program
```

### 7.3 演習問題

**Q7.1**: 完全な最適化パイプラインを Coq で実装せよ（3つ以上の最適化）

**Q7.2**: 抽出したコードをベンチマークし、性能を測定せよ

---

## 8. まとめ

### 8.1 この章で学んだこと

1. **Coq**: 関数型言語での形式検証
2. **Lean**: 現代的な証明支援系
3. **Dafny**: 実用的な検証言語
4. **CompCert**: 実世界の検証済みコンパイラ
5. **NN検証**: 敵対的頑健性の形式検証

### 8.2 実践への応用

- 検証済みMLコンパイラ
- 安全なコンパイラ最適化
- 頑健なNN
- バグのないシステム

---

## 📚 参考文献

### 必読

1. **Software Foundations** (Coq)
   - https://softwarefoundations.cis.upenn.edu/

2. **Theorem Proving in Lean 4**
   - https://leanprover.github.io/theorem_proving_in_lean4/

3. **CompCert**
   - https://compcert.org/

### ツール

- **Coq**: https://coq.inria.fr/
- **Lean 4**: https://leanprover.github.io/
- **Dafny**: https://dafny.org/
- **α,β-CROWN**: https://github.com/Verified-Intelligence/alpha-beta-CROWN

### 次のステップ

→ **06_Compiler_Optimization.md** へ進む
