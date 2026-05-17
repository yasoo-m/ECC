# エンドツーエンド（E2E）テスト

実際のブラウザでの重要なユーザージャーニーをカバーするために E2E テストを使用します。Cypress や Playwright など、Angular ワークスペースですでに設定されているフレームワークを優先してください。

## E2E テストの実行

プロジェクト固有のコマンドについては `package.json` と `angular.json` を確認してください。一般的なパターンには以下があります：

```shell
npm run e2e
pnpm e2e
ng e2e
```

アプリをビルドまたはサーブする必要がある場合は、並列テストエントリポイントを新たに作成するのではなく、既存のプロジェクトスクリプトを使用してください。

## テスト構造

- E2E スペックは `cypress/e2e/` や `e2e/` など、設定されたテストフレームワークに近い場所に保管してください。
- 再利用可能なログイン/セットアップヘルパーはフレームワークのサポートディレクトリに配置してください。
- フィクスチャは明示的に小さくして、各テストが依存するユーザー状態を説明できるようにしてください。

### Cypress の例

```typescript
describe('Login flow', () => {
  it('redirects to dashboard on valid credentials', () => {
    cy.visit('/login');
    cy.get('[data-cy=email]').type('user@example.com');
    cy.get('[data-cy=password]').type('password123');
    cy.get('[data-cy=submit]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

### Playwright の例

```typescript
import {expect, test} from '@playwright/test';

test('redirects to dashboard on valid credentials', async ({page}) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', {name: 'Sign in'}).click();
  await expect(page).toHaveURL(/dashboard/);
});
```

## ベストプラクティス

- アクセシブルなロケーター（`getByRole`、`getByLabel`）または安定した `data-*` 属性を優先してください。
- CSS クラス、DOM の深さ、または偶発的なテキストに依存するセレクターは避けてください。
- 任意のスリープではなく、特定の UI 状態、ルート、またはネットワークレスポンスを待機してください。
- スモークテストは短く保ち、完全なワークフローカバレッジは最も価値の高いパスに限定してください。
