---
title: JavaScriptにおけるプリミティブとオブジェクトの違い
tags: [JavaScript]
created: 2025-04-27T16:09:02+09:00
modified: 2025-12-22T20:07:55+09:00
---

# JavaScriptにおけるプリミティブとオブジェクトの違い

JavaScriptの値には主に2種類のデータ型があります：プリミティブ型とオブジェクト型です。

## プリミティブ型（基本型）[^1]

プリミティブ型は最も基本的なデータ型で、以下の7種類があります：

1. **文字列（String）** - 例：`"こんにちは"`, `'JavaScript'`
2. **数値（Number）** - 例：`42`, `3.14`
3. **論理値（Boolean）** - `true` または `false`の論理値
4. **undefined** - 値が割り当てられていない変数の初期値
5. **null** - 値が存在しないことを意図的に表す特殊値
6. **シンボル（Symbol）** - ES2015で導入された一意で不変の値
7. **BigInt** - 大きな整数を扱うためのデータ型

プリミティブな値はImmutable(不変)であり、一度作成したあとの変更は出来ません。プリミティブにはメソッドがありませんがラッパーオブジェクト[^2]の仕組みによりあたかもメソッドを持っているかのように振る舞うことが出来ます。

## オブジェクト型（参照型）[^3]

オブジェクト型はより複雑なデータ構造で、プロパティとメソッドを持つことができます：

1. **オブジェクト（Object）** - 例：`{name: "田中", age: 30}`
2. **配列（Array）** - 例：`[1, 2, 3, 4]`
3. **関数（Function）** - 例：`function() { return "Hello"; }`
4. **日付（Date）** - 例：`new Date()`
5. **正規表現（RegExp）** - 例：`/pattern/`
6. **Map**, **Set**, **WeakMap**, **WeakSet**
7. その他の組み込みオブジェクト

オブジェクト型の特徴：

- ミュータブル（可変）であり、作成後に内容を変更できます
- 参照によって比較されます（同じ内容でも異なるオブジェクトは等しくない）
- ヒープメモリに格納されます
- プロパティとメソッドを持つことができます

## 等価性の比較と同一性

2 つの値を比較する場合に値のプリミティブ型とオブジェクト型の違いは重要になってきます。

等価性アルゴリズムが４つあります。

## 脚注

[JS Comparison Table](https://dorey.github.io/JavaScript-Equality-Table/)

[^1]: [Primitive (プリミティブ) - MDN Web Docs 用語集](https://developer.mozilla.org/ja/docs/Glossary/Primitive)
[^2]: [ラッパーオブジェクト · JavaScript Primer](https://jsprimer.net/basic/wrapper-object/)
[^3]: [JavaScript のデータ型とデータ構造 - MDN オブジェクト](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Data_structures#%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88)