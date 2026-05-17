# `effect` と `afterRenderEffect` によるサイドエフェクト

Angularにおける**エフェクト**とは、追跡している1つ以上のシグナル値が変化するたびに実行される処理です。

## `effect` を使うべき場面

エフェクトは、シグナルの状態を命令的な非シグナルAPIに同期させるために使います。

**有効なユースケース:**

- アナリティクスのログ記録。
- `localStorage` や `sessionStorage` への状態の同期。
- `<canvas>` やサードパーティのチャートライブラリへのカスタムレンダリングの実行。

**重要なルール: 状態の伝播にエフェクトを使わないこと。**
2つのシグナルを同期させるためにエフェクト内でシグナルの `.set()` や `.update()` を呼び出している場合、それは誤りです。`ExpressionChangedAfterItHasBeenChecked` エラーや無限ループを引き起こします。**状態の派生には常に `computed()` または `linkedSignal()` を使用してください。**

## 基本的な使い方

エフェクトは変更検出プロセス中に非同期で実行されます。常に少なくとも1回は実行されます。

```ts
import { Component, signal, effect } from '@angular/core';

@Component({...})
export class Example {
  count = signal(0);

  constructor() {
    // エフェクトはインジェクションコンテキスト（例: コンストラクター）内で作成する必要があります
    effect((onCleanup) => {
      console.log(`Count changed to ${this.count()}`);

      const timer = setTimeout(() => console.log('Timer finished'), 1000);

      // クリーンアップ関数は次回実行前または破棄時に実行されます
      onCleanup(() => clearTimeout(timer));
    });
  }
}
```

## `afterRenderEffect` によるDOM操作

標準の `effect` はAngularがDOMを更新する_前_に実行されます。シグナルの変化に基づいてDOMを手動で検査または変更する必要がある場合（例: サードパーティUIライブラリの統合）は、`afterRenderEffect` を使用してください。

`afterRenderEffect` はAngularがDOMのレンダリングを完了した後に実行されます。

### レンダーフェーズ

リフロー（強制レイアウトスラッシング）を防ぐため、`afterRenderEffect` はDOMの読み取りと書き込みを特定のフェーズに分割することを強制します。

```ts
import { Component, afterRenderEffect, viewChild, ElementRef } from '@angular/core';

@Component({...})
export class Chart {
  canvas = viewChild.required<ElementRef>('canvas');

  constructor() {
    afterRenderEffect({
      // 1. DOMから読み取る
      earlyRead: () => {
        return this.canvas().nativeElement.getBoundingClientRect().width;
      },
      // 2. DOMに書き込む（前フェーズの結果を受け取る）
      write: (width) => {
        // writeフェーズではDOMを読み取らないこと。
        setupChart(this.canvas().nativeElement, width);
      }
    });
  }
}
```

**利用可能なフェーズ（この順序で実行されます）:**

1. `earlyRead`
2. `write`（ここでは読み取らないこと）
3. `mixedReadWrite`（可能な限り避けること）
4. `read`（ここでは書き込まないこと）

_注意: `afterRenderEffect` はクライアントでのみ実行され、サーバーサイドレンダリング（SSR）中は実行されません。_
