# はじめに — インストールと初回起動

## インストール

**Linux (Mac/WSL):**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows (PowerShell):**

Windowsでは事前に [Git for Windows](https://git-scm.com/downloads/win) のインストールが必要。

```powershell
irm https://claude.ai/install.ps1 | iex
```

## PATHの設定

インストールスクリプトは `claude` バイナリを以下の場所に配置する。

| OS | インストール先 |
|----|--------------|
| Linux | `~/.local/bin/` |
| Windows | `%USERPROFILE%\.local\bin\` |

インストール後にこのディレクトリをPATHに追加する必要がある。

**Linux (Mac/WSL):**

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

zshを使っている場合:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

設定後に `claude --version` が動作すれば成功。

**Windows (PowerShell):**

```powershell
$claudePath = "$env:USERPROFILE\.local\bin"
[System.Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";$claudePath", "User")
```

設定後は新しいターミナルを開いて `claude --version` が動作すれば成功。

## 起動してみる

```bash
# 作業したいプロジェクトのディレクトリに移動してから起動する
cd ~/path/to/your-project
claude
```

## 初回起動時の認証フロー（Proプラン）

1. `claude` を実行するとテーマ選択画面が表示される。好みのスタイルを選択する
2. ログイン方法の選択画面が表示される。`1. Claude account with subscription` を選択する
3. ターミナルにログイン用URLが表示される（ブラウザが自動で開かない場合は `c` キーでURLをコピーしてブラウザで開く）
4. ブラウザにAuthentication Codeが表示される。コードをコピーする
5. ターミナルの `Paste code here if prompted >` にコードを貼り付けて Enter
6. 認証完了後、`>` プロンプトが表示される

`>` プロンプトが出たら成功。日本語で話しかけてみる:

```
> このプロジェクトの構成を説明してください
```

ログアウトするには `/logout` と入力する。

## うまくいかない場合

**「command not found: claude」** → インストールコマンドを再実行する。

**「Invalid API Key」** → `echo $ANTHROPIC_API_KEY` でキーが設定されているか確認。

---

## 次のステップ

- [02-daily-use.md](./02-daily-use.md) — 毎日の作業パターン・使えるプロンプト集
