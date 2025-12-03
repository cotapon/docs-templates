# 実装パターン - React / Next.js

{{PROJECT_NAME}} での React / Next.js 固有の実装パターンを説明します。

<!--
📝 このファイルは React / Next.js プロジェクト用です。
概念説明は 30-implementation-patterns.md を参照してください。
-->

## 📚 目次

1. [Container/Presenter パターン](#containerpresenter-パターン)
2. [Custom Hooks パターン](#custom-hooks-パターン)
3. [Atomic Design 実装例](#atomic-design-実装例)

---

## Container/Presenter パターン

### 型定義

```typescript
// types.ts
export interface UserProfilePresenterProps {
  user: User | null
  isLoading: boolean
  error: string | null
  onEdit: () => void
}
```

### Presenter（表示層）

```tsx
// Presenter.tsx
import { Skeleton } from '@/components/atoms/Skeleton'
import { ErrorMessage } from '@/components/atoms/ErrorMessage'
import { EmptyState } from '@/components/atoms/EmptyState'
import { Card, CardHeader, CardContent } from '@/components/atoms/Card'
import { Button } from '@/components/atoms/Button'
import type { UserProfilePresenterProps } from './types'

/**
 * ユーザープロフィールの表示コンポーネント
 */
export function UserProfilePresenter({
  user,
  isLoading,
  error,
  onEdit,
}: UserProfilePresenterProps) {
  if (isLoading) return <Skeleton />
  if (error) return <ErrorMessage message={error} />
  if (!user) return <EmptyState />

  return (
    <Card>
      <CardHeader>
        <h2>{user.name}</h2>
      </CardHeader>
      <CardContent>
        <p>{user.email}</p>
        <Button onClick={onEdit}>編集</Button>
      </CardContent>
    </Card>
  )
}
```

### Container（ロジック層）

```tsx
// Container.tsx
import { useCallback } from 'react'
import { useRouter } from 'next/navigation'
import { useUser } from '@/hooks/useUser'
import { UserProfilePresenter } from './Presenter'

interface UserProfileContainerProps {
  userId: string
}

/**
 * ユーザープロフィールのロジックコンポーネント
 */
export function UserProfileContainer({ userId }: UserProfileContainerProps) {
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

### エクスポート

```typescript
// index.ts
export { UserProfileContainer as UserProfile } from './Container'
export type { UserProfilePresenterProps } from './types'
```

---

## Custom Hooks パターン

### 基本的なデータ取得フック

```typescript
// src/hooks/useUser.ts
import { useState, useEffect } from 'react'
import type { User } from '@/types'

interface UseUserReturn {
  user: User | null
  isLoading: boolean
  error: string | null
  refetch: () => void
}

/**
 * ユーザー情報を取得するフック
 * @param userId - ユーザーID
 */
export function useUser(userId: string): UseUserReturn {
  const [user, setUser] = useState<User | null>(null)
  const [isLoading, setIsLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  const fetchUser = useCallback(async () => {
    try {
      setIsLoading(true)
      setError(null)
      const response = await fetch(`/api/users/${userId}`)
      if (!response.ok) throw new Error('ユーザーの取得に失敗しました')
      const data = await response.json()
      setUser(data)
    } catch (e) {
      setError(e instanceof Error ? e.message : 'エラーが発生しました')
    } finally {
      setIsLoading(false)
    }
  }, [userId])

  useEffect(() => {
    fetchUser()
  }, [fetchUser])

  return { user, isLoading, error, refetch: fetchUser }
}
```

### TanStack Query を使用した場合

```typescript
// src/hooks/useUser.ts
import { useQuery } from '@tanstack/react-query'
import type { User } from '@/types'

/**
 * ユーザー情報を取得するフック（TanStack Query版）
 */
export function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: async (): Promise<User> => {
      const response = await fetch(`/api/users/${userId}`)
      if (!response.ok) throw new Error('ユーザーの取得に失敗しました')
      return response.json()
    },
  })
}
```

### ミューテーションフック

```typescript
// src/hooks/useUpdateUser.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'
import type { User, UpdateUserInput } from '@/types'

export function useUpdateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async ({ id, data }: { id: string; data: UpdateUserInput }): Promise<User> => {
      const response = await fetch(`/api/users/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      })
      if (!response.ok) throw new Error('更新に失敗しました')
      return response.json()
    },
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['user', data.id] })
    },
  })
}
```

---

## Atomic Design 実装例

### Atoms: Button

```tsx
// src/components/atoms/Button/Button.tsx
import { forwardRef } from 'react'
import { cva, type VariantProps } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700',
        secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
        ghost: 'hover:bg-gray-100',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4',
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
    VariantProps<typeof buttonVariants> {}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={buttonVariants({ variant, size, className })}
        {...props}
      />
    )
  }
)
Button.displayName = 'Button'
```

### Molecules: SearchForm

```tsx
// src/components/molecules/SearchForm/SearchForm.tsx
import { useState, type FormEvent } from 'react'
import { Button } from '@/components/atoms/Button'
import { Input } from '@/components/atoms/Input'

interface SearchFormProps {
  onSearch: (query: string) => void
  placeholder?: string
  isLoading?: boolean
}

export function SearchForm({
  onSearch,
  placeholder = '検索...',
  isLoading = false,
}: SearchFormProps) {
  const [query, setQuery] = useState('')

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault()
    if (query.trim()) {
      onSearch(query.trim())
    }
  }

  return (
    <form onSubmit={handleSubmit} className="flex gap-2">
      <Input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder={placeholder}
        disabled={isLoading}
      />
      <Button type="submit" disabled={isLoading}>
        {isLoading ? '検索中...' : '検索'}
      </Button>
    </form>
  )
}
```

### Organisms: Header

```tsx
// src/components/organisms/Header/Header.tsx
import { Logo } from '@/components/atoms/Logo'
import { NavItem } from '@/components/molecules/NavItem'
import { SearchForm } from '@/components/molecules/SearchForm'
import { UserMenu } from '@/components/molecules/UserMenu'
import type { User } from '@/types'

interface HeaderProps {
  user: User | null
  onSearch: (query: string) => void
}

export function Header({ user, onSearch }: HeaderProps) {
  return (
    <header className="flex items-center justify-between px-4 py-3 border-b">
      <div className="flex items-center gap-8">
        <Logo />
        <nav className="flex gap-4">
          <NavItem href="/dashboard">ダッシュボード</NavItem>
          <NavItem href="/projects">プロジェクト</NavItem>
          <NavItem href="/settings">設定</NavItem>
        </nav>
      </div>
      <div className="flex items-center gap-4">
        <SearchForm onSearch={onSearch} />
        <UserMenu user={user} />
      </div>
    </header>
  )
}
```

### Templates: DashboardLayout

```tsx
// src/components/templates/DashboardLayout/DashboardLayout.tsx
'use client'

import { useRouter } from 'next/navigation'
import { Header } from '@/components/organisms/Header'
import { Sidebar } from '@/components/organisms/Sidebar'
import { useAuth } from '@/hooks/useAuth'

interface DashboardLayoutProps {
  children: React.ReactNode
}

export function DashboardLayout({ children }: DashboardLayoutProps) {
  const { user } = useAuth()
  const router = useRouter()

  const handleSearch = (query: string) => {
    router.push(`/search?q=${encodeURIComponent(query)}`)
  }

  return (
    <div className="min-h-screen flex flex-col">
      <Header user={user} onSearch={handleSearch} />
      <div className="flex flex-1">
        <Sidebar />
        <main className="flex-1 p-6">{children}</main>
      </div>
    </div>
  )
}
```

---

## 関連ドキュメント

- [30-implementation-patterns.md](./30-implementation-patterns.md) - パターン概要
- [80-testing.md](./80-testing.md) - テスト戦略

---

**最終更新**: YYYY-MM-DD
