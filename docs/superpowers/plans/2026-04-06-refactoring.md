# ドキュメント全体リファクタリング 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 7ファイルの難易度順ドキュメントをゴール指向の5ファイル構成に再編し、「Claudeに書かせる」方針で設定説明を統一する

**Architecture:** 既存ファイルの内容を新しいゴール別ファイルに移動・統合する。旧ファイルは全タスク完了後に削除。README は最後に更新する。

**Tech Stack:** Markdown、Git

---

## ファイル変換マップ

| 新ファイル | 統合元 |
|-----------|--------|
| `docs/01-getting-started.md` | `docs/00-getting-started.md` + `docs/02-intermediate.md`（トラブルシューティング末尾） |
| `docs/02-daily-use.md` | `docs/01-basics.md` + `docs/04-use-cases.md` |
| `docs/03-better-prompts.md` | `docs/02-intermediate.md`（プロンプト術・/branch セクション）+ `docs/02-intermediate.md`（使用中トラブルシューティング） |
| `docs/04-customize.md` | `docs/02-intermediate.md`（CLAUDE.md・rules/・settings セクション）+ `docs/05-dotfiles.md` + `docs/03-advanced.md`（フックセクション） |
| `docs/05-extend.md` | `docs/03-advanced.md`（MCP・エージェントセクション）+ `docs/06-extensions.md`（重複除去） |
| `docs/README.md` | 新しいファイル構成を反映 |

**削除対象（Task 7 で実施）:**
- `docs/00-getting-started.md`
- `docs/01-basics.md`
- `docs/02-intermediate.md`
- `docs/03-advanced.md`
- `docs/04-use-cases.md`
- `docs/05-dotfiles.md`
- `docs/06-extensions.md`

---

## 品質チェック基準（各タスクで確認）

各ファイル完成後に以下を確認する:

1. **独立参照可能か** — 前後のファイルを読んでいなくても理解できるか
2. **コピペ動作** — コマンド・プロンプト例がそのまま使えるか
3. **「Claudeに書かせる」が先か** — 設定系セクションはClaude指示例→仕組み解説の順か
4. **---区切りの統一** — セクション間の `---` が適切に使われているか

---

### Task 1: 01-getting-started.md を作成する

**Files:**
- Create: `docs/01-getting-started.md`
- Read: `docs/00-getting-started.md`
- Read: `docs/02-intermediate.md`（末尾のトラブルシューティングセクション）

- [ ] **Step 1: `docs/00-getting-started.md` を読む**

  現在の内容を把握する。

- [ ] **Step 2: `docs/02-intermediate.md` の末尾（トラブルシューティング）を読む**

  「的外れな回答が続く」「会話が長くなって遅い」などのセクションのうち、**起動時のトラブル**（"command not found"・"Invalid API Key" など）を特定する。

  移動対象:
  - **「command not found: claude」** → インストールコマンドを再実行する
  - **「Invalid API Key」** → `echo $ANTHROPIC_API_KEY` でキーが設定されているか確認
  
  移動しないもの（`03-better-prompts.md` へ）:
  - 「的外れな回答が続く」→ /clear
  - 「会話が長くなって遅い」→ /compact
  - 「変更を取り消したい」
  - 「意図しないファイルを変更された」

- [ ] **Step 3: `docs/01-getting-started.md` を作成する**

  以下の構成で書く:

  ```markdown
  # はじめに — インストールと初回起動

  ## インストール
  （00-getting-started.md のインストールセクションをそのままコピー）

  ## PATHの設定
  （00-getting-started.md のPATH設定セクションをそのままコピー）

  ## 起動してみる
  （00-getting-started.md の「起動してみる」セクションをそのままコピー）

  ## 初回起動時の認証フロー（Proプラン）
  （00-getting-started.md の認証フローセクションをそのままコピー）

  ## うまくいかない場合

  **「command not found: claude」** → インストールコマンドを再実行する。

  **「Invalid API Key」** → `echo $ANTHROPIC_API_KEY` でキーが設定されているか確認。

  ---

  ## 次のステップ

  - [02-daily-use.md](./02-daily-use.md) — 毎日の作業パターン・使えるプロンプト集
  ```

