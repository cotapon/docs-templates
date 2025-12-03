# 実装パターン - Vue / Nuxt

{{PROJECT_NAME}} での Vue / Nuxt 固有の実装パターンを説明します。

<!--
📝 このファイルは Vue / Nuxt プロジェクト用です。
概念説明は 30-implementation-patterns.md を参照してください。
-->

## 📚 目次

1. [Container/Presenter パターン](#containerpresenter-パターン)
2. [Composables パターン](#composables-パターン)
3. [Atomic Design 実装例](#atomic-design-実装例)

---

## Container/Presenter パターン

Vue では Composition API を使用して Container/Presenter パターンを実現します。

### 型定義

```typescript
// types.ts
export interface UserProfilePresenterProps {
  user: User | null
  isLoading: boolean
  error: string | null
}

export interface UserProfileEmits {
  (e: 'edit'): void
}
```

### Presenter（表示層）

```vue
<!-- Presenter.vue -->
<script setup lang="ts">
import { Skeleton } from '@/components/atoms/Skeleton'
import { ErrorMessage } from '@/components/atoms/ErrorMessage'
import { EmptyState } from '@/components/atoms/EmptyState'
import { Card, CardHeader, CardContent } from '@/components/atoms/Card'
import { Button } from '@/components/atoms/Button'
import type { UserProfilePresenterProps, UserProfileEmits } from './types'

defineProps<UserProfilePresenterProps>()
const emit = defineEmits<UserProfileEmits>()
</script>

<template>
  <Skeleton v-if="isLoading" />
  <ErrorMessage v-else-if="error" :message="error" />
  <EmptyState v-else-if="!user" />
  <Card v-else>
    <CardHeader>
      <h2>{{ user.name }}</h2>
    </CardHeader>
    <CardContent>
      <p>{{ user.email }}</p>
      <Button @click="emit('edit')">編集</Button>
    </CardContent>
  </Card>
</template>
```

### Container（ロジック層）

```vue
<!-- Container.vue -->
<script setup lang="ts">
import { useRouter } from 'vue-router'
import { useUser } from '@/composables/useUser'
import UserProfilePresenter from './Presenter.vue'

const props = defineProps<{
  userId: string
}>()

const { user, isLoading, error } = useUser(() => props.userId)
const router = useRouter()

const handleEdit = () => {
  router.push(`/users/${props.userId}/edit`)
}
</script>

<template>
  <UserProfilePresenter
    :user="user"
    :is-loading="isLoading"
    :error="error"
    @edit="handleEdit"
  />
</template>
```

### エクスポート

```typescript
// index.ts
export { default as UserProfile } from './Container.vue'
export type { UserProfilePresenterProps } from './types'
```

---

## Composables パターン

Vue の Composition API を使ったロジック再利用パターンです。

### 基本的なデータ取得 Composable

```typescript
// src/composables/useUser.ts
import { ref, watch, type Ref } from 'vue'
import type { User } from '@/types'

interface UseUserReturn {
  user: Ref<User | null>
  isLoading: Ref<boolean>
  error: Ref<string | null>
  refetch: () => Promise<void>
}

/**
 * ユーザー情報を取得する Composable
 * @param userId - ユーザーID（ref または getter 関数）
 */
export function useUser(userId: Ref<string> | (() => string)): UseUserReturn {
  const user = ref<User | null>(null)
  const isLoading = ref(true)
  const error = ref<string | null>(null)

  const fetchUser = async () => {
    const id = typeof userId === 'function' ? userId() : userId.value
    try {
      isLoading.value = true
      error.value = null
      const response = await fetch(`/api/users/${id}`)
      if (!response.ok) throw new Error('ユーザーの取得に失敗しました')
      user.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'エラーが発生しました'
    } finally {
      isLoading.value = false
    }
  }

  // userId の変更を監視
  watch(
    typeof userId === 'function' ? userId : () => userId.value,
    () => fetchUser(),
    { immediate: true }
  )

  return { user, isLoading, error, refetch: fetchUser }
}
```

### VueQuery（TanStack Query）を使用した場合

```typescript
// src/composables/useUser.ts
import { useQuery } from '@tanstack/vue-query'
import type { User } from '@/types'

/**
 * ユーザー情報を取得する Composable（VueQuery版）
 */
export function useUser(userId: Ref<string> | (() => string)) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: async (): Promise<User> => {
      const id = typeof userId === 'function' ? userId() : userId.value
      const response = await fetch(`/api/users/${id}`)
      if (!response.ok) throw new Error('ユーザーの取得に失敗しました')
      return response.json()
    },
  })
}
```

### ミューテーション Composable

```typescript
// src/composables/useUpdateUser.ts
import { useMutation, useQueryClient } from '@tanstack/vue-query'
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

### 状態管理 Composable（Pinia）

```typescript
// src/stores/user.ts
import { defineStore } from 'pinia'
import type { User } from '@/types'

export const useUserStore = defineStore('user', () => {
  const currentUser = ref<User | null>(null)
  const isAuthenticated = computed(() => currentUser.value !== null)

  const setUser = (user: User | null) => {
    currentUser.value = user
  }

  const logout = () => {
    currentUser.value = null
  }

  return {
    currentUser,
    isAuthenticated,
    setUser,
    logout,
  }
})
```

---

## Atomic Design 実装例

### Atoms: Button

```vue
<!-- src/components/atoms/Button/Button.vue -->
<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
}

withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  disabled: false,
})
</script>

