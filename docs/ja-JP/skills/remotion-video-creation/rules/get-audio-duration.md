---
name: get-audio-duration
description: Mediabunnyを使用してオーディオファイルのデュレーションを秒単位で取得する
metadata:
  tags: duration, audio, length, time, seconds, mp3, wav
---

# MediabunnyによるオーディオDurationの取得

Mediabunnyはオーディオファイルのデュレーションを抽出できます。ブラウザ、Node.js、Bun環境で動作します。

## オーディオDurationの取得

```tsx
import { Input, ALL_FORMATS, UrlSource } from "mediabunny";

export const getAudioDuration = async (src: string) => {
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

## 使用方法

```tsx
const duration = await getAudioDuration("https://remotion.media/audio.mp3");
console.log(duration); // 例: 180.5 (秒)
```

## ローカルファイルとの使用

ローカルファイルの場合は、`UrlSource` の代わりに `FileSource` を使用します:

```tsx
import { Input, ALL_FORMATS, FileSource } from "mediabunny";

const input = new Input({
  formats: ALL_FORMATS,
  source: new FileSource(file), // 入力またはドラッグ＆ドロップからのFileオブジェクト
});

const durationInSeconds = await input.computeDuration();
```

## RemotionのstaticFileとの使用

```tsx
import { staticFile } from "remotion";

const duration = await getAudioDuration(staticFile("audio.mp3"));
```
