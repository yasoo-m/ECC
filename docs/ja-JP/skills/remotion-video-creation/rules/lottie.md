---
name: lottie
description: RemotionにLottieアニメーションを埋め込む。
metadata:
  category: Animation
---

# RemotionでLottieアニメーションを使用する

## 前提条件

まず、@remotion/lottieパッケージをインストールする必要があります。
インストールされていない場合は、以下のコマンドを使用します：

```bash
npx remotion add @remotion/lottie # npmを使うプロジェクト
bunx remotion add @remotion/lottie # bunを使うプロジェクト
yarn remotion add @remotion/lottie # yarnを使うプロジェクト
pnpm exec remotion add @remotion/lottie # pnpmを使うプロジェクト
```

## Lottieファイルの表示

Lottieアニメーションをインポートするには：

- Lottieアセットをフェッチする
- 読み込みプロセスを `delayRender()` と `continueRender()` でラップする
- アニメーションデータをstateに保存する
- `@remotion/lottie` パッケージの `Lottie` コンポーネントを使ってLottieアニメーションをレンダリングする

```tsx
import {Lottie, LottieAnimationData} from '@remotion/lottie';
import {useEffect, useState} from 'react';
import {cancelRender, continueRender, delayRender} from 'remotion';

export const MyAnimation = () => {
  const [handle] = useState(() => delayRender('Loading Lottie animation'));

  const [animationData, setAnimationData] = useState<LottieAnimationData | null>(null);

  useEffect(() => {
    fetch('https://assets4.lottiefiles.com/packages/lf20_zyquagfl.json')
      .then((data) => data.json())
      .then((json) => {
        setAnimationData(json);
        continueRender(handle);
      })
      .catch((err) => {
        cancelRender(err);
      });
  }, [handle]);

  if (!animationData) {
    return null;
  }

  return <Lottie animationData={animationData} />;
};
```

## スタイリングとアニメーション

Lottieはスタイルとアニメーションを適用するための `style` プロパティをサポートしています：

```tsx
return <Lottie animationData={animationData} style={{width: 400, height: 400}} />;
```
