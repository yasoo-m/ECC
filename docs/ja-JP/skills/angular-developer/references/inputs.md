# 入力（Inputs）

入力により、親コンポーネントから子コンポーネントにデータを流すことができます。Angular はモダンなアプリケーションにはシグナルベースの `input` API の使用を推奨しています。

## シグナルベースの入力

`input()` 関数を使用して入力を宣言します。これは `InputSignal` を返します。

```ts
import {Component, input, computed} from '@angular/core';

@Component({
  selector: 'app-user',
  template: `<p>User: {{ name() }} ({{ age() }})</p>`,
})
export class User {
  // デフォルト値を持つオプションの入力
  name = input('Guest');

  // 必須の入力
  age = input.required<number>();

  // 入力はリアクティブなシグナル
  label = computed(() => `Name: ${this.name()}`);
}
```

### テンプレートでの使用

```html
<app-user [name]="userName" [age]="25" />
```

## 設定オプション

`input` 関数は設定オブジェクトを受け取ります：

- **エイリアス**：テンプレートで使用されるプロパティ名を変更します。
- **トランスフォーム**：コンポーネントに到達する前に値を変更します。

```ts
import { input, booleanAttribute } from '@angular/core';

@Component({...})
export class CustomButton {
  // エイリアスの例
  label = input('', { alias: 'btnLabel' });

  // 組み込みヘルパーを使用したトランスフォームの例
  disabled = input(false, { transform: booleanAttribute });
}
```

## モデル入力（双方向バインディング）

双方向データバインディングをサポートする入力を作成するには `model()` を使用します。

```ts
@Component({
  selector: 'custom-counter',
  template: `<button (click)="increment()">+</button>`,
})
export class CustomCounter {
  value = model(0);

  increment() {
    this.value.update((v) => v + 1);
  }
}
```

### 使用方法

```html
<!-- シグナルによる双方向バインディング -->
<custom-counter [(value)]="mySignal" />

<!-- プレーンプロパティによる双方向バインディング -->
<custom-counter [(value)]="myProperty" />
```

## デコレーターベースの入力（@Input）

レガシー API は引き続きサポートされていますが、新しいコードには推奨されません。

```ts
import { Component, Input } from '@angular/core';

@Component({...})
export class Legacy {
  @Input({ required: true }) value = 0;
  @Input({ transform: trimString }) label = '';
}
```

## ベストプラクティス

- **シグナルを優先する**：より良いリアクティビティと型安全性のために `@Input()` ではなく `input()` を使用します。
- **必須入力**：ビルド時エラーを得るために必須データには `input.required()` を使用します。
- **純粋なトランスフォーム**：入力トランスフォーム関数は純粋で静的に解析可能であることを確認します。
- **衝突を避ける**：標準の DOM プロパティと衝突する入力名は使用しないでください（例：`id`、`title`）。
