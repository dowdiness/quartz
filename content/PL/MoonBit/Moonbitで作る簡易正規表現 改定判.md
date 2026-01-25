---
title: Moonbitで作る簡易正規表現 改定判
publish: true
tags: [moonbit, 正規表現]
aliases: [Moonbitで作る簡易正規表現 v2, Moonbitで作る簡易正規表現 改訂版]
created: 2025-06-10T15:45:33+09:00
modified: 2026-01-25T18:58:11+09:00
---

# Moonbitで作る簡易正規表現 v2

Claude 4で改良したバージョン

## MoonBit良いよね☺️

MoonBitは新しいプログラミング言語として、現在もコア機能の更新が頻繁に行われています。言語仕様が完全に固まった安定版のリリースまでには、まだ時間がかかるかもしれません。

しかし、現段階でも既にMoonBitは現代的な言語として、開発者の「痒いところに手が届く」使いやすく便利な言語です。言語側でテストやドキュメント、ベンチマークなどはサポートされていて、文法も素直で直感的なものになっています。WebAssemblyにコンパイルされる言語として宣伝されていますが（されていた？）、実際には出力先としてはJavaScriptとC言語、また実験的ですがLLVMが選べます。どこのプラットフォームでも動くポータブルなコードとして、アプリのコアロジックを書くのに適した言語ではないでしょうか？

## 正規表現について

新しい言語を覚えるとしたら取り敢えずそれで何か作ってみたいですよね？ということで、今回はMoonBitに慣れるためにも試しに正規表現エンジンをMoonbitで作ることにしましょう。[^1]

今回作る正規表現は言語に組み込みで付いてくるような本格的なものではなく機能を削った簡易的なものです。実装方法は再帰呼び出しを使用したバックトラッキング方式のオーソドックスなものになります。NFAからDFA変換だったり、VM使ってどうのこうのみたいな話は出てきません。[^2] (なんのコッチャ？と思ったそこのあなた！君こそが理想の想定読者です。そういう話が知りたくて来たのになーと思ったそこのあなた！あなたは既に私より詳しいのでさっさと帰ってください[^3])

また正規表現とは何か？という基本的な話はしません。知らない読者はWikipediaやMDNなどを参考にするかLLMにでも聞いてみてください。[^4]

## 記事の構成

まず最初に完成形のコードを示します。

```moonbit
fn regex_match(
  pattern : String,
  text : String,
  anchoredStart~ : Bool = false,
  anchoredEnd~ : Bool = false
) -> Bool {
  // 行の最初 (^) と 最後に ($) マッチする処理
  if pattern.length() > 0 && pattern.has_prefix("^") {
    regex_match(
      // 先頭の文字 (^) をスキップ
      pattern.substring(start=1),
      text,
      anchoredStart=true,
      anchoredEnd~,
    )
  } else if pattern.length() > 0 && pattern.has_suffix("$") {
    regex_match(
      // 最後の文字（$）をスキップ
      pattern.substring(start=0, end=pattern.length() - 1),
      text,
      anchoredStart~,
      anchoredEnd=true,
    )
  } else {
    // Define a helper that matches pattern at the beginning of 't',
    // requiring end of text if anchoredEnd
    fn inner_Match(p : String, t : String, end_anchor : Bool) -> Bool {
      match (p.view(), t.view()) {
        // If anchored end, text must be empty. Otherwise success.
        (p, t) if p.length() == 0 => not(end_anchor) || t.length() == 0
        // Handle ? quantifier: preceding char is optional (optional)
        ([p0, '?', ..], [t0, ..]) =>
          // Try skipping the char (0 occurrences)
          if inner_Match(p.substring(start=2), t, end_anchor) {
            true
          } else {
            // Try using it once
            match (t.length() > 0 && p0 == t0) || p0 == '.' {
              true =>
                inner_Match(
                  p.substring(start=2),
                  t.substring(start=1),
                  end_anchor,
                )
              false => false
            }
          }
        // Handle * quantifier (zero or more)
        ([p0, '*', ..], [t0, ..]) =>
          // Zero occurrences: if pattern is "a*bc", it becomes "bc" after skipping the first two characters
          if inner_Match(p.substring(start=2), t, end_anchor) {
            true
          } else {
            // One or more: check if we have at least one character that matches
            // For example: if pattern is "b*c" and text is "bbbc",
            // after one loop the text becomes "bbc" but the pattern stays the same
            match (t.length() > 0 && p0 == t0) || p0 == '.' {
              true => inner_Match(p, t.substring(start=1), end_anchor)
              false => false
            }
          }
        (_, t) if t.length() == 0 => false
        ([p0, ..], [t0, ..]) if p0 == t0 || p0 == '.' =>
          inner_Match(p.substring(start=1), t.substring(start=1), end_anchor)
        (_, _) => false
      }
    }
    // If anchored start, only try at position 0
    if anchoredStart {
      return inner_Match(pattern, text, anchoredEnd)
    }
    // Otherwise, try each starting position
    for i = 0; i <= text.length(); i = i + 1 {
      if inner_Match(pattern, text.substring(start=i), anchoredEnd) {
        return true
      }
    }
    return false
  }
}

test {
  assert_eq!(regex_match("hello", "hello"), true)
  assert_eq!(regex_match("hello", "hell"), false)
  assert_eq!(regex_match("h.llo", "hello"), true)
  assert_eq!(regex_match("h.llo", "hllo"), false)
  assert_eq!(regex_match("colou?r", "color"), true)
  assert_eq!(regex_match("colou?r", "colouur"), false)
  assert_eq!(regex_match("ab*c", "abbbc"), true)
  assert_eq!(regex_match("lo$", "hello"), true)
  assert_eq!(regex_match("^hi", "ahihi"), false)
  assert_eq!(regex_match("ab*c", "abc"), true)
}
```

