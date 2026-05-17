# `linkedSignal` による依存状態の管理

`linkedSignal` 関数を使うと、別の状態と本質的に連動した書き込み可能な状態を作成できます。入力や他のシグナルから導出されたデフォルト値を持ちつつ、ユーザーが独立して変更できる状態に最適です。

ソース状態が変化すると、`linkedSignal` は新たに計算された値にリセットされます。

## 基本的な使い方

ソースに基づいて再計算するだけでよい場合は、計算関数を渡してください。`linkedSignal` は `computed` のように機能しますが、得られるシグナルは書き込み可能です（`.set()` や `.update()` を呼び出せます）。

```ts
import { Component, signal, linkedSignal } from '@angular/core';

@Component({...})
export class ShippingMethodPicker {
  shippingOptions = signal(['Ground', 'Air', 'Sea']);

  // 最初のオプションをデフォルト値とする。
  // shippingOptions が変化すると、selectedOption は新しい最初のオプションにリセットされる。
  selectedOption = linkedSignal(() => this.shippingOptions()[0]);

  changeShipping(index: number) {
    // このシグナルは手動で更新することもできる！
    this.selectedOption.set(this.shippingOptions()[index]);
  }
}
```

## 高度な使い方：以前の状態を考慮する

ソース状態が変化したとき、ユーザーの手動選択がまだ有効であれば保持したい場合があります。その場合は、`source` と `computation` を持つオブジェクト構文を使います。

`computation` 関数は、ソースの新しい値と、以前のソース値および以前の `linkedSignal` 値を含む `previous` オブジェクトを受け取ります。

```ts
interface ShippingMethod { id: number; name: string; }

@Component({...})
export class ShippingMethodPicker {
  shippingOptions = signal<ShippingMethod[]>([
    {id: 0, name: 'Ground'}, {id: 1, name: 'Air'}, {id: 2, name: 'Sea'}
  ]);

  selectedOption = linkedSignal<ShippingMethod[], ShippingMethod>({
    source: this.shippingOptions,
    computation: (newOptions, previous) => {
      // 新たに読み込まれたオプションにユーザーが以前選択した
      // オプションが含まれていれば、そのまま選択を保持する。
      // そうでなければ、最初のオプションにリセットする。
      return newOptions.find(opt => opt.id === previous?.value.id) ?? newOptions[0];
    }
  });
}
```

### `linkedSignal` と `computed` と `effect` の使い分け

- `computed` を使う：状態が他の状態から**厳密に**導出されており、手動で更新すべきでない場合。
- `linkedSignal` を使う：状態は他の状態から導出されているが、ユーザーが**必ず**オーバーライドまたは手動更新できる必要がある場合。
- `effect` を使って一方の状態をもう一方に同期させることは**絶対にしない**。それはアンチパターンです。代わりに `computed` または `linkedSignal` を使ってください。
