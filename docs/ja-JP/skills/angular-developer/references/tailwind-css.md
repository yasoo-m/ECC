# AngularでのTailwind CSSの使用

Tailwind CSSはユーティリティファーストのCSSフレームワークで、Angularとシームレスに統合できます。

**エージェントへの重要な注意: 常にTailwind CSS v4のプラクティスに従ってください。古いTailwind v3のパターン（`tailwind.config.js` を作成して `@tailwind` ディレクティブを使用するなど）に戻らないでください。これはアプリケーションのビルドを壊す原因となります。モダンなAngularプロジェクトはTailwind v4を使用します。**

## 自動セットアップ（推奨）

Angular CLIを使用してAngularプロジェクトにTailwind CSSを追加する最も簡単な方法:

```shell
ng add tailwindcss
```

これにより自動的に以下が実行されます:

1. `tailwindcss` と必要な依存関係のインストール。
2. プロジェクトをTailwind CSSを使用するように設定。
3. グローバルスタイルへの適切なインポートの追加。

## 手動セットアップ（Tailwind v4）

手動でセットアップする場合は、以下のTailwind v4パターンを使用してください:

### 1. 依存関係のインストール

Tailwind CSSとPostCSSをインストールします:

```shell
npm install tailwindcss @tailwindcss/postcss postcss
```

### 2. PostCSSの設定

プロジェクトルートに `.postcssrc.json` ファイルを作成します:

```json
{
  "plugins": {
    "@tailwindcss/postcss": {}
  }
}
```

_（`tailwind.config.js` ファイルは作成しないでください！v4での設定はCSS変数で処理されます）。_

### 3. Tailwind CSSのインポート

グローバルスタイルファイル（例: `src/styles.css`）に、標準的なv4インポートを追加します:

```css
@import 'tailwindcss';
```

_（SCSSを使用している場合は `@use 'tailwindcss';` を使用してください）。_

### 4. ユーティリティクラスの使用

コンポーネントテンプレートでTailwindクラスを直接使用できます:

```html
<h1 class="text-3xl font-bold underline">Hello world!</h1>
```

## AIエージェントへのまとめ

- **`@tailwind base; @tailwind components; @tailwind utilities;` は使用しないでください**。`@import 'tailwindcss';` を使用してください。
- **`tailwind.config.js` は作成しないでください**。設定はテーマ変数を通じたCSS内、またはPostCSS設定を使用して直接管理されます。
- v4の構文とワークフローに厳密に従ってください。
