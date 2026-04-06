# 機能を拡張する

Claude Code は外部ツールやカスタムワークフローを追加することで大幅に強化できる。

## 拡張機能の全体像

Claude Code を強化する手段は4種類ある:

| 種類 | 何を追加するか | 置き場所 |
|------|--------------|---------|
| **MCPサーバー** | 外部ツール連携（GitHub, DB, ブラウザ等） | `settings.json` の `mcpServers` |
| **スキル** | `/コマンド名` で呼び出せるワークフロー | `~/.claude/skills/` |
| **エージェント** | 特定のタスクを担う専門家の役割定義 | `~/.claude/agents/` |
| **プラグイン** | スキル・エージェント・設定をまとめたパッケージ | `~/.claude/plugins/` |

---

## MCPサーバー — 外部ツールを繋げる

### Claudeに設定させる（推奨）

使いたい目的を伝えるだけで、適切なサーバーを探して設定してくれる:

```
> GitHubのIssueをClaude から操作したい。
> 適切なMCPサーバーを探して設定してもらえますか？
```

### 仕組み

MCP（Model Context Protocol）は、Claude が外部ツールを呼び出すための標準仕様。サーバーを追加すると、Claude が直接 GitHub を操作したり、ブラウザを動かしたりできるようになる。設定後は `settings.json` の `mcpServers` セクションに追記される。

**設定ファイルの場所**

```
~/.claude/settings.json   # グローバル（全プロジェクト共通）
.claude/settings.json     # プロジェクト固有
```

**代表的なサーバー一覧**

| カテゴリ | 代表的なサーバー | できること |
|---------|----------------|---------|
| ブラウザ | `@playwright/mcp` | ページ操作・スクリーンショット・フォーム入力 |
| バージョン管理 | `@modelcontextprotocol/server-github` | Issue/PR 作成・ブランチ操作 |
| DB | `@modelcontextprotocol/server-postgres` | SQL クエリ・スキーマ確認 |
| ファイルシステム | `@modelcontextprotocol/server-filesystem` | ローカルファイルの読み書き |
| ドキュメント | `context7` | ライブラリの最新ドキュメント取得 |
| セマンティック検索 | `serena` | コードのシンボル解析 |

公式・コミュニティ製サーバーは https://github.com/modelcontextprotocol/servers にある。

### 手動で追加する

**方法1: CLIコマンドで追加（推奨）**

```bash
# グローバル設定（全プロジェクトで使える）
claude mcp add playwright -s user -- npx -y @playwright/mcp

# プロジェクト設定（このプロジェクトのみ）
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```

`-s user` でグローバル（`~/.claude/settings.json`）、省略するとプロジェクト設定（`.claude/settings.json`）に書かれる。

**方法2: Claude に追加させる**

```
> playwright MCPサーバーをグローバル設定に追加してください
```

Claude がコマンドを実行して設定する。

**方法3: settings.json を直接編集**

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

設定後は Claude Code を再起動する。

### 設定の確認

```bash
# 現在の設定を確認
claude mcp list

# 特定のサーバーの詳細
claude mcp get playwright
```

---

## スキル — /コマンドを作る

### Claudeにスキルを作らせる

```
> /commit スキルを作ってください。
> git の規約コミット（feat/fix/refactorなど）を自動化したい
```

Claude が `~/.claude/skills/commit.md` を作成する。

### 仕組み

`/スキル名` で呼び出せるカスタムワークフロー。Markdown ファイルに「Claudeへの指示書」を書くだけで作れる。繰り返し使う作業手順を1コマンドにまとめられる。

```
# スキルなし
> git add -p してから、規約に沿ったコミットメッセージを考えて
> feat/fix/refactor のどれか選んで、スコープも入れてください...

# スキルあり（/commit を定義済み）
> /commit
→ 上記の手順が自動で実行される
```

**スキルの置き場所**

