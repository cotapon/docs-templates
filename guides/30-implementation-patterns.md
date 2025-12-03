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
    ├── index.ts                           # エクスポート
    ├── UserProfile.container.tsx          # ロジック層
    ├── UserProfile.container.spec.tsx     # Containerテスト
    ├── UserProfile.presenter.tsx          # 表示層
    ├── UserProfile.presenter.spec.tsx     # Presenterテスト
    └── UserProfile.types.ts               # 型定義
```

### 実装例

#### 型定義

```typescript
// UserProfile.types.ts
export interface UserProfilePresenterProps {
  user: User | null
  isLoading: boolean
  error: string | null
  onEdit: () => void
}
```

#### Presenter（表示層）

```typescript
// UserProfile.presenter.tsx
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
// UserProfile.container.tsx
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

## 関連ドキュメント

- [20-clean-architecture.md](./20-clean-architecture.md) - レイヤー設計
- [40-domain-modeling.md](./40-domain-modeling.md) - ドメインモデリング
- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [80-testing.md](./80-testing.md) - テスト戦略

---

**最終更新**: YYYY-MM-DD

