# コンポーネント

Angular コンポーネントはアプリケーションの基本的な構成要素です。各コンポーネントは、振る舞いを持つ TypeScript クラス、HTML テンプレート、CSS セレクターで構成されます。

## コンポーネントの定義

`@Component` デコレーターを使用してコンポーネントのメタデータを定義します。

```ts
@Component({
  selector: 'app-profile',
  template: `
    <img src="profile.jpg" alt="Profile photo" />
    <button (click)="save()">Save</button>
  `,
  styles: `
    img {
      border-radius: 50%;
    }
  `,
})
export class Profile {
  save() {
    /* ... */
  }
}
```

## メタデータオプション

- `selector`：テンプレート内でこのコンポーネントを識別する CSS セレクター。
- `template`：インライン HTML テンプレート（小さなテンプレートに推奨）。
- `templateUrl`：外部 HTML ファイルへのパス。
- `styles`：インライン CSS スタイル。
- `styleUrl` / `styleUrls`：外部 CSS ファイルへのパス（複数可）。
- `imports`：このコンポーネントのテンプレートで使用されるコンポーネント、ディレクティブ、パイプのリスト。

## コンポーネントの使用

コンポーネントを使用するには、利用側コンポーネントの `imports` 配列に追加し、テンプレートでセレクターを使用します。

```ts
@Component({
  selector: 'app-root',
  imports: [Profile],
  template: `<app-profile />`,
})
export class App {}
```

## テンプレート制御フロー

Angular は条件レンダリングとループに組み込みブロックを使用します。

### 条件レンダリング（`@if`）

コンテンツを条件付きで表示するには `@if` を使用します。`@else if` と `@else` ブロックを含めることができます。

```html
@if (user.isAdmin) {
<admin-dashboard />
} @else if (user.isModerator) {
<mod-dashboard />
} @else {
<standard-dashboard />
}
```

**結果のエイリアス**：式の結果を保存して再利用できます。

```html
@if (user.settings(); as settings) {
<p>Theme: {{ settings.theme }}</p>
}
```

### ループ（`@for`）

`@for` ブロックはコレクションを反復処理します。`track` 式はパフォーマンスと DOM 再利用のために**必須**です。

```html
<ul>
  @for (item of items(); track item.id; let i = $index, total = $count) {
  <li>{{ i + 1 }}/{{ total }}: {{ item.name }}</li>
  } @empty {
  <li>No items to display.</li>
  }
</ul>
```

**暗黙的変数**：`$index`、`$count`、`$first`、`$last`、`$even`、`$odd`。

### コンテンツの切り替え（`@switch`）

`@switch` ブロックは値に基づいてコンテンツをレンダリングします。厳密等価（`===`）を使用し、**フォールスルーはありません**。

```html
@switch (status()) { @case ('loading') { <app-spinner /> } @case ('error') { <app-error-msg /> }
@case ('success') { <app-data-grid /> } @default {
<p>Unknown status</p>
} }
```

**網羅的型チェック**：`@default never;` を使用して、ユニオン型のすべてのケースが処理されることを確認します。

```html
@switch (state) { @case ('on') { ... } @case ('off') { ... } @default never; // 'standby' のような新しい
// 状態が追加された場合にエラーになる }
```

## 核心的コンセプト

- **ホスト要素**：コンポーネントのセレクターに一致する DOM 要素。
- **ビュー**：ホスト要素内のコンポーネントのテンプレートによってレンダリングされた DOM。
- **スタンドアロン**：デフォルトでコンポーネントはスタンドアロンです（Angular 19 以降、`standalone: true` がデフォルト）。古いバージョンでは `standalone: true` を明示するか、コンポーネントを `NgModule` の一部にする必要があります。
- **コンポーネントツリー**：Angular アプリケーションはコンポーネントのツリーとして構成され、各コンポーネントは子コンポーネントをホストできます。
- **コンポーネントの命名**：プロジェクトがその命名設定を使用するように構成されていない限り、コンポーネントクラスに `Component` サフィックスを追加しないでください（例：AppComponent）。
