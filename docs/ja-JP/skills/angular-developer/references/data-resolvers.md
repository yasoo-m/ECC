# データリゾルバー

データリゾルバーはルートがアクティブになる前にデータをフェッチし、コンポーネントがレンダリング時に必要なデータを持っていることを保証します。

## リゾルバーの作成

`ResolveFn` 型を実装します。

```ts
export const userResolver: ResolveFn<User> = (route, state) => {
  const userService = inject(UserService);
  const id = route.paramMap.get('id')!;
  return userService.getUser(id);
};
```

## ルートの設定

`resolve` キーの下にリゾルバーを追加します。

```ts
{
  path: 'user/:id',
  component: UserProfile,
  resolve: {
    user: userResolver
  }
}
```

## 解決されたデータへのアクセス

### 1. `ActivatedRoute` 経由（従来の方法）

```ts
private route = inject(ActivatedRoute);
data = toSignal(this.route.data);
user = computed(() => this.data().user);
```

### 2. コンポーネント入力経由（モダンな方法）

`provideRouter` で `withComponentInputBinding()` を有効にすると、解決されたデータを `@Input` または `input()` に直接渡せます。

```ts
// app.config.ts
provideRouter(routes, withComponentInputBinding());

// component.ts
user = input.required<User>();
```

## エラーハンドリング

リゾルバーが失敗するとナビゲーションがブロックされます。

- グローバルな処理には `withNavigationErrorHandler` を使用します。
- リゾルバー内で `catchError` を使用して `RedirectCommand` やフォールバックデータを返します。

```ts
return userService
  .get(id)
  .pipe(catchError(() => of(new RedirectCommand(router.parseUrl('/error')))));
```

## ベストプラクティス

- **軽量に保つ**：重要なデータのみをフェッチします。
- **フィードバックを提供する**：リゾルバーが完了するまで UI は古いページに留まるため、ナビゲーション中にグローバルなローディングバーを表示するためにルーターイベントをリッスンします。
