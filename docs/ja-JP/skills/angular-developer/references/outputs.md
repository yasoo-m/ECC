# アウトレット（カスタムイベント）

アウトプットを使うと、子コンポーネントがカスタムイベントを発行でき、親コンポーネントがそれをリッスンできます。Angular はモダンなアプリケーションでは新しい `output()` 関数の使用を推奨しています。

## 関数ベースのアウトプット

`output()` 関数を使ってアウトプットを宣言します。これにより `OutputEmitterRef` が返されます。

```ts
import {Component, output} from '@angular/core';

@Component({
  selector: 'custom-slider',
  template: `<button (click)="changeValue(50)">Set to 50</button>`,
})
export class CustomSlider {
  // イベントデータなしのアウトプット
  panelClosed = output<void>();

  // イベントデータあり（number）のアウトプット
  valueChanged = output<number>();

  changeValue(newValue: number) {
    this.valueChanged.emit(newValue);
  }
}
```

### テンプレートでの使用

アウトプットイベントにバインドするには括弧 `()` を使います。イベントがデータを発行する場合、特別な `$event` 変数を使ってアクセスします。

```html
<custom-slider (panelClosed)="savePanelState()" (valueChanged)="logValue($event)" />
```

## 設定オプション

`output` 関数はエイリアスを指定するための設定オブジェクトを受け取ります。

```ts
@Component({...})
export class CustomSlider {
  // テンプレートでは 'valueChanged' という名前のイベントだが、
  // コンポーネントクラス内では 'changed' としてアクセスする。
  changed = output<number>({ alias: 'valueChanged' });
}
```

## プログラム的なサブスクリプション

コンポーネントを動的に生成する場合、アウトプットにプログラムでサブスクライブできます：

```ts
const componentRef = viewContainerRef.createComponent(CustomSlider);

const subscription = componentRef.instance.valueChanged.subscribe((val) => {
  console.log('Value changed:', val);
});

// 必要に応じて手動でクリーンアップ（Angular は破棄されたコンポーネントを自動でクリーンアップする）
subscription.unsubscribe();
```

## デコレーターベースのアウトプット（@Output）

レガシー API は `@Output()` デコレーターと `EventEmitter` を使用します。引き続きサポートされますが、新しいコードでは推奨されません。

```ts
import { Component, Output, EventEmitter } from '@angular/core';

@Component({...})
export class LegacyExample {
  @Output() valueChanged = new EventEmitter<number>();

  // エイリアスあり
  @Output('customEventName') changed = new EventEmitter<void>();
}
```

## ベストプラクティス

- **`output()` を優先する**：`@Output()` と `EventEmitter` の代わりに関数ベースの `output()` を使用してください。
- **命名**：アウトプット名には `camelCase` を使用します。`on` を前置することは避けてください（例：`onValueChanged` ではなく `valueChanged` を使用する）。
- **DOM バブリングなし**：Angular のカスタムイベントは、ネイティブイベントのように DOM ツリーをバブルアップしません。
- **衝突を避ける**：ネイティブ DOM イベント（`click` や `submit` など）と衝突する名前は選ばないでください。
