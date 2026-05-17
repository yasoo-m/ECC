# エージェント向け Angular CLI ガイド

Angular CLI（`ng`）は Angular ワークスペースを管理するための主要ツールです。プロジェクト構造を変更したり Angular 固有の依存関係を追加したりする際は、手動でのファイル作成や汎用の `npm` コマンドよりも、常に CLI コマンドを優先して使用してください。

## 1. 依存関係の管理

**Angular ライブラリには `npm install` ではなく必ず `ng add` を使用してください。** `ng add` はパッケージのインストールに加え、初期化スキーマティクス（例：`angular.json` の設定、ルートプロバイダーの更新）も実行します。

```bash
ng add @angular/material
ng add tailwindcss
ng add @angular/fire
```

アプリケーションとその依存関係を更新するには（コードマイグレーションが自動的に実行されます）：

```bash
ng update @angular/core@<latest or specific version> @angular/cli<latest or specific version>
```

## 2. コード生成（`ng generate` または `ng g`）

Angular の規約に準拠し、必要な設定ファイルが自動的に更新されるよう、コード生成には必ず CLI を使用してください。

| 対象             | コマンド               | 備考                                                                                          |
| :------------- | :-------------------- | :--------------------------------------------------------------------------------------------- |
| コンポーネント    | `ng g c path/to/name` | コンポーネントを生成します。要求に応じて `--inline-style`（`-s`）または `--inline-template`（`-t`）を使用してください。 |
| サービス         | `ng g s path/to/name` | `@Injectable({providedIn: 'root'})` サービスを生成します。                                      |
| ディレクティブ    | `ng g d path/to/name` | ディレクティブを生成します。                                                                         |
| パイプ           | `ng g p path/to/name` | パイプを生成します。                                                                              |
| ガード           | `ng g g path/to/name` | 関数型ルートガードを生成します。                                                                    |
| 環境設定         | `ng g environments`   | `src/environments/` を生成し、ファイル置換を含む `angular.json` を更新します。               |

_注意：単一のルート定義を生成するコマンドはありません。コンポーネントを生成した後、`app.routes.ts` 内の `Routes` 配列に手動で追加してください。_

## 3. 開発サーバーとプロキシ

ホットモジュール置換（HMR）を使用してローカル開発サーバーを起動します：

```bash
ng serve
```

### バックエンド API プロキシ

開発中に API リクエストをプロキシするには（例：`/api` をローカルの Node サーバーにリルーティング）：

1. `src/proxy.conf.json` を作成します：
   ```json
   {
     "/api/**": {"target": "http://localhost:3000", "secure": false}
   }
   ```
2. `angular.json` の `serve` ターゲット以下を更新します：
   ```json
   "serve": {
     "builder": "@angular/build:dev-server",
     "options": { "proxyConfig": "src/proxy.conf.json" }
   }
   ```

## 4. アプリケーションのビルド

アプリケーションを出力ディレクトリ（デフォルト：`dist/<project-name>/browser`）にコンパイルします。最新の Angular は esbuild ベースの `@angular/build:application` ビルダーを使用します。

```bash
ng build
```

- `ng build` はデフォルトでプロダクション設定を使用し、Ahead-of-Time（AOT）コンパイル、ミニファイ、ツリーシェイキングを有効にします。
- `--configuration` オプションを使用して `angular.json` で定義された特定の設定を対象にできます：`ng build --configuration=staging`。

## 5. テスト

- **ユニットテスト**：設定されたテストランナー（例：Karma または Vitest）でユニットテストを実行するには `ng test` を実行します。
- **エンドツーエンド（E2E）テスト**：`ng e2e` を実行します。E2E フレームワークが設定されていない場合、CLI がインストールを促します（Cypress、Playwright、Puppeteer など）。

## 6. デプロイ

アプリケーションをデプロイするには、まずデプロイメントビルダーを追加してからデプロイコマンドを実行します：

```bash
# Firebase の例
ng add @angular/fire
ng deploy
```
