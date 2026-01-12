---
aliases: [何故Atomicな状態管理が必要なのか?]
created: 2025-02-11T13:15:39+09:00
modified: 2026-01-12T21:41:31+09:00
---

# Atomicな状態管理の実現方法

https://jotai.org/
https://github.com/jotaijs/jotai-location

# 何故Atomicな状態管理が必要なのか?

https://www.youtube.com/watch?v=_ISAA_Jt9kI&t=3s

離れたコンポーネント同士でも状態の共有がしたい
関連性の無いコンポーネント同士でUIの描写に関わる変数を共有して、その変数の更新に合わせてUIの更新がしたい。

Event base

react hooksを使っているコンポーネントが無くなったら
WeakMap

同じフックが使われている場合に

この記事はJotaiの [Core internals](https://jotai.org/docs/guides/core-internals) を参考にしています。そちらをご覧下さい。
# Reactでの状態管理の基本

Reactで状態管理をする際には基本的に `useState` か `useReducer` のどちらかを利用すると思います。何故これらの関数は必要なのでしょうか？ローカル変数では実現できないのか？

**ユーザ操作に合わせてコンポーネントを新しいデータで更新して画面上の表示内容を変更するためです。**

公式ドキュメントの [state：コンポーネントのメモリ](https://ja.react.dev/learn/state-a-components-memory#when-a-regular-variable-isnt-enough) では通常のローカル変数では上手くいかない例とともに、この機能を実現するために必要な要件が明記されています。

> コンポーネントを新しいデータで更新するためには、次の 2 つのことが必要です。
> 
> 1. レンダー間でデータを**保持**する。
> 2. 新しいデータでコンポーネントをレンダー（つまり再レンダー）するよう React に**伝える**。
> 
> [`useState`](https://ja.react.dev/reference/react/useState) フックは、これら 2 つの機能を提供します。
> 
> 1. レンダー間でデータを保持する **state 変数**。
> 2. 変数を更新し、React がコンポーネントを再度レンダーするようにトリガする **state セッタ関数**。

レンダー間での(コンポーネント内での)データの保持と、新しいデータでの再レンダーです

Reactの文脈においてのレンダーの意味はコンポーネント(関数)の実行であって、ブラウザのDOMへの変更やそれに伴う画面の再描画ではありません。詳しくは [レンダーとコミット](https://ja.react.dev/learn/render-and-commit) を参照してください。

```js
const [state, setState] = useState(initialState)
```

# useAtomの実現に必要な要件

```jsx
import { atom } from 'atomic-library';

const countAtom = atom(0); // 初期値0のAtomを作成
```

```jsx
import { useAtom } from 'atomic-library';
import { countAtom } from './atoms';

function Counter() {
  const [count, setCount] = useAtom(countAtom);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

function App() {
  
  return(
	<div>
	  <Counter />
	  
	</div>
  )
}
```

外部に状態を持つ。

`useAtom` のセッタ関数の実行に合わせて同じAtomを使っているコンポーネント全てにおいて`useState` のセッタ関数を実行する。

[Observer pattern](https://zenn.dev/morinokami/books/learning-patterns-1/viewer/observer-pattern) という実装方法があります。決められたデータやイベントの変化に基づいて事前に登録した関数を実行したい時に使える手法です。

PubSubとも

データと表示内容を更新する

# メモリーの最適化

[Memory leak when using atomFamily/selectorFamily](https://github.com/facebookexperimental/Recoil/issues/366)

https://zenn.dev/akfm/scraps/2bc577076ef30c

https://zenn.dev/jotaifriends/articles/0c1f4c3a6ed7e5

Recoilでは`Map`を使っているのが、Jotaiでは`WeakMap`を使っています。

WeakMap

歓迎要件

使われなくなったAtomの自動削除によるメモリーの最適化。