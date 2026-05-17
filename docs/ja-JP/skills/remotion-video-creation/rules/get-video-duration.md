---
name: get-video-duration
description: Mediabunnyを使ってビデオファイルの長さを秒単位で取得する
metadata:
  tags: duration, video, length, time, seconds
---

# Mediabunnyを使ったビデオ長さの取得

Mediabunnyはビデオファイルの長さを取得できます。ブラウザ、Node.js、Bun環境で動作します。

## ビデオの長さを取得する

```tsx
import { Input, ALL_FORMATS, UrlSource } from "mediabunny";

export const getVideoDuration = async (src: string) => {
  const input = new Input({
    formats: ALL_FORMATS,
    source: new UrlSource(src, {
      getRetryDelay: () => null,
    }),
  });

  const durationInSeconds = await input.computeDuration();
  return durationInSeconds;
};
```

## 使用例

```tsx
const duration = await getVideoDuration("https://remotion.media/video.mp4");
console.log(duration); // 例: 10.5 (秒)
```

## ローカルファイルで使用する

ローカルファイルの場合、`UrlSource` の代わりに `FileSource` を使用します：

```tsx
import { Input, ALL_FORMATS, FileSource } from "mediabunny";

const input = new Input({
  formats: ALL_FORMATS,
  source: new FileSource(file), // inputまたはドラッグ＆ドロップのFileオブジェクト
});

const durationInSeconds = await input.computeDuration();
```

## RemotionのstaticFileと組み合わせて使用する

```tsx
import { staticFile } from "remotion";

const duration = await getVideoDuration(staticFile("video.mp4"));
```
