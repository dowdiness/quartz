---
title: ls-lint
publish: false
tags: [utility]
created: 2025-05-21T22:42:10+09:00
modified: 2026-01-12T23:41:09+09:00
---

# ls-lint

https://ls-lint.org/

```yaml
ls:
  .js: kebab-case
  .ts: kebab-case
  .jsx: PascalCase
  .tsx: PascalCase
  .json: kebab-case
  .yml: kebab-case
  .yaml: kebab-case
  .md: kebab-case
  .sh: kebab-case
  .py: snake-case
  .go: snake-case
  .html: kebab-case
  .css: kebab-case
  .scss: kebab-case

  folder: kebab-case
  dockerfile: PascalCase
  env: UPPER_SNAKE_CASE
  license: UPPERCASE
  readme: PascalCase
  gitignore: lowercase
  gitkeep: lowercase

ignore:
  - node_modules
  - dist
  - build
  - .git
  - coverage
  - '*.min.js'

rules:
  kebab-case:
    regex: '^[a-z0-9]+(-[a-z0-9]+)*$'
  snake-case:
    regex: '^[a-z0-9]+(_[a-z0-9]+)*$'
  PascalCase:
    regex: '^[A-Z][a-zA-Z0-9]+$'
  camelCase:
    regex: '^[a-z][a-zA-Z0-9]+$'
  UPPER_SNAKE_CASE:
    regex: '^[A-Z0-9]+(_[A-Z0-9]+)*$'
  lowercase:
    regex: '^[a-z0-9.]+$'
  UPPERCASE:
    regex: '^[A-Z0-9]+$'
```

