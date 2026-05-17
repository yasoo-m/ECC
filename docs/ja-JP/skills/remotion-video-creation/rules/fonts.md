---
name: fonts
description: RemotionでのGoogle Fontsとローカルフォントの読み込み
metadata:
  tags: fonts, google-fonts, typography, text
---

# Remotionでのフォントの使用

## @remotion/google-fontsによるGoogle Fonts

Google Fontsを使用するための推奨方法です。タイプセーフで、フォントの準備ができるまで自動的にレンダリングをブロックします。

### 前提条件

まず、@remotion/google-fontsパッケージをインストールする必要があります。
インストールされていない場合は、以下のコマンドを使用してください:

```bash
npx remotion add @remotion/google-fonts # プロジェクトがnpmを使用している場合
bunx remotion add @remotion/google-fonts # プロジェクトがbunを使用している場合
yarn remotion add @remotion/google-fonts # プロジェクトがyarnを使用している場合
pnpm exec remotion add @remotion/google-fonts # プロジェクトがpnpmを使用している場合
```

```tsx
import { loadFont } from "@remotion/google-fonts/Lobster";

const { fontFamily } = loadFont();

export const MyComposition = () => {
  return <div style={{ fontFamily }}>Hello World</div>;
};
```

ファイルサイズを削減するために、必要なウェイトとサブセットのみを指定することをお勧めします:

```tsx
import { loadFont } from "@remotion/google-fonts/Roboto";

const { fontFamily } = loadFont("normal", {
  weights: ["400", "700"],
  subsets: ["latin"],
});
```

### フォントの読み込み完了を待つ

フォントの準備ができたタイミングを知る必要がある場合は `waitUntilDone()` を使用します:

```tsx
import { loadFont } from "@remotion/google-fonts/Lobster";

const { fontFamily, waitUntilDone } = loadFont();

await waitUntilDone();
```

## @remotion/fontsによるローカルフォント

ローカルフォントファイルには `@remotion/fonts` パッケージを使用します。

### 前提条件

まず、@remotion/fontsをインストールします:

```bash
npx remotion add @remotion/fonts # プロジェクトがnpmを使用している場合
bunx remotion add @remotion/fonts # プロジェクトがbunを使用している場合
yarn remotion add @remotion/fonts # プロジェクトがyarnを使用している場合
pnpm exec remotion add @remotion/fonts # プロジェクトがpnpmを使用している場合
```

### ローカルフォントの読み込み

フォントファイルを `public/` フォルダに配置し、`loadFont()` を使用します:

```tsx
import { loadFont } from "@remotion/fonts";
import { staticFile } from "remotion";

await loadFont({
  family: "MyFont",
  url: staticFile("MyFont-Regular.woff2"),
});

export const MyComposition = () => {
  return <div style={{ fontFamily: "MyFont" }}>Hello World</div>;
};
```

### 複数のウェイトの読み込み

同じファミリー名で各ウェイトを個別に読み込みます:

```tsx
import { loadFont } from "@remotion/fonts";
import { staticFile } from "remotion";

await Promise.all([
  loadFont({
    family: "Inter",
    url: staticFile("Inter-Regular.woff2"),
    weight: "400",
  }),
  loadFont({
    family: "Inter",
    url: staticFile("Inter-Bold.woff2"),
    weight: "700",
  }),
]);
```

### 利用可能なオプション

```tsx
loadFont({
  family: "MyFont", // 必須: CSSで使用する名前
  url: staticFile("font.woff2"), // 必須: フォントファイルのURL
  format: "woff2", // オプション: 拡張子から自動検出
  weight: "400", // オプション: フォントウェイト
  style: "normal", // オプション: normalまたはitalic
  display: "block", // オプション: font-displayの動作
});
```

## コンポーネントでの使用

コンポーネントのトップレベル、または早い段階でインポートされる別のファイル内で `loadFont()` を呼び出します:

```tsx
import { loadFont } from "@remotion/google-fonts/Montserrat";

const { fontFamily } = loadFont("normal", {
  weights: ["400", "700"],
  subsets: ["latin"],
});

export const Title: React.FC<{ text: string }> = ({ text }) => {
  return (
    <h1
      style={{
        fontFamily,
        fontSize: 80,
        fontWeight: "bold",
      }}
    >
      {text}
    </h1>
  );
};
```
