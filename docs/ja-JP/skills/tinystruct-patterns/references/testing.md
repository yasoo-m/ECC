# tinystruct テストパターン

## 使用場面

**JUnit 5** でtinystruct アプリケーションのユニットテストを作成する場合に、ここで説明するテストパターンを使用します。これらのパターンは `@Action` メソッドが正しい結果を返し、ルーティングロジックがシングルトン `ActionRegistry` 内に適切に登録されていることを検証するために不可欠です。

## 動作の仕組み

tinystruct アプリケーションのテストには、アノテーション処理や設定管理などのフレームワークレベルの機能がアクティブであることを保証するための特定のセットアップが必要です。`setUp()` メソッドでアプリケーションの新しいインスタンスを作成して `Settings` オブジェクトを渡すことで、`init()` ライフサイクルをトリガーします。これにより、すべての `@Action` メソッドが検出されて登録されることが保証されます。

`ActionRegistry` はシングルトンであるため、テスト間で副作用が漏れるのを防ぐため、各テスト実行前にアプリケーションの状態を適切に初期化してテストの分離を維持することが重要です。

## 例

### アプリケーションのユニットテスト
```java
import org.junit.jupiter.api.*;
import org.tinystruct.application.ActionRegistry;
import org.tinystruct.system.Settings;

class MyAppTest {

    private MyApp app;

    @BeforeEach
    void setUp() {
        app = new MyApp();
        Settings config = new Settings();
        app.setConfiguration(config);
        app.init(); // @Actionアノテーション処理をトリガー
    }
    void testHello() throws Exception {
        // アプリケーションオブジェクト経由の直接呼び出し
        Object result = app.invoke("hello");
        Assertions.assertEquals("Hello, tinystruct!", result);
    }

    @Test
    void testGreet() throws Exception {
        // 引数付きの呼び出し
        Object result = app.invoke("greet", new Object[]{"James"});
        Assertions.assertEquals("Hello, James!", result);
    }
}
```

### ActionRegistry経由のテスト
ルーティングロジック自体をテストする必要がある場合は、`ActionRegistry` シングルトンを使用してパスマッチングを検証します：

```java
@Test
void testRouting() {
    ActionRegistry registry = ActionRegistry.getInstance();
    // パスがアクションにマッチするか検証
    Action action = registry.getAction("greet/James");
    Assertions.assertNotNull(action);
}
```
参照：`src/test/java/org/tinystruct/application/ActionRegistryTest.java`
