# クリーンアーキテクチャガイド

{{PROJECT_NAME}} におけるクリーンアーキテクチャの実装指針を説明します。

<!-- 
📝 書くべき内容:
- レイヤー構成と各層の責務
- 依存関係のルール（内側への一方向依存）
- 各層に配置するコード例
- テスト戦略

⚠️ このファイルはクリーンアーキテクチャを採用する場合のみ必要です
シンプルなプロジェクトでは削除してください
-->

## 📚 目次

1. [レイヤー構成](#レイヤー構成)
2. [依存関係ルール](#依存関係ルール)
3. [各層の詳細](#各層の詳細)
4. [実装例](#実装例)

---

## レイヤー構成

```
┌─────────────────────────────────────────────────────────┐
│                 Presentation Layer                       │
│              (UI Components, Pages)                      │
├─────────────────────────────────────────────────────────┤
│                 Application Layer                        │
│                   (Use Cases)                            │
├─────────────────────────────────────────────────────────┤
│                   Domain Layer                           │
│         (Entities, Value Objects, Domain Services)       │
├─────────────────────────────────────────────────────────┤
│                Infrastructure Layer                      │
│            (Repositories, External APIs)                 │
└─────────────────────────────────────────────────────────┘
```

### ディレクトリマッピング

```
src/
├── core/
│   ├── domain/           # Domain Layer
│   │   ├── entities/     # エンティティ
│   │   ├── value-objects/# 値オブジェクト
│   │   └── services/     # ドメインサービス
│   ├── application/      # Application Layer
│   │   └── use-cases/    # ユースケース
│   └── infrastructure/   # Infrastructure Layer
│       ├── repositories/ # リポジトリ実装
│       └── api/          # 外部API連携
├── components/           # Presentation Layer
└── app/                  # Presentation Layer (Pages)
```

---

## 依存関係ルール

### 基本原則

**依存は常に内側（Domain）に向かう**

```
Presentation → Application → Domain ← Infrastructure
```

### 許可される依存

| 層 | 依存可能な層 |
|----|-------------|
| Presentation | Application, Domain |
| Application | Domain |
| Infrastructure | Domain |
| Domain | なし（純粋） |

### 禁止される依存

- ❌ Domain → Application
- ❌ Domain → Infrastructure
- ❌ Domain → Presentation
- ❌ Application → Infrastructure（直接参照）

---

## 各層の詳細

### 1. Domain Layer（ドメイン層）

**責務**: ビジネスルールの中核。外部依存なし。

**含まれるもの**:
- エンティティ（Entity）
- 値オブジェクト（Value Object）
- ドメインサービス
- リポジトリインターフェース

```typescript
// src/core/domain/entities/user.ts
export interface User {
  id: string
  email: string
  name: string
  createdAt: Date
}

// src/core/domain/repositories/user-repository.ts
export interface IUserRepository {
  findById(id: string): Promise<User | null>
  save(user: User): Promise<User>
}
```

### 2. Application Layer（アプリケーション層）

**責務**: ユースケースの実装。ドメインオブジェクトを操作。

**含まれるもの**:
- ユースケース
- DTOs（Data Transfer Objects）
- アプリケーションサービス

```typescript
// src/core/application/use-cases/get-user.ts
export class GetUserUseCase {
  constructor(private userRepository: IUserRepository) {}

  async execute(userId: string): Promise<User | null> {
    return this.userRepository.findById(userId)
  }
}
```

### 3. Infrastructure Layer（インフラストラクチャ層）

**責務**: 外部システムとの連携。リポジトリの実装。

**含まれるもの**:
- リポジトリ実装
- APIクライアント
- データベース接続

```typescript
// src/core/infrastructure/repositories/user-repository-impl.ts
export class UserRepositoryImpl implements IUserRepository {
  async findById(id: string): Promise<User | null> {
    // 実際のDB/API呼び出し
  }
}
```

### 4. Presentation Layer（プレゼンテーション層）

**責務**: UIの表示とユーザーインタラクション。

**含まれるもの**:
- Reactコンポーネント
- ページコンポーネント
- カスタムフック

---

## 実装例

### ユースケースの呼び出しフロー

```
[UI Component]
    ↓ calls
[Custom Hook (useUser)]
    ↓ calls
[UseCase (GetUserUseCase)]
    ↓ calls
[Repository Interface (IUserRepository)]
    ↓ implemented by
[Repository Implementation (UserRepositoryImpl)]
    ↓ calls
[External API / Database]
```

### フック経由での呼び出し例

```typescript
// src/hooks/use-user.ts
export function useUser(userId: string) {
  const useCase = useMemo(
    () => new GetUserUseCase(new UserRepositoryImpl()),
    []
  )

  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => useCase.execute(userId),
  })
}
```

---

## テスト戦略

### 各層のテスト方針

| 層 | テスト種類 | モック対象 |
|----|----------|-----------|
| Domain | ユニットテスト | なし（純粋関数） |
| Application | ユニットテスト | Repository |
| Infrastructure | 統合テスト | 外部API/DB |
| Presentation | コンポーネントテスト | UseCase/Hook |

---

## 関連ドキュメント

- [10-architecture.md](./10-architecture.md) - 技術アーキテクチャ
- [30-implementation-patterns.md](./30-implementation-patterns.md) - 実装パターン
- [40-domain-modeling.md](./40-domain-modeling.md) - ドメインモデリング
- [80-testing.md](./80-testing.md) - テスト戦略

---

**最終更新**: YYYY-MM-DD

