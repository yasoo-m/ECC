---
name: gif
description: RemotionでGIF、APNG、AVIF、WebPを表示する
metadata:
  tags: gif, animation, images, animated, apng, avif, webp
---

# Remotionでアニメーション画像を使用する

## 基本的な使い方

`<AnimatedImage>` を使用して、RemotionのタイムラインにGIF、APNG、AVIF、WebP画像を同期させて表示します：

```tsx
import {AnimatedImage, staticFile} from 'remotion';

export const MyComposition = () => {
  return <AnimatedImage src={staticFile('animation.gif')} width={500} height={500} />;
};
```

リモートURLもサポートされています（CORSが有効になっている必要があります）：

```tsx
<AnimatedImage src="https://example.com/animation.gif" width={500} height={500} />
```

## サイズとフィット

`fit` プロパティでコンテナへの画像の収め方を制御します：

```tsx
// 引き伸ばしてフィル（デフォルト）
<AnimatedImage src={staticFile("animation.gif")} width={500} height={300} fit="fill" />

// アスペクト比を維持してコンテナ内に収める
<AnimatedImage src={staticFile("animation.gif")} width={500} height={300} fit="contain" />

// コンテナを埋め、必要に応じてクロップ
<AnimatedImage src={staticFile("animation.gif")} width={500} height={300} fit="cover" />
```

## 再生速度

`playbackRate` でアニメーション速度を制御します：

```tsx
<AnimatedImage src={staticFile("animation.gif")} width={500} height={500} playbackRate={2} /> {/* 2倍速 */}
<AnimatedImage src={staticFile("animation.gif")} width={500} height={500} playbackRate={0.5} /> {/* 半速 */}
```

## ループの動作

アニメーション終了後の動作を制御します：

```tsx
// 無限ループ（デフォルト）
<AnimatedImage src={staticFile("animation.gif")} width={500} height={500} loopBehavior="loop" />

// 一度再生して最終フレームを表示
<AnimatedImage src={staticFile("animation.gif")} width={500} height={500} loopBehavior="pause-after-finish" />

// 一度再生してキャンバスをクリア
<AnimatedImage src={staticFile("animation.gif")} width={500} height={500} loopBehavior="clear-after-finish" />
```

## スタイリング

追加のCSSには `style` プロパティを使用します（サイズは `width` と `height` プロパティで指定）：

```tsx
<AnimatedImage
  src={staticFile('animation.gif')}
  width={500}
  height={500}
  style={{
    borderRadius: 20,
    position: 'absolute',
    top: 100,
    left: 50,
  }}
/>
```

## GIFの長さを取得する

`@remotion/gif` の `getGifDurationInSeconds()` を使用してGIFの長さを取得します。

```bash
npx remotion add @remotion/gif # npmを使うプロジェクト
bunx remotion add @remotion/gif # bunを使うプロジェクト
yarn remotion add @remotion/gif # yarnを使うプロジェクト
pnpm exec remotion add @remotion/gif # pnpmを使うプロジェクト
```

```tsx
import {getGifDurationInSeconds} from '@remotion/gif';
import {staticFile} from 'remotion';

const duration = await getGifDurationInSeconds(staticFile('animation.gif'));
console.log(duration); // 例: 2.5
```

GIFに合わせてコンポジションの長さを設定するのに便利です：

```tsx
import {getGifDurationInSeconds} from '@remotion/gif';
import {staticFile, CalculateMetadataFunction} from 'remotion';

const calculateMetadata: CalculateMetadataFunction = async () => {
  const duration = await getGifDurationInSeconds(staticFile('animation.gif'));
  return {
    durationInFrames: Math.ceil(duration * 30),
  };
};
```

## 代替手段

`<AnimatedImage>` が動作しない場合（ChromeとFirefoxのみサポート）、代わりに `@remotion/gif` の `<Gif>` を使用できます。

```bash
npx remotion add @remotion/gif # npmを使うプロジェクト
bunx remotion add @remotion/gif # bunを使うプロジェクト
yarn remotion add @remotion/gif # yarnを使うプロジェクト
pnpm exec remotion add @remotion/gif # pnpmを使うプロジェクト
```

```tsx
import {Gif} from '@remotion/gif';
import {staticFile} from 'remotion';

export const MyComposition = () => {
  return <Gif src={staticFile('animation.gif')} width={500} height={500} />;
};
```

`<Gif>` コンポーネントは `<AnimatedImage>` と同じプロパティを持ちますが、GIFファイルのみをサポートします。
