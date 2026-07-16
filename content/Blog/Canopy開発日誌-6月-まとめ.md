---
title: Canopy開発日誌-6月-まとめ
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-07-16T10:50:00+09:00
modified: 2026-07-16T18:14:26+09:00
---

# Canopy開発日誌-6月-まとめ

6月は、構造編集のデモを超えて「編集しても壊れないか」を検証し始めた月だった。NodeIdの保持、外部解析の取り込み、ビルド基盤の整理が並び、最終週にloomgenが加わる。[[Canopy開発日誌-6月|通常版の日誌]]・[[Canopy-June-2026-Highlights|Highlights (English)]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

## 1. 構造編集で意味が静かに変わることがなくなった

renameやmoveは動く。ところがbindingをlambdaの前へ動かすと、参照先だけがこっそり変わることがあった。scope graphでshadowingを捕まえ、編集後テキストを再パースして意図したASTになるかで検証する方式に切り替えた。

module直下に加え、block内のbindingでもrename・move・duplicate・extract-to-letが動くようになった。alpha-safe beta reductionの実験も、この流れの中で始まった。

## 2. 編集してもNodeIdが残る

`Var(x)`をlambdaで包むだけでNodeIdが消え、選択や折りたたみも一緒に失われる。構造は正しいのにUIの記憶だけが毀れる、という別種の不具合だ。

`IdentityTransform`とhinted reconcileで、操作の意図（wrap、unwrapで子を残す、など）を宣言できるようにした。hintを安全に出せない操作は、保守的に除外する。

## 3. Markdownが2番目の構造編集対象言語に

見出しやリスト項目に、壊れたparseをまたいでも残るidentityを持たせた。同一リスト内のmoveは、危ない動きを弾いたうえで許可する。SDEG（Structure-Directed Edit Grammar）のidentity側表は、Lambda以外への展開の足場になる。

## 4. 外部解析ツールを、CRDTを壊さずに取り込む

ast-grepの結果を永続状態に混ぜない。スナップショットに紐づく一時情報として取り込み、UTF-16 rangeへ直してdecorationに載せ、古ければ破棄する。text CRDTだけが残る。analysis query layerのPhase 1が、mainに入った。

## 5. js_engineのtest262適合率向上

v0.3.0を出したあと、test262を体系的に進めた。JSON.parseは142/142で100%。lexerは非BMP文字を含めてUTF-16正確になった。Promiseの仕様修正、tombstoneベースのSet iteration、timer queueの2.15倍高速化もこの時期だ。

## 6. ビルド基盤の整理

moon.mod.jsonからmoon.mod（TOML）へ全マニフェストを移行し、13 submoduleをworkspace memberにした。長年の依存解決回避策も片づいた。ベンチマークのリグレッションがPR gateに近づき、「skipしてgreen」は通用しなくなった。

## 7. 月末のloomgen誕生、incr移行、新エディタ2本

6/27〜6/30の4日間に、作業が重なる。loomにloomgenが加わり、incrは3日で`Memo`→`Derived`へ完全移行してレガシー`Signal`を削除した。Canopyではblock-editorのdrag-and-dropとJSON tree editorが、テスト付きで立ち上がった。

---

PR番号は#445から#783まで進んだ。月末にできたloomgenは、7月に手書きコード生成器を削除する転換へつながる。

- GitHub: [dowdiness/canopy](https://github.com/dowdiness/canopy) · [js_engine](https://github.com/dowdiness/js_engine) · [mooncakes.io/user/dowdiness](https://mooncakes.io/user/dowdiness)
- 全文: [[Canopy開発日誌-6月|6月の日誌]]
