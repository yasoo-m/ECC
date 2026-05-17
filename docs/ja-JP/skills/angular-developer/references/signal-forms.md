# シグナルフォーム

シグナルフォームは、対象のAngularバージョンがサポートしている場合、新しいフォームに推奨されます。Angularシグナルを使用して、リアクティブかつ型安全なモデル駆動型のフォーム状態管理を提供します。

シグナルフォームを使用する場合、フィールドの値や型に `null` を使用しないでください。

## インポート

`@angular/forms/signals` から以下をインポートできます:

```ts
import {
  form,
  FormField,
  submit,
  // フィールド状態のルール
  disabled,
  hidden,
  readonly,
  debounce,
  // スキーマヘルパー
  applyWhen,
  applyEach,
  schema,
  // カスタムバリデーション
  validate,
  validateHttp,
  validateStandardSchema,
  // メタデータ
  metadata,
} from '@angular/forms/signals';
```

## フォームの作成

シグナルモデルとともに `form()` 関数を使用します。フォームの構造はモデルから直接導出されます。

```ts
import {Component, signal} from '@angular/core';
import {form, FormField} from '@angular/forms/signals';

@Component({
  // ...
  imports: [FormField],
})
export class Example {
  // 1. 初期値でモデルを定義する（undefinedは避ける）
  userModel = signal({
    name: '', // 重要: 初期値にnullやundefinedは絶対に使用しない
    email: '',
    age: 0, // 数値には0を使用し、nullは使用しない
    address: {
      street: '',
      city: '',
    },
    hobbies: [] as string[], // 配列には[]を使用し、nullは使用しない
  });

  // 誤り - このようにしてはいけない:
  // badModel = signal({
  //   name: null,      // エラー: '' を使用すること
  //   age: null,       // エラー: 0 を使用すること
  //   items: null      // エラー: [] を使用すること
  // });

  // 2. フォームを作成する
  userForm = form(this.userModel);
}
```

## バリデーション

バリデーターを `@angular/forms/signals` からインポートします。

```ts
import {required, email, min, max, minLength, maxLength, pattern} from '@angular/forms/signals';
```

`form()` に渡すスキーマ関数内で使用します:

```ts
userForm = form(this.userModel, (schemaPath) => {
  // 必須
  required(schemaPath.name, {message: 'Name is required'});

  // 条件付き必須
  required(schemaPath.name, {
    when({valueOf}) {
      return valueOf(schemaPath.age) > 10;
    },
  });
  // when は required にのみ使用可能
  // このようにしてはいけない: pattern(p.name, /xxx/, {when /* ERROR */)

  // メールアドレス
  email(schemaPath.email, {message: 'Invalid email'});

  // 数値の最小値/最大値
  min(schemaPath.age, 18);
  max(schemaPath.age, 100);

  // 文字列/配列の最小長/最大長
  minLength(schemaPath.password, 8);
  maxLength(schemaPath.description, 500);

  // パターン（正規表現）
  pattern(schemaPath.zipCode, /^\d{5}$/);
});
```

## FieldState と FormField: 親要素の要件

**FormField**（構造）と **FieldState**（実際のデータ/シグナル）の違いを理解することが重要です。

**ルール**: フィールドを関数として**呼び出す**ことで、そのステートシグナル（valid、touched、dirty、hidden など）にアクセスできます。

```ts
// f は FormField（構造的）
const f = form(signal({cat: {name: 'pirojok-the-cat', age: 5}}));

f.cat.name; // FormField: ここからフラグは取得できない！
f.cat.name.touched(); // エラー: touched() は FormField に存在しない

f.cat.name(); // FieldState: 呼び出すことでシグナルへのアクセスが可能になる
f.cat.name().touched(); // 有効: シグナルへのアクセス
f.cat().name.touched(); // エラー: f.cat() はステートであり、子要素を持たない！
```

テンプレートでも同様です:

```html
<!-- 誤り: 'hidden' プロパティは 'FormField' 型に存在しない -->
@if (bookingForm.hotelDetails.hidden()) { ... }

<!-- 正しい: 最初に呼び出す -->
@if (bookingForm.hotelDetails().hidden()) { ... }
```

## Disabled / Readonly / Hidden

スキーマ内のルールを使用してフィールドのステータスを制御します。

