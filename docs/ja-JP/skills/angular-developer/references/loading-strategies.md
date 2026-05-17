# ルート読み込み戦略

Angular は、初期読み込み時間とナビゲーションの応答性のバランスを取るために、ルートとコンポーネントの読み込みに関する2つの主要な戦略をサポートしています。

## イーガーローディング（Eager Loading）

コンポーネントは初期 JavaScript ペイロードにバンドルされ、即座に利用可能になります。

```ts
{ path: 'home', component: Home }
```

- **メリット**：シームレスなトランジション。
- **デメリット**：初期バンドルサイズが増加する。

## レイジーローディング（Lazy Loading）

コンポーネントやルートは、ユーザーが画面に遷移したときにのみ読み込まれます。これにより JavaScript の独立した「チャンク」が生成されます。

### コンポーネントのレイジーローディング

`loadComponent` を使用して、コンポーネントをオンデマンドで取得します。

```ts
{
  path: 'admin',
  loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent)`,
}
```

### 子ルートのレイジーローディング

`loadChildren` を使用して、ルートのセットを取得します。

```ts
{
  path: 'settings',
  loadChildren: () => import('./settings/settings.routes'),
}
```

## インジェクションコンテキストとレイジーローディング

ローダー関数は現在のルートの**インジェクションコンテキスト**内で実行されます。これにより `inject()` を呼び出して、コンテキストを考慮した読み込み判断が可能になります。

```ts
{
  path: 'dashboard',
  loadComponent: () => {
    const flags = inject(FeatureFlags);
    return flags.isPremium
      ? import('./premium-dashboard')
      : import('./basic-dashboard');
  },
}
```

## 推奨事項

- メインのランディングページには**イーガーローディング**を使用する。
- 初期バンドルを小さく保つために、その他のすべての機能領域には**レイジーローディング**を使用する。
