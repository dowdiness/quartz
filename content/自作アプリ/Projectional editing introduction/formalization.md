# Gradual Projectional Editorのインクリメンタル計算による形式化

## Liu (2023) "Incremental Computation: What Is the Essence?" に基づく再定式化

---

## 1. Incrementalizationの核心とEditorへの対応

Liu の定式化における核心は以下の定義である：

> プログラム *f* と入力変更操作 *⊕* が与えられたとき、*f* の *⊕* の下でのインクリメンタル版 *f'* とは、以下を満たす関数である：
>
> **f(x) = r ⟹ f'(x, y, r) = f(x ⊕ y)**
>
> すなわち、変更後の入力 x ⊕ y に対する結果を、以前の結果 r を再利用して効率的に計算する。

これは「離散領域における微分」であり、微積分の微分が無限小の変化を扱うのに対して、incrementalizationは離散的な変更操作を扱う。

### 1.1 Editorにおけるincrementalization問題の同定

Gradual Projectional Editorシステムを構成する計算は、すべてこの枠組みに落とし込める：

```
f₁: AST → MarkedAST        (Marking: 型チェック + エラー回復)
f₂: MarkedAST → TextCST    (Text Projection)
f₃: MarkedAST → VisualCST  (Visual Projection)
f₄: TextEdit → ASTOp[]     (Text Input Adapter)
f₅: GroveGraph → AST       (Decompose: グラフ → 木)
```

各 fᵢ に対して、入力変更操作 ⊕ᵢ とインクリメンタル版 fᵢ' を求めることがシステム全体の効率化問題である。

### 1.2 三つの分類とEditorアーキテクチャの関係

Liu は incremental computation を三つに分類する：

| 分類 | 定義 | Editor における対応 |
|------|------|---------------------|
| **Incremental algorithms** | 特定の関数 f₀ と特定の変更操作 ⊕₀ に対する個別のアルゴリズム | Hazel Incremental Bidirectional Typing (OOPSLA 2025) — bidirectional typingに特化 |
| **Incremental evaluation frameworks** | 汎用の評価器 eval の変更に対するインクリメンタル版 eval' | Adapton (Hammer et al. 2014)、Salsa (Rust) — 汎用的 demand-driven change propagation |
| **Derivation methods** | f と ⊕ から f' を体系的に導出する手法 | **Liu の III method** — 本文書の中核 |

Kojiさんのシステムでは、三つすべてを組み合わせる：

- **Phase 1 (Marking)**: incremental algorithmとして Hazel の手法を直接実装
- **Phase 2 (Grove CmRDT)**: CRDTの可換操作が ⊕ の役割を果たす → incremental algorithm
- **将来のDSL汎化**: derivation method (III) で、DSL定義から各 fᵢ' を体系的に導出

---

## 2. Systematic Incrementalizationの三段階

Liu の incrementalization は三つの問題を段階的に解く：

### P1: 前回の結果の利用

> f(x ⊕ y) を展開し、x に依存する計算と y に依存する計算を分離する。x に依存する部分が f(x) と一致すれば、それを r で置換する。

**Marking System への適用:**

```
f = mark_synth : (Ctx, Expr) → (MarkedExpr, Type)
⊕ = edit_node  : AST × ASTOp → AST（ノード1つの編集）
```

ASTノード1つが変更されたとき、mark_synth(ctx, expr ⊕ op) の大部分は mark_synth(ctx, expr) = (marked, ty) と同一である。

**P1の適用:**
変更されたノードを含むサブツリーのmarking結果のみが変わる。変更されていないサブツリーの型情報 r_subtree は前回の結果をそのまま再利用できる。

```moonbit
// 非インクリメンタル版
fn mark_synth(ctx : Ctx, expr : Expr) -> (MarkedExpr, Type)

// P1 適用後: 前回結果の再利用
fn mark_synth_incr(
  ctx : Ctx,
  expr : Expr,        // 変更後のAST
  op : ASTOp,         // 変更操作
  prev : MarkedExpr,  // 前回のMarking結果(r)
  prev_ty : Type      // 前回の型(rの一部)
) -> (MarkedExpr, Type)
// 変更されたノードのパスに沿ってのみ再計算
```

### P2: 中間結果のキャッシュ

> f(x) の計算過程で生じる中間値もキャッシュし、f(x ⊕ y) で再利用する。

**Marking System への適用:**

bidirectional typingでは、あるノードの型は親のモード（synth/ana）と兄弟の型情報に依存する。中間結果としてキャッシュすべきは：

