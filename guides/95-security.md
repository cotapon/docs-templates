# セキュリティガイド

{{PROJECT_NAME}} のセキュリティ要件と実装ガイドラインを説明します。

<!-- 
📝 書くべき内容:
- 認証・認可の方針
- APIセキュリティ
- フロントエンドセキュリティ（XSS, CSRF対策）
- データ保護
- 秘密情報の管理

カスタマイズ:
- プロジェクトの認証プロバイダーに合わせて更新
- 該当しないセクションは削除
-->

## 📚 目次

1. [認証（Authentication）](#認証authentication)
2. [認可（Authorization）](#認可authorization)
3. [APIセキュリティ](#apiセキュリティ)
4. [フロントエンドセキュリティ](#フロントエンドセキュリティ)
5. [秘密情報の管理](#秘密情報の管理)

---

## 認証（Authentication）

### 認証プロバイダー

<!-- プロジェクトで使用する認証プロバイダーを記載 -->

**使用**: {{AUTH_PROVIDER}}

### 対応する認証方式

| 方式 | 対応 | 備考 |
|------|:----:|------|
| メール/パスワード | ✅ | 基本認証 |
| OAuth (Google) | ✅ | ソーシャルログイン |
| OAuth (GitHub) | ⬜ | 必要に応じて追加 |
| マジックリンク | ⬜ | 必要に応じて追加 |

### セッション管理

```typescript
// セッションの有効期限
const SESSION_EXPIRY = 7 * 24 * 60 * 60 // 7日間

// トークンリフレッシュのタイミング
const REFRESH_THRESHOLD = 60 * 60 // 残り1時間でリフレッシュ
```

### ログアウト処理

```typescript
async function logout() {
  // 1. サーバーサイドのセッション破棄
  await authClient.signOut()
  
  // 2. クライアントサイドのトークン削除
  localStorage.removeItem('token')
  
  // 3. ログインページへリダイレクト
  router.push('/login')
}
```

---

## 認可（Authorization）

### RBAC（Role-Based Access Control）

<!-- プロジェクトのロール定義を記載 -->

| ロール | 権限 |
|--------|------|
| `admin` | 全機能へのアクセス |
| `member` | 基本機能へのアクセス |
| `viewer` | 閲覧のみ |

### 権限チェックの実装

```typescript
// ミドルウェアでの権限チェック
export function requireRole(allowedRoles: string[]) {
  return async (req: Request) => {
    const user = await getUser(req)
    
    if (!user || !allowedRoles.includes(user.role)) {
      throw new ForbiddenError('この操作を行う権限がありません')
    }
  }
}

// 使用例
app.get('/admin/users', requireRole(['admin']), handleGetUsers)
```

### コンポーネントレベルの権限制御

```tsx
// 権限に応じた表示制御
function AdminPanel() {
  const { user } = useAuth()
  
  if (user?.role !== 'admin') {
    return <AccessDenied />
  }
  
  return <AdminContent />
}
```

---

## APIセキュリティ

### APIキーの管理

```bash
# .env.local（リポジトリにコミット禁止）
API_KEY=your-secret-api-key
DATABASE_URL=your-database-url
```

### 入力バリデーション

```typescript
// Zodを使用したバリデーション
import { z } from 'zod'

const createUserSchema = z.object({
  email: z.string().email('有効なメールアドレスを入力してください'),
  name: z.string().min(1, '名前は必須です').max(100),
  password: z.string().min(8, 'パスワードは8文字以上必要です'),
})

// APIルートでの使用
export async function POST(req: Request) {
  const body = await req.json()
  const validated = createUserSchema.parse(body) // バリデーションエラー時は例外
  // ...
}
```

### Rate Limiting

```typescript
// Rate Limiting の設定例
const rateLimiter = {
  windowMs: 15 * 60 * 1000, // 15分
  max: 100, // 最大100リクエスト
}
```

---

## フロントエンドセキュリティ

### XSS（Cross-Site Scripting）対策

```tsx
// ✅ Good: Reactの自動エスケープを活用
<div>{userInput}</div>

// ❌ Bad: dangerouslySetInnerHTML の使用（必要な場合のみ）
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// 必要な場合はサニタイズ
import DOMPurify from 'dompurify'
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

### CSRF（Cross-Site Request Forgery）対策

```typescript
// CSRFトークンの使用
const csrfToken = await getCsrfToken()

fetch('/api/data', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': csrfToken,
  },
  body: JSON.stringify(data),
})
```

### Content Security Policy

```typescript
// next.config.ts
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: "default-src 'self'; script-src 'self' 'unsafe-inline'",
  },
]
```

---

## 秘密情報の管理

### 環境変数のルール

| 接頭辞 | 公開範囲 | 用途 |
|--------|----------|------|
| `NEXT_PUBLIC_` | クライアント | 公開可能な設定 |
| なし | サーバーのみ | 秘密情報 |

### 秘密情報の保管場所

```
.env.local          # ローカル開発用（.gitignore）
.env.development    # 開発環境用テンプレート
.env.production     # 本番環境用テンプレート（値は空）
```

### 秘密情報の取り扱いルール

1. **リポジトリにコミットしない**
   - `.env.local` は `.gitignore` に含める
   - APIキーをソースコードに直接書かない

2. **環境変数経由でアクセス**
   ```typescript
   const apiKey = process.env.API_KEY
   if (!apiKey) throw new Error('API_KEY is not set')
   ```

3. **本番環境では環境変数サービスを使用**
   - Vercel Environment Variables
   - AWS Secrets Manager
   - etc.

---

## セキュリティチェックリスト

### 開発時

- [ ] 環境変数が `.gitignore` に含まれている
- [ ] `console.log` で機密情報を出力していない
- [ ] 入力値のバリデーションを行っている
- [ ] 適切な権限チェックを行っている

### デプロイ前

- [ ] 本番用の環境変数が設定されている
- [ ] HTTPSが有効になっている
- [ ] セキュリティヘッダーが設定されている
- [ ] 依存関係の脆弱性をチェック（`npm audit`）

---

## 関連ドキュメント

- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [55-error-handling.md](./55-error-handling.md) - エラーハンドリング
- [../specs/database.md](../specs/database.md) - DB設計・RLS

---

**最終更新**: YYYY-MM-DD