```ts
import {disabled, readonly, hidden} from '@angular/forms/signals';

userForm = form(this.userModel, (schemaPath) => {
  // 条件付き無効化
  disabled(schemaPath.password, ({valueOf}) => !valueOf(schemaPath.createAccount));

  // 条件付き非表示（モデルからは削除せず、非表示としてマークするのみ）
  hidden(schemaPath.shippingAddress, ({valueOf}) => valueOf(schemaPath.sameAsBilling));

  // 読み取り専用
  readonly(schemaPath.username);
});
```

## バインディング

`FormField` をインポートし、`[formField]` ディレクティブを使用します。

```ts
import {FormField} from '@angular/forms/signals';
```

`disabled`、`hidden`、`readonly`、`name` などのステート上のすべてのプロパティは自動的にバインドされます。
`name` フィールドは手動でバインド_しないでください_。

**重要: 禁止属性**
`[formField]` を使用する場合、テンプレートで以下の属性を設定してはなりません（静的またはバインドされた形式のいずれも）:

- `min`、`max`（代わりにスキーマ内のバリデーターを使用）
- `value`、`[value]`、`[attr.value]`（`[formField]` によって処理済み）
- `[attr.min]`、`[attr.max]`
- `[disabled]`、`[readonly]`（`[formField]` によって処理済み）

このようにしてはいけません: `<input min="1" [formField]>` や `<input [value]="val" [formField]>`。

```html
<!-- 入力 -->
<input [formField]="userForm.name" />

<!-- チェックボックス -->
<input type="checkbox" [formField]="userForm.isAdmin" />

<!-- セレクト -->
<select [formField]="userForm.country">
  <option value="us">US</option>
</select>

<!-- userForm.name は null にできない。input は null を受け付けないため -->
<input [formField]="userForm.name" />
```

## リアクティブフォーム

`@angular/forms` から `FormControl`、`FormGroup`、`FormArray`、`FormBuilder` を**インポートしないでください**。シグナルフォームはこれらのコンセプトを完全に置き換えます。
シグナルフォームにはビルダーがありません。

## ステートへのアクセス

フォーム内の各フィールドは、そのステートを返す関数です。

```ts
// フィールドを呼び出してアクセスする
const emailState = this.userForm.email();

// 値（WritableSignal）
const value = this.userForm().value();

// バリデーションステート（シグナル）
const isValid = this.userForm().valid();
const isInvalid = this.userForm().invalid();
const errors = this.userForm().errors(); // エラーの配列
const isPending = this.userForm().pending(); // 非同期バリデーション待ち

// インタラクションステート（シグナル）
const isTouched = this.userForm().touched();
const isDirty = this.userForm().dirty();

// 可用性ステート（シグナル）
const isDisabled = this.userForm().disabled();
const isHidden = this.userForm().hidden();
const isReadonly = this.userForm().readonly();
```

重要!: ステートを取得するには、必ずフィールドを呼び出してください。

```ts
form().invalid()
form.field().dirty()
form.field.subfield().touched()
form.a.b.c.d().value()
form.address.ssn().pending()
form().reset()

// 唯一の例外は length です:
form.children.length
form.length // 注意: 括弧なし！
form.client.addresses.length  // "()" なし

@for (income of form.addresses; track $index) {/**/}
```

## 送信

`submit()` 関数を使用します。アクションを実行する前に、すべてのフィールドを自動的にタッチ済みとしてマークします。

**重要**: `submit()` へのコールバックは `async` でなければならず、Promise を返す必要があります。

```ts
import { submit } from '@angular/forms/signals';

// 正しい - async コールバック
onSubmit() {
  submit(this.userForm, async () => {
    // フォームが有効な場合のみ実行される
    await this.apiService.save(this.userModel());
    console.log('Saved!');
  });
}

// 誤り - async キーワードが欠けている
onSubmit() {
  submit(this.userForm, () => {  // エラー: async でなければならない
    console.log('Saved!');
  });
}
```

## エラー処理

`field().errors()` は ValidationError の配列を返します:

```ts
interface ValidationError {
  readonly kind: string;
  readonly message?: string;
}
```

バリデーターから null を返さないでください。
エラーがない場合は undefined を返してください。

### コンテキスト

