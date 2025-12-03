# ドキュメントテンプレート

**フロントエンドプロジェクト向け**のドキュメントテンプレートです。

### 対象技術スタック
- **React系**: Next.js, Remix, Vite + React
- **Vue系**: Nuxt, Vite + Vue
- **その他**: Svelte, SvelteKit, Astro

このテンプレートセットを使って、新しいプロジェクトのドキュメント構造を素早く構築できます。

## 🚀 使い方

### 方法A: Claude Projectで自動生成（推奨）

Claude Projectを使って、対話形式でプロジェクトに合わせたドキュメントを自動生成できます。

1. [Claude](https://claude.ai) でProjectを新規作成
2. 「ファイルを追加」でこのリポジトリを連携
3. 「プロジェクトの指示を設定」に [`CLAUDE_PROJECT_INSTRUCTIONS.md`](./CLAUDE_PROJECT_INSTRUCTIONS.md) の内容をコピー
4. 会話を開始すると、Claudeがプロジェクト情報を質問し、テンプレートからドキュメントを生成

### 方法B: 手動でコピー＆置換

#### 1. テンプレートをコピー

```bash
# ドキュメントディレクトリをコピー
cp -r docs-templates/guides/ your-project/docs/guides/
cp -r docs-templates/specs/ your-project/docs/specs/

# AI向けルールブックをコピー
cp docs-templates/AGENTS.template.md your-project/AGENTS.md
cp docs-templates/AGENTS.template.md your-project/CLAUDE.md
```

#### 2. プレースホルダーを置換

以下のプレースホルダーをプロジェクト固有の値に置換してください：

| プレースホルダー | 説明 | 例 |
|-----------------|------|-----|
| `{{PROJECT_NAME}}` | プロジェクト名 | `My Awesome App` |
| `{{PROJECT_SLUG}}` | プロジェクトのURL用識別子 | `my-awesome-app` |
| `{{GITHUB_REPO}}` | GitHubリポジトリ | `username/my-awesome-app` |
| `{{TECH_STACK}}` | フレームワーク | `Next.js` / `Nuxt` / `Remix` |
| `{{DB_TYPE}}` | データベース | `Supabase` / `PostgreSQL` / `MongoDB` |
| `{{AUTH_PROVIDER}}` | 認証 | `Supabase Auth` / `NextAuth` / `Auth0` |

#### 一括置換スクリプト（bash）

```bash
# 例: プロジェクト名を置換
find docs/ -name "*.md" -exec sed -i '' 's/{{PROJECT_NAME}}/My App/g' {} \;
find docs/ -name "*.md" -exec sed -i '' 's/{{PROJECT_SLUG}}/my-app/g' {} \;
find docs/ -name "*.md" -exec sed -i '' 's/{{GITHUB_REPO}}/username\/my-app/g' {} \;
find docs/ -name "*.md" -exec sed -i '' 's/{{TECH_STACK}}/Next.js/g' {} \;
find docs/ -name "*.md" -exec sed -i '' 's/{{DB_TYPE}}/Supabase/g' {} \;
find docs/ -name "*.md" -exec sed -i '' 's/{{AUTH_PROVIDER}}/Supabase Auth/g' {} \;
```

#### 3. 不要なセクションを削除

各テンプレート内のコメント `<!-- OPTIONAL: ... -->` で囲まれたセクションは、
プロジェクトに該当しない場合は削除してください。

#### 4. 内容を充実させる

各テンプレートには `<!-- 📝 書くべき内容: ... -->` のガイドがあります。
これを参考に、プロジェクト固有の内容を記載してください。

---

## 📂 ディレクトリ構造

```
docs/
├── guides/                  # How-to ガイド（手順・実装方法）
│   ├── 00-overview.md       # ★必須: ドキュメントの起点
│   ├── 10-architecture.md   # 技術スタック・全体設計
│   ├── 20-clean-architecture.md  # レイヤー設計（該当する場合）
│   ├── 30-implementation-patterns.md  # 実装パターン（概念）
│   ├── 31-implementation-patterns-react.md   # React/Next.js 実装例
│   ├── 32-implementation-patterns-vue.md     # Vue/Nuxt 実装例
│   ├── 33-implementation-patterns-svelte.md  # Svelte/SvelteKit 実装例
│   ├── 40-domain-modeling.md     # ドメイン設計
│   ├── 50-coding-standards.md    # ★必須: コーディング規約
│   ├── 55-error-handling.md      # エラーハンドリング
│   ├── 60-ui-components.md       # UI設計
│   ├── 70-data-fetching.md       # データ取得戦略
│   ├── 80-testing.md             # ★必須: テスト戦略
│   ├── 90-development-workflow.md # ★必須: 開発フロー・Git運用
│   └── 95-security.md            # セキュリティ
│
└── specs/                   # What-it-is 仕様書（リファレンス）
    ├── overview.md          # ★必須: プロダクト概要
    ├── requirements.md      # 要件定義
    ├── database.md          # DB設計
    ├── commands.md          # コマンドリファレンス
    ├── examples.md          # コードサンプル
    ├── glossary.md          # 用語集
    ├── features/            # 機能仕様
    │   └── _template.md     # 機能仕様テンプレート
    └── user-flows/          # ユーザーフロー
        └── _template.md     # ユーザーフローテンプレート
```

---

## 📝 各ドキュメントの役割と書くべき内容

### 必須ドキュメント（★マーク）

| ファイル | 役割 | 書くべき内容 |
|---------|------|-------------|
| `AGENTS.md` | AI向けルールブック | プロジェクト概要、必須コマンド、コーディング原則 |
| `guides/00-overview.md` | ドキュメントの起点 | 構成表、品質チェック方法、読む順序 |
| `guides/50-coding-standards.md` | コーディング規約 | 命名規則、禁止事項、フォーマット |
| `guides/80-testing.md` | テスト戦略 | テスト種類、カバレッジ目標、実行方法 |
| `guides/90-development-workflow.md` | 開発フロー | Git運用、PR作成、レビュー基準 |
| `specs/overview.md` | プロダクト概要 | ペルソナ、ユースケース、価値仮説 |

### オプションドキュメント

| ファイル | 役割 | 含めるケース |
|---------|------|-------------|
| `guides/10-architecture.md` | 技術アーキテクチャ | 技術選定理由を共有したい場合 |
| `guides/20-clean-architecture.md` | レイヤー設計 | Clean Architecture採用時 |
| `guides/30-implementation-patterns.md` | 実装パターン | Container/Presenter等のパターン採用時 |
| `guides/40-domain-modeling.md` | ドメイン設計 | DDD採用時 |
| `guides/60-ui-components.md` | UI設計 | デザインシステムがある場合 |
| `guides/70-data-fetching.md` | データ取得戦略 | SWR/React Query等を使う場合 |
| `specs/database.md` | DB設計 | RDB/NoSQLを使う場合 |
| `specs/features/` | 機能仕様 | 機能が複数ある場合 |

---

## 🔢 番号付けの意味

`guides/` 内のファイルは読む順序を番号で示しています：

- `00-09`: 概要・イントロダクション
- `10-19`: アーキテクチャ・技術基盤
- `20-39`: 設計・実装パターン
- `40-49`: ドメイン・モデリング
- `50-59`: コーディング規約・エラー処理
- `60-69`: UI・コンポーネント
- `70-79`: データ取得・状態管理
- `80-89`: テスト・品質
- `90-99`: ワークフロー・セキュリティ

`specs/` 内のファイルは参照用なので番号なしです。

---

## 🛠 カスタマイズのヒント

### 小規模プロジェクトの場合

最低限以下だけで開始可能：

```
docs/
├── guides/
│   ├── 00-overview.md
│   ├── 50-coding-standards.md
│   └── 80-testing.md
└── specs/
    └── overview.md
```

### 大規模プロジェクトの場合

`specs/features/` 配下に機能ごとのファイルを追加：

```
specs/features/
├── authentication.md
├── user-management.md
├── file-upload.md
└── notifications.md
```

---

## 📚 関連リソース

- [Conventional Commits](https://www.conventionalcommits.org/) - コミットメッセージ規約
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) - クリーンアーキテクチャ
- [Testing Library](https://testing-library.com/) - テストライブラリ

---

**最終更新**: 2025-12-03

