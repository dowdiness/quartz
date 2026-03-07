---
title: Incremental Hylomorphism Pipeline
created: 2026-03-07T22:33:12+09:00
modified: 2026-03-07T22:38:37+09:00
tags:
  - compiler
  - incremental-computation
aliases:
  - "AST as Source of Truth: Incremental Hylomorphism Pipeline"
publish: true
---

# AST as Source of Truth: Incremental Hylomorphism Pipeline

注: Claudeに作ってもらった計画書です

## Overview

本ドキュメントは、テキストエディタ・コンパイラ・協調編集を統一的に捉える設計原理をまとめたものである。中心にあるのは次の命題である。

> **ASTを真実の源（source of truth）とし、入力からの構築（anamorphism）と出力への射影（catamorphism）を、インクリメンタルなhylomorphismの連鎖として統一する。**

この設計原理は、serde的なシリアライズ/デシリアライズ、パーサー、インクリメンタル計算、CRDT、UI描画を同じ理論的枠組みの中で理解し、設計するための基盤を提供する。

---

## 1. 基本概念: 構造の構築と破壊

### Catamorphism（構造の破壊 / fold）

既にある再帰的構造を、代数 `F A → A` に従って一段ずつ畳み込む操作。

```
cata : (F A → A) → μF → A
```

構造を持つ値から、構造を持たない値を生成する。情報の流れは**一方向**であり、構造の保持者（データ型）が全知識を持つため、処理は単純になる。

例:
- AST → 評価結果
- AST → 画面表示
- データ型 → JSONバイト列（シリアライズ）

### Anamorphism（構造の構築 / unfold）

余代数 `S → F S` に従って、種（seed）から再帰的構造を一段ずつ展開する操作。

```
ana : (S → F S) → S → νF
```

構造を持たない値から、構造を持つ値を生成する。構築には外部情報の消費（パース）や失敗の可能性が伴い、catamorphismより本質的に複雑になる。

例:
- テキスト → AST（パース）
- JSONバイト列 → データ型（デシリアライズ）
- ユーザー操作 → 内部状態

### Hylomorphism（構築して破壊する）

anamorphismで構造を構築し、catamorphismで破壊する合成操作。

```
hylo : (F B → B) → (A → F A) → A → B
        ^^^^^^^^    ^^^^^^^^^^
        代数(cata)   余代数(ana)
```

中間に生まれる構造 μF は理論上消去可能（deforestation）だが、実用上は中間表現として保持し、最適化や検査の対象とする。

### 非対称性: なぜ構築は破壊より難しいか

| 性質 | Catamorphism（破壊） | Anamorphism（構築） |
|------|---------------------|---------------------|
| 情報の所在 | データ型が全知識を持つ | 知識が二者に分散する |
| 失敗 | 起こらない | 入力が不正なら失敗する |
| 多相性 | 一軸（出力先） | 二軸（入力元 × ターゲット型） |
| 局所変更の影響 | 局所的に留まりやすい | 大域的な構造変化を引き起こしうる |

デシリアライズが複雑になる根本原因はこの非対称性にある。パーサー（入力の構文的知識）とターゲット型（構造的知識）という**二つの独立した知識**を協調させる必要があり、Serdeの Visitor パターンはこの協調プロトコルを実現するための仕組みである。

---

## 2. Finally Tagless: Church Encoding による統一

### Church Encoding としての Serialize

Church encodingとは、データ型を「そのfoldを受理する関数」として表現することである。

```
Church Nat  = ∀r. (r → r) → r → r
Church List = ∀r. (a → r → r) → r → r
Serialize   = ∀S:SerializerSym. Self → S
```

`Serialize` の実装は「任意の代数（SerializerSym実装）を渡されたら、自分をその代数で畳み込める」ことを宣言する。データ型が自分自身の Church encoding を提示するのが `serialize` メソッドである。

### SerializerSym: プロトコルとしてのデータモデル

```moonbit
trait SerializerSym {
  write_bool(Self, Bool) -> Self
  write_int(Self, Int) -> Self
  write_str(Self, String) -> Self
  seq_begin(Self, Int) -> Self
  seq_elem(Self, Self) -> Self
  seq_end(Self) -> Self
  struct_begin(Self, String, Int) -> Self
  struct_field(Self, String, Self) -> Self
  struct_end(Self) -> Self
}
```

これは Finally Tagless の `ExprSym` と同じ構造であり、各フォーマット（JSON, MsgPack, YAML...）がこの trait の異なる「解釈」を提供する。中間値型 `Value` を経由せず、直接出力に書くことでゼロアロケーションを実現できる。

