# アニメーションパターンリファレンス

プレゼンテーション生成時にこのリファレンスを使用してください。意図する印象に合わせてアニメーションを選択します。

## 効果と印象の対応表

| 印象 | アニメーション | ビジュアルの手がかり |
|---------|-----------|-------------|
| **ドラマチック / シネマティック** | スローフェードイン（1〜1.5秒）、大スケールのトランジション（0.9→1）、パララックススクロール | 暗い背景、スポットライト効果、フルブリード画像 |
| **テクノロジー系 / 未来的** | ネオングロウ（box-shadow）、グリッチ/スクランブルテキスト、グリッドリビール | パーティクルシステム（canvas）、グリッドパターン、等幅フォントのアクセント、シアン/マゼンタ/エレクトリックブルー |
| **遊び心 / フレンドリー** | バウンシーイージング（スプリング物理）、フローティング/ボビング | 角丸、パステル/ブライトカラー、手描き風要素 |
| **プロフェッショナル / コーポレート** | 繊細で速いアニメーション（200〜300ms）、クリーンなスライド | ネイビー/スレート/チャコール、正確な間隔、データビジュアライゼーション重視 |
| **穏やか / ミニマル** | 非常にゆっくりとした繊細な動き、ソフトなフェード | 広いホワイトスペース、くすんだカラーパレット、セリフタイポグラフィ、余裕のあるパディング |
| **エディトリアル / マガジン** | スタッガードテキストリビール、画像とテキストの連動 | 強いタイプ階層、プルクォート、グリッドを崩したレイアウト、セリフの見出し＋サンセリフの本文 |

## 入場アニメーション

```css
/* フェード + スライドアップ（最も汎用性が高い） */
.reveal {
    opacity: 0;
    transform: translateY(30px);
    transition: opacity 0.6s var(--ease-out-expo),
                transform 0.6s var(--ease-out-expo);
}
.visible .reveal {
    opacity: 1;
    transform: translateY(0);
}

/* スケールイン */
.reveal-scale {
    opacity: 0;
    transform: scale(0.9);
    transition: opacity 0.6s, transform 0.6s var(--ease-out-expo);
}
.visible .reveal-scale {
    opacity: 1;
    transform: scale(1);
}

/* 左からスライド */
.reveal-left {
    opacity: 0;
    transform: translateX(-50px);
    transition: opacity 0.6s, transform 0.6s var(--ease-out-expo);
}
.visible .reveal-left {
    opacity: 1;
    transform: translateX(0);
}

/* ブラーイン */
.reveal-blur {
    opacity: 0;
    filter: blur(10px);
    transition: opacity 0.8s, filter 0.8s var(--ease-out-expo);
}
.visible .reveal-blur {
    opacity: 1;
    filter: blur(0);
}
```

## 背景エフェクト

```css
/* グラデーションメッシュ — 奥行きのための放射状グラデーションの重ね合わせ */
.gradient-bg {
    background:
        radial-gradient(ellipse at 20% 80%, rgba(120, 0, 255, 0.3) 0%, transparent 50%),
        radial-gradient(ellipse at 80% 20%, rgba(0, 255, 200, 0.2) 0%, transparent 50%),
        var(--bg-primary);
}

/* ノイズテクスチャ — グレイン用のインラインSVG */
.noise-bg {
    background-image: url("data:image/svg+xml,..."); /* インラインSVGノイズ */
}

/* グリッドパターン — 繊細な構造的ライン */
.grid-bg {
    background-image:
        linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),
        linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px);
    background-size: 50px 50px;
}
```

## インタラクティブエフェクト

```javascript
/* ホバー時の3Dチルト — カード/パネルに奥行きを追加 */
class TiltEffect {
    constructor(element) {
        this.element = element;
        this.element.style.transformStyle = 'preserve-3d';
        this.element.style.perspective = '1000px';

        this.element.addEventListener('mousemove', (e) => {
            const rect = this.element.getBoundingClientRect();
            const x = (e.clientX - rect.left) / rect.width - 0.5;
            const y = (e.clientY - rect.top) / rect.height - 0.5;
            this.element.style.transform = `rotateY(${x * 10}deg) rotateX(${-y * 10}deg)`;
        });

        this.element.addEventListener('mouseleave', () => {
            this.element.style.transform = 'rotateY(0) rotateX(0)';
        });
    }
}
```

## トラブルシューティング

| 問題 | 解決策 |
|---------|-----|
| フォントが読み込まれない | FontshareまたはGoogle FontsのURLを確認。CSSでフォント名が一致しているか確認 |
| アニメーションが起動しない | Intersection Observerが動作しているか確認。`.visible` クラスが追加されているか確認 |
| スクロールスナップが機能しない | htmlに `scroll-snap-type: y mandatory` があるか確認。各スライドに `scroll-snap-align: start` が必要 |
| モバイルの問題 | 768pxのブレークポイントで重いエフェクトを無効化。タッチイベントをテスト。パーティクル数を減らす |
| パフォーマンスの問題 | `will-change` を控えめに使用。`transform`/`opacity` アニメーションを優先。スクロールハンドラをスロットリング |
