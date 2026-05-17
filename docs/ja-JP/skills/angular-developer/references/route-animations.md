# ルートトランジションアニメーション

Angular ルーターは、ルート間のスムーズなビジュアルトランジションのためにブラウザの **View Transitions API** をサポートしています。

## ビュートランジションの有効化

ルーター設定に `withViewTransitions()` を追加します。

```ts
provideRouter(routes, withViewTransitions());
```

これは**プログレッシブエンハンスメント**です。API をサポートしていないブラウザでも、ルーターは引き続き動作しますが、トランジションアニメーションは行われません。

## 仕組み

1. ブラウザが古い状態のスクリーンショットを撮ります。
2. ルーターが DOM を更新します（新しいコンポーネントをアクティベートします）。
3. ブラウザが新しい状態のスクリーンショットを撮ります。
4. ブラウザが2つの状態の間をアニメーションします。

## CSS によるカスタマイズ

トランジションは**グローバル CSS ファイル**でカスタマイズします（コンポーネントスコープの CSS ではありません）。

`::view-transition-old()` と `::view-transition-new()` 疑似要素を使用します。

```css
/* 例：クロスフェード + スライド */
::view-transition-old(root) {
  animation: 90ms cubic-bezier(0.4, 0, 1, 1) both fade-out;
}
::view-transition-new(root) {
  animation: 210ms cubic-bezier(0, 0, 0.2, 1) 90ms both fade-in;
}
```

## 高度な制御

`onViewTransitionCreated` を使用して、ナビゲーションコンテキストに基づいてトランジションをスキップしたり、動作をカスタマイズしたりします。

```ts
withViewTransitions({
  onViewTransitionCreated: ({transition, from, to}) => {
    // 特定のルートに対してアニメーションをスキップ
    if (to.url === '/no-animation') {
      transition.skipTransition();
    }
  },
});
```

## ベストプラクティス

- **グローバルスタイル**：ビューカプセル化の問題を避けるため、トランジションアニメーションは常に `styles.css` で定義してください。
- **ビュートランジション名**：ルートをまたいでスムーズにトランジションさせたい要素（ヘッダー画像など）には、一意の `view-transition-name` を割り当ててください。
