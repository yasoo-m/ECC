---
name: audio
description: Remotionでのオーディオとサウンドの使用 - インポート、トリミング、音量、速度、ピッチ
metadata:
  tags: audio, media, trim, volume, speed, loop, pitch, mute, sound, sfx
---

# Remotionでのオーディオの使用

## 前提条件

まず、@remotion/mediaパッケージをインストールする必要があります。
インストールされていない場合は、以下のコマンドを使用してください:

```bash
npx remotion add @remotion/media # プロジェクトがnpmを使用している場合
bunx remotion add @remotion/media # プロジェクトがbunを使用している場合
yarn remotion add @remotion/media # プロジェクトがyarnを使用している場合
pnpm exec remotion add @remotion/media # プロジェクトがpnpmを使用している場合
```

## オーディオのインポート

`@remotion/media` の `<Audio>` を使用してコンポジションにオーディオを追加します。

```tsx
import { Audio } from "@remotion/media";
import { staticFile } from "remotion";

export const MyComposition = () => {
  return <Audio src={staticFile("audio.mp3")} />;
};
```

リモートURLもサポートされています:

```tsx
<Audio src="https://remotion.media/audio.mp3" />
```

デフォルトでは、オーディオは最初から、フル音量でフルレングスで再生されます。
複数の `<Audio>` コンポーネントを追加することで、複数のオーディオトラックを重ねることができます。

## トリミング

`trimBefore` と `trimAfter` を使用してオーディオの一部を削除します。値はフレーム単位です。

```tsx
const { fps } = useVideoConfig();

return (
  <Audio
    src={staticFile("audio.mp3")}
    trimBefore={2 * fps} // 最初の2秒をスキップ
    trimAfter={10 * fps} // 10秒マークで終了
  />
);
```

オーディオはコンポジションの先頭から再生を開始しますが、指定された部分のみが再生されます。

## 遅延

`<Sequence>` でオーディオをラップして、開始タイミングを遅らせます:

```tsx
import { Sequence, staticFile } from "remotion";
import { Audio } from "@remotion/media";

const { fps } = useVideoConfig();

return (
  <Sequence from={1 * fps}>
    <Audio src={staticFile("audio.mp3")} />
  </Sequence>
);
```

オーディオは1秒後に再生を開始します。

## 音量

静的な音量を設定する（0から1）:

```tsx
<Audio src={staticFile("audio.mp3")} volume={0.5} />
```

または、現在のフレームに基づいた動的な音量にコールバックを使用する:

```tsx
import { interpolate } from "remotion";

const { fps } = useVideoConfig();

return (
  <Audio
    src={staticFile("audio.mp3")}
    volume={(f) =>
      interpolate(f, [0, 1 * fps], [0, 1], { extrapolateRight: "clamp" })
    }
  />
);
```

`f` の値はコンポジションのフレームではなく、オーディオが再生を開始するときに0から始まります。

## ミュート

`muted` を使用してオーディオを無音にします。動的に設定できます:

```tsx
const frame = useCurrentFrame();
const { fps } = useVideoConfig();

return (
  <Audio
    src={staticFile("audio.mp3")}
    muted={frame >= 2 * fps && frame <= 4 * fps} // 2秒から4秒の間をミュート
  />
);
```

## 速度

`playbackRate` を使用して再生速度を変更します:

```tsx
<Audio src={staticFile("audio.mp3")} playbackRate={2} /> {/* 2倍速 */}
<Audio src={staticFile("audio.mp3")} playbackRate={0.5} /> {/* 半分の速度 */}
```

逆再生はサポートされていません。

## ループ

`loop` を使用してオーディオを無限にループさせます:

```tsx
<Audio src={staticFile("audio.mp3")} loop />
```

`loopVolumeCurveBehavior` を使用して、ループ時のフレームカウントの動作を制御します:

- `"repeat"`: フレームカウントが各ループでリセットされる（デフォルト）
- `"extend"`: フレームカウントが継続して増加する

```tsx
<Audio
  src={staticFile("audio.mp3")}
  loop
  loopVolumeCurveBehavior="extend"
  volume={(f) => interpolate(f, [0, 300], [1, 0])} // 複数ループにわたってフェードアウト
/>
```

## ピッチ

`toneFrequency` を使用して速度に影響せずにピッチを調整します。値の範囲は0.01から2です:

```tsx
<Audio
  src={staticFile("audio.mp3")}
  toneFrequency={1.5} // ピッチを高くする
/>
<Audio
  src={staticFile("audio.mp3")}
  toneFrequency={0.8} // ピッチを低くする
/>
```

ピッチシフトはサーバーサイドレンダリング中のみ機能し、Remotion StudioのプレビューやPlayer内では機能しません。