1. **各ノードのtyping context (Γ)**: letバインディングやλで拡張されたcontext
2. **各ノードのtyping mode**: synth vs ana、および期待される型
3. **各サブ式の推論された型**: 兄弟ノード間の情報伝搬に必要

```moonbit
// P2: 中間結果をキャッシュする拡張版
struct MarkingCache {
  // ノードID → (context, mode, 推論型) の対応
  entries : Map[NodeId, CacheEntry]
}

struct CacheEntry {
  ctx : Ctx               // そのノードでのtyping context
  mode : TypingMode       // Synth | Ana(expected_type)
  result_type : Type      // 推論/チェック結果の型
  marked : MarkedExpr     // marking結果
}
```

### P3: 補助情報の発見

> f(x) では計算されないが、f(x ⊕ y) の効率化に役立つ補助値を見つける。

**Marking System への適用:**

bidirectional typingの伝搬パターンを分析すると、以下の補助情報が有用：

1. **依存関係グラフ**: 「ノードAの型変更がノードBの型に影響するか」の前計算
2. **変数使用マップ**: 「変数 x がどのholeで使われているか」→ binding変更時の影響範囲特定
3. **型制約の逆引き**: 「型 T を持つノードはどれか」→ 型定義変更時の一括更新

```moonbit
// P3: Markingには直接現れないが増分更新に有用な補助情報
struct AuxInfo {
  // 変数名 → 使用箇所のノードID集合
  var_uses : Map[String, Set[NodeId]]
  // ノードID → そのノードの型に依存するノードID集合
  type_deps : Map[NodeId, Set[NodeId]]
  // typing context中の束縛 → 影響を受けるノードID集合
  binding_deps : Map[(String, Type), Set[NodeId]]
}
```

**重要**: P3 はコスト分析を必要とする。補助情報の維持コストが、それによる高速化を上回らないことを確認する。たとえば `type_deps` の完全な維持は更新コストが高い場合があり、近似的な（保守的な）依存関係を使うことも選択肢。

---

## 3. III Method: Iterate-Incrementalize-Implement

Liu の III method は incrementalization を核に据えた体系的なプログラム設計・最適化手法である：

> **I1 (Iterate)**: 最小の増分で反復的に所望の出力に到達する方法を決定する
> **I2 (Incrementalize)**: 各反復での高コスト操作を、有用な追加値の維持により増分化する
> **I3 (Implement)**: 維持される値を効率的に格納・アクセスするためのデータ構造を設計する

### 3.1 Marking SystemへのIIIの適用

**I1 (Iterate): ASTの変更を最小増分に分解**

ASTの編集を最小単位に分解する。最小増分は以下の4種類のGrove操作に対応する：

```moonbit
pub enum MinimalOp {
  InsertVertex(VertexId, NodeLabel)    // ノード追加
  InsertEdge(EdgeId, Src, Dst, Slot)   // 辺追加（親子関係設定）
  DeleteEdge(EdgeId)                   // 辺削除（親子関係解除）
  SetProperty(VertexId, Key, Value)    // プロパティ変更
}
```

任意のAST編集は、これらの最小操作の列として表現される。各最小操作が ⊕ の1ステップに対応。

```
例: `let x = 1 + 2` の `+` を `*` に変更
→ SetProperty(add_node_id, "operator", Mul)
→ 1つの最小操作
```

```
例: `1 + 2` を `(1 + 2) * 3` にラップ
→ InsertVertex(mul_id, MulLabel)
→ InsertVertex(lit3_id, LitLabel)
→ SetProperty(lit3_id, "value", 3)
→ DeleteEdge(parent_to_add)            // 元の親からaddを外す
→ InsertEdge(e1, mul_id, add_id, Left) // mulの左子としてadd
→ InsertEdge(e2, mul_id, lit3_id, Right)
→ InsertEdge(e3, parent, mul_id, slot) // 元の親にmulを接続
→ 7つの最小操作
```

**I2 (Incrementalize): 各最小操作後のMarkingを増分化**

各最小操作 op に対して、mark_synth' を P1-P3 で導出する：

