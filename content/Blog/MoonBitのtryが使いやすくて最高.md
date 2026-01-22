---
title: MoonBitのtry?が使いやすくて最高
publish: false
tags: [blog]
aliases: [MoonBitのtry?が使いやすくて最高なので知ってほしい, MoonBitのtry?が使いやすくて最高]
created: 2025-12-24T00:42:08+09:00
modified: 2026-01-22T08:19:52+09:00
---

# MoonBitのtry?が使いやすくて最高なので知ってほしい

こんにちは。MoonBit書いてますか？
私は今、MoonBitでラムダ計算のコンパイラを書いています。[^1]

実装はこちらです。まだ全然未完成ですが、言語処理系の練習として作っています。
https://github.com/dowdiness/tapl-rescript/tree/moon-migration/moonbit

今回はこの実装自体ではなく、書いていて特に良いと感心した**MoonBit のエラー処理の設計**、とくに `try?` がとても使いやすいという話をしたいと思います。

## MoonBit の Option と Result

TypeScript ではなく MoonBit を使う理由の一つに、`Result` 型の存在があると思います。[^2]
MooniBitには `Struct` と `Enum` による代数的データ型があり、ビルトインのライブラリとして [Option](https://mooncakes.io/docs/moonbitlang/core/option) と [Result](https://mooncakes.io/docs/moonbitlang/core/result) が提供されています。

一方で、エラーが起こりうる処理をすべて `Result` で表現していくと、どうしても記述量が増えがちです。Resultを返す関数では成功時は `Ok(...)` に、失敗時は `Err(...)` に値を包んで返す必要があり、この値を使う場合には毎回パターンマッチでResult型から中身を取り出さなければなりません。

これはエラーを発生させて `try ... catch` でまとめて例外処理するスタイルと比べると、手間がかかって面倒だと感じるかもしれません。

しかし MoonBit では、こういったエラー処理を楽に書くための機能があります。

---

## MoonBit におけるエラー型

MoonBit では、すべてのエラーは `Error` 型として扱われます。  
ただし `Error` を直接生成することはできず、実際には **自分でエラー型を定義して使う** ことになります。

公式サイトよりの例:

```MoonBit
// https://docs.moonbitlang.com/en/latest/language/error-handling.html

suberror E1 { E1(Int)}   // Int をペイロードとして持つエラー型
suberror E2              // ペイロードを持たないエラー型

suberror E3 {            // 普通の enum と同じように複数のコンストラクタを持てる
	A
	B(Int, x~ : String)
	C(mut x~ : String, Char, y~ : Bool)
}
```

これらはすべて `Error` の部分型になっており、型変換出来ますパターンマッチによって分岐できます。

```MonnBit
fn f(e : Error) -> Unit {
	match e {
		E2 => println("E2")
		A => println("A")
		B(i, x~) => println("B(\{i}, \{x})")
		_ => println("unknown error")
	}
}
```

`_` はすべてのエラーにマッチするワイルドカードで、複数のエラー型を網羅性チェックの観点からも `Error` のパターンマッチングでは基本的に入れておく必要があります。

エラーを発生させたい場合は `raise` を使います。
つまり MoonBit には、

- 例外的に制御を中断する仕組み（`raise`）
- それを型安全に扱う仕組み（`Error` と部分型）

の両方が用意されています。

---

## `try?` による Result への変換

ここで本題の `try?` です。

`try?` は、**エラーを投げうる式を `Result` 型の値に変換する構文** です。  
つまり、成功すれば `Ok(value)` 途中で `raise` されたら `Err(error)` という形に自動で変換してくれます。

公式ドキュメントの例：

```MoonBit
test {
	let res = try? (div(6, 0) * div(6, 3))
	inspect(
		res,
		content=(
			#|Err("division by zero")
		),
	)
}
```

ここでポイントなのは、

- `div(6, 0)` はエラーを raise する
- でも `try?` が付いているのでプログラムは中断されず
- そのまま `Err(...)` として値に変換される

という点です。

つまり MoonBit では、

- 普段は例外スタイルで素直に処理を書き
- 境界で `try?` を使って `Result` に落とす

という書き方ができます。

これによって、

- 内部は読みやすく直線的なコード
- 外部との境界では型でエラーを表現

という、かなり理想的な分離ができます。

---

## 例外と Result の「いいとこ取り」

Result 型は安全だけど冗長になりがちで、  
例外は楽だけど型に出てこない、というのはよくある話ですが、

MoonBit の `raise` + `try?` の組み合わせは、

- 実装側は例外っぽく書けて
- API 境界では Result にできる

という、かなり実用的な折衷案になっていると感じます。

個人的には、

> Result を返したいけど、全部 Result で書きたくはない

という気持ちにちょうど刺さる設計で、とても気に入っています。

[^1]: [型システム入門](https://www.ohmsha.co.jp/book/9784274069116/)と[Essentials of Compilation](https://mitpress.mit.edu/9780262047760/essentials-of-compilation/)を参考にして書いています

[^2]: こういった型を知らない方は [Railway Oriented Programming](https://fsharpforfunandprofit.com/rop/) が参考になるでしょう

ちなみに見た目はRustの[?演算子](https://doc.rust-lang.org/std/result/#the-question-mark-operator-)に似ていますが、やっていることはかなり違います。Rustの `?` はResultを返す関数内で式の最後に `?` を付けた場合、評価結果がErrならば外側の関数の返り値としてそのErrを早期リターンとして返し、Okの場合はOkの中身を取り出した値を返すものです。MoonBitの `!try` はエラーが出る可能性のある式をResult型の値へ変換する機能です。