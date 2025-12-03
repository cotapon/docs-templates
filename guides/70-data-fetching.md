# データフェッチ戦略ガイド

{{PROJECT_NAME}} のデータ取得方針と実装ルールを説明します。

<!-- 
📝 書くべき内容:
- データ取得の基本方針
- 使用するライブラリ（SWR/React Query等）
- キャッシュ戦略
- エラーハンドリング
- ローディング状態の管理

カスタマイズ:
- プロジェクトで使用するデータフェッチライブラリに合わせて更新
-->

## 📚 目次

1. [基本方針](#基本方針)
2. [データ取得パターン](#データ取得パターン)
3. [キャッシュ戦略](#キャッシュ戦略)
4. [エラーハンドリング](#エラーハンドリング)
5. [ローディング状態](#ローディング状態)

---

## 基本方針

### 原則

1. **UseCase経由での取得**: データ取得は必ずUseCaseを通じて行う
2. **型安全性**: APIレスポンスは必ず型定義する
3. **エラーハンドリング**: 失敗時のユーザー体験を考慮
4. **キャッシュ活用**: 不要なリクエストを避ける

### データ取得ライブラリ

<!-- プロジェクトで使用するライブラリを記載 -->

**選択肢**:
- SWR
- TanStack Query (React Query)
- fetch + useState
- Server Components（Next.js）

---

## データ取得パターン

### パターン1: カスタムフック + UseCase

```typescript
// src/hooks/use-user.ts
import { useQuery } from '@tanstack/react-query'
import { getUserUseCase } from '@/core/application/use-cases/get-user'

/**
 * ユーザー情報を取得するフック
 */
export function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => getUserUseCase.execute(userId),
    staleTime: 5 * 60 * 1000, // 5分
  })
}
```

### パターン2: SWR

```typescript
// src/hooks/use-user.ts
import useSWR from 'swr'
import { getUserUseCase } from '@/core/application/use-cases/get-user'

export function useUser(userId: string) {
  const { data, error, isLoading, mutate } = useSWR(
    userId ? ['user', userId] : null,
    () => getUserUseCase.execute(userId)
  )

  return {
    user: data,
    isLoading,
    error: error?.message,
    refresh: mutate,
  }
}
```

### パターン3: Server Components（Next.js）

```typescript
// src/app/users/[id]/page.tsx
import { getUserUseCase } from '@/core/application/use-cases/get-user'

export default async function UserPage({ params }: { params: { id: string } }) {
  const user = await getUserUseCase.execute(params.id)
  
  return <UserProfile user={user} />
}
```

---

## キャッシュ戦略

### キャッシュキーの設計

```typescript
// 命名規則: [リソース名, 識別子, オプション]
const queryKeys = {
  users: {
    all: ['users'] as const,
    detail: (id: string) => ['users', id] as const,
    list: (filters: Filters) => ['users', 'list', filters] as const,
  },
  projects: {
    all: ['projects'] as const,
    detail: (id: string) => ['projects', id] as const,
  },
}
```

### キャッシュ時間の目安

| データ種類 | staleTime | 理由 |
|-----------|-----------|------|
| ユーザープロフィール | 5分 | 頻繁に変わらない |
| リスト一覧 | 1分 | 他ユーザーの更新を反映 |
| 設定情報 | 10分 | ほぼ変わらない |
| リアルタイムデータ | 0 | 常に最新が必要 |

### キャッシュの無効化

```typescript
// 特定のキャッシュを無効化
queryClient.invalidateQueries({ queryKey: ['users', userId] })

// 関連するキャッシュをすべて無効化
queryClient.invalidateQueries({ queryKey: ['users'] })
```

---

## エラーハンドリング

### フック内でのエラー処理

```typescript
export function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => getUserUseCase.execute(userId),
    retry: 3, // 3回リトライ
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  })
}
```

### コンポーネントでのエラー表示

```tsx
function UserProfile({ userId }: { userId: string }) {
  const { data: user, error, isLoading } = useUser(userId)

  if (isLoading) return <Skeleton />
  if (error) return <ErrorMessage message={error.message} />
  if (!user) return <NotFound />

  return <UserCard user={user} />
}
```

### グローバルエラーハンドリング

```typescript
// src/lib/query-client.ts
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 3,
      staleTime: 60 * 1000,
    },
    mutations: {
      onError: (error) => {
        // グローバルエラー通知
        toast.error(error.message)
      },
    },
  },
})
```

---

## ローディング状態

### Skeleton UIパターン

```tsx
// ローディング中はスケルトンを表示
function UserProfile({ userId }: Props) {
  const { user, isLoading } = useUser(userId)

  if (isLoading) {
    return (
      <Card>
        <Skeleton className="h-12 w-12 rounded-full" />
        <Skeleton className="h-4 w-[200px]" />
        <Skeleton className="h-4 w-[150px]" />
      </Card>
    )
  }

  return <UserCard user={user} />
}
```

### Suspenseパターン

```tsx
// React Suspense を使用
function UserPage({ userId }: Props) {
  return (
    <Suspense fallback={<UserProfileSkeleton />}>
      <UserProfile userId={userId} />
    </Suspense>
  )
}
```

---

## ミューテーション（データ更新）

### 楽観的更新

```typescript
export function useUpdateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (data: UpdateUserData) => updateUserUseCase.execute(data),
    onMutate: async (newData) => {
      // 既存のクエリをキャンセル
      await queryClient.cancelQueries({ queryKey: ['user', newData.id] })
      
      // 前の値を保存
      const previousUser = queryClient.getQueryData(['user', newData.id])
      
      // 楽観的に更新
      queryClient.setQueryData(['user', newData.id], newData)
      
      return { previousUser }
    },
    onError: (err, newData, context) => {
      // エラー時はロールバック
      queryClient.setQueryData(['user', newData.id], context?.previousUser)
    },
    onSettled: (data, error, variables) => {
      // 完了後にキャッシュを再検証
      queryClient.invalidateQueries({ queryKey: ['user', variables.id] })
    },
  })
}
```

---

## 関連ドキュメント

- [20-clean-architecture.md](./20-clean-architecture.md) - レイヤー設計
- [55-error-handling.md](./55-error-handling.md) - エラーハンドリング
- [80-testing.md](./80-testing.md) - データフェッチのテスト

---

**最終更新**: YYYY-MM-DD

