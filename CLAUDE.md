# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**フロントエンドプロジェクト向け**ドキュメントテンプレート。React/Next.js, Vue/Nuxt, Svelte等のプロジェクトのドキュメント作成を効率化するためのテンプレート集。

**2つの利用方法:**
1. **Claude Project自動化**: CLAUDE_PROJECT_INSTRUCTIONS.md を使用してAIと対話しながら自動生成
2. **手動コピー**: プレースホルダーを置換して使用

## Repository Structure

```
├── AGENTS.template.md              # AIエージェント用コンテキストテンプレート
├── CLAUDE_PROJECT_INSTRUCTIONS.md  # Claude Project自動化ガイド
├── guides/                         # HOW-TO: 実装ガイド (番号順に読む)
│   ├── 00-overview.md             # エントリーポイント・ナビゲーション
│   ├── 50-coding-standards.md     # コーディング規約 (必須)
│   ├── 80-testing.md              # テスト戦略 (必須)
│   └── 90-development-workflow.md # Git運用 (必須)
└── specs/                          # WHAT-IS: 仕様リファレンス
    ├── overview.md                 # プロダクト概要 (必須)
    ├── features/_template.md       # 機能仕様テンプレート
    └── user-flows/_template.md     # ユーザーフローテンプレート
```

## Placeholder Variables

テンプレート内の以下のプレースホルダーをプロジェクト固有の値に置換:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PROJECT_NAME}}` | プロジェクト名 | My App |
| `{{PROJECT_SLUG}}` | URL用識別子 (kebab-case) | my-app |
| `{{GITHUB_REPO}}` | GitHubリポジトリ | user/my-app |
| `{{TECH_STACK}}` | フレームワーク | Next.js, Nuxt |
| `{{DB_TYPE}}` | データベース | Supabase, PostgreSQL |
| `{{AUTH_PROVIDER}}` | 認証プロバイダ | Supabase Auth, NextAuth |

## Document Categories

### 必須ドキュメント (最小構成)
- `guides/00-overview.md` - ドキュメント全体の案内
- `guides/50-coding-standards.md` - コーディング規約
- `guides/80-testing.md` - テスト戦略
- `guides/90-development-workflow.md` - Git運用・PR
- `specs/overview.md` - プロダクト概要
- `AGENTS.md` - AIエージェント用コンテキスト

### オプションドキュメント
- `guides/10-architecture.md` - 技術スタック・設計
- `guides/20-clean-architecture.md` - レイヤードアーキテクチャ
- `guides/30-implementation-patterns.md` - デザインパターン
- `guides/40-domain-modeling.md` - ドメインモデリング
- `guides/55-error-handling.md` - エラーハンドリング
- `guides/60-ui-components.md` - UIコンポーネント設計
- `guides/70-data-fetching.md` - データフェッチ戦略
- `guides/95-security.md` - セキュリティ

## Template Conventions

### Placeholder Comments
テンプレート内のガイダンスコメント:
- `<!-- 📝 書くべき内容: ... -->` - 記述すべき内容の説明
- `<!-- OPTIONAL: ... -->` - 省略可能なセクション

### Numbering System (guides/)
- 00-09: Overview
- 10-19: Architecture
- 20-39: Design patterns
- 40-49: Domain modeling
- 50-59: Coding standards
- 60-69: UI components
- 70-79: Data fetching
- 80-89: Testing
- 90-99: Workflow/Security

## Key Principles

1. **guides/** = HOW-TO (手順・実装パターン)
2. **specs/** = WHAT-IS (仕様・リファレンス)
3. 小規模プロジェクトは必須ドキュメントのみで開始
4. テンプレートは日本語主体、技術用語は英語
