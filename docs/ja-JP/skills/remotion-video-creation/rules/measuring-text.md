---
name: measuring-text
description: テキストのサイズ測定、コンテナへのテキスト収め、オーバーフローの確認
metadata:
  tags: measure, text, layout, dimensions, fitText, fillTextBox
---

# Remotionでテキストを測定する

## 前提条件

@remotion/layout-utilsがインストールされていない場合はインストールします：

```bash
npx remotion add @remotion/layout-utils # npmを使うプロジェクト
bunx remotion add @remotion/layout-utils # bunを使うプロジェクト
yarn remotion add @remotion/layout-utils # yarnを使うプロジェクト
pnpm exec remotion add @remotion/layout-utils # pnpmを使うプロジェクト
```

## テキストのサイズを測定する

`measureText()` を使ってテキストの幅と高さを計算します：

```tsx
import { measureText } from "@remotion/layout-utils";

const { width, height } = measureText({
  text: "Hello World",
  fontFamily: "Arial",
  fontSize: 32,
  fontWeight: "bold",
});
```

結果はキャッシュされるため、同じ呼び出しはキャッシュ結果を返します。

## テキストを幅に収める

`fitText()` を使ってコンテナに最適なフォントサイズを見つけます：

```tsx
import { fitText } from "@remotion/layout-utils";

const { fontSize } = fitText({
  text: "Hello World",
  withinWidth: 600,
  fontFamily: "Inter",
  fontWeight: "bold",
});

return (
  <div
    style={{
      fontSize: Math.min(fontSize, 80), // 80pxで上限を設定
      fontFamily: "Inter",
      fontWeight: "bold",
    }}
  >
    Hello World
  </div>
);
```

## テキストのオーバーフローを確認する

`fillTextBox()` を使ってテキストがボックスを超えるか確認します：

```tsx
import { fillTextBox } from "@remotion/layout-utils";

const box = fillTextBox({ maxBoxWidth: 400, maxLines: 3 });

const words = ["Hello", "World", "This", "is", "a", "test"];
for (const word of words) {
  const { exceedsBox } = box.add({
    text: word + " ",
    fontFamily: "Arial",
    fontSize: 24,
  });
  if (exceedsBox) {
    // テキストがオーバーフローするため適切に処理
    break;
  }
}
```

## ベストプラクティス

**フォントを先に読み込む：** フォントが読み込まれた後にのみ測定関数を呼び出します。

```tsx
import { loadFont } from "@remotion/google-fonts/Inter";

const { fontFamily, waitUntilDone } = loadFont("normal", {
  weights: ["400"],
  subsets: ["latin"],
});

waitUntilDone().then(() => {
  // ここで安全に測定できる
  const { width } = measureText({
    text: "Hello",
    fontFamily,
    fontSize: 32,
  });
})
```

**validateFontIsLoadedを使う：** フォント読み込みの問題を早期に発見します：

```tsx
measureText({
  text: "Hello",
  fontFamily: "MyCustomFont",
  fontSize: 32,
  validateFontIsLoaded: true, // フォントが読み込まれていない場合にエラーをスロー
});
```

**フォントプロパティを一致させる：** 測定とレンダリングで同じプロパティを使用します：

```tsx
const fontStyle = {
  fontFamily: "Inter",
  fontSize: 32,
  fontWeight: "bold" as const,
  letterSpacing: "0.5px",
};

const { width } = measureText({
  text: "Hello",
  ...fontStyle,
});

return <div style={fontStyle}>Hello</div>;
```

**パディングとボーダーを避ける：** レイアウトの差異を防ぐために `border` の代わりに `outline` を使用します：

```tsx
<div style={{ outline: "2px solid red" }}>Text</div>
```
