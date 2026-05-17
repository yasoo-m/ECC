---
name: can-decode
description: Mediabunnyを使用してブラウザで動画をデコードできるか確認する
metadata:
  tags: decode, validation, video, audio, compatibility, browser
---

# 動画がデコードできるか確認する

再生を試みる前に、Mediabunnyを使用してブラウザで動画をデコードできるか確認します。

## `canDecode()` 関数

この関数はどのプロジェクトにもコピー＆ペーストできます。

```tsx
import { Input, ALL_FORMATS, UrlSource } from "mediabunny";

export const canDecode = async (src: string) => {
  const input = new Input({
    formats: ALL_FORMATS,
    source: new UrlSource(src, {
      getRetryDelay: () => null,
    }),
  });

  try {
    await input.getFormat();
  } catch {
    return false;
  }

  const videoTrack = await input.getPrimaryVideoTrack();
  if (videoTrack && !(await videoTrack.canDecode())) {
    return false;
  }

  const audioTrack = await input.getPrimaryAudioTrack();
  if (audioTrack && !(await audioTrack.canDecode())) {
    return false;
  }

  return true;
};
```

## 使用方法

```tsx
const src = "https://remotion.media/video.mp4";
const isDecodable = await canDecode(src);

if (isDecodable) {
  console.log("Video can be decoded");
} else {
  console.log("Video cannot be decoded by this browser");
}
```

## Blobとの使用

ファイルのアップロードやドラッグ＆ドロップには `BlobSource` を使用します:

```tsx
import { Input, ALL_FORMATS, BlobSource } from "mediabunny";

export const canDecodeBlob = async (blob: Blob) => {
  const input = new Input({
    formats: ALL_FORMATS,
    source: new BlobSource(blob),
  });

  // 上記と同じバリデーションロジック
};
```
