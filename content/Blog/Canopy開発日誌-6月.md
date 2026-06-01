---
title: Canopy開発日誌-6月
publish: false
tags: [projectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-06-02T01:25:58+09:00
---

# Canopy開発日誌-6月

2026年6月のCanopy開発ログ。

6月1日は、5月末に進めたLambdaのscope graphやgo-to-definitionを、実際の構造編集へ近づける作業が中心だった。
同時に、CanopyとMoonDspの関係、性能改善の優先度、エージェントの使い分けも整理した。

## 2026/6/1

### Canopy

まず、CanopyとMoonDspの長期的な接続構想を整理した。
MoonDspは別リポジトリの音楽DSL / DSPエンジンなので、Canopyへすぐ統合する対象ではない。

今は、Canopyは構造編集の土台、MoonDspは音楽DSL側の実験、incr / Loomは共有基盤として分けて考えることにした。
関連PR: [#445](https://github.com/dowdiness/canopy/pull/445)

Lambda projectionでは、小さな整理を先に進めた。
`ProjNode` の構築helper、SourceMap token helper、Block / Holeのtyped viewなどを使い、projection処理の重複を減らした。

その後、module-level bindingをfirst-classな `LetDef` projection nodeとして扱う変更を入れた。
以前はbinding rowがinit expressionの `NodeId` やsynthetic idを借りていたので、Structure modeのdrag/dropやbinding-level editで責務が曖昧だった。

`LetDef` が実際のnodeになったことで、binding rowを構造編集の対象として扱いやすくなった。
binderの位置取得は、引き続き `@scope.binder_span` / `@scope.go_to_definition` が担当する。
関連PR: [#437](https://github.com/dowdiness/canopy/pull/437)、[#438](https://github.com/dowdiness/canopy/pull/438)、[#439](https://github.com/dowdiness/canopy/pull/439)、[#440](https://github.com/dowdiness/canopy/pull/440)、[#446](https://github.com/dowdiness/canopy/pull/446)、[#448](https://github.com/dowdiness/canopy/pull/448)

性能面では、BAND 2bのevidence gateを走らせた。
ここでは最適化を書かず、まず本当に遅いのかを測った。

`to_flat_proj_incremental` は1000 defsで数msまで伸びることが分かった。
ただし、実際のCanopy内のLambda documentはまだそこまで大きくない。

そのため、この最適化は一旦parkした。
大きなdocumentやimport機能が出てきたら、改めて取り組む。
関連PR: [#447](https://github.com/dowdiness/canopy/pull/447)

依存関係まわりでは、rle submoduleの更新と、examples配下のVite / Playwright / React系の更新も行った。
PlaywrightはpackageだけでなくCI containerも合わせる必要があった。
関連PR: [#435](https://github.com/dowdiness/canopy/pull/435)、[#280](https://github.com/dowdiness/canopy/pull/280)

### loom

Loomでは `examples/json-settings` に、last-good semantic projection attachmentのchecked exampleを追加した。
docsにあったtemplateを、実際にテストできるexampleへ落とした形。

ポイントは、`@incr.Derived` のcomputeをpureに保ち、last-goodの保持を外側の `settle` に任せたこと。
`Watch::read()` の `ReadError` も、parser/projection failureとは分けて `GraphBlocked` として扱うようにした。
関連PR: [loom #206](https://github.com/dowdiness/loom/pull/206)

Lambda example側ではBlock / Holeのtyped syntax viewを追加し、その後 `LetDef` term wrapperを入れた。
Canopyのfirst-class `LetDef` projection nodeはこのLoom側の変更を受けて進んでいる。
関連PR: [loom #207](https://github.com/dowdiness/loom/pull/207)、[loom #209](https://github.com/dowdiness/loom/pull/209)

incr dependencyも整理した。
Loomの3 consumerをregistryの `dowdiness/incr@0.7.0` へ寄せた。

Canopy側の追従も見たが、loom配下のincr submoduleに別作業が残っていたので一旦pauseした。
関連PR: [loom #208](https://github.com/dowdiness/loom/pull/208)

### js_engine

js_engineでは、benchmarkを読みやすくする作業を進めた。
focused repeat benchmark runnerを追加し、複数回の測定からmedianやCVを見られるようにした。

単発の結果ではなく、ノイズを見ながら判断するための道具になっている。
関連PR: [js_engine #186](https://github.com/dowdiness/js_engine/pull/186)、[js_engine #187](https://github.com/dowdiness/js_engine/pull/187)

startup側では、`new_interpreter` の中を分けて測り、final realm stampingを軽くした。
runtimeで作られるiterator callbackやPromise callbackにもrealmを付ける整理をした。
関連PR: [js_engine #188](https://github.com/dowdiness/js_engine/pull/188)、[js_engine #192](https://github.com/dowdiness/js_engine/pull/192)

Arrayまわりでは、`sort` や `splice` などのmutatorで、明示的な `undefined` とholeの扱いがずれる問題を直した。
genericなArray mutatorでは、Proxy receiverを考慮して `HasProperty` 相当の経路を使う必要がある、というメモも残した。
関連PR: [js_engine #191](https://github.com/dowdiness/js_engine/pull/191)

### 作業運用

Claude / Codex / piの使い分けも整理した。

長い実装計画はCodexが書き、Claude/Opusは設計と実行管理を持つ。
Codexはpre-PR reviewだけでなく、実装順序や検証点を含む計画を書く役割も持つ。

この運用はLoomのjson-settings exampleで最初に使い、`GraphBlocked` stateのような設計漏れも拾えた。
