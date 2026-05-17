---
name: text-animations
description: Remotionのタイポグラフィとテキストアニメーションパターン。
metadata:
  tags: typography, text, typewriter, highlighter ken
---

## テキストアニメーション

`useCurrentFrame()` に基づいて、文字列を1文字ずつ削減してタイプライター効果を作成します。

## タイプライター効果

点滅カーソルと最初の文の後のポーズを持つ高度な例については、[Typewriter](assets/text-animations-typewriter.tsx) を参照してください。

タイプライター効果には常に文字列スライスを使用してください。1文字ずつの透明度は使用しないでください。

## 単語のハイライト

ハイライトペンで引くようなアニメーションについては、[Word Highlight](assets/text-animations-word-highlight.tsx) を参照してください。
