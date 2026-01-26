---
title: Package manager
publish: false
tags: [programming-language]
aliases: [Managing Projects with Packages, Untitled]
created: 2025-05-14T13:23:09+09:00
modified: 2026-01-26T15:59:12+09:00
---

# Managing Projects with Packages

Moonbitのプロジェクト管理について
## Packages and modules

Moonbitのパッケージはソースコードファイルと `moon.pkg.json` 設定ファイルによって構成されます。パッケージには、エントリーポイントとして `main` 関数を持つ `main` パッケージ、あるいは他のパッケージからインポート可能なライブラリパッケージのいずれかがあります。



```json/moon.pkg.json
{
    "import": [
        "moonbit-community/language/packages/pkgA",
        {
            "path": "moonbit-community/language/packages/pkgC",
            "alias": "c"
        }
    ]
}
```

```Moonbit
pub fn add1(x : Int) -> Int {
  @c.incr(@pkgA.incr(x))
}
```

### Internal Packages

You can define internal packages that are only available for certain packages.

Code in `a/b/c/internal/x/y/z` are only available to packages `a/b/c` and `a/b/c/**`.

## アクセスコントロール

Moonbitでは、型や関数、トレイト（traits）の公開レベルを可視性(Visibility) 制御するために、いくつかのアクセスレベルが定義されています。これにより、モジュール間の依存性を整理し、インターフェースの設計を明確にできます。

外部のパッケージから見た型の可視性は以下の通りです：

- **private type**: `priv` パッケージ外からは全くアクセスできません。内部実装専用です。
- **abstract type**: デフォルトの可視性。型の名前は公開されますが、その中身（フィールドや実装）は非公開です。インターフェースとしてのみ利用可能です。
- **readonly types**: `pub` 型の内容を読み取り専用で公開します。インスタンスの生成や変更はできません。
- **fully public types**: pub(all) フィールドを含めてすべてが公開されており、外部から自由にインスタンス化や操作が可能です。

### traitsとsealed traitsのアクセス制限

トレイト（traits）やsealed traitsも同様にアクセスレベルを持ち、外部とのインターフェース設計において重要な役割を果たします。

- **Private traits**: `priv trait` 完全に非公開で、パッケージ内の型でのみ実装されます。
- **Abstract traits**: トレイト自体は公開されますが、実装の詳細は非公開です。ユーザーはこのトレイトを実装することはできません。
- **Readonly traits**: `pub trait` トレイトのメソッドなどは外部から読み取り専用でアクセス可能です。実装を拡張することはできません。
- **Fully public traits**: `pub(open) trait` トレイト自体が完全に公開されており、外部のパッケージでも自由に実装できます。

これらのアクセス制御を組み合わせることで、安全かつ柔軟なAPI設計が可能となります。