# dotfiles で Claude Code をカスタマイズする

配布している dotfiles リポジトリをインポートすると使えるようになる機能の説明。

## セットアップ

```bash
git clone <dotfiles-repo-url>
cd dotfiles
bash setup.sh
```

以下が `~/.claude/` にシンボリックリンクとして配置される:

- `~/.claude/rules/` — AIの振る舞いを定義するルール
- `~/.claude/agents/` — カスタムエージェント定義
- `~/.claude/skills/` — スラッシュコマンドで呼び出せるスキル
- `~/.claude/plugins/` — インストール済みプラグイン設定
- `~/.claude/settings.json` — Claude Code 全体の設定

## 1. ルール（Rules）

`~/.claude/rules/` のMarkdownファイルが全プロジェクトで自動適用される。「毎回書かなくていいCLAUDE.md」として機能する。

**`agents.md`** — エージェント自動起動ルール

```
複雑な機能リクエスト → planner エージェント
コード変更直後     → code-reviewer エージェント
バグ修正・新機能   → tdd-guide エージェント
```

**`coding-style.md`** — コーディングスタイル

- オブジェクトは常に新規作成（ミューテーション禁止）
- ファイルは200〜400行、最大800行
- コミット前のセキュリティチェック必須

**`git-workflow.md`** — Gitワークフロー

コミットメッセージのフォーマット（`feat:`, `fix:`, `refactor:` など）とPR作成手順。

**`verification.md`** — 検証戦略

実装完了の基準と検証手順。テストカバレッジ80%以上を要求する。

## 2. カスタムエージェント（Agents）

`~/.claude/agents/` のMarkdownファイルがサブエージェントとして使える。Claude Codeが適切なタイミングで自動起動する。

| エージェント | 役割 | 自動起動のタイミング |
|-------------|------|-------------------|
| `planner` | 実装計画・アーキテクチャ設計 | 複雑な機能リクエスト時 |
| `tdd-guide` | テスト駆動開発の実践 | 新機能追加・バグ修正時 |
| `code-reviewer` | コード品質レビュー | コード変更直後 |
| `security-reviewer` | セキュリティ脆弱性の検出 | コミット前 |
| `build-error-resolver` | ビルド・型エラーの修正 | ビルド失敗時 |
| `e2e-runner` | Playwright E2Eテストの実行 | 重要なユーザーフロー確認時 |
| `refactor-cleaner` | 不要コードの削除 | コードメンテナンス時 |

**使用例**

```
> 認証システムを実装したい

→ planner が自動起動して実装計画を作成
→ tdd-guide がテストを先に書く
→ code-reviewer が実装後にレビュー
→ security-reviewer が認証コードをセキュリティチェック
```

## 3. スキル（Skills）

`/スキル名` で呼び出せるカスタムスラッシュコマンド。

**`/commit`** — 規約に沿ったコミット作成。stagingされたファイルを確認して、規約に沿ったメッセージを提案・実行する。

**`/handover`** — セッション引き継ぎドキュメント作成。現在の作業状況・決定事項・次のアクションをまとめる。

**`/tdd-workflow`** — RED→GREEN→REFACTORサイクルの原則とチェックリストをコンテキストに注入する。

**`/security-review`** — 認証・SQLインジェクション・XSS・シークレット管理など、包括的なセキュリティチェックリストを適用する。

**`/pptx`** — PowerPointファイルの作成・編集（pptxgenjs使用）。

## 4. プラグイン（Plugins）

セットアップスクリプトが自動でインストールするプラグイン。

| プラグイン | 機能 |
|----------|------|
| **superpowers** | スキルシステム（スラッシュコマンド）を有効化 |
| **serena** | コードのシンボル解析・セマンティック検索 |
| **github** | GitHub Issues/PR の操作 |
| **playwright** | ブラウザ自動操作・スクリーンショット |
| **context7** | ライブラリの最新ドキュメントを自動取得 |

### superpowers — スキルシステムの有効化

**変化前:** `/commit` などのスラッシュコマンドは存在しない。Claude Code のデフォルト動作のみ。

**変化後:** カスタムコマンドが使えるようになり、状況を検知して自動適用される。

```
# superpowers なし
> /commit  ← 「コマンドが見つかりません」

# superpowers あり
> /commit  ← staging 内容を確認し、規約に沿ったメッセージを提案・実行
```

代表的なスキルの動作:

`/superpowers:brainstorming` — 実装前の要件整理。コードを書く前に Claude が質問してくる:

```
> 認証システムを作りたい

← Claude が質問:
  「JWT とセッションどちらを使いますか？」
  「ソーシャルログインは必要ですか？」
  「リフレッシュトークンの有効期限は？」
→ 要件が具体化されてから実装を開始
```

