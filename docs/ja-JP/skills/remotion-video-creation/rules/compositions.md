---
name: compositions
description: コンポジション、スティル、フォルダー、デフォルトプロップ、動的メタデータの定義
metadata:
  tags: composition, still, folder, props, metadata
---

`<Composition>` はレンダリング可能な動画のコンポーネント、幅、高さ、fps、デュレーションを定義します。

通常、`src/Root.tsx` ファイルに配置されます。

```tsx
import { Composition } from "remotion";
import { MyComposition } from "./MyComposition";

export const RemotionRoot = () => {
  return (
    <Composition
      id="MyComposition"
      component={MyComposition}
      durationInFrames={100}
      fps={30}
      width={1080}
      height={1080}
    />
  );
};
```

## デフォルトプロップ

コンポーネントの初期値を提供するために `defaultProps` を渡します。
値はJSONシリアライズ可能である必要があります（`Date`、`Map`、`Set`、`staticFile()` はサポートされています）。

```tsx
import { Composition } from "remotion";
import { MyComposition, MyCompositionProps } from "./MyComposition";

export const RemotionRoot = () => {
  return (
    <Composition
      id="MyComposition"
      component={MyComposition}
      durationInFrames={100}
      fps={30}
      width={1080}
      height={1080}
      defaultProps={{
        title: "Hello World",
        color: "#ff0000",
      } satisfies MyCompositionProps}
    />
  );
};
```

`defaultProps` の型安全性を確保するために、`interface` ではなく `type` 宣言をプロップに使用してください。

## フォルダー

サイドバーでコンポジションを整理するために `<Folder>` を使用します。
フォルダー名には文字、数字、ハイフンのみ使用できます。

```tsx
import { Composition, Folder } from "remotion";

export const RemotionRoot = () => {
  return (
    <>
      <Folder name="Marketing">
        <Composition id="Promo" /* ... */ />
        <Composition id="Ad" /* ... */ />
      </Folder>
      <Folder name="Social">
        <Folder name="Instagram">
          <Composition id="Story" /* ... */ />
          <Composition id="Reel" /* ... */ />
        </Folder>
      </Folder>
    </>
  );
};
```

## スティル

単一フレーム画像には `<Still>` を使用します。`durationInFrames` や `fps` は不要です。

```tsx
import { Still } from "remotion";
import { Thumbnail } from "./Thumbnail";

export const RemotionRoot = () => {
  return (
    <Still
      id="Thumbnail"
      component={Thumbnail}
      width={1280}
      height={720}
    />
  );
};
```

## メタデータの計算

`calculateMetadata` を使用して、データに基づいて寸法、デュレーション、プロップを動的にします。

```tsx
import { Composition, CalculateMetadataFunction } from "remotion";
import { MyComposition, MyCompositionProps } from "./MyComposition";

const calculateMetadata: CalculateMetadataFunction<MyCompositionProps> = async ({
  props,
  abortSignal,
}) => {
  const data = await fetch(`https://api.example.com/video/${props.videoId}`, {
    signal: abortSignal,
  }).then((res) => res.json());

  return {
    durationInFrames: Math.ceil(data.duration * 30),
    props: {
      ...props,
      videoUrl: data.url,
    },
  };
};

export const RemotionRoot = () => {
  return (
    <Composition
      id="MyComposition"
      component={MyComposition}
      durationInFrames={100} // プレースホルダー、上書きされる
      fps={30}
      width={1080}
      height={1080}
      defaultProps={{ videoId: "abc123" }}
      calculateMetadata={calculateMetadata}
    />
  );
};
```

この関数は `props`、`durationInFrames`、`width`、`height`、`fps`、コーデック関連のデフォルトを返すことができます。レンダリング開始前に一度実行されます。
