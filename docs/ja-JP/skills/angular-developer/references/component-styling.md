# コンポーネントスタイリング

Angular コンポーネントは、カプセル化とモジュール性を実現するために、テンプレートに特定のスタイルを定義できます。

## スタイルの定義

スタイルはインラインまたは別ファイルで定義できます。

```ts
@Component({
  selector: 'app-photo',
  // インラインスタイル
  styles: `
    img {
      border-radius: 50%;
    }
  `,
  // または外部ファイル
  styleUrl: 'photo.component.css',
})
export class Photo {}
```

## ビューエンカプセレーション

すべてのコンポーネントには、スタイルのスコープを決定するビューエンカプセレーション設定があります。

| モード                            | 動作                                                                                      |
| :------------------------------ | :-------------------------------------------------------------------------------------------- |
| `Emulated`（デフォルト）            | 一意の HTML 属性を使用してスタイルをコンポーネントにスコープします。グローバルスタイルは引き続き影響する可能性があります。 |
| `ShadowDom`                     | ブラウザのネイティブ Shadow DOM API を使用してスタイルを完全に分離します。                        |
| `None`                          | カプセル化を無効にします。コンポーネントスタイルはグローバルになります。                       |
| `ExperimentalIsolatedShadowDom` | コンポーネントのスタイルのみが適用されることを厳密に保証します。                                   |

### 使用方法

```ts
import { ViewEncapsulation } from '@angular/core';

@Component({
  ...,
  encapsulation: ViewEncapsulation.None,
})
export class GlobalStyled {}
```

## 特殊セレクター

### `:host`

コンポーネントのホスト要素（コンポーネントのセレクターに一致する要素）を対象にします。

```css
:host {
  display: block;
  border: 1px solid black;
}
```

### `:host-context()`

先祖の何らかの条件に基づいてホスト要素を対象にします。

```css
/* 先祖に 'theme-dark' クラスがある場合にスタイルを適用する */
:host-context(.theme-dark) {
  background-color: #333;
}
```

### `::ng-deep`

特定のルールのビューエンカプセレーションを無効にし、子コンポーネントに「漏れ」させます。
**注意：Angular チームは `::ng-deep` の使用を強く推奨しません。** これは後方互換性のためにのみサポートされています。

## テンプレート内のスタイル

コンポーネントのテンプレートで直接 `<style>` 要素を使用できます。ビューエンカプセレーションのルールは引き続き適用されます。

```html
<style>
  .dynamic-class {
    color: red;
  }
</style>
<div class="dynamic-class">Hello</div>
```

## 外部スタイル

CSS での `<link>` や `@import` の使用は外部スタイルとして扱われます。**外部スタイルはエミュレートされたビューエンカプセレーションの影響を受けません。**