```moonbit
fn mark_incr(
  cache : MarkingCache,
  aux : AuxInfo,
  op : MinimalOp
) -> (MarkingCache, AuxInfo) {
  match op {
    SetProperty(vid, key, value) => {
      // 影響範囲: vid からルートまでのパス上のノード
      let affected = ancestors(vid)
      // affected のみ再marking、他はcacheから取得
      ...
    }
    InsertVertex(vid, label) => {
      // 新ノードの初期marking（EmptyHoleとして）
      ...
    }
    InsertEdge(eid, src, dst, slot) => {
      // dst の typing context が変わる（親がsrcになる）
      // dst のサブツリー全体を再markingする必要があるかも
      // ただし typing context が実質的に変わらなければスキップ可
      ...
    }
    DeleteEdge(eid) => {
      // 外れた子ノードは context を失う → EmptyHole化
      ...
    }
  }
}
```

**核心的な洞察**: bidirectional typingでは情報の流れが明確である。

- **synthモード**: 情報は子 → 親（bottom-up）
- **anaモード**: 情報は親 → 子（top-down）

したがって、ノードvの変更の影響は：

- synthモードの祖先をルート方向に伝搬（型が変われば）
- anaモードの子孫に伝搬（typing contextや期待型が変われば）

最悪ケースはルートからリーフまでO(depth)だが、型が途中で変化しなければ伝搬は**早期終了**できる。

```moonbit
// 伝搬の早期終了条件
fn propagate_up(
  cache : MarkingCache,
  node : NodeId,
  new_child_type : Type
) -> Unit {
  let parent = cache.parent(node)
  let prev_entry = cache.entries[parent]
  // 親を再markingする
  let (new_marked, new_type) = remark(prev_entry.ctx, parent, ...)
  if new_type == prev_entry.result_type {
    // 型が変化しなければ、これ以上の祖先への伝搬は不要
    return
  }
  // 型が変わった → さらに上に伝搬
  cache.entries[parent] = CacheEntry { result_type: new_type, .. }
  propagate_up(cache, parent, new_type)
}
```

**I3 (Implement): 効率的なデータ構造の設計**

Liuの I3 ステップは「集合操作が定数時間になるようなデータ構造の選択」。ここでは：

| 維持すべき値 | データ構造 | 操作の計算量 |
|-------------|-----------|-------------|
| MarkingCache (ノード→型情報) | HashMap | O(1) lookup/update |
| AuxInfo.var_uses (変数→使用箇所) | HashMap + HashSet | O(1) amortized |
| AuxInfo.type_deps (依存関係) | HashMap + HashSet | O(1) amortized |
| 祖先パス (root方向探索) | 親ポインタ付き木 | O(depth) |
| GroveGraph (CmRDT) | adjacency map | O(1) per edge op |

**Order Maintenance (Hazel OOPSLA 2025)** も I3 の選択に含まれる。Hazelでは、ASTノードの「兄弟間順序」をorder maintenance data structureで管理し、再marking時の順序依存を O(1) amortized で判定している。

---

## 4. 各サブシステムへのIII適用の詳細

### 4.1 Text Projection: MarkedAST → String

```
f₂ = pretty_print : MarkedAST → String
⊕₂ = mark_change  : MarkedAST × Δ → MarkedAST
```

**I1**: MarkedAST の変更を最小増分に分解。Pretty printの対象は変更されたサブツリーの表現のみ。

**I2**: 各ノードの文字列表現と文字列オフセット位置をキャッシュ。変更されたノード以下のみ再生成。

**I3**: Piece Table または Rope で出力文字列を管理。部分的な文字列の挿入・削除が O(log n)。

```moonbit
struct TextProjectionCache {
  // ノードID → (開始offset, 終了offset, 文字列断片)
  fragments : Map[NodeId, TextFragment]
  // 出力全体を管理するRope
  output : Rope
}

fn text_project_incr(
  cache : TextProjectionCache,
  changed_nodes : Set[NodeId]
) -> TextProjectionCache {
  for node in changed_nodes {
    let new_text = render_node(node)
    let old_fragment = cache.fragments[node]
    // Ropeの該当範囲を置換
    cache.output = cache.output.replace(
      old_fragment.start, old_fragment.end, new_text
    )
    // fragmentのオフセット群を更新
    update_offsets(cache.fragments, node, new_text.length - old_fragment.length)
  }
  cache
}
```

### 4.2 Grove CmRDT: GroveGraph → AST (Decompose)

```
f₅ = decompose : GroveGraph → Expr  (グラフからwell-formed ASTを抽出)
⊕₅ = grove_op  : GroveGraph × GroveOp → GroveGraph
```

GroveのCmRDT操作は**可換**であるため、incrementalizationに特に適している。

**I1**: 最小増分は単一のGroveOp（上述の4種類）。CRDTの可換性により、操作の適用順序に関わらず同じ結果に収束する。

