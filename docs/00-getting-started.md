# 00. はじめに — インストールと初回起動

このファイルで学べること:
- Claude Codeのインストール
- APIキーの設定
- 初回起動の確認

---

## 前提条件

以下がインストールされていることを確認してください。

```bash
node --version   # v18以上が必要
npm --version    # npmが使えること
```

Node.jsが入っていない場合は https://nodejs.org/ からインストールしてください。

---

## インストール

```bash
npm install -g @anthropic-ai/claude-code
```

インストール確認:

```bash
claude --version
```

バージョン番号が表示されれば成功です。

---

## APIキーの設定

Claude Codeの利用にはAnthropicのAPIキーが必要です。

### APIキーの取得

1. https://console.anthropic.com/ にアクセス
2. アカウント作成またはログイン
3. 「API Keys」→「Create Key」でキーを発行

### 環境変数に設定

**Mac / Linux:**

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

毎回設定するのが面倒な場合は `~/.zshrc` または `~/.bashrc` に追記してください:

```bash
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc
source ~/.zshrc
```

**Windows (PowerShell):**

```powershell
$env:ANTHROPIC_API_KEY = "sk-ant-..."
```

永続化する場合:

```powershell
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "sk-ant-...", "User")
```

---

## 初回起動

プロジェクトのディレクトリに移動して起動します:

```bash
cd your-project
claude
```

初回起動時はセットアップの確認画面が表示されます。指示に従って進めてください。

起動すると `>` プロンプトが表示されます。日本語で話しかけてみましょう:

```
> このプロジェクトの構成を説明してください
```

---

## うまくいかない場合

**「command not found: claude」と表示される**
→ `npm install -g @anthropic-ai/claude-code` を再実行してください。Pathが通っていない場合は `npx @anthropic-ai/claude-code` でも起動できます。

**「Invalid API Key」エラー**
→ `ANTHROPIC_API_KEY` が正しく設定されているか確認してください: `echo $ANTHROPIC_API_KEY`

---

次のステップ: [01-basics.md](./01-basics.md) — 基本操作を学ぶ
