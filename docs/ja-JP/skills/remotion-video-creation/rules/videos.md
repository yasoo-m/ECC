---
name: videos
description: Remotionでビデオを埋め込む - トリミング、音量、速度、ループ、ピッチ
metadata:
  tags: video, media, trim, volume, speed, loop, pitch
---

# Remotionでビデオを使用する

## 前提条件

まず、@remotion/mediaパッケージをインストールする必要があります。
インストールされていない場合は、以下のコマンドを使用します：

```bash
npx remotion add @remotion/media # npmを使うプロジェクト
bunx remotion add @remotion/media # bunを使うプロジェクト
yarn remotion add @remotion/media # yarnを使うプロジェクト
pnpm exec remotion add @remotion/media # pnpmを使うプロジェクト
```

コンポジションにビデオを埋め込むには `@remotion/media` の `<Video>` を使用します。

```tsx
import { Video } from "@remotion/media";
import { staticFile } from "remotion";

export const MyComposition = () => {
  return <Video src={staticFile("video.mp4")} />;
};
```

リモートURLもサポートされています：

```tsx
<Video src="https://remotion.media/video.mp4" />
```

## トリミング

`trimBefore` と `trimAfter` を使ってビデオの一部を削除します。値は秒単位です。

```tsx
const { fps } = useVideoConfig();

return (
  <Video
    src={staticFile("video.mp4")}
    trimBefore={2 * fps} // 最初の2秒をスキップ
    trimAfter={10 * fps} // 10秒地点で終了
  />
);
```

## 遅延

ビデオが表示されるタイミングを遅らせるには `<Sequence>` でラップします：

```tsx
import { Sequence, staticFile } from "remotion";
import { Video } from "@remotion/media";

const { fps } = useVideoConfig();

return (
  <Sequence from={1 * fps}>
    <Video src={staticFile("video.mp4")} />
  </Sequence>
);
```

ビデオは1秒後に表示されます。

## サイズと位置

`style` プロパティでサイズと位置を制御します：

```tsx
<Video
  src={staticFile("video.mp4")}
  style={{
    width: 500,
    height: 300,
    position: "absolute",
    top: 100,
    left: 50,
    objectFit: "cover",
  }}
/>
```

## 音量

静的な音量を設定します（0〜1）：

```tsx
<Video src={staticFile("video.mp4")} volume={0.5} />
```

または現在のフレームに基づいた動的な音量のコールバックを使用します：

```tsx
import { interpolate } from "remotion";

const { fps } = useVideoConfig();

return (
  <Video
    src={staticFile("video.mp4")}
    volume={(f) =>
      interpolate(f, [0, 1 * fps], [0, 1], { extrapolateRight: "clamp" })
    }
  />
);
```

ビデオを完全にミュートするには `muted` を使用します：

```tsx
<Video src={staticFile("video.mp4")} muted />
```

## 速度

再生速度を変更するには `playbackRate` を使用します：

```tsx
<Video src={staticFile("video.mp4")} playbackRate={2} /> {/* 2倍速 */}
<Video src={staticFile("video.mp4")} playbackRate={0.5} /> {/* 半速 */}
```

逆再生はサポートされていません。

## ループ

ビデオを無限ループするには `loop` を使用します：

```tsx
<Video src={staticFile("video.mp4")} loop />
```

ループ時のフレームカウントの動作を制御するには `loopVolumeCurveBehavior` を使用します：

- `"repeat"`：フレームカウントが各ループでリセットされます（`volume` コールバック用）
- `"extend"`：フレームカウントが継続してインクリメントされます

```tsx
<Video
  src={staticFile("video.mp4")}
  loop
  loopVolumeCurveBehavior="extend"
  volume={(f) => interpolate(f, [0, 300], [1, 0])} // 複数ループにわたってフェードアウト
/>
```

## ピッチ

速度に影響せずにピッチを調整するには `toneFrequency` を使用します。値の範囲は0.01〜2です：

```tsx
<Video
  src={staticFile("video.mp4")}
  toneFrequency={1.5} // 高いピッチ
/>
<Video
  src={staticFile("video.mp4")}
  toneFrequency={0.8} // 低いピッチ
/>
```

ピッチシフトはサーバーサイドレンダリング時のみ機能し、Remotion Studioのプレビューや `<Player />` では機能しません。
