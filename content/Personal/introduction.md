---
title: Introduction
publish: true
tags: [personal, portfolio]
aliases: [Developer Portfolio(仮), introduction]
created: 2025-12-21T13:18:07+09:00
modified: 2026-03-13T14:43:09+09:00
---

# Developer Portfolio(仮)

## 石本 幸士

**Location:** Kyoto, Japan
**GitHub:** [@dowdiness](https://github.com/dowdiness)

## 自己紹介

インタラクティブな体験づくりを軸に、フロントエンド開発へ取り組んできました。Max/MSP/Jitterでの制作を原点に、Haskellで宣言的な設計思想を学び、現在はReactを中心に個人開発・コミュニティ活動を継続しています。PyCon公式サイトや「弱いzine」へのボランティア参加、ラムダ計算エディター、React Flowを用いたブラウザシンセ「フローサウンド」などを通じて、操作感と実装の見通しを両立させるUI開発を強みとして磨いてきました。今後はフロントエンドエンジニアとして、業務でもこの強みを活かしていきます。

## スキルセット

### プログラミング言語

- **主に使っている言語:** TypeScript, Ruby
- 一番書いたことのある言語はTypeScriptです。現在は参加していませんが、[ECMAScript 仕様輪読会](https://esspec.connpass.com/)にてECMAScriptの仕様を読んでいました。JavaScriptの仕様には詳しい方だと思います。
- RubyはRailsでのバックエンドのコードと、簡単なスクリプトや競技プログラミングに参加する際に使っていました。
- **興味のある言語:** ReScript、MoonBit、Haskell、Rust
- 関数型言語が好みかつ、静的型付言語の方が得意です。コンパイラやプログラミング言語の理論に興味があります。趣味のコンパイラ開発やPoCとしてアイディアを試す際には[Haskell](https://github.com/dowdiness/scheme-in-haskell)など関数型言語を使っています。

### フロントエンド

- **UI Library:** React、Vue.js、Alpine.js
- **Framework:** Next.js、Nuxt.js、Astro.js
- **Tools:** Vite、typia、tailwindcss

Reactを使ったフロントエンドの開発が得意です。Reactのコンポーネントのレンダーやエフェクト、状態管理のライフサイクルの仕組みに関する理解、React内部の状態と外部に持っている状態との連携などには自信があります。

デザインやユーザーにとっての使いやすさを考えるのが好きです。待ち時間が発生しない軽いサイトになるように、コード量の削減や遅延ロードなどサイトを重くしない工夫は常に意識しています。コンパイラの勉強をしていることもありバンドラーの仕組みにも詳しいです。

静的サイトやJS側のロジックの少ないサイトの場合は、Astro.jsとAlpine.jsも選択肢に入れています。

### バックエンド

- Node.js、Ruby on Rails、PostgreSQL
- TypeScriptを使ったフルスタック開発に興味があります。
- フロントエンドに比べるとバックエンドの知見が少ないので、Jamstack寄りの構成を選びがちです。

### 語学

- 日常的に英語の本を読んだりポッドキャストを聞いています [TOEIC 840点](https://iibc.cloudcerts.jp/viewer/cert/5aJemlWBgNAqgu68NgOA5VmIbVAVQ8JR0MzOrkQOYdWeF9qPgV9BbkuOeKl8N9JP)
- 中国語も趣味でやっています

## プロジェクト

### [Lambda Calculus CRDT Editor](https://lambda-editor.koji-ishimoto.workers.dev/)

*TypeScript*
- [GitHub](https://github.com/dowdiness/til/tree/main/crdt)
- [eg-walkerの論文](https://arxiv.org/abs/2409.14252)

複数人で同時に編集してもデータが壊れない仕組み（CRDT）を用いたライブラリを自作し、その上でラムダ計算をリアルタイムで共同編集できるエディタを開発。論文を読み、アルゴリズムを理解した上で、実際に動くソフトウェアとして実装し、UIまで含めて１人で作成しています。

### [tapl-rescript](https://github.com/dowdiness/tapl-rescript)

*JavaScript / ReScript*

- [「Types and Programming Languages（型システム入門）」](https://www.ohmsha.co.jp/book/9784274069116/)の実装を[ReScript](https://rescript-lang.org/)で行うプロジェクト
- 2023年から2024年まで[TAPL.ts](https://taplts.connpass.com)にて、型システム入門の輪読会に参加
- LLVM IRへとコンパイルするλ計算のコンパイラを自分で実装し、理論として学んだことを実践
- [NPM Package](https://www.npmjs.com/package/@antisatori/tapl) として公開
- [MoonBit版](https://github.com/dowdiness/tapl-rescript/tree/moon-migration/moonbit)への書き換えも進行中

### フローサウンド

*TypeScript / React / React Flow*
- [GitHub](https://github.com/dowdiness/flow-sound)

テキストだけでのコード編集ではない、意味論ベースでのProjectional Editingに興味があります。DSLのビジュアルベースでのUI/UXの探求のためにノードベースでのオーディオプログラミング環境の構築に挑みました。

### [弱いzine](https://yowai.band)

*TypeScript / Gatsby / React*
- [GitHub](https://github.com/dowdiness/yowai-zine)

「こころおきなく居られるweb zine」をコンセプトに、友人と共同で制作したWeb雑誌サイト。機能性よりも「触りたくなる体験」を重視し、以下の技術的課題に取り組みました。

- CSSレイアウトの設計を工夫し、ブラウザ上での**縦書き記事表示**を実現
- Web Audio APIを活用した**サイト内音楽プレイヤー**の構築
- アニメーションを多用しつつ、読書体験を妨げない**没入感のあるインタラクション設計**

デザイン・実装ともに担当。「心地よい操作感」と「コンテンツへの集中」を両立させるUI設計を実践しました。

### [pycon.jp 2020 公式サイト](https://pycon.jp/2020/)

*Vue / Nuxt.js / tailwindcss*
- [GitHub](https://github.com/pyconjp/pycon.jp.2020.ui)

PyCon.jp 2020 の公式サイト制作にボランティアとして参加。主に[papi-tokei](https://github.com/papi-tokei)と2人で作成。Nuxt.jsのセットアップやサイト全体のラフなデザインを担当し、イベントの詳細が決まるより先に、後から対応できる柔軟な枠組みづくりを行いました。

### [twitter-clone](https://github.com/dowdiness/twitter-clone)

*Nuxt.js / Firebase*
- FirebaseとNuxt.jsを使用した簡易的なSNSアプリ
- フルスタックWeb開発のスキルを実証

## 学びの蓄積

**[MainVault](https://github.com/dowdiness/MainVault)**
- Obsidianを使用して学んだことをメモとして管理し、[Quartz](https://github.com/jackyzha0/quartz)により記事として公開しています

## 興味・学習の方向性

- [Ink&Switch](https://www.inkandswitch.com/) や [Future of Programming Lab](https://neurocy.notion.site/Future-of-Programming-Lab-241d162461a04064ae1fd9ae32bf4cb1) が取り組むような、**開発者体験を進化させるプログラミング環境やツールの研究・実装**
- コンパイラ設計と型システムの理論と実践
- 形式手法（formal methods）、特に[Lean](https://lean-lang.org/)や[TLA+](https://lamport.azurewebsites.net/tla/tla.html)のような証明のできるシステム
- コンピュータサイエンスの基礎の深い理解
- オペレーティングシステムの内部構造

---