- [ ] **Step 4: 品質チェック**

  確認項目:
  - [ ] インストールコマンドがコピペで動く形か
  - [ ] 認証フローの手順が番号付きで明確か
  - [ ] トラブルシューティングが起動時のものだけか（使用中のものは含まれていないか）

- [ ] **Step 5: コミット**

  ```bash
  git add docs/01-getting-started.md
  git commit -m "docs: 01-getting-started.md を作成（インストール・起動・初回トラブルシューティング）"
  ```

---

### Task 2: 02-daily-use.md を作成する

**Files:**
- Create: `docs/02-daily-use.md`
- Read: `docs/01-basics.md`
- Read: `docs/04-use-cases.md`

- [ ] **Step 1: `docs/01-basics.md` を読む**

  基本操作（コマンド一覧・ショートカット・rewind/resume/branch の使い分け）を把握する。

- [ ] **Step 2: `docs/04-use-cases.md` を読む**

  プロンプト集のカテゴリと各例を把握する。

- [ ] **Step 3: `docs/02-daily-use.md` を作成する**

  以下の構成で書く:

  ```markdown
  # 毎日の使い方

  ## 基本操作
  （01-basics.md の「起動と終了」「よく使うコマンド」「よく使うショートカットキー」をそのままコピー）

  ## 話しかけ方のコツ

  `>` プロンプトに日本語で入力するだけ。具体的に書くほど精度が上がる。
  
  詳しいプロンプトの書き方は [03-better-prompts.md](./03-better-prompts.md) を参照。

  ---

  ## よくある作業パターン

  コピペして使えるプロンプト例。

  ### コーディング
  （04-use-cases.md の「コーディング」セクションをそのままコピー）

  ### ドキュメント作成
  （04-use-cases.md の「ドキュメント作成」セクションをそのままコピー）

  ### ルールを書かせる
  （04-use-cases.md の「ルールを書かせる」セクションをそのままコピー）

  ### その他
  （04-use-cases.md の「その他」セクションをそのままコピー）

  ---

  ## セッション管理

  （01-basics.md の「rewind と resume」セクションをそのままコピー）
  （01-basics.md の「会話リセットのタイミング」「rewindとresumeの使い分け表」をそのままコピー）

  ---

  ## 次のステップ

  - [03-better-prompts.md](./03-better-prompts.md) — より正確に動かしたいとき
  - [04-customize.md](./04-customize.md) — プロジェクトのルールを設定したいとき
  ```

- [ ] **Step 4: 品質チェック**

  確認項目:
  - [ ] 基本操作のコマンド表がそのまま使えるか
  - [ ] プロンプト例がすべてコピペ可能な形（`> ` プレフィックス付き）か
  - [ ] 深いプロンプト論が含まれていないか（含まれていたら03へのリンクに置き換える）
  - [ ] セッション管理の使い分け表が含まれているか

- [ ] **Step 5: コミット**

  ```bash
  git add docs/02-daily-use.md
  git commit -m "docs: 02-daily-use.md を作成（基本操作・プロンプト集・セッション管理）"
  ```

---

### Task 3: 03-better-prompts.md を作成する

**Files:**
- Create: `docs/03-better-prompts.md`
- Read: `docs/02-intermediate.md`

- [ ] **Step 1: `docs/02-intermediate.md` を読む**

  移動対象のセクションを特定する:
  - 「プロンプトの書き方」セクション全体（具体的に書く・コンテキスト・制約・分割）
  - 「/branch — 会話を分岐して複数のアプローチを探索する」セクション
  - 「トラブルシューティング」セクション（ただし起動時トラブルを除く）
  
  移動しないもの（Task 4 で使う）:
  - CLAUDE.md セクション
  - ~/.claude/rules/ セクション
  - よく使う設定セクション

