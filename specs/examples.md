# 実装サンプル集

{{PROJECT_NAME}} での典型的な実装パターンのコードサンプルを集めています。

<!-- 
📝 書くべき内容:
- よく使う実装パターンのサンプルコード
- テストコードのサンプル
- APIモックのサンプル
- エラーハンドリングのサンプル

カスタマイズ:
- プロジェクトで実際に使用するパターンを追加
-->

## 📚 目次

1. [コンポーネント](#コンポーネント)
2. [カスタムフック](#カスタムフック)
3. [テスト](#テスト)
4. [APIモック](#apiモック)
5. [フォーム](#フォーム)

---

## コンポーネント

### 基本的なコンポーネント

```tsx
// src/components/ui/Button.tsx
import { cva, type VariantProps } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700',
        secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
        outline: 'border border-gray-300 hover:bg-gray-50',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4 text-base',
        lg: 'h-12 px-6 text-lg',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean
}

/**
 * 汎用ボタンコンポーネント
 */
export function Button({
  variant,
  size,
  isLoading,
  disabled,
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      className={buttonVariants({ variant, size })}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading ? <Spinner /> : children}
    </button>
  )
}
```

### Container/Presenterパターン

```tsx
// src/components/features/UserProfile/UserProfile.types.ts
export interface UserProfilePresenterProps {
  user: User | null
  isLoading: boolean
  error: string | null
  onEdit: () => void
}

// src/components/features/UserProfile/UserProfile.presenter.tsx
export function UserProfilePresenter({
  user,
  isLoading,
  error,
  onEdit,
}: UserProfilePresenterProps) {
  if (isLoading) return <Skeleton />
  if (error) return <ErrorMessage message={error} />
  if (!user) return <EmptyState message="ユーザーが見つかりません" />

  return (
    <Card>
      <CardHeader>
        <Avatar src={user.avatarUrl} alt={user.name} />
        <h2>{user.name}</h2>
      </CardHeader>
      <CardContent>
        <p>{user.email}</p>
        <Button onClick={onEdit}>編集</Button>
      </CardContent>
    </Card>
  )
}

// src/components/features/UserProfile/UserProfile.container.tsx
export function UserProfileContainer({ userId }: { userId: string }) {
  const { user, isLoading, error } = useUser(userId)
  const router = useRouter()

  const handleEdit = useCallback(() => {
    router.push(`/users/${userId}/edit`)
  }, [router, userId])

  return (
    <UserProfilePresenter
      user={user}
      isLoading={isLoading}
      error={error}
      onEdit={handleEdit}
    />
  )
}
```

---

## カスタムフック

### データ取得フック

```tsx
// src/hooks/use-user.ts
import { useQuery } from '@tanstack/react-query'

/**
 * ユーザー情報を取得するフック
 */
export function useUser(userId: string) {
  const query = useQuery({
    queryKey: ['user', userId],
    queryFn: async () => {
      const response = await fetch(`/api/users/${userId}`)
      if (!response.ok) throw new Error('Failed to fetch user')
      return response.json() as Promise<User>
    },
    enabled: !!userId,
  })

  return {
    user: query.data ?? null,
    isLoading: query.isLoading,
    error: query.error?.message ?? null,
    refetch: query.refetch,
  }
}
```

### ローカルストレージフック

```tsx
// src/hooks/use-local-storage.ts
import { useState, useEffect } from 'react'

/**
 * ローカルストレージと同期するフック
 */
export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue

    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch {
      return initialValue
    }
  })

  const setValue = (value: T | ((val: T) => T)) => {
    const valueToStore = value instanceof Function ? value(storedValue) : value
    setStoredValue(valueToStore)
    window.localStorage.setItem(key, JSON.stringify(valueToStore))
  }

  return [storedValue, setValue] as const
}
```

---

## テスト

### コンポーネントテスト

```tsx
// src/components/ui/__tests__/Button.spec.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Button } from '../Button'

describe('Button', () => {
  describe('レンダリング', () => {
    it('テキストを表示する', () => {
      render(<Button>Click me</Button>)
      expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument()
    })
  })

  describe('インタラクション', () => {
    it('クリックイベントを発火する', async () => {
      const handleClick = vi.fn()
      render(<Button onClick={handleClick}>Click me</Button>)

      await userEvent.click(screen.getByRole('button'))

      expect(handleClick).toHaveBeenCalledTimes(1)
    })

    it('disabled時はクリックできない', async () => {
      const handleClick = vi.fn()
      render(<Button onClick={handleClick} disabled>Click me</Button>)

      await userEvent.click(screen.getByRole('button'))

      expect(handleClick).not.toHaveBeenCalled()
    })
  })

  describe('ローディング状態', () => {
    it('ローディング中はスピナーを表示する', () => {
      render(<Button isLoading>Click me</Button>)
      expect(screen.getByRole('button')).toBeDisabled()
    })
  })
})
```

### フックテスト

```tsx
// src/hooks/__tests__/use-user.spec.tsx
import { renderHook, waitFor } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useUser } from '../use-user'

const wrapper = ({ children }: { children: React.ReactNode }) => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  })
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

describe('useUser', () => {
  it('ユーザーデータを取得する', async () => {
    const { result } = renderHook(() => useUser('user-1'), { wrapper })

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false)
    })

    expect(result.current.user).toEqual({
      id: 'user-1',
      name: 'Test User',
      email: 'test@example.com',
    })
  })
})
```

---

## APIモック

### MSW ハンドラー

```tsx
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw'

export const handlers = [
  // ユーザー取得
  http.get('/api/users/:id', ({ params }) => {
    const { id } = params
    return HttpResponse.json({
      id,
      name: 'Test User',
      email: 'test@example.com',
    })
  }),

  // ユーザー一覧
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: '1', name: 'User 1', email: 'user1@example.com' },
      { id: '2', name: 'User 2', email: 'user2@example.com' },
    ])
  }),

  // ユーザー作成
  http.post('/api/users', async ({ request }) => {
    const body = await request.json()
    return HttpResponse.json(
      { id: 'new-id', ...body },
      { status: 201 }
    )
  }),

  // エラーケース
  http.get('/api/users/error', () => {
    return HttpResponse.json(
      { error: 'User not found' },
      { status: 404 }
    )
  }),
]
```

---

## フォーム

### React Hook Form + Zod

```tsx
// src/components/features/LoginForm/LoginForm.tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const loginSchema = z.object({
  email: z.string().email('有効なメールアドレスを入力してください'),
  password: z.string().min(8, 'パスワードは8文字以上必要です'),
})

type LoginFormData = z.infer<typeof loginSchema>

export function LoginForm({ onSubmit }: { onSubmit: (data: LoginFormData) => void }) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  })

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label htmlFor="email">メールアドレス</label>
        <input
          id="email"
          type="email"
          {...register('email')}
          className="w-full border rounded px-3 py-2"
        />
        {errors.email && (
          <p className="text-red-500 text-sm">{errors.email.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="password">パスワード</label>
        <input
          id="password"
          type="password"
          {...register('password')}
          className="w-full border rounded px-3 py-2"
        />
        {errors.password && (
          <p className="text-red-500 text-sm">{errors.password.message}</p>
        )}
      </div>

      <Button type="submit" isLoading={isSubmitting}>
        ログイン
      </Button>
    </form>
  )
}
```

---

## 関連ドキュメント

- [../guides/30-implementation-patterns.md](../guides/30-implementation-patterns.md) - 実装パターン
- [../guides/80-testing.md](../guides/80-testing.md) - テスト戦略
- [../guides/50-coding-standards.md](../guides/50-coding-standards.md) - コーディング規約

---

**最終更新**: YYYY-MM-DD

