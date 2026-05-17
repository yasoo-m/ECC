---
name: measuring-dom-nodes
description: RemotionでDOM要素のサイズを測定する
metadata:
  tags: measure, layout, dimensions, getBoundingClientRect, scale
---

# RemotionでDOMノードを測定する

Remotionはビデオコンテナに `scale()` トランスフォームを適用するため、`getBoundingClientRect()` から得られる値に影響します。正確な測定値を得るには `useCurrentScale()` を使用してください。

## 要素のサイズを測定する

```tsx
import { useCurrentScale } from "remotion";
import { useRef, useEffect, useState } from "react";

export const MyComponent = () => {
  const ref = useRef<HTMLDivElement>(null);
  const scale = useCurrentScale();
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });

  useEffect(() => {
    if (!ref.current) return;
    const rect = ref.current.getBoundingClientRect();
    setDimensions({
      width: rect.width / scale,
      height: rect.height / scale,
    });
  }, [scale]);

  return <div ref={ref}>測定するコンテンツ</div>;
};
```
