# ルートへのナビゲーション

Angular はルート間を遷移するための宣言的な方法とプログラム的な方法の両方を提供します。

## 宣言的なナビゲーション（`RouterLink`）

アンカー要素に `RouterLink` ディレクティブを使用します。

```ts
import {RouterLink, RouterLinkActive} from '@angular/router';

@Component({
  imports: [RouterLink, RouterLinkActive],
  template: `
    <nav>
      <a routerLink="/dashboard" routerLinkActive="active-link">Dashboard</a>
      <a [routerLink]="['/user', userId]">Profile</a>
    </nav>
  `,
})
export class Nav {
  userId = '123';
}
```

- **絶対パス**：`/` で始まります（例：`/settings`）。
- **相対パス**：先頭に `/` がありません。一つ上のレベルに移動するには `../` を使います。

## プログラム的なナビゲーション（`Router`）

`Router` サービスを注入して TypeScript コードからナビゲートします。

### `router.navigate()`

コマンドの配列を使用します。

```ts
private router = inject(Router);
private route = inject(ActivatedRoute);

// 標準的なナビゲーション
this.router.navigate(['/profile']);

// パラメーター付き
this.router.navigate(['/search'], {
  queryParams: { q: 'angular' },
  fragment: 'results'
});

// 相対ナビゲーション
this.router.navigate(['edit'], { relativeTo: this.route });
```

### `router.navigateByUrl()`

文字列パスを使用します。絶対ナビゲーションや完全な URL に最適です。

```ts
this.router.navigateByUrl('/products/123?view=details');

// 履歴の現在のエントリを置き換える
this.router.navigateByUrl('/login', {replaceUrl: true});
```

## URL パラメーター

- **ルートパラメーター**：パスの一部（例：`/user/123`）。
- **クエリパラメーター**：`?` の後（例：`/search?q=query`）。
- **マトリックスパラメーター**：セグメントにスコープされる（例：`/products;category=books`）。
