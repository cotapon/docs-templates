# ドメインモデリングガイド

{{PROJECT_NAME}} のドメインモデル設計ガイドです。

<!-- 
📝 書くべき内容:
- ドメインモデルの設計手順
- エンティティと値オブジェクトの区別
- 集約の設計方針
- ドメインサービスの使い方
- 命名規則

⚠️ このファイルはDDD（ドメイン駆動設計）を採用する場合に有用です
シンプルなプロジェクトでは削除または簡略化してください
-->

## 📚 目次

1. [モデリング手順](#モデリング手順)
2. [エンティティ](#エンティティ)
3. [値オブジェクト](#値オブジェクト)
4. [集約](#集約)
5. [ドメインサービス](#ドメインサービス)

---

## モデリング手順

### 1. 要件からドメインを抽出

1. **ユースケースを確認**: [specs/requirements.md](../specs/requirements.md)
2. **名詞を抽出**: エンティティ候補
3. **動詞を抽出**: ドメインサービス・メソッド候補
4. **属性を整理**: プロパティ候補

### 2. 境界づけられたコンテキストを特定

<!-- プロジェクトのコンテキスト境界を記載 -->

例:
- **ユーザー管理コンテキスト**: User, Profile, Authentication
- **プロジェクト管理コンテキスト**: Project, Task, Member

### 3. ユビキタス言語を定義

| 用語 | 定義 | コード上の表現 |
|-----|------|---------------|
| <!-- 用語 --> | <!-- 定義 --> | <!-- 型名など --> |

---

## エンティティ

**定義**: 一意の識別子を持ち、ライフサイクルを通じて同一性を保つオブジェクト

### 特徴

- 一意のIDを持つ
- 属性が変わっても同一性を維持
- ライフサイクルがある（作成→更新→削除）

### 実装例

```typescript
// src/core/domain/entities/user.ts

/**
 * ユーザーエンティティ
 * @remarks 一意のIDで識別される
 */
export interface User {
  /** 一意の識別子 */
  id: string
  /** メールアドレス */
  email: string
  /** 表示名 */
  name: string
  /** 作成日時 */
  createdAt: Date
  /** 更新日時 */
  updatedAt: Date
}

/**
 * ユーザーを作成する
 */
export function createUser(params: CreateUserParams): User {
  return {
    id: generateId(),
    email: params.email,
    name: params.name,
    createdAt: new Date(),
    updatedAt: new Date(),
  }
}
```

---

## 値オブジェクト

**定義**: 属性の値によって等価性が決まるオブジェクト。不変。

### 特徴

- IDを持たない
- 属性の値で等価性を判断
- 不変（Immutable）
- 交換可能

### 実装例

```typescript
// src/core/domain/value-objects/email.ts

/**
 * メールアドレス値オブジェクト
 * @remarks バリデーション済みのメールアドレスを表現
 */
export class Email {
  private constructor(private readonly value: string) {}

  /**
   * メールアドレスを作成
   * @throws バリデーションエラー
   */
  static create(value: string): Email {
    if (!this.isValid(value)) {
      throw new Error('無効なメールアドレスです')
    }
    return new Email(value)
  }

  private static isValid(value: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
  }

  toString(): string {
    return this.value
  }

  equals(other: Email): boolean {
    return this.value === other.value
  }
}
```

---

## 集約

**定義**: 関連するエンティティと値オブジェクトのまとまり。集約ルートを通じてのみアクセス。

### 設計原則

1. **集約ルート**: 外部からのアクセスポイント
2. **トランザクション境界**: 集約単位で整合性を保証
3. **小さく保つ**: 1集約 = 1トランザクション

### 例

```
[Order Aggregate]
├── Order (集約ルート)
├── OrderItem (エンティティ)
└── Money (値オブジェクト)
```

---

## ドメインサービス

**定義**: エンティティや値オブジェクトに属さないドメインロジック

### 使用するケース

- 複数のエンティティにまたがる操作
- 外部サービスとの連携が必要な計算
- エンティティに持たせると不自然な操作

### 実装例

```typescript
// src/core/domain/services/permission-service.ts

/**
 * 権限チェックドメインサービス
 */
export class PermissionService {
  /**
   * ユーザーがリソースにアクセス可能か判定
   */
  canAccess(user: User, resource: Resource): boolean {
    // 複数エンティティにまたがるロジック
    return user.roles.some(role => 
      resource.allowedRoles.includes(role)
    )
  }
}
```

---

## 命名規則

| 種類 | 規則 | 例 |
|-----|------|-----|
| エンティティ | 名詞（単数形） | `User`, `Order`, `Project` |
| 値オブジェクト | 名詞（概念を表す） | `Email`, `Money`, `Address` |
| ドメインサービス | 名詞 + Service | `PermissionService` |
| リポジトリ | I + 名詞 + Repository | `IUserRepository` |

---

## 関連ドキュメント

- [20-clean-architecture.md](./20-clean-architecture.md) - レイヤー設計
- [30-implementation-patterns.md](./30-implementation-patterns.md) - 実装パターン
- [../specs/requirements.md](../specs/requirements.md) - 要件定義
- [../specs/database.md](../specs/database.md) - DB設計

---

**最終更新**: YYYY-MM-DD

