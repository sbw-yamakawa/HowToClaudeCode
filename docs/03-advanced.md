# 03. 応用機能

このファイルで学べること:
- MCPサーバーの設定と使い方
- エージェント機能（並列タスク・サブエージェント）
- フック（Hooks）の設定

---

## MCP（Model Context Protocol）サーバー

MCPサーバーを使うと、Claude Codeの能力を拡張できます。
たとえば、ブラウザ操作・GitHub操作・データベース接続などが可能になります。

### 設定ファイルの場所

```
~/.claude/settings.json   # グローバル設定（全プロジェクト共通）
.claude/settings.json     # プロジェクト固有の設定
```

### 設定例（GitHub MCPサーバー）

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

### 設定例（Playwright — ブラウザ操作）

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

### 使い方

設定後にClaude Codeを再起動すると、MCPサーバーが利用可能になります。

```
> GitHubのissue #123の内容を確認して、対応するコードを修正してください

> ブラウザでlocalhost:3000を開いて、ログイン画面のスクリーンショットを撮ってください
```

### 利用可能なMCPサーバー一覧

https://github.com/modelcontextprotocol/servers に公式・コミュニティ製のサーバーがあります。

---

## エージェント機能

### サブエージェント（並列処理）

複数の独立したタスクを並列で処理させることができます。

```
> 以下を並列で実行してください:
> 1. src/auth/以下のファイルにユニットテストを追加する
> 2. src/api/以下のJSDocコメントを補完する
> 3. 未使用のimportを全ファイルから削除する
```

Claude Codeがサブエージェントを立ち上げて各タスクを並列実行します。

### 長いタスクの実行

```
> このリポジトリ全体のTypeScriptエラーをすべて修正してください。
> 一度にすべてではなく、ファイルごとに順番に修正してください。
```

---

## フック（Hooks）

フックを使うと、Claude Codeがツールを実行する前後に自動でコマンドを実行できます。

### 設定場所

`.claude/settings.json` の `hooks` セクション:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint -- --fix"
          }
        ]
      }
    ]
  }
}
```

### よく使うフックの例

**ファイル保存後に自動でLint実行:**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "npm run lint -- --fix" }]
      }
    ]
  }
}
```

**コマンド実行前に確認ログを記録:**

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

---

次のステップ: [04-use-cases.md](./04-use-cases.md) — 実際の使い方例
