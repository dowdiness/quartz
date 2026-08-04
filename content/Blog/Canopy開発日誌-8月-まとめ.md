---
title: Canopy開発日誌-8月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-08-04T23:10:00+09:00
modified: 2026-08-04T23:14:31+09:00
---

# Canopy開発日誌-8月-まとめ

7月末に完成した診断基盤とCommonMark実装を土台に、新しいプロダクトが生まれ、js_engineは大規模な書き換えキャンペーンに入った。日々の記録は[[Canopy開発日誌-8月|通常版の日誌]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. loomがCommonMarkを完成させる

月初、7月から続いていたMarkdown実装が一気に完成した。raw HTMLポリシー、entity参照のデコード、autolinkとinline HTML、link/image参照の解決——CommonMarkのブロック・インライン構文がほぼ1日（8/1）で揃っている。

完成後は間を置かず性能作業へ移った。1万行文書の性能envelope測定、block再パース後のtoken buffer保持、reference lookaheadのprefix-linear化を経て、差分再パースの単位をkeyで安定させる「reactive keyed MarkdownIR shell」まで到達した。月の前半で「正しく動く」から「差分更新に載る」への切り替えを終えている。

## 2. Loomarkという新しいプロダクトが生まれる

8/1、「Loomark Markdown editorのための純粋なアプリケーションコア」という説明を持つ独立パッケージが立ち上がった。7/15にCanopy本体から切り出した`js_ffi`・`dom_boundary`が、ここで初めて「Canopyを土台にした別プロダクト」を生む形で使われている。

その後の3日間で、typed Markdown foundations → private dev host → **ローカルファーストなドキュメント所有権の設計** → ブロックエディタprototypeと、驚くほどの速さで育った。所有権の設計は特に丁寧で、「Loomarkは今、Markdown文書を編集できるが所有はできない」という問題意識から出発し、「テキストではなく操作履歴を永続化する」という1行の原則を立て、Codex（GPT-5）による3回の独立レビューと、動くprobeによる検証（アサーションを意図的に壊してから戻す較正手順込み）を経て固めている。8/4にはRUIによるMarkdownプレビュー、ブロック整形ツールバー、ブロックエディタの一連のインタラクションが揃い、prototypeと呼べる段階に達した。

## 3. incrがv0.15.0をリリース

typed-sheetアプリケーションの所有権整理とWatch上のrooted reader統合を経て、scope-owned post-GC maintenanceを実装し、0.15.0をリリースした（8/3）。7月のtyped spreadsheetコラボレーション実験（Cloudflare Workers上のリアルタイム協調編集）を支えた基盤の延長線上にある。

## 4. js_engineが「インタプリタ全体のスタックセーフ化」に入る

7/29に決めた「部分トランポリン」方針が、7/31から本格的なキャンペーンへ発展した。GitHub issueごとに`codex/issue-NNN`というブランチを立て、別々のコーディングエージェントが担当してmainへマージしていく形で、**4日間で約300コミット**が積み上がっている。

対象はordinary call・spread call・constructor・receiver/member access・`Array.prototype.map`・`Function.prototype.apply`・timer/event loopと多岐にわたるが、"suspend"（中断可能にする）→"test with RED"（先に失敗するテストを書く）→"fix"という手順と、`docs(stack-safety)`での契約明文化は一貫している。狙いは「ゲスト側の再帰深さに関わらずホスト呼び出し深さは一定に保つ」という不変条件を、静的検証からランタイム実行そのものへ広げること。月初の4日間だけでこの密度は、この一ヶ月がjs_engineにとって大きな書き換えの月になることを示している。

## 5. Canopy本体はワークスペースを再編する

Loomark誕生とdom-boundary/js_ffiの独立を受け、8/4にワークスペースディレクトリを大規模に再編した。foundation・runtime・Rabbita・domain・editor adapter・application slice・primary Canopyモジュールを、依存境界を強制するテストを添えて配置し直す22コミットの移動になっている。モノレポとしての形を、新しく生まれたプロダクト群に合わせて整え直す動きが続いている。

---

CommonMarkの完成とLoomarkの誕生が、月初の4日間で連鎖的に起きた。js_engineのスタックセーフ化キャンペーンは規模が大きく、この先も月の重心になりそうだ。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [dowdiness/loom](https://github.com/dowdiness/loom) · [dowdiness/incr](https://github.com/dowdiness/incr) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-8月|8月の日誌]]
