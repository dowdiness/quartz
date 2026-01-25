---
title: Base Template
publish: false
tags: [" "]
aliases: [無題のファイル]
created: 2026-01-04T20:50:52+09:00
modified: 2026-01-25T23:57:12+09:00
---

# Run Length Encoding

Maybe we can make something like this?

```
struct RleVec[T]{ .. } derive (RleView, RleEdit)
struct RleVecCached[T]{ .. } derive (RleView, RleEdit, RleCacheOps)

enum Rle[T] {
  Vec(RleVec[T])
  VecCached(RleVecCached[T])
}
```

Trait を使うかどうかのCodexとの会話

 Got it — then a shared interface trait makes sense, even without default impls, to enforce
  “all data structures expose the same core operations.”

  If the goal is a uniform interface across multiple data shapes (text, list, tree,
  spreadsheet), I’d separate:

  - A small, generic “sequence-like” trait that captures common editor ops (length, slice/
    query, insert/delete/replace, iter ranges, etc.).
  - Domain-specific traits per structure (tree ops, table ops) layered on top.


  - run_count, total_atom_len, total_content_len
  - search, iter_slices, split_at, push
  on performance needs.

  Two cautions:

  - Keep the trait small and stable; otherwise you lock yourself into RLE‑specific semantics
    across other data structures.
  - Don’t make cache-specific methods part of the trait — keep those on a separate “cached”
    trait if needed.

  If you want, I can draft a candidate trait for the “sequence-like” core and show how it would
  extend to tree/list/table later.

Here’s a sketch that keeps RleVec/RleVecCached aligned while leaving room for other data
  structures. This is “shape only” — minimal shared interface + optional cache‑only trait.

  ///| Core interface for RLE-like sequences (read-only ops).
  pub trait RleView[T] {
    is_empty(Self) -> Bool
    run_count(Self) -> Int
    total_atom_len(Self) -> Int
    total_content_len(Self) -> Int
    search(Self, Int) -> SearchResult?
    iter_slices(Self, start~ : Int, end~ : Int) -> Result[Iter[Slice[T]], RleError]
    iter_slices_clamped(Self, start~ : Int, end~ : Int) -> Iter[Slice[T]]
    iter(Self) -> Iter[T]
  }

  ///| Mutating ops; keep in a separate trait so other models can opt out.
  pub trait RleEdit[T] {
    push(Self, T) -> Result[Unit, RleError]
    split_at(Self, Int) -> Result[(Self, Self), RleError]
    clear(Self) -> Unit
    concat(Self, Self) -> Self
    extend(Self, Self) -> Unit
  }

  ///| Cache-only ops (RleVecCached can implement this).
  pub trait RleCacheOps[T] {
    cursor(Self) -> RleCursor[T]
    to_rle_vec(Self) -> RleVec[T]
    from_rle_vec(RleVec[T]) -> Self
  }

  ///| Example usage: code can be generic over any RleView.
    start : Int,
    end : Int,
  ) -> Array[Slice[T]] {
    match vec.iter_slices(start~, end~) {
      Ok(it) => it.to_array()
      Err(_) => []
    }
  }

  Then implement on both RleVec and RleVecCached in event-graph-walker/rle (no defaults needed,
  just a guarantee that both expose the same surface).

  For the broader “text / list / tree / spreadsheet” goal, I’d make a higher‑level editor trait
  that’s not RLE‑specific (no SearchResult or Slice[T]). Example: DocModel with len, splice,
  iter_range, apply_ops, and specialized traits for tree/list/table ops. RLE types can
  implement that too, but you won’t lock everything into RLE‑centric semantics.

  If you want, I can draft a concrete DocModel sketch that fits your editor use‑cases.


› sketch more


• Here’s a concise yet expanded skill trait hierarchy sketch in mbt format, mapping text,
  list, tree, and table forms plus example operations without modifying files:

  Trait: Skill
    Description: Represents a specific ability or expertise.
    Subtraits:
      - BasicSkill
          Description: Foundational skills with simple traits.
          Properties:
            - name (string)
            - level (integer) // e.g., 1 to 5
      - CompositeSkill
          Description: Skills composed of multiple subskills.
          Properties:
            - name (string)
            - subskills (list of Skill)
      - SpecializedSkill (inherits BasicSkill)
          Description: Advanced skills with specialization detail.
          Properties:
            - specialization (string)
  Data Structures:
    Text: Skill name and specialization as strings.
    List: Collection of subskills inside CompositeSkill.
    Tree: CompositeSkill forms a tree with nested subskills.
    Table: Could map skill names vs. levels and types for quick lookup.
  Example Operations:
    - AddSkill: Add a BasicSkill or SpecializedSkill.
    - ComposeSkill: Create CompositeSkill from existing skills.
    - GetSkillLevel: Retrieve level from BasicSkill.
    - ListSubskills: Enumerate subskills under CompositeSkill.

  This shows how the traits relate and how different data representations correspond to
  the structure, enabling modular skill management.

