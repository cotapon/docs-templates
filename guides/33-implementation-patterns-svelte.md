# 実装パターン - Svelte / SvelteKit

{{PROJECT_NAME}} での Svelte / SvelteKit 固有の実装パターンを説明します。

<!--
📝 このファイルは Svelte / SvelteKit プロジェクト用です。
概念説明は 30-implementation-patterns.md を参照してください。

Note: Svelte 5 の Runes ($state, $derived 等) を使用した例を含みます。
-->

## 📚 目次

1. [Container/Presenter パターン](#containerpresenter-パターン)
2. [Stores / Runes パターン](#stores--runes-パターン)
3. [Atomic Design 実装例](#atomic-design-実装例)

---

## Container/Presenter パターン

Svelte ではコンポーネント自体がシンプルなため、必要な場合のみ分離します。

### 型定義

```typescript
// types.ts
export interface UserProfilePresenterProps {
  user: User | null
  isLoading: boolean
  error: string | null
}
```

### Presenter（表示層）

```svelte
<!-- Presenter.svelte -->
<script lang="ts">
  import Skeleton from '@/components/atoms/Skeleton.svelte'
  import ErrorMessage from '@/components/atoms/ErrorMessage.svelte'
  import EmptyState from '@/components/atoms/EmptyState.svelte'
  import Card from '@/components/atoms/Card.svelte'
  import Button from '@/components/atoms/Button.svelte'
  import type { UserProfilePresenterProps } from './types'

  // Svelte 5: $props()
  let {
    user,
    isLoading,
    error,
    onEdit,
  }: UserProfilePresenterProps & { onEdit: () => void } = $props()
</script>

{#if isLoading}
  <Skeleton />
{:else if error}
  <ErrorMessage message={error} />
{:else if !user}
  <EmptyState />
{:else}
  <Card>
    <svelte:fragment slot="header">
      <h2>{user.name}</h2>
    </svelte:fragment>
    <p>{user.email}</p>
    <Button onclick={onEdit}>編集</Button>
  </Card>
{/if}
```

### Container（ロジック層）

```svelte
<!-- Container.svelte -->
<script lang="ts">
  import { goto } from '$app/navigation'
  import { userStore } from '@/stores/user'
  import UserProfilePresenter from './Presenter.svelte'

  // Svelte 5: $props()
  let { userId }: { userId: string } = $props()

  // Store から派生状態を作成
  let user = $derived($userStore.users.get(userId) ?? null)
  let isLoading = $derived($userStore.isLoading)
  let error = $derived($userStore.error)

  // 初回ロード
  $effect(() => {
    userStore.fetchUser(userId)
  })

  function handleEdit() {
    goto(`/users/${userId}/edit`)
  }
</script>

<UserProfilePresenter {user} {isLoading} {error} onEdit={handleEdit} />
```

### エクスポート

```typescript
// index.ts
export { default as UserProfile } from './Container.svelte'
export type { UserProfilePresenterProps } from './types'
```

---

## Stores / Runes パターン

Svelte 5 では Runes を使用した状態管理が推奨されます。

### Svelte 5 Runes を使用したパターン

```typescript
// src/stores/user.svelte.ts
import type { User } from '@/types'

/**
 * ユーザー情報を管理する Store
 */
function createUserStore() {
  let users = $state<Map<string, User>>(new Map())
  let isLoading = $state(false)
  let error = $state<string | null>(null)

  async function fetchUser(userId: string) {
    if (users.has(userId)) return

    try {
      isLoading = true
      error = null
      const response = await fetch(`/api/users/${userId}`)
      if (!response.ok) throw new Error('ユーザーの取得に失敗しました')
      const user: User = await response.json()
      users.set(userId, user)
    } catch (e) {
      error = e instanceof Error ? e.message : 'エラーが発生しました'
    } finally {
      isLoading = false
    }
  }

  async function updateUser(userId: string, data: Partial<User>) {
    try {
      isLoading = true
      error = null
      const response = await fetch(`/api/users/${userId}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      })
      if (!response.ok) throw new Error('更新に失敗しました')
      const user: User = await response.json()
      users.set(userId, user)
    } catch (e) {
      error = e instanceof Error ? e.message : 'エラーが発生しました'
    } finally {
      isLoading = false
    }
  }

  return {
    get users() { return users },
    get isLoading() { return isLoading },
    get error() { return error },
    fetchUser,
    updateUser,
  }
}

export const userStore = createUserStore()
```

### 従来の Writable Store パターン

```typescript
// src/stores/user.ts
import { writable, derived, type Readable } from 'svelte/store'
import type { User } from '@/types'

interface UserState {
  users: Map<string, User>
  isLoading: boolean
  error: string | null
}

function createUserStore() {
  const { subscribe, update } = writable<UserState>({
    users: new Map(),
    isLoading: false,
    error: null,
  })

  async function fetchUser(userId: string) {
    update(state => ({ ...state, isLoading: true, error: null }))

    try {
      const response = await fetch(`/api/users/${userId}`)
      if (!response.ok) throw new Error('ユーザーの取得に失敗しました')
      const user: User = await response.json()
      update(state => {
        state.users.set(userId, user)
        return { ...state, isLoading: false }
      })
    } catch (e) {
      update(state => ({
        ...state,
        isLoading: false,
        error: e instanceof Error ? e.message : 'エラーが発生しました',
      }))
    }
  }

  return {
    subscribe,
    fetchUser,
  }
}