`/superpowers:systematic-debugging` — バグに遭遇したとき「とりあえず試す」を禁止し、4フェーズで根本原因を追う:

```
Phase 1: 症状の観察（何が起きているか）
Phase 2: 仮説の形成（なぜ起きているか）
Phase 3: 仮説の検証（証拠を集める）
Phase 4: 修正と検証（直した証拠を示す）
```

スキルは Markdown ファイルとして定義されており、チーム間で共有・バージョン管理できる。

### serena — シンボルレベルのコード理解

**変化前:** Claude はファイル全体を読んでからコードを理解する。大きなファイルでトークン消費が多い。

**変化後:** 関数・クラス単位でシンボルを検索し、必要な部分だけ読む。

```
# serena なし
> UserService クラスの認証メソッドを修正してください
→ user-service.ts（400行）を全部読む → 修正

# serena あり
→ get_symbols_overview で UserService の構造を把握
→ authenticate() メソッドの本体だけ読む → 修正
（トークン消費: 1/10 程度）
```

追加されるツール:

| ツール | 用途 |
|--------|------|
| `find_symbol` | クラス・関数名で即座に場所を特定 |
| `get_symbols_overview` | ファイルの構造を一覧で把握 |
| `find_referencing_symbols` | その関数を使っている箇所を全探索 |

### github — GitHub との直接連携

**変化前:** `gh` CLI コマンドを手動で打つか、ブラウザで操作する。

**変化後:** 会話の中で Issue/PR を操作できる。

```
> このバグの Issue を作成して、マイルストーン「v2.0」に紐付けて
→ API 経由で直接 GitHub に作成

> PR #123 のレビューコメントを確認して
→ コメント内容をフェッチして要約・対応
```

`gh` コマンドと異なる点:
- CLI コマンドを覚えなくてよい
- 認証情報の取り回しが自動化される
- 結果をコンテキストに保持してタスクを連続実行できる

### playwright — ブラウザを Claude の「目と手」にする

**変化前:** UI の動作確認は人間が手動で行う。

**変化後:** Claude がブラウザを操作して確認・テストできる。

```
> localhost:3000 でログインフローをテストして
→ ブラウザを起動
→ フォームに入力
→ スクリーンショット撮影
→ 「エラーメッセージが表示されています: パスワードが短すぎます」と報告
```

特に有効なシーン:
- 実装後の視覚的確認（見た目が崩れていないか）
- E2E テストの自動作成・実行
- API が返したデータが画面に正しく表示されているかの検証

### context7 — 「古い知識」問題の解消

**変化前:** Claude は学習データに基づいて回答する。ライブラリの API が変わっていても古い書き方を提案する可能性がある。

**変化後:** 回答前に最新の公式ドキュメントを自動フェッチする。

```
# context7 なし
> Next.js 15 の App Router でキャッシュを無効化したい
→ 古いバージョンの revalidate 書き方を提案（動かない）

# context7 あり
→ Next.js 15 の公式ドキュメントを取得
→ unstable_noStore() の正しい使い方を提案（最新）
```

特に効果が大きい状況:
- メジャーバージョンアップが激しいフレームワーク（Next.js, React, Prisma）
- SDK の API が変更されたケース
- 設定ファイルの書き方が変わった場合（webpack → vite 等）

## 5. settings.json

**許可済みコマンド** — よく使うコマンドを事前許可して確認プロンプトをスキップ

```json
"permissions": {
  "allow": [
    "Bash(git checkout:*)",
    "Bash(git commit:*)",
    "Bash(git push:*)",
    "Bash(npm:*)",
    "Bash(node:*)"
  ],
  "defaultMode": "dontAsk"
}
```

**タイムスタンプ自動挿入フック** — プロンプト送信時に現在時刻を自動挿入。「今日」「明日」などの相対表現を正確に解釈させるため。

```json
"hooks": {
  "UserPromptSubmit": [
    {
      "hooks": [{ "type": "command", "command": "bash ~/.claude/scripts/inject-timestamp.sh" }]
    }
  ]
}
```

**カスタムステータスライン** — 入力欄の上にモデル名・変更行数・Gitブランチ・コンテキスト使用率をリアルタイム表示

```json
"statusLine": {
  "type": "command",
  "command": "bash ~/.claude/statusline-command.sh"
}
```

```
So4.6 │ +12/-3 │ feature/auth │ myproject
ctx ▊▊▊░░ 35% │ 5h ████░ 52% │ 7d ██░░░ 23%
```

**日本語設定**

```json
"language": "Japanese"
```

**実験的なエージェントチーム機能**

```json
"env": {
  "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
}
```
