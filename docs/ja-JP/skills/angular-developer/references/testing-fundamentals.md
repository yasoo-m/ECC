# テストの基礎

このガイドでは、Angularのユニットテストおよびコンポーネントテストを記述するための基本的な原則と実践を説明します。プロジェクトにすでに設定されているテストランナーを使用してください。

## 核心哲学: 非同期ファースト

最新のAngularアプリケーションは、特にシグナルやゾーンレス変更検知を使用する場合、状態変更を非同期にスケジュールすることが多いです。テストはこれを考慮する必要があります。

「Act（実行）、Wait（待機）、Assert（検証）」パターンを推奨します:

1. **Act（実行）:** 状態を更新するかアクションを実行します（例: コンポーネントの入力を設定する、ボタンをクリックする）。
2. **Wait（待機）:** `await fixture.whenStable()` を使用して、フレームワークがスケジュールされた更新を処理しレンダリングするのを待ちます。
3. **Assert（検証）:** 結果を確認します。

### 基本的なテスト構造の例

```ts
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {MyComponent} from './my.component';

describe('MyComponent', () => {
  let component: MyComponent;
  let fixture: ComponentFixture<MyComponent>;
  let h1: HTMLElement;

  beforeEach(async () => {
    // 1. テストモジュールを設定する
    await TestBed.configureTestingModule({
      imports: [MyComponent],
    }).compileComponents();

    // 2. コンポーネントフィクスチャーを作成する
    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
    h1 = fixture.nativeElement.querySelector('h1');
  });

  it('should display the default title', async () => {
    // ACT: （暗黙的）コンポーネントはデフォルト状態で作成される。
    // 初期データバインディングを待機する。
    await fixture.whenStable();
    // 初期状態を検証する。
    expect(h1.textContent).toContain('Default Title');
  });

  it('should display a different title after a change', async () => {
    // ACT: コンポーネントのタイトルプロパティを変更する。
    component.title.set('New Test Title');

    // 非同期更新が完了するのを待機する。
    await fixture.whenStable();

    // DOMが更新されたことを検証する。
    expect(h1.textContent).toContain('New Test Title');
  });
});
```

## TestBed と ComponentFixture

- **`TestBed`**: テスト専用のAngularモジュールを作成するための主要なユーティリティです。テストに必要なコンポーネントの宣言、サービスの提供、インポートのセットアップのために `beforeEach` 内で `TestBed.configureTestingModule({...})` を使用します。
- **`ComponentFixture`**: 作成されたコンポーネントのインスタンスとその環境を扱うためのハンドルです。
  - `fixture.componentInstance`: コンポーネントのクラスインスタンスにアクセスします。
  - `fixture.nativeElement`: コンポーネントのルートDOM要素にアクセスします。
  - `fixture.debugElement`: `nativeElement` をAngular固有のラッパーでラップしたもので、DOMをより安全かつプラットフォームに依存しない方法でクエリできます（例: `debugElement.query(By.css('p'))`）。