`validate()`、`disabled()`、`applyWhen` などのルールに渡される関数は、コンテキストオブジェクトを受け取ります。その構造を理解することが**重要**です:

```ts
validate(
  schemaPath.username,
  ({
    value, // Signal<T>: フィールドの現在値（書き込み可能）
    fieldTree, // FieldTree<T>: サブフィールド（グループ/配列の場合）
    state, // FieldState<T>: state.valid()、state.dirty() などのフラグへのアクセス
    valueOf, // (path) => T: 他のフィールドの値を読む（依存関係を追跡）例: valueOf(schemaPath.password)
    stateOf, // (path) => FieldState: 他のフィールドのステートにアクセス 例: stateOf(schemaPath.password).valid()
    pathKeys, // Signal<string[]>: ルートからこのフィールドへのパス
  }) => {
    // 誤り: if (touched()) ... （touched はコンテキスト内にない）
    // 正しい: if (state.touched()) ...

    if (value() === 'admin') {
      return {kind: 'reserved', message: 'Username admin is reserved'};
    }
  },
);
```

### 重要: パスはシグナルではない

`form()` コールバック内で、`schemaPath` とその子要素（例: `schemaPath.user.name`）は**シグナルではなく**、**呼び出し可能でもありません**。

```ts
// 誤り - エラーが発生します:
applyWhen(p.ssn, () => p.ssn().touched(), (ssnField) => { ... });

// 正しい - stateOf() を使用してパスのステートを取得する:
applyWhen(p.ssn, ({ stateOf }) => stateOf(p.ssn).touched(), (ssnField) => { ... });

// 正しい - valueOf() を使用してパスの値を取得する:
applyWhen(p.ssn, ({ valueOf }) => valueOf(p.ssn) !== '', (ssnField) => { ... });
```

### 複数アイテム

- アイテムごとにルールを適用するには `applyEach` を使用します。
- **重要**: `applyEach` のコールバックは引数を**1つだけ**取ります（アイテムパス）。2つではありません:

```ts
// 正しい - 引数1つ
applyEach(s.items, (item) => {
  required(item.name);
});

// 誤り - インデックスを渡さない
applyEach(s.items, (item, index) => {
  // エラー: コールバックは引数を1つしか取らない
  required(item.name);
});
```

- テンプレートではアイテムの反復に `@for` を使用します。
- 配列からアイテムを削除するには、データ内の該当アイテムをそのまま削除してください。
- **`select` バインディング**: `<select [formField]="form.country">` にバインドできます。オプションには `value` 属性があることを確認してください。

### ネストした @for ループ

**重要**: AngularにはAngularには `$parent` がありません。ネストされたループでは、外側のインデックスを変数に保存してください:

```html
<!-- 誤り - $parent は存在しない -->
@for (item of form.items; track $index) { @for (option of item.options; track $index) {
<button (click)="removeOption($parent.$index, $index)">Remove</button>
<!-- エラー -->
} }

<!-- 正しい - let を使用して外側のインデックスを保存する -->
@for (item of form.items; track $index; let outerIndex = $index) { @for (option of item.options;
track $index) {
<button (click)="removeOption(outerIndex, $index)">Remove</button>
} }
```

### フォームボタンの無効化

```html
<button [disabled]="form().invalid() || form().pending()" />
<!-- または -->
<button [disabled]="taxForm.invalid()" />
```

inputに `[disabled]` を使用しないでください。`[formField]` がこれを行います。
inputに `[readonly]` を使用しないでください。`[formField]` がこれを行います。
フィールドを無効化または読み取り専用にする必要がある場合は、スキーマ内の `disabled()` または `readonly()` ルールを使用してください。

### 非同期バリデーション

非同期には `validate()` を使用せず、代わりに `validateAsync()` を使用してください:

**重要**:

1. `params` オプションはバリデートする値を返す関数でなければなりません。
2. `onError` ハンドラーは**必須**です - 省略不可です！