```
~/.claude/skills/
  commit.md         # /commit で呼び出せる
  daily-review.md   # /daily-review で呼び出せる
  deploy-check.md   # /deploy-check で呼び出せる
```

### スキルファイルの書き方

```markdown
---
name: commit
description: Gitコミットを規約に沿って作成するスキル
---

以下の手順でコミットを作成してください:

1. `git status` と `git diff --staged` で変更内容を確認する
2. 変更の種類に応じてコミットタイプを選ぶ:
   - `feat:` 新機能
   - `fix:` バグ修正
   - `refactor:` リファクタリング
   - `docs:` ドキュメント
   - `test:` テスト
   - `chore:` その他
3. コミットメッセージは日本語で書く
4. 実行前に内容を確認して承認を求める
```

**フロントマターの説明**

| フィールド | 意味 |
|-----------|------|
| `name` | コマンド名（`/名前` で呼び出す） |
| `description` | スキルの説明（Claude がいつ使うか判断するのに使われる） |

### 活用例

**`/handover`** — セッション引き継ぎドキュメントを作る

```markdown
---
name: handover
description: 作業を引き継ぐためのサマリーを作成する
---

以下の内容をMarkdownで出力してください:
1. 現在の作業状況（何をしていたか）
2. 完了したこと
3. 未完了のこと・次のアクション
4. 重要な決定事項とその理由
5. 注意点・地雷情報
```

**`/security-review`** — セキュリティチェックを実行する

```markdown
---
name: security-review
description: コミット前のセキュリティチェック。認証・入力値・APIキー漏れを確認
---

以下の観点でコードをレビューしてください:
- [ ] ハードコードされたAPIキー・シークレットがない
- [ ] ユーザー入力にバリデーションがある
- [ ] SQLインジェクション対策がある（パラメータ化クエリ）
- [ ] XSS対策がある（HTMLエスケープ）
- [ ] エラーメッセージが機密情報を含まない
問題があれば重要度（CRITICAL/HIGH/MEDIUM）で報告してください。
```

---

## エージェント — 専門家の役割を定義する

### Claudeにエージェントを作らせる

```
> sql-reviewer エージェントを作ってください。
> SQLのN+1クエリとインデックスの問題をレビューする役割です
```

Claude が `.claude/agents/sql-reviewer.md` を作成する。

### 仕組み

Claude Code は自分自身をサブエージェント（子プロセス）として起動し、タスクを委譲できる。サブエージェントは独立したコンテキストウィンドウを持つため、メインの会話を汚染せずに重い作業を分離できる。

ここでいう「汚染」とは、大量のファイル内容・ログ・中間出力がメインの会話のコンテキストに流れ込むことを指す。汚染されると利用可能なコンテキスト量が圧迫され、Claudeが以前の指示や会話の流れを参照しにくくなる。

```
メインClaude（会話の流れを管理）
  ├── サブエージェントA（テスト追加）  ← 独自コンテキスト
  ├── サブエージェントB（Lint修正）    ← 独自コンテキスト
  └── サブエージェントC（ドキュメント）← 独自コンテキスト
```

各エージェントが独立して作業するため、互いの作業結果に引きずられない。

`description` に「いつ使うか」を書いておくと、Claude が文脈を読んで自動的に呼び出す。

### エージェントファイルの書き方

```markdown
---
name: sql-reviewer
description: SQLクエリを書いたとき、またはDB関連のコードを変更したときに使う
---

あなたはデータベース専門家です。
提供されたSQLまたはDBアクセスコードを以下の観点でレビューしてください:

**パフォーマンス**
- N+1クエリの有無
- インデックスが適切に使われているか
- 不要なカラムを SELECT していないか（SELECT * の禁止）

**セキュリティ**
- パラメータ化クエリが使われているか
- ORMを通さない生SQL実行がないか

**保守性**
- クエリが複雑すぎないか
- 説明コメントが必要な箇所はないか

問題点を箇条書きで報告し、改善後のコード例を示してください。
```

