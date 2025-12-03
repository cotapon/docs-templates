# AGENTS.md

AI Agent Instructions (Claude Code / その他自動エージェント向け)

<!-- 
📝 このファイルの目的:
- AI エージェントがプロジェクトを理解するためのコンテキスト提供
- 必須ルール・禁止事項の明示
- ドキュメント構造へのナビゲーション

カスタマイズ:
- {{PROJECT_NAME}} をプロジェクト名に置換
- プロジェクト固有のルールを追加
- 不要なセクションは削除
-->

英語で think して、日本語で output してください
あらゆるコード変更では、全ての関数・主要処理ブロックに簡潔なコメントを付け、JSDoc も必ず記述してください。
大きなファイルに処理を詰め込まず、責務ごとに抽象化してモジュール分割してください。再利用が見込める処理はユーティリティとして切り出してください。
この AGENTS.md を更新した場合は、同一内容を `CLAUDE.md` にも反映して同期してください。

## 📚 ドキュメント構造

ドキュメントは2つのカテゴリに体系化されています：

- **`docs/guides/`**: How-to ガイド（手順・実装方法）
- **`docs/specs/`**: What-it-is 仕様書（要件・設計・リファレンス）

**起点**: [docs/guides/00-overview.md](./docs/guides/00-overview.md)

**迷ったら必ず`docs/guides/00-overview.md`を確認してください。**

**プロダクト仕様ハブ**: [docs/specs/overview.md](./docs/specs/overview.md) — プロジェクト概要・ペルソナ・ユースケース等の入口

---

## 🤖 AI Agentの振る舞い指示

### コード編集時の必須手順（厳守）

**重要**: コードを編集したら以下を**必ず実行**すること：

```bash
npm run lint && npx tsc --noEmit && npm run test
```

これらの確認を怠った場合、PRがマージされません。

### コミュニケーション原則
- 回答・説明・コメントはすべて**日本語**で行うこと
- 技術的なディスカッションでも敬意を持った簡潔かつ丁寧な文体を維持すること
- 専門用語は適切に使用しつつ、必要に応じて補足説明を加えること

---

## 🎯 重要な原則

### 1. APIファースト原則
実装前に必ずAPI仕様を確認し、ドメインモデルはAPI仕様に準拠すること。

### 2. YAGNI（You Aren't Gonna Need It）
不要な最適化や追加機能は避け、要求されたタスクに集中すること。

<!-- OPTIONAL: Clean Architecture を採用する場合 -->
### 3. クリーンアーキテクチャ厳守
レイヤー間の依存関係を正しく保ち、ドメイン層は外部依存ゼロを維持すること。
<!-- /OPTIONAL -->

### 4. 品質保証プロセス必須
コード編集後は必ずLint、型チェック、テストを実行すること。

---

## 📖 主要ドキュメントへのクイックリンク

### 必読ガイド（docs/guides/ - 優先度順）
1. **[00-overview.md](./docs/guides/00-overview.md)** - プロジェクト全体像とナビゲーション
2. **[10-architecture.md](./docs/guides/10-architecture.md)** - システムアーキテクチャ
3. **[50-coding-standards.md](./docs/guides/50-coding-standards.md)** - コーディング規約
4. **[80-testing.md](./docs/guides/80-testing.md)** - テスト・品質保証
5. **[90-development-workflow.md](./docs/guides/90-development-workflow.md)** - 開発ワークフロー

<!-- OPTIONAL: 詳細ガイドがある場合 -->
### 実装時の参照先
- **[30-implementation-patterns.md](./docs/guides/30-implementation-patterns.md)** - 実装パターン
- **[40-domain-modeling.md](./docs/guides/40-domain-modeling.md)** - ドメインモデル
- **[60-ui-components.md](./docs/guides/60-ui-components.md)** - UIコンポーネント設計
- **[70-data-fetching.md](./docs/guides/70-data-fetching.md)** - データフェッチ戦略
<!-- /OPTIONAL -->

### 仕様・リファレンス資料（docs/specs/）
- **[overview.md](./docs/specs/overview.md)** - プロダクト概要・ペルソナ・ユースケース
- **[requirements.md](./docs/specs/requirements.md)** - 要件・仕様原本
- **[database.md](./docs/specs/database.md)** - DB 設計
- **[commands.md](./docs/specs/commands.md)** - よく使うコマンド
- **[examples.md](./docs/specs/examples.md)** - 実装サンプル
- **[glossary.md](./docs/specs/glossary.md)** - 用語集

---

## 🚀 クイックスタート

### 開発環境セットアップ
```bash
# 依存関係インストール
npm install

# 開発サーバー起動
npm run dev

# カスタムポート
PORT=8080 npm run dev
```

### コード編集時の必須確認
```bash
# 品質チェック（コード編集後必須）
npm run lint && npx tsc --noEmit && npm run test

# 個別実行
npm run lint              # ESLint
npx tsc --noEmit          # 型チェック
npm run test              # ユニットテスト
npx playwright test       # E2Eテスト
```

---

## 📂 プロジェクト構造概要

<!-- 
📝 プロジェクトのディレクトリ構造に合わせて更新してください
-->

```
src/
├─ app/                 # ルート定義
├─ components/          # UIコンポーネント
├─ hooks/               # カスタムフック
├─ lib/                 # ユーティリティ
└─ types/               # 型定義
```

詳細は **[10-architecture.md](./docs/guides/10-architecture.md)** を参照してください。

---

## 📝 コーディング規約

- **TypeScript**: strict モード前提
- **命名規則**:
    - Hooks: `useSomething`
    - Components: `PascalCase`
    - Files: kebab-case
- **Import**: `@/` エイリアス活用、相対パスのネストを避ける
- **型安全性**: `any` は避ける
- **ESLint**: `no-console`（warn/error のみ許容）

詳細は **[50-coding-standards.md](./docs/guides/50-coding-standards.md)** を参照してください。

---

## 🧪 テスト・品質保証

- **ユニットテスト**: Testing Library + Vitest
- **E2Eテスト**: Playwright
- **カバレッジ**: `npm run test:coverage` で確認

詳細は **[80-testing.md](./docs/guides/80-testing.md)** を参照してください。

---

## 🔒 セキュリティ

- 秘密情報は `.env.local` に保管（リポジトリにコミット禁止）
- API キーは環境変数経由で管理

---

## 🔄 コミット / PR 方針

### コミットルール
- **実行タイミング**: ユーザーからの明示的な依頼がある場合のみ
- **Conventional Commits**: `feat(scope): ...` 形式
- **メッセージ本文**: 日本語で記述

### PRルール
- **必須項目**: 背景、対応内容、テスト証跡
- **マージ前確認**: `npm run lint` と `npm run test` の成功確認
- **マージ方式**: Squash Merge

詳細は **[90-development-workflow.md](./docs/guides/90-development-workflow.md)** を参照してください。

---

## 📞 サポート

### 質問・フィードバック
- GitHub Issues: [{{GITHUB_REPO}}/issues](https://github.com/{{GITHUB_REPO}}/issues)

### 参考リソース
- **Next.js Documentation**: https://nextjs.org/docs
- **Conventional Commits**: https://www.conventionalcommits.org/

---

## 🔄 ドキュメント更新履歴

- YYYY-MM-DD: 初版作成

---

**Note**: このファイルは全AI Agentが共通で参照するルールブックです。CLAUDE.mdに変更が入った場合はこのファイルも、AGENTS.mdを更新した場合は必ず `CLAUDE.md` も同期してください。

