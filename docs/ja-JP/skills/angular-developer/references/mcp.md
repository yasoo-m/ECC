# Angular CLI MCP サーバー

Angular CLI には Model Context Protocol（MCP）サーバーが含まれており、AI アシスタント（Cursor、Gemini CLI、JetBrains AI など）が Angular CLI と直接やり取りできるようになります。コード生成、コードの近代化、サンプルの取得、ビルド/テストの実行を行うツールを提供します。

## 利用可能なツール（デフォルト）

MCP サーバーが有効化されると、AI エージェントは以下のツールにアクセスできます：

| 名前                        | 説明                                                                                                      |
| :-------------------------- | :-------------------------------------------------------------------------------------------------------- |
| `ai_tutor`                  | インタラクティブな AI 搭載の Angular チューターを起動します。                                             |
| `find_examples`             | モダンな Angular 機能に関する権威あるベストプラクティスのコードサンプルを検索します。                     |
| `get_best_practices`        | Angular ベストプラクティスガイドを取得します（スタンドアロンコンポーネント、型付きフォームなどに必須）。  |
| `list_projects`             | `angular.json` を読み取り、ワークスペース内のすべてのアプリケーションとライブラリを一覧表示します。       |
| `onpush_zoneless_migration` | コードを分析し、`OnPush` 変更検知（ゾーンレスの前提条件）への移行計画を提供します。                      |
| `search_documentation`      | `https://angular.dev` の公式ドキュメントを検索します。                                                    |

## 実験的ツール

一部のツールは `--experimental-tool`（または `-E`）フラグを使って明示的に有効化する必要があります。

| 名前                       | 説明                                                             |
| :------------------------- | :----------------------------------------------------------------------- |
| `build`                    | `ng build` を使って一度限りのビルドを実行します。                |
| `devserver.start`          | 開発サーバー（`ng serve`）を非同期で起動します。即座に返ります。 |
| `devserver.stop`           | 開発サーバーを停止します。                                       |
| `devserver.wait_for_build` | 実行中の開発サーバーの最新ビルドのログを返します。               |
| `e2e`                      | エンドツーエンドテストを実行します。                             |
| `modernize`                | 最新のベストプラクティスと構文に合わせてコードの移行を行います。 |
| `test`                     | プロジェクトのユニットテストを実行します。                       |

## 設定

MCP サーバーを使用するには、ホスト環境（IDE または CLI）が `npx @angular/cli mcp` を実行するよう設定します。

### Antigravity IDE

プロジェクトのルートに `.antigravity/mcp.json` というファイルを作成します：

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

### Gemini CLI

プロジェクトルートに `.gemini/settings.json` を作成します：

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

### Cursor

プロジェクトルートに `.cursor/mcp.json` を作成します（またはグローバルに `~/.cursor/mcp.json`）：

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

### VS Code

`.vscode/mcp.json` を作成します：

```json
{
  "servers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

## コマンドオプション

設定の `args` 配列に MCP サーバーへの引数を渡せます：

- `--read-only`：プロジェクトを変更しないツールのみを登録します。
- `--local-only`：インターネット接続を必要としないツールのみを登録します。
- `--experimental-tool`（`-E`）：特定の実験的ツールを有効化します（例：`-E build`、`-E devserver`）。

実験的ツールを有効にした読み取り専用モードの例：

```json
"args": ["-y", "@angular/cli", "mcp", "--read-only", "-E", "build", "-E", "modernize"]
```
