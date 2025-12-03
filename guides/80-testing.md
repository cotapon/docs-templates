# テスト・品質保証ガイド

{{PROJECT_NAME}} のテスト戦略と品質保証フローを説明します。

<!-- 
📝 書くべき内容:
- テストの種類と使い分け
- テストの書き方（Given-When-Then）
- カバレッジ目標
- テスト実行方法
- モック・スタブの使い方

★必須ファイル: すべてのプロジェクトで作成してください
-->

## 📚 目次

1. [テスト戦略](#テスト戦略)
2. [テストの書き方](#テストの書き方)
3. [テストの種類](#テストの種類)
4. [モック・スタブ](#モックスタブ)
5. [テスト実行](#テスト実行)

---

## テスト戦略

### テストピラミッド

```
        /\
       /  \
      / E2E \        少ない（重要フロー）
     /--------\
    /Integration\    中程度
   /--------------\
  /     Unit       \  多い（ロジック中心）
 /------------------\
```

### カバレッジ目標

| 種類 | 目標 | 説明 |
|------|------|------|
| ユニットテスト | 70%以上 | ビジネスロジック中心 |
| 統合テスト | 主要パス | API連携、DB操作 |
| E2Eテスト | クリティカルパス | ログイン、決済など |

### 何をテストするか

#### ✅ テストすべき

- ビジネスロジック
- エッジケース
- エラーハンドリング
- ユーザーインタラクション

#### ❌ テスト不要

- 外部ライブラリの動作
- 単純なgetter/setter
- 型定義のみのファイル

---

## テストの書き方

### Given-When-Then パターン

```typescript
describe('UserProfile', () => {
  describe('ユーザー情報の表示', () => {
    it('ユーザー名とメールアドレスを表示する', () => {
      // Given: テストデータとモックの準備
      const user = { name: '山田太郎', email: 'yamada@example.com' }

      // When: コンポーネントをレンダリング
      render(<UserProfile user={user} />)

      // Then: 期待する結果を検証
      expect(screen.getByText('山田太郎')).toBeInTheDocument()
      expect(screen.getByText('yamada@example.com')).toBeInTheDocument()
    })
  })
})
```

### 命名規則

```typescript
// describe: 対象を記述
describe('UserProfileContainer', () => {
  // describe: シナリオを記述
  describe('ユーザーが存在する場合', () => {
    // it: 期待する振る舞いを記述
    it('プロフィール情報を表示する', () => {
      // ...
    })
  })

  describe('ユーザーが存在しない場合', () => {
    it('エラーメッセージを表示する', () => {
      // ...
    })
  })
})
```

### アサーションのベストプラクティス

```typescript
// ✅ Good: 具体的なアサーション
expect(screen.getByRole('button', { name: '保存' })).toBeEnabled()

// ❌ Bad: 曖昧なアサーション
expect(button).toBeTruthy()
```

---

## テストの種類

### 1. ユニットテスト

単一の関数・コンポーネントをテスト。

#### Vitest

```typescript
// src/lib/utils/__tests__/format-date.spec.ts
import { describe, it, expect } from 'vitest'
import { formatDate } from '../format-date'

describe('formatDate', () => {
  it('日付をYYYY/MM/DD形式でフォーマットする', () => {
    const date = new Date('2024-01-15')
    expect(formatDate(date)).toBe('2024/01/15')
  })

  it('nullの場合は空文字を返す', () => {
    expect(formatDate(null)).toBe('')
  })
})
```

#### Jest

```typescript
// src/lib/utils/__tests__/format-date.test.ts
import { formatDate } from '../format-date'

describe('formatDate', () => {
  test('日付をYYYY/MM/DD形式でフォーマットする', () => {
    const date = new Date('2024-01-15')
    expect(formatDate(date)).toBe('2024/01/15')
  })

  test('nullの場合は空文字を返す', () => {
    expect(formatDate(null)).toBe('')
  })
})
```

> **Note**: Vitestは `describe`, `it`, `expect` をグローバルに使用可能（設定による）。Jestはデフォルトでグローバル。

### 2. コンポーネントテスト

UIコンポーネントの表示と振る舞いをテスト。

#### React (Testing Library)

```typescript
// src/components/ui/__tests__/Button.spec.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Button } from '../Button'

describe('Button', () => {
  it('クリックイベントを発火する', async () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>クリック</Button>)

    await userEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('disabled時はクリックできない', async () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick} disabled>クリック</Button>)

    await userEvent.click(screen.getByRole('button'))

    expect(handleClick).not.toHaveBeenCalled()
  })
})
```

#### Vue (Testing Library)

```typescript
// src/components/ui/__tests__/Button.spec.ts
import { render, screen } from '@testing-library/vue'
import userEvent from '@testing-library/user-event'
import Button from '../Button.vue'

describe('Button', () => {
  it('クリックイベントを発火する', async () => {
    const { emitted } = render(Button, {
      slots: { default: 'クリック' }
    })

    await userEvent.click(screen.getByRole('button'))

    expect(emitted()).toHaveProperty('click')
  })

  it('disabled時はクリックできない', async () => {
    const { emitted } = render(Button, {
      props: { disabled: true },
      slots: { default: 'クリック' }
    })

    await userEvent.click(screen.getByRole('button'))

    expect(emitted()).not.toHaveProperty('click')
  })
})
```

#### Svelte (Testing Library)

```typescript
// src/components/ui/__tests__/Button.spec.ts
import { render, screen } from '@testing-library/svelte'
import userEvent from '@testing-library/user-event'
import Button from '../Button.svelte'

describe('Button', () => {
  it('クリックイベントを発火する', async () => {
    const handleClick = vi.fn()
    render(Button, {
      props: { onclick: handleClick },
      // Svelte 5: slots are passed differently
    })

    await userEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

### 3. 統合テスト

複数のコンポーネントやモジュールの連携をテスト。

```typescript
// src/features/auth/__tests__/login-flow.spec.tsx
describe('ログインフロー', () => {
  it('正しい認証情報でログインできる', async () => {
    // モックAPIのセットアップ
    server.use(
      rest.post('/api/login', (req, res, ctx) => {
        return res(ctx.json({ token: 'mock-token' }))
      })
    )

    render(<LoginPage />)

    await userEvent.type(screen.getByLabelText('メールアドレス'), 'test@example.com')
    await userEvent.type(screen.getByLabelText('パスワード'), 'password123')
    await userEvent.click(screen.getByRole('button', { name: 'ログイン' }))

    await waitFor(() => {
      expect(screen.getByText('ダッシュボード')).toBeInTheDocument()
    })
  })
})
```

### 4. E2Eテスト

ブラウザでの実際のユーザー操作をテスト。

#### Playwright

```typescript
// tests/e2e/login.spec.ts
import { test, expect } from '@playwright/test'

test.describe('ログイン', () => {
  test('正常にログインできる', async ({ page }) => {
    await page.goto('/login')

    await page.fill('[name="email"]', 'test@example.com')
    await page.fill('[name="password"]', 'password123')
    await page.click('button[type="submit"]')

    await expect(page).toHaveURL('/dashboard')
    await expect(page.getByText('ようこそ')).toBeVisible()
  })
})
```

#### Cypress

```typescript
// cypress/e2e/login.cy.ts
describe('ログイン', () => {
  it('正常にログインできる', () => {
    cy.visit('/login')

    cy.get('[name="email"]').type('test@example.com')
    cy.get('[name="password"]').type('password123')
    cy.get('button[type="submit"]').click()

    cy.url().should('include', '/dashboard')
    cy.contains('ようこそ').should('be.visible')
  })

  it('不正な認証情報でエラーを表示する', () => {
    cy.visit('/login')

    cy.get('[name="email"]').type('wrong@example.com')
    cy.get('[name="password"]').type('wrongpassword')
    cy.get('button[type="submit"]').click()

    cy.contains('認証に失敗しました').should('be.visible')
  })
})
```

#### E2Eツール比較

| 特徴 | Playwright | Cypress |
|------|-----------|---------|
| マルチブラウザ | Chrome, Firefox, Safari, Edge | Chrome, Firefox, Edge |
| 並列実行 | ネイティブサポート | Cypress Cloud必要 |
| デバッグ | Trace Viewer | Time Travel |
| API テスト | サポート | `cy.request()` |

---

## モック・スタブ

### API モック（MSW）

#### MSW v2（推奨）

```typescript
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw'

export const handlers = [
  http.get('/api/users/:id', ({ params }) => {
    const { id } = params
    return HttpResponse.json({
      id,
      name: 'テストユーザー',
      email: 'test@example.com',
    })
  }),

  http.post('/api/users', async ({ request }) => {
    const body = await request.json()
    return HttpResponse.json({ id: '1', ...body }, { status: 201 })
  }),

  // エラーレスポンス
  http.get('/api/error', () => {
    return HttpResponse.json(
      { message: 'Internal Server Error' },
      { status: 500 }
    )
  }),
]
```

#### MSW v1（レガシー）

```typescript
// src/mocks/handlers.ts
import { rest } from 'msw'

export const handlers = [
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params
    return res(
      ctx.json({
        id,
        name: 'テストユーザー',
        email: 'test@example.com',
      })
    )
  }),
]
```

### 関数モック

```typescript
// vi.fn() を使用
const mockFn = vi.fn()
mockFn.mockReturnValue('mocked value')

// vi.spyOn() を使用
const spy = vi.spyOn(userService, 'getUser')
spy.mockResolvedValue({ id: '1', name: 'Test' })
```

### モジュールモック

```typescript
// モジュール全体をモック
vi.mock('@/lib/api-client', () => ({
  apiClient: {
    get: vi.fn(),
    post: vi.fn(),
  },
}))
```

---

## テスト実行

### コマンド

```bash
# ユニットテスト（Vitest）
npm run test              # 実行
npm run test:watch        # ウォッチモード
npm run test:coverage     # カバレッジ付き

# ユニットテスト（Jest）
npm test                  # 実行
npm test -- --watch       # ウォッチモード
npm test -- --coverage    # カバレッジ付き

# E2Eテスト（Playwright）
npm run test:e2e          # ヘッドレス実行
npm run test:e2e:ui       # UIモード（デバッグ用）
npx playwright test --debug  # デバッグモード

# E2Eテスト（Cypress）
npm run cypress:open      # インタラクティブモード
npm run cypress:run       # ヘッドレス実行
npx cypress run --spec "cypress/e2e/login.cy.ts"  # 特定ファイル実行
```

### CI/CD での実行

```yaml
# .github/workflows/test.yml
- name: Run unit tests
  run: npm run test:coverage

- name: Run E2E tests
  run: npm run test:e2e
```

### テストの前処理

```typescript
// vitest.setup.ts
import '@testing-library/jest-dom'
import { server } from './src/mocks/server'

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

---

## トラブルシューティング

### よくある問題

| 問題 | 解決策 |
|------|--------|
| テストがタイムアウト | `waitFor` のタイムアウトを延長 |
| 非同期処理が完了しない | `await` の追加、`act` で包む |
| モックが効かない | `vi.mock` をファイルの先頭に移動 |
| スナップショットの不一致 | `npm run test -- -u` で更新 |

---

## 関連ドキュメント

- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [55-error-handling.md](./55-error-handling.md) - エラーケースのテスト
- [90-development-workflow.md](./90-development-workflow.md) - 開発ワークフロー
- [../specs/examples.md](../specs/examples.md) - テストコードサンプル

---

**最終更新**: YYYY-MM-DD

