# HTMLプレゼンテーションテンプレート

スライドプレゼンテーション生成のリファレンスアーキテクチャ。すべてのプレゼンテーションはこの構造に従います。

## ベースHTML構造

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Presentation Title</title>

    <!-- フォント: FontshareまたはGoogle Fontsを使用 — システムフォントは不可 -->
    <link rel="stylesheet" href="https://api.fontshare.com/v2/css?f[]=..." />

    <style>
      /* ===========================================
           CSSカスタムプロパティ（テーマ）
           ここを変更することで全体の見た目が変わります
           =========================================== */
      :root {
        /* カラー — 選択したスタイルプリセットから */
        --bg-primary: #0a0f1c;
        --bg-secondary: #111827;
        --text-primary: #ffffff;
        --text-secondary: #9ca3af;
        --accent: #00ffcc;
        --accent-glow: rgba(0, 255, 204, 0.3);

        /* タイポグラフィ — clamp()を必ず使用 */
        --font-display: "Clash Display", sans-serif;
        --font-body: "Satoshi", sans-serif;
        --title-size: clamp(2rem, 6vw, 5rem);
        --subtitle-size: clamp(0.875rem, 2vw, 1.25rem);
        --body-size: clamp(0.75rem, 1.2vw, 1rem);

        /* 間隔 — clamp()を必ず使用 */
        --slide-padding: clamp(1.5rem, 4vw, 4rem);
        --content-gap: clamp(1rem, 2vw, 2rem);

        /* アニメーション */
        --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
        --duration-normal: 0.6s;
      }

      /* ===========================================
           ベーススタイル
           =========================================== */
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }

      /* --- ここにviewport-base.cssの内容を貼り付け --- */

      /* ===========================================
           アニメーション
           .visibleクラスでトリガー（JSがスクロール時に追加）
           =========================================== */
      .reveal {
        opacity: 0;
        transform: translateY(30px);
        transition:
          opacity var(--duration-normal) var(--ease-out-expo),
          transform var(--duration-normal) var(--ease-out-expo);
      }

      .slide.visible .reveal {
        opacity: 1;
        transform: translateY(0);
      }

      /* 順番に表示するための子要素のスタッガー */
      .reveal:nth-child(1) {
        transition-delay: 0.1s;
      }
      .reveal:nth-child(2) {
        transition-delay: 0.2s;
      }
      .reveal:nth-child(3) {
        transition-delay: 0.3s;
      }
      .reveal:nth-child(4) {
        transition-delay: 0.4s;
      }

      /* ... プリセット固有のスタイル ... */
    </style>
  </head>
  <body>
    <!-- オプション: プログレスバー -->
    <div class="progress-bar"></div>

    <!-- オプション: ナビゲーションドット -->
    <nav class="nav-dots"><!-- JSで生成 --></nav>

    <!-- スライド -->
    <section class="slide title-slide">
      <h1 class="reveal">Presentation Title</h1>
      <p class="reveal">Subtitle or author</p>
    </section>

    <section class="slide">
      <div class="slide-content">
        <h2 class="reveal">Slide Title</h2>
        <p class="reveal">Content...</p>
      </div>
    </section>

    <!-- その他のスライド... -->

    <script>
      /* ===========================================
           スライドプレゼンテーションコントローラー
           =========================================== */
      class SlidePresentation {
        constructor() {
          this.slides = document.querySelectorAll(".slide");
          this.currentSlide = 0;
          this.setupIntersectionObserver();
          this.setupKeyboardNav();
          this.setupTouchNav();
          this.setupProgressBar();
          this.setupNavDots();
        }

        setupIntersectionObserver() {
          // スライドがビューポートに入ったときに.visibleクラスを追加
          // CSSアニメーションを効率的にトリガー
        }

        setupKeyboardNav() {
          // 矢印キー、スペース、Page Up/Down
        }

        setupTouchNav() {
          // モバイル向けタッチ/スワイプサポート
        }

        setupProgressBar() {
          // スクロール時にプログレスバーを更新
        }

        setupNavDots() {
          // 重要: ビルド前に必ずクリアする — outerHTMLがドットのレンダリング中にキャプチャされた場合、
          // ファイルを再度開くと既存のドットの上に重複したセットが追加されてしまう。
          this.navDotsContainer.innerHTML = "";
          // ナビゲーションドットを生成・管理
        }
      }

      new SlidePresentation();
    </script>
  </body>
