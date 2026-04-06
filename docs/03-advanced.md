# 応用機能

## MCP（Model Context Protocol）サーバー

MCP（Model Context Protocol）は、ClaudeにAI側から外部ツールやサービスを呼び出す能力を追加するための標準仕様。MCPサーバーを組み込むことで、ブラウザ操作・GitHub操作・DB接続といった機能がそのまま使えるようになる。

**設定ファイルの場所**

```
~/.claude/settings.json   # グローバル（全プロジェクト共通）
.claude/settings.json     # プロジェクト固有
```

**GitHub MCPサーバーの設定例**

```json
{
  "mcpServers": {
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

**Playwright（ブラウザ操作）の設定例**

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    }
  }
}
```

設定後にClaude Codeを再起動すると使えるようになる。

```
> GitHubのissue #123の内容を確認して、対応するコードを修正してください
> ブラウザでlocalhost:3000を開いて、ログイン画面のスクリーンショットを撮ってください
```

公式・コミュニティ製サーバーは https://github.com/modelcontextprotocol/servers にある。

### Claude自身にMCPを設定させる

JSONを手書きせず、ClaudeにMCP設定を追加させることができる。

**方法1: CLIコマンドを実行させる**

```
> claude mcp add コマンドを使って、Playwright MCPをグローバルに設定してください
```

ClaudeがターミナルでCLIコマンドを実行して設定する:

```bash
# Claude が実行するコマンドのイメージ
claude mcp add playwright -s user -- npx -y @playwright/mcp
```

`-s user` でグローバル設定（`~/.claude/settings.json`）、
省略するとプロジェクト設定（`.claude/settings.json`）に書かれる。

**方法2: 設定ファイルを直接編集させる**

```
> .claude/settings.json にGitHub MCPサーバーを追加してください。
> トークンは環境変数 GITHUB_TOKEN から読み込むようにしてください。
```

ClaudeがJSONを読んで、`mcpServers` セクションに追記する。

**方法3: まるごと任せる**

使いたいMCPサーバー名だけ伝えれば、調べて設定まで完結させられる。

```
> context7 というMCPサーバーを設定してください。
> 何が必要か調べて、settings.json に追加してください。
```

設定後に再起動が必要な場合はClaudeが案内してくれる。

## エージェント機能

### サブエージェント

Claude Codeは自分自身をサブエージェント（子プロセス）として起動し、タスクを委譲できる。
サブエージェントは独立したコンテキストウィンドウを持つため、メインの会話を汚染せずに重い作業を分離できる。
ここでいう「汚染」とは、大量のファイル内容・ログ・中間出力がメインの会話のコンテキストに流れ込むことを指す。汚染されると利用可能なコンテキスト量が圧迫され、Claudeが以前の指示や会話の流れを参照しにくくなる（最終的には古い内容が切り捨てられる）。

**サブエージェントの分離効果**

```
メインClaude（会話の流れを管理）
  ├── サブエージェントA（テスト追加）  ← 独自コンテキスト
  ├── サブエージェントB（Lint修正）    ← 独自コンテキスト
  └── サブエージェントC（ドキュメント）← 独自コンテキスト
```

各エージェントが独立して作業するため、互いの作業結果に引きずられない。

**`agents/` フォルダで専門化・再利用する**

繰り返し使いたい役割（レビュー、セキュリティ確認など）は定義ファイルを置いておくと、
Claudeが適切なタイミングで自動的に呼び出してくれる。

定義場所:

```
~/.claude/agents/   # グローバル（全プロジェクトで使える）
.claude/agents/     # プロジェクト固有
```

定義ファイルの書き方（例: `.claude/agents/code-reviewer.md`）:

```markdown
---
name: code-reviewer
description: コードレビューを行う。コードを書いた直後に使う。
---

あなたはコードレビュアーです。
以下の観点でレビューしてください:
- 可読性・命名
- セキュリティ上の問題
- パフォーマンス上の懸念
問題点を箇条書きで報告し、改善案を提示してください。
```

`description` に「いつ使うか」を書いておくのがポイント。Claudeが文脈を読んで自動呼び出しする。
手動で呼び出すこともできる:

```
> code-reviewer エージェントを使って src/auth.ts をレビューしてください
```

**エージェント定義は必須ではない**

Claudeは複雑なタスクと判断すると、`agents/` フォルダの定義がなくても自律的にサブエージェントを作成してタスクを委譲する。

```
> このリポジトリのすべてのファイルのTypeScriptエラーを修正してください
（→ Claudeが自分でサブエージェントを生成し、ファイルごとに並列処理する）
```

ユーザーが意識しなくても、裏で自動的にサブエージェントが動いている。

**まとめ: 定義あり vs 定義なし**

| | 定義なし（自律） | 定義あり（`agents/`） |
|---|---|---|
| いつ使われるか | Claudeが必要と判断したとき | descriptionに合うタイミング＋手動 |
| 再利用性 | なし（使い捨て） | あり（毎回同じ役割で呼べる） |
| カスタマイズ | できない | プロンプトで細かく制御できる |
| 向いている場面 | 大規模・複雑なタスク | レビュー・テストなど繰り返し作業 |

---

### エージェントチーム（Agent Teams）

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

`description` に「いつ使うか」を書いておくと、Claudeが文脈を読んで自動的に使い分けてくれる。

**長いタスク**

```
> このリポジトリ全体のTypeScriptエラーをすべて修正してください。
> ファイルごとに順番に修正してください。
```

## フック（Hooks）

Claude Codeがツールを実行する前後に、自動でコマンドを走らせる仕組み。

`.claude/settings.json` の `hooks` セクションに書く:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "npm run lint -- --fix" }
        ]
      }
    ]
  }
}
```

**コマンド実行前にログを記録する例**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "echo '[Hook] Bash実行前' >> .claude/log.txt" }]
      }
    ]
  }
}
```
