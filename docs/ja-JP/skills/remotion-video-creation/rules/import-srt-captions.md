---
name: import-srt-captions
description: @remotion/captionsを使ってRemotionに.srt字幕ファイルをインポートする
metadata:
  tags: captions, subtitles, srt, import, parse
---

# Remotionへの.srt字幕のインポート

既存の `.srt` 字幕ファイルがある場合、`@remotion/captions` の `parseSrt()` を使ってRemotionにインポートできます。

## 前提条件

まず、@remotion/captionsパッケージをインストールする必要があります。
インストールされていない場合は、以下のコマンドを使用します：

```bash
npx remotion add @remotion/captions # npmを使うプロジェクト
bunx remotion add @remotion/captions # bunを使うプロジェクト
yarn remotion add @remotion/captions # yarnを使うプロジェクト
pnpm exec remotion add @remotion/captions # pnpmを使うプロジェクト
```

## .srtファイルの読み込み

`staticFile()` で `public` フォルダ内の `.srt` ファイルを参照し、フェッチしてパースします：

```tsx
import {useState, useEffect, useCallback} from 'react';
import {AbsoluteFill, staticFile, useDelayRender} from 'remotion';
import {parseSrt} from '@remotion/captions';
import type {Caption} from '@remotion/captions';

export const MyComponent: React.FC = () => {
  const [captions, setCaptions] = useState<Caption[] | null>(null);
  const {delayRender, continueRender, cancelRender} = useDelayRender();
  const [handle] = useState(() => delayRender());

  const fetchCaptions = useCallback(async () => {
    try {
      const response = await fetch(staticFile('subtitles.srt'));
      const text = await response.text();
      const {captions: parsed} = parseSrt({input: text});
      setCaptions(parsed);
      continueRender(handle);
    } catch (e) {
      cancelRender(e);
    }
  }, [continueRender, cancelRender, handle]);

  useEffect(() => {
    fetchCaptions();
  }, [fetchCaptions]);

  if (!captions) {
    return null;
  }

  return <AbsoluteFill>{/* ここでキャプションを使用 */}</AbsoluteFill>;
};
```

リモートURLもサポートされています。`staticFile()` の代わりにURLを指定してリモートファイルを `fetch()` できます。

## インポートしたキャプションの使用

パース後、キャプションは `Caption` 形式となり、すべての `@remotion/captions` ユーティリティで使用できます。