• Here’s a more fleshed‑out sketch that separates:

  - universal editor surface (DocModel)
  - sequence‑like specialization (SeqModel)
  - tree/table specific traits
  - RLE adapters to the sequence model

  ///| Core model trait shared across all data shapes.
  pub trait DocModel {
    len(Self) -> Int
    is_empty(Self) -> Bool
    /// Apply a batch of ops (domain-specific enum).
    apply(Self, Array[DocOp]) -> Result[Unit, DocError]
    /// Read-only projection for rendering or diffing.
    snapshot(Self) -> DocSnapshot
  }

  ///| Sequence-like models (text, list, spreadsheet rows).
  pub trait SeqModel[T] {
    len(Self) -> Int
    slice(Self, start~ : Int, end~ : Int) -> Result[Array[T], DocError]
    iter(Self, start~ : Int, end~ : Int) -> Result[Iter[T], DocError]
    splice(Self, start~ : Int, end~ : Int, insert~ : Array[T]) -> Result[Unit, DocError]
  }

  ///| Tree models (movable tree, outline, AST).
  pub trait TreeModel[Node] {
    root(Self) -> Node
    parent(Self, Node) -> Node?
    children(Self, Node) -> Array[Node]
    insert_child(Self, parent~ : Node, index~ : Int, node~ : Node) -> Result[Unit,
  DocError]
    move(Self, node~ : Node, new_parent~ : Node, index~ : Int) -> Result[Unit, DocError]
    remove(Self, node~ : Node) -> Result[Unit, DocError]
  }

  ///| Table models (spreadsheet-like).
  pub trait TableModel[Cell] {
    rows(Self) -> Int
    cols(Self) -> Int
    get(Self, row~ : Int, col~ : Int) -> Result[Cell, DocError]
    set(Self, row~ : Int, col~ : Int, cell~ : Cell) -> Result[Unit, DocError]
    insert_row(Self, index~ : Int) -> Result[Unit, DocError]
    insert_col(Self, index~ : Int) -> Result[Unit, DocError]
    delete_row(Self, index~ : Int) -> Result[Unit, DocError]
    delete_col(Self, index~ : Int) -> Result[Unit, DocError]
  }

  ///| RLE-specific view for internal algorithms or performance-critical paths.
  pub trait RleView[T] {
    run_count(Self) -> Int
    total_atom_len(Self) -> Int
    total_content_len(Self) -> Int
    search(Self, Int) -> SearchResult?
    iter_slices(Self, start~ : Int, end~ : Int) -> Result[Iter[Slice[T]], RleError]
  }

  ///| Optional RLE edit trait.
  pub trait RleEdit[T] {
    push(Self, T) -> Result[Unit, RleError]
    split_at(Self, Int) -> Result[(Self, Self), RleError]
    clear(Self) -> Unit
  }

  ///| Cache-only extras.
  pub trait RleCacheOps[T] {
    cursor(Self) -> RleCursor[T]
  }

  ///| Example adapter: RleVec implements SeqModel[Char] via string impl.
  pub impl SeqModel[Char] for RleVec[String] {
    pub fn len(self : RleVec[String]) -> Int {
      self.total_content_len()
    }

    pub fn slice(self : RleVec[String], start~ : Int, end~ : Int)
      -> Result[Array[Char], DocError] {
      match self.iter_slices(start~, end~) {
        Ok(it) => {
          let out : Array[Char] = []
          for slice in it {
            for c in Slice::to_inner(slice) {
              out.push(c)
            }
          }
          Ok(out)
        }
        Err(_) => Err(DocError::IndexOutOfBounds)
      }
    }

    pub fn iter(self : RleVec[String], start~ : Int, end~ : Int)
      -> Result[Iter[Char], DocError] {
      match self.iter_slices(start~, end~) {
        Ok(it) => {
          let chars : Array[Char] = []
          for slice in it {
            for c in Slice::to_inner(slice) {
              chars.push(c)
            }
          }
          Ok(chars.iter())
        }
        Err(_) => Err(DocError::IndexOutOfBounds)
      }
    }

    pub fn splice(self : RleVec[String], start~ : Int, end~ : Int, insert~ : Array[Char])
      -> Result[Unit, DocError] {
      // Placeholder: map to RLE split + concat + from_string.
      Err(DocError::Unsupported)
    }
  }

  How this lines up with your goal:

  - Text: SeqModel[Char] (backed by RleVec/RleVecCached, or a gap buffer later).
  - Reorderable list: SeqModel[Item] (maybe tree-backed but still sequence ops).
  - Movable tree: TreeModel[NodeId].
  - Spreadsheet: TableModel[Cell].

  The key is: app-facing code targets DocModel + a specialization trait (Seq/Tree/Table).
  RLE types can also expose RleView for low‑level performance paths without leaking those
  details into the general editor API.

  If you want, I can turn this into actual interfaces in event-graph-walker/rle/
  traits.mbt (or a new event-graph-walker/model package) and show how RleVec +
  RleVecCached implement them.