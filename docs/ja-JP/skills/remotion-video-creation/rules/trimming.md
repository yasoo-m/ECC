---
name: trimming
description: Remotionのトリミングパターン - アニメーションの開始・終了のカット
metadata:
  tags: sequence, trim, clip, cut, offset
---

`<Sequence>` に負の `from` 値を使用してアニメーションの開始をトリムします。

## 先頭のトリム

負の `from` 値は時間を後ろにずらし、アニメーションが途中から始まるようにします：

```tsx
import { Sequence, useVideoConfig } from "remotion";

const fps = useVideoConfig();

<Sequence from={-0.5 * fps}>
  <MyAnimation />
</Sequence>
```

アニメーションはその進行の15フレーム目から表示されます - 最初の15フレームがトリムされます。
`<MyAnimation>` 内では、`useCurrentFrame()` は0ではなく15から始まります。

## 末尾のトリム

指定した長さの後にコンテンツをアンマウントするには `durationInFrames` を使用します：

```tsx

<Sequence durationInFrames={1.5 * fps}>
  <MyAnimation />
</Sequence>
```

アニメーションは45フレーム再生され、その後コンポーネントがアンマウントされます。

## トリムと遅延

Sequenceをネストして開始のトリムと表示タイミングの遅延を組み合わせます：

```tsx
<Sequence from={30}>
  <Sequence from={-15}>
    <MyAnimation />
  </Sequence>
</Sequence>
```

内側のSequenceは開始から15フレームをトリムし、外側のSequenceは結果を30フレーム遅延させます。
