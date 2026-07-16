---
title: Canopy開発日誌-5月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:45:00+09:00
modified: 2026-07-16T10:45:00+09:00
---

# Canopy開発日誌-5月-まとめ

2分で読める月次まとめ。日々の詳細は[[Canopy開発日誌-5月|通常版の日誌]]を、英語版は[[Canopy-May-2026-Highlights|Highlights (English)]]を参照。もとの5月の日誌は生の作業メモに近い形で書かれているので、このまとめもその雰囲気を少し引き継いでいる。

> Canopyは、ソースコードを文字列ではなく構造（IR）として扱うエディタです。文字列を正として保ちつつ、そこから導出したプログラムの意味単位を直接操作することで、安全な構造編集やAI・複数人との協調作業がしやすくなります。詳しくは[[Canopyとは]]。

## 1. RabbitaとCodeMirrorがようやく安定

CanopyのUI層であるRabbitaとCodeMirrorをつなぐグルーコードは、それまで混沌としていてバグが多く、エディタを触っていると途中で動かなくなることがあった。このクラスのバグが月初に直った。単一の機能よりもこの修正のほうが重要だった――ランダムに固まるエディタの上には、何を作っても意味がない。

## 2. 隠しボタンが明示的な境界に

見えないDOMボタン経由で行われていた操作が、明示的なイベント購読と境界へ置き換わった。小さな変更だが、「エディタが今何をしているか」を推測するのではなく、追える対象にした。

## 3. InspectorとOp log

InspectorパネルとOp logが実装され、エディタの内部状態が「デバッグ時の推測」から「画面上で実際に見えるもの」になった。

## 4. incrのAPIを見直す

[Build Systems à la Carte](https://hackage.haskell.org/package/build)を読みながら、incrライブラリの評価モデルと公開APIを整理し直す作業が進んだ。この地道な作業が、2ヶ月後の7月に0.13.0・0.14.0という2つのAPI境界整理リリースへとつながっていく。

## 5. Cognition: AI文脈の土台

「Cognition」という、AI用ナレッジベース機能の最初の断片が現れた。ワークスペースの概念、コンテキストpacking、AIプロバイダへ渡す入力・返ってきた結果・キャンセルやretryを追跡するprovider boundary。エディタ自身の状態をAIが使えるコンテキストにする、という狙いはこの月にはまだ実を結ばないが、その土台がここから始まっている。

## 6. Lambdaに本物のナビゲーション

Lambdaのscope graphとgo-to-definitionが実装され、Ideal側のscope annotationも同じ解決結果へ寄っていった――これが7月に完成する名前解決統合の最初の兆しになる。あわせて、ephemeral presenceやbyte codecが共有部品として切り出され、incrのtyped spreadsheet demoとjs_engineのbytecode benchmarkという、それぞれの性能・実験用の入口も生まれた。

---

5月は、AIコンテキスト（Cognition）と性能計測という2つの流れが静かに始まった月。どちらもこの時点ではまだ実を結ばないが、後にどちらも中心的な役割を担うことになる。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/incr](https://github.com/dowdiness/incr) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-5月|5月の日誌]]
