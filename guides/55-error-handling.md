# エラーハンドリングガイド

{{PROJECT_NAME}} のエラーハンドリング方針と実装パターンを説明します。

<!-- 
📝 書くべき内容:
- エラーの分類（ビジネスエラー/システムエラー/バリデーションエラー）
- 各層でのエラー処理方針
- ユーザー向けメッセージと開発者向けログの分離
- 共通エラー定義

カスタマイズ:
- プロジェクトのエラー処理方針に合わせて更新
-->

## 📚 目次

1. [エラーの分類](#エラーの分類)
2. [各層のエラーハンドリング](#各層のエラーハンドリング)
3. [エラーメッセージの設計](#エラーメッセージの設計)
4. [実装パターン](#実装パターン)

---

## エラーの分類

### ビジネスエラー

ビジネスルール違反によるエラー。ユーザーに具体的なメッセージを表示。

例:
- 権限不足
- バリデーションエラー
- リソース未存在

### システムエラー

予期しないシステムの問題。一般的なエラーメッセージを表示し、詳細はログに記録。

例:
- ネットワークエラー
- データベース接続エラー
- 外部API障害

---

## 各層のエラーハンドリング

### Domain層

- 純粋なビジネスルール違反を検出
- カスタムエラークラスを使用

```typescript
// src/core/domain/errors/domain-errors.ts
export class ValidationError extends Error {
  constructor(message: string) {
    super(message)
    this.name = 'ValidationError'
  }
}

export class NotFoundError extends Error {
  constructor(resource: string, id: string) {
    super(`${resource} not found: ${id}`)
    this.name = 'NotFoundError'
  }
}
```

### Application層（UseCase）

- Domain/Infrastructureのエラーを捕捉
- ユーザー向けメッセージに変換

```typescript
// src/core/application/use-cases/get-user.ts
export class GetUserUseCase {
  async execute(userId: string): Promise<User> {
    try {
      const user = await this.userRepository.findById(userId)
      if (!user) {
        throw new NotFoundError('User', userId)
      }
      return user
    } catch (error) {
      // ユーザー向けメッセージに変換
      throw this.translateError(error)
    }
  }

  private translateError(error: unknown): Error {
    if (error instanceof NotFoundError) {
      return new AppError('ユーザーが見つかりませんでした', error)
    }
    return new AppError('予期しないエラーが発生しました', error)
  }
}
```

### Infrastructure層

- 外部サービスのエラーをラップ
- リトライ処理を実装

```typescript
// src/core/infrastructure/api/api-client.ts
export class ApiClient {
  async get<T>(url: string): Promise<T> {
    try {
      const response = await fetch(url)
      if (!response.ok) {
        throw new ApiError(response.status, await response.text())
      }
      return response.json()
    } catch (error) {
      if (error instanceof ApiError) throw error
      throw new NetworkError('ネットワーク接続に問題があります')
    }
  }
}
```

### Presentation層（UI）

- エラー状態を表示
- エラーバウンダリで予期しないエラーをキャッチ

```typescript
// src/components/features/UserProfile/UserProfile.presenter.tsx
export function UserProfilePresenter({ error }: Props) {
  if (error) {
    return (
      <Alert variant="destructive">
        <AlertDescription>{error}</AlertDescription>
      </Alert>
    )
  }
  // ...
}
```

---

## エラーメッセージの設計

### ユーザー向けメッセージ

| カテゴリ | メッセージ例 |
|---------|------------|
| 認証エラー | 「ログインが必要です」 |
| 権限エラー | 「この操作を行う権限がありません」 |
| バリデーション | 「メールアドレスの形式が正しくありません」 |
| 未存在 | 「指定されたデータが見つかりませんでした」 |
| ネットワーク | 「通信エラーが発生しました。再度お試しください」 |
| システム | 「予期しないエラーが発生しました」 |

### 開発者向けログ

```typescript
// 開発者向けには詳細情報をログ出力
console.error('[GetUserUseCase] Failed to fetch user', {
  userId,
  error: originalError,
  stack: originalError.stack,
})
```

---

## 実装パターン

### カスタムエラークラス

```typescript
// src/core/domain/errors/app-error.ts
export class AppError extends Error {
  /** ユーザーに表示するメッセージ */
  readonly userMessage: string
  /** 元のエラー（デバッグ用） */
  readonly cause?: Error

  constructor(userMessage: string, cause?: Error) {
    super(cause?.message ?? userMessage)
    this.name = 'AppError'
    this.userMessage = userMessage
    this.cause = cause
  }
}
```

### エラーメッセージ定数

```typescript
// src/core/domain/constants/error-messages.ts
export const ERROR_MESSAGES = {
  AUTH: {
    REQUIRED: 'ログインが必要です',
    EXPIRED: 'セッションが期限切れです。再度ログインしてください',
    FORBIDDEN: 'この操作を行う権限がありません',
  },
  VALIDATION: {
    REQUIRED: (field: string) => `${field}は必須です`,
    INVALID_FORMAT: (field: string) => `${field}の形式が正しくありません`,
  },
  NETWORK: {
    ERROR: '通信エラーが発生しました。再度お試しください',
    TIMEOUT: 'リクエストがタイムアウトしました',
  },
  SYSTEM: {
    UNEXPECTED: '予期しないエラーが発生しました',
  },
} as const
```

### フック内でのエラーハンドリング

```typescript
// src/hooks/use-user.ts
export function useUser(userId: string) {
  const [error, setError] = useState<string | null>(null)

  const fetchUser = useCallback(async () => {
    try {
      const user = await getUser(userId)
      return user
    } catch (e) {
      const message = e instanceof AppError 
        ? e.userMessage 
        : ERROR_MESSAGES.SYSTEM.UNEXPECTED
      setError(message)
      return null
    }
  }, [userId])

  // ...
}
```

---

## 関連ドキュメント

- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [80-testing.md](./80-testing.md) - エラーケースのテスト方針
- [20-clean-architecture.md](./20-clean-architecture.md) - レイヤー間のエラー伝播

---

**最終更新**: YYYY-MM-DD

