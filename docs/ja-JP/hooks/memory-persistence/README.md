# メモリ永続化フック

これらのライフサイクルフック定義は、Claude CodeプラグインおよびマニュアルインストールにおけるECCのメモリ永続化コントラクトを文書化します。

実行可能な実装は `scripts/hooks/` にあります：

- `session-start.js` は境界付けられた事前コンテキストを読み込み、プロジェクト状態を検出し、セッションメタデータを準備します。
- `pre-compact.js` はコンテキスト圧縮前に状態をキャプチャします。
- `session-end.js` はトランスクリプトメタデータが利用可能な場合にセッション終了サマリーを永続化します。
- `observe-runner.js` は継続学習のためのツール使用観察を記録します。
- `session-activity-tracker.js` はECC2のステータスと可観測性のためのツール使用とファイルアクティビティを記録します。

インストール済みフックグラフは引き続き `hooks/hooks.json` です。このディレクトリは、ハーネス監査と長文ドキュメントが参照する安定した人間が読めるライフサイクル定義サーフェスです。

## ライフサイクルコントラクト

| イベント | フック | 目的 | ブロッキング |
|---|---|---|---|
| `SessionStart` | `session:start` | 境界付けられた事前コンテキストとプロジェクトメタデータを読み込む | いいえ |
| `PreCompact` | `pre:compact` | 圧縮前に状態を保存する | いいえ |
| `PreToolUse` | `pre:observe:continuous-learning` | 学習シグナルのためのツール意図をキャプチャ | いいえ |
| `PostToolUse` | `post:observe:continuous-learning` | 学習シグナルのためのツール結果をキャプチャ | いいえ |
| `PostToolUse` | `post:session-activity-tracker` | ECC2メトリクス用のツールとファイルアクティビティを記録 | いいえ |
| `Stop` | `stop:format-typecheck` | 編集後のバッチ品質ゲート | フック失敗時にブロック |
| `Stop` | `stop:check-console-log` | 変更されたファイルのデバッグログを監査 | フック出力による警告/エラー |

## オペレーター期待値

- デフォルトでは永続化をローカルに保つ。
- ユーザーが明示的にインテグレーションを有効にしない限り、トランスクリプトやツールトレースをホストサービスに送信しない。
- `ECC_SESSION_START_MAX_CHARS` でセッション開始時に読み込まれるコンテキストを制限する。
- `ECC_SESSION_START_CONTEXT=off` でオプトアウトを許可する。
- `ECC_HOOK_PROFILE` と `ECC_DISABLED_HOOKS` でライフサイクルフックをプロファイルゲート管理する。

## 関連ファイル

- `hooks/hooks.json`
- `hooks/README.md`
- `scripts/hooks/session-start.js`
- `scripts/hooks/pre-compact.js`
- `scripts/hooks/session-end.js`
- `scripts/hooks/observe-runner.js`
- `scripts/hooks/session-activity-tracker.js`
- `docs/architecture/observability-readiness.md`
