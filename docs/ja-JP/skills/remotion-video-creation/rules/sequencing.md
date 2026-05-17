---
name: sequencing
description: Remotionのシーケンスパターン - アイテムの遅延、トリム、長さの制限
metadata:
  tags: sequence, series, timing, delay, trim
---

`<Sequence>` を使用して、タイムライン上で要素が表示されるタイミングを遅らせます。

```tsx
import { Sequence } from "remotion";

const {fps} = useVideoConfig();

<Sequence from={1 * fps} durationInFrames={2 * fps} premountFor={1 * fps}>
  <Title />
</Sequence>
<Sequence from={2 * fps} durationInFrames={2 * fps} premountFor={1 * fps}>
  <Subtitle />
</Sequence>
```

デフォルトでは、コンポーネントをabsolute fillの要素でラップします。
アイテムをラップしない場合は `layout` プロパティを使用します：

```tsx
<Sequence layout="none">
  <Title />
</Sequence>
```

## プリマウント

これはコンポーネントが実際に再生される前にタイムラインに読み込みます。
`<Sequence>` には必ずプリマウントを設定してください！

```tsx
<Sequence premountFor={1 * fps}>
  <Title />
</Sequence>
```

## Series

要素が重複なく順番に再生される場合は `<Series>` を使用します。

```tsx
import {Series} from 'remotion';

<Series>
  <Series.Sequence durationInFrames={45}>
    <Intro />
  </Series.Sequence>
  <Series.Sequence durationInFrames={60}>
    <MainContent />
  </Series.Sequence>
  <Series.Sequence durationInFrames={30}>
    <Outro />
  </Series.Sequence>
</Series>;
```

`<Sequence>` と同様に、`layout` プロパティが `none` に設定されていない限り、`<Series.Sequence>` 使用時にアイテムはデフォルトでabsolute fill要素でラップされます。

### オーバーラップありのSeries

シーケンスを重ねるには負のオフセットを使用します：

```tsx
<Series>
  <Series.Sequence durationInFrames={60}>
    <SceneA />
  </Series.Sequence>
  <Series.Sequence offset={-15} durationInFrames={60}>
    {/* SceneAが終わる15フレーム前に開始 */}
    <SceneB />
  </Series.Sequence>
</Series>
```

## Sequence内のフレーム参照

Sequence内では、`useCurrentFrame()` はローカルフレーム（0から開始）を返します：

```tsx
<Sequence from={60} durationInFrames={30}>
  <MyComponent />
  {/* MyComponent内では、useCurrentFrame()は60-89ではなく0-29を返す */}
</Sequence>
```

## ネストされたSequence

複雑なタイミングのためにSequenceをネストできます：

```tsx
<Sequence from={0} durationInFrames={120}>
  <Background />
  <Sequence from={15} durationInFrames={90} layout="none">
    <Title />
  </Sequence>
  <Sequence from={45} durationInFrames={60} layout="none">
    <Subtitle />
  </Sequence>
</Sequence>
```
