# 開発ワークフロー・Git運用ガイド

{{PROJECT_NAME}} での日々の開発フロー、Git の運用ルールを説明します。

<!-- 
📝 書くべき内容:
- 開発フロー（TDD/BDD）
- ブランチ戦略
- コミットメッセージ規約
- PR作成・レビューガイドライン
- CI/CDパイプライン

★必須ファイル: すべてのプロジェクトで作成してください
-->

## 📚 目次

1. [開発フロー](#開発フロー)
2. [ブランチ戦略](#ブランチ戦略)
3. [コミットメッセージ規約](#コミットメッセージ規約)
4. [PR作成ガイド](#pr作成ガイド)
5. [コードレビュー](#コードレビュー)

---

## 開発フロー

### 基本フロー

```
1. 要件確認 → 2. ブランチ作成 → 3. 実装 → 4. テスト → 5. PR作成 → 6. レビュー → 7. マージ
```

### 詳細手順

#### 1. 要件を確認

- [specs/requirements.md](../specs/requirements.md) を確認
- 不明点があれば事前に確認

#### 2. ブランチを作成

```bash
git checkout develop
git pull origin develop
git checkout -b feature/123-add-user-profile
```

#### 3. 実装（TDD推奨）

```bash
# テストを書く（Red）
npm run test:watch

# 実装する（Green）
# リファクタリング（Refactor）
```

#### 4. 品質チェック

```bash
npm run lint && npx tsc --noEmit && npm run test
```

#### 5. コミット・プッシュ

```bash
git add .
git commit -m "feat(user): ユーザープロフィール機能を追加"
git push origin feature/123-add-user-profile
```

#### 6. PR作成

GitHubでPRを作成し、レビューを依頼

---

## ブランチ戦略

### Git-flow

```
main ─────────────────────────────── 本番環境
  ↑
release/* ────────────────────────── リリース準備
  ↑
develop ──────────────────────────── 開発の最新
  ↑
feature/* ────────────────────────── 機能開発
hotfix/* ─────────────────────────── 緊急修正 → main + develop
```

### ブランチの種類

| ブランチ | 用途 | 分岐元 | マージ先 |
|---------|------|--------|---------|
| `main` | 本番環境 | - | - |
| `develop` | 開発最新 | - | `main` |
| `feature/*` | 機能開発 | `develop` | `develop` |
| `hotfix/*` | 緊急修正 | `main` | `main` + `develop` |
| `release/*` | リリース準備 | `develop` | `main` + `develop` |

### ブランチ命名規則

```
<type>/<issue-number>-<short-description>
```

**例**:
```
feature/123-user-authentication
feature/456-add-dark-mode
hotfix/789-fix-login-error
```

---

## コミットメッセージ規約

### Conventional Commits

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type一覧

| Type | 用途 | 例 |
|------|------|-----|
| `feat` | 新機能 | `feat(auth): ログイン機能を追加` |
| `fix` | バグ修正 | `fix(button): ホバー時の色を修正` |
| `docs` | ドキュメント | `docs(readme): セットアップ手順を更新` |
| `style` | フォーマット | `style: インデントを修正` |
| `refactor` | リファクタリング | `refactor(api): エラーハンドリングを改善` |
| `perf` | パフォーマンス | `perf(list): レンダリングを最適化` |
| `test` | テスト | `test(user): プロフィールテストを追加` |
| `chore` | 雑務 | `chore(deps): 依存関係を更新` |

### 良いコミットメッセージの例

```
feat(auth): OAuthログイン機能を追加

GoogleとGitHubのOAuth認証フローを実装しました。

- OAuthコールバックハンドラーを追加
- トークンリフレッシュロジックを実装
- プロフィール同期機能を追加

Closes #123
```

---

## PR作成ガイド

### PRタイトル

```
<type>(<scope>): <description>
```

**例**:
```
feat(auth): add OAuth login support
fix(button): correct hover state color
```

### PR説明テンプレート

```markdown
## 概要
<!-- この PR の目的を簡潔に説明 -->

## 変更内容
- 変更点1
- 変更点2

## テスト
- [ ] ユニットテスト追加
- [ ] 統合テスト追加
- [ ] E2Eテスト追加（必要に応じて）

## スクリーンショット
<!-- UI変更がある場合は添付 -->

## チェックリスト
- [ ] `npm run lint` が通る
- [ ] `npx tsc --noEmit` が通る
- [ ] `npm run test` が通る
- [ ] ドキュメントを更新した（必要に応じて）

## 関連Issue
Closes #XX
```

### PRサイズの目安

- ✅ 変更行数: 200-400行以内
- ✅ 変更ファイル数: 10ファイル以内
- ✅ レビュー時間: 30分以内

---

## コードレビュー

### レビュアーのチェックポイント

#### 機能性
- [ ] 要件を満たしているか
- [ ] エッジケースを考慮しているか
- [ ] エラーハンドリングが適切か

#### コード品質
- [ ] 可読性が高いか
- [ ] 重複コードがないか
- [ ] 命名が適切か
- [ ] 設計パターンに従っているか

#### テスト
- [ ] テストが追加されているか
- [ ] テストが意味のある内容か
- [ ] カバレッジが維持されているか

### レビューコメントのラベル

| ラベル | 意味 |
|--------|------|
| 🐛 **Bug** | バグの可能性 |
| 💡 **Suggestion** | 改善提案（任意） |
| ❓ **Question** | 質問 |
| ⚠️ **Warning** | 注意喚起 |
| 🔒 **Security** | セキュリティ懸念 |
| ✅ **Approval** | 承認 |

**例**:
```
💡 Suggestion: Optional chainingを使うとよりシンプルになります

user.profile.name → user?.profile?.name
```

---

## コミット前チェックリスト

```bash
# 必須コマンド
npm run lint && npx tsc --noEmit && npm run test
```

### 詳細チェック

- [ ] TypeScriptのエラーがない
- [ ] ESLintの警告がない
- [ ] すべてのテストが通る
- [ ] 不要な `console.log` がない
- [ ] コミットメッセージが適切

---

## トラブルシューティング

### コミットメッセージを修正

```bash
# 最新のコミットメッセージを修正
git commit --amend -m "feat(auth): correct message"
```

### developの最新を取り込む

```bash
git checkout feature/123-xxx
git merge develop
# または
git rebase develop
```

### コンフリクトの解消

```bash
git checkout develop
git pull origin develop
git checkout feature/123-xxx
git merge develop
# コンフリクトを解消
git add .
git commit -m "chore: resolve merge conflicts"
git push
```

---

## 関連ドキュメント

- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [80-testing.md](./80-testing.md) - テスト戦略
- [AGENTS.md](../../AGENTS.md) - AI向けルールブック

---

**最終更新**: YYYY-MM-DD

