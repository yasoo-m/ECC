---
name: display-captions
description: TikTokスタイルのページと単語ハイライトによるRemotionでのキャプション表示
metadata:
  tags: captions, subtitles, display, tiktok, highlight
---

# Remotionでのキャプション表示

このガイドでは、すでに `Caption` フォーマットでキャプションを持っていることを前提に、Remotionでキャプションを表示する方法を説明します。

## 前提条件

まず、@remotion/captionsパッケージをインストールする必要があります。
インストールされていない場合は、以下のコマンドを使用してください:

```bash
npx remotion add @remotion/captions # プロジェクトがnpmを使用している場合
bunx remotion add @remotion/captions # プロジェクトがbunを使用している場合
yarn remotion add @remotion/captions # プロジェクトがyarnを使用している場合
pnpm exec remotion add @remotion/captions # プロジェクトがpnpmを使用している場合
```

## ページの作成

`createTikTokStyleCaptions()` を使用してキャプションをページにグループ化します。`combineTokensWithinMilliseconds` オプションは一度に表示される単語数を制御します:

```tsx
import {useMemo} from 'react';
import {createTikTokStyleCaptions} from '@remotion/captions';
import type {Caption} from '@remotion/captions';

// キャプションを切り替える頻度（ミリ秒単位）
// 値が大きいほど = 1ページあたりの単語数が多い
// 値が小さいほど = 単語数が少ない（より単語ごとに表示される）
const SWITCH_CAPTIONS_EVERY_MS = 1200;

const {pages} = useMemo(() => {
  return createTikTokStyleCaptions({
    captions,
    combineTokensWithinMilliseconds: SWITCH_CAPTIONS_EVERY_MS,
  });
}, [captions]);
```

## シーケンスを使ったレンダリング

ページをマップし、各ページを `<Sequence>` 内でレンダリングします。ページのタイミングから開始フレームとデュレーションを計算します:

```tsx
import {Sequence, useVideoConfig, AbsoluteFill} from 'remotion';
import type {TikTokPage} from '@remotion/captions';

const CaptionedContent: React.FC = () => {
  const {fps} = useVideoConfig();

  return (
    <AbsoluteFill>
      {pages.map((page, index) => {
        const nextPage = pages[index + 1] ?? null;
        const startFrame = (page.startMs / 1000) * fps;
        const endFrame = Math.min(
          nextPage ? (nextPage.startMs / 1000) * fps : Infinity,
          startFrame + (SWITCH_CAPTIONS_EVERY_MS / 1000) * fps,
        );
        const durationInFrames = endFrame - startFrame;

        if (durationInFrames <= 0) {
          return null;
        }

        return (
          <Sequence
            key={index}
            from={startFrame}
            durationInFrames={durationInFrames}
          >
            <CaptionPage page={page} />
          </Sequence>
        );
      })}
    </AbsoluteFill>
  );
};
```

## 単語のハイライト

キャプションページには `tokens` が含まれており、現在発話中の単語をハイライトするために使用できます:

```tsx
import {AbsoluteFill, useCurrentFrame, useVideoConfig} from 'remotion';
import type {TikTokPage} from '@remotion/captions';

const HIGHLIGHT_COLOR = '#39E508';

const CaptionPage: React.FC<{page: TikTokPage}> = ({page}) => {
  const frame = useCurrentFrame();
  const {fps} = useVideoConfig();

  // シーケンス開始からの相対的な現在時刻
  const currentTimeMs = (frame / fps) * 1000;
  // ページ開始時刻を加算して絶対時刻に変換
  const absoluteTimeMs = page.startMs + currentTimeMs;

  return (
    <AbsoluteFill style={{justifyContent: 'center', alignItems: 'center'}}>
      <div style={{fontSize: 80, fontWeight: 'bold', whiteSpace: 'pre'}}>
        {page.tokens.map((token) => {
          const isActive =
            token.fromMs <= absoluteTimeMs && token.toMs > absoluteTimeMs;

          return (
            <span
              key={token.fromMs}
              style={{color: isActive ? HIGHLIGHT_COLOR : 'white'}}
            >
              {token.text}
            </span>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```
