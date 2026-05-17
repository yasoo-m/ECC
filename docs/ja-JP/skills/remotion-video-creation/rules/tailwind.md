---
name: tailwind
description: RemotionでTailwindCSSを使用する。
metadata:
---

プロジェクトにTailwindCSSがインストールされている場合、RemotionでTailwindCSSを使用できますし、使用すべきです。

`transition-*` や `animate-*` クラスは使用しないでください。アニメーションは常に `useCurrentFrame()` フックを使用して実装してください。

TailwindをRemotionプロジェクトで使用するには、まずインストールと有効化が必要です。手順については <https://www.remotion.dev/docs/tailwind> をWebFetchで取得してください。
