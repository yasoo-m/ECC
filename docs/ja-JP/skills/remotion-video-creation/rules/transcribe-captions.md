---
name: transcribe-captions
description: Remotionでキャプション生成のために音声をテキスト変換する
metadata:
  tags: captions, transcribe, whisper, audio, speech-to-text
---

# 音声のテキスト変換

Remotionはキャプション生成のための音声テキスト変換に、いくつかの組み込みオプションを提供しています：

- `@remotion/install-whisper-cpp` - Whisper.cppを使用してサーバー上でローカル変換。高速で無料ですが、サーバーインフラが必要です。
  <https://remotion.dev/docs/install-whisper-cpp>

- `@remotion/whisper-web` - WebAssemblyを使用してブラウザ内で変換。サーバー不要で無料ですが、WASMのオーバーヘッドにより遅くなります。
  <https://remotion.dev/docs/whisper-web>

- `@remotion/openai-whisper` - クラウドベースの変換にOpenAI Whisper APIを使用。高速でサーバー不要ですが、有料です。
  <https://remotion.dev/docs/openai-whisper/openai-whisper-api-to-captions>