**I2**: decompose の中間結果として以下をキャッシュ：

- 各頂点の「勝利辺」（conflictが解決された後の親子関係）
- conflict情報（同じスロットに複数の辺 → ConflictHole生成）

```moonbit
struct DecomposeCache {
  // 各頂点の現在の親辺
  winning_edges : Map[VertexId, EdgeId]
  // 各スロットの辺集合（conflict検出用）
  slot_edges : Map[(VertexId, ChildSlot), Set[EdgeId]]
  // 現在のAST（キャッシュ）
  current_ast : Expr
}

fn decompose_incr(
  cache : DecomposeCache,
  op : GroveOp
) -> DecomposeCache {
  match op {
    InsertEdge(eid, src, dst, slot) => {
      let edges = cache.slot_edges[(dst, slot)]
      edges.insert(eid)
      if edges.size() == 1 {
        // conflict なし → 通常の親子関係設定
        cache.winning_edges[dst] = eid
        // ASTの該当箇所を更新
      } else {
        // conflict → ConflictHole生成
        // 該当箇所をConflictHole(hole_id, [child1, child2, ...]) に
      }
    }
    DeleteEdge(eid) => {
      // 辺削除 → conflict 解消の可能性
      // slot_edgesから除去、残り1本なら勝者確定
    }
    // ...
  }
}
```

**I3**: adjacency mapはHashMapで実装。conflict検出のslot_edgesもHashMap。全操作がO(1) amortized。

### 4.3 Text Input Adapter: TextEdit → ASTOp[]

```
f₄ = translate : (TextEdit, CursorContext, AST) → ASTOp[]
```

ここでのincrementalizationは他とは質的に異なる。f₄ は「変換」であって、大きなデータ構造の更新ではない。しかし Liu の枠組みは依然として有用。

**TextFragmentのgradual parsingにおけるincrementalization:**

```
f_parse = parse_fragment : (String, Ctx) → PartialParse[]
⊕_char = append_char    : String × Char → String
```

ユーザーがTextFragment内で文字を1つ入力するたびに、パース候補を再計算する。

**I1**: 最小増分は1文字の追加/削除。

**I2**: 前回のパース候補リストを再利用。多くの場合、1文字の追加は候補をフィルタリングするだけ（"le" → "let" の候補が絞られる）。

```moonbit
struct ParseState {
  text : String
  candidates : Array[PartialParse]
  // 前回のトークン境界位置（再トークナイズの範囲を限定）
  token_boundaries : Array[Int]
}

fn parse_incr(
  state : ParseState,
  edit : CharEdit  // 1文字の挿入 or 削除
) -> ParseState {
  let new_text = apply(state.text, edit)
  // 変更位置のトークンのみ再トークナイズ
  let affected_token = find_affected_token(state.token_boundaries, edit.pos)
  let new_tokens = retokenize(new_text, affected_token)
  // 候補をフィルタリング（多くの場合、追加で完全再パースは不要）
  let new_candidates = state.candidates
    .filter(fn(c) { c.is_compatible_with(new_tokens) })
  // 新しい候補が出現する可能性もチェック
  if keyword_completed(new_tokens) {
    new_candidates.push(generate_ast_template(new_tokens))
  }
  ParseState { text: new_text, candidates: new_candidates, .. }
}
```

---

## 5. 最適化の統合: Optimization by Incrementalization

Liu の重要な洞察は「incrementalization による最適化は微分による積分に相当する」ということ。フィボナッチ関数のincrementalizeから線形時間アルゴリズムが導出されるように、**増分版を求めてからループで包む**ことで全体の最適化が得られる。

### 5.1 Marking全体の計算量分析

**非インクリメンタル版:**
```
mark_synth(ctx, ast) : O(|AST|) — ASTの全ノードを走査
```

**インクリメンタル版 (III適用後):**
```
1操作あたり:
  mark_incr(cache, aux, op) : O(depth × branching_factor_at_change_point)
  
  最良ケース: O(1) — 型が変化しない末端ノードの変更
  最悪ケース: O(|AST|) — ルートの型が変わる変更
  典型ケース: O(depth) — 型の変更が祖先方向に伝搬するがどこかで止まる
```

**「積分」による全体最適化:**

ASTを最初から構築する場合、n個のノードを1つずつ追加する操作の列として表現できる。各追加のincrementalコストが典型的にO(depth)なら：

```
全体の構築コスト = Σᵢ O(depth_i) ≈ O(n × average_depth) ≈ O(n log n)
```