- [ ] **Step 2: `docs/03-better-prompts.md` を作成する**

  以下の構成で書く:

  ```markdown
  # より正確に動かすには

  ## 良いプロンプトの書き方
  （02-intermediate.md の「プロンプトの書き方」セクション全体をコピー）
  （「具体的に書く」「コンテキストを与える」「制約を明示する」「大きなタスクは分割する」）

  ---

  ## 複数アプローチを探索する（/branch）
  （02-intermediate.md の「/branch」セクション全体をコピー）

  ---

  ## うまくいかないときの対処

  **的外れな回答が続く** → `/clear` でリセットして指示し直す

  **会話が長くなって遅い** → `/compact` で要約するか `/clear` でリセット

  **変更を取り消したい** → `git checkout -- <file>` か「さっきの変更を元に戻してください」と頼む

  **意図しないファイルを変更された** → `autoApprove: false`（デフォルト）にしておけば変更前に確認が入る

  ---

  ## 次のステップ

  - [04-customize.md](./04-customize.md) — プロジェクトのルールや設定を整えたいとき
  - [05-extend.md](./05-extend.md) — できることを増やしたいとき（MCP・エージェント・スキル）
  ```

- [ ] **Step 3: 品質チェック**

  確認項目:
  - [ ] 悪い例/良い例のコードブロックが含まれているか
  - [ ] `/branch` の使い分け表（/rewind との違い）が含まれているか
  - [ ] トラブルシューティングが「使用中」のものだけか（起動時は含まれていないか）

- [ ] **Step 4: コミット**

  ```bash
  git add docs/03-better-prompts.md
  git commit -m "docs: 03-better-prompts.md を作成（プロンプト術・/branch・トラブルシューティング）"
  ```

---

### Task 4: 04-customize.md を作成する

**Files:**
- Create: `docs/04-customize.md`
- Read: `docs/02-intermediate.md`（CLAUDE.md・rules/・settings セクション）
- Read: `docs/05-dotfiles.md`
- Read: `docs/03-advanced.md`（フックセクション）

- [ ] **Step 1: ソースファイルの対象セクションを確認する**

  `02-intermediate.md` から移動するセクション:
  - 「CLAUDE.md — プロジェクトルールの設定」
  - 「~/.claude/rules/ — 全プロジェクト共通ルール」
  - 「よく使う設定」（設定ファイル・/config・Output style）

  `03-advanced.md` から移動するセクション:
  - 「フック（Hooks）」セクション全体

  `05-dotfiles.md` は全体を統合する。

- [ ] **Step 2: `docs/04-customize.md` を作成する**

  **重要:** 各セクションは「Claudeへの指示例」を先に示し、その後「仕組みの解説」を添える形にする。

  以下の構成で書く:

  ```markdown
  # 自分仕様に設定する

  Claude Codeは設定ファイルを自分で読み書きできる。手でJSONを書く前に、まずClaudeに頼んでみる。

  ---

  ## CLAUDE.md — プロジェクトのルールを設定する

  ### Claudeに生成させる（推奨）

  プロジェクトのルートで `/init` を実行するとClaudeがコードベースを解析してCLAUDE.mdを生成する。

  ```
  > /init
  ```

  生成後は不要な項目を削除し、チーム固有のルールを追記するだけでよい。

  ### CLAUDE.md でできること

  （02-intermediate.md の「CLAUDE.md — プロジェクトルールの設定」の内容を反映）
  （起動時に自動読み込みされる・サブディレクトリにも置ける・効果的な書き方のガイド）

  ---

  ## ~/.claude/rules/ — 全プロジェクト共通ルール

  ### Claudeに書かせる

  自分の好みを言葉で伝えると、ルールファイルとして整理してくれる:

  ```
  > 私のコーディングスタイルを ~/.claude/rules/coding-style.md にまとめてください。
  > イミュータブルなコード・関数は50行以内・TypeScriptのanyは使わない
  ```

  ### 仕組み

  （02-intermediate.md の「~/.claude/rules/」の内容を反映）
  （スコープの違い・カテゴリ別分割パターン・CLAUDE.mdとの使い分け表）

  ---

  ## 設定ファイル（settings.json）

  ### /config でインタラクティブに設定する（推奨）

  ```
  /config
  ```

  メニューから選ぶだけで `~/.claude/settings.json` に保存される。

  ### Claudeに設定させる

  ```
  > ファイルを保存するたびに lint を自動実行するフックを設定してください
  ```

  Claudeが `settings.json` を読んで適切に追記する。追記後に「何が書かれたか」を説明してもらうとよい。

  ### 設定の3スコープ

  （02-intermediate.md の設定ファイルの表をコピー）
  （グローバル・プロジェクト・プロジェクトローカルの優先順位）

  ### 主な設定項目

  （02-intermediate.md の /config で変更できる設定一覧の表をコピー）
  （Output style の説明を含む）

  ---

  ## フック — ツール実行の前後に自動処理

  ### Claudeにフックを設定させる

  ```
  > ファイルを編集するたびに自動でlintを実行するフックを追加してください
  > コマンド実行前にログを記録するフックも設定してください
  ```

  ### 仕組み

  （03-advanced.md の「フック（Hooks）」セクション全体をコピー）
  （PreToolUse・PostToolUse・matcher・設定例）

  ---

  ## dotfiles — チームで設定を共有する

  ### Claudeにsetup.shを書かせる

  ```
  > チームのClaude Code設定をdotfilesで共有したい。
  > rules/とagents/をシンボリックリンクで管理するsetup.shを作ってください
  ```

  ### 仕組み

  （05-dotfiles.md の内容を統合）
  （シンボリックリンクで~/.claude/以下に配置される・インポートできる内容の一覧）

  ---

  ## 次のステップ

  - [05-extend.md](./05-extend.md) — MCPサーバー・スキル・エージェントで機能を拡張する
  ```

