---
name: animations
description: Remotionの基本的なアニメーションスキル
metadata:
  tags: animations, transitions, frames, useCurrentFrame
---

すべてのアニメーションは `useCurrentFrame()` フックによって駆動される必要があります。
アニメーションは秒単位で記述し、`useVideoConfig()` の `fps` 値を掛け合わせてください。

```tsx
import { useCurrentFrame } from "remotion";

export const FadeIn = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = interpolate(frame, [0, 2 * fps], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <div style={{ opacity }}>Hello World!</div>
  );
};
```

CSSトランジションやアニメーションは禁止です - 正しくレンダリングされません。
TailwindのアニメーションクラスNameは禁止です - 正しくレンダリングされません。