export const userStore = createUserStore()
```

### SvelteKit の load 関数を使用したパターン

```typescript
// src/routes/users/[id]/+page.ts
import type { PageLoad } from './$types'
import type { User } from '@/types'

export const load: PageLoad = async ({ params, fetch }) => {
  const response = await fetch(`/api/users/${params.id}`)

  if (!response.ok) {
    return {
      user: null,
      error: 'ユーザーの取得に失敗しました',
    }
  }

  const user: User = await response.json()
  return { user, error: null }
}
```

```svelte
<!-- src/routes/users/[id]/+page.svelte -->
<script lang="ts">
  import type { PageData } from './$types'
  import { UserProfile } from '@/components/features/UserProfile'

  let { data }: { data: PageData } = $props()
</script>

{#if data.error}
  <p class="text-red-500">{data.error}</p>
{:else if data.user}
  <UserProfile user={data.user} />
{/if}
```

---

## Atomic Design 実装例

### Atoms: Button

```svelte
<!-- src/components/atoms/Button/Button.svelte -->
<script lang="ts">
  import type { Snippet } from 'svelte'

  interface Props {
    variant?: 'primary' | 'secondary' | 'ghost'
    size?: 'sm' | 'md' | 'lg'
    disabled?: boolean
    type?: 'button' | 'submit' | 'reset'
    onclick?: () => void
    children: Snippet
  }

  let {
    variant = 'primary',
    size = 'md',
    disabled = false,
    type = 'button',
    onclick,
    children,
  }: Props = $props()

  const baseClasses = 'inline-flex items-center justify-center rounded-md font-medium transition-colors'

  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
    ghost: 'hover:bg-gray-100',
  }

  const sizeClasses = {
    sm: 'h-8 px-3 text-sm',
    md: 'h-10 px-4',
    lg: 'h-12 px-6 text-lg',
  }
</script>

<button
  {type}
  {disabled}
  {onclick}
  class="{baseClasses} {variantClasses[variant]} {sizeClasses[size]}"
  class:opacity-50={disabled}
  class:cursor-not-allowed={disabled}
>
  {@render children()}
</button>
```

### Molecules: SearchForm

```svelte
<!-- src/components/molecules/SearchForm/SearchForm.svelte -->
<script lang="ts">
  import Button from '@/components/atoms/Button/Button.svelte'
  import Input from '@/components/atoms/Input/Input.svelte'

  interface Props {
    placeholder?: string
    isLoading?: boolean
    onSearch: (query: string) => void
  }

  let {
    placeholder = '検索...',
    isLoading = false,
    onSearch,
  }: Props = $props()

  let query = $state('')

  function handleSubmit(e: SubmitEvent) {
    e.preventDefault()
    if (query.trim()) {
      onSearch(query.trim())
    }
  }
</script>

<form onsubmit={handleSubmit} class="flex gap-2">
  <Input
    bind:value={query}
    {placeholder}
    disabled={isLoading}
  />
  <Button type="submit" disabled={isLoading}>
    {isLoading ? '検索中...' : '検索'}
  </Button>
</form>
```

### Organisms: Header

```svelte
<!-- src/components/organisms/Header/Header.svelte -->
<script lang="ts">
  import Logo from '@/components/atoms/Logo/Logo.svelte'
  import NavItem from '@/components/molecules/NavItem/NavItem.svelte'
  import SearchForm from '@/components/molecules/SearchForm/SearchForm.svelte'
  import UserMenu from '@/components/molecules/UserMenu/UserMenu.svelte'
  import type { User } from '@/types'

  interface Props {
    user: User | null
    onSearch: (query: string) => void
  }

  let { user, onSearch }: Props = $props()
</script>

<header class="flex items-center justify-between px-4 py-3 border-b">
  <div class="flex items-center gap-8">
    <Logo />
    <nav class="flex gap-4">
      <NavItem href="/dashboard">ダッシュボード</NavItem>
      <NavItem href="/projects">プロジェクト</NavItem>
      <NavItem href="/settings">設定</NavItem>
    </nav>
  </div>
  <div class="flex items-center gap-4">
    <SearchForm {onSearch} />
    <UserMenu {user} />
  </div>
</header>
```

### Templates: DashboardLayout

```svelte
<!-- src/components/templates/DashboardLayout/DashboardLayout.svelte -->
<script lang="ts">
  import { goto } from '$app/navigation'
  import Header from '@/components/organisms/Header/Header.svelte'
  import Sidebar from '@/components/organisms/Sidebar/Sidebar.svelte'
  import { authStore } from '@/stores/auth'
  import type { Snippet } from 'svelte'

  interface Props {
    children: Snippet
  }

  let { children }: Props = $props()

  let user = $derived($authStore.user)

  function handleSearch(query: string) {
    goto(`/search?q=${encodeURIComponent(query)}`)
  }
</script>

<div class="min-h-screen flex flex-col">
  <Header {user} onSearch={handleSearch} />
  <div class="flex flex-1">
    <Sidebar />
    <main class="flex-1 p-6">
      {@render children()}
    </main>
  </div>
</div>
```

---

## 関連ドキュメント

- [30-implementation-patterns.md](./30-implementation-patterns.md) - パターン概要
- [80-testing.md](./80-testing.md) - テスト戦略

---

**最終更新**: YYYY-MM-DD