- [ ] **Step 3: 品質チェック**

  確認項目:
  - [ ] 各セクションが「Claudeへの指示例」→「仕組みの解説」の順になっているか
  - [ ] `settings.json` の3スコープ表が含まれているか
  - [ ] フックの設定例（PostToolUse のJSON）が含まれているか
  - [ ] dotfiles の `setup.sh` の解説が含まれているか

- [ ] **Step 4: コミット**

  ```bash
  git add docs/04-customize.md
  git commit -m "docs: 04-customize.md を作成（CLAUDE.md・rules・設定・フック・dotfiles）"
  ```

---

### Task 5: 05-extend.md を作成する

**Files:**
- Create: `docs/05-extend.md`
- Read: `docs/03-advanced.md`
- Read: `docs/06-extensions.md`

- [ ] **Step 1: 重複箇所を特定する**

  以下が両ファイルに重複している:

  | トピック | 03-advanced.md | 06-extensions.md |
  |---------|---------------|-----------------|
  | MCPの概念説明 | 「MCPとは何か」の説明あり | 「MCPサーバーとは」の説明あり |
  | MCPの設定方法 | 「Claudeに設定させる」3方法 | 「追加方法」3方法 |
  | サブエージェントの仕組み | 「エージェント機能」セクション全体 | 「カスタムエージェントとは」 |

  統合方針:
  - MCPセクションは `06-extensions.md` の内容を主軸にし、`03-advanced.md` の「探し方」「設定確認コマンド」を補完
  - エージェントセクションは `06-extensions.md` の「どう作るか」に `03-advanced.md` の「サブエージェントの分離効果」「定義あり vs 定義なし表」を統合
  - 重複する説明文は1つに絞る

