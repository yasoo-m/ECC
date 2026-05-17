# RouterTestingHarness を使ったテスト

ルーティングを伴うコンポーネントをテストする場合、**ルーターや関連サービスをモック化しない**ことが重要です。代わりに `RouterTestingHarness` を使用してください。これにより、実際のアプリケーションに近い環境でルーティングロジックをテストするための堅牢で信頼性の高い方法が提供されます。

ハーネスを使用することで、実際のルーター設定、ガード、リゾルバーをテストでき、より意味のあるテストにつながります。

## ルーターテストのセットアップ

`RouterTestingHarness` はルーティングシナリオをテストするための主要なツールです。また、`TestBed` 設定内で `provideRouter` 関数を使ってテスト用ルートを提供する必要があります。

### セットアップ例

```ts
import {TestBed} from '@angular/core/testing';
import {provideRouter} from '@angular/router';
import {RouterTestingHarness} from '@angular/router/testing';
import {Dashboard} from './dashboard.component';
import {HeroDetail} from './hero-detail.component';

describe('Dashboard Component Routing', () => {
  let harness: RouterTestingHarness;

  beforeEach(async () => {
    // 1. テスト用ルートで TestBed を設定
    await TestBed.configureTestingModule({
      providers: [
        // テスト固有のルートで provideRouter を使用
        provideRouter([
          {path: '', component: Dashboard},
          {path: 'heroes/:id', component: HeroDetail},
        ]),
      ],
    }).compileComponents();

    // 2. RouterTestingHarness を作成
    harness = await RouterTestingHarness.create();
  });
});
```

### 主要なコンセプト

1. **`provideRouter([...])`**：テスト用のルーティング設定を提供します。テスト対象のコンポーネントが正しく機能するために必要なルートを含める必要があります。
2. **`RouterTestingHarness.create()`**：ハーネスを非同期で作成・初期化し、ルート URL（`/`）への初回ナビゲーションを実行します。

## ルーターテストの作成

ハーネスを作成したら、ナビゲーションを駆動し、ルーターの状態やアクティベートされたコンポーネントの状態に対してアサーションを行えます。

### 例：ナビゲーションのテスト

```ts
it('should navigate to a hero detail when a hero is selected', async () => {
  // 1. 初期コンポーネントにナビゲートし、そのインスタンスを取得
  const dashboard = await harness.navigateByUrl('/', Dashboard);

  // ダッシュボードにヒーローを選択するメソッドがあると仮定
  const heroToSelect = {id: 42, name: 'Test Hero'};
  dashboard.selectHero(heroToSelect);

  // ナビゲーションをトリガーするアクション後の安定を待つ
  await harness.fixture.whenStable();

  // 2. URL に対してアサート
  expect(harness.router.url).toEqual('/heroes/42');

  // 3. ナビゲーション後にアクティベートされたコンポーネントを取得
  const heroDetail = await harness.getHarness(HeroDetail);

  // 4. 新しいコンポーネントの状態に対してアサート
  expect(await heroDetail.componentInstance.hero.name).toBe('Test Hero');
});

it('should get the activated component directly', async () => {
  // 一度にナビゲートしてコンポーネントインスタンスを取得
  const dashboardInstance = await harness.navigateByUrl('/', Dashboard);

  expect(dashboardInstance).toBeInstanceOf(Dashboard);
});
```

### ベストプラクティス

- **ハーネスでナビゲートする**：ナビゲーションをシミュレートするには常に `harness.navigateByUrl()` を使用してください。このメソッドはアクティベートされたコンポーネントのインスタンスで解決される Promise を返します。
- **ルーターの状態にアクセスする**：`harness.router` を使用してライブのルーターインスタンスにアクセスし、状態に対してアサートします（例：`harness.router.url`）。
- **アクティベートされたコンポーネントの取得**：現在アクティベートされているルーティングコンポーネントのコンポーネントハーネスインスタンスを取得するには `harness.getHarness(ComponentType)` を、`DebugElement` を取得するには `harness.routeDebugElement` を使用してください。
- **安定を待つ**：ナビゲーションを引き起こすアクションを実行した後は、アサーションを行う前に必ず `await harness.fixture.whenStable()` でルーティングの完了を待ってください。