これは非インクリメンタルなO(n)より大きいが、**変更のみに対するコスト**はO(depth)であり、エディタのインタラクティブな応答時間としては十分。

### 5.2 高レベル抽象の力

Liu が繰り返し強調するのは「高レベルの抽象がincrementalizationを劇的に容易かつ強力にする」という点。

Editorシステムにおける対応：

| 抽象レベル | 具体例 | incrementalization の容易さ |
|-----------|--------|---------------------------|
| **低レベル**: テキストバッファ上の文字操作 | 従来のテキストエディタ | 困難。文字の挿入/削除とAST構造変更の関係が不明確 |
| **中レベル**: AST上のノード操作 | 構造エディタ (MPS風) | 中程度。木構造の局所性を利用可能 |
| **高レベル**: Grove CmRDT上の可換操作 | Hazel/本アーキテクチャ | **容易**。操作の可換性がincrementalizationの正当化を簡潔にする |
| **最高レベル**: DSL型システム規則 | bidirectional typing rules | **最も容易**。型規則の構造がそのまま依存関係グラフになる |

bidirectional typingの規則がそのまま増分化のガイドになるのは、Liu の言う「高レベル制御抽象（再帰）による解析・変換の容易さ」の好例。

---

## 6. 部分評価との関係

Liu は incrementalization と partial evaluation の双方向の関係を指摘する：

> - 部分評価はincrementalizationの特殊ケース
> - Incrementalizationは一般化部分評価の特殊ケース

### 6.1 EditorにおけるPartial Evaluation的側面

**Text Input Adapter は partial evaluation として捉えられる:**

```
interpret(grammar, input) = AST
```

grammar は DSL の文法定義で固定、input はユーザーの入力テキスト。すると：

```
PE(interpret, grammar) = interpret_grammar = parser_for_grammar
```

つまり、文法定義に対してインタプリタを部分評価すると、その文法に特化したパーサーが得られる。これは古典的な Futamura projection の analogy。

**Marking System も partial evaluation として捉えられる:**

```
type_check(rules, expr) = MarkedExpr
```

rules は DSL の型付け規則で固定、expr は変わる。すると：

```
PE(type_check, rules) = type_checker_for_rules
```

DSL定義の型規則に特化した型チェッカーが自動生成される。これはまさに将来の「DSL定義からの自動導出」（課題C）への道筋。

### 6.2 Generalized Partial Evaluation と Gradual の統一

Liu の一般化部分評価は「プログラム f と入力 x についての任意の情報を利用する」。Gradual typing もまさにこれ：

```
x = x_prev ⊕ y  かつ  f(x_prev) = r  → incremental computation
type(e) = ?     （型が不明）         → gradual typing
```

どちらも「部分的な情報の下での最善の計算」を目指す。

- **Gradual typing**: 型情報が部分的 → 分かる範囲で型チェック、不明部分は ? で据え置き
- **Incremental computation**: 変更情報が部分的 → 変わった部分だけ再計算、残りは前回結果を再利用

この共通構造は偶然ではない。Hazel の marking が incremental bidirectional typing と自然に結合するのは、両者が「部分情報の下での最善の処理」という同じ原理に基づいているから。

---

## 7. MVPへの具体的適用: Phase 1の設計

### 7.1 Phase 1 における incrementalization の位置づけ

Phase 1 の目標は「Marking の property test が通る MoonBit ライブラリ」。ここでは**まだ incremental にしない**が、将来の incrementalization を阻害しないデータ設計にする。

具体的には、III の I3（データ構造設計）を**先行して**検討し、incrementalization を妨げない構造を選択する：

```moonbit
// Phase 1: 非インクリメンタルだが、将来のキャッシュを阻害しない設計
pub struct MarkingResult {
  marked : MarkedExpr
  // Phase 1 では使わないが、構造に含めておく
  // Phase が進んだら MarkingCache に昇格する
  node_types : Map[NodeId, Type]  // 各ノードの推論型
  node_modes : Map[NodeId, TypingMode]  // 各ノードのモード
}

fn mark_synth(ctx : Ctx, expr : Expr) -> MarkingResult {
  // ... marking 実装 ...
  // 途中結果を node_types, node_modes に記録しながら進む
}
```

### 7.2 Property Test の incrementalization 検証

Liu の方法論では、incrementalization の正しさは以下で検証される：

```
∀ x, y, r. f(x) = r → f'(x, y, r) = f(x ⊕ y)
```

これは直接 property test にできる：

