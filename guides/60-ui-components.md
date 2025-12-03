# UIコンポーネント設計ガイド

{{PROJECT_NAME}} のUI設計原則とコンポーネント設計ガイドラインです。

<!-- 
📝 書くべき内容:
- デザイン原則（ブランドパーソナリティ）
- カラーパレット
- タイポグラフィ
- レイアウト・スペーシング
- コンポーネントの分類
- アニメーション・インタラクション

カスタマイズ:
- プロジェクトのデザインシステムに合わせて更新
- 使用しているUIライブラリの情報を追加
-->

## 📚 目次

1. [デザイン原則](#デザイン原則)
2. [カラーパレット](#カラーパレット)
3. [タイポグラフィ](#タイポグラフィ)
4. [レイアウト・スペーシング](#レイアウトスペーシング)
5. [コンポーネント分類](#コンポーネント分類)
6. [アニメーション](#アニメーション)

---

## デザイン原則

### ブランドパーソナリティ

<!-- プロジェクトのブランドイメージを記載 -->

| 特性 | 説明 |
|------|------|
| <!-- 特性1 --> | <!-- 説明 --> |
| <!-- 特性2 --> | <!-- 説明 --> |

例:
| 特性 | 説明 |
|------|------|
| Professional | 信頼感と専門性を重視 |
| Modern | 最新のデザイントレンドを取り入れる |
| Accessible | すべてのユーザーが使いやすい |

### 設計原則

1. **一貫性**: 同じ要素は同じ見た目・振る舞い
2. **シンプルさ**: 必要最小限の要素で構成
3. **アクセシビリティ**: WCAG 2.1 AA準拠を目指す
4. **レスポンシブ**: モバイルファーストで設計

---

## カラーパレット

### メインカラー

<!-- Tailwind CSS のクラス名で定義する場合 -->

| 名前 | 用途 | Tailwindクラス | HEX |
|------|------|---------------|-----|
| Primary | メインアクション、リンク | `bg-primary` | `#0066CC` |
| Secondary | サブアクション | `bg-secondary` | `#6B7280` |
| Accent | 強調、通知 | `bg-accent` | `#F59E0B` |

### セマンティックカラー

| 名前 | 用途 | Tailwindクラス |
|------|------|---------------|
| Success | 成功状態 | `text-green-600` |
| Warning | 警告状態 | `text-yellow-600` |
| Error | エラー状態 | `text-red-600` |
| Info | 情報 | `text-blue-600` |

### 背景・テキスト

| 名前 | ライトモード | ダークモード |
|------|------------|------------|
| Background | `bg-white` | `bg-gray-900` |
| Text Primary | `text-gray-900` | `text-gray-100` |
| Text Secondary | `text-gray-600` | `text-gray-400` |

---

## タイポグラフィ

### フォントファミリー

```css
/* Tailwind設定例 */
fontFamily: {
  sans: ['Inter', 'Noto Sans JP', 'sans-serif'],
  mono: ['Fira Code', 'monospace'],
}
```

### フォントサイズ

| 名前 | サイズ | 用途 |
|------|-------|------|
| xs | 12px | キャプション、補足 |
| sm | 14px | ラベル、ボタン |
| base | 16px | 本文 |
| lg | 18px | リード文 |
| xl | 20px | 見出し4 |
| 2xl | 24px | 見出し3 |
| 3xl | 30px | 見出し2 |
| 4xl | 36px | 見出し1 |

---

## レイアウト・スペーシング

### スペーシングスケール

```
4px単位でスケーリング
1 = 4px
2 = 8px
3 = 12px
4 = 16px
6 = 24px
8 = 32px
12 = 48px
16 = 64px
```

### レイアウトパターン

#### コンテナ幅

```typescript
// 最大幅の設定
const containerMaxWidth = {
  sm: '640px',   // スモールコンテンツ
  md: '768px',   // ミディアム
  lg: '1024px',  // ラージ
  xl: '1280px',  // エクストララージ
}
```

#### グリッドシステム

```tsx
// 12カラムグリッド
<div className="grid grid-cols-12 gap-4">
  <div className="col-span-8">メインコンテンツ</div>
  <div className="col-span-4">サイドバー</div>
</div>
```

---

## コンポーネント分類

### 1. 基本コンポーネント（`ui/`）

再利用可能な最小単位のコンポーネント。

```
src/components/ui/
├── button.tsx
├── input.tsx
├── card.tsx
├── dialog.tsx
└── ...
```

### 2. 機能コンポーネント（`features/`）

ビジネスロジックを含むコンポーネント。

```
src/components/features/
├── UserProfile/
├── ProjectList/
└── ...
```

### 3. レイアウトコンポーネント（`layouts/`）

ページ構造を定義するコンポーネント。

```
src/components/layouts/
├── Header/
├── Footer/
├── Sidebar/
└── MainLayout/
```

---

## アニメーション

### 基本原則

- **目的のあるアニメーション**: 意味のないアニメーションは避ける
- **控えめな動き**: 0.2〜0.3秒程度の短いアニメーション
- **アクセシビリティ**: `prefers-reduced-motion` を尊重

### 推奨値

```css
/* トランジション */
transition-duration: 200ms;
transition-timing-function: ease-out;

/* アニメーション */
animation-duration: 300ms;
animation-timing-function: ease-in-out;
```

### 実装例

```tsx
// Tailwind CSS でのアニメーション
<button className="transition-colors duration-200 hover:bg-primary-dark">
  Click me
</button>

// フェードイン
<div className="animate-in fade-in duration-300">
  Content
</div>
```

### reduced-motion対応

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 実装例

### ボタンコンポーネント

```tsx
// src/components/ui/button.tsx
import { cva, type VariantProps } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors',
  {
    variants: {
      variant: {
        primary: 'bg-primary text-white hover:bg-primary-dark',
        secondary: 'bg-secondary text-white hover:bg-secondary-dark',
        outline: 'border border-gray-300 hover:bg-gray-50',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4 text-base',
        lg: 'h-12 px-6 text-lg',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ variant, size, className, ...props }: ButtonProps) {
  return (
    <button className={buttonVariants({ variant, size, className })} {...props} />
  )
}
```

---

## 関連ドキュメント

- [30-implementation-patterns.md](./30-implementation-patterns.md) - 実装パターン
- [50-coding-standards.md](./50-coding-standards.md) - コーディング規約
- [80-testing.md](./80-testing.md) - コンポーネントテスト

---

**最終更新**: YYYY-MM-DD

