# Angular Aria

Angular Aria（`@angular/aria`）は、一般的なWAI-ARIAパターンを実装するヘッドレスでアクセシブルなディレクティブのコレクションです。これらのディレクティブは、キーボードインタラクション、ARIA属性、フォーカス管理、スクリーンリーダーのサポートを処理します。

**AIエージェントとしての役割は、HTMLの構造とCSSスタイリングを提供すること**であり、ディレクティブが複雑なアクセシビリティロジックを処理します。

## ヘッドレスコンポーネントのスタイリング

Angular Ariaコンポーネントはヘッドレスであるため、デフォルトのスタイルは付属していません。ディレクティブが自動的に適用するARIA属性や構造クラスに基づいて、CSSで各状態をスタイリングする**必要があります**。

CSSでターゲットとする一般的なARIA属性：

- `[aria-expanded="true"]` / `[aria-expanded="false"]`
- `[aria-selected="true"]`
- `[aria-disabled="true"]`
- `[aria-current="page"]`（ナビゲーション用）

---

**重要**: このパッケージを使用する前に、パッケージマネージャーでインストールする必要があります。プロジェクトにインストールされていることを確認してください。必要な場合は `npm install @angular/aria` でインストールしてください。

## 1. アコーディオン

関連するコンテンツを展開・折りたたみ可能なセクションに整理します。

**使用場面:** アコーディオンは、コンテンツの多いページでスクロールを減らすために、ユーザーが一度に1つずつ展開できる論理的なグループにコンテンツを整理するレイアウトコンポーネントです。FAQ、長いフォーム、または情報の段階的な開示に使用しますが、プライマリナビゲーションや、ユーザーが複数のコンテンツセクションを同時に表示する必要があるシナリオには使用しないでください。

**インポート:** `import { AccordionContent, AccordionGroup, AccordionPanel, AccordionTrigger } from '@angular/aria/accordion';`

**ディレクティブ:** `ngAccordionGroup`、`ngAccordionTrigger`、`ngAccordionPanel`、`ngAccordionContent`（遅延ロード用）。

```ts
@Component({
  selector: 'app-cmp',
  imports: [AccordionContent, AccordionGroup, AccordionPanel, AccordionTrigger],
  template: `...`,
  styles: [],
})
export class App {
  protected readonly title = signal('angular-app');
}
```

```html
<div ngAccordionGroup [multiExpandable]="false">
  <div class="accordion-item">
    <button ngAccordionTrigger panelId="panel-1" class="accordion-header">
      Section 1
      <span class="icon">▼</span>
    </button>
    <div ngAccordionPanel panelId="panel-1" class="accordion-panel">
      <ng-template ngAccordionContent>
        <p>Lazy loaded content here.</p>
      </ng-template>
    </div>
  </div>
</div>
```

**スタイリング戦略:**
トリガーの `[aria-expanded]` 属性をターゲットにしてアイコンを回転させ、パネルの表示をスタイリングします。

```css
.accordion-header[aria-expanded='true'] .icon {
  transform: rotate(180deg);
}

/* パネルディレクティブがDOM削除を処理しますが、トランジションをスタイリングできます */
.accordion-panel {
  padding: 1rem;
  border-top: 1px solid #ccc;
}
```

---

## 2. リストボックス

オプションのリストを表示するための基本的なディレクティブです。可視の選択リスト（ドロップダウンではない）に使用します。

**使用場面:** 可視の選択可能なリスト（単一または複数選択）。

**インポート:** `import {Listbox, Option} from '@angular/aria/listbox';`

**ディレクティブ:** `ngListbox`、`ngOption`。

```ts
@Component({
  selector: 'app-cmp',
  imports: [Listbox, Option],
  template: `...`,
  styles: [],
})
export class App {
  protected readonly title = signal('angular-app');
}
```

```html
<!-- 水平または垂直方向 -->
<ul ngListbox [(values)]="selectedItems" orientation="horizontal" [multi]="true">
  <li ngOption value="apple" class="option">Apple</li>
  <li ngOption value="banana" class="option">Banana</li>
</ul>
```

**スタイリング戦略:**
選択状態には `[aria-selected="true"]`、フォーカスされたアイテムには `:focus-visible` または `[data-active]` をターゲットにします（Angular Ariaはroving tabindexまたはactivedescendantを使用します）。

```css
.option {
  padding: 8px;
  cursor: pointer;
}
.option[aria-selected='true'] {
  background: #e0f7fa;
  font-weight: bold;
}
/* フォーカス状態はariaで管理されます */
.option:focus-visible {
  outline: 2px solid blue;
}
```

---

## 3. コンボボックス、セレクト、マルチセレクト

これらのパターンは、`ngCombobox` とポップアップ内の `ngListbox` を組み合わせます。

- **コンボボックス**: テキスト入力 + ポップアップ（オートコンプリートに使用）。
- **セレクト**: 読み取り専用コンボボックス + 単一選択リストボックス。
- **マルチセレクト**: 読み取り専用コンボボックス + 複数選択リストボックス。

