---
name: charts
description: Remotionのチャートとデータビジュアライゼーションパターン。棒グラフ、円グラフ、ヒストグラム、プログレスバー、データ駆動アニメーションを作成するときに使用します。
metadata:
  tags: charts, data, visualization, bar-chart, pie-chart, graphs
---

# Remotionのチャート

Remotionでは通常のReactコード（HTMLとSVG）を使用して棒グラフを作成できます。D3.jsも使用できます。

## `useCurrentFrame()` によって駆動されないアニメーションの禁止

サードパーティライブラリのすべてのアニメーションを無効にしてください。
レンダリング中にちらつきが発生します。
代わりに、すべてのアニメーションを `useCurrentFrame()` から駆動させてください。

## 棒グラフのアニメーション

基本的な実装例については、[棒グラフの例](assets/charts/bar-chart.tsx)を参照してください。

### スタガードバー

バーの高さをアニメーションし、次のようにスタガーさせることができます:

```tsx
const STAGGER_DELAY = 5;
const frame = useCurrentFrame();
const {fps} = useVideoConfig();

const bars = data.map((item, i) => {
  const delay = i * STAGGER_DELAY;
  const height = spring({
    frame,
    fps,
    delay,
    config: {damping: 200},
  });
  return <div style={{height: height * item.value}} />;
});
```

## 円グラフのアニメーション

stroke-dashoffsetを使用してセグメントをアニメーションさせ、12時の位置から開始します。

```tsx
const frame = useCurrentFrame();
const {fps} = useVideoConfig();

const progress = interpolate(frame, [0, 100], [0, 1]);

const circumference = 2 * Math.PI * radius;
const segmentLength = (value / total) * circumference;
const offset = interpolate(progress, [0, 1], [segmentLength, 0]);

<circle r={radius} cx={center} cy={center} fill="none" stroke={color} strokeWidth={strokeWidth} strokeDasharray={`${segmentLength} ${circumference}`} strokeDashoffset={offset} transform={`rotate(-90 ${center} ${center})`} />;
```