初めにそのままの文字列にマッチングするリテラルマッチングから初めていき、少しずつ対応する特殊文字を増やしていきます。

今回対応する特殊文字は下記の５つです。

|特殊文字|意味|
|---|---|
|.|任意の1文字|
|?|直前の文字は省略可能(0か1)|
|*|直前の文字の0回以上の繰り返し(0..)|
|^|行の先頭|
|$|行の最後|

エスケープなどもありません。（だって記事が長くなるから...）

MoonBitは公式のブラウザで動く [Playground](https://try.moonbitlang.com/) を用意しています。手元で開発環境を用意するのが億劫な場合にはこちらのサイトでサンプルコードを実行しながらお読みください。

# リテラルマッチングの対応

正規表現パターン(以下パターンと呼ぶ)と対象文字列(以下テキストと呼ぶ)を引数に取り、一致したら `true` を、しなかった場合は `false` を返す関数になります。

```moonbit
fn regex_match(pattern : String, text : String) -> Bool {
  match (pattern.view(), text.view()) {
    // パターンが空文字列だった場合, テキストが空文字列かどうかでBoolを返す
    (p, t) if p.length() == 0 => t.length() == 0
    // パターンが空文字列でなくテキストが空文字列だった場合、falseを返す
    (_, t) if t.length() == 0 => false
    // ２つの文字列(String)の先頭の文字(Char)が同じだった場合
    ([p0, ..], [t0, ..]) if p0 == t0 =>
      // 先頭の文字を削除して再帰呼び出し
      regex_match(pattern.substring(start=1), text.substring(start=1))
    // どのパターンにも一致しなかった場合、falseを返す
    (_, _) => false
  }
}

test {
  // Literal match: exact equality
  assert_eq!(regex_match("hello", "hello"), true) // identical strings match
  assert_eq!(regex_match("hello", "hell"), false) // text shorter than pattern fails
  assert_eq!(regex_match("hello", "hello!"), false) // extra text fails
}
```

ここでMoonBitの文法について少し説明します。

パターンマッチング構文は以下のような形になります：

```moonbit
match パターンマッチングする対象 {
  pattern1 if 条件 => 処理1
  pattern2 => 処理2
  ...
}
```

MoonBitには [Array Pattern](https://docs.moonbitlang.com/en/latest/language/fundamentals.html#array-pattern) という機能があり、`view()` を使って文字列を文字の配列のように扱うことができます。`view` は他の言語でいうsliceに当たるもので、配列のような連続したデータに対する可変長なポインターのようなものです。

`(pattern.view(), text.view())` により [tuple](https://docs.moonbitlang.com/en/latest/language/fundamentals.html#tuple) を作り、それをパターンマッチングしています。

`[p0, ..]` という表記は、「先頭の文字を `p0` に束縛し、残りの部分は無視する」という意味です。

この実装では文字列が完全に一致する場合のみ `true` を返します。正規表現ではパターンがテキストの一部に一致すれば成功とみなすことが多いですが、まずは基本的な構造を理解するため、完全一致から始めています。

# ワイルドカード(.)への対応

次に、任意の1文字にマッチする `.` (ドット)を追加しましょう。

```moonbit
fn regex_match(pattern : String, text : String) -> Bool {
  match (pattern.view(), text.view()) {
    // パターンが空文字列だった場合, テキストが空文字列かどうかでBoolを返す
    (p, t) if p.length() == 0 => t.length() == 0
    // パターンが空文字列でなくテキストが空文字列だった場合、falseを返す
    (_, t) if t.length() == 0 => false
    // ドット(.)は任意の1文字にマッチ
    (['.', ..], [_, ..]) =>
      regex_match(pattern.substring(start=1), text.substring(start=1))
    // ２つの文字列(String)の先頭の文字(Char)が同じだった場合
    ([p0, ..], [t0, ..]) if p0 == t0 =>
      // 先頭の文字を削除して再帰呼び出し
      regex_match(pattern.substring(start=1), text.substring(start=1))
    // どのパターンにも一致しなかった場合、falseを返す
    (_, _) => false
  }
}

test {
  // Literal match: exact equality
  assert_eq!(regex_match("hello", "hello"), true)
  assert_eq!(regex_match("hello", "hell"), false)
  assert_eq!(regex_match("hello", "hello!"), false)
  
  // Wildcard match: . matches any single character
  assert_eq!(regex_match("h.llo", "hello"), true) // . matches 'e'
  assert_eq!(regex_match("h.llo", "hallo"), true) // . matches 'a'
  assert_eq!(regex_match("h.llo", "hllo"), false) // . must match something
  assert_eq!(regex_match(".", "a"), true) // . matches any single char
  assert_eq!(regex_match(".", ""), false) // . requires a character
}
```

`.` は任意の1文字にマッチするため、テキストに何らかの文字があれば（つまりテキストが空でなければ）マッチ成功として、両方の文字列から1文字ずつ削除して再帰呼び出しを行います。

# 0か1回(?)への対応

`?` は直前の文字が0回または1回現れることを表します。つまり「あってもなくてもよい」という意味です。

```moonbit
fn regex_match(pattern : String, text : String) -> Bool {
  match (pattern.view(), text.view()) {
    // パターンが空文字列だった場合, テキストが空文字列かどうかでBoolを返す
    (p, t) if p.length() == 0 => t.length() == 0
    // ? quantifier: 直前の文字は省略可能
    ([p0, '?', ..], t) =>
      // まず文字が0回現れる場合を試す（スキップ）
      if regex_match(pattern.substring(start=2), text) {
        true
      } else {
        // 1回現れる場合を試す
        match t {
          [t0, ..] if (p0 == t0 || p0 == '.') && t.length() > 0 =>
            regex_match(pattern.substring(start=2), text.substring(start=1))
          _ => false
        }
      }
    // パターンが空文字列でなくテキストが空文字列だった場合、falseを返す
    (_, t) if t.length() == 0 => false
    // ドット(.)は任意の1文字にマッチ
    (['.', ..], [_, ..]) =>
      regex_match(pattern.substring(start=1), text.substring(start=1))
    // ２つの文字列(String)の先頭の文字(Char)が同じだった場合
    ([p0, ..], [t0, ..]) if p0 == t0 =>
      // 先頭の文字を削除して再帰呼び出し
      regex_match(pattern.substring(start=1), text.substring(start=1))
    // どのパターンにも一致しなかった場合、falseを返す
    (_, _) => false
  }
}

test {
  // Literal match
  assert_eq!(regex_match("hello", "hello"), true)
  assert_eq!(regex_match("hello", "hell"), false)
  
  // Wildcard match
  assert_eq!(regex_match("h.llo", "hello"), true)
  assert_eq!(regex_match("h.llo", "hllo"), false)
  
  // ? quantifier: 0 or 1 occurrence
  assert_eq!(regex_match("colou?r", "color"), true) // u appears 0 times
  assert_eq!(regex_match("colou?r", "colour"), true) // u appears 1 time
  assert_eq!(regex_match("colou?r", "colouur"), false) // u appears 2 times
  assert_eq!(regex_match("a?b", "b"), true) // a appears 0 times
  assert_eq!(regex_match("a?b", "ab"), true) // a appears 1 time
}
```

`?` の処理では、まず直前の文字を0回とする場合（つまりスキップする場合）を試します。それが失敗した場合に、1回とする場合を試します。この順序は重要で、貪欲でない（non-greedy）マッチングを実現しています。

# 0回以上の繰り返し(*)への対応

`*` は直前の文字が0回以上現れることを表します。

```moonbit
fn regex_match(pattern : String, text : String) -> Bool {
  match (pattern.view(), text.view()) {
    // パターンが空文字列だった場合, テキストが空文字列かどうかでBoolを返す
    (p, t) if p.length() == 0 => t.length() == 0
    // * quantifier: 0回以上の繰り返し
    ([p0, '*', ..], t) =>
      // まず0回の場合を試す（文字をスキップ）
      if regex_match(pattern.substring(start=2), text) {
        true
      } else {
        // 1回以上の場合を試す
        match t {
          [t0, ..] if (p0 == t0 || p0 == '.') && t.length() > 0 =>
            // テキストから1文字消費し、パターンはそのまま（さらなる繰り返しのため）
            regex_match(pattern, text.substring(start=1))
          _ => false
        }
      }
    // ? quantifier: 直前の文字は省略可能
    ([p0, '?', ..], t) =>
      // まず文字が0回現れる場合を試す（スキップ）
      if regex_match(pattern.substring(start=2), text) {
        true
      } else {
        // 1回現れる場合を試す
        match t {
          [t0, ..] if (p0 == t0 || p0 == '.') && t.length() > 0 =>
            regex_match(pattern.substring(start=2), text.substring(start=1))
          _ => false
        }
      }
    // パターンが空文字列でなくテキストが空文字列だった場合、falseを返す
    (_, t) if t.length() == 0 => false
    // ドット(.)は任意の1文字にマッチ
    (['.', ..], [_, ..]) =>
      regex_match(pattern.substring(start=1), text.substring(start=1))
    // ２つの文字列(String)の先頭の文字(Char)が同じだった場合
    ([p0, ..], [t0, ..]) if p0 == t0 =>
      // 先頭の文字を削除して再帰呼び出し
      regex_match(pattern.substring(start=1), text.substring(start=1))
    // どのパターンにも一致しなかった場合、falseを返す
    (_, _) => false
  }
}

test {
  // Literal match
  assert_eq!(regex_match("hello", "hello"), true)
  assert_eq!(regex_match("hello", "hell"), false)
  
  // Wildcard match
  assert_eq!(regex_match("h.llo", "hello"), true)
  assert_eq!(regex_match("h.llo", "hllo"), false)
  
  // ? quantifier
  assert_eq!(regex_match("colou?r", "color"), true)
  assert_eq!(regex_match("colou?r", "colour"), true)
  assert_eq!(regex_match("colou?r", "colouur"), false)
  
  // * quantifier: 0 or more occurrences
  assert_eq!(regex_match("ab*c", "ac"), true) // b appears 0 times
  assert_eq!(regex_match("ab*c", "abc"), true) // b appears 1 time
  assert_eq!(regex_match("ab*c", "abbbc"), true) // b appears 3 times
  assert_eq!(regex_match("a.*c", "abc"), true) // .* matches any string
  assert_eq!(regex_match("a.*c", "axyzc"), true) // .* matches "xyz"
}
```

`*` の処理では、まず0回の場合（文字をスキップ）を試します。それが失敗した場合、1文字を消費しますが、パターンはそのまま保持します。これにより、同じ文字の繰り返しを処理できます。

重要なポイントは、`*` の場合にパターンから `p0*` の部分を削除しないことです。これにより、さらなる繰り返しが可能になります。

# 部分マッチングへの対応

これまでの実装では完全一致のみを扱っていましたが、通常の正規表現では文字列の一部にパターンがマッチすれば成功とします。この機能を追加しましょう。

```moonbit
fn regex_match(pattern : String, text : String) -> Bool {
  // 内部的なマッチング関数：文字列の開始位置からのマッチを試す
  fn match_here(p : String, t : String) -> Bool {
    match (p.view(), t.view()) {
      // パターンが空文字列だった場合、成功
      (p, _) if p.length() == 0 => true
      // * quantifier: 0回以上の繰り返し
      ([p0, '*', ..], t) =>
        // まず0回の場合を試す（文字をスキップ）
        if match_here(p.substring(start=2), t) {
          true
        } else {
          // 1回以上の場合を試す
          match t {
            [t0, ..] if (p0 == t0 || p0 == '.') && t.length() > 0 =>
              // テキストから1文字消費し、パターンはそのまま
              match_here(p, t.substring(start=1))
            _ => false
          }
        }
      // ? quantifier: 直前の文字は省略可能
      ([p0, '?', ..], t) =>
        // まず文字が0回現れる場合を試す（スキップ）
        if match_here(p.substring(start=2), t) {
          true
        } else {
          // 1回現れる場合を試す
          match t {
            [t0, ..] if (p0 == t0 || p0 == '.') && t.length() > 0 =>
              match_here(p.substring(start=2), t.substring(start=1))
            _ => false
          }
        }
      // パターンが空文字列でなくテキストが空文字列だった場合、falseを返す
      (_, t) if t.length() == 0 => false
      // ドット(.)は任意の1文字にマッチ
      (['.', ..], [_, ..]) =>
        match_here(p.substring(start=1), t.substring(start=1))
      // ２つの文字列(String)の先頭の文字(Char)が同じだった場合
      ([p0, ..], [t0, ..]) if p0 == t0 =>
        // 先頭の文字を削除して再帰呼び出し
        match_here(p.substring(start=1), t.substring(start=1))
      // どのパターンにも一致しなかった場合、falseを返す
      (_, _) => false
    }
  }
  
  // テキストの各位置からマッチを試す
  for i = 0; i <= text.length(); i = i + 1 {
    if match_here(pattern, text.substring(start=i)) {
      return true
    }
  }
  return false
}

test {
  // 部分マッチングのテスト
  assert_eq!(regex_match("ell", "hello"), true) // "ell" matches in "hello"
  assert_eq!(regex_match("lo", "hello"), true) // "lo" matches at end
  assert_eq!(regex_match("he", "hello"), true) // "he" matches at start
  assert_eq!(regex_match("xyz", "hello"), false) // "xyz" doesn't match anywhere
  
  // 既存の機能も動作することを確認
  assert_eq!(regex_match("h.llo", "hello world"), true) // wildcard with partial match
  assert_eq!(regex_match("wo?rld", "hello world"), true) // optional with partial match
  assert_eq!(regex_match("l*o", "hello"), true) // repetition with partial match
}
```

ここで重要な変更は、パターンが空になったときに `true` を返すようにしたことです（完全一致では両方が空の場合のみ `true` でした）。そして、メインの関数でテキストの各位置からマッチを試すループを追加しました。

# 境界アサーション(^, $)への対応

最後に、行の開始(`^`)と終了(`$`)を表すアンカーを追加します。これらは文字を消費せず、位置を指定する特殊な文字です。

```moonbit
fn regex_match(
  pattern : String,
  text : String,
  anchoredStart~ : Bool = false,
  anchoredEnd~ : Bool = false
) -> Bool {
  // 行の最初 (^) と 最後に ($) マッチする処理
  if pattern.length() > 0 && pattern.has_prefix("^") {
    regex_match(
      // 先頭の文字 (^) をスキップ
      pattern.substring(start=1),
      text,
      anchoredStart=true,
      anchoredEnd~,
    )
  } else if pattern.length() > 0 && pattern.has_suffix("$") {
    regex_match(
      // 最後の文字（$）をスキップ
      pattern.substring(start=0, end=pattern.length() - 1),
      text,
      anchoredStart~,
      anchoredEnd=true,
    )
  } else {
    // 内部的なマッチング関数
    fn match_here(p : String, t : String, end_anchor : Bool) -> Bool {
      match (p.view(), t.view()) {
        // パターンが空の場合：end_anchorがtrueなら文字列も空である必要がある
        (p, t) if p.length() == 0 => not(end_anchor) || t.length() == 0
        // * quantifier: 0回以上の繰り返し
        ([p0, '*', ..], t) =>
          // まず0回の場合を試す
          if match_here(p.substring(start=2), t, end_anchor) {
            true
          } else {
            // 1回以上の場合を試す
            match t {
              [t0, ..] if (p0 == t0 || p0 == '.') && t.length() > 0 =>
                match_here(p, t.substring(start=1), end_anchor)
              _ => false
            }
          }
        // ? quantifier: 直前の文字は省略可能
        ([p0, '?', ..], t) =>
          // まず0回の場合を試す
          if match_here(p.substring(start=2), t, end_anchor) {
            true
          } else {
            // 1回の場合を試す
            match t {
              [t0, ..] if (p0 == t0 || p0 == '.') && t.length() > 0 =>
                match_here(p.substring(start=2), t.substring(start=1), end_anchor)
              _ => false
            }
          }
        // パターンが残っているのにテキストが空の場合
        (_, t) if t.length() == 0 => false
        // ドット(.)は任意の1文字にマッチ
        (['.', ..], [_, ..]) =>
          match_here(p.substring(start=1), t.substring(start=1), end_anchor)
        // 文字が一致する場合
        ([p0, ..], [t0, ..]) if p0 == t0 =>
          match_here(p.substring(start=1), t.substring(start=1), end_anchor)
        // どのパターンにも一致しない場合
        (_, _) => false
      }
    }
    
    // anchoredStartがtrueの場合、位置0からのみマッチを試す
    if anchoredStart {
      return match_here(pattern, text, anchoredEnd)
    }
    
    // そうでなければ、各位置からマッチを試す
    for i = 0; i <= text.length(); i = i + 1 {
      if match_here(pattern, text.substring(start=i), anchoredEnd) {
        return true
      }
    }
    return false
  }
}
```

 が見つかると `anchoredEnd=true` に設定され、パターンが終了したときにテキストも終了している必要があります。

# まとめ

以上で、簡易的な正規表現エンジンが完成しました。実装した機能をまとめると：

1. **リテラルマッチング**: 文字通りの文字列マッチング
2. **ワイルドカード(.)**: 任意の1文字にマッチ
3. **オプション(?)**: 直前の文字が0回または1回
4. **繰り返し(*)**: 直前の文字が0回以上
5. **部分マッチング**: パターンがテキストの一部にマッチ
6. **境界アサーション**: 行の開始(^)と終了($)

この実装はバックトラッキング方式を使用しており、理解しやすい一方で、複雑なパターンに対しては性能の問題が生じる可能性があります[^5]。本格的な正規表現エンジンでは、NFAやDFAへの変換、あるいはVM-based approaches などのより効率的な手法が使われています。

MoonBitの特徴的な機能として、パターンマッチング、Array Pattern、view、名前付き引数などを活用することで、読みやすく保守しやすいコードを書くことができました。

# より詳しく学ぶために

正規表現エンジンの実装についてより深く学びたい場合は、以下のリソースがおすすめです：

- [Regular Expression Matching Can Be Simple And Fast](https://swtch.com/~rsc/regexp/) by Russ Cox
- [Crafting Interpreters](https://craftinginterpreters.com/) by Robert Nystrom
- [Automata Theory An Algorithmic Approach](https://michaelblondin.com/automata/) by _[Javier Esparza](https://www7.in.tum.de/~esparza/) and [Michael Blondin](https://mblondin.espaceweb.usherbrooke.ca/)_

MoonBitについてより詳しく知りたい場合は、[公式ドキュメント](https://docs.moonbitlang.com/)を参照してください。

[^1]: 本当は [Theory of Computation](https://ocw.mit.edu/courses/18-404j-theory-of-computation-fall-2020/) の授業を見たからと、 [Russ Cox](https://swtch.com/~rsc/regexp/) の記事を読んで正規表現を自作してみたくなったからです。もちろん私は新しい言語を覚えるなら最初にやることはやっぱり [JSON パーサ](https://zenn.dev/mattn/articles/3b01651b7a42b3) か [ Lisp インタプリタ](https://github.com/kanaka/mal) だよね～などと言いだすような狂人ではありません。本当です。

[^2]: 何故なら解説出来るほど詳しくないので😢。詳しくは [Russ Cox](https://swtch.com/~rsc/regexp/) の記事を読んでください。

[^3]: というか正直難しくて分からないことが大量にあるので教えてほしい。誰か[正規表現](https://gihyo.jp/book/2015/978-4-7741-7270-5)や [オートマタの本](https://michaelblondin.com/automata/) を一緒に読んでくれませんか？

[^4]: どうやら正規表現とは何かを簡潔に説明するのは難しいらしく、検索すると奥歯に物が挟まったようなふにゃふにゃした正規表現の説明を沢山見つけることが出来ます。可愛いね。

[^5]: 先読みや後方参照をサポートしたものは正確には正規表現ではなく、[NLOG](https://cyberagent.ai/blog/research/19391/)と呼ばれているらしいです。また、バックトラッキング方式は最悪の場合指数時間になる可能性があるため、本格的な実装では状態機械ベースのアプローチが好まれます。
```