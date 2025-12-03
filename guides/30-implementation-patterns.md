# 実装パターンガイド

{{PROJECT_NAME}} で採用している実装パターンとテンプレートを説明します。

<!-- 
📝 書くべき内容:
- プロジェクトで採用している実装パターン
- 各パターンの目的と使い方
- コード例とディレクトリ構造
- テスト方針

例:
- Container/Presenter パターン
- Repository パターン
- Factory パターン
- Registry パターン
-->

## 📚 目次

1. [Container/Presenter パターン](#containerpresenter-パターン)
2. [Repository パターン](#repository-パターン)
3. [カスタムフックパターン](#カスタムフックパターン)
4. [Atomic Design パターン](#atomic-design-パターン)

---

## Container/Presenter パターン

UIコンポーネントをロジック（Container）と表示（Presenter）に分離するパターンです。

### 目的

- **関心の分離**: ビジネスロジックとUIを分離
- **テスタビリティ**: 各層を独立してテスト可能
- **再利用性**: Presenterは異なるデータソースで再利用可能

### ディレクトリ構造

```
src/components/features/
└── UserProfile/
    ├── index.ts              # エクスポート
    ├── Container.tsx         # ロジック層
    ├── Container.spec.tsx    # Containerテスト
    ├── Presenter.tsx         # 表示層
    ├── Presenter.spec.tsx    # Presenterテスト
    ├── Presenter.stories.tsx # Storybook（Presenterのみ）
    └── types.ts              # 型定義
```

> **Note**: Storybookは表示層（Presenter）のみ作成。Containerはロジック層のためStorybook不要。

### 実装例

#### 型定義

```typescript
// types.ts
export interface UserProfilePresenterProps {
  user: User | null
  isLoading: boolean
  error: string | null
  onEdit: () => void
}
```

#### Presenter（表示層）

```typescript
// Presenter.tsx
/**
 * ユーザープロフィールの表示コンポーネント
 * @param props - UserProfilePresenterProps
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

#### Container（ロジック層）

```typescript
// Container.tsx
/**
 * ユーザープロフィールのロジックコンポーネント
 * データ取得とイベントハンドリングを担当
 */
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

### テスト方針

| 対象 | テスト内容 |
|------|----------|
| Presenter | UI表示、各状態（loading/error/empty）の確認 |
| Container | データ取得、イベントハンドラーの呼び出し |

---

## Repository パターン

データアクセスを抽象化し、ビジネスロジックからデータソースを隠蔽するパターンです。

### 目的

- **データソースの抽象化**: API/DB切り替えが容易
- **テスタビリティ**: モックに置き換え可能
- **一貫性**: データアクセスの方法を統一

### 実装例

#### インターフェース（Domain層）

```typescript
// src/core/domain/repositories/user-repository.ts
export interface IUserRepository {
  findById(id: string): Promise<User | null>
  findAll(): Promise<User[]>
  save(user: User): Promise<User>
  delete(id: string): Promise<void>
}
```

#### 実装（Infrastructure層）

```typescript
// src/core/infrastructure/repositories/user-repository-impl.ts
export class UserRepositoryImpl implements IUserRepository {
  constructor(private apiClient: ApiClient) {}

  async findById(id: string): Promise<User | null> {
    const response = await this.apiClient.get(`/users/${id}`)
    return response.data
  }

  // ... その他のメソッド
}
```

---

## カスタムフックパターン

Reactのカスタムフックを使ってロジックを再利用可能にするパターンです。

### 命名規則

- `use` プレフィックス必須
- 動詞 + 名詞: `useUserProfile`, `useFetchData`

### 実装例

```typescript
// src/hooks/use-user.ts
/**
 * ユーザー情報を取得するフック
 * @param userId - ユーザーID
 * @returns ユーザー情報、ローディング状態、エラー
 */
export function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null)
  const [isLoading, setIsLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setIsLoading(true)
        const data = await userRepository.findById(userId)
        setUser(data)
      } catch (e) {
        setError('ユーザーの取得に失敗しました')
      } finally {
        setIsLoading(false)
      }
    }

    fetchUser()
  }, [userId])

  return { user, isLoading, error }
}
```

---

## Atomic Design パターン

UIコンポーネントを5つの階層に分類し、再利用性と一貫性を高めるパターンです。

### 目的

- **再利用性**: 小さな部品から組み立てることで再利用を促進
- **一貫性**: デザインシステムとの整合性を保つ
- **スケーラビリティ**: 大規模なUIでも管理しやすい構造
- **チーム開発**: 役割分担が明確になる

### 5つの階層

```
Pages（ページ）
  ↑
Templates（テンプレート）
  ↑
Organisms（有機体）
  ↑
Molecules（分子）
  ↑
Atoms（原子）
```

| 階層 | 説明 | 例 |
|------|------|-----|
| **Atoms** | 最小単位のUI要素 | Button, Input, Label, Icon, Badge |
| **Molecules** | Atomsを組み合わせた小さな機能単位 | SearchForm, FormField, MenuItem |
| **Organisms** | Molecules/Atomsで構成される複雑なUI | Header, Sidebar, Card, DataTable |
| **Templates** | ページのレイアウト構造（データなし） | DashboardLayout, AuthLayout |
| **Pages** | Templatesに実データを流し込んだもの | DashboardPage, LoginPage |

### ディレクトリ構造

```
src/components/
├── atoms/
│   ├── Button/
│   │   ├── index.ts
│   │   ├── Button.tsx
│   │   ├── Button.spec.tsx
│   │   └── Button.stories.tsx
│   ├── Input/
│   ├── Label/
│   └── Icon/
├── molecules/
│   ├── SearchForm/
│   │   ├── index.ts
│   │   ├── SearchForm.tsx
│   │   ├── SearchForm.spec.tsx
│   │   └── SearchForm.stories.tsx
│   ├── FormField/
│   └── NavItem/
├── organisms/
│   ├── Header/
│   ├── Sidebar/
│   └── UserCard/
├── templates/
│   ├── DashboardLayout/
│   └── AuthLayout/
└── pages/
    ├── Dashboard/
    └── Login/
```

> **Note**: テスト（`.spec.tsx`）とStorybook（`.stories.tsx`）はコンポーネントと同一ディレクトリに配置（コロケーション）。

### 実装例

#### Atoms: Button

```typescript
// src/components/atoms/Button/Button.tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  children: React.ReactNode
  onClick?: () => void
  disabled?: boolean
}

export function Button({
  variant = 'primary',
  size = 'md',
  children,
  onClick,
  disabled,
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  )
}
```

#### Molecules: SearchForm

```typescript
// src/components/molecules/SearchForm/SearchForm.tsx
import { Button } from '@/components/atoms/Button'
import { Input } from '@/components/atoms/Input'

interface SearchFormProps {
  onSearch: (query: string) => void
  placeholder?: string
}

export function SearchForm({ onSearch, placeholder = '検索...' }: SearchFormProps) {
  const [query, setQuery] = useState('')

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    onSearch(query)
  }

  return (
    <form onSubmit={handleSubmit} className="search-form">
      <Input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder={placeholder}
      />
      <Button type="submit" variant="primary">
        検索
      </Button>
    </form>
  )
}
```

#### Organisms: Header

```typescript
// src/components/organisms/Header/Header.tsx
import { Logo } from '@/components/atoms/Logo'
import { NavItem } from '@/components/molecules/NavItem'
import { SearchForm } from '@/components/molecules/SearchForm'
import { UserMenu } from '@/components/molecules/UserMenu'

interface HeaderProps {
  user: User | null
  onSearch: (query: string) => void
}

export function Header({ user, onSearch }: HeaderProps) {
  return (
    <header className="header">
      <Logo />
      <nav className="header-nav">
        <NavItem href="/dashboard">ダッシュボード</NavItem>
        <NavItem href="/projects">プロジェクト</NavItem>
      </nav>
      <SearchForm onSearch={onSearch} />
      <UserMenu user={user} />
    </header>
  )
}
```

#### Templates: DashboardLayout

```typescript
// src/components/templates/DashboardLayout/DashboardLayout.tsx
import { Header } from '@/components/organisms/Header'
import { Sidebar } from '@/components/organisms/Sidebar'

interface DashboardLayoutProps {
  children: React.ReactNode
}

export function DashboardLayout({ children }: DashboardLayoutProps) {
  const { user } = useAuth()
  const handleSearch = useSearch()

  return (
    <div className="dashboard-layout">
      <Header user={user} onSearch={handleSearch} />
      <div className="dashboard-body">
        <Sidebar />
        <main className="dashboard-content">
          {children}
        </main>
      </div>
    </div>
  )
}
```

### 依存ルール

上位階層は下位階層のみに依存できます。

```
✅ 許可される依存
  - Molecules → Atoms
  - Organisms → Molecules, Atoms
  - Templates → Organisms, Molecules, Atoms
  - Pages → Templates, Organisms, Molecules, Atoms

❌ 禁止される依存
  - Atoms → Molecules（下位が上位に依存）
  - Molecules → Organisms
  - 同階層間の依存（Atom A → Atom B）※例外あり
```

### Container/Presenter との併用

Atomic Design と Container/Presenter パターンは併用可能です。

```
src/components/organisms/UserCard/
├── Container.tsx    # Container: データ取得
├── Presenter.tsx    # Presenter: 表示（Atoms/Moleculesを使用）
└── index.ts
```

---

## 関連ドキュメント

- [20-clean-architecture.md](./20-clean-architecture.md) - レイヤー設計
- [40-domain-modeling.md](./40-domain-modeling.md) - ドメインモデリング
- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [80-testing.md](./80-testing.md) - テスト戦略

---

**最終更新**: YYYY-MM-DD