- [ ] **Step 2: `docs/05-extend.md` を作成する**

  以下の構成で書く:

  ```markdown
  # 機能を拡張する

  Claude Codeは外部ツールやカスタムワークフローを追加することで大幅に強化できる。

  ## 拡張機能の全体像

  （06-extensions.md の全体像の表をコピー）
  （MCPサーバー / スキル / エージェント / プラグインの4種類）

  ---

  ## MCPサーバー — 外部ツールを繋げる

  ### Claudeに設定させる（推奨）

  使いたい目的を伝えるだけで、適切なサーバーを探して設定してくれる:

  ```
  > GitHubのIssueをClaude から操作したい。
  > 適切なMCPサーバーを探して設定してもらえますか？
  ```

  ### 仕組み

  MCP（Model Context Protocol）は、Claudeが外部ツールを呼び出すための標準仕様。
  設定後は `settings.json` の `mcpServers` セクションに追記される。

  （06-extensions.md の代表的なサーバー一覧の表をコピー）

  ### 手動で追加する

  （06-extensions.md の「追加方法」3種をコピー — CLIコマンド・Claude経由・直接編集）

  ### 設定の確認

  ```bash
  claude mcp list
  claude mcp get playwright
  ```

  ---

  ## スキル — /コマンドを作る

  ### Claudeにスキルを作らせる

  ```
  > /commit スキルを作ってください。
  > git の規約コミット（feat/fix/refactorなど）を自動化したい
  ```

  Claudeが `~/.claude/skills/commit.md` を作成する。

  ### 仕組み

  （06-extensions.md の「スキルとは」「スキルファイルの書き方」をコピー）

  ### 活用例

  （06-extensions.md の「/handover」「/security-review」の例をコピー）

  ---

  ## エージェント — 専門家の役割を定義する

  ### Claudeにエージェントを作らせる

  ```
  > sql-reviewer エージェントを作ってください。
  > SQLのN+1クエリとインデックスの問題をレビューする役割です
  ```

  Claudeが `.claude/agents/sql-reviewer.md` を作成する。

  ### 仕組み

  `description` に「いつ使うか」を書いておくと、Claudeが文脈を読んで自動的に呼び出す。

  （03-advanced.md の「サブエージェントの分離効果」の図をコピー）

  ### エージェントファイルの書き方

  （06-extensions.md の「エージェントファイルの書き方」をコピー）
  （フロントマターのフィールド表・ツール制限の例）

  ### 役割別の設計パターン

  （06-extensions.md の「役割別エージェントの設計パターン」をコピー）
  （計画・実装・検証フェーズの例）

  ### チームで使う

  （03-advanced.md の「エージェントチーム」セクションをコピー）
  （逐次実行・並列実行のパターン）

  ### 定義あり vs 定義なし

  （03-advanced.md の「定義あり vs 定義なし」比較表をコピー）

  ---

  ## プラグイン — まとめてインポートする

  （06-extensions.md の「プラグイン」セクション全体をコピー）
  （ディレクトリ構造・探し方・自分で作る方法）

  ### よく使われるプラグイン

  （06-extensions.md の「よく使われるプラグインの紹介」をコピー）
  （superpowers・serena・context7）

  ---

  ## まとめ：拡張の追加手順

  （06-extensions.md の「まとめ」フローをコピー）
  （目的決める → 手段選ぶ → 追加する → 確認する）

  ---

  ## 次のステップ

  - [04-customize.md](./04-customize.md) — 設定ファイルやフックを整える
  ```

- [ ] **Step 3: 品質チェック**

  確認項目:
  - [ ] MCPの概念説明が1か所にまとまっているか（重複していないか）
  - [ ] エージェントの「サブエージェント分離の仕組み」が含まれているか
  - [ ] 「定義あり vs 定義なし」比較表が含まれているか
  - [ ] 「Claudeに作らせる指示例」が各セクションの冒頭にあるか
  - [ ] プラグイン（superpowers・serena・context7）の紹介が含まれているか

- [ ] **Step 4: コミット**

  ```bash
  git add docs/05-extend.md
  git commit -m "docs: 05-extend.md を作成（MCP・スキル・エージェント・プラグイン、重複除去）"
  ```

---

### Task 6: README.md を更新する

**Files:**
- Modify: `docs/README.md`

- [ ] **Step 1: `docs/README.md` を読む**

  現在の内容と新しいファイル構成の差分を確認する。