**使用場面:** コンボボックスは、テキスト入力をポップアップと同期させる低レベルのプリミティブディレクティブで、オートコンプリート、セレクト、マルチセレクトパターンの基盤となるロジックとして機能します。カスタムフィルタリング、独自の選択要件、または標準の文書化されたコンポーネントから逸脱した特殊な入力-ポップアップ連携を構築する場合に特に使用してください。

**インポート:**

```
  import {Combobox, ComboboxInput, ComboboxPopupContainer} from '@angular/aria/combobox';
  import {Listbox, Option} from '@angular/aria/listbox';
```

**ディレクティブ:** `ngCombobox`、`ngComboboxInput`、`ngComboboxPopupContainer`、`ngListbox`、`ngOption`。

```html
<!-- 例：標準セレクト -->
<div ngCombobox [readonly]="true">
  <button ngComboboxInput class="select-trigger">
    {{ selectedValue() || 'Choose an option' }}
  </button>

  <ng-template ngComboboxPopupContainer>
    <ul ngListbox [(values)]="selectedValue" class="dropdown-menu">
      <li ngOption value="option1">Option 1</li>
      <li ngOption value="option2">Option 2</li>
    </ul>
  </ng-template>
</div>
```

**スタイリング戦略:**
ポップアップコンテナをコンテンツの上に浮かぶドロップダウンのように見せるスタイリングをします（CDK Overlayと組み合わせることが多い）。

```css
.select-trigger {
  width: 200px;
  padding: 8px;
  text-align: left;
}
.dropdown-menu {
  list-style: none;
  padding: 0;
  margin: 0;
  border: 1px solid #ccc;
  background: white;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
```

---

## 4. メニューとメニューバー

アクション、コマンド、コンテキストメニュー用（フォーム選択には使用しない）。

**使用場面:** メニューバーは、インターフェース全体に渡って持続するデスクトップスタイルのアプリケーションコマンドバー（例：ファイル、編集、表示）を構築するための高レベルのナビゲーションパターンです。完全な水平キーボードサポートを備えた論理的なトップレベルカテゴリへの複雑なコマンドの整理に最適ですが、シンプルなスタンドアロンアクションリストや水平スペースが制約されるモバイルファーストのレイアウトには避けてください。

**インポート:** `import {MenuBar, Menu, MenuContent, MenuItem} from '@angular/aria/menu';`

**ディレクティブ:** `ngMenuBar`、`ngMenu`、`ngMenuItem`、`ngMenuTrigger`。

```html
<!-- メニューバーの例 -->
<ul ngMenuBar class="menubar">
  <li ngMenuItem value="file">
    <button ngMenuTrigger [menu]="fileMenu">File</button>
  </li>
</ul>

<ul ngMenu #fileMenu="ngMenu" class="menu">
  <li ngMenuItem value="new">New</li>
  <li ngMenuItem value="open">Open</li>
</ul>
```

**スタイリング戦略:**
メニューバーにはflexboxを使用します。トリガーの状態に基づいてサブメニューを表示・非表示にします。

```css
.menubar {
  display: flex;
  gap: 10px;
  list-style: none;
  padding: 0;
}
.menu {
  background: white;
  border: 1px solid #ccc;
  padding: 5px 0;
}
.menu li {
  padding: 5px 15px;
  cursor: pointer;
}
```

---

## 5. タブ

1つのパネルのみが表示される層状のコンテンツセクション。

**使用場面:** タブコンポーネントは、関連するコンテンツを明確なナビゲート可能なセクションに整理し、ユーザーがページを離れることなくカテゴリやビューを切り替えられるようにします。設定パネル、複数トピックのドキュメント、ダッシュボードに理想的ですが、順次ワークフロー（ステッパー）や7〜8セクションを超えるナビゲーションには避けてください。

**インポート:** `import {Tab, Tabs, TabList, TabPanel, TabContent} from '@angular/aria/tabs';`

**ディレクティブ:** `ngTabs`、`ngTabList`、`ngTab`、`ngTabPanel`、`ngTabContent`。

```html
<div ngTabs>
  <ul ngTabList class="tab-list">
    <li ngTab value="profile" class="tab-btn">Profile</li>
    <li ngTab value="security" class="tab-btn">Security</li>
  </ul>

  <div ngTabPanel value="profile" class="tab-panel">
    <ng-template ngTabContent>Profile Settings</ng-template>
  </div>
  <div ngTabPanel value="security" class="tab-panel">
    <ng-template ngTabContent>Security Settings</ng-template>
  </div>
</div>
```

**スタイリング戦略:**
タブボタンの `[aria-selected="true"]` をターゲットにします。

```css
.tab-list {
  display: flex;
  border-bottom: 2px solid #ccc;
  list-style: none;
  padding: 0;
}
.tab-btn {
  padding: 10px 20px;
  cursor: pointer;
  border-bottom: 2px solid transparent;
}
.tab-btn[aria-selected='true'] {
  border-bottom-color: blue;
  font-weight: bold;
}
.tab-panel {
  padding: 20px;
}
```

---

## 6. ツールバー