### Deserialize の消去

Serde の Visitor が複雑になる理由は、Church encoding の**消費者**がターゲット型を選ぶ際に associated type が必要になるからである。

この複雑さは、各フォーマットの Value 型（JsonValue, MsgPackValue 等）に `Serialize` を実装させることで消去できる。

```
シリアライズ:   Point.serialize(json_writer)        -- Point → JSON bytes
フォーマット変換: json_value.serialize(msgpack_writer) -- JSON → MsgPack（Value不要）
デシリアライズ:  json_value.serialize(value_builder)  -- JsonValue → Value → Point
```

必要な trait は三つだけ:

```moonbit
trait SerializerSym { ... }  // プロトコル（データモデル）
trait Serialize { ... }      // Self → プロトコルで記述（Church encoding）
trait FromValue { ... }      // Value → Self（Fixed-Type Projection）
```

Deserialize trait、Visitor pattern、MapAccess / SeqAccess は全て不要になる。

---

## 3. 境界のパターン: あらゆるシステム境界に現れる Hylomorphism

```
外部表現₁ ←(ana)→ 内部表現 ←(cata)→ 外部表現₂
```

この構造はあらゆるシステム境界で反復する。

| システム | 入力 (ana) | 内部表現 | 出力 (cata) |
|----------|-----------|----------|-------------|
| コンパイラ | ソースコード → AST | IR | IR → ターゲットコード |
| serde | JSON bytes → JsonValue | Value / 型 | 型 → MsgPack bytes |
| UI (TEA) | ユーザー操作 → Model | Model | Model → View |
| エディタ | テキスト → AST | AST | AST → 画面表示 |
| ネットワーク | パケット → メッセージ型 | メッセージ型 | メッセージ型 → アプリ処理 |

コンパイラは単一の hylomorphism ではなく、**ファンクタが段階ごとに変わる hylomorphism の連鎖**である。

```
ana₁ → μF₁ → cata₁/ana₂ → μF₂ → cata₂/ana₃ → μF₃ → cata₃
パース    AST    lowering      MIR    codegen       ...
```

---

## 4. 全体パイプライン: loom + incr + CRDT

### パイプライン図

```
  ユーザーA の編集操作       ユーザーB の編集操作
        │                         │
        ▼                         ▼
  ┌───────────┐    同期     ┌───────────┐
  │ CRDT Ops  │◄──────────►│ CRDT Ops  │
  └─────┬─────┘            └─────┬─────┘
        │ ① fold                 │ ① fold
        ▼                         ▼
  ┌───────────┐            ┌───────────┐
  │  文書状態  │            │  文書状態  │
  │ (テキスト) │            │ (テキスト) │
  └─────┬─────┘            └─────┬─────┘
        │ ② ana (loom)           │ ② ana (loom)
        ▼                         ▼
  ┌───────────┐            ┌───────────┐
  │    AST    │            │    AST    │
  │(穴+エラー)│            │(穴+エラー)│
  └─────┬─────┘            └─────┬─────┘
        │ ③ cata (incr)          │ ③ cata (incr)
        ▼                         ▼
  ┌───────────┐            ┌───────────┐
  │ 型付きAST │            │ 型付きAST │
  └─────┬─────┘            └─────┬─────┘
        │ ④ cata                 │ ④ cata
        ▼                         ▼
  ┌───────────┐            ┌───────────┐
  │  画面表示  │            │  画面表示  │
  └───────────┘            └───────────┘
```

### 各境界の責務

**① CRDT Ops → 文書状態** (fold)
- 操作列を畳み込んで文書のテキスト状態を得る
- eg-walker / FugueMax が並行挿入の順序を決定する
- 出力: テキストバッファ + 変更のダメージ領域

**② 文書状態 → AST** (anamorphism — loom)
- テキストからトークン化 → パースで AST を構築する
- エラー回復（`expect`, `skip_until`, `skip_until_balanced`）で不完全な入力からも構造を維持する
- インクリメンタルパース: ダメージ領域のみ再パースし、変更のないサブツリーは再利用する
- 出力: 穴（hole）やエラーノードを含みうる AST

**③ AST → 型付きAST** (catamorphism — incr)
- 名前解決、型検査、フロー解析などの意味解析を行う
- incr / Salsa 的な依存グラフ追跡でインクリメンタルに再計算する
- AST の木構造を横断する依存関係（名前参照、型の伝播）を追跡する
- 出力: 型情報、診断メッセージ、補完候補などの materialized view