```moonbit
// Marking incrementalization の正しさ property
test "marking_incrementalization_correctness" {
  // ランダムな AST を生成
  let ast = gen_random_ast()
  // 非インクリメンタル marking
  let result1 = mark_synth(empty_ctx, ast)
  
  // ランダムな操作を適用
  let op = gen_random_op(ast)
  let ast2 = apply_op(ast, op)
  
  // 非インクリメンタルに再 marking
  let result2 = mark_synth(empty_ctx, ast2)
  
  // インクリメンタル marking
  let result_incr = mark_incr(result1, op)
  
  // 一致を確認
  assert_eq!(result2.marked, result_incr.marked)
  assert_eq!(result2.node_types, result_incr.node_types)
}
```

### 7.3 Phase 1 実装の判断基準

Phase 1 で non-incremental な mark_synth を実装し、上記の property test を**将来の incrementalization の受け入れテスト**として先に書いておく。Phase 3 以降で mark_incr を実装した際、同じテストでcorrectnessを検証できる。

---

## 8. 高レベル抽象の段階的導入ロードマップ

Liu の表 (Table 5.1) をEditorプロジェクトに適用：

| 抽象レベル | 言語機能 | III ステップ | 対応するPhase |
|-----------|---------|------------|-------------|
| ループ + プリミティブ | AST走査ループ | I2 のみ | Phase 1 |
| 集合式 | GroveGraph のノード/辺集合 | I2 + I3 | Phase 2 |
| 再帰関数 | mark_synth の再帰構造 | I1 + I2 | Phase 1→3 |
| 論理規則 | bidirectional typing rules | I1 + I2 + I3 | Phase 3→将来 |
| オブジェクト | Editor全体のモジュール構造 | I2 (モジュール間) | Phase 4-5 |

### 段階的な抽象の引き上げ

**現在 (Phase 1)**: MoonBit の match/enum で直接 marking を実装。再帰関数レベル。

**近い将来 (Phase 3以降)**: typing rules を宣言的に記述し、そこから marking 関数を導出。論理規則レベル。

**遠い将来 (DSL汎化)**: DSL定義から typing rules, marking, projection, input adapter を自動導出。Liu の「高レベル抽象に対する体系的 incrementalization」が最も力を発揮する段階。

---

## 9. まとめ: Incrementalization as the Unifying Principle

Liu の "Incremental Computation: What Is the Essence?" から得られる核心的な教訓をEditorプロジェクトに適用すると：

**Incrementalization は Gradual Projectional Editor の統一原理である。**

- **Marking** = 型情報の incremental computation
- **Projection** = 表現の incremental computation  
- **Text Input Adapter** = パースの incremental computation
- **Grove CmRDT** = 構造の incremental computation（CRDTの可換性が ⊕ の性質を保証）
- **Gradual typing** = incrementalization と partial evaluation の共通一般化

すべてが `f(x) = r ⟹ f'(x, y, r) = f(x ⊕ y)` という同じ形式に帰着する。

そして Liu の最も重要なメッセージ：**高レベル抽象を恐れるな**。集合、再帰、論理規則といった高レベル抽象こそが、体系的な incrementalization を可能にし、結果として手動最適化より優れたアルゴリズムを導出できる。

MoonBit の型システムと代数的データ型はこの方向に適しており、将来的に DSL の型規則を宣言的に記述し、そこから incremental type checker を自動導出する道が開ける。

---

## 参考文献

- Liu, Y. A. (2023/2025). "Incremental Computation: What Is the Essence?" Foundations and Trends in Programming Languages. arXiv:2312.07946
- Liu, Y. A. (2013). "Systematic Program Design: From Clarity to Efficiency." Cambridge University Press.
- Omar, C. et al. (2017). "Toward Semantic Foundations for Program Editors." SNAPL 2017.
- Omar, C. et al. (2017). "Hazelnut: A Bidirectionally Typed Structure Editor Calculus." POPL 2017.
- Adams, M. et al. (2025). "Grove: A Bidirectionally Typed Structure Editor Calculus." POPL 2025.
- (OOPSLA 2025). "Incremental Bidirectional Typing via Order Maintenance."
- Moon, D. et al. (2023). "Gradual Structure Editing with Obligations." VL/HCC 2023.
- Hammer, M. A. et al. (2014). "Adapton: Composable, Demand-Driven Incremental Computation." PLDI 2014.
- Cai, Y. et al. (2014). "A Theory of Changes for Higher-Order Languages." PLDI 2014.
