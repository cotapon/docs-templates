# AGENTS.md

{{PROJECT_NAME}} の AI Agent 向けコンテキストファイル

日本語で回答してください。

## このプロジェクトについて

<!-- 📝 1-2文でプロジェクトの目的を記載 -->

**技術スタック**: {{TECH_STACK}}, {{DB_TYPE}}, {{AUTH_PROVIDER}}

## 作業の進め方

### 変更の検証

コードを編集したら **必ず lint / 型チェック / テストを実行** してから完了を報告すること。
コマンドは `docs/specs/commands.md` を参照。

### 詳細情報の探し方

| 知りたいこと | 読むファイル |
|-------------|-------------|
| ドキュメント全体の起点 | `docs/guides/00-overview.md` |
| 技術スタック・構造 | `docs/guides/10-architecture.md` |
| コーディング規約 | `docs/guides/50-coding-standards.md` |
| テストの書き方 | `docs/guides/80-testing.md` |
| Git運用・PR作成 | `docs/guides/90-development-workflow.md` |
| プロダクト仕様 | `docs/specs/overview.md` |
| DB設計 | `docs/specs/database.md` |
| コマンド一覧 | `docs/specs/commands.md` |

**迷ったら `docs/guides/00-overview.md` を最初に確認。**

## 基本原則

1. **YAGNI**: 要求されていない機能や最適化は追加しない
2. **品質保証**: 変更後は必ず検証してから完了

## コミット

- **形式**: `feat(scope): 説明`（Conventional Commits）
- **タイミング**: ユーザーから依頼された場合のみ

---

このファイルを更新したら `CLAUDE.md` にも同期すること。