**④ 型付きAST → 画面表示** (catamorphism)
- 型情報付き AST からシンタックスハイライト、インデント、エラー表示等を計算する
- 仮想 DOM / incremental rendering でダメージ領域のみ再描画する

### incr の役割: Hylomorphism のメモ化

incr は各境界の hylomorphism を**メモ化**する。入力が部分的に変化したとき、影響を受けるノードだけ再計算する。

```
incr なし: 1文字変更 → 全体を再パース → 全体を再型検査 → 全体を再描画
incr あり: 1文字変更 → 3トークン再解析 → 2関数の型を再検査 → 該当行を再描画
```

これは materialized view の差分更新と同じ概念である。AST がベーステーブル、型情報や画面表示が materialized view に対応し、ベーステーブルの更新が view に差分的に伝播する。

---

## 5. 各境界の技術的課題

### 境界①: CRDT → 文書状態

**粒度の不一致**。CRDT は文字単位の操作を扱うが、パーサーが必要とする情報は「どのトークンが影響を受けたか」である。一文字の変更がトークンの種類を変えることもある（例: `42` → `4.2` で Int が Float になる）。

**到着順序の不定性**。CRDT では操作の到着順序が保証されない。複数ユーザーの同時編集がどの順序で適用されるかは FugueMax の順序決定に依存し、結果によってトークン化が変わりうる。

**必要な解決**: 文字位置の差分からトークンレベルのダメージ領域への変換アダプタ。

### 境界②: 文書状態 → トークン列

**レクサー状態のコンテキスト依存**。テンプレートリテラル、ヒアドキュメント、ネストしたコメントなど、レクサーの状態が前後の文脈に依存する場合がある。一文字の変更がレクサー状態機械を根本的に変え、後続の全トークンに影響しうる。

**必要な解決**: 各トークンにレクサー状態のスナップショットを保持し、再トークン化の伝播が「前回と同じ状態に戻ったら停止」する仕組み。

### 境界③: トークン列 → AST（最難関）

**局所変更の大域的影響**。テキストの一文字変更が AST の親子関係を根本的に変えうる（例: `else` の `e` を消すと if 文の構造が崩壊する）。

**エラー回復の安定性**。エラー状態でのパース結果がユーザーの編集に対して安定的でなければならない。同じエラー状況に対して常に同じ回復戦略を取ること（べき等性）で、無関係な部分の AST が揺れないことを保証する必要がある。

**構造の構築は構造の破壊より本質的に不安定**。catamorphism では局所変更の影響は局所的に留まりやすいが、anamorphism では入力の局所変更がツリー全体の構造変化を引き起こしうる。これがインクリメンタルパースの核心的な難しさである。

### 境界④: AST → 意味解析

**木構造を横断する依存関係**。AST は木だが、意味的な依存関係は DAG である。関数 `foo` の型検査が関数 `bar` の型に依存するなど、木構造上の親子関係とは無関係な依存が存在する。

**意味を通じたダメージの伝播**。一つの関数のシグネチャ変更が、別ファイルの型検査結果を無効化する。物理的に遠い場所にダメージが飛ぶ。

**必要な解決**: incr の依存グラフが AST の親子関係ではなく、名前解決や型の依存関係に基づいて構築される仕組み（Salsa のクエリベースの依存追跡）。

### 境界を跨ぐ問題: 粒度の不一致

各段階の「変更単位」が異なる。

```
CRDT     → 文字単位
レクサー → トークン単位
パーサー → ASTノード単位
型検査   → シンボル/クエリ単位
描画     → ピクセル/行単位
```

各段階の「何が変わったか」を次の段階の「何を再計算すべきか」に変換するマッピング自体に計算コストがかかる。全段階の連鎖を 16ms 以内に完了させるために、各変換の効率が要求される。

---

## 6. Finally Tagless による拡張性

### エラーと穴の表現

Finally Tagless のtrait拡張により、穴やエラーを開いた形で追加できる。

```moonbit
// 基本言語
trait ExprSym {
  lit(Int) -> Self
  add(Self, Self) -> Self
}

// 穴の追加 — 既存コード変更なし
trait HoleSym {
  hole(HoleId) -> Self
  error(ErrorInfo, Self) -> Self
}
```

これは recursion scheme の「ファンクタ F の拡張」に対応する。enum（μF）への variant 追加は閉じた操作だが、trait の追加は開いた操作であり、Expression Problem の解がここでも活きる。

### Two-Layer Architecture

構造の観察（最適化、変換）が必要な場面では、Finally Tagless と具体的な enum を組み合わせる二層構造を用いる。