**フロントマターのフィールド**

| フィールド | 必須 | 説明 |
|-----------|------|------|
| `name` | ○ | エージェント名（英数字・ハイフン） |
| `description` | ○ | いつ使うか。自動起動の判断に使われる |
| `tools` | △ | 使用を許可するツールのリスト（省略すると全ツール使用可） |

**ツールを制限する例（読み取り専用エージェント）**

```markdown
---
name: code-explainer
description: コードの解説を求められたとき
tools: [Read, Grep, Glob]
---

コードの解説に徹します。ファイルを変更しません。
...
```

### 役割別の設計パターン

**計画フェーズ**

```markdown
---
name: planner
description: 新機能追加・大きなリファクタリング・アーキテクチャ変更を依頼されたとき
---
実装を始める前に設計を整理します。
ファイルを変更する前に必ず計画をMarkdownで出力し、承認を得てから進みます。
```

**実装フェーズ**

```markdown
---
name: tdd-guide
description: バグ修正や新機能の実装を行うとき
---
テストを先に書き（RED）、最小限の実装でテストを通し（GREEN）、
コードを整理します（REFACTOR）。テストなしで実装を進めません。
```

**検証フェーズ**

```markdown
---
name: code-reviewer
description: コードを書いた直後・コミット前
---
読みやすさ・セキュリティ・パフォーマンスの観点でレビューします。
問題点を重要度順に列挙し、修正案を提示します。
```

### チームで使う

複数の専門エージェントを組み合わせてより大きなタスクをこなす設計パターン。

**チームの定義**

```
~/.claude/agents/
  ├── planner.md        # 実装計画を立てる
  ├── tdd-guide.md      # テストを書く
  ├── code-reviewer.md  # コードをレビューする
  └── security-reviewer.md  # セキュリティを検証する
```

**逐次実行（パイプライン型）**

前のエージェントの結果を次のエージェントに渡す:

```
> 以下の順番でエージェントを使って新機能を実装してください:
> 1. planner で設計する
> 2. tdd-guide でテストを書く
> 3. 実装する
> 4. code-reviewer でレビューする
> 5. security-reviewer でセキュリティ確認する
```

**並列実行（ファンアウト型）**

独立したタスクを同時に走らせる:

```
> 以下を並列で実行してください:
> 1. src/auth/以下にユニットテストを追加する
> 2. src/api/以下のJSDocコメントを補完する
> 3. 未使用のimportを全ファイルから削除する
```

**チーム設計のコツ**

| 役割 | description の書き方例 |
|------|----------------------|
| 計画 | 「複雑な機能・アーキテクチャ決定時に使う」 |
| 実装 | 「バグ修正や新機能実装後に使う」 |
| レビュー | 「コードを書いた直後に必ず使う」 |
| セキュリティ | 「ユーザー入力・認証・APIを扱うコードを書いたら使う」 |

### 定義あり vs 定義なし

Claudeは複雑なタスクと判断すると、`agents/` フォルダの定義がなくても自律的にサブエージェントを作成してタスクを委譲する。

> このリポジトリのすべてのファイルのTypeScriptエラーを修正してください
（→ Claudeが自分でサブエージェントを生成し、ファイルごとに並列処理する）

ユーザーが意識しなくても、裏で自動的にサブエージェントが動いている。

| | 定義なし（自律） | 定義あり（`agents/`） |
|---|---|---|
| いつ使われるか | Claudeが必要と判断したとき | descriptionに合うタイミング＋手動 |
| 再利用性 | なし（使い捨て） | あり（毎回同じ役割で呼べる） |
| カスタマイズ | できない | プロンプトで細かく制御できる |
| 向いている場面 | 大規模・複雑なタスク | レビュー・テストなど繰り返し作業 |

---

## プラグイン — まとめてインポートする

