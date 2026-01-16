---
aliases: [演算子]
created: 2024-11-01T20:10:11+09:00
modified: 2026-01-16T19:22:45+09:00
---

https://doc.rust-lang.org/book/appendix-02-operators.html

# 演算子

## 短絡評価(Short-circuiting)

Rustの演算子には短絡評価のものが２つある

```rust
false && true // 論理積(短絡評価) 左がfalseなら右は実行されない
true || false // 論理和(短絡評価) 左がtrueなら右は実行されない
false & true // Bitwise AND(論理積) 両方とも実行される
true | false // Bitwise OR(論理和) 両方とも実行される
```

# シンボル

## [コメント](https://doc.rust-lang.org/book/appendix-02-operators.html#appendix-b-operators-and-symbols:~:text=Table%20B%2D7%20shows%20symbols%20that%20create%20comments.)

Rustのコメントは六種類ある。普通のコメントは別にドキュメンテーションコメントという埋め込みドキュメントを書くための機能がある。

普通のコメント

```rust
// ラインコメント(Line comment)

/*
	ブロックコメント
*/
```

ドキュメンテーションコメント

```rust
//! インナーラインドックコメント(Inner line doc comment)

/// アウターラインドックコメント(Outer line doc comment)

/*!
	インナーブロックドックコメント(Inner block doc comment)
*/

/**
	アウターブロックドックコメント(Outer block doc comment)
*/
```

### 参考

- [Rustのドキュメンテーションコメントの書き方](https://zenn.dev/masaki_wk/articles/20230715-rust-doc-comment)

## タプル