- [ ] **Step 2: `docs/README.md` を更新する**

  以下の内容に書き直す:

  ```markdown
  # Claude Code 使い方ガイド

  ターミナルで動くAIコーディングアシスタント。コード生成・バグ修正・ドキュメント作成など、開発まわりのことをなんでも頼める。

  ## どこから読むか

  **やりたいことから選ぶ:**

  | やりたいこと | ドキュメント |
  |------------|------------|
  | まずインストールして使い始めたい | [01-getting-started.md](./01-getting-started.md) |
  | 毎日の作業に使えるプロンプトが知りたい | [02-daily-use.md](./02-daily-use.md) |
  | もっと正確に動かしたい（プロンプト術） | [03-better-prompts.md](./03-better-prompts.md) |
  | CLAUDE.md・設定ファイルを整えたい | [04-customize.md](./04-customize.md) |
  | MCP・エージェント・スキルを追加したい | [05-extend.md](./05-extend.md) |

  **順番に読む場合:**

  01 → 02 → 03 → 04 → 05

  ## こんなことができる

  - コードを書く・直す
  - バグの原因を調べる
  - README・仕様書を書く
  - リファクタリング・テストを任せる
  - ファイルをまたいだ大きな変更を依頼する
  ```

- [ ] **Step 3: 品質チェック**

  確認項目:
  - [ ] 新しい5ファイルがすべてリンクされているか
  - [ ] 旧ファイル（00〜06）へのリンクが残っていないか
  - [ ] 「やりたいことから選ぶ」表が読者の目的に対応しているか

- [ ] **Step 4: コミット**

  ```bash
  git add docs/README.md
  git commit -m "docs: README.md を新しいファイル構成に合わせて更新"
  ```

---

### Task 7: 旧ファイルを削除する

**Files:**
- Delete: `docs/00-getting-started.md`
- Delete: `docs/01-basics.md`
- Delete: `docs/02-intermediate.md`
- Delete: `docs/03-advanced.md`
- Delete: `docs/04-use-cases.md`
- Delete: `docs/05-dotfiles.md`
- Delete: `docs/06-extensions.md`

- [ ] **Step 1: 新ファイルに内容が移っているか最終確認する**

  各新ファイルを確認し、旧ファイルの重要な内容が漏れていないことを確認する:

  | 確認ポイント | 移動先 |
  |------------|--------|
  | インストールコマンド（Linux/Windows） | `01-getting-started.md` |
  | 基本コマンド表・ショートカット表 | `02-daily-use.md` |
  | rewind/resume/branch の使い分け表 | `02-daily-use.md` |
  | プロンプトの4原則（悪い例/良い例） | `03-better-prompts.md` |
  | CLAUDE.md の効果的な書き方 | `04-customize.md` |
  | Output style の説明 | `04-customize.md` |
  | フックの設定例（JSON） | `04-customize.md` |
  | MCP代表的サーバー一覧 | `05-extend.md` |
  | エージェント「定義あり vs 定義なし」表 | `05-extend.md` |
  | superpowers・serena・context7 の紹介 | `05-extend.md` |

- [ ] **Step 2: 旧ファイルを削除する**

  ```bash
  git rm docs/00-getting-started.md \
         docs/01-basics.md \
         docs/02-intermediate.md \
         docs/03-advanced.md \
         docs/04-use-cases.md \
         docs/05-dotfiles.md \
         docs/06-extensions.md
  ```

- [ ] **Step 3: コミット**

  ```bash
  git commit -m "docs: 旧ファイルを削除（00〜06）、新構成（01〜05）へ完全移行"
  ```

---

### Task 8: 最終確認

- [ ] **Step 1: ファイル構成を確認する**

  ```bash
  ls docs/
  ```

  期待される出力:
  ```
  README.md
  01-getting-started.md
  02-daily-use.md
  03-better-prompts.md
  04-customize.md
  05-extend.md
  superpowers/
  ```

- [ ] **Step 2: 各ファイルのリンクが壊れていないか確認する**

  各ファイルの「次のステップ」リンクが正しい宛先を指しているか確認する:
  - `01-getting-started.md` → `02-daily-use.md`
  - `02-daily-use.md` → `03-better-prompts.md`・`04-customize.md`
  - `03-better-prompts.md` → `04-customize.md`・`05-extend.md`
  - `04-customize.md` → `05-extend.md`
  - `05-extend.md` → `04-customize.md`

- [ ] **Step 3: 全文の「---」統一を確認する**

  各ファイルでセクション区切りの `---` が適切に使われているか目視確認する。

- [ ] **Step 4: 完了コミット（必要に応じて修正があれば）**

  ```bash
  git add -A
  git commit -m "docs: リファクタリング最終調整"
  ```
