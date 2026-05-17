# テンプレート駆動フォーム

テンプレート駆動フォームは、双方向データバインディング（`[(ngModel)]`）を使用して、テンプレートで変更が加えられるとコンポーネントのデータモデルを更新し、その逆も同様に行います。シンプルなフォームに最適で、HTMLテンプレート内のディレクティブを使用してフォームの状態とバリデーションを管理します。

## 主要なディレクティブ

テンプレート駆動フォームは `FormsModule` に依存しており、以下の主要なディレクティブを提供します:

- `NgModel`: フォーム要素の値の変更をデータモデルと調整します（`[(ngModel)]`）。
- `NgForm`: `<form>` タグにバインドされたトップレベルの `FormGroup` を自動的に作成します。
- `NgModelGroup`: DOM要素にバインドされたネストされた `FormGroup` を作成します。

## セットアップ

まず、コンポーネントまたはモジュールに `FormsModule` をインポートします。

```ts
import {Component} from '@angular/core';
import {FormsModule} from '@angular/forms';

@Component({
  selector: 'app-user-form',
  imports: [FormsModule],
  templateUrl: './user-form.component.html',
})
export class UserForm {
  user = {name: '', role: 'Guest'};

  onSubmit() {
    console.log('Form submitted!', this.user);
  }
}
```

## フォームテンプレートの構築

### `[(ngModel)]` による双方向バインディング

入力要素に `[(ngModel)]` を使用します。**`[(ngModel)]` を使用するすべての要素には `name` 属性が必須です。** Angularは `name` 属性を使用して、親 `NgForm` にコントロールを登録します。

```html
<form #userForm="ngForm" (ngSubmit)="onSubmit()">
  <!-- 基本的な入力 -->
  <div>
    <label for="name">Name:</label>
    <input type="text" id="name" required [(ngModel)]="user.name" name="name" #nameCtrl="ngModel" />
  </div>

  <!-- セレクトボックス -->
  <div>
    <label for="role">Role:</label>
    <select id="role" [(ngModel)]="user.role" name="role">
      <option value="Admin">Admin</option>
      <option value="Guest">Guest</option>
    </select>
  </div>

  <!-- 送信ボタン（フォームが無効な場合は無効化） -->
  <button type="submit" [disabled]="!userForm.form.valid">Submit</button>
</form>
```

## フォームとコントロールの状態

Angularは状態に基づいてコントロールとフォームにCSSクラスを自動的に適用します:

| 状態 | Trueの場合のクラス | Falseの場合のクラス |
| :------------- | :-------------------------------- | :------------- |
| 訪問済み | `ng-touched` | `ng-untouched` |
| 値が変更済み | `ng-dirty` | `ng-pristine` |
| 値が有効 | `ng-valid` | `ng-invalid` |
| フォーム送信済み | `ng-submitted`（`<form>` のみ） | - |

これらのクラスをCSSでビジュアルフィードバックとして使用できます:

```css
.ng-valid[required],
.ng-valid.required {
  border-left: 5px solid #42a948; /* 緑 */
}
.ng-invalid:not(form) {
  border-left: 5px solid #a94442; /* 赤 */
}
```

## バリデーションとエラーメッセージ

条件付きでエラーメッセージを表示するには、`ngModel` ディレクティブをテンプレート参照変数（例: `#nameCtrl="ngModel"`）にエクスポートします。

```html
<input type="text" id="name" required [(ngModel)]="user.name" name="name" #nameCtrl="ngModel" />

<!-- コントロールが無効かつ（タッチ済みまたはダーティ）の場合のみエラーを表示 -->
@if (nameCtrl.invalid && (nameCtrl.dirty || nameCtrl.touched)) {
<div class="alert alert-danger">
  @if (nameCtrl.errors?.['required']) {
  <div>Name is required.</div>
  }
</div>
}
```

## フォームの送信

1. `<form>` 要素に `(ngSubmit)` イベントを使用します。
2. `NgForm` テンプレート参照変数（例: `[disabled]="!userForm.form.valid"`）を使用して、送信ボタンの無効状態をフォーム全体の有効性にバインドします。

## フォームのリセット

フォームをプリスティン状態（値とバリデーションフラグをクリア）にプログラムでリセットするには、`NgForm` インスタンスの `reset()` メソッドを使用します。

```html
<button type="button" (click)="userForm.reset()">Reset</button>
```