スキル・エージェント・設定をまとめたパッケージ。他の人が作ったセットをそのまま取り込める。`~/.claude/plugins/` に置いて有効化する。

**ディレクトリ構造の例**

```
~/.claude/plugins/superpowers/
  skills/
    commit.md
    brainstorming.md
    systematic-debugging.md
    tdd-workflow.md
  agents/
    code-reviewer.md
    security-reviewer.md
  README.md
```

**プラグインの探し方**

| 方法 | 説明 |
|------|------|
| **dotfilesリポジトリ** | チームや個人が公開しているdotfilesをインポートする |
| **npmパッケージ** | MCPサーバーはnpmで配布されているものが多い |
| **GitHub検索** | `claude-code-skills` `claude-agents` などで検索する |

```bash
# 公開されているプラグインをインポートする例
git clone https://github.com/someone/claude-plugins ~/.claude/plugins/someone
```

**自分で作る**

チーム内で使うプラグインをまとめて管理できる:

```
your-team-claude-config/
  plugins/
    our-workflow/
      skills/
        daily-standup.md   # 毎日のスタンドアップまとめ
        pr-review.md       # PR レビューフォーマット
      agents/
        db-reviewer.md     # DB専任レビュアー
  README.md
  setup.sh               # シンボリックリンクを張るスクリプト
```

**setup.sh の例**

```bash
#!/bin/bash
DOTFILES_DIR="$(cd "$(dirname "$0")" && pwd)"

# スキルをリンク
mkdir -p ~/.claude/skills
for f in "$DOTFILES_DIR/plugins/our-workflow/skills/"*.md; do
  ln -sf "$f" ~/.claude/skills/
done

# エージェントをリンク
mkdir -p ~/.claude/agents
for f in "$DOTFILES_DIR/plugins/our-workflow/agents/"*.md; do
  ln -sf "$f" ~/.claude/agents/
done

echo "セットアップ完了"
```

### よく使われるプラグイン

**superpowers**

スキルシステムを中心に、デバッグ・ブレインストーミング・TDDワークフローを提供するプラグイン。

| スキル | 用途 |
|--------|------|
| `/superpowers:brainstorming` | 実装前に要件を整理。Claudeが質問して曖昧さを解消する |
| `/superpowers:systematic-debugging` | 「とりあえず試す」を禁止し、4フェーズで根本原因を追う |
| `/superpowers:tdd-workflow` | RED→GREEN→REFACTOR サイクルをガイドする |

**serena**

ファイル全体を読む代わりに、関数・クラス単位でコードを理解する MCP サーバー。大きなコードベースでトークン消費を大幅に削減できる。

```
# serena なし: user-service.ts (400行) を全部読む
# serena あり: UserService.authenticate() の本体だけ読む（1/10 のトークン）
```

**context7**

ライブラリの最新ドキュメントを回答前に自動フェッチする MCP サーバー。Claude の学習データが古くても、常に最新 API で答えてくれる。

```
# context7 なし: 古いバージョンの書き方を提案（動かない）
# context7 あり: 最新の公式ドキュメントを取得して正確に回答
```

---

## まとめ：拡張の追加手順

```
1. 目的を決める（「GitHubを操作したい」「コミット手順を自動化したい」）
         ↓
2. 適切な手段を選ぶ
   - 外部ツール連携 → MCPサーバー
   - 繰り返し手順 → スキル（/コマンド）
   - 特定の専門役割 → エージェント
   - まとめてインポート → プラグイン
         ↓
3. 追加する
   - MCP: claude mcp add コマンド
   - スキル/エージェント: ~/.claude/ にMarkdownファイルを置く
   - プラグイン: setup.sh を実行
         ↓
4. Claude に「何が使えるか」を確認する
   > 使えるMCPサーバーとスキルを教えてください
```

---

## 次のステップ

- [04-customize.md](./04-customize.md) — 設定ファイルやフックを整える
