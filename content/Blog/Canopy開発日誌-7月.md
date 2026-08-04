---
title: Canopy開発日誌-7月
publish: true
tags: [blog, canopy, projectional-editing]
created: 2026-01-04T20:50:52+09:00
modified: 2026-08-04T21:50:00+09:00
---

# Canopy開発日誌-7月

2026年7月のCanopy開発ログ（日次記録）。月の要約は[[Canopy開発日誌-7月-まとめ|7月-まとめ]]。

> ソースコードを構造（IR）として編集する MoonBit 製エディタ。概要は[[Canopyとは]]。

全体方針: [MoonDsp / Canopy ecosystem vision](https://github.com/dowdiness/canopy/blob/main/docs/research/2026-06-01-moondsp-canopy-ecosystem-vision.md)

## 今月の大きな流れ

月全体の流れは[[Canopy開発日誌-7月-まとめ|7月-まとめ]]にまとめた。以下は週次ヘッダと日付見出しによる作業記録。PR番号の一覧は文末の[PR索引](#pr索引)にある。GenUI の設計詳細は[[Canopy-GenUI実験-2026年7月|別記事]]。

## 7月第1週: incr破壊的リリースとloomgen拡張（7/1〜7/6）

Idealのレシピ化とloomgenの`.loomgrammar`追加から始まった週。7/3に`@scope`一本化と`EditError`移行、7/4〜7/5でincr 0.13.0・0.14.0が連続リリース。js_engineは非同期iterationのtest262が100%に届く。

## 2026/7/1

### Canopy / Ideal UIリファクタとincr追従

Idealまわりの「レシピ化」リファクタが4件まとまった。

- **`cx()` joinヘルパー**: ボタン・タブで繰り返していた条件付きCSSクラス結合を切り出し、button/tabのレシピへ適用（[#822](https://github.com/dowdiness/canopy/pull/822)）。
- **`ui/panel.mbt`**: パネルのクロムを`PanelVariant{Default,Inspector}`として抽出（[#825](https://github.com/dowdiness/canopy/pull/825)）。
- **`ui/menu_item.mbt`**: action overlayの項目トーンを`MenuItemTone{Normal,Danger}`として抽出（[#827](https://github.com/dowdiness/canopy/pull/827)）。
- `rename_binding_by_id`内の手書き宣言探索も共有ヘルパーへ整理（[#821](https://github.com/dowdiness/canopy/pull/821)）。

incrがv0.12.0へ上がったのに合わせ、CanopyもSignal→Input、map_eq→mapへの追従移行を行った（[#826](https://github.com/dowdiness/canopy/pull/826)）。6月末に完了したincr側のMemo→Derived移行とSignal削除の、Canopy側の受け皿となる。

### loom / loomgenの入力形式拡張

loomgenに`.loomgrammar`という単独ファイル形式を追加した（[#547](https://github.com/dowdiness/loom/pull/547)）。これまでの`#loom.rule`アノテーションと同じ規則本体記法を、MoonBitソースに埋め込まず独立ファイルとして書けるようにした。アノテーション出力とバイト単位で一致することをdifferential testで確認している。

そのほかloomgen周りの細かい修正:

- アノテーション引数が不正なときに「アノテーションが見つからない」と誤報告していたバグを修正（[#550](https://github.com/dowdiness/loom/pull/550)）
- ホスト側コードへ再突入するための`Native(RuleName)`というIR脱出ハッチノードを追加。HTMLのタグスタック照合のような文脈依存処理を、生成された文法から呼び出せるようにした（[#551](https://github.com/dowdiness/loom/pull/551)）
- lexer_skeleton.g.mbtのモードディスパッチ骨格生成器（[#553](https://github.com/dowdiness/loom/pull/553)）とCIツールチェーンのフォーマット差異修正（[#552](https://github.com/dowdiness/loom/pull/552)）

主なPR / Issue: canopy [#821](https://github.com/dowdiness/canopy/pull/821), [#822](https://github.com/dowdiness/canopy/pull/822), [#825](https://github.com/dowdiness/canopy/pull/825), [#826](https://github.com/dowdiness/canopy/pull/826), [#827](https://github.com/dowdiness/canopy/pull/827) / loom [#547](https://github.com/dowdiness/loom/pull/547), [#550](https://github.com/dowdiness/loom/pull/550), [#551](https://github.com/dowdiness/loom/pull/551), [#552](https://github.com/dowdiness/loom/pull/552), [#553](https://github.com/dowdiness/loom/pull/553)

## 2026/7/2

### Canopy / CI障害の修復

前日の[#820](https://github.com/dowdiness/canopy/pull/820)（moon.mod移行+E2E分離）で、web-buildステップの`run:`行が誤って落ちていたことが判明した。`- name:`だけで`run:`も`uses:`もないstepはGitHub Actions上でワークフロー全体が無効になる扱いになり、6/30以降の全CIが「0ジョブ実行」で失敗し続けていた。これをその日のうちに復旧した（[#833](https://github.com/dowdiness/canopy/pull/833)）。あわせて監査駆動のクリーンアップも行い、未追跡docsの整理、3件の実行計画（typed EditError移行・ideal orchestration抽出・lambda名前解決統合）の追加、`@wire`に置き換えられ不要になった`sync_protocol.mbt` shimの削除を行った（[#832](https://github.com/dowdiness/canopy/pull/832)）。

### loom / incr pin整合とloomgenの初消費

前日のCI障害の原因はloom側のpinのずれにもあり、incr 0.11.0→0.12.0への追従とdeny-warnエラー修正を2段構えで行った（[#566](https://github.com/dowdiness/loom/pull/566), [#568](https://github.com/dowdiness/loom/pull/568)）。`.loomgrammar`のヘッダー先読みにあった括弧深さのバグ（かっこの中の`IDENT '='`を次の規則のヘッダーと誤認する）も、複数エージェントによるレビューで見つけて直した（[#555](https://github.com/dowdiness/loom/pull/555)）。

lambda exampleが、loomgen `--spec`出力の最初の実コンパイル消費者になった（[#558](https://github.com/dowdiness/loom/pull/558)）。調査の結果、当初の設計は誤った前提（結合enumがすでにloomgenの出力そのものであることに気づいていなかった）に基づいていたことが分かり、配線するだけで済んだ。lambdaのトークンenum重複も削除した（[#567](https://github.com/dowdiness/loom/pull/567)）。

主なPR / Issue: canopy [#832](https://github.com/dowdiness/canopy/pull/832), [#833](https://github.com/dowdiness/canopy/pull/833) / loom [#555](https://github.com/dowdiness/loom/pull/555), [#558](https://github.com/dowdiness/loom/pull/558), [#566](https://github.com/dowdiness/loom/pull/566), [#567](https://github.com/dowdiness/loom/pull/567), [#568](https://github.com/dowdiness/loom/pull/568)

## 2026/7/3

### Canopy / 名前解決統合とtyped EditError

Lambdaの名前解決を`@scope`へ一本化した（[#839](https://github.com/dowdiness/canopy/pull/839)、#129を閉じる）。編集層に重複していた4つの関数（`free_vars`+`collect_var_usages`+`init_ref_resolves`+`free_name_would_rebind_to`）を、scope graphに対する1つのクエリ`references_outside_subtree`へ置き換え、scope.mbtから113行を削った。22件の差分テストで、旧`free_vars`とscope graphの結果が一致することを確認している。

編集層のエラーをString依存から型付きの`EditError`（12 variants）へ移す作業も、lambda・JSON・Markdownの3言語すべてとLanguageSpecの実行時契約、editor境界にまたがって完了した（[#840](https://github.com/dowdiness/canopy/pull/840)）。Markdownの`compute_move_block.mbt`が最も手間のかかる部分で、9個のprivateヘルパーと20以上のエラーパスを扱った。

### Canopy / 定数畳み込みミドルウェアの追加と撤回

同じ日に、興味深い設計判断があった。`EditMiddleware`パイプラインに`FoldConstants`ミドルウェアを追加し、`Bop(_, Int(a), Int(b))`のようなノードへの`CommitEdit`を横取りして`1+2`を`3`へ畳み込む機能を入れた（[#842](https://github.com/dowdiness/canopy/pull/842)）。ところが数時間後、これを撤回した（[#843](https://github.com/dowdiness/canopy/pull/843)）。理由は、`EditMiddleware`はguard（検証・アクセス制御・監査ログ）のためのものであり、AST変換のためのものではない、という原則に立ち返ったこと。破壊的な簡約をミドルウェアで行うと、CRDT履歴上のユーザーの元の式（`1 + 2`と書いた意図そのもの）が失われてしまう。この問題はすでにe-graph最適化器と評価annotationで非破壊的に解決されている（ホバーで畳み込み結果が見える。ソーステキストは変わらない）。TODO §8にこの設計上の結論を書き残した。

### Canopy / その他

大きめのPRとして、**Idealのorchestrationロジックを`examples/ideal/main`（6,008行・29ファイル）からライブラリパッケージへ抽出するStep 2-8**を進めた（[#845](https://github.com/dowdiness/canopy/pull/845)）。structural edit opからtree edit opへの変換をcompanionへ、scope annotationを`lang/lambda/scope`へ、action context構築を`lang/lambda/edits`+companionへ、それぞれ移し、Rabbita/DOM/CodeMirrorの型がライブラリパッケージへ漏れていないことを確認した。

その他の小さな変更: outlineツリーの矢印キー押下ごとのナビゲーションパスをキャッシュし初回押下だけO（n）のDFSにする性能改善（[#844](https://github.com/dowdiness/canopy/pull/844)）、`ReconcileTraceEvent`の発行をreconcileパイプラインへ配線（[#846](https://github.com/dowdiness/canopy/pull/846)）、37件の古い実行計画の整理（[#849](https://github.com/dowdiness/canopy/pull/849)）、eg-walker/loomのsubmodule bump連鎖とそれに伴うLayoutコンストラクタの3引数化対応（[#850](https://github.com/dowdiness/canopy/pull/850), [#851](https://github.com/dowdiness/canopy/pull/851), [#852](https://github.com/dowdiness/canopy/pull/852), [#853](https://github.com/dowdiness/canopy/pull/853)）。

### incr / 0.13.0への布石とincr_tea独立

incrの互換API surfaceを一括削除する0.13.0破壊的リリースの実行計画が固まった（[#341](https://github.com/dowdiness/incr/pull/341), [#342](https://github.com/dowdiness/incr/pull/342)）。ワークスペース境界とpinの整合をチェックする仕組み（[#347](https://github.com/dowdiness/incr/pull/347), [#348](https://github.com/dowdiness/incr/pull/348)）も入った。

TEAフレームワークを`examples/incr_tea`から独立ワークスペースモジュール`dowdiness/incr_tea`へ切り出した（[#349](https://github.com/dowdiness/incr/pull/349)）。incr本体のfacadeだけをimportする形で34ファイルを移動し、識別ADR（[#350](https://github.com/dowdiness/incr/pull/350)）では、incr_teaはあくまで実験的なもので、公開もされておらず0.13.0の安定性保証の対象でもなく、Rabbitaの置き換えでもないと明記した。目的は、実際のUIワークロードでincrの公開facadeに圧をかけ続けること。

### js_engine

前日直した§10.2.11 TDZ事前チェックの適用範囲を広げ、call/generatorの分割代入パラメータにも同様のTDZ検出を追加した（[#484](https://github.com/dowdiness/js_engine/pull/484)）。docs更新も（[#483](https://github.com/dowdiness/js_engine/pull/483)）。

### loom / GrammarIRのAST-builder backend

grammar emitter向けにAST-builder backendを追加し（[#573](https://github.com/dowdiness/loom/pull/573)）、grammar-parityのfixture再生成と`--regenerate-fixtures`フラグ（[#580](https://github.com/dowdiness/loom/pull/580)）、`root`という名前の規則が自動生成のエントリポイント関数と衝突し無限再帰を起こす問題のガード（[#581](https://github.com/dowdiness/loom/pull/581)）、pretty printerの`flat_width`キャッシュによるO（1）Group判定（[#572](https://github.com/dowdiness/loom/pull/572)）が入った。

そのうえで、**`emit_grammar_module()`を「レンダー前のASTを返す」形へ抽出**し、脆い`source.contains()`文字列チェックの代わりに構造的な`derive(Eq)`アサーションでテストできるようにした（[#584](https://github.com/dowdiness/loom/pull/584)）。ADR（[#587](https://github.com/dowdiness/loom/pull/587)）は、この文字列マッチングが抱えていた2種類の脆さを名指ししている。ひとつは`@pretty.render_string`の出力が`moon fmt`の期待と食い違う「フォーマット結合」、もうひとつは`source.contains("fn parse_foo")`が「どこかにその部分文字列がある」ことしか保証しない「暗黙のアサーション」。golden-fileテストも同じレンダリング経路を通る以上この問題を共有するとして退け、snapshotテストも当時MoonBitのテストフレームワークにその基盤がなかったため見送っている。フォーマットカスケードのsubmodule bumpは、`moon fmt`がeg-walker内の必要なimport/aliasを削ってしまい`check-strict`を壊したため撤回した（[#588](https://github.com/dowdiness/loom/pull/588)）。

主なPR / Issue: canopy [#839](https://github.com/dowdiness/canopy/pull/839), [#840](https://github.com/dowdiness/canopy/pull/840), [#842](https://github.com/dowdiness/canopy/pull/842), [#843](https://github.com/dowdiness/canopy/pull/843), [#844](https://github.com/dowdiness/canopy/pull/844), [#845](https://github.com/dowdiness/canopy/pull/845), [#846](https://github.com/dowdiness/canopy/pull/846), [#849](https://github.com/dowdiness/canopy/pull/849), [#850](https://github.com/dowdiness/canopy/pull/850), [#851](https://github.com/dowdiness/canopy/pull/851), [#852](https://github.com/dowdiness/canopy/pull/852), [#853](https://github.com/dowdiness/canopy/pull/853) / incr [#341](https://github.com/dowdiness/incr/pull/341), [#342](https://github.com/dowdiness/incr/pull/342), [#347](https://github.com/dowdiness/incr/pull/347), [#348](https://github.com/dowdiness/incr/pull/348), [#349](https://github.com/dowdiness/incr/pull/349), [#350](https://github.com/dowdiness/incr/pull/350) / js_engine [#483](https://github.com/dowdiness/js_engine/pull/483), [#484](https://github.com/dowdiness/js_engine/pull/484) / loom [#569](https://github.com/dowdiness/loom/pull/569), [#572](https://github.com/dowdiness/loom/pull/572), [#573](https://github.com/dowdiness/loom/pull/573), [#580](https://github.com/dowdiness/loom/pull/580), [#581](https://github.com/dowdiness/loom/pull/581), [#584](https://github.com/dowdiness/loom/pull/584), [#587](https://github.com/dowdiness/loom/pull/587), [#588](https://github.com/dowdiness/loom/pull/588)

## 2026/7/4

### Canopy / Patch panelとincr 0.13.0追従

Ideal editorのPatch panelに`StructuredChange`イベント表示を配線した（[#854](https://github.com/dowdiness/canopy/pull/854)）。**配線の過程で実バグも見つけた**――`build_projection_memos`の`reconcile_node`が、`trace_ref`が`Some`のときは常にデフォルト（ヒントなし）の`reconcile`へ切り替えてしまい、トレース有効時にLambdaのヒントベース再構成が丸ごと無効化されていた。incr 0.13.0（loomより1日遅れ）への再pinも完了し、削除された名前を使っているコードがゼロだったためpinのみの変更で済んだ（[#855](https://github.com/dowdiness/canopy/pull/855)、wasm-gc 3689/3689）。

そのほか、`ReconcileTraceEvent`発行のプロパティ/スナップショットテスト11件（130/130、[#856](https://github.com/dowdiness/canopy/pull/856)）、`PatchEntry`ごとのトレーススナップショット追加（Patch panelの履歴表示がその時点のdiffを正しく示すように、[#859](https://github.com/dowdiness/canopy/pull/859)）、agent-memoryのdocsポリシー修正（[#857](https://github.com/dowdiness/canopy/pull/857)）、[#845](https://github.com/dowdiness/canopy/pull/845)のdoc-lag解消（[#860](https://github.com/dowdiness/canopy/pull/860)）が続いた。

### incr / 0.13.0破壊的リリース

issue #345の決定に基づき、互換API surfaceを段階的な非推奨化を経ずに直接0.13.0として削除した（[#351](https://github.com/dowdiness/incr/pull/351)）。`TrackedCell`は`InputField`へ、`Reactive`は`EagerDerived`へ、`FunctionalRelation`は`MapRelation`へ、`Database`/`Readable` traitは削除。`moon bench --release`のbefore/afterで検証し、pushホットパスは24%高速化、他の見かけ上の劣化はマシンノイズと確認した。

### js_engine / regex性能とnative build修正、lookbehind実装

`regex_search`の位置ループで発生していたタイムアウトを、候補位置スキャンの最適化で解消した（[#485](https://github.com/dowdiness/js_engine/pull/485)）。静的に絞り込める先頭文字（リテラル・文字クラス・最小回数>0の量指定子）を持つパターンでは、`advance_to_candidate`が一気に先へ進めるようになり、110万位置の非マッチスキャンが実質1回の走査に短縮された。`moon build --target native`が`moon clean`後に壊れる問題も直し（[#486](https://github.com/dowdiness/js_engine/pull/486)）、decodeURI/decodeURIComponentのDontEnum属性とURIError検証を追加した（[#492](https://github.com/dowdiness/js_engine/pull/492)）。RegExpのlookbehind assertion `(?<=...)`/`(?<!...)`をパーサー・マッチャー・キャプチャまで含めてフル実装した（[#493](https://github.com/dowdiness/js_engine/pull/493)、test262 20/20）。

### loom / lex mode APIとdocs検証の強化

`ParserContext`にparser駆動のlex mode API（`lex_mode()`/`set_lex_mode`）を追加し、`SwitchLexMode` IRノードの前提を整えた（[#589](https://github.com/dowdiness/loom/pull/589)）。前日のincr 0.13.0リリースへの追従では、canopyより先にloomが動いた――loomのpinを0.13.0へ上げ、これをブロックしていたevent-graph-walkerのpin（moonc format-syncに追従できておらずFormat Checkを壊していた）を直し、example modulesとegglogの二重manifestまで含めたpin一掃を完了した（[#590](https://github.com/dowdiness/loom/pull/590), [#591](https://github.com/dowdiness/loom/pull/591), [#592](https://github.com/dowdiness/loom/pull/592)）。

**大掛かりなdocs検証**も入った（[#594](https://github.com/dowdiness/loom/pull/594)）。リンク切れやテスト件数の古い主張を修正し、`check-docs.sh`のbaselineをゼロへ戻した（実は`.claude/worktrees/`という古いディレクトリへスキャナが迷い込み、27件の恒久エラーとして黙って固定されていた）。README Quick Startのコマンドを実際に実行する`quickstart-check`、`.mbti`のドリフトを検出する`mbti-check`、検証不能な絶対統計値（"47個のテスト"のような表現）を禁じるdocsポリシーも新設した。loomgen側では、生成された出力が実際に`@grammar` APIに対してコンパイルできることを証明するGrammarIRのcompile-regression fixtureが入り（[#597](https://github.com/dowdiness/loom/pull/597)）、"derive don't state"ポリシーの徹底（CLAUDE.mdの古いテスト件数コメント削除、ADR Statusの意味論明文化）も進んだ（[#613](https://github.com/dowdiness/loom/pull/613)）。

主なPR / Issue: canopy [#854](https://github.com/dowdiness/canopy/pull/854), [#855](https://github.com/dowdiness/canopy/pull/855), [#856](https://github.com/dowdiness/canopy/pull/856), [#857](https://github.com/dowdiness/canopy/pull/857), [#859](https://github.com/dowdiness/canopy/pull/859), [#860](https://github.com/dowdiness/canopy/pull/860) / incr [#351](https://github.com/dowdiness/incr/pull/351) / js_engine [#485](https://github.com/dowdiness/js_engine/pull/485), [#486](https://github.com/dowdiness/js_engine/pull/486), [#489](https://github.com/dowdiness/js_engine/pull/489), [#492](https://github.com/dowdiness/js_engine/pull/492), [#493](https://github.com/dowdiness/js_engine/pull/493) / loom [#589](https://github.com/dowdiness/loom/pull/589), [#590](https://github.com/dowdiness/loom/pull/590), [#591](https://github.com/dowdiness/loom/pull/591), [#592](https://github.com/dowdiness/loom/pull/592), [#594](https://github.com/dowdiness/loom/pull/594), [#597](https://github.com/dowdiness/loom/pull/597), [#612](https://github.com/dowdiness/loom/pull/612), [#613](https://github.com/dowdiness/loom/pull/613)

## 2026/7/5

### incr / 0.14.0リリースと Expr[T] の追加

前週の0.13.0に続き、「公開API境界の整理」を掲げた0.14.0リリースがこの日にまとまった。Phase 0として`Input::new`等に`#deprecated`を付け、`Scope::watch(derived)`でwatch作成・scope登録・初回読みを1回にまとめてGCの穴（初回読み前に`Runtime::gc()`すると上流グラフが掃かれてしまう）を塞いだ（[#353](https://github.com/dowdiness/incr/pull/353)）。Phase 1では、使われていないghost handle型（`InputId[T]`等）を削除し（[#354](https://github.com/dowdiness/incr/pull/354)）、その直後に見つかったinterior-mutationの穴――`InternTable`の可視性を`pub`に絞っても構造体リテラルの構築は防げるがフィールドの読み出しでは可変な`Array`がそのまま渡ってしまい、外から`table.values.push(...)`できてしまう――をMoonBitのフィールド単位`priv`で塞いだ（[#355](https://github.com/dowdiness/incr/pull/355)）。Phase 2では`get_result`の戻り値を`Result[T, CycleError]`から`Result[T, ReadError]`へ変え、破棄済みinputへのアクセスが`Err(Disposed(id))`になるようにした（[#357](https://github.com/dowdiness/incr/pull/357)）。これらをまとめてv0.14.0としてリリースした（[#358](https://github.com/dowdiness/incr/pull/358)、7件の破壊的変更）。

リリース直後、遅延評価の数式コンビネータ層`Expr[T]`（Track E）が入った（[#359](https://github.com/dowdiness/incr/pull/359)）。`(price.expr() * quantity.expr()).derived(label="subtotal")`のように演算子オーバーロードで式を組み立てられる。6月末のMemo→Derived移行以来ずっと残っていたissue #123を閉じた。

### js_engine / 非同期イテレーションプロトコル実装

`for await (... of ...)`と非同期イテレーションプロトコル全体（`Symbol.asyncIterator`、`%AsyncIteratorPrototype%`、`CreateAsyncFromSyncIterator`）を実装した（[#494](https://github.com/dowdiness/js_engine/pull/494)）。ここから、仕様の隅を突く一連の修正が続いた。`%AsyncFromSyncIteratorPrototype%`を共有realm-ownedプロトタイプへ移し、ラップしたsync iteratorの`.next()`がPromiseを返す場合の`.then()`連鎖漏れを直し（async-gen-dstr 0/954→954/954、[#495](https://github.com/dowdiness/js_engine/pull/495)）、async generatorの`yield*`委譲が実は同期パスを使ってしまっていたバグを直し（[#496](https://github.com/dowdiness/js_engine/pull/496)）、チェーンしたpromiseがrejectしたときsync iteratorを先に閉じる`closeOnRejection`セマンティクスを追加した（50→70/76、[#499](https://github.com/dowdiness/js_engine/pull/499)）。sync側の`yield*`委譲にも独立したバグがあり、`extract_iter_result`がgetterを経由せず直接プロパティを読んでいたため、getter経由の値取得やgetterが投げたエラーの伝播が仕様通りに動いていなかった（28→50/82、[#501](https://github.com/dowdiness/js_engine/pull/501)）。

### loom / M16 EBNF演算子とベンチマークCI

`@fragment`束縛を実装し、手書きの`@grammar.Expr`断片を`#loom.rule`から参照できるようにした（未束縛の参照には専用の`MissingFragment`エラー、[#615](https://github.com/dowdiness/loom/pull/615), [#616](https://github.com/dowdiness/loom/pull/616)）。incr 0.14への追従として`@incr.Runtime::new()`を`@incr.Runtime()`へ21箇所置き換えた（[#617](https://github.com/dowdiness/loom/pull/617)）。`ParserContext`にcontext-sensitiveな文法向けのヘルパー（`current_node_kind`、`peek_index`、`finish_nodes_until`）を追加し、テストを書く過程で`finish_node`が`node_stack`をpopし忘れていた実バグも見つけて直した（[#619](https://github.com/dowdiness/loom/pull/619), [#631](https://github.com/dowdiness/loom/pull/631)）。

**週次のベンチマークリグレッションジョブ**を新設した（[#618](https://github.com/dowdiness/loom/pull/618)）。設計の過程で2つの問題を先に潰した。ひとつは、コミット済みのbaselineが4か月古く、82件の「リグレッション」の大半が単なる環境ドリフトだったこと。もうひとつは、アイドル状態のマシンでも連続実行だけで65パーセンタイルの差が出ること――これは、最大3回の実行すべてで再現した場合だけリグレッションとして数える永続性フィルタで対処した。CIランナーの実測が開発機より一律2倍近く速いことも分かり、baselineを較正し直した（[#621](https://github.com/dowdiness/loom/pull/621)）。

`examples/html/`という実戦的なHTMLパーサーをloomgenで構築した（[#625](https://github.com/dowdiness/loom/pull/625)）。手書き相当のコードのうち、loomgenが生成できたのはトークン/spec enumなど約19%にとどまり、残り約62%は再帰下降やエラー回復といった手書きロジックのままだった、という実測が興味深い。M16として、`Token~`（Emit: 無ければ黙ってスキップ）、`Token!`（EmitOr: 診断+placeholder）、`@until(Token)`（ErrorUntil: 同期点まで読み飛ばし）という3つの新しいEBNF演算子を追加し（[#632](https://github.com/dowdiness/loom/pull/632), [#634](https://github.com/dowdiness/loom/pull/634), [#639](https://github.com/dowdiness/loom/pull/639)）、CSSパーサーの実例でこの3つすべてを検証した。

### Canopy

実験的な単語ナビゲーション機能を追加した（[#861](https://github.com/dowdiness/canopy/pull/861)）。moji側の生のUAX #29境界の上に、句読点やCJKの連なりを1単位として扱い、着地点をgrapheme境界へ吸着させるVS Code風の単語左右移動を実装した（「1か月以内に消える可能性が十分ある」実験扱い）。incr 0.14準備のためloom submodule pointerもbumpした（[#870](https://github.com/dowdiness/canopy/pull/870)）。

主なPR / Issue: canopy [#861](https://github.com/dowdiness/canopy/pull/861), [#870](https://github.com/dowdiness/canopy/pull/870) / incr [#352](https://github.com/dowdiness/incr/pull/352), [#353](https://github.com/dowdiness/incr/pull/353), [#354](https://github.com/dowdiness/incr/pull/354), [#355](https://github.com/dowdiness/incr/pull/355), [#357](https://github.com/dowdiness/incr/pull/357), [#358](https://github.com/dowdiness/incr/pull/358), [#359](https://github.com/dowdiness/incr/pull/359) / js_engine [#494](https://github.com/dowdiness/js_engine/pull/494), [#495](https://github.com/dowdiness/js_engine/pull/495), [#496](https://github.com/dowdiness/js_engine/pull/496), [#499](https://github.com/dowdiness/js_engine/pull/499), [#501](https://github.com/dowdiness/js_engine/pull/501) / loom [#615](https://github.com/dowdiness/loom/pull/615), [#616](https://github.com/dowdiness/loom/pull/616), [#617](https://github.com/dowdiness/loom/pull/617), [#618](https://github.com/dowdiness/loom/pull/618), [#619](https://github.com/dowdiness/loom/pull/619), [#621](https://github.com/dowdiness/loom/pull/621), [#625](https://github.com/dowdiness/loom/pull/625), [#631](https://github.com/dowdiness/loom/pull/631), [#632](https://github.com/dowdiness/loom/pull/632), [#634](https://github.com/dowdiness/loom/pull/634), [#639](https://github.com/dowdiness/loom/pull/639)

## 2026/7/6

前日の続きが日付をまたいで収束した、比較的静かな一日。js_engineのsync `yield*`委譲修正が最後の30件を回収し、test262が**80/80（100%）**に到達した（[#502](https://github.com/dowdiness/js_engine/pull/502)）。`get_optional_method`がデータプロパティしか見ておらずアクセサgetterを呼び出していなかったのが原因で、test262のpoisoned-getterパターンを踏んでいた。

loomでは、docs中のバッククォート `@pkg.Symbol` 表記をコミット済み`.mbti`と突き合わせて検証する`check-docs-symbols.py`を追加し（[#633](https://github.com/dowdiness/loom/pull/633)）、strict-LL（1）方式（ordered choiceではない）を採る決定を再確認する過程で、`A?`のようなnullableな選択肢が非空FIRSTを持つ場合のガード漏れ――生成される`Any → Fail`フォールバックが、文法上のε許容ケースを黙って実行時パースエラーへ変えてしまう――を見つけて直した（[#638](https://github.com/dowdiness/loom/pull/638)）。`#loom.ident` / `#loom.literal` / `#loom.trivia`にデフォルトのlexパターンを与え、よくあるトークン役割では明示的な`#loom.pattern(...)`を省略できるようにした（[#641](https://github.com/dowdiness/loom/pull/641)）。

主なPR / Issue: incr [#359](https://github.com/dowdiness/incr/pull/359) / js_engine [#502](https://github.com/dowdiness/js_engine/pull/502) / loom [#633](https://github.com/dowdiness/loom/pull/633), [#634](https://github.com/dowdiness/loom/pull/634), [#638](https://github.com/dowdiness/loom/pull/638), [#639](https://github.com/dowdiness/loom/pull/639), [#641](https://github.com/dowdiness/loom/pull/641)

## 7月第2週: 再入ガードとinterpretへの転換（7/7〜7/13）

7/8の評価戦略ADRを起点に、MoonDspデバッグで見つかった再入を7/9〜7/10で契約化した。7/10に手書き`emit_grammar.mbt`を削除しinterpretベースへ。週末にJSXとGenUI実験が立ち上がる。

## 2026/7/7

### incr / whiteboxテスト移行とQuickCheck

kernel/engineの`internal/*`パッケージにテストが一切なく、無関係なパッケージのwbtestファイルがカーネル変更のたびに揺れていた問題に対応し、テストファイルの一部をkernel/engineへ移すパイロットを行った（[#360](https://github.com/dowdiness/incr/pull/360)）。この過程で見つかった、`Runtime::Runtime`とテスト側の手動ミラー初期化の重複を、単一の`RuntimeCore::RuntimeCore()`コンストラクタへ統合した（[#361](https://github.com/dowdiness/incr/pull/361)）。`moonbitlang/quickcheck`を導入し、ランダム化したpull/push操作の下で購読者の対称性・dispatch tableの整合・push到達可能性のゼロ/非ゼロゲートという3つの不変条件を検証するプロパティテストを追加した（[#362](https://github.com/dowdiness/incr/pull/362)）。さらに、push-Effectとバッチロールバックのパスも追加し、テスト自体をミューテーションテスト（意図的に本番コードを壊して赤くなることを確認してから戻す）で検証した（[#363](https://github.com/dowdiness/incr/pull/363)）。このプロセス自体が、構造的な不変条件だけでは見えないロールバックのバグと、push到達可能カウントの減算バグという2つの実バグを見つけている。

### loom

lambda exampleを手書き文法から移行するための前提として、Pratt優先順位のEBNF記法（`@prefix`/`@app`/`@prec`/`@skip`）を`@grammar.Expr::PrattApp`/`PrattBinary`へ落とすルールを追加した（[#645](https://github.com/dowdiness/loom/pull/645)）。HTML exampleのblackboxテストで、閉じタグが不一致（`<div></span>`）のときrecoveryが1文字も消費できず無限ループに陥るハングを、`skip_until_progress`への切り替えで直した（[#647](https://github.com/dowdiness/loom/pull/647)）。

主なPR / Issue: incr [#360](https://github.com/dowdiness/incr/pull/360), [#361](https://github.com/dowdiness/incr/pull/361), [#362](https://github.com/dowdiness/incr/pull/362), [#363](https://github.com/dowdiness/incr/pull/363) / loom [#645](https://github.com/dowdiness/loom/pull/645), [#647](https://github.com/dowdiness/loom/pull/647), [#648](https://github.com/dowdiness/loom/pull/648)

## 2026/7/8

変更件数は少ないが、翌日以降の作業につながる2つの更新が入った。incrでは、pull/push/Datalog-fixpointという3つの評価戦略の間の相互作用ルールを明文化するADRを書いた（[#372](https://github.com/dowdiness/incr/pull/372)）。ADR本文を読むと、単なる整理を超えた踏み込んだ決定が並んでいる。まずセルを「現在値だけに対して純粋（pure-of-current-values）」と「履歴依存（history-dependent）」の2種に分類し、前者だけがbackdatingや検証スキップといった透過的キャッシュの対象になれると定めた。この帰結として、Solid風に`createMemo(prev => ...)`のような形で前回値をすべてのcomputeへデフォルトで渡す設計は、キャッシュ層が観測可能になり透過性が静かに崩れるという理由で明示的に却下されている。履歴依存の状態が必要な場面には`mut`キャプチャとAccumulatorという2つの逃げ道を認めつつ、`fold`/`pairwise`という第一級プリミティブは「予約はするが今は作らない」とした。

型ではなく実行時に強制する理由も踏み込んで書かれている。handle分割（Reader/Writer）もtrait objectも、MoonBitのclosureが任意の環境をキャプチャできる以上、捕まえたwriterがcompute内でそのままコンパイルされてしまい、バグを型では防げない。context/tokenを渡すSalsa的な設計は本当に型で防げるが、全computeのシグネチャを作り直す必要があるため今回は却下、ただし「もしゼロから設計し直すなら」の第一候補として記録された。3つのエンジンを1つの`Runtime`にまとめる理由も実装の都合ではなく、incr_teaのUI Watchがparserのmemoに依存し、moondspのeager foldがmemoを読むという、戦略をまたぐ依存辺が実在するからだと明記されている。ライブラリを分ければこのseamはユーザー空間の橋渡しコードへ移動するだけで、伝播基盤（revision・dirty marking・subscriber・batch・GC）を3重に複製することになる。最後にこのADRは、「fold engineが本当にpush側だけのleafとして実装できるか」「同種のバグが再発しないか」という2つの反証可能な予測を立てて締めている。このADRが、翌々日に入る`force_set`の再入防止ガードの種になる。

もうひとつはMoonDsp。この期間で唯一のMoonDspへの変更として、`MiniAuthoringPipeline`に「最後に成功した文書」を保持する`AcceptedDerived`チャンネルを追加した（[#227](https://github.com/dowdiness/MoonDsp/pull/227)、dowdiness/moondsp#184向けの「Phase 0リハーサル」）。このPRのeager foldが、`parsed.get()`→memo再計算→`source_edit_base.set(current)`という経路でincr内部の再入伝播を引き起こすことが、翌日のincrデバッグで判明する。根本原因の特定には4回の実装試行を要した、とADR自身が振り返っている。

主なPR / Issue: incr [#372](https://github.com/dowdiness/incr/pull/372) / loom [#650](https://github.com/dowdiness/loom/pull/650) / MoonDsp [#227](https://github.com/dowdiness/MoonDsp/pull/227)

## 2026/7/9

### incr / force_setの再入安全化とL2 semantic oracle

前日のMoonDspデバッグで見つかった再入伝播の問題に対処した。`tracking.stack`の長さを見るガードを追加し、`Input::force_set`がreactive compute中に呼ばれたら中断するようにした（[#373](https://github.com/dowdiness/incr/pull/373)）。Codexと手動監査によるレビューでADRに5件の訂正が入り、実は強制ポイントが2か所（既存のphase遷移ガードと、今回追加したpull-compute側のガード）あることや、Datalog/fixpointの合法表の追加、`on_change`の書き込みハザードの明記などが記録された（[#374](https://github.com/dowdiness/incr/pull/374)）。さらに、fixpointのルール本体やGCはtracking frameを持たずこのガードをすり抜けていたため、phaseそのものを見るガード（`phase != Idle`で中断）を追加して穴を塞いだ（[#378](https://github.com/dowdiness/incr/pull/378)）。ガードの適用範囲を「購読者のいない書き込みだけ」へ狭めることは、純粋性・検証負担の両面から明示的に却下し（[#379](https://github.com/dowdiness/incr/pull/379)）、代わりに`mut`キャプチャによる履歴依存状態というエスケープハッチをcookbookに書いた（[#380](https://github.com/dowdiness/incr/pull/380)）。

もうひとつの流れとして、「L2 semantic oracle」という新しいプロパティテスト層が生まれた。実際のRuntimeと、ランダムな操作列に対して素朴に全再計算する参照モデルを突き合わせるテストを追加し（chain/diamond/条件付き動的依存など7パターン、[#381](https://github.com/dowdiness/incr/pull/381)）、`ReachableDerived[Int]`にも拡張し（[#382](https://github.com/dowdiness/incr/pull/382)）、ネストしたバッチのcommit/rollbackを検証する状態機械プロパティテストも追加した（[#383](https://github.com/dowdiness/incr/pull/383)）。いずれもミューテーションテストで検証されている。

### js_engine

`IteratorClose`が、外側の完了がthrowのときに`return()`の戻り値へのIsObjectチェックをスキップし、元の`ThrowCompletion`を優先するよう修正した（[#503](https://github.com/dowdiness/js_engine/pull/503)、§7.4.11）。realm prototypeのwbtestを集約するリファクタ（[#508](https://github.com/dowdiness/js_engine/pull/508)）と、2140行あった`builtins_map_set.mbt`を役割ごとに分割するリファクタ（[#509](https://github.com/dowdiness/js_engine/pull/509)）も入った。これは翌日から始まる`cache-for-X`移行の下準備になる。

### loom

Markdownのインライン構文（強調のデリミタスタックアルゴリズム、文書全体にまたがるlink reference definition、対応の取れた括弧を要するlinkの宛先）は、`derive(Eq, Debug)`な`Pred`が持てない可変なホスト状態を必要とするため、loomgenでは生成せず`@native`のまま残す、という恒久的な決定をADRに記した（[#643](https://github.com/dowdiness/loom/pull/643)）。これは「まだ手が回っていない」保留ではなく、明示的な「生成しない」決定だとADRは強調している。理由は6月に`Custom(fn)`をIRから締め出した判断（#541）と同根で、強調のデリミタスタック処理も、文書全体を先に読んでから解決するlink reference definitionも、`GrammarIr`が「データだけ」であるという不変条件の中には収まらない。覆すとしたら、2つ目の言語が独立して同種のデリミタマッチングを必要とするようになるか、可変状態を持たずに`derive(Eq, Debug)`を保てる新しいプリミティブが設計されたときだけ、と再検討条件も明記されている。loomgenのターゲットはCommonMark**ブロック**部分集合のみで、インラインは対象外とするのが最終的な線引き。`examples/moonbit`という新しいMoonBit言語のスケルトンパーサーが、block/if式（[#654](https://github.com/dowdiness/loom/pull/654)）、match式（[#655](https://github.com/dowdiness/loom/pull/655)）、MoonBit公式の優先順位表に沿ったPratt演算子式（[#656](https://github.com/dowdiness/loom/pull/656)）へと育った。`@native(name)`（手書きのホストparse関数を不透明な参照として文法に組み込む、[#659](https://github.com/dowdiness/loom/pull/659)）、`GoalTokenSource`（パーサーが指定した位置を指定した字句ゴールで再トークン化するオーバーレイ、[#660](https://github.com/dowdiness/loom/pull/660)）も入った。GoalTokenSourceは明示的にjs_engineの差分再パースパイプライン向けに書かれたもので、loomが将来JSも文法ホストの対象にする最初の具体的な証拠になる。`@native`が最後のChoice節にあるときはelse節として振る舞う仕様も追加した（[#661](https://github.com/dowdiness/loom/pull/661)）。

主なPR / Issue: canopy [#871](https://github.com/dowdiness/canopy/pull/871) / incr [#373](https://github.com/dowdiness/incr/pull/373), [#374](https://github.com/dowdiness/incr/pull/374), [#378](https://github.com/dowdiness/incr/pull/378), [#379](https://github.com/dowdiness/incr/pull/379), [#380](https://github.com/dowdiness/incr/pull/380), [#381](https://github.com/dowdiness/incr/pull/381), [#382](https://github.com/dowdiness/incr/pull/382), [#383](https://github.com/dowdiness/incr/pull/383), [#384](https://github.com/dowdiness/incr/pull/384) / js_engine [#503](https://github.com/dowdiness/js_engine/pull/503), [#508](https://github.com/dowdiness/js_engine/pull/508), [#509](https://github.com/dowdiness/js_engine/pull/509) / loom [#643](https://github.com/dowdiness/loom/pull/643), [#649](https://github.com/dowdiness/loom/pull/649), [#651](https://github.com/dowdiness/loom/pull/651), [#653](https://github.com/dowdiness/loom/pull/653), [#654](https://github.com/dowdiness/loom/pull/654), [#655](https://github.com/dowdiness/loom/pull/655), [#656](https://github.com/dowdiness/loom/pull/656), [#659](https://github.com/dowdiness/loom/pull/659), [#660](https://github.com/dowdiness/loom/pull/660), [#661](https://github.com/dowdiness/loom/pull/661)

## 2026/7/10

### incr / InputViewとScope::on_dispose

MoonBitのview構文（`input[:]`）を使った、読み取り専用の不変な`InputView`を追加した（[#385](https://github.com/dowdiness/incr/pull/385)、#369を閉じる）。0.14.0リリース基準へ素直に再適用する形で入れ直し、trackedな`get()`とuntrackedな`peek()`を区別した（[#386](https://github.com/dowdiness/incr/pull/386)）。マウント時に登録したリソースを、子→親の順で確実に1回だけ後始末する`Scope::on_dispose`も追加した（[#388](https://github.com/dowdiness/incr/pull/388)）。

### js_engine / cache-for-X migrationが本格化

builtinのconstructor/prototypeインストールをatomicかつrealm間で一貫させる「cache-for-X」契約の展開が始まった（[#512](https://github.com/dowdiness/js_engine/issues/512)）。Map/Setのiteratorスロットヘルパーをspec-neutralな共通コードへ抽出し（[#510](https://github.com/dowdiness/js_engine/pull/510)）、`install_realm_pinned_builtin_constructor`という、realm_proto_cacheの固定・constructorの`.prototype`凍結・`env.def_builtin`を1回で行うヘルパーをMap/Setに試験導入した（[#511](https://github.com/dowdiness/js_engine/pull/511)）。これは、手で分割してインストールしていたためrealmをまたいで静かにズレることがあった暗黙の不変条件を、明文化した契約に置き換えるもの。WeakMap/WeakSet（[#513](https://github.com/dowdiness/js_engine/pull/513)）、Array（[#514](https://github.com/dowdiness/js_engine/pull/514)、bagged-prototype版の契約も追加）と、対象ファミリーを順に移行していった。

### loom / EBNF演算子の追加とemit_grammar.mbtの引退

EBNFの表現力が着々と広がった一日でもある: HTMLのlexerがscript/styleの内容を単なる`Text`としてタグ付けしていた（parser側の`RawTextLeaf`分岐が死んでいた）問題をmode-awareなstep lexerで直し（[#662](https://github.com/dowdiness/loom/pull/662)）、宣言的なエラー回復のための`@error_node(Kind, Token)`構文（[#663](https://github.com/dowdiness/loom/pull/663)）、オプショナルなスキップトークンを消費してから必須トークンを要求する`Token~>`（ExpectSkip、[#666](https://github.com/dowdiness/loom/pull/666)）、差分再利用に向く区切り付きリストの`{Sep}`（separated-list、[#667](https://github.com/dowdiness/loom/pull/667)）を追加した。

しかし**この日いちばん大きな出来事は、手書きのコード生成パスそのものを丸ごと削除したこと**（[#672](https://github.com/dowdiness/loom/pull/672)）。ベンチマークの結果、tree-walkingする`@grammar.interpret`が、フルパースでは生成コードよりわずかに遅い（約1.25倍）ものの、差分再パースではむしろ速い（浅いケースで0.95倍、深いケースで0.91倍）ことが分かり、生成コードパスと同等の性能に達したと判断された。`emit_grammar.mbt`の766行を削除し、`mbt_ast.mbt`を581行から90行まで削り、8個の未使用型・10個の未使用variant・死んだPretty実装を除去した。loomgenが自身の存在理由の一部（「速いから生成コードを使う」）を検証で否定し、interpretベースへ一本化した節目になる。

もう一つ、incrの新しいforce_setガード（[#373](https://github.com/dowdiness/incr/pull/373)）が実際にバグを捕まえた例があった。Lambdaのtypecheckパイプラインが`Derived::recompute_inner`実行中に`Input::set`を呼んでガードに引っかかり、propagation完了後のon_change境界へ同期処理を移す形で直した（[#675](https://github.com/dowdiness/loom/pull/675)）。

主なPR / Issue: canopy [#873](https://github.com/dowdiness/canopy/pull/873) / incr [#385](https://github.com/dowdiness/incr/pull/385), [#386](https://github.com/dowdiness/incr/pull/386), [#388](https://github.com/dowdiness/incr/pull/388) / js_engine [#510](https://github.com/dowdiness/js_engine/pull/510), [#511](https://github.com/dowdiness/js_engine/pull/511), [#513](https://github.com/dowdiness/js_engine/pull/513), [#514](https://github.com/dowdiness/js_engine/pull/514) / loom [#662](https://github.com/dowdiness/loom/pull/662), [#663](https://github.com/dowdiness/loom/pull/663), [#666](https://github.com/dowdiness/loom/pull/666), [#667](https://github.com/dowdiness/loom/pull/667), [#672](https://github.com/dowdiness/loom/pull/672), [#675](https://github.com/dowdiness/loom/pull/675), [#676](https://github.com/dowdiness/loom/pull/676)

## 2026/7/11

### Canopy / JSXという4番目の言語ターゲット

CanopyにJSXという新しい言語ターゲットが生まれた。Lambda/JSON/Markdownに続く4番目になる。JSX計画のPhase 1を締め、loomの新しいstreaming-prefix JSX文法を取り込んだ（[#876](https://github.com/dowdiness/canopy/pull/876)）。`lang/jsx/proj`として、CSTから`ProjNode[JsxNode]`への読み取り専用projectionを実装した（[#877](https://github.com/dowdiness/canopy/pull/877)）。既存の「言語追加手順書」通りに、`@core.build_projection_memos`や`ProjNode::leaf`/`branch`、`SourceMap::set_token_span`をそのまま再利用し、手組みの再構成ロジックを足さずに済んだ。Phase 2で要求されている「存在証明」テスト――単調に伸びていくJSXの断片（トークンの途中で切れたものも含む）を`@loom.Parser`へ流し込み、タグ名が確定した時点で`NodeId`が安定することを確認する――も入れた。Codexによる実装後レビューが実バグを発見しており、未クローズの要素で自身の開始タグの`>`を閉じタグの`>`と誤認識して`close_bracket`を記録していた問題を、3件のリグレッションテストとともに直した。まだ編集用のcompanionパッケージはなく、双方向編集はPhase 3へ持ち越しになっている。

### js_engine / cache-for-Xの完了、host object API、private field

cache-for-X移行がPromise（[#515](https://github.com/dowdiness/js_engine/pull/515)）とboxed primitives（String/Number/Boolean/Symbol、[#516](https://github.com/dowdiness/js_engine/pull/516)）で完了し、6つあった`install_realm_pinned_builtin_constructor_*`亜種を1つの関数と`pub(all) enum BuiltinCtorPrototypeInstall`へ統合した（[#520](https://github.com/dowdiness/js_engine/pull/520)）。realm-proto walk/StdlibHooks/get_*_proto+prototype chainという3つの独立したbuiltinプロパティ検索機構の統合は、いまはあえて見送り、境界を図で明文化するにとどめた（[#521](https://github.com/dowdiness/js_engine/pull/521)）。

続けて、埋め込み利用者向けの**host-object API**を実装した（JSからは見えない不透明な`HostSlotKey`のget/set/delete/has、メソッド・アクセサ・凍結データ・host slotを1回で組み立てる`make_host_object`ファクトリ、embeddingの手引きまで、[#522](https://github.com/dowdiness/js_engine/pull/522), [#523](https://github.com/dowdiness/js_engine/pull/523), [#524](https://github.com/dowdiness/js_engine/pull/524)）。JSON.stringifyのProxy trapがstringify走査中に発火しない問題も直し（test262 55.2%→75.9%、[#526](https://github.com/dowdiness/js_engine/pull/526)）、**class private field・private method・static blockをES2022仕様通り実装**した（[#527](https://github.com/dowdiness/js_engine/pull/527)、2394テスト）。`obj.#x = val`という代入だけは翌日への持ち越しになった。

### loom / GrammarIRプロパティテストとJSXの文法側

GrammarIRのプロパティテストが成熟し、複数規則+`Ref`対応と参照インタプリタによる相互検証（[#679](https://github.com/dowdiness/loom/pull/679)）、反例最小化のための`Shrink`実装（[#682](https://github.com/dowdiness/loom/pull/682)）、23個の`Expr` variant全体への再帰的`Shrink`と診断メッセージの一致検証（[#684](https://github.com/dowdiness/loom/pull/684)、この過程で参照実装と実インタプリタのエラー文言の不一致を発見）が入った。

**CanopyのJSX計画に対応するloom側の文法**として、streaming-prefix JSX文法exampleを実装した（[#680](https://github.com/dowdiness/loom/pull/680)）。モード切り替え式のlexer（Children/TagBody/JsExprRaw（depth））、文字列リテラルを意識した波括弧深さ追跡、HTMLの回復方針（診断はノード生成を妨げない、`skip_until_progress`を使う）を踏まえた再帰下降パーサーで、73/73テストが通った。実装の過程でHTMLの`parse_html_root`が`RootNode`を二重に包んでいたバグも見つけて直した（[#686](https://github.com/dowdiness/loom/pull/686)）。

そのほかloomgen周りの小さな進展: `#loom.recovery("sync")`アノテーションでHTML/JSX/Lambdaの手書き`is_sync_point`を生成コードへ移行（[#690](https://github.com/dowdiness/loom/pull/690), [#692](https://github.com/dowdiness/loom/pull/692)）、`@fragment`参照ごとの`pub let frag_<name>`宣言自動生成（[#693](https://github.com/dowdiness/loom/pull/693)）、JSONの`syntax_kind.mbt`をloomgen生成へ移す作業（[#696](https://github.com/dowdiness/loom/pull/696)）、ブロックレベルのトークンを行単位で生成する`#loom.line_pattern`（長らく開いていた#561を閉じ、後のMarkdown向けレクサ生成の布石になる、[#698](https://github.com/dowdiness/loom/pull/698)）。

### incr

9個のワークスペースメンバーがincr 0.14.1のままだったのを0.14.2へ揃え、architecture-boundaryチェックの失敗を解消した（[#389](https://github.com/dowdiness/incr/pull/389)）。

主なPR / Issue: canopy [#875](https://github.com/dowdiness/canopy/pull/875), [#876](https://github.com/dowdiness/canopy/pull/876), [#877](https://github.com/dowdiness/canopy/pull/877), [#878](https://github.com/dowdiness/canopy/pull/878) / incr [#389](https://github.com/dowdiness/incr/pull/389) / js_engine [#515](https://github.com/dowdiness/js_engine/pull/515), [#516](https://github.com/dowdiness/js_engine/pull/516), [#520](https://github.com/dowdiness/js_engine/pull/520), [#521](https://github.com/dowdiness/js_engine/pull/521), [#522](https://github.com/dowdiness/js_engine/pull/522), [#523](https://github.com/dowdiness/js_engine/pull/523), [#524](https://github.com/dowdiness/js_engine/pull/524), [#526](https://github.com/dowdiness/js_engine/pull/526), [#527](https://github.com/dowdiness/js_engine/pull/527) / loom [#679](https://github.com/dowdiness/loom/pull/679), [#680](https://github.com/dowdiness/loom/pull/680), [#682](https://github.com/dowdiness/loom/pull/682), [#684](https://github.com/dowdiness/loom/pull/684), [#686](https://github.com/dowdiness/loom/pull/686), [#690](https://github.com/dowdiness/loom/pull/690), [#692](https://github.com/dowdiness/loom/pull/692), [#693](https://github.com/dowdiness/loom/pull/693), [#696](https://github.com/dowdiness/loom/pull/696), [#698](https://github.com/dowdiness/loom/pull/698)

## 2026/7/12

前日仕上げたclass private fieldに、残っていた`obj.#x = val`の代入を実装した（`PrivateMemberAssign`ノード追加、[#532](https://github.com/dowdiness/js_engine/pull/532)）。

Canopyに、JSXの上に立つ**「generative UI（GenUI）」という新しい実験**が姿を見せ始めた。GenUIデモの安定化として3つのバグを直した（[#885](https://github.com/dowdiness/canopy/pull/885)）。`reset_jsx_state`がmoon.pkgのexportには残っているのにMoonBit側の定義が消えておりリンカが黙って落としていた問題、ストリーミング中の途中経過（`<div><p>text</p>\n<`のような断片）が空タグの`Element`回復ノードを生み、それが`document.createElement('')`のクラッシュにつながっていた問題（空タグのコンテナは透過的な回復殻として扱い、子要素を親へ昇格させる形で解決）、プレビュー領域に強制していたCSSの枠線・パディングを外し、入力側の`class`属性だけで見た目が決まるようにした問題。

このデモを支えるため、JSXのFFIセッションに所有権の契約を導入した（[#887](https://github.com/dowdiness/canopy/pull/887)）。それまでモジュールレベルのグローバル1つに依存していたのを、セッションごとにパーサー・projection memo・DOM registry・revision counter・診断・破棄処理を持つ形へ変えた。

loomでは、`#loom.line_mode`のterm variantが未マッチ入力を同モード内の別レクサへ委譲できる`#loom.fallback_lex`を追加した（[#703](https://github.com/dowdiness/loom/pull/703)）。7/11の`#loom.line_pattern`に続く、行指向レクサ生成の一歩。

主なPR / Issue: canopy [#885](https://github.com/dowdiness/canopy/pull/885), [#887](https://github.com/dowdiness/canopy/pull/887) / js_engine [#532](https://github.com/dowdiness/js_engine/pull/532) / loom [#703](https://github.com/dowdiness/loom/pull/703)

## 2026/7/13

### Canopy / GenUI入力の垂直スライス

GenUIの「入力」側を一気に押し進めた（[#890](https://github.com/dowdiness/canopy/pull/890)）。リクエストのライフサイクル、固定チャンクでのリプレイ、制約付き候補検証、JSXのdry-run/commit回復を整え、JSXのDOMパッチ契約を凍結した。このPRの目玉は、GenUIで描画された出力を実際に使う最初の消費者として、Phase 4のホスト所有JSON/CSV Data Explorer（フィルタリング・選択保持・サマリー・詳細表示・アクセシブルなブラウザ契約）を追加したこと。「functional core, imperative shell」――データの導出は純粋に保ち、DOM/セッションの副作用は境界に閉じ込める――という分割が明示されている。

reconcileの核心的なバグも直した（[#892](https://github.com/dowdiness/canopy/pull/892)）。同じkeyを持つ重複sibling（たとえば同じkeyのJSX子要素2つ）が、patch diff中に接頭辞安定な識別子を得られていなかった。core・JSX両方にリグレッションテストを追加し、生成されたJSXパッチ適用のQuickCheckカバレッジも足した。

### js_engine / 監査からの改善計画実行

前日の「Improve skill」監査（commit 556952f）が出した6件の優先順位付き改善計画を、その日のうちに3件実行した。private memberのyield/await早期エラー検出漏れ（`expr_has_yield`/`expr_has_await`が`PrivateMember`/`PrivateMemberAssign`/`PrivateIdent`に再帰しておらず、`(yield this).#x`のような不正なパラメータデフォルトがパースを通ってしまっていた）を直し（[#535](https://github.com/dowdiness/js_engine/pull/535)）、JSON.stringifyがgetter・Proxy trap・Map/Set/Promiseのexpandoを無視して`{}`に固定していた問題を、`@runtime.is_array`という正準操作経由へ配線し直して解決し（[#537](https://github.com/dowdiness/js_engine/pull/537)）、CIのunit-testジョブを部分的な`architecture-state-audit`から完全な`architecture-audit`へ切り替えた（[#538](https://github.com/dowdiness/js_engine/pull/538)）。roadmapドキュメントも、async iteration・RegExp lookbehind・class privateがすでに出荷済みなのに未実装のまま書かれていた食い違いを直した（[#536](https://github.com/dowdiness/js_engine/pull/536)）。

### incr

`Scope::dispose`がteardown effectを実行する前にscopeを閉じるようにし、再入時も子→cleanup hook→所有セルの順序と「1回だけ実行」を保証した（[#391](https://github.com/dowdiness/incr/pull/391)）。incr_teaのcontrolled DOMプロパティ（`value`/`checked`/`disabled`/`selected`）が、同一HTMLの高速パスをスキップする際に書き戻されず、無関係な再レンダー後にcontrolled inputの値がずれてしまう問題も直した（[#393](https://github.com/dowdiness/incr/pull/393)）。

### loom / line-mode lexer生成の総仕上げ

`#loom.pattern`と`#loom.line_pattern`の共存を拒否するガード（[#704](https://github.com/dowdiness/loom/pull/704)）から始まり、生成した`line_lexer.g.mbt`を実際のMarkdown入力（LF/CRLF両方）へ流して動かす、最初のコンパイル・実行つき受け入れfixtureを追加した（[#705](https://github.com/dowdiness/loom/pull/705)）。line-modeヘルパーを`lexer_skeleton.g.mbt`のオーバーライドポイント経由で配線し（[#706](https://github.com/dowdiness/loom/pull/706)）、その後は生成コードが手書きコードを静かに壊さないことを証明するテストの塔を積み上げた。同一パッケージへのヘルパー書き込みを強制的に失敗させて確認するリグレッション（[#707](https://github.com/dowdiness/loom/pull/707)）、96ケースのQuickCheckプロパティ（[#708](https://github.com/dowdiness/loom/pull/708)）、独立したバイト精度オラクルと突き合わせるカスタムジェネレータ（[#709](https://github.com/dowdiness/loom/pull/709)）と続いた。ベンチマーク検出のポリシーも明文化した（[#711](https://github.com/dowdiness/loom/pull/711)）。

主なPR / Issue: canopy [#890](https://github.com/dowdiness/canopy/pull/890), [#891](https://github.com/dowdiness/canopy/pull/891), [#892](https://github.com/dowdiness/canopy/pull/892) / incr [#390](https://github.com/dowdiness/incr/pull/390), [#391](https://github.com/dowdiness/incr/pull/391), [#392](https://github.com/dowdiness/incr/pull/392), [#393](https://github.com/dowdiness/incr/pull/393), [#397](https://github.com/dowdiness/incr/pull/397) / js_engine [#534](https://github.com/dowdiness/js_engine/pull/534), [#535](https://github.com/dowdiness/js_engine/pull/535), [#536](https://github.com/dowdiness/js_engine/pull/536), [#537](https://github.com/dowdiness/js_engine/pull/537), [#538](https://github.com/dowdiness/js_engine/pull/538) / loom [#704](https://github.com/dowdiness/loom/pull/704), [#705](https://github.com/dowdiness/loom/pull/705), [#706](https://github.com/dowdiness/loom/pull/706), [#707](https://github.com/dowdiness/loom/pull/707), [#708](https://github.com/dowdiness/loom/pull/708), [#709](https://github.com/dowdiness/loom/pull/709), [#711](https://github.com/dowdiness/loom/pull/711)

## 7月第3週: GenUI実験の迷走とLegible Instrumentへの転換（7/14〜7/20）

GenUIに7/29の判断日を付け、loomgenはMarkdown向けline-modeレクサ生成の総仕上げへ。週の後半、Codex/Ollamaを使った大掛かりなprovider比較実装はmainへ入らないまま終わり、代わって"Legible Instrument"という意思決定ワークスペース型の設計へ転換した。web appはfeatureごとのモジュール境界抽出が始まった。

## 2026/7/14

### incr / 保持コストのベンチマークと「直さない」という決定

Duplixに着想を得た保持コスト（retention cost）のベンチマーク一式を追加した（[#398](https://github.com/dowdiness/incr/pull/398)）。放置された購読者・破棄し忘れたeagerセル・動的サブグラフのchurnといった「後片付けを忘れたときのコスト」を28本のベンチマークで定量化した。数字が具体的で、たとえば同一rootでのpush経路は1,000件で24.3µs、10,000件で757.5µsまで伸びる一方、破棄・GC制御を入れると75–84倍安くなる。重要なのは、いくつかの問題を「まだ直っていない」と明記して締めたこと――一部の制御は完全にはフラットにならず、10,000件のケースは1,000件に比べて超線形になる。これに続くdocsで、以前の実行計画にあった「3つのresolver亜種が併存している」という前提が古く、実際は7/2の[#839](https://github.com/dowdiness/canopy/pull/839)（Lambda名前解決の`@scope`統合）ですでに解消済みだったことを見つけて訂正した（[#400](https://github.com/dowdiness/incr/pull/400)）。

### loom / speculativeからlookaheadへ

グラマー-as-dataのlambda回復が、生成された共有sync述語を使うようになり、パラメータの型注釈が壊れているときの診断とCST構造の対応が回復した（[#714](https://github.com/dowdiness/loom/pull/714)）。Markdownの純粋な先読み（checkpoint/restoreのペア）を`ParserContext::speculative`へ移行し（[#715](https://github.com/dowdiness/loom/pull/715)）、その直後に`speculative`という名前自体を`lookahead`へ変更した（[#717](https://github.com/dowdiness/loom/pull/717)）。地味なリネームだが、翌々日の「structural prefix dispatch」に向けたAPI明確化の布石になる。ベンチマークの棚卸しも行い、105行あったbaseline-onlyの行をevent-graph-walkerの廃止分と確認し、実行結果は5件のしきい値超過をすべて計測ノイズと判定した（[#713](https://github.com/dowdiness/loom/pull/713)）。

主なPR / Issue: incr [#398](https://github.com/dowdiness/incr/pull/398), [#400](https://github.com/dowdiness/incr/pull/400) / loom [#710](https://github.com/dowdiness/loom/pull/710), [#713](https://github.com/dowdiness/loom/pull/713), [#714](https://github.com/dowdiness/loom/pull/714), [#715](https://github.com/dowdiness/loom/pull/715), [#717](https://github.com/dowdiness/loom/pull/717)

## 2026/7/15

### Canopy / GenUIの現在地と kill date 付きの次の実験

GenUI のブラウザ障害回復スライスを完成させた（[#893](https://github.com/dowdiness/canopy/pull/893)、Playwright 14/14）。検証・dry-run・DOM 適用・回復・「厳密に 1 回だけコミットされる」ことの証跡までを扱う。実プロバイダ（Gemini 等）との接続はまだ含まれていない。決定論的な非同期ドライバも追加し（[#894](https://github.com/dowdiness/canopy/pull/894)）、chunk・キャンセル・遅延到着を処理する。

実プロバイダを繋ぐための削除可能な実験の設計書も出た（[#897](https://github.com/dowdiness/canopy/pull/897)）。出発点は「構文的に妥当な commit は value の証拠にならない」という問題意識で、**2026/7/29** を継続/削除の判断日（kill date）とし、証拠不十分なら DELETE をデフォルトとする。却下した代替案・実験スコープ・前提条件の詳細は [[Canopy-GenUI実験-2026年7月|GenUI 実験の設計メモ]] にまとめた。

これとは別に、Idealのapp層をモジュール化するリファクタも入った（[#895](https://github.com/dowdiness/canopy/pull/895)）。明示的なroot reactive stateとaction-overlayパッケージの抽出に加え、共有JS FFI基盤を新しいリポジトリ`dowdiness/js_ffi`として切り出し、`lib/dom-boundary`をCanopy本体から独立させた。

### js_engine / v0.6.0リリース

コマンドのトークナイズ処理を、Test262 runnerとベンチマークツールで別々に実装されていた`shlex_split`もどきから、1つの共有`tokenize_command`ヘルパーへ統一した（[#539](https://github.com/dowdiness/js_engine/pull/539)）。より大きな一手として、同じJSレルムを繰り返しの`eval`/`call_json`呼び出しにまたがって保持し続ける、永続的なroot-package `Engine`を追加した（[#540](https://github.com/dowdiness/js_engine/pull/540)、#242を閉じる）。これまでの一回限りの`run*`facadeでは呼び出しのたびにインタプリタを作り直す必要があったが、embedderはこれで状態を保ったまま呼び出しを重ねられるようになった。JSON境界は厳密に保ち（可変なグローバル`JSON`もgetterも許さない）、明示的なmicrotask/timerのcheckpoint制御も持つ。これらをまとめて**v0.6.0**としてリリースした（[#541](https://github.com/dowdiness/js_engine/pull/541)）。6/27の[#476](https://github.com/dowdiness/js_engine/pull/476)からここまでの全変更を含む。

### incr

incr_teaの親子合成・意味的アイデンティティ・世代交代・リクエストの順序付けとリプレイを扱う、package-privateな機能的コアを追加した（[#401](https://github.com/dowdiness/incr/pull/401)）。256子のedit-to-flush p95が16,700µsのゲートに対し500/600/300µsで収まることを確認しつつ、公開`Machine`型や子ごとのreactive graphの追加は、2つ目の実利用が現れるまで意図的に見送るという判断を明記した。保持コストの残存分についても追加調査を行い、「まだ直さない」という結論を記録している（[#402](https://github.com/dowdiness/incr/pull/402)）。

### loom

「structural prefix dispatch」の準備として、Markdownの候補コンテキストに応じたブロック再パーサー選択を整理した（[#718](https://github.com/dowdiness/loom/pull/718)）。#484のネイティブcode-span実装そのものではなく、その前段の準備。前日の`speculative`→`lookahead`リネームの成果を実際に使う形で、prefixを持つ構造ブロック（fence/quote）を所有root/blockquote/list-itemへ正しくルーティングし、CommonMarkの段落中断規則（インデントされたコードブロックには空行が必要、prefix付きのfence/quoteはcontainerの所有権を保つ）も直した。

主なPR / Issue: canopy [#893](https://github.com/dowdiness/canopy/pull/893), [#894](https://github.com/dowdiness/canopy/pull/894), [#895](https://github.com/dowdiness/canopy/pull/895), [#897](https://github.com/dowdiness/canopy/pull/897) / incr [#401](https://github.com/dowdiness/incr/pull/401), [#402](https://github.com/dowdiness/incr/pull/402), [#403](https://github.com/dowdiness/incr/pull/403), [#404](https://github.com/dowdiness/incr/pull/404) / js_engine [#539](https://github.com/dowdiness/js_engine/pull/539), [#540](https://github.com/dowdiness/js_engine/pull/540), [#541](https://github.com/dowdiness/js_engine/pull/541) / loom [#718](https://github.com/dowdiness/loom/pull/718)

## 2026/7/16

### Canopy / GenUIの実験を「測れる価値」でゲートする

前日の設計書に続き、実プロバイダ実験を進める前提を1つずつ固めた。プロバイダ実験は測定可能な価値指標が出るまでゲートする方針を明文化し（[#898](https://github.com/dowdiness/canopy/pull/898)）、設計レビューで浮かんだ懸念を閉じた（[#899](https://github.com/dowdiness/canopy/pull/899)）。そのうえで、使い捨て前提のローカルLLM垂直スライスを追加した（[#900](https://github.com/dowdiness/canopy/pull/900)）——実装は`examples`配下に隔離し、モデル選定は事前登録、プロダクションビルドではスパイクそのものを隠す、という3段構え。「まず動くものを作る」より先に「捨てられる形で作る」ことを優先した設計になっている。

### incr / incr_teaのcontrolled formとtyped spreadsheet

incr_teaでcontrolled reconciliationのコストを計測し（[#405](https://github.com/dowdiness/incr/pull/405)）、controlled form properties（value/checked相当のcontrolled入力プロパティ）を追加した（[#406](https://github.com/dowdiness/incr/pull/406)）。value/eventペイロードをneutralな形へ整理し直し（[#407](https://github.com/dowdiness/incr/pull/407)）、typed spreadsheetのincr_tea proofを拡張した（[#408](https://github.com/dowdiness/incr/pull/408)）。

### js_engine / v0.6.0リリース

前日までの変更をv0.6.0としてリリースした（[#541](https://github.com/dowdiness/js_engine/pull/541)）。test262はPassed/Executedでstrict 90.6%・non-strict 90.1%（Passed/Discoveredだと約70%）、unit testは2,355/2,355。以降のPRはこのリリースを起点にした整理で、embedded runtimeのビジョンとロードマップを定義し（[#544](https://github.com/dowdiness/js_engine/pull/544)）、Stage 1埋め込み契約を文書化し（[#545](https://github.com/dowdiness/js_engine/pull/545)、[#546](https://github.com/dowdiness/js_engine/pull/546)でCI検証）、安定した埋め込みベンチマークのbaselineを整えた（[#547](https://github.com/dowdiness/js_engine/pull/547)）。

### loom / structural prefix dispatchとMarkdown code span

7/15の`lookahead`リネームを受けて、Markdownのブロック再パーサー選択を「候補コンテキストに応じたstructural prefix dispatch」へ整理する準備を進めた（[#718](https://github.com/dowdiness/loom/pull/718)）。正規化されたcode span解析（[#719](https://github.com/dowdiness/loom/pull/719)）、Markdown実行ロードマップのdocs化（[#722](https://github.com/dowdiness/loom/pull/722)）、surface rewriteをまたいでprojection identityを保持する変更（[#724](https://github.com/dowdiness/loom/pull/724)）も入った。

主なPR / Issue: canopy [#896](https://github.com/dowdiness/canopy/pull/896), [#897](https://github.com/dowdiness/canopy/pull/897), [#898](https://github.com/dowdiness/canopy/pull/898), [#899](https://github.com/dowdiness/canopy/pull/899), [#900](https://github.com/dowdiness/canopy/pull/900) / incr [#405](https://github.com/dowdiness/incr/pull/405), [#406](https://github.com/dowdiness/incr/pull/406), [#407](https://github.com/dowdiness/incr/pull/407), [#408](https://github.com/dowdiness/incr/pull/408) / js_engine [#541](https://github.com/dowdiness/js_engine/pull/541), [#542](https://github.com/dowdiness/js_engine/pull/542), [#543](https://github.com/dowdiness/js_engine/pull/543), [#544](https://github.com/dowdiness/js_engine/pull/544), [#545](https://github.com/dowdiness/js_engine/pull/545), [#546](https://github.com/dowdiness/js_engine/pull/546), [#547](https://github.com/dowdiness/js_engine/pull/547) / loom [#718](https://github.com/dowdiness/loom/pull/718), [#719](https://github.com/dowdiness/loom/pull/719), [#722](https://github.com/dowdiness/loom/pull/722), [#724](https://github.com/dowdiness/loom/pull/724)

## 2026/7/17

### Canopy / ローカルLLM feasibility studyと、宙に浮いたprovider比較実装

GenUI実験の「価値の証拠」を集める作業が本格化した。人間中心の知識環境を優先する方針ドキュメント（[#901](https://github.com/dowdiness/canopy/pull/901)）を起点に、Ollamaで動くローカルLLMの技術的feasibility studyをまとめ、MoonBit側の準備コア・凍結candidate adapter・dry-run失敗のtest・v1/v2 2回の実行結果を記録した（[#902](https://github.com/dowdiness/canopy/pull/902)）。生成的UIをドキュメントエンジンとして定義するdocs（[#903](https://github.com/dowdiness/canopy/pull/903)）、外部コーディングエージェント（Codex等）がどこまで触ってよいかの境界を記録するdocs（[#904](https://github.com/dowdiness/canopy/pull/904)）、プロバイダ比較ベンチマークの計画（[#905](https://github.com/dowdiness/canopy/pull/905)）も同日に入った。

この日のagent履歴を見ると、mainへ入らなかった作業もある。`feat/genui-provider-comparison-implementation`ブランチで、Codex App Serverクライアント、fail-closedなプロセス分離、Ollamaとの比較オーケストレーション、credentialed smoke pathまで、30件近いコミットで作り込まれたが、このブランチはmainへマージされないまま残った。翌日以降、規模を絞った「1回だけの検証つきE2E」へ設計が寄っていく。

### js_engine

v0.6.0で入った永続的root `Engine`が、同期的な失敗のあとも再利用できるかを検証するtestが動き始めた（[#548](https://github.com/dowdiness/js_engine/pull/548)、マージは7/18）。

### loom

projection identity計画をアーカイブした（[#725](https://github.com/dowdiness/loom/pull/725)）。

主なPR / Issue: canopy [#901](https://github.com/dowdiness/canopy/pull/901), [#902](https://github.com/dowdiness/canopy/pull/902), [#903](https://github.com/dowdiness/canopy/pull/903), [#904](https://github.com/dowdiness/canopy/pull/904), [#905](https://github.com/dowdiness/canopy/pull/905) / loom [#725](https://github.com/dowdiness/loom/pull/725)

## 2026/7/18

### Canopy / provider実験の縮小と機会発見プロトコル

前日の大掛かりなprovider比較実装をmainへ入れる代わりに、スコープを絞った「最小provider E2E」を設計し（[#907](https://github.com/dowdiness/canopy/pull/907)）、1回のプロバイダ試行だけをMoonBit経由でcommitする最小コマンドとして実装した（[#908](https://github.com/dowdiness/canopy/pull/908)）。プロセスライフサイクルの分離やterminal outcomeの保存など、堅牢化のコミットが続く。実プロバイダ実験の次に何を試すかを決める「opportunity discovery protocol」も定義した（[#909](https://github.com/dowdiness/canopy/pull/909)）。

### incr / ドキュメント整理と契約強化

`incr_tea`のProgram mount teardownをグローバルにし（[#411](https://github.com/dowdiness/incr/pull/411)）、Datalogのrelation-ruleライフタイムを強制する修正（[#412](https://github.com/dowdiness/incr/pull/412)）が入った。ドキュメント側では、保持ポリシーの明文化、バックログ統合、アーカイブ廃止（履歴はgitに残す方針へ）、リーダーの入口整理が一気に進み（[#413](https://github.com/dowdiness/incr/pull/413)）、CI action pinningの計画も固まった（[#414](https://github.com/dowdiness/incr/pull/414)）。

### js_engine

Engine再利用の同期失敗特性テストが入った（[#548](https://github.com/dowdiness/js_engine/pull/548)）。

主なPR / Issue: canopy [#907](https://github.com/dowdiness/canopy/pull/907), [#908](https://github.com/dowdiness/canopy/pull/908), [#909](https://github.com/dowdiness/canopy/pull/909) / incr [#411](https://github.com/dowdiness/incr/pull/411), [#412](https://github.com/dowdiness/incr/pull/412), [#413](https://github.com/dowdiness/incr/pull/413), [#414](https://github.com/dowdiness/incr/pull/414) / js_engine [#548](https://github.com/dowdiness/js_engine/pull/548)

## 2026/7/19

### Canopy / Legible Instrument——GenUIのもう一つの方向

GenUIに、チャット的な自由記述UIとは違う方向が生まれた。旅程（itinerary）の意思決定を対象にした"Legible Instrument"（判読可能な道具）という設計方針を定義し、意思決定ワークスペースを実装、生成された旅程のidentity保持やインタラクション作法のdocsまで含めて仕上げた（[#913](https://github.com/dowdiness/canopy/pull/913)）。「構文的に妥当」なだけでなく「何が起きたか読み取れる」ことを狙いに据えた実験で、7/17までのprovider比較のような「まずAPIを繋ぐ」方向とは対照的に、「表示の仕方」から入っている。

### incr / ドキュメント監査

"Audit project by improve"というそのままの名前のコミットで、プロジェクト全体のドキュメント監査を行った。bare `file.md:line`リンクの検証（[#415](https://github.com/dowdiness/incr/pull/415)）、fixpointライフサイクルのdisposal契約整理と合わせ、完了済みplanのクローズが連日続いた。dataflowのretractable correctness spike（[#416](https://github.com/dowdiness/incr/pull/416)）とその独立オラクル検証（[#417](https://github.com/dowdiness/incr/pull/417)）も入った。

### loom

loomgenのpattern validatorをallowlistパーサーへ統一し（[#726](https://github.com/dowdiness/loom/pull/726)）、パターン解析の責務を分割し（[#727](https://github.com/dowdiness/loom/pull/727)）、正規表現キャプチャからトークンpayloadを抽出する変更（[#728](https://github.com/dowdiness/loom/pull/728)）が入った。emitter向けpre-mergeチェックリスト（[#729](https://github.com/dowdiness/loom/pull/729)）、grammar productionを行単位で分離する修正（[#730](https://github.com/dowdiness/loom/pull/730)）、named token-set predicateの生成（[#731](https://github.com/dowdiness/loom/pull/731)）も。

### MoonDsp

定数畳み込みを経てもruntime controlsが保持されるようにした（[#228](https://github.com/dowdiness/MoonDsp/pull/228)）。

主なPR / Issue: canopy [#913](https://github.com/dowdiness/canopy/pull/913) / incr [#415](https://github.com/dowdiness/incr/pull/415), [#416](https://github.com/dowdiness/incr/pull/416), [#417](https://github.com/dowdiness/incr/pull/417) / loom [#726](https://github.com/dowdiness/loom/pull/726), [#727](https://github.com/dowdiness/loom/pull/727), [#728](https://github.com/dowdiness/loom/pull/728), [#729](https://github.com/dowdiness/loom/pull/729), [#730](https://github.com/dowdiness/loom/pull/730), [#731](https://github.com/dowdiness/loom/pull/731) / MoonDsp [#228](https://github.com/dowdiness/MoonDsp/pull/228)

## 2026/7/20

### Canopy / GenUI semantic coreとwebの大規模モジュール化

GenUIのsemantic core Phase 0のfixtureを固め（[#914](https://github.com/dowdiness/canopy/pull/914)）、PKEチャット（過去のagent履歴を素材にした独立プロトタイプ）を試作した（[#915](https://github.com/dowdiness/canopy/pull/915)）。同日、`examples/web`全体のモジュール境界整理が一気に進み、Posts・Memo・JSON・Markdown・GenUI Possibilities・Lambdaの各featureをownership境界として抽出し、feature単位でスタイルシートの所有権も明示した（[#918](https://github.com/dowdiness/canopy/pull/918)〜[#925](https://github.com/dowdiness/canopy/pull/925)の8件）。GenUIだけでなく、web appそのものをfeatureごとに切り出す大規模リファクタが始まった。

### incr

controlled textarea（input/selectと同様の扱い）（[#418](https://github.com/dowdiness/incr/pull/418)）、fixpoint disposal契約の整理（[#420](https://github.com/dowdiness/incr/pull/420)）、typed spreadsheetへの強いcommand境界追加（[#421](https://github.com/dowdiness/incr/pull/421)）。

主なPR / Issue: canopy [#914](https://github.com/dowdiness/canopy/pull/914), [#915](https://github.com/dowdiness/canopy/pull/915), [#918](https://github.com/dowdiness/canopy/pull/918), [#919](https://github.com/dowdiness/canopy/pull/919), [#920](https://github.com/dowdiness/canopy/pull/920), [#921](https://github.com/dowdiness/canopy/pull/921), [#922](https://github.com/dowdiness/canopy/pull/922), [#923](https://github.com/dowdiness/canopy/pull/923), [#924](https://github.com/dowdiness/canopy/pull/924), [#925](https://github.com/dowdiness/canopy/pull/925) / incr [#418](https://github.com/dowdiness/incr/pull/418), [#420](https://github.com/dowdiness/incr/pull/420), [#421](https://github.com/dowdiness/incr/pull/421)

## 7月第4週: web再構成とEGW認可（7/21〜7/27）

feature境界の抽出が一段落し、EGWコラボレーション基盤の認可切り替えとWaku（React フレームワーク）移行という2つの大きな再構成が動いた週。loomはMarkdown強調記法のdelimiter-run所有権を巡る調査から、grammarのprogress/recovery契約整備へ進んだ。

## 2026/7/21

### Canopy

前日始めたweb feature境界抽出の続き。Resume relay（[#926](https://github.com/dowdiness/canopy/pull/926)）、Lambda ast-grep relay（[#927](https://github.com/dowdiness/canopy/pull/927)）、GenUI feature boundary（[#928](https://github.com/dowdiness/canopy/pull/928)）を切り離した。event-graph-walker依存を更新し、CodeRabbitのrate limitをgating対象から外した。

### js_engine

release PRのメタデータゲートを自動化した（[#549](https://github.com/dowdiness/js_engine/pull/549)）。

主なPR / Issue: canopy [#926](https://github.com/dowdiness/canopy/pull/926), [#927](https://github.com/dowdiness/canopy/pull/927), [#928](https://github.com/dowdiness/canopy/pull/928) / js_engine [#549](https://github.com/dowdiness/js_engine/pull/549)

## 2026/7/22〜7/23

canopy本体は静かな2日間だったが、js_engineとloomで並行して作業が進んだ。js_engineではEngineのcheckpoint失敗特性を洗い出し（[#551](https://github.com/dowdiness/js_engine/pull/551)）、microtask queueのポリシー遷移を独立したモジュールへ抽出し（[#555](https://github.com/dowdiness/js_engine/pull/555)）、Honoやdiagnosticsのゴールをまとめ（[#553](https://github.com/dowdiness/js_engine/pull/553)）、compat-tableのfeatureカバレッジ診断をCIへ追加した（[#556](https://github.com/dowdiness/js_engine/pull/556)）。

loomではMarkdownの強調記法まわりで「delimiter frontier」の回帰investigationを行い、クリーンなPRとして再構成した（[#736](https://github.com/dowdiness/loom/pull/736)）。続けて、継続判定（continuation decision）を素朴なbool判定から型付きの`ContinuationDecision`/`Handler`へ置き換える作業に着手し、root/setext・blockquote・list-itemの継続判定を順番に型付き化した（[#737](https://github.com/dowdiness/loom/pull/737)）。EGWとの統合をv0.5.0へ更新した（[#738](https://github.com/dowdiness/loom/pull/738)）。

主なPR / Issue: js_engine [#551](https://github.com/dowdiness/js_engine/pull/551), [#553](https://github.com/dowdiness/js_engine/pull/553), [#555](https://github.com/dowdiness/js_engine/pull/555), [#556](https://github.com/dowdiness/js_engine/pull/556) / loom [#736](https://github.com/dowdiness/loom/pull/736), [#737](https://github.com/dowdiness/loom/pull/737), [#738](https://github.com/dowdiness/loom/pull/738)

## 2026/7/24

### Canopy / EGW認可基盤とprotocol v3

collaboration（複数人での同時編集）向けにevent-graph-walker（EGW）認可基盤を導入した（[#938](https://github.com/dowdiness/canopy/pull/938)）。EGW/loomの境界を定義するdocsから始まり、EGW peer-syncのプロダクト化計画、wire非互換性の実証を経て、protocolをv3へ直接切り替えた。あわせてsyncのrecovery responseを正しいpeerへ紐づける修正（[#941](https://github.com/dowdiness/canopy/pull/941)）、Markdown block editorでのsetext heading保持（[#943](https://github.com/dowdiness/canopy/pull/943)）、モード切り替えをまたいだMarkdownブロック選択の復元（[#944](https://github.com/dowdiness/canopy/pull/944)）も入った。Markdownの意味的identityブリッジ設計のスパイクもdocs化した（[#942](https://github.com/dowdiness/canopy/pull/942)）。

### incr / EGW認可へ

typed spreadsheetのcommitted stateをEGW認可経由にルーティングした（[#422](https://github.com/dowdiness/incr/pull/422)）。pure projection core・mutable adapter shellを分け、bootstrap/local/remote/resetの認可変更を1つのprojection経路と1つのouter Runtime batchへ通し、readback失敗・rollback/retry・reset・concurrent operationsなど15件の統合テストで確認した。Plan 013 Phase 3はここで完了、ブラウザ側はまだadapter前のbaselineのまま。typed spreadsheetのcollaboration proof（[#423](https://github.com/dowdiness/incr/pull/423)）、deprecated trait呼び出しの整理（[#424](https://github.com/dowdiness/incr/pull/424)）も続いた。

### loom

EGW/egraph/egglogの警告クリーンアップpointerを更新し（[#742](https://github.com/dowdiness/loom/pull/742), [#743](https://github.com/dowdiness/loom/pull/743)）、deprecated trait呼び出しの整理（[#740](https://github.com/dowdiness/loom/pull/740), [#741](https://github.com/dowdiness/loom/pull/741), [#744](https://github.com/dowdiness/loom/pull/744)）、生成された警告サイトの除去（[#748](https://github.com/dowdiness/loom/pull/748)）を行った。moon.mod世代のsubmodule群が一斉にEGW新版へ追従した1日になる。

主なPR / Issue: canopy [#938](https://github.com/dowdiness/canopy/pull/938), [#941](https://github.com/dowdiness/canopy/pull/941), [#942](https://github.com/dowdiness/canopy/pull/942), [#943](https://github.com/dowdiness/canopy/pull/943), [#944](https://github.com/dowdiness/canopy/pull/944), [#945](https://github.com/dowdiness/canopy/pull/945) / incr [#422](https://github.com/dowdiness/incr/pull/422), [#423](https://github.com/dowdiness/incr/pull/423), [#424](https://github.com/dowdiness/incr/pull/424) / loom [#740](https://github.com/dowdiness/loom/pull/740), [#741](https://github.com/dowdiness/loom/pull/741), [#742](https://github.com/dowdiness/loom/pull/742), [#743](https://github.com/dowdiness/loom/pull/743), [#744](https://github.com/dowdiness/loom/pull/744), [#745](https://github.com/dowdiness/loom/pull/745), [#748](https://github.com/dowdiness/loom/pull/748)

## 2026/7/25

### Canopy / 依存更新

event-graph-walker submoduleを更新し（[#946](https://github.com/dowdiness/canopy/pull/946)）、Dependabotによる一連の依存更新（actions/download-artifact、actions/setup-node、vite、@ast-grep/cli、@tailwindcss/vite、ws、typescript、vitest、@types/nodeなど計9件）をまとめて取り込んだ。

### incr / Cloudflare上のリアルタイムコラボレーション

GC-rootの記録をdisposal却下時にも保持するよう修正し（[#426](https://github.com/dowdiness/incr/pull/426)）、Cloudflare上でtyped spreadsheetのリアルタイムコラボレーションを構築した（[#427](https://github.com/dowdiness/incr/pull/427)）。Cloudflare Worker本体、room単位のrelay、admission policy、ブラウザ側WebSocketトランスポート、WorkerからSPAを配信する仕組み、別ブラウザコンテキストでの協調編集検証、websocket hibernation復帰、room capabilityの脅威モデルdocs、インシデント対応の運用docsまで、1つのPRとして23コミットにわたって作り込んだ。sqliteベースのdurable object migrationをfree planで使う調整も入っている。

### js_engine

stable Engineのdiagnostic契約を決定し（[#564](https://github.com/dowdiness/js_engine/pull/564)）、class privateフィールドへの代入演算子対応（[#567](https://github.com/dowdiness/js_engine/pull/567)）、継承されたsetterでprimitive receiverを保持する修正（[#566](https://github.com/dowdiness/js_engine/pull/566)）、Hono 4.12.31の受け入れfixture追加（[#569](https://github.com/dowdiness/js_engine/pull/569)）、構造化Engine diagnosticsの追加（[#568](https://github.com/dowdiness/js_engine/pull/568)）が入った。

### loom / grammarのprogress/recovery契約

grammar（文法）のprogress契約とrecovery契約を強制する大きな変更が入った（[#756](https://github.com/dowdiness/loom/pull/756)）。同一カーソル位置でルールが再度呼ばれたときの検出ロジックはあったが、`progress_of`がreferenceされたルールのsubtreeを参照のたびに丸ごと再展開しており、参照の深さに対して指数関数的にコストが増えていた（chainの深さ22で約1秒、26で65秒）。生成された文法は兄弟位置間でfragmentを共有する形が多く、まさにこの形を踏み抜く。`(rule slot, visiting path)`をキーにしたmemo化で解消し、深さ200・400でも即座にコンパイルできるようになった。cycle guardの結果を呼び出し元へ正しく伝播させる修正（guardが発火したのに「成功した」扱いになりHTMLが偽のclose tagを出していた）、repeat groupのreuseパスにもprogress契約を適用する修正、5箇所に重複していたprogress判定を`required_progress`へ集約する整理も同じPRに含まれる。複数のコミットが"Co-Authored-By: Claude Opus 5"としてクレジットされている。lex-mode境界のdocs整理（[#758](https://github.com/dowdiness/loom/pull/758), [#759](https://github.com/dowdiness/loom/pull/759)）も同日に動いた。

主なPR / Issue: canopy [#946](https://github.com/dowdiness/canopy/pull/946) / incr [#426](https://github.com/dowdiness/incr/pull/426), [#427](https://github.com/dowdiness/incr/pull/427) / js_engine [#564](https://github.com/dowdiness/js_engine/pull/564), [#566](https://github.com/dowdiness/js_engine/pull/566), [#567](https://github.com/dowdiness/js_engine/pull/567), [#568](https://github.com/dowdiness/js_engine/pull/568), [#569](https://github.com/dowdiness/js_engine/pull/569) / loom [#756](https://github.com/dowdiness/loom/pull/756), [#758](https://github.com/dowdiness/loom/pull/758), [#759](https://github.com/dowdiness/loom/pull/759)

## 2026/7/26

### Canopy / Waku移行の設計と実装着手

統一されたWaku（Reactフレームワーク）へのweb移行計画を文書化し、移行作業のリンクや、Waku側の機能パリティが揃うまでMini-MLを残す方針を明記した（[#980](https://github.com/dowdiness/canopy/pull/980)）。同日中に実装へ着手し、dual Vite/Waku foundation、Waku Wrangler設定の分離、MoonBitプラグイン利用側の移行、Waku型契約の安定化までを1本のPRにまとめてmainへ入れた（[#983](https://github.com/dowdiness/canopy/pull/983)）。Waku Demo Hubとルートのライフサイクルも構築し（[#984](https://github.com/dowdiness/canopy/pull/984)）、Journeyページを`/journey`へ移行した（[#985](https://github.com/dowdiness/canopy/pull/985)）。Loom/Alga依存の更新（[#958](https://github.com/dowdiness/canopy/pull/958)）や、孤立サロゲート文字列の扱いを揃える修正（[#982](https://github.com/dowdiness/canopy/pull/982)）も入っている。

### incr

spreadsheetデモのテーマをimpeccable themeへ整え、typed spreadsheetのcollaboration room UIとBlank semantics（空セルの意味論）を追加した（[#428](https://github.com/dowdiness/incr/pull/428)）。

### js_engine

sloppy modeでのprimitive this値のボックス化（[#573](https://github.com/dowdiness/js_engine/pull/573)）、pending jobsのlookup特性の明文化（[#581](https://github.com/dowdiness/js_engine/pull/581)）、AST走査のcontainment traversal一元化（[#582](https://github.com/dowdiness/js_engine/pull/582)）、ES2015 early errorsの強制（[#583](https://github.com/dowdiness/js_engine/pull/583)）、格納されるゼロキーの正規化（[#584](https://github.com/dowdiness/js_engine/pull/584)）、host-owned JSONをEngineへ注入する機能（[#585](https://github.com/dowdiness/js_engine/pull/585)）、engine実行guardrail契約の決定（[#595](https://github.com/dowdiness/js_engine/pull/595)）、`ArrayBuffer`のnew target観測タイミング修正（[#591](https://github.com/dowdiness/js_engine/pull/591)、#205を閉じる）、strict blockでの関数宣言instantiation修正（[#587](https://github.com/dowdiness/js_engine/pull/587)）、`String`検索メソッドでの`Symbol.match`尊重（[#588](https://github.com/dowdiness/js_engine/pull/588)）、bound functionでのtarget prototype保持（[#589](https://github.com/dowdiness/js_engine/pull/589)）、for-of/iteratorの残存問題解消（[#594](https://github.com/dowdiness/js_engine/pull/594)、#207を閉じる）、RegExp Unicodeエスケープとcase folding対応（[#590](https://github.com/dowdiness/js_engine/pull/590)）と、spec適合の修正が10件以上まとまって入った。

### loom

json-settings authoring pipelineのベンチマークを追加し（[#761](https://github.com/dowdiness/loom/pull/761)）、opaqueな`ModeLexer`のレシピをdocs化し（[#762](https://github.com/dowdiness/loom/pull/762)）、`ParserContext`のlex-mode API（7/10導入の`lex_mode()`/`set_lex_mode`）のライフサイクルを提案し（[#763](https://github.com/dowdiness/loom/pull/763)）、そのうえで**API自体を削除した**（[#764](https://github.com/dowdiness/loom/pull/764)、breaking change）。structural prefix dispatchへの移行が進み、lex-mode APIが役目を終えたことを示す変更になる。

### MoonDsp

Windows上でのスケジューラcrackle（音声プチノイズ）調査が入った。production-pathスケジューラのテスト追加、PCM診断の比較、Windows crackle harnessの追加と続き、調査結果をdocsに記録した。

主なPR / Issue: canopy [#958](https://github.com/dowdiness/canopy/pull/958), [#980](https://github.com/dowdiness/canopy/pull/980), [#982](https://github.com/dowdiness/canopy/pull/982), [#983](https://github.com/dowdiness/canopy/pull/983), [#984](https://github.com/dowdiness/canopy/pull/984), [#985](https://github.com/dowdiness/canopy/pull/985) / incr [#428](https://github.com/dowdiness/incr/pull/428) / js_engine [#573](https://github.com/dowdiness/js_engine/pull/573), [#581](https://github.com/dowdiness/js_engine/pull/581), [#582](https://github.com/dowdiness/js_engine/pull/582), [#583](https://github.com/dowdiness/js_engine/pull/583), [#584](https://github.com/dowdiness/js_engine/pull/584), [#585](https://github.com/dowdiness/js_engine/pull/585), [#587](https://github.com/dowdiness/js_engine/pull/587), [#588](https://github.com/dowdiness/js_engine/pull/588), [#589](https://github.com/dowdiness/js_engine/pull/589), [#590](https://github.com/dowdiness/js_engine/pull/590), [#591](https://github.com/dowdiness/js_engine/pull/591), [#594](https://github.com/dowdiness/js_engine/pull/594), [#595](https://github.com/dowdiness/js_engine/pull/595) / loom [#761](https://github.com/dowdiness/loom/pull/761), [#762](https://github.com/dowdiness/loom/pull/762), [#763](https://github.com/dowdiness/loom/pull/763), [#764](https://github.com/dowdiness/loom/pull/764)

## 2026/7/27

### Canopy / Waku移行が完了し、GenUIも/genuiへ

Waku移行を一気に完走させた。Posts（[#987](https://github.com/dowdiness/canopy/pull/987)）、JSON editor（[#988](https://github.com/dowdiness/canopy/pull/988)）、Markdown editor（[#989](https://github.com/dowdiness/canopy/pull/989)）、Mini-MLとMemo（[#990](https://github.com/dowdiness/canopy/pull/990)）、Session Resume（[#991](https://github.com/dowdiness/canopy/pull/991)）を順に`/posts`・`/json`・`/markdown`などのネイティブWakuルートへ移し、**GenUIも`/genui`へ移行し、記録済みのproduction replayを保持する形で**着地させた（[#993](https://github.com/dowdiness/canopy/pull/993)）。実プロバイダとの生きた接続ではなく、記録済みのやり取りを再生する形で見せるという決定が、この移行の中で下されている。Waku Workerへの切り替え準備（[#994](https://github.com/dowdiness/canopy/pull/994)）も同日に入った。

### incr

spreadsheetのAIコンテキスト変換を一元化した（[#429](https://github.com/dowdiness/incr/pull/429)）。localityフォローアップの完了をdocsに記録した。

### js_engine

モジュール名前空間のsuper代入とTDZ修正（[#592](https://github.com/dowdiness/js_engine/pull/592)）、private実行制御のコア追加（[#598](https://github.com/dowdiness/js_engine/pull/598)）、曖昧なmodule export伝播の修正（[#593](https://github.com/dowdiness/js_engine/pull/593)）、class/superの残存問題解消（[#596](https://github.com/dowdiness/js_engine/pull/596)、#206を閉じる）、Proxyの内部操作をビルトインをまたいで正しくdispatchする修正（[#597](https://github.com/dowdiness/js_engine/pull/597)、#577を閉じる）、milestone 6 baselineの更新（[#599](https://github.com/dowdiness/js_engine/pull/599)）、execution controlをdispatchへ配線（[#600](https://github.com/dowdiness/js_engine/pull/600)）。

### loom

`ParserContext`から削除済みlex-mode APIのdead private stateを除去し（[#765](https://github.com/dowdiness/loom/pull/765)）、**parser駆動のrole span**をMarkdownへ追加した（[#766](https://github.com/dowdiness/loom/pull/766)）。7/11にJSXで先行した`#loom.recovery`系の生成パターンが、ここでMarkdownにも及んだ形になる。

主なPR / Issue: canopy [#987](https://github.com/dowdiness/canopy/pull/987), [#988](https://github.com/dowdiness/canopy/pull/988), [#989](https://github.com/dowdiness/canopy/pull/989), [#990](https://github.com/dowdiness/canopy/pull/990), [#991](https://github.com/dowdiness/canopy/pull/991), [#993](https://github.com/dowdiness/canopy/pull/993), [#994](https://github.com/dowdiness/canopy/pull/994) / incr [#429](https://github.com/dowdiness/incr/pull/429) / js_engine [#592](https://github.com/dowdiness/js_engine/pull/592), [#593](https://github.com/dowdiness/js_engine/pull/593), [#596](https://github.com/dowdiness/js_engine/pull/596), [#597](https://github.com/dowdiness/js_engine/pull/597), [#598](https://github.com/dowdiness/js_engine/pull/598), [#599](https://github.com/dowdiness/js_engine/pull/599), [#600](https://github.com/dowdiness/js_engine/pull/600) / loom [#765](https://github.com/dowdiness/loom/pull/765), [#766](https://github.com/dowdiness/loom/pull/766)

## 7月第5週: Waku完了・v0.7.0・GenUI kill date（7/28〜7/31）

Waku移行を仕上げ、js_engineがv0.7.0を出した。7/29のGenUI kill dateは、劇的な宣言なしに実質決着した。月末はbtreeのCRDT実証実験とdiagnostics基盤の立ち上げで締めくくった。

## 2026/7/28

### Canopy / Waku移行の仕上げ

CloudflareのWaku deploy設定をWorkers Buildsへ移し（[#995](https://github.com/dowdiness/canopy/pull/995)）、Waku preloadのリカバリ再読み込みを有限回に抑える修正（[#996](https://github.com/dowdiness/canopy/pull/996)）を入れた。改善計画docsのPRもマージした（[#986](https://github.com/dowdiness/canopy/pull/986)）。

### incr

決定論的なtyped spreadsheet explanationsを追加した（[#430](https://github.com/dowdiness/incr/pull/430)、「なぜこの値になったか」を説明する機能）。Rabbita explanation inspectorのDOM E2Eテストも入れた（[#431](https://github.com/dowdiness/incr/pull/431)）。

### js_engine / v0.7.0リリース

release実装状況の同期（[#602](https://github.com/dowdiness/js_engine/pull/602)）、CLIバージョンメタデータの同期（[#603](https://github.com/dowdiness/js_engine/pull/603)）を経て、**v0.7.0をリリース**した（[#550](https://github.com/dowdiness/js_engine/pull/550)、Koji Ishimoto氏がgithub-actions botと共著）。test262はPassed/Executedでstrict 91.3%・non-strict 90.7%、unit testは2,621/2,621。v0.6.0からの純増は+222 strict/+224 non-strict。構造化Engine diagnostics（`run_diagnostic`等、失敗の種類・フェーズ・操作・エンジン整合性・保留effect・pending jobsまで分類）、host-owned JSONのEngineへの直接注入、RegExp Unicodeモードのエスケープ検証とcase foldingが主な追加点になる。READMEをCI edition artifactから生成するよう切り替えた（[#613](https://github.com/dowdiness/js_engine/pull/613)）も同日。

### loom

roleが省かれる境界条件をpinするtest（[#768](https://github.com/dowdiness/loom/pull/768)）、agent向けのreadiness certification gateをCIへ追加（[#767](https://github.com/dowdiness/loom/pull/767)）、先頭バックスラッシュのparityを保持する修正（[#769](https://github.com/dowdiness/loom/pull/769)）、emphasis backslash parityのカバレッジ追加（[#462](https://github.com/dowdiness/loom/pull/462)）、**text-levelのbackslash escapeを完成させ**（[#720](https://github.com/dowdiness/loom/pull/720)）、**heading markerの形式（ATX/setext）を区別する破壊的変更**（[#747](https://github.com/dowdiness/loom/pull/747)）が続いた。lowering元ソースの共有による高速化とregression gate、非BMP文字のescaped recoveryの扱い修正も同日入っている。

主なPR / Issue: canopy [#986](https://github.com/dowdiness/canopy/pull/986), [#995](https://github.com/dowdiness/canopy/pull/995), [#996](https://github.com/dowdiness/canopy/pull/996) / incr [#430](https://github.com/dowdiness/incr/pull/430), [#431](https://github.com/dowdiness/incr/pull/431) / js_engine [#550](https://github.com/dowdiness/js_engine/pull/550), [#602](https://github.com/dowdiness/js_engine/pull/602), [#603](https://github.com/dowdiness/js_engine/pull/603), [#604](https://github.com/dowdiness/js_engine/pull/604), [#613](https://github.com/dowdiness/js_engine/pull/613) / loom [#462](https://github.com/dowdiness/loom/pull/462), [#720](https://github.com/dowdiness/loom/pull/720), [#747](https://github.com/dowdiness/loom/pull/747), [#767](https://github.com/dowdiness/loom/pull/767), [#768](https://github.com/dowdiness/loom/pull/768), [#769](https://github.com/dowdiness/loom/pull/769)

## 2026/7/29 — GenUIの kill date

7/15に設定した**GenUI継続/削除の判断日**を迎えた。mainブランチの履歴を見る限り、この日付を境にした明示的な「keep」または「delete」の宣言コミットは見当たらない。実態としては、7/17に作り込まれた大規模なCodex/Ollamaプロバイダ比較実装（`feat/genui-provider-comparison-implementation`ブランチ、30コミット近く）はマージされないまま残り、代わって7/19の"Legible Instrument"（意思決定ワークスペースとしてのGenUI）と、7/27にWaku移行の一部として`/genui`へ着地した「記録済みproduction replay」版が生き残った。判断日が来る前に、証拠不十分な部分は自然に淘汰され、証拠のあった部分だけが製品ルートとして残っていた、という形になる。

### Canopy / btreeの締めとhint inertness

legacy Vite配信を廃止し（[#997](https://github.com/dowdiness/canopy/pull/997)）、完了したWaku移行のdocsをアーカイブした（[#998](https://github.com/dowdiness/canopy/pull/998)）。coreでは、適用対象を失ったstructural identity hintを無害化する修正が入った（[#981](https://github.com/dowdiness/canopy/pull/981)）。バッチの正味の効果が打ち消し合う編集（wrapしてunwrap）で、stale hintがunhinted pathより悪い結果（identityを失う）を生むことをまずtestでpinし、`prop_unapplicable_hint_is_inert`というプロパティ名を与え、原因（LCS除外の判断がhintの「存在」ではなく「適用可能性」を見るべきだった）を特定してから直すという手順を踏んだ。複数のコミットに"Co-Authored-By: Claude Opus 5"のクレジットが付いている。

新しく独立した`dowdiness/btree`パッケージ（O(log n)の位置インデックスaccess/insert/delete/rangeを持つcounted B+ tree）のinvariant強化が始まった。min-degree演算の境界、非正のleaf span拒否、mutableな内部状態の隠蔽、insert境界の事前検証、root再初期化の禁止などをまとめて直した（[#1000](https://github.com/dowdiness/canopy/pull/1000)）。

### incr / loom

incrではbounded spreadsheet aggregate queries（集計クエリ）を追加した（[#432](https://github.com/dowdiness/incr/pull/432)）。loomでは前日分岐した`feat/markdown-escape-heading-lane`ブランチをマージし（[#770](https://github.com/dowdiness/loom/pull/770)）、event-graph-walkerを0.6.0へ更新した（[#771](https://github.com/dowdiness/loom/pull/771)）。強調記法（`*`/`_`）のデリミタランを誰が所有するかという設計判断を提案し（[#773](https://github.com/dowdiness/loom/pull/773)）、レビューを経て受理し（[#774](https://github.com/dowdiness/loom/pull/774)）、**private delimiter-run resolverのコア実装**を追加した（[#775](https://github.com/dowdiness/loom/pull/775)）。7/22のdelimiter frontier investigationから始まった調査が、ここで実装に入った。

### js_engine

releaseのcandidate manifest契約を確立し（[#629](https://github.com/dowdiness/js_engine/pull/629)）、post-parseの走査をスタックセーフにする修正を入れ（[#632](https://github.com/dowdiness/js_engine/pull/632)）、コールバリュー/文実行境界での「部分トランポリン」方式を採用したという設計判断を記録した。

主なPR / Issue: canopy [#981](https://github.com/dowdiness/canopy/pull/981), [#997](https://github.com/dowdiness/canopy/pull/997), [#998](https://github.com/dowdiness/canopy/pull/998), [#1000](https://github.com/dowdiness/canopy/pull/1000) / incr [#432](https://github.com/dowdiness/incr/pull/432) / js_engine [#629](https://github.com/dowdiness/js_engine/pull/629), [#632](https://github.com/dowdiness/js_engine/pull/632) / loom [#770](https://github.com/dowdiness/loom/pull/770), [#771](https://github.com/dowdiness/loom/pull/771), [#773](https://github.com/dowdiness/loom/pull/773), [#774](https://github.com/dowdiness/loom/pull/774), [#775](https://github.com/dowdiness/loom/pull/775)

## 2026/7/30

### Canopy / CRDTの意図保存に関する実証実験

btreeのinvariant強化が続いた。空のinsert spliceの正規化（[#1009](https://github.com/dowdiness/canopy/pull/1009)）、cumulative span overflowをatomicに拒否する修正（[#1010](https://github.com/dowdiness/canopy/pull/1010)）、任意cardinalityのcallback spliceのバランス調整（[#1026](https://github.com/dowdiness/canopy/pull/1026)）、cross-parent境界の正規化（[#1032](https://github.com/dowdiness/canopy/pull/1032)）と、この日だけで4件のfixが入った。0.2.0リリースノートの準備も始まった（[#1033](https://github.com/dowdiness/canopy/pull/1033)）。

もっとも興味深いのは、Canopyのtext CRDTと編集レイヤー（operation layer）の意図保存能力を検証した実証実験（probe）。論文"Beyond Text Editing: Algebraic Manipulation of Source Code"（Pulo, ICSME 2026）のFig. 3を、Lambda言語上のCanopyで再現した（[#1030](https://github.com/dowdiness/canopy/pull/1030)）。2つのレプリカが同じ基底から分岐し、片方は`Rename(msg -> message)`（宣言と全使用箇所）、もう片方は`MoveParam(msg, +1)`（引数リストの入れ替え）を行う。text CRDTだけを見ると、両方のマージ順序で収束はするが、**意図した意味のマージにはならず**、パースは通るのに変数が2つunboundになる。対照実験としてdisjoint renameでは意図通りにマージされ、text層が「gitなら衝突と報告するはずの場面で、静かに違う意味へ収束する」ことを実証した。

続けて、実際に出荷されているoperation layer（`lang/lambda/edits`のTreeEditOp + `compute_text_edit` + scope-graphのcapture check）で同じシナリオを再生した。4つの予測を事前にpinしたうえで検証し、(1)確認: post-move状態への`Rename`のreplayは意図通りのマージを生成（free variable 0、text-CRDT基準では2）。(2)確認＋副産物の発見: Rename×SwapChildrenは操作として可換だが、比較の過程でshipped済みSwapChildrenのtrivia欠陥（`=>`の後のスペースが消える）を発見。(3)確認: 同じペアをtext CRDT越しに見ると2つunboundになる。(4)予測より強く確認: operation層は`Rename(x->y)`を`CaptureViolation`として拒否するが、text層はそのまま適用して変数がすべて解決する（free variable 0）誤ったプログラムを生成する——**capture失敗はafter-the-factの検出には映らず、operation時点のside conditionだけがそれを捕まえられる**、という結論。

このほかWebのResume workbench paneを合成し直し（[#1028](https://github.com/dowdiness/canopy/pull/1028)）、構造化パーサー診断のeditorスライスを計画し（[#1041](https://github.com/dowdiness/canopy/pull/1041)）、order-tree境界正規化を更新し（[#1043](https://github.com/dowdiness/canopy/pull/1043)）、Markdownの見出しspanをLoomトークンへ適合させた（[#1046](https://github.com/dowdiness/canopy/pull/1046)）。

### loom / CommonMark強調記法の統合

前日のdelimiter-run resolver coreを実際のCommonMark強調解決へ統合した（[#776](https://github.com/dowdiness/loom/pull/776)）。リスト相対のblockquote prefixの消費（[#789](https://github.com/dowdiness/loom/pull/789)）、HTML blockの終了ポリシーの共有化（[#790](https://github.com/dowdiness/loom/pull/790)）、emphasis oracleをhermeticにするtest（[#791](https://github.com/dowdiness/loom/pull/791)）、delimiter-heavy parserのベンチマークgate（[#792](https://github.com/dowdiness/loom/pull/792)）、**checked canonical formatter**（不正な出力をfail-fastで検出、[#793](https://github.com/dowdiness/loom/pull/793)）が同日にまとまり、CommonMark実装が完成に近づいた。

主なPR / Issue: canopy [#1009](https://github.com/dowdiness/canopy/pull/1009), [#1010](https://github.com/dowdiness/canopy/pull/1010), [#1026](https://github.com/dowdiness/canopy/pull/1026), [#1028](https://github.com/dowdiness/canopy/pull/1028), [#1030](https://github.com/dowdiness/canopy/pull/1030), [#1032](https://github.com/dowdiness/canopy/pull/1032), [#1033](https://github.com/dowdiness/canopy/pull/1033), [#1041](https://github.com/dowdiness/canopy/pull/1041), [#1043](https://github.com/dowdiness/canopy/pull/1043), [#1046](https://github.com/dowdiness/canopy/pull/1046) / loom [#776](https://github.com/dowdiness/loom/pull/776), [#789](https://github.com/dowdiness/loom/pull/789), [#790](https://github.com/dowdiness/loom/pull/790), [#791](https://github.com/dowdiness/loom/pull/791), [#792](https://github.com/dowdiness/loom/pull/792), [#793](https://github.com/dowdiness/loom/pull/793)

## 2026/7/31

### Canopy / 診断（diagnostics）基盤の立ち上げ

月最終日は、構造化パーサー診断をeditorへ届ける基盤づくりに費やされた。構造化されたパーサー診断を保持するeditor側の修正（[#1042](https://github.com/dowdiness/canopy/pull/1042)）、btreeのスカラー不変条件の証明（[#1048](https://github.com/dowdiness/canopy/pull/1048)）、Markdownエディタchromeをレスポンシブかつ最小限にする変更（[#1049](https://github.com/dowdiness/canopy/pull/1049)）、Cloudflareデプロイのビルド出力先を修正するCI fix（[#1052](https://github.com/dowdiness/canopy/pull/1052)）、evidence-basedなUIレビュー手順のdocs化、diagnosticsの境界を決めるdocs（[#1051](https://github.com/dowdiness/canopy/pull/1051)）、PR-ready validation順序を強制するchore（[#1050](https://github.com/dowdiness/canopy/pull/1050)）が入った。

そのうえで、source-awareな診断spanをeditorへ取り込み（[#1069](https://github.com/dowdiness/canopy/pull/1069)）、source-backedなプレーンテキスト診断レンダラーを追加し（[#1085](https://github.com/dowdiness/canopy/pull/1085)）、検証済みのdiagnostic fixを追加し（[#1086](https://github.com/dowdiness/canopy/pull/1086)）、**Lambdaのdiagnosticフィックスをeditorプロトコル経由で実際に適用できる**ところまで仕上げた（[#1087](https://github.com/dowdiness/canopy/pull/1087)）。診断を「表示するだけ」から「その場で直せる」機能へ一段引き上げて、7月の作業が終わっている。

### loom / 診断spanとCommonMark監査ツールの復旧

canopyの診断基盤と歩調を合わせ、source-awareな診断spanとラベルを追加し（[#795](https://github.com/dowdiness/loom/pull/795)）、source-backedな診断レンダラーを追加し（[#796](https://github.com/dowdiness/loom/pull/796)）、検証済みdiagnostic fixを追加した（[#804](https://github.com/dowdiness/loom/pull/804)）。CommonMark監査ツールを復旧させ（[#805](https://github.com/dowdiness/loom/pull/805)）、Lambdaの「else節欠落」診断fixを追加した（[#806](https://github.com/dowdiness/loom/pull/806)）。

主なPR / Issue: canopy [#1042](https://github.com/dowdiness/canopy/pull/1042), [#1048](https://github.com/dowdiness/canopy/pull/1048), [#1049](https://github.com/dowdiness/canopy/pull/1049), [#1050](https://github.com/dowdiness/canopy/pull/1050), [#1051](https://github.com/dowdiness/canopy/pull/1051), [#1052](https://github.com/dowdiness/canopy/pull/1052), [#1069](https://github.com/dowdiness/canopy/pull/1069), [#1085](https://github.com/dowdiness/canopy/pull/1085), [#1086](https://github.com/dowdiness/canopy/pull/1086), [#1087](https://github.com/dowdiness/canopy/pull/1087) / loom [#795](https://github.com/dowdiness/loom/pull/795), [#796](https://github.com/dowdiness/loom/pull/796), [#804](https://github.com/dowdiness/loom/pull/804), [#805](https://github.com/dowdiness/loom/pull/805), [#806](https://github.com/dowdiness/loom/pull/806)

## PR索引

週ごとに折りたたんだ PR / Issue 一覧。GitHub 上の詳細への索引。

<details>
<summary>7月第1週（7/1〜7/6）</summary>

### 2026/7/1

**Canopy / loom**

canopy [#821](https://github.com/dowdiness/canopy/pull/821), [#822](https://github.com/dowdiness/canopy/pull/822), [#825](https://github.com/dowdiness/canopy/pull/825), [#826](https://github.com/dowdiness/canopy/pull/826), [#827](https://github.com/dowdiness/canopy/pull/827) / loom [#547](https://github.com/dowdiness/loom/pull/547), [#550](https://github.com/dowdiness/loom/pull/550), [#551](https://github.com/dowdiness/loom/pull/551), [#552](https://github.com/dowdiness/loom/pull/552), [#553](https://github.com/dowdiness/loom/pull/553)

### 2026/7/2

**Canopy / loom**

canopy [#832](https://github.com/dowdiness/canopy/pull/832), [#833](https://github.com/dowdiness/canopy/pull/833) / loom [#555](https://github.com/dowdiness/loom/pull/555), [#558](https://github.com/dowdiness/loom/pull/558), [#566](https://github.com/dowdiness/loom/pull/566), [#567](https://github.com/dowdiness/loom/pull/567), [#568](https://github.com/dowdiness/loom/pull/568)

### 2026/7/3

**Canopy / incr / loom**

canopy [#839](https://github.com/dowdiness/canopy/pull/839), [#840](https://github.com/dowdiness/canopy/pull/840), [#842](https://github.com/dowdiness/canopy/pull/842), [#843](https://github.com/dowdiness/canopy/pull/843), [#844](https://github.com/dowdiness/canopy/pull/844), [#845](https://github.com/dowdiness/canopy/pull/845), [#846](https://github.com/dowdiness/canopy/pull/846), [#849](https://github.com/dowdiness/canopy/pull/849), [#850](https://github.com/dowdiness/canopy/pull/850), [#851](https://github.com/dowdiness/canopy/pull/851), [#852](https://github.com/dowdiness/canopy/pull/852), [#853](https://github.com/dowdiness/canopy/pull/853) / incr [#341](https://github.com/dowdiness/incr/pull/341), [#342](https://github.com/dowdiness/incr/pull/342), [#347](https://github.com/dowdiness/incr/pull/347), [#348](https://github.com/dowdiness/incr/pull/348), [#349](https://github.com/dowdiness/incr/pull/349), [#350](https://github.com/dowdiness/incr/pull/350) / js_engine [#483](https://github.com/dowdiness/js_engine/pull/483), [#484](https://github.com/dowdiness/js_engine/pull/484) / loom [#569](https://github.com/dowdiness/loom/pull/569), [#572](https://github.com/dowdiness/loom/pull/572), [#573](https://github.com/dowdiness/loom/pull/573), [#580](https://github.com/dowdiness/loom/pull/580), [#581](https://github.com/dowdiness/loom/pull/581), [#584](https://github.com/dowdiness/loom/pull/584), [#587](https://github.com/dowdiness/loom/pull/587), [#588](https://github.com/dowdiness/loom/pull/588)

### 2026/7/4

**incr 0.13.0 / js_engine / loom**

canopy [#854](https://github.com/dowdiness/canopy/pull/854), [#855](https://github.com/dowdiness/canopy/pull/855), [#856](https://github.com/dowdiness/canopy/pull/856), [#857](https://github.com/dowdiness/canopy/pull/857), [#859](https://github.com/dowdiness/canopy/pull/859), [#860](https://github.com/dowdiness/canopy/pull/860) / incr [#351](https://github.com/dowdiness/incr/pull/351) / js_engine [#485](https://github.com/dowdiness/js_engine/pull/485), [#486](https://github.com/dowdiness/js_engine/pull/486), [#489](https://github.com/dowdiness/js_engine/pull/489), [#492](https://github.com/dowdiness/js_engine/pull/492), [#493](https://github.com/dowdiness/js_engine/pull/493) / loom [#589](https://github.com/dowdiness/loom/pull/589), [#590](https://github.com/dowdiness/loom/pull/590), [#591](https://github.com/dowdiness/loom/pull/591), [#592](https://github.com/dowdiness/loom/pull/592), [#594](https://github.com/dowdiness/loom/pull/594), [#597](https://github.com/dowdiness/loom/pull/597), [#612](https://github.com/dowdiness/loom/pull/612), [#613](https://github.com/dowdiness/loom/pull/613)

### 2026/7/5

**incr 0.14.0 / js_engine / loom**

canopy [#861](https://github.com/dowdiness/canopy/pull/861), [#870](https://github.com/dowdiness/canopy/pull/870) / incr [#352](https://github.com/dowdiness/incr/pull/352), [#353](https://github.com/dowdiness/incr/pull/353), [#354](https://github.com/dowdiness/incr/pull/354), [#355](https://github.com/dowdiness/incr/pull/355), [#357](https://github.com/dowdiness/incr/pull/357), [#358](https://github.com/dowdiness/incr/pull/358), [#359](https://github.com/dowdiness/incr/pull/359) / js_engine [#494](https://github.com/dowdiness/js_engine/pull/494), [#495](https://github.com/dowdiness/js_engine/pull/495), [#496](https://github.com/dowdiness/js_engine/pull/496), [#499](https://github.com/dowdiness/js_engine/pull/499), [#501](https://github.com/dowdiness/js_engine/pull/501) / loom [#615](https://github.com/dowdiness/loom/pull/615), [#616](https://github.com/dowdiness/loom/pull/616), [#617](https://github.com/dowdiness/loom/pull/617), [#618](https://github.com/dowdiness/loom/pull/618), [#619](https://github.com/dowdiness/loom/pull/619), [#621](https://github.com/dowdiness/loom/pull/621), [#625](https://github.com/dowdiness/loom/pull/625), [#631](https://github.com/dowdiness/loom/pull/631), [#632](https://github.com/dowdiness/loom/pull/632), [#634](https://github.com/dowdiness/loom/pull/634), [#639](https://github.com/dowdiness/loom/pull/639)

### 2026/7/6

**js_engine / loom**

incr [#359](https://github.com/dowdiness/incr/pull/359) / js_engine [#502](https://github.com/dowdiness/js_engine/pull/502) / loom [#633](https://github.com/dowdiness/loom/pull/633), [#634](https://github.com/dowdiness/loom/pull/634), [#638](https://github.com/dowdiness/loom/pull/638), [#639](https://github.com/dowdiness/loom/pull/639), [#641](https://github.com/dowdiness/loom/pull/641)

</details>

<details>
<summary>7月第2週（7/7〜7/13）</summary>

### 2026/7/7

**incr / loom**

incr [#360](https://github.com/dowdiness/incr/pull/360), [#361](https://github.com/dowdiness/incr/pull/361), [#362](https://github.com/dowdiness/incr/pull/362), [#363](https://github.com/dowdiness/incr/pull/363) / loom [#645](https://github.com/dowdiness/loom/pull/645), [#647](https://github.com/dowdiness/loom/pull/647), [#648](https://github.com/dowdiness/loom/pull/648)

### 2026/7/8

**incr ADR / MoonDsp**

incr [#372](https://github.com/dowdiness/incr/pull/372) / loom [#650](https://github.com/dowdiness/loom/pull/650) / MoonDsp [#227](https://github.com/dowdiness/MoonDsp/pull/227)

### 2026/7/9

**incr再入ガード / js_engine / loom**

canopy [#871](https://github.com/dowdiness/canopy/pull/871) / incr [#373](https://github.com/dowdiness/incr/pull/373), [#374](https://github.com/dowdiness/incr/pull/374), [#378](https://github.com/dowdiness/incr/pull/378), [#379](https://github.com/dowdiness/incr/pull/379), [#380](https://github.com/dowdiness/incr/pull/380), [#381](https://github.com/dowdiness/incr/pull/381), [#382](https://github.com/dowdiness/incr/pull/382), [#383](https://github.com/dowdiness/incr/pull/383), [#384](https://github.com/dowdiness/incr/pull/384) / js_engine [#503](https://github.com/dowdiness/js_engine/pull/503), [#508](https://github.com/dowdiness/js_engine/pull/508), [#509](https://github.com/dowdiness/js_engine/pull/509) / loom [#643](https://github.com/dowdiness/loom/pull/643), [#649](https://github.com/dowdiness/loom/pull/649), [#651](https://github.com/dowdiness/loom/pull/651), [#653](https://github.com/dowdiness/loom/pull/653), [#654](https://github.com/dowdiness/loom/pull/654), [#655](https://github.com/dowdiness/loom/pull/655), [#656](https://github.com/dowdiness/loom/pull/656), [#659](https://github.com/dowdiness/loom/pull/659), [#660](https://github.com/dowdiness/loom/pull/660), [#661](https://github.com/dowdiness/loom/pull/661)

### 2026/7/10

**emit_grammar削除 / cache-for-X**

canopy [#873](https://github.com/dowdiness/canopy/pull/873) / incr [#385](https://github.com/dowdiness/incr/pull/385), [#386](https://github.com/dowdiness/incr/pull/386), [#388](https://github.com/dowdiness/incr/pull/388) / js_engine [#510](https://github.com/dowdiness/js_engine/pull/510), [#511](https://github.com/dowdiness/js_engine/pull/511), [#513](https://github.com/dowdiness/js_engine/pull/513), [#514](https://github.com/dowdiness/js_engine/pull/514) / loom [#662](https://github.com/dowdiness/loom/pull/662), [#663](https://github.com/dowdiness/loom/pull/663), [#666](https://github.com/dowdiness/loom/pull/666), [#667](https://github.com/dowdiness/loom/pull/667), [#672](https://github.com/dowdiness/loom/pull/672), [#675](https://github.com/dowdiness/loom/pull/675), [#676](https://github.com/dowdiness/loom/pull/676)

### 2026/7/11

**JSX / js_engine / loom**

canopy [#875](https://github.com/dowdiness/canopy/pull/875), [#876](https://github.com/dowdiness/canopy/pull/876), [#877](https://github.com/dowdiness/canopy/pull/877), [#878](https://github.com/dowdiness/canopy/pull/878) / incr [#389](https://github.com/dowdiness/incr/pull/389) / js_engine [#515](https://github.com/dowdiness/js_engine/pull/515), [#516](https://github.com/dowdiness/js_engine/pull/516), [#520](https://github.com/dowdiness/js_engine/pull/520), [#521](https://github.com/dowdiness/js_engine/pull/521), [#522](https://github.com/dowdiness/js_engine/pull/522), [#523](https://github.com/dowdiness/js_engine/pull/523), [#524](https://github.com/dowdiness/js_engine/pull/524), [#526](https://github.com/dowdiness/js_engine/pull/526), [#527](https://github.com/dowdiness/js_engine/pull/527) / loom [#679](https://github.com/dowdiness/loom/pull/679), [#680](https://github.com/dowdiness/loom/pull/680), [#682](https://github.com/dowdiness/loom/pull/682), [#684](https://github.com/dowdiness/loom/pull/684), [#686](https://github.com/dowdiness/loom/pull/686), [#690](https://github.com/dowdiness/loom/pull/690), [#692](https://github.com/dowdiness/loom/pull/692), [#693](https://github.com/dowdiness/loom/pull/693), [#696](https://github.com/dowdiness/loom/pull/696), [#698](https://github.com/dowdiness/loom/pull/698)

### 2026/7/12

**GenUI / js_engine / loom**

canopy [#885](https://github.com/dowdiness/canopy/pull/885), [#887](https://github.com/dowdiness/canopy/pull/887) / js_engine [#532](https://github.com/dowdiness/js_engine/pull/532) / loom [#703](https://github.com/dowdiness/loom/pull/703)

### 2026/7/13

**GenUI垂直スライス / line-mode lexer**

canopy [#890](https://github.com/dowdiness/canopy/pull/890), [#891](https://github.com/dowdiness/canopy/pull/891), [#892](https://github.com/dowdiness/canopy/pull/892) / incr [#390](https://github.com/dowdiness/incr/pull/390), [#391](https://github.com/dowdiness/incr/pull/391), [#392](https://github.com/dowdiness/incr/pull/392), [#393](https://github.com/dowdiness/incr/pull/393), [#397](https://github.com/dowdiness/incr/pull/397) / js_engine [#534](https://github.com/dowdiness/js_engine/pull/534), [#535](https://github.com/dowdiness/js_engine/pull/535), [#536](https://github.com/dowdiness/js_engine/pull/536), [#537](https://github.com/dowdiness/js_engine/pull/537), [#538](https://github.com/dowdiness/js_engine/pull/538) / loom [#704](https://github.com/dowdiness/loom/pull/704), [#705](https://github.com/dowdiness/loom/pull/705), [#706](https://github.com/dowdiness/loom/pull/706), [#707](https://github.com/dowdiness/loom/pull/707), [#708](https://github.com/dowdiness/loom/pull/708), [#709](https://github.com/dowdiness/loom/pull/709), [#711](https://github.com/dowdiness/loom/pull/711)

</details>

<details>
<summary>7月第3週（7/14〜7/20）</summary>

### 2026/7/14

**incr / loom**

incr [#398](https://github.com/dowdiness/incr/pull/398), [#400](https://github.com/dowdiness/incr/pull/400) / loom [#710](https://github.com/dowdiness/loom/pull/710), [#713](https://github.com/dowdiness/loom/pull/713), [#714](https://github.com/dowdiness/loom/pull/714), [#715](https://github.com/dowdiness/loom/pull/715), [#717](https://github.com/dowdiness/loom/pull/717)

### 2026/7/15

**GenUI kill date / v0.6.0**

canopy [#893](https://github.com/dowdiness/canopy/pull/893), [#894](https://github.com/dowdiness/canopy/pull/894), [#895](https://github.com/dowdiness/canopy/pull/895), [#897](https://github.com/dowdiness/canopy/pull/897) / incr [#401](https://github.com/dowdiness/incr/pull/401), [#402](https://github.com/dowdiness/incr/pull/402), [#403](https://github.com/dowdiness/incr/pull/403), [#404](https://github.com/dowdiness/incr/pull/404) / js_engine [#539](https://github.com/dowdiness/js_engine/pull/539), [#540](https://github.com/dowdiness/js_engine/pull/540), [#541](https://github.com/dowdiness/js_engine/pull/541) / loom [#718](https://github.com/dowdiness/loom/pull/718)

### 2026/7/16

**v0.6.0リリース / GenUIゲーティング**

canopy [#896](https://github.com/dowdiness/canopy/pull/896), [#897](https://github.com/dowdiness/canopy/pull/897), [#898](https://github.com/dowdiness/canopy/pull/898), [#899](https://github.com/dowdiness/canopy/pull/899), [#900](https://github.com/dowdiness/canopy/pull/900) / incr [#405](https://github.com/dowdiness/incr/pull/405), [#406](https://github.com/dowdiness/incr/pull/406), [#407](https://github.com/dowdiness/incr/pull/407), [#408](https://github.com/dowdiness/incr/pull/408) / js_engine [#541](https://github.com/dowdiness/js_engine/pull/541), [#542](https://github.com/dowdiness/js_engine/pull/542), [#543](https://github.com/dowdiness/js_engine/pull/543), [#544](https://github.com/dowdiness/js_engine/pull/544), [#545](https://github.com/dowdiness/js_engine/pull/545), [#546](https://github.com/dowdiness/js_engine/pull/546), [#547](https://github.com/dowdiness/js_engine/pull/547) / loom [#718](https://github.com/dowdiness/loom/pull/718), [#719](https://github.com/dowdiness/loom/pull/719), [#722](https://github.com/dowdiness/loom/pull/722), [#724](https://github.com/dowdiness/loom/pull/724)

### 2026/7/17

**ローカルLLM feasibility / 未マージのprovider比較**

canopy [#901](https://github.com/dowdiness/canopy/pull/901), [#902](https://github.com/dowdiness/canopy/pull/902), [#903](https://github.com/dowdiness/canopy/pull/903), [#904](https://github.com/dowdiness/canopy/pull/904), [#905](https://github.com/dowdiness/canopy/pull/905) / loom [#725](https://github.com/dowdiness/loom/pull/725)

### 2026/7/18

**最小provider E2E / opportunity discovery**

canopy [#907](https://github.com/dowdiness/canopy/pull/907), [#908](https://github.com/dowdiness/canopy/pull/908), [#909](https://github.com/dowdiness/canopy/pull/909) / incr [#411](https://github.com/dowdiness/incr/pull/411), [#412](https://github.com/dowdiness/incr/pull/412), [#413](https://github.com/dowdiness/incr/pull/413), [#414](https://github.com/dowdiness/incr/pull/414) / js_engine [#548](https://github.com/dowdiness/js_engine/pull/548)

### 2026/7/19

**Legible Instrument / loomgen pattern validator**

canopy [#913](https://github.com/dowdiness/canopy/pull/913) / incr [#415](https://github.com/dowdiness/incr/pull/415), [#416](https://github.com/dowdiness/incr/pull/416), [#417](https://github.com/dowdiness/incr/pull/417) / loom [#726](https://github.com/dowdiness/loom/pull/726), [#727](https://github.com/dowdiness/loom/pull/727), [#728](https://github.com/dowdiness/loom/pull/728), [#729](https://github.com/dowdiness/loom/pull/729), [#730](https://github.com/dowdiness/loom/pull/730), [#731](https://github.com/dowdiness/loom/pull/731) / MoonDsp [#228](https://github.com/dowdiness/MoonDsp/pull/228)

### 2026/7/20

**web feature境界抽出**

canopy [#914](https://github.com/dowdiness/canopy/pull/914), [#915](https://github.com/dowdiness/canopy/pull/915), [#918](https://github.com/dowdiness/canopy/pull/918), [#919](https://github.com/dowdiness/canopy/pull/919), [#920](https://github.com/dowdiness/canopy/pull/920), [#921](https://github.com/dowdiness/canopy/pull/921), [#922](https://github.com/dowdiness/canopy/pull/922), [#923](https://github.com/dowdiness/canopy/pull/923), [#924](https://github.com/dowdiness/canopy/pull/924), [#925](https://github.com/dowdiness/canopy/pull/925) / incr [#418](https://github.com/dowdiness/incr/pull/418), [#420](https://github.com/dowdiness/incr/pull/420), [#421](https://github.com/dowdiness/incr/pull/421)

</details>

<details>
<summary>7月第4週（7/21〜7/27）</summary>

### 2026/7/21

**web境界分離**

canopy [#926](https://github.com/dowdiness/canopy/pull/926), [#927](https://github.com/dowdiness/canopy/pull/927), [#928](https://github.com/dowdiness/canopy/pull/928) / js_engine [#549](https://github.com/dowdiness/js_engine/pull/549)

### 2026/7/22〜7/23

**delimiter frontier / continuation型付け**

js_engine [#551](https://github.com/dowdiness/js_engine/pull/551), [#553](https://github.com/dowdiness/js_engine/pull/553), [#555](https://github.com/dowdiness/js_engine/pull/555), [#556](https://github.com/dowdiness/js_engine/pull/556) / loom [#736](https://github.com/dowdiness/loom/pull/736), [#737](https://github.com/dowdiness/loom/pull/737), [#738](https://github.com/dowdiness/loom/pull/738)

### 2026/7/24

**EGW認可基盤 / protocol v3**

canopy [#938](https://github.com/dowdiness/canopy/pull/938), [#941](https://github.com/dowdiness/canopy/pull/941), [#942](https://github.com/dowdiness/canopy/pull/942), [#943](https://github.com/dowdiness/canopy/pull/943), [#944](https://github.com/dowdiness/canopy/pull/944), [#945](https://github.com/dowdiness/canopy/pull/945) / incr [#422](https://github.com/dowdiness/incr/pull/422), [#423](https://github.com/dowdiness/incr/pull/423), [#424](https://github.com/dowdiness/incr/pull/424) / loom [#740](https://github.com/dowdiness/loom/pull/740), [#741](https://github.com/dowdiness/loom/pull/741), [#742](https://github.com/dowdiness/loom/pull/742), [#743](https://github.com/dowdiness/loom/pull/743), [#744](https://github.com/dowdiness/loom/pull/744), [#745](https://github.com/dowdiness/loom/pull/745), [#748](https://github.com/dowdiness/loom/pull/748)

### 2026/7/25

**Cloudflareコラボレーション / grammar progress契約**

canopy [#946](https://github.com/dowdiness/canopy/pull/946) / incr [#426](https://github.com/dowdiness/incr/pull/426), [#427](https://github.com/dowdiness/incr/pull/427) / js_engine [#564](https://github.com/dowdiness/js_engine/pull/564), [#566](https://github.com/dowdiness/js_engine/pull/566), [#567](https://github.com/dowdiness/js_engine/pull/567), [#568](https://github.com/dowdiness/js_engine/pull/568), [#569](https://github.com/dowdiness/js_engine/pull/569) / loom [#756](https://github.com/dowdiness/loom/pull/756), [#758](https://github.com/dowdiness/loom/pull/758), [#759](https://github.com/dowdiness/loom/pull/759)

### 2026/7/26

**Waku移行着手**

canopy [#958](https://github.com/dowdiness/canopy/pull/958), [#980](https://github.com/dowdiness/canopy/pull/980), [#982](https://github.com/dowdiness/canopy/pull/982), [#983](https://github.com/dowdiness/canopy/pull/983), [#984](https://github.com/dowdiness/canopy/pull/984), [#985](https://github.com/dowdiness/canopy/pull/985) / incr [#428](https://github.com/dowdiness/incr/pull/428) / js_engine [#573](https://github.com/dowdiness/js_engine/pull/573), [#581](https://github.com/dowdiness/js_engine/pull/581), [#582](https://github.com/dowdiness/js_engine/pull/582), [#583](https://github.com/dowdiness/js_engine/pull/583), [#584](https://github.com/dowdiness/js_engine/pull/584), [#585](https://github.com/dowdiness/js_engine/pull/585), [#587](https://github.com/dowdiness/js_engine/pull/587), [#588](https://github.com/dowdiness/js_engine/pull/588), [#589](https://github.com/dowdiness/js_engine/pull/589), [#590](https://github.com/dowdiness/js_engine/pull/590), [#591](https://github.com/dowdiness/js_engine/pull/591), [#594](https://github.com/dowdiness/js_engine/pull/594), [#595](https://github.com/dowdiness/js_engine/pull/595) / loom [#761](https://github.com/dowdiness/loom/pull/761), [#762](https://github.com/dowdiness/loom/pull/762), [#763](https://github.com/dowdiness/loom/pull/763), [#764](https://github.com/dowdiness/loom/pull/764)

### 2026/7/27

**Waku移行完了 / GenUIが/genuiへ**

canopy [#987](https://github.com/dowdiness/canopy/pull/987), [#988](https://github.com/dowdiness/canopy/pull/988), [#989](https://github.com/dowdiness/canopy/pull/989), [#990](https://github.com/dowdiness/canopy/pull/990), [#991](https://github.com/dowdiness/canopy/pull/991), [#993](https://github.com/dowdiness/canopy/pull/993), [#994](https://github.com/dowdiness/canopy/pull/994) / incr [#429](https://github.com/dowdiness/incr/pull/429) / js_engine [#592](https://github.com/dowdiness/js_engine/pull/592), [#593](https://github.com/dowdiness/js_engine/pull/593), [#596](https://github.com/dowdiness/js_engine/pull/596), [#597](https://github.com/dowdiness/js_engine/pull/597), [#598](https://github.com/dowdiness/js_engine/pull/598), [#599](https://github.com/dowdiness/js_engine/pull/599), [#600](https://github.com/dowdiness/js_engine/pull/600) / loom [#765](https://github.com/dowdiness/loom/pull/765), [#766](https://github.com/dowdiness/loom/pull/766)

</details>

<details>
<summary>7月第5週（7/28〜7/31）</summary>

### 2026/7/28

**Waku仕上げ / js_engine v0.7.0**

canopy [#986](https://github.com/dowdiness/canopy/pull/986), [#995](https://github.com/dowdiness/canopy/pull/995), [#996](https://github.com/dowdiness/canopy/pull/996) / incr [#430](https://github.com/dowdiness/incr/pull/430), [#431](https://github.com/dowdiness/incr/pull/431) / js_engine [#550](https://github.com/dowdiness/js_engine/pull/550), [#602](https://github.com/dowdiness/js_engine/pull/602), [#603](https://github.com/dowdiness/js_engine/pull/603), [#604](https://github.com/dowdiness/js_engine/pull/604), [#613](https://github.com/dowdiness/js_engine/pull/613) / loom [#462](https://github.com/dowdiness/loom/pull/462), [#720](https://github.com/dowdiness/loom/pull/720), [#747](https://github.com/dowdiness/loom/pull/747), [#767](https://github.com/dowdiness/loom/pull/767), [#768](https://github.com/dowdiness/loom/pull/768), [#769](https://github.com/dowdiness/loom/pull/769)

### 2026/7/29

**GenUI kill date / btree始動**

canopy [#981](https://github.com/dowdiness/canopy/pull/981), [#997](https://github.com/dowdiness/canopy/pull/997), [#998](https://github.com/dowdiness/canopy/pull/998), [#1000](https://github.com/dowdiness/canopy/pull/1000) / incr [#432](https://github.com/dowdiness/incr/pull/432) / js_engine [#629](https://github.com/dowdiness/js_engine/pull/629), [#632](https://github.com/dowdiness/js_engine/pull/632) / loom [#770](https://github.com/dowdiness/loom/pull/770), [#771](https://github.com/dowdiness/loom/pull/771), [#773](https://github.com/dowdiness/loom/pull/773), [#774](https://github.com/dowdiness/loom/pull/774), [#775](https://github.com/dowdiness/loom/pull/775)

### 2026/7/30

**CRDT意図保存probe / CommonMark強調統合**

canopy [#1009](https://github.com/dowdiness/canopy/pull/1009), [#1010](https://github.com/dowdiness/canopy/pull/1010), [#1026](https://github.com/dowdiness/canopy/pull/1026), [#1028](https://github.com/dowdiness/canopy/pull/1028), [#1030](https://github.com/dowdiness/canopy/pull/1030), [#1032](https://github.com/dowdiness/canopy/pull/1032), [#1033](https://github.com/dowdiness/canopy/pull/1033), [#1041](https://github.com/dowdiness/canopy/pull/1041), [#1043](https://github.com/dowdiness/canopy/pull/1043), [#1046](https://github.com/dowdiness/canopy/pull/1046) / loom [#776](https://github.com/dowdiness/loom/pull/776), [#789](https://github.com/dowdiness/loom/pull/789), [#790](https://github.com/dowdiness/loom/pull/790), [#791](https://github.com/dowdiness/loom/pull/791), [#792](https://github.com/dowdiness/loom/pull/792), [#793](https://github.com/dowdiness/loom/pull/793)

### 2026/7/31

**診断基盤 / CommonMark監査復旧**

canopy [#1042](https://github.com/dowdiness/canopy/pull/1042), [#1048](https://github.com/dowdiness/canopy/pull/1048), [#1049](https://github.com/dowdiness/canopy/pull/1049), [#1050](https://github.com/dowdiness/canopy/pull/1050), [#1051](https://github.com/dowdiness/canopy/pull/1051), [#1052](https://github.com/dowdiness/canopy/pull/1052), [#1069](https://github.com/dowdiness/canopy/pull/1069), [#1085](https://github.com/dowdiness/canopy/pull/1085), [#1086](https://github.com/dowdiness/canopy/pull/1086), [#1087](https://github.com/dowdiness/canopy/pull/1087) / loom [#795](https://github.com/dowdiness/loom/pull/795), [#796](https://github.com/dowdiness/loom/pull/796), [#804](https://github.com/dowdiness/loom/pull/804), [#805](https://github.com/dowdiness/loom/pull/805), [#806](https://github.com/dowdiness/loom/pull/806)

</details>
