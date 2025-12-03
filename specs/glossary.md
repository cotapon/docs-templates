# 用語集・前提知識

{{PROJECT_NAME}} で使用される用語と基礎概念を定義します。

<!-- 
📝 書くべき内容:
- プロジェクト固有の用語
- 技術用語の定義
- 略語の正式名称
- 前提となる知識レベル

カスタマイズ:
- プロジェクト固有の用語を追加
-->

## 📚 目次

1. [プロジェクト用語](#プロジェクト用語)
2. [技術用語](#技術用語)
3. [略語一覧](#略語一覧)
4. [前提知識](#前提知識)

---

## プロジェクト用語

<!-- プロジェクト固有の用語を定義 -->

| 用語 | 定義 |
|------|------|
| <!-- 用語1 --> | <!-- 定義 --> |
| <!-- 用語2 --> | <!-- 定義 --> |

<!-- 
例:
| 用語 | 定義 |
|------|------|
| プロジェクト | 複数のユーザーが参加する作業単位 |
| メンバー | プロジェクトに参加しているユーザー |
| オーナー | プロジェクトの作成者。管理権限を持つ |
-->

---

## 技術用語

### アーキテクチャ・設計

| 用語 | 説明 |
|------|------|
| **Clean Architecture** | 関心の分離を徹底し、依存関係を内側に向けるアーキテクチャパターン |
| **Container/Presenter** | ロジック（Container）と表示（Presenter）を分離するUIパターン |
| **Domain Layer** | ビジネスロジックの中核。外部依存を持たない純粋な層 |
| **UseCase Layer** | アプリケーション固有のビジネスルールを実装する層 |
| **Infrastructure Layer** | 外部サービス（API、DB）との通信を担う層 |
| **Presentation Layer** | UIコンポーネントとユーザー操作を担う層 |

### テスト・品質

| 用語 | 説明 |
|------|------|
| **BDD** | Behavior-Driven Development。振る舞い駆動開発 |
| **TDD** | Test-Driven Development。テスト駆動開発 |
| **Given-When-Then** | BDDでのテストケース記述形式 |
| **Test Pyramid** | ユニット→統合→E2Eの階層構造 |
| **E2E Test** | End-to-Endテスト。ブラウザ操作を通じたテスト |

### 認証・認可

| 用語 | 説明 |
|------|------|
| **RBAC** | Role-Based Access Control。ロールベースアクセス制御 |
| **RLS** | Row Level Security。行レベルセキュリティ |
| **OAuth** | Open Authorization。認可の標準プロトコル |
| **JWT** | JSON Web Token。認証トークン形式 |

### フロントエンド

| 用語 | 説明 |
|------|------|
| **Server Components** | サーバーサイドでレンダリングされるReactコンポーネント |
| **Client Components** | クライアントで実行されるReactコンポーネント |
| **Hydration** | サーバーでレンダリングしたHTMLにJSを適用するプロセス |
| **SSR** | Server-Side Rendering。サーバーサイドレンダリング |
| **CSR** | Client-Side Rendering。クライアントサイドレンダリング |

---

## 略語一覧

| 略語 | 正式名称 | 説明 |
|------|---------|------|
| API | Application Programming Interface | プログラム間のインターフェース |
| BDD | Behavior-Driven Development | 振る舞い駆動開発 |
| CI/CD | Continuous Integration / Delivery | 継続的インテグレーション/デリバリー |
| CRUD | Create, Read, Update, Delete | 基本的なデータ操作 |
| DTO | Data Transfer Object | データ転送オブジェクト |
| E2E | End-to-End | 端から端まで |
| JWT | JSON Web Token | 認証トークン形式 |
| PR | Pull Request | プルリクエスト |
| RBAC | Role-Based Access Control | ロールベースアクセス制御 |
| RLS | Row Level Security | 行レベルセキュリティ |
| RSC | React Server Components | Reactサーバーコンポーネント |
| SPA | Single Page Application | シングルページアプリケーション |
| SSR | Server-Side Rendering | サーバーサイドレンダリング |
| SWR | Stale-While-Revalidate | データフェッチ戦略 |
| TDD | Test-Driven Development | テスト駆動開発 |
| UI/UX | User Interface / Experience | ユーザーインターフェース/体験 |
| YAGNI | You Aren't Gonna Need It | 必要になるまで作らない原則 |

---

## 前提知識

### このプロジェクトで想定する技術レベル

1. **TypeScript**: 型システム、ジェネリクス、型推論の基礎
2. **React**: Hooks、コンポーネントライフサイクル、状態管理
3. **{{TECH_STACK}}**: 基本的な使い方
4. **Git**: ブランチ操作、プルリクエスト

### 推奨学習リソース

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [React Documentation](https://react.dev/)
- [Testing Library Documentation](https://testing-library.com/docs/)

---

## 関連ドキュメント

- [../guides/10-architecture.md](../guides/10-architecture.md) - アーキテクチャ詳細
- [../guides/80-testing.md](../guides/80-testing.md) - テスト戦略
- [../guides/90-development-workflow.md](../guides/90-development-workflow.md) - 開発フロー

---

**最終更新**: YYYY-MM-DD

