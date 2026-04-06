# git worktree コンテンツ追加 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `02-daily-use.md` と `05-extend.md` に git worktree の使い方を追加し、それぞれ独立して参照できるドキュメントとする。

**Architecture:** 2つのファイルをそれぞれ独立したタスクとして編集する。`02-daily-use.md` は「ブランチ同時編集」、`05-extend.md` は「エージェント隔離実行」にフォーカスする。

**Tech Stack:** Markdown（`.md`）のみ。テストはファイルを読んで内容・配置を確認する形で行う。

---

## 変更ファイル一覧

| 操作 | ファイル | 内容 |
|------|---------|------|
| Modify | `docs/02-daily-use.md` | 「セッション管理」直後に worktree セクション追加 |
| Modify | `docs/05-extend.md` | 「エージェント」直後・「プラグイン」前に worktree セクション追加 |

---

### Task 1: `02-daily-use.md` に worktree セクションを追加する

**Files:**
- Modify: `docs/02-daily-use.md`（「セッション管理」セクション末尾の直後、`## 次のステップ` の直前に挿入）

- [ ] **Step 1: ファイルの挿入位置を確認する**

```bash
grep -n "次のステップ" docs/02-daily-use.md
```

期待出力例: `316:## 次のステップ`（行番号は前後するが、`## 次のステップ` の直前が挿入位置）

- [ ] **Step 2: `## 次のステップ` の直前に以下を挿入する**

`docs/02-daily-use.md` の `## 次のステップ` の直前（空行を1行入れてから）に以下のブロックを追加する:

```markdown
---

## git worktree — 複数ブランチを同時に作業する

### Claudeに頼む（推奨）

feature ブランチで作業中に緊急対応が入った場合:

```
> feature/payment ブランチで作業中ですが、main に緊急バグが見つかりました。
> git worktree を使って、今の作業を壊さずに hotfix ブランチを別のフォルダで作業したいです。
```

Claude がワークツリーの作成から作業ディレクトリの案内まで行ってくれる。

### 仕組み

通常の git はブランチを切り替えると作業ディレクトリが変わる。worktree を使うと、**複数のブランチを別々のディレクトリで同時にチェックアウト**できる。

```
プロジェクト/
├── my-project/          # メイン (feature/payment)
│   └── ...
└── my-project-hotfix/   # worktree (hotfix/login-bug)
    └── ...
```

2つのターミナルウィンドウを開いて、それぞれで独立して作業できる。

### 基本コマンド集

```bash
# worktree を追加（新しいブランチを作りながら）
git worktree add ../my-project-hotfix -b hotfix/login-bug

# worktree を追加（既存ブランチを使う）
git worktree add ../my-project-hotfix hotfix/login-bug

# 現在の worktree 一覧を確認
git worktree list

# worktree を削除
git worktree remove ../my-project-hotfix

# 削除済みワークツリーの参照を掃除
git worktree prune
```

### よくある使い方パターン

**ホットフィックス対応**

feature ブランチで作業中に緊急バグの対応が必要になったとき:

```bash
# main から hotfix ブランチを作成して別フォルダでチェックアウト
git worktree add ../my-project-hotfix -b hotfix/login-bug main
cd ../my-project-hotfix
# → 緊急バグを修正してコミット・マージ
cd ../my-project
git worktree remove ../my-project-hotfix
```

**PRレビュー確認**

レビュー対象ブランチをビルドして動作確認したいとき:

```bash
# レビュー用に別ディレクトリでチェックアウト
git worktree add ../my-project-review feature/other-team-pr
cd ../my-project-review
# → ビルド・動作確認
cd ../my-project
git worktree remove ../my-project-review
```

**長期ブランチの並行維持**

develop と main を常に参照したいとき:

```bash
# main を常に別ディレクトリで開いておく
git worktree add ../my-project-main main
# → リリースチェックや差分確認に使う
```

### 注意点

**同一ファイルを2か所で同時に編集しない**

worktree は同じ Git オブジェクトを共有している。両方の worktree で同じファイルを変更すると、コンフリクトが発生しやすい。編集するファイルをディレクトリ単位で分けて作業する。

**node_modules / .env は共有されない**

worktree 内では `npm install` を再実行する必要がある。`.env` もメインからコピーが必要。

**同じブランチを2つの worktree でチェックアウトできない**

同一ブランチは1つの worktree にしか割り当てられない。別のブランチを派生させて使う。

### クリーンアップ

```bash
# worktree のディレクトリを削除してから登録も解除
git worktree remove ../my-project-hotfix

# ディレクトリが手動削除済みの場合は prune で参照を掃除
git worktree prune

# 全 worktree の確認
git worktree list
```
```

- [ ] **Step 3: 挿入後にファイルを読んで配置を確認する**

`docs/02-daily-use.md` を開き、以下を確認する:
- `## git worktree — 複数ブランチを同時に作業する` が存在する
- `## セッション管理` の直後（`---` で区切られている）に配置されている
- `## 次のステップ` の直前に配置されている
- `### Claudeに頼む（推奨）` → `### 仕組み` → `### 基本コマンド集` → `### よくある使い方パターン` → `### 注意点` → `### クリーンアップ` の順になっている

- [ ] **Step 4: コミットする**

