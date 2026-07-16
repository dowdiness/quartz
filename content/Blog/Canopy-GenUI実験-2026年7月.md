---
title: Canopy GenUI実験（2026年7月）
publish: true
tags: [blog, canopy, genui, projectional-editing]
created: 2026-07-16T13:35:00+09:00
modified: 2026-07-16T14:16:19+09:00
aliases: [Canopy GenUI実験（2026年7月）]
---

# Canopy GenUI実験（2026年7月）

2026年7月に Canopy で立ち上がった **generative UI（GenUI）** 実験の設計と現在地をまとめた記事。日次の作業ログは[[Canopy開発日誌-7月|7月の日誌]]を参照。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## GenUI とは何か

GenUI は、LLM からストリーミングで届く出力を **構造的に検証された JSX** として逐次パースし、差分 reconcile で DOM へ反映する実験である。Lambda・JSON・Markdown に続く JSX（7/11）が入ったあと、数日のうちにストリーミングセッション、重複 sibling 識別子の修正、決定論的な非同期ドライバ、ブラウザ障害回復スライスまで進んだ。

ここでの「UI」は、テキストをそのまま貼り付けるのではなく、**JSX という構造**として扱う点が Canopy の他機能と共通している。

## いま証明できていること

7/15 時点で、次の足場までは揃っている。

- ストリーミング JSX のパースと DOM reconcile
- dry-run / commit / recovery の境界
- 決定論的な非同期ドライバ（chunk・キャンセル・遅延到着の処理）
- Playwright によるブラウザ障害回復（14/14）

ただし **実プロバイダ（Gemini 等）との接続はまだ含まれていない**。いま動いているのは合成データと no-op 系の経路が中心である。

## 核心の問題意識：構文が正しいだけでは足りない

実験設計の出発点は次の一文にある。

> 構文的に妥当な commit は、value の証拠にならない。

つまり、LLM が返した JSX がパースに通ることと、「誰かが実際に使える UI であること」は別問題だ。プロバイダは no-op や無意味なパネル、タスクを満たさない妥当なテーブルを返すこともできる。現状のレンダラーは検証済みの `table` / `filter` / `summary` ノードを空の宣言的マーカーへ落とすだけで、実際に動く JSON/CSV Explorer は別の固定ホスト UI として存在している。

この状態のままプロバイダを繋いでも、測れるのは「候補の形」であって「使える UI」ではない。

## 次の実験で証明したいこと

実プロバイダの前に、次を前提条件として置いている。

**セッション所有の、1つだけの機能的 projection**——検証済み candidate が構造だけを選び、信頼されたホスト context がデータと操作状態を供給し、既存セッションの dry-run / commit / recovery 境界が可視 Explorer を唯一の DOM 所有者として持つこと。

## 却下した代替案

設計書では、採らなかった案も明記されている。

| 案 | 却下理由 |
| --- | --- |
| マーカー tree と別 Explorer の二重同期 | DOM を二重更新する分散トランザクションになり、focus / listener / custom-element 効果を DOM スナップショットで巻き戻せない |
| 描画せずスコアリングだけ | 「人が使えるか」を証明できず、継続/削除判断が骨抜きになる |
| ブラウザから直接 Gemini API | API キーはブラウザに露出する以上 secret にならない |
| 汎用プロバイダ IF の先行設計 | アダプタ 1 つでは renderer 非依存の不変条件を確立できない |

## 実プロバイダ実験のスコープ（設計のみ）

[Canopy PR #897](https://github.com/dowdiness/canopy/pull/897) の設計書では、次のように限定されている。

- プロバイダ実装・資格情報の使用は **まだ認可しない**（設計ドキュメントのみ）
- バックエンドは Gemini Developer API の `gemini-3.5-flash` を固定
- ストリーミングではなく一括のスキーマ制約付き JSON 生成
- ローカル限定の same-origin プロキシ経由
- 3 つの固定 fixture とブラインドな有用性評価
- コスト / レイテンシの上限

## 判断日：2026年7月29日

**2026/7/29** を継続 / 削除の判断日（kill date）として設けている。証拠が不十分なら **DELETE をデフォルト** とする。

この日を境に、Canopy が LLM 出力を直接構造編集の対象として本線に乗せるのか、足場だけ残して畳むのかが見える。

## 関連リンク

- 月次まとめ: [[Canopy開発日誌-7月-まとめ|7月まとめ]] §5
- 全文日誌: [[Canopy開発日誌-7月|7月の日誌]]（7/11 JSX、7/12〜7/15 GenUI）
- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [PR #897 設計書](https://github.com/dowdiness/canopy/pull/897)
