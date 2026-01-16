---
title: React
publish: false
tags: [frontend, javascript, react]
created: 2025-05-10T01:34:37+09:00
modified: 2026-01-15T15:23:52+09:00
---

# React

## 内部設計

[react-fiber-architecture](https://github.com/acdlite/react-fiber-architecture)

[React - Basic Theoretical Concepts](https://github.com/reactjs/react-basic)

[React Internals Deep Dive](https://jser.dev/series/react-source-code-walkthrough)

[Paul O Shannessy - Building React From Scratch](https://youtu.be/_MAD4Oly9yg?si=hftkERr-6wsqxy-M)

[Getting Closure on React Hooks](https://www.swyx.io/hooks)

## Compound Components

[Building Type-Safe Compound Components](https://tkdodo.eu/blog/building-type-safe-compound-components)

## Skills

[writing-react-effects](https://www.reddit.com/r/reactjs/comments/1pxv4lf/i_made_a_decision_tree_to_stop_myself_from/)

## Tearing

concurrent rendering によってレンダリングが中断された場合に起こりえる、同じ状態を参照していても画面上では異なるバージョンの状態を示してしまう現象です。Reactのレンダリング中に参照する状態が一貫性のない状態になると発生します。

[What is tearing?](https://github.com/reactwg/react-18/discussions/69)

`useSyncExternalStore` を使えば外部のストアとReactの状態（とUI更新）をSync出来ます。しかしこれはconcurrent renderingで使うと tearing を起こす可能性があります。

### 参考

- [use-valtio](https://github.com/valtiojs/use-valtio?tab=readme-ov-file#but-why)
- [Will this React global state work in concurrent rendering?](https://github.com/dai-shi/will-this-react-global-state-work-in-concurrent-rendering)
- [React 18 for External Store Libraries](https://youtu.be/oPfSC5bQPR8?si=mDk6zQFAxTir0Cn0)
- [React Tearing in Jotai and Reducer](https://github.com/pmndrs/jotai/discussions/2752)