```bash
git add docs/02-daily-use.md
git commit -m "docs: git worktree — 複数ブランチ同時編集の使い方を02-daily-use.mdに追加"
```

---

### Task 2: `05-extend.md` に worktree セクションを追加する

**Files:**
- Modify: `docs/05-extend.md`（「エージェント」セクション末尾の直後、`## プラグイン` の直前に挿入）

- [ ] **Step 1: ファイルの挿入位置を確認する**

```bash
grep -n "## プラグイン" docs/05-extend.md
```

期待出力例: `385:## プラグイン — まとめてインポートする`（行番号は前後するが、この見出しの直前が挿入位置）

- [ ] **Step 2: `## プラグイン` の直前に以下を挿入する**

`docs/05-extend.md` の `## プラグイン — まとめてインポートする` の直前（空行を1行入れてから）に以下のブロックを追加する:

```markdown
---

## git worktree — エージェントに隔離環境を与える

### Claudeに頼む（推奨）

```
> 実装計画を実行する際、メインブランチに影響を与えたくありません。
> 各サブエージェントを独立した git worktree で動かしてください。
```

Claude がワークツリーの作成・割り当て・後片付けまで行ってくれる。

### 仕組み

サブエージェントを worktree 内で動かすと、**メインの作業ブランチを汚さずに実験的な実装**ができる。エージェントが変更を加えても、worktree を削除すれば元に戻せる。

```
メインClaude（master ブランチ）
  ├── サブエージェントA → worktree-a/ (feature/auth)
  ├── サブエージェントB → worktree-b/ (feature/payment)
  └── サブエージェントC → worktree-c/ (feature/notification)
```

各エージェントが独立したブランチ＋ディレクトリで作業するため、互いのファイル変更が衝突しない。

### ユースケース

**実装計画の並列実行**

独立したタスクを複数のエージェントで同時に進める:

```
> 以下を worktree を使って並列で実行してください:
> 1. src/auth/ のユニットテストを追加する
> 2. src/api/ の JSDoc コメントを補完する
> 3. 未使用の import を全ファイルから削除する
> 各タスクは独立した worktree で行ってください。
```

**破壊的変更の試験的実施**

```
> 依存ライブラリを v2 → v3 に更新したい。
> まず worktree を作って試してみてください。
> 問題なければ main にマージ、問題あれば worktree を削除してください。
```

**E2Eテスト用の隔離環境**

```
> E2Eテストを別の worktree で実行してください。
> テスト中にファイルが変更されても、メインの作業に影響しないようにしたいです。
```

### worktree エージェントのライフサイクル

```
1. worktree 作成
   git worktree add ../feature-worktree -b feature/xxx
         ↓
2. エージェントが worktree 内で作業
   （ファイル変更・コミット）
         ↓
3a. 成功 → メインブランチにマージ
    git merge feature/xxx
    git worktree remove ../feature-worktree
         ↓
3b. 失敗 → worktree ごと破棄
    git worktree remove --force ../feature-worktree
    git branch -D feature/xxx
```

### 注意点・トラブルシューティング

**worktree 残骸の掃除**

エージェントが異常終了した場合、worktree のディレクトリが残ることがある:

```bash
# 参照が切れた worktree を掃除
git worktree prune

# 全 worktree を確認
git worktree list
```

**コンフリクト発生時の対処**

複数のエージェントが同じファイルを変更した場合はマージコンフリクトが起きる。エージェントごとに担当ディレクトリを明示して指示することで防げる:

```
> エージェントAは src/auth/ のみ、エージェントBは src/api/ のみを変更してください。
```

**node_modules の重複に注意**

worktree ごとに `node_modules` が必要になるため、ディスク容量を多く使う。ライブラリのインストールが重い場合は `--no-checkout` で必要なファイルだけを取得する方法もある:

```bash
git worktree add --no-checkout ../feature-worktree -b feature/xxx
```
```

- [ ] **Step 3: 挿入後にファイルを読んで配置を確認する**

`docs/05-extend.md` を開き、以下を確認する:
- `## git worktree — エージェントに隔離環境を与える` が存在する
- 「エージェント」セクション（`## エージェント — 専門家の役割を定義する`）の末尾直後に配置されている
- `## プラグイン — まとめてインポートする` の直前に配置されている
- `### Claudeに頼む（推奨）` → `### 仕組み` → `### ユースケース` → `### worktree エージェントのライフサイクル` → `### 注意点・トラブルシューティング` の順になっている

- [ ] **Step 4: コミットする**

```bash
git add docs/05-extend.md
git commit -m "docs: git worktree — エージェント隔離実行の使い方を05-extend.mdに追加"
```

---

## セルフレビュー結果

**スペックカバレッジ:**
- [x] 複数ブランチ同時編集（02-daily-use.md） → Task 1
- [x] エージェント隔離実行（05-extend.md） → Task 2
- [x] Claudeに頼む → 仕組み → コマンド → パターン → 注意点 → クリーンアップ の順
- [x] 各ファイルが独立して完結している
- [x] コマンドがコピペで動く形式

**プレースホルダー:** なし

**型・命名の一貫性:** ファイル名・セクションタイトル・コマンドはTask 1・Task 2で重複する箇所がなく、それぞれのユースケースに沿った命名になっている。
