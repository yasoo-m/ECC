# tinystruct アーキテクチャと設定

## 使用場面

CLIとHTTPを同等に扱う軽量で高性能なJavaフレームワークが必要な場合に**tinystruct**を選択します。マイクロサービス、コマンドラインユーティリティ、フットプリントが小さく依存関係のないJSONハンドリングが必要なデータ駆動アプリケーションの構築に理想的です。変更なしにターミナルとWebサーバーの両方でロジックを一度だけ書いて公開したい場合に使用します。

## 動作の仕組み

### コアアーキテクチャ

フレームワークはURLパターン（またはコマンド文字列）を `Action` オブジェクトにマッピングするシングルトン `ActionRegistry` で動作します。リクエストが到着すると、システムはパスを解決して対応するメソッドハンドルを呼び出します。

#### 主要な抽象化

| クラス/インターフェース | 役割 |
|---|---|
| `AbstractApplication` | すべてのtinystruct アプリケーションの基底クラス。これを拡張します。 |
| `@Action` アノテーション | メソッドをURIパス（Web）またはコマンド名（CLI）にマッピングします。単一のルーティングプリミティブ。 |
| `ActionRegistry` | URLパターンを `Action` オブジェクトにregexでマッピングするシングルトン。直接インスタンス化しない。 |
| `Action` | ディスパッチ用の `MethodHandle` + regexパターン + 優先度 + `Mode` をラップします。 |
| `Context` | リクエストごとの状態ストア。`getContext()` でアクセス。CLIの引数とHTTPのリクエスト/レスポンスを保持。 |
| `Dispatcher` | CLIエントリポイント（`bin/dispatcher`）。アプリケーションを読み込むには `--import` を読み取ります。 |
| `HttpServer` | 組み込みのNettyベースHTTPサーバー。`bin/dispatcher start --import org.tinystruct.system.HttpServer` で起動。 |

### パッケージマップ

```
org.tinystruct/
├── AbstractApplication.java      ← これを拡張する
├── Application.java              ← インターフェース
├── ApplicationException.java     ← チェック済み例外
├── ApplicationRuntimeException.java ← 非チェック例外
├── application/
│   ├── Action.java               ← ランタイムアクションラッパー
│   ├── ActionRegistry.java       ← シングルトンルートレジストリ
│   └── Context.java              ← リクエストコンテキスト
├── system/
│   ├── annotation/Action.java    ← @Actionアノテーション + Mode列挙型
│   ├── Dispatcher.java           ← CLIディスパッチャー
│   ├── HttpServer.java           ← 組み込みHTTPサーバー
│   ├── EventDispatcher.java      ← イベントバス
│   └── Settings.java             ← application.propertiesを読み取る
├── data/component/Builder.java   ← JSONシリアライゼーション（Gson/Jacksonの代わりに使用）
└── http/                         ← Request、Response、Constants
```

### テンプレートの動作とディスパッチフロー

デフォルトでは、フレームワークはビューテンプレートが必要だと仮定します。`templateRequired` が `true` の場合、`toString()` は `src/main/resources/themes/<ClassName>.view` 内の `.view` ファイルを探します。`getContext()` を使って状態を管理し、`setVariable("name", value)` でテンプレートにデータを渡します。テンプレートは補間に `[%name%]` を使用します。

## 例

### 最小アプリケーションの初期化
```java
@Override
public void init() {
    this.setTemplateRequired(false); // データのみのアプリでは.viewテンプレートのルックアップをスキップ
}
```

### アクション定義とCLI呼び出し
```java
@Action("hello")
public String hello() {
    return "Hello, tinystruct!";
}
```
**ディスパッチャー経由での実行：**
```bash
bin/dispatcher hello
```

### 設定へのアクセス
`src/main/resources/application.properties` に配置：
```java
String port = this.getConfiguration("server.port");
```
