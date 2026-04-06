# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

社内開発者向けのClaude Code使い方ハンズオン資料。完全初心者（インストール未経験）から応用機能を使いこなしたいメンバーまでをカバーするMarkdownドキュメント集。

## ドキュメント構成

ゴール指向の5ファイル構成。読者が「やりたいこと」でファイルを選べる。

```
docs/
  README.md                # 入口ガイド・読む順番の案内
  01-getting-started.md    # 「使い始めたい」— インストール〜初回起動
  02-daily-use.md          # 「毎日の作業に使いたい」— 基本操作・プロンプト集
  03-better-prompts.md     # 「より正確に動かしたい」— プロンプト術・トラブルシューティング
  04-customize.md          # 「自分仕様に設定したい」— CLAUDE.md・rules・設定・フック・dotfiles
  05-extend.md             # 「できることを増やしたい」— MCP・スキル・エージェント・プラグイン
  superpowers/
    specs/                 # 設計ドキュメント
    plans/                 # 実装計画
```

## 設計方針

- 各ファイルは独立して参照できる（前のファイルを読んでいなくてもわかる）
- コマンドはコピペで動く形で記載する
- 設定系セクションは「Claudeへの指示例」→「仕組みの解説」の順で記述する
- 対象読者: 日本語話者の開発者（Claude Code未使用〜中級者）
- 形式: Markdown（`.md`）

## ファイル編集時の注意

- 既存ドキュメントのトーンに合わせる（技術的かつ親しみやすい日本語）
- コードブロックはコピペで動く形にする
- 各ドキュメントの末尾に「次のステップ」リンクを維持する
- セクション区切りは `---` で統一する
- `docs/superpowers/` 配下のファイルは計画・仕様ドキュメント（`.gitignore`対象だが参照・編集可能）