```ts
import {resource} from '@angular/core';
import {validateAsync} from '@angular/forms/signals';

userForm = form(this.userModel, (s) => {
  validateAsync(s.username, {
    // 1. 関数でなければならない - params はコンテキストを受け取り、値を返す
    params: ({value}) => value(),

    // 2. リソースを作成する - ファクトリーはシグナルを受け取る
    factory: (username) =>
      resource({
        params: username, // resource() では 'params' を使用
        loader: async ({params: value}) => {
          await new Promise((resolve) => setTimeout(resolve, 1000));
          return value === 'taken';
        },
      }),

    // 3. 成功をエラーにマッピングする
    onSuccess: (isTaken) =>
      isTaken ? {kind: 'taken', message: 'Username is already taken'} : undefined,

    // 4. エラー処理 - これは必須です！
    onError: () => ({kind: 'error', message: 'Validation failed'}),
  });
});
```

**誤りの例:**

```ts
// 誤り - params は関数でなければならない
validateAsync(s.username, {
  params: s.username, // エラー: ({ value }) => value() でなければならない
  // ...
});

// 誤り - onError が欠けている（必須です！）
validateAsync(s.username, {
  params: ({value}) => value(),
  factory: (username) =>
    resource({
      /* ... */
    }),
  onSuccess: (result) => (result ? {kind: 'error'} : undefined),
  // エラー: 'onError' が欠けているが必須！
});
```

### リソースの使用

**重要**: Angularの `resource()` では、入力シグナルに `params` を使用してください。

```ts
// 正しい
resource({
  params: mySignal,
  loader: async ({params: value}) => {
    /* ... */
  },
});

// 誤り
resource({
  request: mySignal, // エラー: 'params' にすべき
  loader: async ({request}) => {
    /* ... */
  },
});
```

UIとモデル間の同期を遅延させるには `debounce()` を使用します。

```ts
import {debounce} from '@angular/forms/signals';

userForm = form(this.userModel, (s) => {
  // モデルの更新を300ms遅延させる
  debounce(s.username, 300);
});
```

### 条件付きバリデーション

```ts
form(
  data,
  (path) => {
    applyWhen(
      name,
      ({value}) => value() !== 'admin',
      (namePath) => {
        validate(namePath.last /* ... */);
        disable(namePath.last /* ... */);
      },
    );
  },
  {injector: TestBed.inject(Injector)},
);
```

`applyWhen` は最初の引数にマッピングされたパスを渡します。
親フィールドが必要な場合は、それを `applyWhen` に渡してください:

```ts
form(
  data,
  (path) => {
    applyWhen(
      cat,
      ({value}) => value().name !== 'admin',
      (catPath) => {
        require(cat.catPath /* ... */);
      },
    );
  },
  {injector: TestBed.inject(Injector)},
);
```

## よくある落とし穴（やってはいけないこと）

| エラーシナリオ             | 誤り（よくあるミス）                                          | 正しい方法                                                  |
| :------------------------- | :------------------------------------------------------------ | :---------------------------------------------------------- |
| **フラグへのアクセス**     | `form.field.valid()`                                          | `form.field().valid()`                                      |
| **値へのアクセス**         | `form.field.value()`                                          | `form.field().value()`                                      |
| **値の設定**               | `form.field.set(x)`                                           | モデルシグナルを更新: `this.model.update(...)`               |
| **フォームルートフラグ**   | `form.invalid()`                                              | `form().invalid()`                                          |
| **二重呼び出し**           | `form.field()()`                                              | `form.field().value()`                                      |
| **ルールコンテキスト**     | `({ touched }) => touched()`                                  | `({ state }) => state.touched()`                            |
| **パスの呼び出し**         | `applyWhen(p.foo, () => p.foo() === 'x')`                     | `applyWhen(p.foo, ({ valueOf }) => valueOf(p.foo) === 'x')` |
| **applyWhen の引数**       | `applyWhen(condition, () => {...})`                           | `applyWhen(path, condition, schemaFn)` - 引数3つ必要        |
| **配列の長さ**             | `form.items().length`                                         | `form.items.length`（構造的）                               |
| **複数選択配列**           | `<select [formField]="form.tags">` (string[])                 | 配列フィールドにはチェックボックスを使用                     |
| **readonly 属性**          | `<input readonly [formField]>`                                | スキーマで `readonly()` ルールを使用                         |
| **min/max 属性**           | `<input min="1" max="10">`                                    | スキーマで `min()` と `max()` ルールを使用                   |
| **value バインディング**   | `<input [value]="val">`                                       | `[formField]` と共に `[value]` を使用しない                  |
| **when オプション**        | `pattern(p.x, /.../, {when: ...})`                            | `when` は `required()` でのみ動作する                       |
| **Submit コールバック**    | `submit(form, () => { ... })`                                 | `submit(form, async () => { ... })`                         |
| **Async params**           | `params: s.field`                                             | `params: ({ value }) => value()`                            |
| **Async onError**          | `onError` を省略する                                          | `validateAsync` では `onError` は必須                        |
| **resource() API**         | `request: signal`                                             | `params: signal`                                            |
| **applyEach の引数**       | `applyEach(s.items, (item, index) => ...)`                    | `applyEach(s.items, (item) => ...)`                         |
| **ネストした @for**        | `$parent.$index`                                              | `let outerIndex = $index` を使用                             |
| **FormState インポート**   | `import { FormState }`                                        | `FormState` は存在しない。`FieldState` を使用                |
| **モデル内の Null**        | `signal({ name: null })`                                      | `signal({ name: '' })` または `signal({ age: 0 })`           |
| **Validate 構文**          | `validate(s.field, { value } => ...)`                         | `validate(s.field, ({ value }) => ...)`                     |
| **チェックボックス配列**   | `[formField]="form.tags"` (string[])                          | チェックボックスは `boolean` にのみバインドする               |

