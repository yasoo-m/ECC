# アウトレットを使ったルートの表示

`RouterOutlet` ディレクティブは、Angularが現在のURLに対応するコンポーネントをレンダリングするためのプレースホルダーです。

## 基本的な使い方

テンプレートに `<router-outlet />` を含めます。Angularはルーティングされたコンポーネントをアウトレットの直後の兄弟要素として挿入します。

```html
<app-header /> <router-outlet />
<!-- ルートのコンテンツがここに表示される -->
<app-footer />
```

## ネストされたアウトレット

子ルートには、親コンポーネントのテンプレート内に独自の `<router-outlet />` が必要です。

```ts
// 親コンポーネントのテンプレート
<h1>Settings</h1>
<router-outlet /> <!-- ProfileやSecurityなどの子コンポーネントがここにレンダリングされる -->
```

## 名前付きアウトレット（セカンダリルート）

ページには複数のアウトレットを設定できます。アウトレットに `name` を割り当てることで、特定のアウトレットをターゲットにできます。デフォルト名は `'primary'` です。

```html
<router-outlet />
<!-- プライマリ -->
<router-outlet name="sidebar" />
<!-- セカンダリ -->
```

ルート設定で `outlet` を定義します:

```ts
{
  path: 'chat',
  component: Chat,
  outlet: 'sidebar'
}
```

## アウトレットのライフサイクルイベント

`RouterOutlet` はコンポーネントが変更されるときにイベントを発行します:

- `activate`: 新しいコンポーネントがインスタンス化された。
- `deactivate`: コンポーネントが破棄された。
- `attach` / `detach`: `RouteReuseStrategy` と共に使用される。

```html
<router-outlet (activate)="onActivate($event)" />
```

## `routerOutletData` によるデータの受け渡し

`routerOutletData` インプットを使用して、ルーティングされたコンポーネントにコンテキストデータを渡すことができます。コンポーネントはシグナルとして `ROUTER_OUTLET_DATA` インジェクショントークン経由でこのデータにアクセスします。

```ts
// 親コンポーネント内
<router-outlet [routerOutletData]="{ theme: 'dark' }" />

// ルーティングされたコンポーネント内
outletData = inject(ROUTER_OUTLET_DATA) as Signal<{ theme: string }>;
```
