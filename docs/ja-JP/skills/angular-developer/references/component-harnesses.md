# コンポーネントハーネスを使用したテスト

コンポーネントハーネスは、テストでコンポーネントを操作するための標準的かつ推奨される方法です。コンポーネントの内部 DOM 構造の変更からテストを保護することで、壊れにくく読みやすいテストを実現するユーザー中心の API を提供します。

## なぜハーネスを使用するのか？

- **堅牢性：** コンポーネントの内部 HTML や CSS クラスをリファクタリングしてもテストが壊れません。
- **可読性：** テストは DOM クエリ（`fixture.nativeElement.querySelector(...)`）を通じてではなく、ユーザーの観点からのインタラクション（例：`button.click()`、`slider.getValue()`）を記述します。
- **再利用性：** 同じハーネスをユニットテストと E2E テストの両方で使用できます。

Angular Material はライブラリ内のすべてのコンポーネントにテストハーネスを提供しています。

## ユニットテストでのハーネスの使用

`TestbedHarnessEnvironment` はユニットテストでハーネスを使用するためのエントリーポイントです。

### 例：`MatButtonHarness` を使用したテスト

```ts
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {MatButtonHarness} from '@angular/material/button/testing';
import {MyButtonContainerComponent} from './my-button-container.component';

describe('MyButtonContainerComponent', () => {
  let fixture: ComponentFixture<MyButtonContainerComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [MyButtonContainerComponent, MatButtonModule],
    }).compileComponents();

    fixture = TestBed.createComponent(MyButtonContainerComponent);
    // コンポーネントのフィクスチャ用のハーネスローダーを作成する
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should find a button with specific text', async () => {
    // "Submit" というテキストを持つ MatButton のハーネスをロードする
    const submitButton = await loader.getHarness(MatButtonHarness.with({text: 'Submit'}));

    // ハーネス API を使用してコンポーネントを操作する
    expect(await submitButton.isDisabled()).toBe(false);
    await submitButton.click();

    // ... アサーション
  });
});
```

### 主要なコンセプト

1. **`HarnessLoader`**：ハーネスインスタンスを検索して作成するためのオブジェクトです。`TestbedHarnessEnvironment.loader(fixture)` を使用してコンポーネントのフィクスチャ用のローダーを取得します。

2. **`loader.getHarness(HarnessClass)`**：最初にマッチするコンポーネントのハーネスインスタンスを非同期で検索して返します。

3. **`HarnessClass.with({ ... })`**：多くのハーネスは静的な `with` メソッドを提供し、`HarnessPredicate` を返します。これにより、テキスト、セレクター、無効状態などのプロパティに基づいてコンポーネントをフィルタリングして検索できます。テストしたいコンポーネントを正確に対象にするために常にこれを使用してください。

4. **ハーネス API：** ハーネスインスタンスを取得したら、そのメソッド（例：`.click()`、`.getText()`、`.getValue()`）を使用してコンポーネントを操作します。これらのメソッドは非同期操作と変更検出の待機を自動的に処理します。
