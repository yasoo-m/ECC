# リアクティブフォーム

リアクティブフォームは、フォーム入力を処理するためのモデル駆動のアプローチを提供します。オブザーバブルストリームを基盤として構築され、データモデルへの同期アクセスを提供するため、テンプレート駆動フォームよりもスケーラブルでテストしやすい特徴があります。

## コアクラス

リアクティブフォームは `@angular/forms` の以下の基本クラスで構成されます：

- `FormControl`：個別の入力の値と有効性を管理します。
- `FormGroup`：コントロールのグループ（オブジェクトのような構造）を管理します。
- `FormArray`：数値インデックスによるコントロールの配列を管理します。
- `FormBuilder`：コントロールインスタンス作成のためのファクトリーメソッドを提供するサービス。

## セットアップ

コンポーネントに `ReactiveFormsModule` をインポートします。

```ts
import {Component, inject} from '@angular/core';
import {ReactiveFormsModule, FormGroup, FormControl, Validators, FormBuilder} from '@angular/forms';

@Component({
  selector: 'app-profile-editor',
  imports: [ReactiveFormsModule],
  templateUrl: './profile-editor.component.html',
})
export class ProfileEditor {
  private fb = inject(FormBuilder);

  // FormBuilder を使った簡潔な定義
  profileForm = this.fb.group({
    firstName: ['', Validators.required],
    lastName: [''],
    address: this.fb.group({
      street: [''],
      city: [''],
    }),
    aliases: this.fb.array([this.fb.control('')]),
  });

  onSubmit() {
    console.warn(this.profileForm.value);
  }
}
```

## テンプレートバインディング

モデルをビューにバインドするためのディレクティブを使用します：

- `[formGroup]`：`FormGroup` を `<form>` または `<div>` にバインドします。
- `formControlName`：グループ内の名前付きコントロールを入力にバインドします。
- `formGroupName`：ネストされた `FormGroup` をバインドします。
- `formArrayName`：ネストされた `FormArray` をバインドします。
- `[formControl]`：スタンドアロンの `FormControl` をバインドします。

```html
<form [formGroup]="profileForm" (ngSubmit)="onSubmit()">
  <input type="text" formControlName="firstName" />

  <div formGroupName="address">
    <input type="text" formControlName="street" />
  </div>

  <div formArrayName="aliases">
    @for (alias of aliases.controls; track $index) {
    <input type="text" [formControlName]="$index" />
    }
  </div>

  <button type="submit" [disabled]="!profileForm.valid">Submit</button>
</form>
```

## コントロールへのアクセス

特に `FormArray` のコントロールに簡単にアクセスするためのゲッターを使用します。

```ts
get aliases() {
  return this.profileForm.get('aliases') as FormArray;
}

addAlias() {
  this.aliases.push(this.fb.control(''));
}
```

## 値の更新

- `patchValue()`：指定されたプロパティのみを更新します。構造の不一致があっても警告なく無視します。
- `setValue()`：モデル全体を置き換えます。フォームの構造を厳密に強制します。

```ts
updateProfile() {
  this.profileForm.patchValue({
    firstName: 'Nancy',
    address: { street: '123 Drew Street' }
  });
}
```

## 統合変更イベント

モダンな Angular（v18+）では、すべてのコントロールに単一の `events` オブザーバブルが提供され、値、ステータス、pristine、touched、リセット、サブミットイベントを追跡できます。

```ts
import {ValueChangeEvent, StatusChangeEvent} from '@angular/forms';

this.profileForm.events.subscribe((event) => {
  if (event instanceof ValueChangeEvent) {
    console.log('New value:', event.value);
  }
});
```

## 手動ステート管理

- `markAsTouched()` / `markAllAsTouched()`：サブミット時にバリデーションエラーを表示するのに便利です。
- `markAsDirty()` / `markAsPristine()`：値が変更されたかどうかを追跡します。
- `updateValueAndValidity()`：値とステータスの再計算を手動でトリガーします。
- ほとんどのメソッドに `{ emitEvent: false }` または `{ onlySelf: true }` オプションを渡して伝播を制御できます。
