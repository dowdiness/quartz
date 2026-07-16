---
title: Canopy開発日誌-5月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:45:00+09:00
modified: 2026-07-16T16:10:00+09:00
---

# Canopy開発日誌-5月-まとめ

4月にできた骨格の上で、次はすぐ新機能を足す——そう見えていた。5月に先に来たのは、接続の安定化とUnicode基盤の整備だった。活動は5/7から。日々の記録は[[Canopy開発日誌-5月|通常版の日誌]]、英語版は[[Canopy-May-2026-Highlights|Highlights (English)]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. Unicode正しさと編集履歴

Canopyの活動は5/7から。SyncEditorにcausal snapshotを足し、Idealでは編集の因果関係をGraphvizで見るhistory viewの土台を作った。

並行してissue #216のUnicode正しさ監査が始まり、`moji`（UAX #29）の導入、MarkdownのZWSP処理、bridgeのposition単位整理が進んだ。編集応答benchmark、Canvas、Intent panelもこの時期だ。派手な機能ではないが、あとから名前解決や外部解析を載せる土台になる。

## 2. RabbitaとCodeMirrorがようやく安定

RabbitaとCodeMirrorをつなぐグルーコードは、それまで触っていると突然動かなくなることがあった。新機能の検証より先に、接続の安定化が来た。

ここが固まってようやく、後半のInspector整備やCognitionの実験に手を伸ばせた。エディタが途中で固まるとそれ以降の作業はすべて止まる——5月前半の判断は、その教訓から出ている。

## 3. 内部状態を推測から追跡へ

見えないDOMボタン経由の操作をやめ、明示的なイベント購読へ置き換えた。エディタが今何をしているか、推測ではなく追跡できるようになった。

デバッグ時に推測に頼っていた内部状態は、InspectorパネルとOp logで画面上から見えるようにした。「触ったら動く」から「なぜ動いたか説明できる」へ、観測の置き方が変わった。

## 4. incrの評価モデルを見直す

[Build Systems à la Carte](https://hackage.haskell.org/package/build)を読みながら、incrの評価モデルと公開APIを整理した。spreadsheet demoやbyte codecの切り出しもこの時期に増えた。

地味に見えるが、2ヶ月後の0.13.0・0.14.0へそのままつながる。7月の破壊的リリースと再入ガードは、この月の見直しの延長線上にある。

## 5. Cognition: AIが読むエディタの断片

「Cognition」と名付けたAI用ナレッジベースの断片が現れた。ワークスペース、コンテキストpacking、provider boundary——エディタの状態をAIが使える形にする狙いだ。

5月の時点ではまだ実を結んでいない。7月のGenUI実験に至る「エディタと生成UI」の問いの、もっと早い芽生えでもある。

## 6. Lambdaに本物のナビゲーション

scope graphとgo-to-definitionを実装し、Idealのscope annotationも同じ解決へ寄せた。7月の`@scope`一本化の前段階になる。

ephemeral presenceの整理やjs_engineのbytecode benchmarkも並行して進んだ。構造編集エディタに、言語サーバっぽい「跳ぶ」操作が初めて載った月だ。

---

前半はUnicode・moji・Intent panel。後半はRabbita接続・Cognition・scope graph。6月のNodeId保持やSDEG展開も、この月の地基の上に載る。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/incr](https://github.com/dowdiness/incr) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-5月|5月の日誌]]