</html>
```

## 必須JavaScriptの機能

すべてのプレゼンテーションに以下が必要です：

1. **SlidePresentationクラス** — メインコントローラー（以下を含む）：
   - キーボードナビゲーション（矢印キー、スペース、Page Up/Down）
   - タッチ/スワイプサポート
   - マウスホイールナビゲーション
   - プログレスバーの更新
   - ナビゲーションドット

2. **Intersection Observer** — スクロールトリガーアニメーション用：
   - スライドがビューポートに入ったときに `.visible` クラスを追加
   - CSSトランジションを効率的にトリガー

3. **オプションの拡張機能**（選択したスタイルに合わせる）：
   - カスタムカーソルとトレイル
   - パーティクルシステム背景（canvas）
   - パララックスエフェクト
   - ホバー時の3Dチルト
   - マグネティックボタン
   - カウンターアニメーション

4. **インライン編集**（フェーズ1でユーザーが選択した場合のみ — 「いいえ」の場合は完全にスキップ）：
   - 編集トグルボタン（デフォルト非表示、ホバーホットゾーンまたはEキーで表示）
   - localStorageへの自動保存
   - ファイルのエクスポート/保存機能
   - 以下の「インライン編集の実装」セクションを参照

## インライン編集の実装（オプトインのみ）

**フェーズ1でユーザーがインライン編集に「いいえ」を選択した場合、編集関連のHTML、CSS、JSを一切生成しないでください。**

**CSS `~` 兄弟セレクターをホバーベースの表示/非表示に使用しないでください。** CSS単体のアプローチ（`edit-hotzone:hover ~ .edit-toggle`）は、トグルボタンの `pointer-events: none` がホバーチェーンを壊すため失敗します：ユーザーがホットゾーンにホバー → ボタンが表示 → マウスがボタンに向かって移動 → ホットゾーンを離れる → クリック前にボタンが消える。

**必須アプローチ：400msの遅延タイムアウトを持つJSベースのホバー。**

HTML：

```html
<div class="edit-hotzone"></div>
<button class="edit-toggle" id="editToggle" title="Edit mode (E)">Edit</button>
```

CSS（表示/非表示はJSクラスのみで制御）：

```css
/* ここではCSS ~兄弟セレクターを使用しないでください！
   pointer-events: noneがホバーチェーンを壊します。
   遅延タイムアウトを持つJSを使用する必要があります。 */
.edit-hotzone {
  position: fixed;
  top: 0;
  left: 0;
  width: 80px;
  height: 80px;
  z-index: 10000;
  cursor: pointer;
}
.edit-toggle {
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.3s ease;
  z-index: 10001;
}
.edit-toggle.show,
.edit-toggle.active {
  opacity: 1;
  pointer-events: auto;
}
```

JS（3つのインタラクション方法）：

```javascript
// 1. トグルボタンのクリックハンドラー
document.getElementById("editToggle").addEventListener("click", () => {
  editor.toggleEditMode();
});

// 2. 400msのグレース期間を持つホットゾーンのホバー
const hotzone = document.querySelector(".edit-hotzone");
const editToggle = document.getElementById("editToggle");
let hideTimeout = null;

hotzone.addEventListener("mouseenter", () => {
  clearTimeout(hideTimeout);
  editToggle.classList.add("show");
});
hotzone.addEventListener("mouseleave", () => {
  hideTimeout = setTimeout(() => {
    if (!editor.isActive) editToggle.classList.remove("show");
  }, 400);
});
editToggle.addEventListener("mouseenter", () => {
  clearTimeout(hideTimeout);
});
editToggle.addEventListener("mouseleave", () => {
  hideTimeout = setTimeout(() => {
    if (!editor.isActive) editToggle.classList.remove("show");
  }, 400);
});

// 3. ホットゾーンの直接クリック
hotzone.addEventListener("click", () => {
  editor.toggleEditMode();
});

