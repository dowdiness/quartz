---
title: MoonBitのtry?が使いやすくて最高
publish: false
tags: [" "]
aliases: [MoonBitのtry?が使いやすくて最高]
created: 2025-12-24T00:42:08+09:00
modified: 2025-12-24T11:37:23+09:00
---

# MoonBitのtry?が使いやすくて最高

こんにちは。MoonBit書いてますか？
私はMoonBitでラムダ計算のコンパイラを書いています。[^1]

https://github.com/dowdiness/tapl-rescript/tree/moon-migration/moonbit

まだ全然未完成なので今回これの解説はしませんが、MoonBitを書いていく中でエラー処理が便利で洗練されていてとても良かったので今回はこれについて語りたいと思います。

## MoonBitのOptionとResult

TypeScriptではなくMoonBitを使う理由にResult型があるはずです。
MooniBitは代数的データ型である `Enum` を持っており、ビルトインのデータ型として [Option](https://mooncakes.io/docs/moonbitlang/core/option) と [Result](https://mooncakes.io/docs/moonbitlang/core/result) を提供しています。

エラーが起こる可能性のあるプログラムを書くたびにResultに変換していくのは結構面倒です。Result型を返す式はそのために `Ok` と `Err` に包まって、この値を使う場合にはパターンマッチでResult型から取り出さなければいけません。

これはエラーを発生させて `try ... catch` を比べると手間がかかって面倒だと感じることもあると思います。

実はMoonBitのエラー処理の仕組みは優秀なのでこれに任せちゃいましょう。

## MoonBitでのエラー処理

MoonBitにおいてエラーの値は全て `Error` 型により表すことが出来ます。
ただしこのエラーは直接作ることは出来ず、エラーを発生させたい場合には具体的なエラー型を自分で作る必要があります。

```MoonBit
suberror E1 Int // Intをペイロードとして持つエラー型

suberror E2 // ペイロードを持たないエラー型

suberror E3 { // 普通の enum と同じように3つのコンストラクタを持つエラー型
  A
  B(Int, x~ : String)
  C(mut x~ : String, Char, y~ : Bool)
}
```

この `Error` は全てのエラー型の部分型となっており、パターンマッチングによりエラーの種類により処理を分けることが可能性です。`_` は全てのエラーに対応するワイルドカードであり、網羅性チェックのためにもエラーのパターンマッチングでは必ず使う必要があります。

```MoonBit
fn f(e : Error) -> Unit {
  match e {
    E2 => println("E2")
    A => println("A")
    B(i, x~) => println("B(\{i}, \{x})")
    _ => println("unknown error")
  }
}
```

エラーを起こす場合には raiseを使います

## try? によるResult型への変換

エラーを起こす可能性のある式の前に `try?` と書くことによりResult型の値へと変換する機能です。

公式サイトの例

```Moonbit
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

[^1]: [型システム入門](https://www.ohmsha.co.jp/book/9784274069116/)と[Essentials of Compilation](https://mitpress.mit.edu/9780262047760/essentials-of-compilation/)を参考にして書いています