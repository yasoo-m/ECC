# tinystruct @Action ルーティングリファレンス

## 使用場面

アプリケーションで `@Action` アノテーションを使用して、CLIコマンドとHTTPエンドポイントの両方にルートを定義します。特定のパスにロジックをマッピングする必要がある場合、パラメータ化されたリクエストを処理する場合（例：IDによるリソースの取得）、または特定のHTTPメソッド（GET、POSTなど）に実行を制限しながら、環境をまたいで一貫したコマンド構造を維持したい場合に適しています。

## 動作の仕組み

`ActionRegistry` は `@Action` アノテーションをパースしてルーティングテーブルを構築します。パラメータ化されたメソッドに対して、フレームワークはJavaパラメータ型（int、Stringなど）を対応するregexセグメントに自動的にマッピングして内部マッチングパターンを生成します。例えば、`getUser(int id)` は数字をターゲットとするregexを生成し、`search(String query)` は汎用のパスセグメントをターゲットとします。

リクエストがディスパッチされると、`ActionRegistry` は `Request` と `Response` のような依存関係が指定されている場合、現在のリクエストの `Context` から直接取り出してアクションメソッドに自動的にインジェクトします。実行はさらに `Mode` の値によってフィルタリングされ、単一のパスがターミナルコマンドか特定タイプのHTTPリクエストかによって異なるロジックを呼び出せます。

### Mode値

| Mode | トリガーされるタイミング |
|---|---|
| `DEFAULT` | CLIとHTTPの両方（GET、POSTなど） |
| `CLI` | CLIディスパッチャーのみ |
| `HTTP_GET` | HTTP GETのみ |
| `HTTP_POST` | HTTP POSTのみ |
| `HTTP_PUT` | HTTP PUTのみ |
| `HTTP_DELETE` | HTTP DELETEのみ |
| `HTTP_PATCH` | HTTP PATCHのみ |

## 例

### 基本的なアクション宣言
```java
@Action(
    value = "path/subpath",          // 必須：URIセグメントまたはCLIコマンド
    description = "何をするか",      // --helpの出力に表示される
    mode = Mode.HTTP_POST,           // デフォルト：Mode.DEFAULT（CLIとHTTPの両方）
    options = {},                    // CLIオプションフラグ
    example = "curl -X POST http://localhost:8080/path/subpath/42"
)
public String myAction(int id) { ... }
```

### パラメータ化されたパス（Regex生成）
```java
@Action("user/{id}")
public String getUser(int id) { ... }
// → パターン：^/?user/(-?\d+)$

@Action("search")
public String search(String query) { ... }
// → パターン：^/?search/([^/]+)$
```

### RequestとResponseのインジェクション
```java
@Action(value = "upload", mode = Mode.HTTP_POST)
public String upload(Request<?, ?> req, Response<?, ?> res) throws ApplicationException {
    // req.getParameter("file"), res.setHeader(...), など
    return "ok";
}
```
