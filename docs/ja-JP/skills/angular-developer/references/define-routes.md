# ルートの定義

ルートは特定の URL パスに対してどのコンポーネントをレンダリングするかを定義するオブジェクトです。

## 基本設定

`Routes` 配列にルートを定義し、`appConfig` の `provideRouter` を使用して提供します。

```ts
// app.routes.ts
export const routes: Routes = [
  {path: '', component: HomePage},
  {path: 'admin', component: AdminPage},
];

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
```

## URL パス

- **静的**：完全な文字列に一致します（例：`'admin'`）。
- **ルートパラメーター**：コロンが前置された動的セグメント（例：`'user/:id'`）。
- **ワイルドカード**：`**` を使用して任意の URL に一致します。「見つかりません」ページに有用です。**常に配列の最後に配置してください。**

## マッチング戦略

Angular は**先勝ち**戦略を使用します。具体的なルートは具体性の低いルートより前に記述する必要があります。

## リダイレクト

`redirectTo` を使用して一つのパスを別のパスに向けます。

```ts
{ path: 'articles', redirectTo: '/blog' },
{ path: 'blog', component: Blog },
```

## ページタイトル

アクセシビリティのためにルートにタイトルを関連付けます。タイトルは静的または動的（`ResolveFn` またはカスタム `TitleStrategy` 経由）にできます。

```ts
{ path: 'home', component: Home, title: 'Home Page' }
```

## ルートデータとプロバイダー

- **静的データ**：`data` プロパティを使用してメタデータを付加します。
- **ルートプロバイダー**：`providers` 配列を使用して特定のルートとその子に依存関係をスコープします。

## ネスト（子）ルート

`children` プロパティを使用してサブビューを定義します。親コンポーネントには `<router-outlet />` が必要です。

```ts
{
  path: 'product/:id',
  component: Product,
  children: [
    { path: 'info', component: ProductInfo },
    { path: 'reviews', component: ProductReviews },
  ],
}
```
