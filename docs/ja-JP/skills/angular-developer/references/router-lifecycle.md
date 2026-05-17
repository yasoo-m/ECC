# ルーターのライフサイクルとイベント

Angular ルーターは `Router.events` オブザーバブルを通じてイベントを発行し、開始から終了までのナビゲーションのライフサイクルを追跡できます。

## 主要なルーターイベント（時系列順）

1. **`NavigationStart`**：ナビゲーションが開始されます。
2. **`RoutesRecognized`**：ルーターが URL をルートにマッチさせます。
3. **`GuardsCheckStart` / `End`**：`canActivate`、`canMatch` などの評価が行われます。
4. **`ResolveStart` / `End`**：データ解決フェーズ（リゾルバーによるデータの取得）。
5. **`NavigationEnd`**：ナビゲーションが正常に完了しました。
6. **`NavigationCancel`**：ナビゲーションがキャンセルされました（例：ガードが `false` を返した）。
7. **`NavigationError`**：ナビゲーションが失敗しました（例：リゾルバーでのエラー）。

## イベントのサブスクライブ

`Router` を注入して `events` オブザーバブルをフィルタリングします。

```ts
import {Router, NavigationStart, NavigationEnd} from '@angular/router';

export class MyService {
  private router = inject(Router);

  constructor() {
    this.router.events.pipe(filter((e) => e instanceof NavigationEnd)).subscribe((event) => {
      console.log('Navigated to:', event.url);
    });
  }
}
```

## デバッグ

アプリケーション起動時にすべてのルーティングイベントの詳細なコンソールログを有効化します。

```ts
provideRouter(routes, withDebugTracing());
```

## よくあるユースケース

- **ローディングインジケーター**：`NavigationStart` が発火したときにスピナーを表示し、`NavigationEnd`/`Cancel`/`Error` で非表示にします。
- **アナリティクス**：`NavigationEnd` をリッスンしてページビューを追跡します。
- **スクロール管理**：カスタムスクロール動作のために `Scroll` イベントに応答します。
