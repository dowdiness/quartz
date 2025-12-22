---
title: Base
publish: false
tags: [" "]
aliases: [Untitled]
created: 2025-05-21T22:14:08+09:00
modified: 2025-12-22T20:47:56+09:00
---

# Quartz

## デプロイの仕方

Obsidianで書いた公開したい記事のFrontmatterのpublishをtrueに設定する。
[MainVault](https://github.com/dowdiness/MainVault/tree/main) へと記事の変更をプッシュする。

Quartzのフォルダ内で

```sh
git submodule update --remote
npx quartz sync 
```

を実行する。

