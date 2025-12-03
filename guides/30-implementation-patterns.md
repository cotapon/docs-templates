# 実装パターンガイド

{{PROJECT_NAME}} で採用している実装パターンとテンプレートを説明します。

<!--
📝 書くべき内容:
- プロジェクトで採用している実装パターン
- 各パターンの目的と使い方
- ディレクトリ構造

このファイルは概念説明のみ。
フレームワーク固有の実装例は以下を参照:
- 31-implementation-patterns-react.md (React/Next.js)
- 32-implementation-patterns-vue.md (Vue/Nuxt)
- 33-implementation-patterns-svelte.md (Svelte/SvelteKit)
-->

## 📚 目次

1. [Container/Presenter パターン](#containerpresenter-パターン)
2. [Repository パターン](#repository-パターン)
3. [ロジック再利用パターン](#ロジック再利用パターン)
4. [Atomic Design パターン](#atomic-design-パターン)
5. [フレームワーク別実装ガイド](#フレームワーク別実装ガイド)

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

### 責務の分離

| 層 | 責務 | 含むもの |
|---|------|---------|
| **Container** | ロジック | データ取得、状態管理、イベントハンドラー |
| **Presenter** | 表示 | UI描画、スタイル、ユーザー操作の受付 |

### テスト方針

| 対象 | テスト内容 |
|------|----------|
| Presenter | UI表示、各状態（loading/error/empty）の確認 |
| Container | データ取得、イベントハンドラーの呼び出し |

### フレームワーク別の実装

| フレームワーク | Container | Presenter |
|--------------|-----------|-----------|
| **React** | Custom Hooks + JSX | Function Component |
| **Vue** | Composition API (`<script setup>`) | Template |
| **Svelte** | Stores / Runes | .svelte Component |

---

## Repository パターン

データアクセスを抽象化し、ビジネスロジックからデータソースを隠蔽するパターンです。

### 目的

- **データソースの抽象化**: API/DB切り替えが容易
- **テスタビリティ**: モックに置き換え可能
- **一貫性**: データアクセスの方法を統一

### インターフェース設計

```typescript
// src/core/domain/repositories/user-repository.ts
export interface IUserRepository {
  findById(id: string): Promise<User | null>
  findAll(): Promise<User[]>
  save(user: User): Promise<User>
  delete(id: string): Promise<void>
}
```

### ディレクトリ構造

```
src/core/
├── domain/
│   └── repositories/
│       └── user-repository.ts      # インターフェース
└── infrastructure/
    └── repositories/
        └── user-repository-impl.ts # 実装
```

---

## ロジック再利用パターン

ビジネスロジックを再利用可能な単位に分離するパターンです。フレームワークによって実装方法が異なります。

### フレームワーク別の名称

| フレームワーク | 名称 | ファイル配置 |
|--------------|------|-------------|
| **React** | Custom Hooks | `src/hooks/useXxx.ts` |
| **Vue** | Composables | `src/composables/useXxx.ts` |
| **Svelte** | Stores / Runes | `src/stores/xxx.ts` |

### 共通の命名規則

- `use` プレフィックス（React/Vue）
- 動詞 + 名詞: `useUserProfile`, `useFetchData`

### 責務

- 状態管理
- 副作用（API呼び出し等）
- ビジネスロジック

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
│   └── Button/
│       ├── index.ts
│       ├── Button.tsx          # or .vue / .svelte
│       ├── Button.spec.tsx
│       └── Button.stories.tsx
├── molecules/
│   └── SearchForm/
├── organisms/
│   └── Header/
├── templates/
│   └── DashboardLayout/
└── pages/
    └── Dashboard/
```

> **Note**: テスト（`.spec.tsx`）とStorybook（`.stories.tsx`）はコンポーネントと同一ディレクトリに配置（コロケーション）。

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

## フレームワーク別実装ガイド

各フレームワークの具体的な実装例は以下を参照してください：

| フレームワーク | ガイド |
|--------------|--------|
| **React / Next.js** | [31-implementation-patterns-react.md](./31-implementation-patterns-react.md) |
| **Vue / Nuxt** | [32-implementation-patterns-vue.md](./32-implementation-patterns-vue.md) |
| **Svelte / SvelteKit** | [33-implementation-patterns-svelte.md](./33-implementation-patterns-svelte.md) |

---

## 関連ドキュメント

- [20-clean-architecture.md](./20-clean-architecture.md) - レイヤー設計
- [40-domain-modeling.md](./40-domain-modeling.md) - ドメインモデリング
- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [80-testing.md](./80-testing.md) - テスト戦略

---

**最終更新**: YYYY-MM-DD
