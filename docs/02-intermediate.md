# 02. 効果的な使い方・設定

このファイルで学べること:
- 良いプロンプトの書き方
- CLAUDE.mdでプロジェクトのルールを設定する方法
- よく使う設定

---

## 良いプロンプトの書き方

### 具体的に書く

```
# 悪い例
> バグを直して

# 良い例
> src/api/users.tsの42行目でTypeErrorが発生します。
> "Cannot read property 'id' of undefined" というエラーです。
> userオブジェクトがnullのケースを考慮して修正してください。
```

### コンテキストを与える

```
> このプロジェクトはNext.js 14 + TypeScript + Prismaで構成されています。
> ユーザー認証にNextAuthを使っています。
> パスワードリセット機能を追加してください。
```

### 制約を明示する

```
> テストを修正しないでください。実装側のコードだけ変更してください。

> 既存のコードスタイルに合わせてください。型は明示的に書いてください。

> 一度にすべて変更せず、まず方針を説明してから実装してください。
```

### 段階的に進める

大きなタスクは分割して依頼すると精度が上がります:

```
# Step 1: 方針確認
> 決済機能を追加したいです。Stripeを使う予定ですが、どんなファイルを作る必要があるか教えてください。

# Step 2: 実装
> 先ほどの方針で、まずStripeのWebhookハンドラーから実装してください。
```

---

## CLAUDE.md — プロジェクトのルール設定

CLAUDE.mdはプロジェクトのルールやコンテキストをClaude Codeに伝えるファイルです。
プロジェクトルートに置くと、起動時に自動で読み込まれます。

### 作成例

```markdown
# プロジェクトについて

このプロジェクトはECサイトのバックエンドAPIです。
Node.js 20 + TypeScript + Express + PostgreSQL (Prisma)を使っています。

## コーディングルール

- 関数は必ず型注釈を書く
- async/awaitを使う（Promiseチェーンは使わない）
- エラーハンドリングはResult型パターンを使う（src/types/result.ts参照）
- テストはVitest + MSWで書く

## よく使うコマンド

- `npm run dev` — 開発サーバー起動
- `npm test` — テスト実行
- `npm run lint` — Lint実行
- `npx prisma studio` — DB GUI起動

## 注意事項

- 本番環境のDBには直接繋がないこと
- APIキーは.envから取得すること（ハードコード禁止）
```

### 置き場所

```
your-project/
  CLAUDE.md          # プロジェクト全体のルール（ここが基本）
  src/
    CLAUDE.md        # srcディレクトリ固有のルール（任意）
```

---

## よく使う設定

### モデルの選択

```bash
claude --model claude-opus-4-5    # 最高性能（複雑なタスク向け）
claude --model claude-sonnet-4-5  # バランス型（デフォルト）
claude --model claude-haiku-4-5   # 高速・低コスト（簡単なタスク向け）
```

### 設定ファイル

`~/.claude/settings.json` でデフォルト設定を変えられます:

```json
{
  "model": "claude-sonnet-4-5",
  "autoApprove": false
}
```

---

## トラブルシューティング

**Claudeが的外れな回答をする**
→ `/clear` でリセットして、より具体的に指示し直してください。

**コンテキストが長くなって遅くなった**
→ `/compact` で会話を要約するか、`/clear` でリセットしてください。

**変更を取り消したい**
→ `git diff` で変更を確認して `git checkout -- <file>` で戻せます。Claude Codeに「さっきの変更を元に戻してください」と頼む方法もあります。

**意図しないファイルを変更された**
→ 設定で `autoApprove: false`（デフォルト）にしておくと、変更前に確認が入ります。

---

次のステップ: [03-advanced.md](./03-advanced.md) — 応用機能