## 大規模フォームの例

### `src/app/app.ts`

```ts
import {Component, signal, ChangeDetectionStrategy} from '@angular/core';
import {
  form,
  FormField,
  submit,
  required,
  email,
  min,
  hidden,
  applyEach,
  validate,
} from '@angular/forms/signals';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [FormField],
  templateUrl: './app.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class App {
  model = signal({
    personalInfo: {
      firstName: '',
      lastName: '',
      email: '',
      age: 0,
    },
    tripDetails: {
      destination: 'Mars',
      launchDate: '',
    },
    package: {
      tier: 'economy',
      extras: [] as string[],
    },
    companions: [] as Array<{name: string; relation: string}>,
  });

  bookingForm = form(this.model, (s) => {
    required(s.personalInfo.firstName, {message: 'First name is required'});
    required(s.personalInfo.lastName, {message: 'Last name is required'});
    required(s.personalInfo.email, {message: 'Email is required'});
    email(s.personalInfo.email, {message: 'Invalid email address'});
    required(s.personalInfo.age, {message: 'Age is required'});
    min(s.personalInfo.age, 18, {message: 'Must be at least 18'});

    required(s.tripDetails.destination);
    required(s.tripDetails.launchDate);
    validate(s.tripDetails.launchDate, ({value}) => {
      const date = new Date(value());
      if (isNaN(date.getTime())) return undefined;
      const today = new Date();
      if (date < today) {
        return {kind: 'pastData', message: 'Launch date must be in the future'};
      }
      return undefined;
    });

    // valueOf は、ルール内で他のフィールドの値にアクセスするために使用される
    hidden(s.package.extras, ({valueOf}) => valueOf(s.package.tier) === 'economy');

    applyEach(s.companions, (companion) => {
      required(companion.name, {message: 'Companion name required'});
      required(companion.relation, {message: 'Relation required'});
    });
  });

  addCompanion() {
    this.model.update((m) => ({
      ...m,
      companions: [...m.companions, {name: '', relation: ''}],
    }));
  }

  removeCompanion(index: number) {
    this.model.update((m) => ({
      ...m,
      companions: m.companions.filter((_, i) => i !== index),
    }));
  }

  onSubmit() {
    // 重要: submit コールバックは async でなければならない
    submit(this.bookingForm, async () => {
      console.log('Booking Confirmed:', this.model());
      // 非同期処理が必要な場合:
      // await this.apiService.save(this.model());
    });
  }
}
```

### `src/app/app.html`

