# サービスの作成と使用

Angular のサービスは、複数のコンポーネントや他のサービスがアクセスする必要があるデータフェッチ、ビジネスロジック、または状態管理を扱う再利用可能なコードです。

## サービスの作成

Angular CLI を使用してサービスを生成できます：

```bash
ng generate service my-data
```

または、TypeScript クラスを手動で作成して `@Injectable()` でデコレートすることもできます。

```ts
import {Injectable} from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class BasicDataStore {
  private data: string[] = [];

  addData(item: string): void {
    this.data.push(item);
  }

  getData(): string[] {
    return [...this.data];
  }
}
```

### `providedIn: 'root'` オプション

ほとんどのサービスには `providedIn: 'root'` の使用が推奨されます。これにより Angular は以下を行います：

- アプリケーション全体に対して**シングルインスタンス（シングルトン）を作成**します。
- `providers` 配列に列挙しなくても**どこからでも自動的に利用可能**にします。
- **ツリーシェイキングを有効**にし、サービスが実際にどこかで注入されている場合にのみ最終的な JavaScript バンドルに含まれるようにします。

## サービスの注入

サービスを作成したら、`inject()` 関数を使用してコンポーネント、ディレクティブ、または他のサービスに注入できます。

### コンポーネントへの注入

```ts
import {Component, inject} from '@angular/core';
import {BasicDataStore} from './basic-data-store.service';

@Component({
  selector: 'app-example',
  template: `
    <div>
      <p>Data items: {{ dataStore.getData().length }}</p>
      <button (click)="dataStore.addData('New Item')">Add Item</button>
    </div>
  `,
})
export class Example {
  // サービスをクラスフィールドとして注入する
  dataStore = inject(BasicDataStore);
}
```

### 別のサービスへの注入

サービスはまったく同じ方法で他のサービスを注入できます。

```ts
import {Injectable, inject} from '@angular/core';
import {AdvancedDataStore} from './advanced-data-store.service';

@Injectable({
  providedIn: 'root',
})
export class BasicDataStore {
  // 別のサービスを注入する
  private advancedDataStore = inject(AdvancedDataStore);

  private data: string[] = [];

  getData(): string[] {
    // このサービスと注入されたサービスのデータを結合する
    return [...this.data, ...this.advancedDataStore.getData()];
  }
}
```

## 高度なサービスパターン

`providedIn: 'root'` がほとんどのシナリオをカバーしていますが、以下のような場合に他の方法が必要になることがあります：

- **コンポーネント固有のインスタンス**：コンポーネントがサービスの独立したインスタンスを必要とする場合、コンポーネントの `@Component({ providers: [MyService] })` 配列で直接提供します。
- **ファクトリープロバイダー**：動的な作成のため。
- **値プロバイダー**：設定オブジェクトを注入するため。
