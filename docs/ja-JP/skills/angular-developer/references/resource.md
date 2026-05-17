# `resource` を使った非同期リアクティビティ

> [!IMPORTANT]
> `resource` API は現在 Angular で実験的な機能です。

`Resource` は非同期データフェッチを Angular のシグナルベースのリアクティビティに組み込みます。依存関係が変わるたびに非同期ローダー関数を実行し、状態と結果を同期的なシグナルとして公開します。

## 基本的な使い方

`resource` 関数は主に2つのプロパティを持つオプションオブジェクトを受け取ります：

1. `params`：リアクティブな計算（`computed` のようなもの）。ここで読み取られるシグナルが変わると、リソースは再フェッチします。
2. `loader`：パラメーターに基づいてデータを取得する非同期関数。

```ts
import { Component, resource, signal, computed } from '@angular/core';

@Component({...})
export class UserProfile {
  userId = signal('123');

  userResource = resource({
    // userId をリアクティブに追跡
    params: () => ({ id: this.userId() }),

    // params が変わるたびに実行される
    loader: async ({ params, abortSignal }) => {
      const response = await fetch(`/api/users/${params.id}`, { signal: abortSignal });
      if (!response.ok) throw new Error('Network error');
      return response.json();
    }
  });

  // 計算済みシグナル内でリソースの値を使用する
  userName = computed(() => {
    if (this.userResource.hasValue()) {
      return this.userResource.value()?.name;
    } else {
      return 'Loading...';
    }
  });
}
```

## リクエストの中断

前のローダーがまだ実行中に `params` シグナルが変わった場合、`Resource` は提供された `abortSignal` を使って実行中のリクエストを中断しようとします。**`fetch` の呼び出しには常に `abortSignal` を渡してください。**

## データの再読み込み

params を変えずにローダーを強制的に再実行させるには、`.reload()` を呼び出します。

```ts
this.userResource.reload();
```

## リソースのステータスシグナル

`Resource` オブジェクトは現在の状態を読み取るためのいくつかのシグナルを提供します：

- `value()`：解決済みデータ、または `undefined`。
- `hasValue()`：型ガードとなるブール値。値が存在する場合は `true`。
- `isLoading()`：ローダーが現在実行中かどうかを示すブール値。
- `error()`：ローダーがスローしたエラー、または `undefined`。
- `status()`：正確な状態を表す文字列定数（`'idle'`、`'loading'`、`'resolved'`、`'error'`、`'reloading'`、`'local'`）。

## ローカルミューテーション

リソースの値を直接楽観的に更新できます。これにより状態が `'local'` に変わります。

```ts
this.userResource.value.set({name: 'Optimistic Update'});
```

## `httpResource` を使ったリアクティブなデータフェッチ

Angular の `HttpClient` を使用している場合は、`httpResource` の使用を優先してください。これはインターセプターを含む Angular の HTTP スタックを活用しながら、同じシグナルベースのリソース API を提供する専用ラッパーです。
