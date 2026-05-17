# インジェクションコンテキスト

`inject()` 関数はコードが**インジェクションコンテキスト**内で実行されている場合にのみ使用できます。

## インジェクションコンテキストが利用可能な場所

インジェクションコンテキストは以下の場所で自動的に利用可能です：

1. DI によってインスタンス化されるクラス（`@Injectable`、`@Component`、`@Directive`、`@Pipe`）の**フィールド初期化子**。
2. DI によってインスタンス化されるクラスの**コンストラクター**。
3. `useFactory` または `InjectionToken` 設定で指定された**ファクトリー関数**。
4. Angular によって実行される**関数型 API**（例：関数型ルートガード、リゾルバー、インターセプター）。

```ts
@Component({...})
export class Example {
  // 有効：フィールド初期化子
  private router = inject(Router);

  constructor() {
    // 有効：コンストラクター
    const http = inject(HttpClient);
  }

  onClick() {
    // 無効：インジェクションコンテキストではない
    // const auth = inject(AuthService);
  }
}
```

## `runInInjectionContext`

インジェクションコンテキスト内で関数を実行する必要がある場合（動的コンポーネントの作成やテストでよく必要になります）、`runInInjectionContext` を使用します。これには既存のインジェクター（`EnvironmentInjector` または `Injector` など）へのアクセスが必要です。

```ts
import {Injectable, inject, EnvironmentInjector, runInInjectionContext} from '@angular/core';

@Injectable({providedIn: 'root'})
export class MyService {
  private injector = inject(EnvironmentInjector);

  doSomethingDynamic() {
    runInInjectionContext(this.injector, () => {
      // ここで inject() を使用することが有効になる
      const router = inject(Router);
    });
  }
}
```

## `assertInInjectionContext`

ユーティリティ関数が有効なコンテキストから呼び出されることを保証するために `assertInInjectionContext` を使用します。そうでない場合は明確なエラーをスローします。

```ts
import {assertInInjectionContext, inject, ElementRef} from '@angular/core';

export function injectNativeElement<T extends Element>(): T {
  assertInInjectionContext(injectNativeElement);
  return inject(ElementRef).nativeElement;
}
```