<template>
  <button
    :class="[
      'inline-flex items-center justify-center rounded-md font-medium transition-colors',
      {
        'bg-blue-600 text-white hover:bg-blue-700': variant === 'primary',
        'bg-gray-200 text-gray-900 hover:bg-gray-300': variant === 'secondary',
        'hover:bg-gray-100': variant === 'ghost',
        'h-8 px-3 text-sm': size === 'sm',
        'h-10 px-4': size === 'md',
        'h-12 px-6 text-lg': size === 'lg',
        'opacity-50 cursor-not-allowed': disabled,
      },
    ]"
    :disabled="disabled"
  >
    <slot />
  </button>
</template>
```

### Molecules: SearchForm

```vue
<!-- src/components/molecules/SearchForm/SearchForm.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { Button } from '@/components/atoms/Button'
import { Input } from '@/components/atoms/Input'

interface Props {
  placeholder?: string
  isLoading?: boolean
}

interface Emits {
  (e: 'search', query: string): void
}

withDefaults(defineProps<Props>(), {
  placeholder: '検索...',
  isLoading: false,
})

const emit = defineEmits<Emits>()
const query = ref('')

const handleSubmit = () => {
  if (query.value.trim()) {
    emit('search', query.value.trim())
  }
}
</script>

<template>
  <form @submit.prevent="handleSubmit" class="flex gap-2">
    <Input
      v-model="query"
      :placeholder="placeholder"
      :disabled="isLoading"
    />
    <Button type="submit" :disabled="isLoading">
      {{ isLoading ? '検索中...' : '検索' }}
    </Button>
  </form>
</template>
```

### Organisms: Header

```vue
<!-- src/components/organisms/Header/Header.vue -->
<script setup lang="ts">
import { Logo } from '@/components/atoms/Logo'
import { NavItem } from '@/components/molecules/NavItem'
import { SearchForm } from '@/components/molecules/SearchForm'
import { UserMenu } from '@/components/molecules/UserMenu'
import type { User } from '@/types'

interface Props {
  user: User | null
}

interface Emits {
  (e: 'search', query: string): void
}

defineProps<Props>()
defineEmits<Emits>()
</script>

<template>
  <header class="flex items-center justify-between px-4 py-3 border-b">
    <div class="flex items-center gap-8">
      <Logo />
      <nav class="flex gap-4">
        <NavItem to="/dashboard">ダッシュボード</NavItem>
        <NavItem to="/projects">プロジェクト</NavItem>
        <NavItem to="/settings">設定</NavItem>
      </nav>
    </div>
    <div class="flex items-center gap-4">
      <SearchForm @search="$emit('search', $event)" />
      <UserMenu :user="user" />
    </div>
  </header>
</template>
```

### Templates: DashboardLayout

```vue
<!-- src/components/templates/DashboardLayout/DashboardLayout.vue -->
<script setup lang="ts">
import { useRouter } from 'vue-router'
import { Header } from '@/components/organisms/Header'
import { Sidebar } from '@/components/organisms/Sidebar'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()
const router = useRouter()

const handleSearch = (query: string) => {
  router.push({ path: '/search', query: { q: query } })
}
</script>

<template>
  <div class="min-h-screen flex flex-col">
    <Header :user="userStore.currentUser" @search="handleSearch" />
    <div class="flex flex-1">
      <Sidebar />
      <main class="flex-1 p-6">
        <slot />
      </main>
    </div>
  </div>
</template>
```

---

## 関連ドキュメント

- [30-implementation-patterns.md](./30-implementation-patterns.md) - パターン概要
- [80-testing.md](./80-testing.md) - テスト戦略

---

**最終更新**: YYYY-MM-DD
