# tinystruct システムと使用リファレンス

## 使用場面

CLIとHTTPモード間でステートフルなインタラクションを処理する必要がある場合、Webアプリケーションでユーザーセッションを管理する場合、またはイベント駆動アーキテクチャを使用してアプリケーションモジュール間の疎結合な通信を実装する場合に、ここで説明するシステムと使用パターンを使用します。

## 動作の仕組み

フレームワークの `Context` はリクエスト固有の状態のプライマリデータストアとして機能します。CLIモードでは、`--key value` として渡されるフラグが `--` プレフィックス付きで自動的にパースされて `Context` に保存されるため、アクションメソッドがコマンドパラメータを簡単に取得できます。Webアプリケーションでは、システムは `Request` オブジェクト経由で標準セッション管理を提供し、複数のHTTPリクエストをまたいでユーザーデータを保存できます。

内部の `EventDispatcher` は非同期イベントバスを実現します。カスタム `Event` クラスを定義してハンドラーを登録（通常はアプリケーションの `init()` メソッド内）することで、メインの実行パスをブロックせずにバックグラウンドタスク（メールの送信や監査ログなど）をトリガーできます。

## 例

### コンテキストとCLI引数
```java
@Action("echo")
public String echo() {
    // CLI: bin/dispatcher echo --words "Hello World"
    Object words = getContext().getAttribute("--words");
    if (words != null) return words.toString();
    return "No words provided";
}
```

### セッション管理（Webモード）
```java
@Action(value = "login", mode = Mode.HTTP_POST)
public String login(Request request) {
    request.getSession().setAttribute("userId", "42");
    return "Logged in";
}

@Action("profile")
public String profile(Request request) {
    Object userId = request.getSession().getAttribute("userId");
    if (userId == null) return "Not logged in";
    return "User: " + userId;
}
```

### イベントシステム
```java
// 1. イベントを定義する
public class OrderCreatedEvent implements org.tinystruct.system.Event<Order> {
    private final Order order;
    public OrderCreatedEvent(Order order) { this.order = order; }

    @Override public String getName() { return "order_created"; }
    @Override public Order getPayload() { return order; }
}

// 2. ハンドラーを登録する
EventDispatcher.getInstance().registerHandler(OrderCreatedEvent.class, event -> {
    CompletableFuture.runAsync(() -> sendConfirmationEmail(event.getPayload()));
});

// 3. ディスパッチする
EventDispatcher.getInstance().dispatch(new OrderCreatedEvent(newOrder));
```

### アプリケーションの実行
```bash
# CLIモード
bin/dispatcher hello
bin/dispatcher echo --words "Hello" --import com.example.HelloApp

# HTTPサーバー（デフォルトで:8080でリッスン）
bin/dispatcher start --import org.tinystruct.system.HttpServer

# データベースユーティリティ
bin/dispatcher generate --table users
bin/dispatcher sql-query "SELECT * FROM users"
```
