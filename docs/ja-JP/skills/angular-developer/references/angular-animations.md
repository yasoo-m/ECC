# Angular アニメーション

Angularで要素をアニメーションさせる場合は、**まず `package.json` でプロジェクトのAngularバージョンを確認してください**。
モダンなアプリケーション（**Angular v20.2以上**）では、`animate.enter` と `animate.leave` を使ったネイティブCSSを優先してください。古いアプリケーションでは、非推奨の `@angular/animations` パッケージを使用する必要がある場合があります。

## 1. ネイティブCSSアニメーション（v20.2以上 推奨）

モダンなAngularは `animate.enter` と `animate.leave` を提供し、要素がDOMに追加・削除される際にアニメーションさせます。これらは適切なタイミングでCSSクラスを適用します。

### `animate.enter` と `animate.leave`

要素に直接使用して、enter・leaveフェーズ中にCSSクラスを適用します。Angularはアニメーション完了時にenterクラスを自動的に削除します。`animate.leave` の場合、Angularはアニメーションが完了するまで要素をDOMから削除しません。

`animate.enter` の例：

```html
@if (isShown()) {
<div class="enter-container" animate.enter="enter-animation">
  <p>The box is entering.</p>
</div>
}
```

```css
/* トランジションを使用する場合は開始スタイルを定義してください */
.enter-container {
  border: 1px solid #dddddd;
  margin-top: 1em;
  padding: 20px;
  font-weight: bold;
  font-size: 20px;
}
.enter-container p {
  margin: 0;
}
.enter-animation {
  animation: slide-fade 1s;
}
@keyframes slide-fade {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

_注意: `animate.leave` は削除される子要素に追加できます。_

### イベントバインディングとサードパーティライブラリ

`(animate.enter)` と `(animate.leave)` にバインドして、関数を呼び出したりGSAPなどのJSライブラリを使用することができます。

```html
@if(show()) {
<div (animate.leave)="onLeave($event)">...</div>
}
```

```ts
import { AnimationCallbackEvent } from '@angular/core';

onLeave(event: AnimationCallbackEvent) {
  // カスタムアニメーションロジックをここに記述
  // 重要: Angularが要素を削除できるよう、完了時に必ず animationComplete() を呼び出してください！
  event.animationComplete();
}
```

## 2. 高度なCSSアニメーション

CSSは高度なアニメーションシーケンスのための強力なツールを提供しています。

### 状態とスタイルのアニメーション

プロパティバインディングを使用して要素のCSSクラスを切り替え、トランジションをトリガーします。

```html
<div [class.open]="isOpen">...</div>
```

```css
div {
  transition: height 0.3s ease-out;
  height: 100px;
}
div.open {
  height: 200px;
}
```

### Autoハイトのアニメーション

`css-grid` を使用してautoハイトへのアニメーションが可能です。

```css
.container {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.3s;
}
.container.open {
  grid-template-rows: 1fr;
}
.container > div {
  overflow: hidden;
}
```

### スタッガリングと並列アニメーション

- **スタッガリング**: リスト内のアイテムに異なる値の `animation-delay` または `transition-delay` を使用します。
- **並列**: `animation` ショートハンドで複数のアニメーションを適用します（例：`animation: rotate 3s, fade-in 2s;`）。

### プログラムによる制御

標準のWeb APIを使用してアニメーションを直接取得します：

```ts
const animations = element.getAnimations();
animations.forEach((anim) => anim.pause());
```

## 3. レガシーアニメーションDSL（非推奨）

古いプロジェクト（v20.2以前、または `@angular/animations` が既に大量に使用されているプロジェクト）では、コンポーネントメタデータDSLを使用します。

**重要:** レガシーアニメーションと `animate.enter`/`leave` を同じコンポーネント内で混在させないでください。

### セットアップ

```ts
bootstrapApplication(App, {
  providers: [provideAnimationsAsync()],
});
```

### トランジションの定義

```ts
import {signal} from '@angular/core';
import {trigger, state, style, animate, transition} from '@angular/animations';

@Component({
  animations: [
    trigger('openClose', [
      state('open', style({opacity: 1})),
      state('closed', style({opacity: 0})),
      transition('open <=> closed', [animate('0.5s')]),
    ]),
  ],
  template: `<div [@openClose]="isOpen() ? 'open' : 'closed'">...</div>`,
})
export class OpenClose {
  isOpen = signal(true);
}
```
