# ルートガード

ルートガードは、ユーザーがルートへ遷移できるか、またはルートから離れられるかを制御します。

## ガードの種類

- **`CanActivate`**：ユーザーはこのルートにアクセスできるか？（例：認証チェック）
- **`CanActivateChild`**：ユーザーはこのルートの子にアクセスできるか？
- **`CanDeactivate`**：ユーザーはこのルートから離れられるか？（例：未保存の変更）
- **`CanMatch`**：このルートはマッチングの対象として考慮すべきか？（例：フィーチャーフラグ）`false` を返すと、ルーターは他のルートのチェックを継続します。

## ガードの作成

Angular 15 以降、ガードは通常、関数として定義します。

```ts
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    return true;
  }

  // ログインページへリダイレクト
  return router.parseUrl('/login');
};
```

## ガードの適用

ルート設定に配列として追加します。順番通りに実行されます。

```ts
{
  path: 'admin',
  component: Admin,
  canActivate: [authGuard],
  canActivateChild: [adminChildGuard],
  canDeactivate: [unsavedChangesGuard]
}
```

## 戻り値

- `boolean`：`true` で許可、`false` でブロック。
- `UrlTree` または `RedirectCommand`：別のルートへリダイレクト。
- `Observable` または `Promise`：上記の型に解決されます。

## セキュリティに関する注意

**クライアントサイドのガードはサーバーサイドのセキュリティの代わりにはなりません。** サーバー側で常に権限を検証してください。
