---
title: Introduction
publish: true
tags: [personal, portfolio]
aliases: [Developer Portfolio(仮), introduction]
created: 2025-12-21T13:18:07+09:00
modified: 2026-01-19T20:23:19+09:00
---

# Developer Portfolio(仮)

## 石本 幸士 (Ishimoto Koji)

**Location:** Kyoto, Japan
**GitHub:** [@dowdiness](https://github.com/dowdiness)

## About Me

コンパイラーとComputer Scienceに深い興味を持つソフトウェア開発者です。好奇心旺盛で、言語学習や異文化理解を通じて新しい視点や出会いを大切にしています。技術的な探求と実用的な開発の両面で、継続的に学び続けることを心がけています。

インタラクティブなものが好きだからプログラミング

### 価値観

- **コンテキストの共有**を重視し、相手が理解しやすい形でコミュニケーションを取ることを心がけています
- 外国語学習を通じて、日本語圏では出会えない知識や人々と繋がることに価値を感じています
- 技術の本質的な理解を追求し、基礎から応用まで幅広く学ぶことを大切にしています

## スキルセット

### 興味関心

- フルスタックなウェブ開発
- コンパイラ、型システム、Structure editor
- 自作言語に興味があります
- 形式手法（formal methods）、特に[Lean](https://lean-lang.org/)や[TLA+](https://lamport.azurewebsites.net/tla/tla.html)のような証明の出来るシステム

### プログラミング言語

- **主に使っている言語:** TypeScript, Ruby
- 一番書いたことのある言語はTypeScriptです。現在は参加していませんが、[ECMAScript 仕様輪読会](https://esspec.connpass.com/)にてECMAScriptの仕様を読んでいました。JavaScriptの仕様には詳しい方だと思います。
- RubyはRailsでのバックエンドのコードと、簡単なスクリプトや競技プログラミングに参加する際に使っていました。
- **興味のある言語:** ReScript、MoonBit、Haskell、Rust
- 関数型言語が好みかつ、静的型付言語の方が得意です。コンパイラやプログラミング言語の理論に興味があります。趣味のコンパイラ開発やPoCとしてアイディアを試す際には[Haskell](https://github.com/dowdiness/scheme-in-haskell)など関数型言語を使っています。

### フロントエンド

- **UI Library:** React、 Vue.js、 Alpine.js
- **FrameWork**: Next.js、 Nuxt.js、 Astro.js
- **Tools:** Vite、typia、tailwindcss、

Reactを使ったフロントエンドの開発が得意です。Reactのコンポーメントのレンダーやエフェクト、状態管理のライフサイクルの仕組みに関する理解、React内部の状態と外部に持っている状態との連携などには自信があります。使い勝手やシンプルさなどを考慮するとフロントエンドの開発にはReactが一番適していると思っています。

デザインやユーザーにとっての使いやすさを考えるのが好きです。待ち時間が発生しない軽いサイトになるように、コード量の削減や遅延ロードなどサイトを重くしない工夫は常に意識しています。コンパイラの勉強をしていることもありバンドラーに仕組みにも詳しいです。

静的サイトやJS側のロジックの少ないサイトの場合は、Astro.jsとAlpine.jsも選択肢に入れています。

### バックエンド

- Node.js、Ruby on Rails、PostgreSQL
- TypeScriptを使ったフルスタック開発に興味があります。
- フロントエンドに比べるとバックエンドの知見が少ないので、Jamstack寄りの構成を選びがちです。

## 取り組んでいるプロジェクト

### **[tapl-rescript](https://github.com/dowdiness/tapl-rescript)**

*JavaScript/ReScript*
- [「Types and Programming Languages（型システム入門）」](https://www.ohmsha.co.jp/book/9784274069116/)の実装を[ReScript](https://rescript-lang.org/)で行うプロジェクトです
- 2023年から2024年まで[TAPL.ts](https://taplts.connpass.com)にて、型システム入門の輪読会に参加していました
- LLVM IRへとコンパイルするλ計算のコンパイラを自分で実装することにより、理論として学んだことの実践をしています
- まだまだ未完成なところも多いですが [NPM Package](https://www.npmjs.com/package/@antisatori/tapl) として公開しています
- Moonbitを試してみたいと思い[書き換えた](https://github.com/dowdiness/tapl-rescript/tree/moon-migration/moonbit)バージョンを現在作っています

### **[twitter-clone](https://github.com/dowdiness/twitter-clone)**

*Nuxt.js*, *Firebase*
- FirebaseとNuxt.jsを使用した簡易的なＳＮＳアプリです
- フルスタックWeb開発のスキルを実証
- 少し内容が古いかもしれません

https://github.com/dowdiness/flow-sound/tree/main

https://github.com/dowdiness/til/tree/main/audiocontext-vite

## デザイン

### [pycon.jp 2020 公式サイト](https://pycon.jp/2020/)
*Vue*, *Nuxt.js*, *tailwindcss*

- [GitHub](https://github.com/pyconjp/pycon.jp.2020.ui)
- Pycon.jp 2020 の公式サイト制作に関わりました。
- 主に私と[papi-tokei](https://github.com/papi-tokei)で作成しました。
- 私はNuxt.jsのセットアップやサイト全体のラフなデザインをしています。イベントのイメージカラーやサイトのデザインのイメージなどが決まるよりも先に制作を始めており、後に詳細が決まった際に対応できるような枠組み作りをしました。

### **[yowai-zine](https://yowai.band)**
*TypeScript*, *Gatsby*

- [GitHub](https://github.com/dowdiness/yowai-zine)
- 「こころおきなく居られるweb zine」
- 友人と一緒に趣味として作った同人的なWeb雑誌です

## 📝学びの蓄積

**[MainVault](https://github.com/dowdiness/MainVault)**
- Obsidianを使用して学んだことをメモとして後から見直せるように管理しています
- 必要な場合には、[Quartz](https://github.com/jackyzha0/quartz)により記事として公開する仕組みも作っています

## Interests & Learning Focus

### Current Focus
- **Compiler Design:** コンパイラの設計と実装
- **Type Systems:** 型システムの理論と実践
- **Computer Science Fundamentals:** CS基礎の深い理解
- **Operating Systems:** OSの内部構造の学習

### Beyond Code

- 外国語学習を通じた異文化理解
- 知識の体系化と公開（Obsidian + Quartz）
- クリエイティブコーディングとアート表現

## Development Philosophy

技術の表面的な使い方だけでなく、その背後にある原理や仕組みを理解することを重視しています。基礎理論（型システム、コンパイラ、OS）から実践的なアプリケーション開発まで、幅広い領域で学び続けることで、より深い洞察と柔軟な問題解決能力を育んでいます。

また、学んだことを整理し公開することで、知識を社会に還元し、他の学習者との対話を通じてさらに理解を深めることを大切にしています。

---

*Last updated: December 21, 2025*