関連するコントロールをグループ化します（テキストフォーマットなど）。

**使用場面:** ツールバーは、頻繁にアクセスされる関連コントロールを1つの論理コンテナにグループ化するための組織コンポーネントです。テキストフォーマットやメディアコントロールなど、繰り返しアクションが必要なワークフローのキーボード効率（矢印キーナビゲーション）と視覚的構造を強化するために最適です。

**インポート:** `import {Toolbar, ToolbarWidget, ToolbarWidgetGroup} from '@angular/aria/toolbar';`

**ディレクティブ:** `ngToolbar`、`ngToolbarWidget`、`ngToolbarWidgetGroup`。

```html
<div ngToolbar class="toolbar">
  <div ngToolbarWidgetGroup [multi]="true" role="group" aria-label="Formatting">
    <button ngToolbarWidget value="bold" class="tool-btn">B</button>
    <button ngToolbarWidget value="italic" class="tool-btn">I</button>
  </div>
</div>
```

**スタイリング戦略:**
ツールバー内の `[aria-pressed="true"]`（トグルボタン用）または `[aria-checked="true"]`（ラジオグループ用）をターゲットにします。

```css
.toolbar {
  display: flex;
  gap: 5px;
  padding: 8px;
  background: #f5f5f5;
}
.tool-btn {
  padding: 5px 10px;
  border: 1px solid #ccc;
}
.tool-btn[aria-pressed='true'],
.tool-btn[aria-checked='true'] {
  background: #ddd;
}
```

---

## 7. ツリー

階層データを表示します（ファイルシステム、ネストされたナビゲーションなど）。

**使用場面:** ツリーコンポーネントは、ファイルシステム、組織図、複雑なサイトアーキテクチャなど、深くネストされた階層データ構造のナビゲーションと表示のために設計されています。ユーザーがブランチを展開・折りたたむ必要がある複数レベルの関係に特に使用しますが、フラットリスト、データテーブル、またはシンプルな選択メニューには避けてください。

**インポート:** `import {Tree, TreeItem, TreeItemGroup} from '@angular/aria/tree';`

**ディレクティブ:** `ngTree`、`ngTreeItem`、`ngTreeGroup`。

```html
<ul ngTree class="tree">
  <li ngTreeItem value="documents">
    <span class="tree-label">Documents</span>
    <ul ngTreeGroup class="tree-group">
      <li ngTreeItem value="resume">Resume.pdf</li>
    </ul>
  </li>
</ul>
```

**スタイリング戦略:**
`[aria-expanded]` をターゲットにして子を表示・非表示にするか、シェブロンアイコンを回転させます。ネストされたグループに `padding-left` を使用して階層を表示します。

```css
.tree,
.tree-group {
  list-style: none;
  padding-left: 20px;
}
.tree-label::before {
  content: '> ';
  display: inline-block;
  transition: transform 0.2s;
}
li[aria-expanded='true'] > .tree-label::before {
  transform: rotate(90deg);
}
```

## 8. グリッド

矢印キーでナビゲーションを可能にする、セルの双方向インタラクティブコレクションです。

**使用場面:** データテーブル、カレンダー、スプレッドシート、インタラクティブ要素のレイアウトパターン。
**ディレクティブ:** `ngGrid`、`ngGridRow`、`ngGridCell`、`ngGridCellWidget`。

```html
<table ngGrid [multi]="true" [enableSelection]="true" class="grid-table">
  <tr ngGridRow>
    <th ngGridCell role="columnheader">Name</th>
    <th ngGridCell role="columnheader">Status</th>
  </tr>
  <tr ngGridRow>
    <td ngGridCell>Project A</td>
    <td ngGridCell [(selected)]="isSelected">
      <button ngGridCellWidget (activated)="onActivate()">Active</button>
    </td>
  </tr>
</table>
```

**スタイリング戦略:**
選択されたセルには `[aria-selected="true"]`、アクティブなセルには `:focus-visible`（roving tabindex）または コンテナの `[aria-activedescendant]` をターゲットにします。

```css
.grid-table {
  border-collapse: collapse;
}
[ngGridCell] {
  padding: 8px;
  border: 1px solid #ddd;
}
[ngGridCell][aria-selected='true'] {
  background: #e3f2fd;
}
/* フォーカス状態はroving tabindexで管理されます */
[ngGridCell]:focus-visible {
  outline: 2px solid #2196f3;
  outline-offset: -2px;
}
```

## エージェントへの一般的なルール

1. **これらの特定のAriaパターンを実装する際は、`<select>` などのネイティブHTML要素を使用しないでください**。`ng*` ディレクティブを使用してください。
2. **CSSは手動で処理してください**: `Angular Aria` はスタイルを提供しません。ディレクティブが自動的に切り替えるネイティブARIA属性（`aria-expanded`、`aria-selected` など）をターゲットにしたCSSを記述する必要があります。
3. **遅延ロード**: 重いコンテンツパネルの遅延レンダリングを確保するために、`ng-template` 内で提供されている構造ディレクティブ（`ngAccordionContent`、`ngTabContent`）を常に使用してください。
