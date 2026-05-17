---
name: images
description: Remotionで<Img>コンポーネントを使って画像を埋め込む
metadata:
  tags: images, img, staticFile, png, jpg, svg, webp
---

# Remotionで画像を使用する

## `<Img>` コンポーネント

画像を表示するには常に `remotion` の `<Img>` コンポーネントを使用します：

```tsx
import { Img, staticFile } from "remotion";

export const MyComposition = () => {
  return <Img src={staticFile("photo.png")} />;
};
```

## 重要な制限事項

**`remotion` の `<Img>` コンポーネントを使用する必要があります。** 以下は使用しないでください：

- ネイティブHTMLの `<img>` 要素
- Next.jsの `<Image>` コンポーネント
- CSSの `background-image`

`<Img>` コンポーネントはレンダリング前に画像が完全に読み込まれることを保証し、ビデオエクスポート中のちらつきや空白フレームを防ぎます。

## staticFile()を使ったローカル画像

`public/` フォルダに画像を配置し、`staticFile()` で参照します：

```
my-video/
├─ public/
│  ├─ logo.png
│  ├─ avatar.jpg
│  └─ icon.svg
├─ src/
├─ package.json
```

```tsx
import { Img, staticFile } from "remotion";

<Img src={staticFile("logo.png")} />
```

## リモート画像

リモートURLは `staticFile()` なしで直接使用できます：

```tsx
<Img src="https://example.com/image.png" />
```

リモート画像はCORSが有効になっている必要があります。

アニメーションGIFの場合は、代わりに `@remotion/gif` の `<Gif>` コンポーネントを使用してください。

## サイズと位置

`style` プロパティでサイズと位置を制御します：

```tsx
<Img
  src={staticFile("photo.png")}
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

## 動的な画像パス

動的なファイル参照にはテンプレートリテラルを使用します：

```tsx
import { Img, staticFile, useCurrentFrame } from "remotion";

const frame = useCurrentFrame();

// 画像シーケンス
<Img src={staticFile(`frames/frame${frame}.png`)} />

// プロパティに基づいて選択
<Img src={staticFile(`avatars/${props.userId}.png`)} />

// 条件付き画像
<Img src={staticFile(`icons/${isActive ? "active" : "inactive"}.svg`)} />
```

このパターンは以下に役立ちます：

- 画像シーケンス（フレームごとのアニメーション）
- ユーザー固有のアバターやプロフィール画像
- テーマに基づくアイコン
- 状態依存のグラフィックス

## 画像サイズの取得

`getImageDimensions()` を使用して画像のサイズを取得します：

```tsx
import { getImageDimensions, staticFile } from "remotion";

const { width, height } = await getImageDimensions(staticFile("photo.png"));
```

アスペクト比の計算やコンポジションのサイズ設定に便利です：

```tsx
import { getImageDimensions, staticFile, CalculateMetadataFunction } from "remotion";

const calculateMetadata: CalculateMetadataFunction = async () => {
  const { width, height } = await getImageDimensions(staticFile("photo.png"));
  return {
    width,
    height,
  };
};
```
