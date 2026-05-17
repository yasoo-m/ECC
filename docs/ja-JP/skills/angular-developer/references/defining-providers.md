# 依存性プロバイダーの定義

Angular は依存性注入（DI）システムに依存関係を提供する自動的な方法と手動の方法を提供します。

## 自動プロビジョン

サービスを提供する最も一般的な方法は、`@Injectable()` に `providedIn: 'root'` を使用することです。

### InjectionToken

非クラスの依存関係（設定オブジェクト、関数、プリミティブ）には `InjectionToken` を使用します。`InjectionToken` は自動的に提供することもできます。

```ts
import {InjectionToken} from '@angular/core';

export interface AppConfig {
  apiUrl: string;
}

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config', {
  providedIn: 'root',
  factory: () => ({apiUrl: 'https://api.example.com'}),
});
```

## 手動プロビジョン

`providedIn` のないサービス、特定のコンポーネント用の新しいインスタンスが必要な場合、またはランタイム値を設定する場合に `providers` 配列を使用します。

```ts
@Component({
  providers: [
    // { provide: LocalService, useClass: LocalService } の省略形
    LocalService,

    // useClass: 実装を切り替える
    {provide: Logger, useClass: BetterLogger},

    // useValue: 静的な値を提供する
    {provide: API_URL_TOKEN, useValue: 'https://api.example.com'},

    // useFactory: 値を動的に生成する
    {
      provide: ApiClient,
      useFactory: (http = inject(HttpClient)) => new ApiClient(http),
    },

    // useExisting: エイリアスを作成する
    {provide: OldLogger, useExisting: NewLogger},

    // multi: 同じトークンに複数の値を配列として提供する
    {provide: INTERCEPTOR_TOKEN, useClass: AuthInterceptor, multi: true},
  ],
})
export class Example {}
```

## プロバイダーのスコープ

- **アプリケーションブートストラップ**：グローバルシングルトン。HTTP クライアント、ロギング、アプリ全体の設定に使用します。
- **コンポーネント/ディレクティブ**：分離されたインスタンス。コンポーネント固有の状態やフォームに使用します。サービスはコンポーネントが破棄されると破棄されます。
- **ルート**：特定のルートでのみロードされる機能固有のサービス。

## ライブラリパターン：`provide*` 関数

ライブラリ作者は設定をカプセル化するためにプロバイダー配列を返す関数をエクスポートする必要があります：

```ts
export function provideAnalytics(config: AnalyticsConfig): Provider[] {
  return [{provide: ANALYTICS_CONFIG, useValue: config}, AnalyticsService];
}
```