- **Layer 1 (Abstract)**: Finally Tagless traits — 拡張可能な構文定義
- **Layer 2 (Concrete)**: Enum — 構造の観察、パターンマッチ、最適化

```moonbit
// 具体 AST 上で最適化を適用し、tagless API を通じて再解釈する
fn replay[T : ExprSym + HoleSym](e : ConcreteAst) -> T { ... }
```

---

## 7. Recursion Scheme の適用限界

Recursion scheme で捉えられるものと捉えられないものの境界を認識することが重要である。

### 捉えられるもの

- **構造の変換**: ある表現から別の表現への変換（パース、コード生成、シリアライズ）
- **構造の走査**: 既存の構造に対する集約・射影（評価、表示、型情報の抽出）
- **構造の差分更新**: hylomorphism のメモ化としてのインクリメンタル計算

### 捉えられないもの

- **双方向の情報伝播**: 型推論における単一化のように、構造の中で情報が上下左右に流れる場合は制約解消という別のパラダイムが必要
- **因果的に順序づけられた操作の統合**: CRDT の操作統合は、単純な fold ではなく因果グラフの走査を必要とする
- **時間をまたぐ再利用**: インクリメンタル計算は hylomorphism のメモ化として捉えられるが、何を再利用し何を無効化するかの判断自体は recursion scheme の外にある

### 複雑さの三軸

構造の構築が難しくなる方向性は三つあり、それぞれ独立に効く。

1. **多相性の軸が増える**: フォーマット × 型、さらに文法規則が加わる
2. **情報の流れが双方向になる**: 型推論のように制約が相互に伝播する
3. **時間が入る**: 過去の結果を再利用する必要が生じる

最初の軸だけなら Serde 的 Visitor で解決可能。二番目が入ると制約解消。三番目が入るとインクリメンタル計算。loom + incr は一と三を同時に扱う。

---

## 8. 構造の構築ユースケース: 単純から複雑へ

| 複雑さ | ユースケース | 多相性 | 解決パラダイム |
|--------|-------------|--------|---------------|
| 最低 | リテラルのパース (`"42" → Int`) | なし | 単純な関数 |
| 低 | 固定スキーマのパース (`CSV → Point`) | なし | 専用パーサー |
| 中 | 同一フォーマット × 複数型 | 型軸のみ | 中間値 + FromValue |
| 中高 | 複数フォーマット × 複数型 | 両軸 | SerializerSym 三段分解 |
| 高 | 再帰的AST構築 | 両軸 + 再帰 | パーサーコンビネータ (loom) |
| 高 | 型推論 | 双方向 | 制約解消 / 単一化 |
| 最高 | CRDT + インクリメンタル再構築 | 両軸 + 時間 | 依存グラフ追跡 (incr) |

---

## 9. 設計判断のガイド

### プロトコル設計: Trait か Value か

```
構造の破壊が必要？
├─ Yes → Finally Tagless (Church encoding)
│        SerializerSym 的な trait でゼロアロケーション
└─ No
   └─ 構造の構築が必要？
      ├─ 一軸（フォーマット固定 or 型固定）
      │  → 専用パーサー / 専用 FromValue
      └─ 二軸（フォーマット × 型）
         ├─ パフォーマンス重要 → Visitor (Church encoding の双対)
         └─ 簡潔さ重要 → Value 中間表現で三段分解
```

### インクリメンタル化の判断

```
変更頻度は？
├─ バッチ（一括処理） → インクリメンタル化不要、単純な hylo
└─ インタラクティブ（16ms以内）
   ├─ 変更の影響範囲は局所的？
   │  ├─ Yes → ダメージ領域ベースの部分再計算
   │  └─ No  → incr / Salsa 的な依存グラフ追跡
   └─ 複数ユーザーの並行編集がある？
      ├─ Yes → CRDT + ダメージ伝播
      └─ No  → 単一ユーザーのインクリメンタル計算
```

---

## 参考文献

- Meijer, E., Fokkinga, M., & Paterson, R. (1991). *Functional Programming with Bananas, Lenses, Envelopes and Barbed Wire.*
- Carette, J., Kiselyov, O., & Shan, C. (2009). *Finally Tagless, Partially Evaluated.* JFP.
- Omar, C. et al. (2019). *Hazel: A Live Functional Programming Environment with Typed Holes.*
- Arvo, J. et al. (2022). *Grove: A Collaborative Structure Editor.*
- Matklad. (2023). *Resilient LL Parsing Tutorial.*
- Rust Serde documentation. https://serde.rs/
