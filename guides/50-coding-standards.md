# コーディング規約

開発時に必ず守るべきルールをまとめます。迷った場合は [AGENTS.md](../../AGENTS.md) と ESLint の設定を参照してください。

<!-- 
📝 書くべき内容:
- 言語・フレームワーク固有のルール
- 命名規則（ファイル、変数、関数、コンポーネント）
- フォーマット（インデント、改行）
- 禁止事項（any禁止、console.log禁止など）
- インポート順序
- エラーハンドリング方針
- コメント・JSDocのルール

★必須ファイル: すべてのプロジェクトで作成してください
-->

## 共通原則

- TypeScript は `strict` 前提。`any` の使用禁止、null 安全を徹底する
- YAGNI: 要求されていない機能や抽象化は入れない
- 変更後は `npm run lint && npx tsc --noEmit && npm run test` を必ず実行
- ログは `console` を多用せず、必要な箇所のみ warn/error で残す

<!-- OPTIONAL: ユーティリティライブラリの指定がある場合 -->
## ユーティリティライブラリ

<!-- 例: lodash禁止、es-toolkit使用など -->
- **lodash は使用禁止**: 代わりに `es-toolkit` を使用する
- 理由: バンドルサイズの削減、Tree-shaking の最適化
<!-- /OPTIONAL -->

---

## 命名規則

| 種別 | 規則 | 例 |
|------|------|-----|
| ファイル名 | kebab-case | `user-profile.tsx` |
| コンポーネント | PascalCase | `UserProfile` |
| 関数 | camelCase | `getUserById` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 型/インターフェース | PascalCase | `UserProfile`, `IUserRepository` |
| フック | `use` プレフィックス | `useAuth`, `useUser` |
| テストファイル | `*.spec.ts(x)` | `UserProfile.spec.tsx` |
| イベントハンドラー | `handle` + 動詞 | `handleClick`, `handleSubmit` |
| boolean変数 | `is/has/can` + 形容詞/名詞 | `isLoading`, `hasError`, `canEdit` |

### テストファイルの配置

- 対象ファイルと同階層に配置
- 同一ディレクトリに複数 spec が並ぶ場合は `__tests__/` 配下へまとめる

---

## インポート順序

```typescript
// 1. 外部ライブラリ（React系を先頭に）
import { useState, useEffect } from 'react'
import { clsx } from 'clsx'

// 2. 内部モジュール（エイリアス使用）
import { Button } from '@/components/ui/button'
import { useAuth } from '@/hooks/use-auth'

// 3. 相対パス（同一ディレクトリ・親ディレクトリ）
import { UserProfileProps } from './types'
import { formatDate } from '../utils'

// 4. 型のみのインポート（type キーワード使用）
import type { User } from '@/types'
```

### インポートのルール

```typescript
// ✅ Good: エイリアス使用
import { Button } from '@/components/ui/button'

// ❌ Bad: 深いネスト
import { Button } from '../../../components/ui/button'
```

---

## 型・関数設計

### 戻り値の型

戻り値は可能な限り具体的な型を返す：

```typescript
// ✅ Good: 具体的な型
function getUser(id: string): User | null {
  // ...
}

// ❌ Bad: any
function getUser(id: string): any {
  // ...
}
```

### 副作用の明示

関数は副作用の有無をコメントか命名で明示：

```typescript
/**
 * ユーザーを取得する
 * @remarks 副作用: API呼び出し
 */
async function fetchUser(id: string): Promise<User> {
  // ...
}

/**
 * ユーザー名を整形する（純粋関数）
 */
function formatUserName(user: User): string {
  return `${user.firstName} ${user.lastName}`
}
```

### ユーティリティの切り出し

再利用できるロジックは `src/lib/` などのユーティリティへ切り出す。

---

## 禁止事項

| 項目 | 理由 | 代替案 |
|------|------|--------|
| `any` の使用 | 型安全性の喪失 | `unknown` + 型ガード |
| `// @ts-ignore` | 型エラーの隠蔽 | 適切な型定義 |
| `console.log` の本番残留 | パフォーマンス・セキュリティ | ロガー使用 or 削除 |
| 直接の DOM 操作 | React の仮想 DOM と競合 | useRef + useEffect |
| `var` の使用 | スコープの問題 | `const` / `let` |

---

## エラーハンドリング

詳細は [55-error-handling.md](./55-error-handling.md) を参照。

### 基本方針

- 非同期処理は `try/catch` で囲む
- ユーザー向けメッセージと開発者向けログを分離
- `Promise.all` 使用時は部分失敗の挙動を設計する

```typescript
// エラーハンドリング例
try {
  const result = await fetchData()
  return result
} catch (error) {
  // 開発者向けログ
  console.error('Data fetch failed:', error)
  // ユーザー向けエラー
  throw new Error('データの取得に失敗しました')
}
```

---

## コンポーネント設計

### Props の設計

```typescript
// ✅ Good: 明示的な Props 型
interface ButtonProps {
  label: string
  onClick: () => void
  disabled?: boolean
}

export function Button({ label, onClick, disabled = false }: ButtonProps) {
  // ...
}
```

### デフォルト値

```typescript
// ✅ Good: デストラクチャリングでデフォルト値
function Component({ size = 'md', variant = 'primary' }: Props) {
  // ...
}
```

---

## テストと品質

- Given/When/Then（または Arrange/Act/Assert）でテスト記述を統一
- コンテナとプレゼンターを分離し、それぞれを独立にテスト
- コードの差分に対して少なくともユニットテストを追加

詳細は [80-testing.md](./80-testing.md) を参照。

---

## ドキュメント

### JSDoc

すべての公開関数・コンポーネントには JSDoc を記述：

```typescript
/**
 * ユーザープロフィールを表示するコンポーネント
 * @param userId - 表示するユーザーのID
 * @returns ユーザープロフィールのJSX
 */
export function UserProfile({ userId }: { userId: string }) {
  // ...
}
```

### コメントのルール

```typescript
// ✅ Good: なぜそうしたかを説明
// 外部APIの制限により、100件ずつ分割して取得
const BATCH_SIZE = 100

// ❌ Bad: 何をしているかの説明（コードを見ればわかる）
// 配列をループ
for (const item of items) { ... }
```

---

## コミットメッセージ

Conventional Commits 形式を使用：

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type の種類

| Type | 用途 |
|------|------|
| `feat` | 新機能 |
| `fix` | バグ修正 |
| `docs` | ドキュメント |
| `style` | フォーマット |
| `refactor` | リファクタリング |
| `test` | テスト追加・修正 |
| `chore` | ビルド・設定変更 |

### 例

```
feat(auth): ログイン機能を追加

OAuthプロバイダーとしてGoogleを追加しました。

Closes #123
```

---

## 関連ドキュメント

- [AGENTS.md](../../AGENTS.md) - AI向けプロジェクトコンテキスト
- [55-error-handling.md](./55-error-handling.md) - エラーハンドリング
- [80-testing.md](./80-testing.md) - テスト戦略
- [90-development-workflow.md](./90-development-workflow.md) - 開発ワークフロー

---

**最終更新**: YYYY-MM-DD

