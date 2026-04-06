# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

社内開発者向けのClaude Code使い方ハンズオン資料。完全初心者（インストール未経験）から応用機能を使いこなしたいメンバーまでをカバーするMarkdownドキュメント集。

## ドキュメント構成

```
docs/
  README.md                # 入口ガイド・読む順番の案内
  00-getting-started.md    # インストール〜初回起動
  01-basics.md             # 基本操作・よく使うコマンド
  02-intermediate.md       # 効果的なプロンプト・CLAUDE.md・設定
  03-advanced.md           # MCP・エージェント・フック
  04-use-cases.md          # 開発者向けユースケース集
  05-dotfiles.md           # dotfilesでカスタマイズ
  superpowers/
    specs/                 # 設計ドキュメント
    plans/                 # 実装計画
```

## 設計方針

- 各ファイルは独立して参照できる（前のファイルを読んでいなくてもわかる）
- コマンドはコピペで動く形で記載する
- プロンプト例は実際の業務に近い具体例を使う
- 対象読者: 日本語話者の開発者（Claude Code未使用・初心者）
- 形式: Markdown（`.md`）

## ファイル編集時の注意

- 既存ドキュメントのトーンに合わせる（技術的かつ親しみやすい日本語）
- コードブロックはコピペで動く形にする
- 各ドキュメントの末尾に「次のステップ」リンクを維持する
- `docs/superpowers/` 配下のファイルは計画・仕様ドキュメント（`.gitignore`対象だが参照・編集可能）
