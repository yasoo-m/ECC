# 依存性注入（DI）の基礎

依存性注入（DI）は、アプリケーションの異なる部分に機能を「注入」することで、コードを整理して共有するための設計パターンです。これにより、コードの保守性、スケーラビリティ、テスト容易性が向上します。

## Angular における DI の仕組み

コードが Angular の DI システムと相互作用する主な方法が 2 つあります：

1. **プロビジョン（提供）**：値（オブジェクト、関数、プリミティブ）を DI システムで利用可能にする。
2. **インジェクション（注入）**：DI システムにそれらの値を要求する。

Angular のコンポーネント、ディレクティブ、サービスは自動的に DI に参加します。

## サービス

**サービス**はアプリケーション全体でデータと機能を共有する最も一般的な方法です。`@Injectable()` でデコレートされた TypeScript クラスです。

### サービスの作成

`@Injectable` デコレーターで `providedIn: 'root'` オプションを使用すると、アプリケーション全体で利用可能なシングルトンサービスになります。これはほとんどのサービスに推奨されるアプローチです。

```ts
import {Injectable} from '@angular/core';

@Injectable({
  providedIn: 'root', // どこでも利用可能なシングルトンにする
})
export class AnalyticsLogger {
  trackEvent(category: string, value: string) {
    console.log('Analytics event logged:', {category, value});
  }
}
```

サービスの一般的な用途：

- データクライアント（API 呼び出し）
- 状態管理
- 認証と認可
- ロギングとエラーハンドリング
- ユーティリティ関数

## 依存関係の注入

依存関係を要求するには Angular の `inject()` 関数を使用します。

### `inject()` 関数

`inject()` 関数を使用してサービス（または提供された他のトークン）のインスタンスを取得できます。

```ts
import {Component, inject} from '@angular/core';
import {Router} from '@angular/router';
import {AnalyticsLogger} from './analytics-logger.service';

@Component({
  selector: 'app-navbar',
  template: `<a href="#" (click)="navigateToDetail($event)">Detail Page</a>`,
})
export class Navbar {
  // クラスフィールド初期化子を使用して依存関係を注入する
  private router = inject(Router);
  private analytics = inject(AnalyticsLogger);

  navigateToDetail(event: Event) {
    event.preventDefault();
    this.analytics.trackEvent('navigation', '/details');
    this.router.navigate(['/details']);
  }
}
```

### `inject()` はどこで使用できるか？（インジェクションコンテキスト）

`inject()` は**インジェクションコンテキスト**内でコードが実行されている場合に呼び出せます。最も一般的なインジェクションコンテキストは、コンポーネント、ディレクティブ、またはサービスの構築時です。

`inject()` を呼び出せる有効な場所：

1. **クラスフィールド初期化子**（推奨）
2. **コンストラクター本体**
3. **ルートガードとリゾルバー**（インジェクションコンテキスト内で実行される）
4. プロバイダーで使用される**ファクトリー関数**

```typescript
import {Component, Directive, Injectable, inject, ElementRef} from '@angular/core';
import {HttpClient} from '@angular/common/http';

// 1. コンポーネント内（フィールド初期化子とコンストラクター）
@Component({
  /*...*/
})
export class Example {
  private service1 = inject(MyService); // 有効なフィールド初期化子

  private service2: MyService;
  constructor() {
    this.service2 = inject(MyService); // 有効なコンストラクター本体
  }
}

// 2. ディレクティブ内
@Directive({
  /*...*/
})
export class MyDirective {
  private element = inject(ElementRef); // 有効なフィールド初期化子
}

// 3. サービス内
@Injectable({providedIn: 'root'})
export class MyService {
  private http = inject(HttpClient); // 有効なフィールド初期化子
}

// 4. ルートガード（関数型）内
export const authGuard = () => {
  const auth = inject(AuthService); // 有効なルートガード
  return auth.isAuthenticated();
};
```
