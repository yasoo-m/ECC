---
name: timing
description: Remotionの補間カーブ - 線形、イージング、スプリングアニメーション
metadata:
  tags: spring, bounce, easing, interpolation
---

シンプルな線形補間は `interpolate` 関数を使用して行います。

```ts title="100フレームかけて0から1へ"
import {interpolate} from 'remotion';

const opacity = interpolate(frame, [0, 100], [0, 1]);
```

デフォルトでは値はクランプされないため、[0, 1]の範囲外の値になることがあります。
クランプする方法は以下の通りです：

```ts title="外挿ありで100フレームかけて0から1へ"
const opacity = interpolate(frame, [0, 100], [0, 1], {
  extrapolateRight: 'clamp',
  extrapolateLeft: 'clamp',
});
```

## スプリングアニメーション

スプリングアニメーションはより自然な動きになります。
時間の経過とともに0から1へ変化します。

```ts title="100フレームかけて0から1へのスプリングアニメーション"
import {spring, useCurrentFrame, useVideoConfig} from 'remotion';

const frame = useCurrentFrame();
const {fps} = useVideoConfig();

const scale = spring({
  frame,
  fps,
});
```

### 物理プロパティ

デフォルト設定は `mass: 1, damping: 10, stiffness: 100` です。
これにより、アニメーションは落ち着く前に少しバウンスします。

設定は以下のように上書きできます：

```ts
const scale = spring({
  frame,
  fps,
  config: {damping: 200},
});
```

バウンスなしの自然な動きに推奨される設定は `{ damping: 200 }` です。

よく使われる設定：

```tsx
const smooth = {damping: 200}; // スムーズ、バウンスなし（繊細な表示）
const snappy = {damping: 20, stiffness: 200}; // スナッピー、最小限のバウンス（UI要素）
const bouncy = {damping: 8}; // バウンシーな入場（遊び心のあるアニメーション）
const heavy = {damping: 15, stiffness: 80, mass: 2}; // 重い、ゆっくり、小さなバウンス
```

### 遅延

アニメーションはデフォルトで即座に開始されます。
`delay` パラメータを使って指定フレーム数だけアニメーションを遅らせます。

```tsx
const entrance = spring({
  frame: frame - ENTRANCE_DELAY,
  fps,
  delay: 20,
});
```

### 長さ

`spring()` は物理プロパティに基づいた自然な長さを持ちます。
特定の長さにアニメーションを引き伸ばすには `durationInFrames` パラメータを使用します。

```tsx
const spring = spring({
  frame,
  fps,
  durationInFrames: 40,
});
```

### spring()とinterpolate()の組み合わせ

スプリングの出力（0-1）をカスタム範囲にマッピングします：

```tsx
const springProgress = spring({
  frame,
  fps,
});

// 回転にマッピング
const rotation = interpolate(springProgress, [0, 1], [0, 360]);

<div style={{rotate: rotation + 'deg'}} />;
```

### スプリングの加算

スプリングは数値を返すだけなので、算術演算が可能です：

```tsx
const frame = useCurrentFrame();
const {fps, durationInFrames} = useVideoConfig();

const inAnimation = spring({
  frame,
  fps,
});
const outAnimation = spring({
  frame,
  fps,
  durationInFrames: 1 * fps,
  delay: durationInFrames - 1 * fps,
});

const scale = inAnimation - outAnimation;
```

## イージング

`interpolate` 関数にイージングを追加できます：

```ts
import {interpolate, Easing} from 'remotion';

const value1 = interpolate(frame, [0, 100], [0, 1], {
  easing: Easing.inOut(Easing.quad),
  extrapolateLeft: 'clamp',
  extrapolateRight: 'clamp',
});
```

デフォルトのイージングは `Easing.linear` です。
その他の凸性：

- `Easing.in` - ゆっくり開始して加速
- `Easing.out` - 速く開始して減速
- `Easing.inOut`

カーブ（最も線形から最も曲線的な順）：

- `Easing.quad`
- `Easing.sin`
- `Easing.exp`
- `Easing.circle`

イージング関数を作るには凸性とカーブを組み合わせる必要があります：

```ts
const value1 = interpolate(frame, [0, 100], [0, 1], {
  easing: Easing.inOut(Easing.quad),
  extrapolateLeft: 'clamp',
  extrapolateRight: 'clamp',
});
```

3次ベジェ曲線もサポートされています：

```ts
const value1 = interpolate(frame, [0, 100], [0, 1], {
  easing: Easing.bezier(0.8, 0.22, 0.96, 0.65),
  extrapolateLeft: 'clamp',
  extrapolateRight: 'clamp',
});
```
