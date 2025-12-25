---
title: Tree-sitter
publish: true
tags: [compiler, parser]
created: 2025-06-02T21:42:25+09:00
modified: 2025-12-24T12:09:06+09:00
---

# Tree-sitter

https://tree-sitter.github.io/tree-sitter

使い方が分かりづらいですが、 `tree-sitter` の CLI tool を入れて、

```sh
tree-sitter init
```

をした後は `grammar.js` を目的の文法に合わせて書き換えていくだけでよい。

沢山ファイルが生成されて気になるが、基本的にこれらのファイルは人の手で直接触る必要はない。