// 4. キーボードショートカット（Eキー、テキスト編集中はスキップ）
document.addEventListener("keydown", (e) => {
  if (
    (e.key === "e" || e.key === "E") &&
    !e.target.getAttribute("contenteditable")
  ) {
    editor.toggleEditMode();
  }
});
```

**重要: `exportFile()` はouterHTMLをキャプチャする前に編集状態を除去する必要があります。**

ユーザーが編集モードでCtrl+Sを押すと、`document.documentElement.outerHTML` がライブDOM（`body.edit-active`、すべてのテキスト要素の `contenteditable="true"`、トグルボタンとバナーの `.active`/`.show` クラスを含む）をキャプチャします。保存されたファイルを開く人は、点線のアウトライン、チェックマークボタン、編集バナーが表示され、永久に編集モードのように見えます。

`exportFile()` は常にこのように実装してください：

```javascript
exportFile() {
    // 保存されたファイルがクリーンに開くよう、一時的に編集状態を除去
    const editableEls = Array.from(document.querySelectorAll('[contenteditable]'));
    editableEls.forEach(el => el.removeAttribute('contenteditable'));
    document.body.classList.remove('edit-active');

    // トグルボタンとバナーからUIクラスも除去
    const editToggle = document.getElementById('editToggle');
    const editBanner = document.querySelector('.edit-banner');
    editToggle?.classList.remove('active', 'show');
    editBanner?.classList.remove('active', 'show');

    const html = '<!DOCTYPE html>\n' + document.documentElement.outerHTML;

    // ユーザーが編集を続けられるよう編集状態を復元
    document.body.classList.add('edit-active');
    editableEls.forEach(el => el.setAttribute('contenteditable', 'true'));
    editToggle?.classList.add('active');
    editBanner?.classList.add('active');

    const blob = new Blob([html], { type: 'text/html' });
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = 'presentation.html';
    a.click();
    URL.revokeObjectURL(a.href);
}
```

## 画像パイプライン（画像がない場合はスキップ）

フェーズ1でユーザーが「画像なし」を選択した場合は完全にスキップします。画像が提供された場合は、HTML生成前に処理します。

**依存関係：** `pip install Pillow`

### 画像処理

```python
from PIL import Image, ImageDraw

# 円形クロップ（モダン/クリーンなスタイルのロゴ用）
def crop_circle(input_path, output_path):
    img = Image.open(input_path).convert('RGBA')
    w, h = img.size
    size = min(w, h)
    left, top = (w - size) // 2, (h - size) // 2
    img = img.crop((left, top, left + size, top + size))
    mask = Image.new('L', (size, size), 0)
    ImageDraw.Draw(mask).ellipse([0, 0, size, size], fill=255)
    img.putalpha(mask)
    img.save(output_path, 'PNG')

# リサイズ（HTMLを肥大化させる大きすぎる画像用）
def resize_max(input_path, output_path, max_dim=1200):
    img = Image.open(input_path)
    img.thumbnail((max_dim, max_dim), Image.LANCZOS)
    img.save(output_path, quality=85)
```

| 状況 | 操作 |
| -------------------------------- | ----------------------------- |
| 角丸デザインの正方形ロゴ | `crop_circle()` |
| 1MB超の画像 | `resize_max(max_dim=1200)` |
| アスペクト比が不正 | `img.crop()` で手動クロップ |

処理済み画像は `_processed` サフィックスで保存します。元ファイルは上書きしないでください。

### 画像の配置

**ローカルで表示されるため直接ファイルパスを使用**（base64は不可）：

```html
<img src="assets/logo_round.png" alt="Logo" class="slide-image logo" />
<img
  src="assets/screenshot.png"
  alt="Screenshot"
  class="slide-image screenshot"
/>
```

```css
.slide-image {
  max-width: 100%;
  max-height: min(50vh, 400px);
  object-fit: contain;
  border-radius: 8px;
}
.slide-image.screenshot {
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}
.slide-image.logo {
  max-height: min(30vh, 200px);
}
```

**ボーダー/シャドウカラーは選択したスタイルのアクセントカラーに合わせてください。** 複数のスライドで同じ画像を繰り返さないでください（タイトルとクロージングのロゴは例外）。

**配置パターン：** タイトルスライドでロゴを中央に。スクリーンショットはテキストとの2カラムレイアウトで。フルブリード画像はテキストオーバーレイのあるスライド背景として（控えめに）。

---

## コード品質

**コメント：** 各セクションに何をするのか、どう変更するのかを説明する明確なコメントを付けてください。

**アクセシビリティ：**

- セマンティックHTML（`<section>`、`<nav>`、`<main>`）
- キーボードナビゲーションが完全に機能する
- 必要な箇所にARIAラベル
- `prefers-reduced-motion` サポート（viewport-base.cssに含まれる）

## ファイル構成

単一プレゼンテーション：

```
presentation.html    # 自己完結型、CSS/JSすべてインライン
assets/              # 画像のみ（ある場合）
```

1つのプロジェクト内に複数のプレゼンテーション：

```
[name].html
[name]-assets/
```
