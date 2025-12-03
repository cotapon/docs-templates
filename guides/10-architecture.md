# 技術アーキテクチャ

{{PROJECT_NAME}} の技術スタック、アーキテクチャ設計、および実装の詳細を説明します。

<!-- 
📝 書くべき内容:
- 採用技術とバージョン
- 各技術の選定理由（なぜこの技術を選んだか）
- ディレクトリ構造と各フォルダの役割
- レイヤー構成（該当する場合）
- 設定ファイル一覧
- 依存関係管理方針

カスタマイズ:
- 使用していない技術のセクションは削除
- プロジェクト固有のディレクトリ構造に更新
-->

## 📚 目次

1. [技術スタック](#技術スタック)
2. [アーキテクチャ概要](#アーキテクチャ概要)
3. [ディレクトリ構造](#ディレクトリ構造)
4. [設定ファイル](#設定ファイル)

---

## 技術スタック

### フレームワーク

#### {{TECH_STACK}} x.x.x

**選定理由**:
<!-- なぜこのフレームワークを選んだか理由を記載 -->
- 理由1: 
- 理由2: 
- 理由3: 

**設定ファイル**: `xxx.config.ts`

<!-- 
例: Next.js の場合
#### Next.js 14.x
**選定理由**:
- App Router による直感的なルーティング
- React Server Components のサポート
- ビルトインの最適化機能（画像、フォント）

**設定ファイル**: `next.config.ts`
-->

### 言語

#### TypeScript x.x

**選定理由**:
- 型安全性による開発効率向上
- IDE サポートの充実
- リファクタリングの安全性

**設定**: strict モード有効（`tsconfig.json`）

**型定義の配置**:
- グローバル型: `src/types/`
- コンポーネント固有の型: コンポーネントファイル内

### スタイリング

<!-- Tailwind CSS / CSS Modules / styled-components など -->

#### Tailwind CSS x.x

**選定理由**:
- ユーティリティファーストで開発速度向上
- カスタマイズ性が高い
- 本番ビルドサイズの最適化

**設定ファイル**: `tailwind.config.ts`

### データベース / バックエンド

#### {{DB_TYPE}}

**選定理由**:
- 理由1: 
- 理由2: 

**環境変数**:
- `DATABASE_URL` - データベース接続文字列
- <!-- その他の環境変数 -->

### テスティング

| ツール | 用途 | 設定ファイル |
|-------|------|------------|
| Vitest | ユニットテスト | `vitest.config.ts` |
| Playwright | E2Eテスト | `playwright.config.ts` |
| Testing Library | コンポーネントテスト | - |

---

## アーキテクチャ概要

<!-- プロジェクトのアーキテクチャパターンを記載 -->

### アーキテクチャパターン

このプロジェクトは **レイヤードアーキテクチャ** を採用しています。

```
┌─────────────────────────────────────┐
│   Presentation Layer (UI)           │  ← src/app, src/components
├─────────────────────────────────────┤
│   Business Logic Layer              │  ← src/hooks (React) / src/composables (Vue) / src/stores (Svelte)
├─────────────────────────────────────┤
│   Data Access Layer                 │  ← src/services, src/api
├─────────────────────────────────────┤
│   External Services (APIs, DB)      │  ← 外部 API, Database
└─────────────────────────────────────┘
```

<!-- OPTIONAL: Clean Architecture の場合 -->
<!--
このプロジェクトは **クリーンアーキテクチャ** を採用しています。
詳細は [20-clean-architecture.md](./20-clean-architecture.md) を参照。
-->

### コンポーネント設計原則

1. **単一責任の原則**: 1つのコンポーネントは1つの責任のみ
2. **コンポジション**: 小さなコンポーネントを組み合わせる
3. **Props のドリルダウン回避**: 状態管理を活用（React: Context API / Vue: Provide/Inject / Svelte: Stores）

---

## ディレクトリ構造

<!-- プロジェクトの実際の構造に合わせて更新 -->

```
{{PROJECT_SLUG}}/
├── .github/                    # GitHub設定（Actions, templates）
├── docs/                       # ドキュメント
│   ├── guides/                 # 実装ガイド（How-to）
│   └── specs/                  # 仕様リファレンス（What-is）
├── public/                     # 静的ファイル
├── src/                        # ソースコード
│   ├── app/                    # ページ・ルーティング（※フレームワークにより異なる）
│   ├── components/             # UIコンポーネント
│   │   ├── ui/                 # 基本UIコンポーネント
│   │   └── features/           # 機能別コンポーネント
│   ├── hooks/                  # ロジック再利用（※下記注参照）
│   ├── lib/                    # ユーティリティ関数
│   └── types/                  # 型定義
├── tests/                      # テスト
│   └── e2e/                    # E2Eテスト
├── AGENTS.md                   # AI向けコンテキスト
└── package.json                # npm設定
```

### ディレクトリの役割

| ディレクトリ | 役割 |
|------------|------|
| `src/app/` | ページコンポーネント、ルーティング |
| `src/components/` | 再利用可能なUIコンポーネント |
| `src/hooks/` | ロジック再利用（下記注参照） |
| `src/lib/` | ユーティリティ関数・ヘルパー |
| `src/types/` | TypeScript型定義 |
| `tests/e2e/` | E2Eテスト（Playwright/Cypress） |
| `docs/` | プロジェクトドキュメント |

> **Note**: ロジック再利用のディレクトリ名はフレームワークにより異なります：
> - **React**: `src/hooks/` (Custom Hooks)
> - **Vue**: `src/composables/` (Composables)
> - **Svelte**: `src/stores/` (Stores/Runes)

---

## 設定ファイル

| ファイル | 役割 |
|---------|------|
| `tsconfig.json` | TypeScript設定（strictモード） |
| `eslint.config.mjs` | ESLint設定 |
| `vitest.config.ts` | Vitest設定 |
| `playwright.config.ts` | Playwright設定 |
| <!-- 追加 --> | <!-- 追加 --> |

---

## 関連ドキュメント

- [00-overview.md](./00-overview.md) - ドキュメントの起点
- [20-clean-architecture.md](./20-clean-architecture.md) - レイヤー設計（該当する場合）
- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [../specs/database.md](../specs/database.md) - データベース設計
- [../specs/commands.md](../specs/commands.md) - コマンドリファレンス

---

**最終更新**: YYYY-MM-DD

