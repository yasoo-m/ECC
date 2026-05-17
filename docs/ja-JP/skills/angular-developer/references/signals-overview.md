# Angular シグナル概要

シグナルは、モダンなAngularアプリケーションにおけるリアクティビティの基盤です。**シグナル**とは、値が変更されたときに関心のあるコンシューマーに通知する、値のラッパーです。

## 書き込み可能なシグナル（`signal`）

`signal()` を使用して、直接更新できる状態を作成します。

```ts
import {signal} from '@angular/core';

// 書き込み可能なシグナルを作成
const count = signal(0);

// 値を読み取る（常にゲッター関数を呼び出す必要がある）
console.log(count());

// 値を直接更新する
count.set(3);

// 前の値に基づいて更新する
count.update((value) => value + 1);
```

### 読み取り専用として公開する

サービスから状態を公開する場合、外部からの変更を防ぐために読み取り専用バージョンを公開するのがベストプラクティスです。

```ts
private readonly _count = signal(0);
// コンシューマーはこれを読み取れるが、.set() や .update() は呼び出せない
readonly count = this._count.asReadonly();
```

## 算出シグナル（`computed`）

`computed()` を使用して、他のシグナルから値を導出する読み取り専用のシグナルを作成します。

- **遅延評価**: 導出関数は算出シグナルが読み取られるまで実行されません。
- **メモ化**: 結果はキャッシュされます。依存するシグナルのいずれかが変更されたときのみ再計算されます。
- **動的な依存関係**: 導出中に_実際に読み取られた_シグナルのみが追跡されます。

```ts
import {signal, computed} from '@angular/core';

const count = signal(0);
const doubleCount = computed(() => count() * 2);

// doubleCount は count が変更されると自動的に更新される。
```

## リアクティブコンテキスト

**リアクティブコンテキスト**とは、Angularが依存関係を確立するためにシグナルの読み取りを監視するランタイム状態です。

Angularは以下を評価する際に自動的にリアクティブコンテキストに入ります:

- `computed` シグナル
- `effect` コールバック
- `linkedSignal` の計算
- コンポーネントテンプレート

### 追跡なしの読み取り（`untracked`）

リアクティブコンテキスト内でシグナルを読み取る際に、依存関係を作成_せずに_（シグナルが変更されてもコンテキストが再実行されないように）読み取る必要がある場合は、`untracked()` を使用します。

```ts
import {effect, untracked} from '@angular/core';

effect(() => {
  // このエフェクトは currentUser が変更されたときのみ実行される。
  // counter がここで読み取られても、counter が変更されたときには実行されない。
  console.log(`User: ${currentUser()}, Count: ${untracked(counter)}`);
});
```

### リアクティブコンテキストでの非同期操作

リアクティブコンテキストは**同期**コードに対してのみ有効です。`await` の後のシグナル読み取りは追跡されません。**常に非同期境界の前にシグナルを読み取ってください。**

```ts
// 誤り: theme() は await の後に読み取られるため追跡されない
effect(async () => {
  const data = await fetchUserData();
  console.log(theme());
});

// 正しい: await の前にシグナルを読み取る
effect(async () => {
  const currentTheme = theme();
  const data = await fetchUserData();
  console.log(currentTheme);
});
```
