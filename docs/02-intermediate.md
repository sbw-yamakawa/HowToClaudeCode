# 効果的な使い方・設定

## プロンプトの書き方

**具体的に書く**

```
# 悪い
> バグを直して

# 良い
> src/api/users.tsの42行目でTypeErrorが発生します。
> "Cannot read property 'id' of undefined" というエラーです。
> userがnullのケースを考慮して修正してください。
```

**コンテキストを与える**

```
> このプロジェクトはNext.js 14 + TypeScript + Prismaで構成されています。
> 認証にNextAuthを使っています。
> パスワードリセット機能を追加してください。
```

**制約を明示する**

```
> テストは変更しないでください。実装側だけ直してください。
> 一度にすべて変更せず、まず方針を説明してから実装してください。
```

**大きなタスクは分割する**

```
# Step 1: 方針確認
> 決済機能を追加したいです。どんなファイルを作る必要があるか教えてください。

# Step 2: 実装
> では、まずStripeのWebhookハンドラーから実装してください。
```

## CLAUDE.md — プロジェクトルールの設定

プロジェクトルートに置くと起動時に自動で読み込まれる。毎回同じ指示を書かなくて済む。

**`/init` で自動生成する（推奨）**

```bash
> /init
```

プロジェクトを自動解析して CLAUDE.md の初期テンプレートを生成してくれる。手動で一から書くより、既存コードベースの構成・使用技術に合った内容が得られる。生成後に不要な項目を削除したり、チーム固有のルールを追記するだけでよい。

**手動で書く場合の例:**

```markdown
# プロジェクトについて

ECサイトのバックエンドAPI。Node.js 20 + TypeScript + Express + PostgreSQL (Prisma)。

## コーディングルール

- 関数は必ず型注釈を書く
- async/awaitを使う（Promiseチェーンは使わない）
- エラーハンドリングはResult型パターン（src/types/result.ts参照）
- テストはVitest + MSWで書く

## よく使うコマンド

- `npm run dev` — 開発サーバー起動
- `npm test` — テスト実行
- `npm run lint` — Lint実行

## 注意事項

- 本番DBには直接繋がない
- APIキーは.envから取得（ハードコード禁止）
```

サブディレクトリに置くこともできる:

```
your-project/
  CLAUDE.md          # プロジェクト全体のルール
  src/
    CLAUDE.md        # srcディレクトリ固有のルール（任意）
```

## よく使う設定

**モデルの選択**

```bash
claude --model claude-opus-4-5    # 最高性能（複雑なタスク向け）
claude --model claude-sonnet-4-5  # バランス型（デフォルト）
claude --model claude-haiku-4-5   # 高速・低コスト
```

**設定ファイル** (`~/.claude/settings.json`)

```json
{
  "model": "claude-sonnet-4-5",
  "autoApprove": false
}
```

## トラブルシューティング

**的外れな回答が続く** → `/clear` でリセットして指示し直す

**会話が長くなって遅い** → `/compact` で要約するか `/clear` でリセット

**変更を取り消したい** → `git checkout -- <file>` か「さっきの変更を元に戻してください」と頼む

**意図しないファイルを変更された** → `autoApprove: false`（デフォルト）にしておけば変更前に確認が入る
