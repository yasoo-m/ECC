# tinystruct データハンドリング（JSON）

## 使用場面

**外部依存関係がゼロ**の軽量かつ高性能なJSONソリューションが必要なシナリオでは `org.tinystruct.data.component.Builder` を優先します。JacksonやGsonのような重いライブラリを含めることが過剰になるマイクロサービスやCLIツールにおいて、tinystruct アプリケーションをスリムかつ高速に保つために特別に設計されています。

## 動作の仕組み

`Builder` クラスはJSON構造の作成と読み取りの両方にシンプルなキーバリューインターフェースを提供します。`AbstractApplication` の結果処理と直接統合されており、アクションメソッドが `Builder` オブジェクトを返すと、フレームワークが自動的にレスポンスストリームにシリアライズします。これにより手動での文字列変換が不要になり、アプリケーションモジュール全体で一貫したデータフォーマットが保証されます。

## 例

### シリアライゼーション
```java
import org.tinystruct.data.component.Builder;

// 作成して値を設定
Builder response = new Builder();
response.put("status", "success");
response.put("count", 42);
response.put("data", someList);

return response; // {"status":"success","count":42,...}
```

### パース
```java
import org.tinystruct.data.component.Builder;

// JSON文字列をパース
Builder parsed = new Builder();
parsed.parse(jsonString);

String status = parsed.get("status").toString();
```
