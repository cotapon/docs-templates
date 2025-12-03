# ドキュメント概要とナビゲーション

{{PROJECT_NAME}} のドキュメントセット全体を素早く把握するための起点です。開発を始める前に本ページと [AGENTS.md](../../AGENTS.md) を確認してください。

<!-- 
📝 書くべき内容:
- ドキュメントの目的と構成
- 各ファイルの役割一覧（テーブル形式推奨）
- 品質チェックコマンド
- 最初に読むべきドキュメントの案内

カスタマイズ:
- プロジェクトに存在しないファイルは表から削除
- プロジェクト固有の目的を追加
-->

## 目的

- ドキュメントの配置と役割を明確化し、迷子にならないようにする
- 品質プロセス（Lint/型チェック/テスト）の実行を徹底する
- <!-- プロジェクト固有の目的を追加 -->

## ドキュメント構成

### guides/（実装ガイド：How-to）

読む順番で番号付け。実装時の手順・規約を提供します。

| ファイル | 内容 |
|---------|------|
| `00-overview.md`（本書）| ドキュメントの起点と必読事項 |
| `10-architecture.md` | 技術スタック・システム全体像 |
| `20-clean-architecture.md` | レイヤー境界と依存関係ルール |
| `30-implementation-patterns.md` | 実装パターンとテンプレート |
| `40-domain-modeling.md` | ドメインモデル設計ガイド |
| `50-coding-standards.md` | コーディング規約・命名・フォーマット |
| `55-error-handling.md` | エラーハンドリング |
| `60-ui-components.md` | UI/UX とコンポーネント設計 |
| `70-data-fetching.md` | データフェッチ戦略 |
| `80-testing.md` | テスト戦略と品質保証フロー |
| `90-development-workflow.md` | 開発フローとGit運用 |
| `95-security.md` | セキュリティ |

<!-- 
💡 ヒント: プロジェクトに不要なガイドは削除してください
最小構成: 00-overview, 50-coding-standards, 80-testing, 90-development-workflow
-->

### specs/（仕様リファレンス：What-is）

参照用の仕様書。順不同でアクセス可能です。

| ファイル | 内容 |
|---------|------|
| `overview.md` | プロダクト概要・ペルソナ・ユースケースのハブ |
| `requirements.md` | 要件・仕様原本 |
| `database.md` | DB設計・スキーマ |
| `commands.md` | よく使うCLIコマンド |
| `examples.md` | 実装サンプル集 |
| `glossary.md` | 用語集・前提知識 |
| `user-flows/` | ユーザーフロー仕様 |
| `features/` | 機能別の詳細仕様 |

## 品質チェック（必須）

コードを変更した場合は必ず以下を連続実行してください：

```bash
npm run lint && npx tsc --noEmit && npm run test
```

E2E が必要な場合は `npx playwright test` も実行します。実行結果はプルリク時に記録してください。

## 次のステップ

1. 仕様やドメイン理解が必要なら `specs/overview.md` と `specs/requirements.md` を確認
2. 技術構成を把握するなら `guides/10-architecture.md`
3. コーディング前に `guides/50-coding-standards.md` を再確認
4. テストを書く前に `guides/80-testing.md` を参照

---

## 関連ドキュメント

- [AGENTS.md](../../AGENTS.md) / [CLAUDE.md](../../CLAUDE.md) - AI向けプロジェクトコンテキスト
- [specs/overview.md](../specs/overview.md) - プロダクト概要