```html
<form (submit)="onSubmit(); $event.preventDefault()">
  <h1>Interstellar Booking</h1>

  <section>
    <h2>Personal Info</h2>

    <label>
      First Name
      <input [formField]="bookingForm.personalInfo.firstName" />
      @if (bookingForm.personalInfo.firstName().touched() &&
      bookingForm.personalInfo.firstName().errors().length) {
      <span>{{ bookingForm.personalInfo.firstName().errors()[0].message }}</span>
      }
    </label>

    <label>
      Last Name
      <input [formField]="bookingForm.personalInfo.lastName" />
      @if (bookingForm.personalInfo.lastName().touched() &&
      bookingForm.personalInfo.lastName().errors().length) {
      <span>{{ bookingForm.personalInfo.lastName().errors()[0].message }}</span>
      }
    </label>

    <label>
      Email
      <input type="email" [formField]="bookingForm.personalInfo.email" />
      @if (bookingForm.personalInfo.email().touched() &&
      bookingForm.personalInfo.email().errors().length) {
      <span>{{ bookingForm.personalInfo.email().errors()[0].message }}</span>
      }
    </label>

    <label>
      Age
      <input type="number" [formField]="bookingForm.personalInfo.age" />
      @if (bookingForm.personalInfo.age().touched() &&
      bookingForm.personalInfo.age().errors().length) {
      <span>{{ bookingForm.personalInfo.age().errors()[0].message }}</span>
      }
    </label>
  </section>

  <section>
    <h2>Trip Details</h2>

    <label>
      Destination
      <select [formField]="bookingForm.tripDetails.destination">
        <option value="Mars">Mars</option>
        <option value="Moon">Moon</option>
        <option value="Titan">Titan</option>
      </select>
    </label>

    <label>
      Launch Date
      <input type="date" [formField]="bookingForm.tripDetails.launchDate" />
      @if (bookingForm.tripDetails.launchDate().touched() &&
      bookingForm.tripDetails.launchDate().errors().length) {
      <span>{{ bookingForm.tripDetails.launchDate().errors()[0].message }}</span>
      }
    </label>
  </section>

  <section>
    <h2>Package</h2>

    <label>
      <input type="radio" value="economy" [formField]="bookingForm.package.tier" />
      Economy
    </label>
    <label>
      <input type="radio" value="business" [formField]="bookingForm.package.tier" />
      Business
    </label>
    <label>
      <input type="radio" value="first" [formField]="bookingForm.package.tier" />
      First Class
    </label>

    @if (!bookingForm.package.extras().hidden()) {
    <div>
      <h3>Extras</h3>
      <!-- 配列の複数選択には select multiple を使用する -->
      <select multiple [formField]="bookingForm.package.extras">
        <option value="wifi">WiFi</option>
        <option value="gym">Gym</option>
      </select>
    </div>
    }
  </section>

  <section>
    <h2>Companions</h2>
    <button type="button" (click)="addCompanion()">Add Companion</button>

    @for (companion of bookingForm.companions; track $index) {
    <div>
      <input [formField]="companion.name" placeholder="Name" />
      @if (companion.name().touched() && companion.name().errors().length) {
      <span>{{ companion.name().errors()[0].message }}</span>
      }

      <input [formField]="companion.relation" placeholder="Relation" />
      @if (companion.relation().touched() && companion.relation().errors().length) {
      <span>{{ companion.relation().errors()[0].message }}</span>
      }

      <button type="button" (click)="removeCompanion($index)">Remove</button>
    </div>
    }
  </section>

  <button [disabled]="bookingForm().invalid()">Submit</button>
</form>
```

## ビルドエラーからの回復

ビルドエラーが発生した場合、最も一般的な修正方法を示します:

### `Property 'value' does not exist on type 'FieldTree'`

**問題**: 最初に呼び出さずにフィールドの `.value()` に直接アクセスしています。

```ts
// 誤り
const val = this.form.field.value();
// 正しい
const val = this.form.field().value();
```

### `Property 'set' does not exist on type 'FieldTree'`

**問題**: フォームツリーに対して値を設定しようとしています。シグナルフォームはモデル駆動型です。

```ts
// 誤り
this.form.address.street.set('Main St');
// 正しい - 代わりにモデルシグナルを更新する
this.model.update((m) => ({...m, address: {...m.address, street: 'Main St'}}));
```

### `Type 'string[]' is not assignable to type 'string'`

**問題**: 配列フィールドに単一値の `<select>` で `[formField]` をバインドしています。

```html
<!-- 誤り - assignees は string[]、select は string を期待している -->
<select [formField]="form.assignees">
  ...
</select>

<!-- 正しい - 配列フィールドには select multiple を使用する -->
<select multiple [formField]="form.assignees">
  <option value="us">US</option>
</select>
```
