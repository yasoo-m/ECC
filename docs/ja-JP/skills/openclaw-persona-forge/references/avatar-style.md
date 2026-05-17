# ステップ5：アバタースタイル & 画像生成

すべてのロブスターアバターは**統一されたビジュアルスタイル**を使用し、ロブスターファミリーのスタイル一貫性を確保する必要があります。
アバターは3つの情報を伝える必要があります：**種としての形態 + 性格のヒント + 特徴的な道具**

## スタイルリファレンス

アダム（Adam）— ロブスター族の創世神、このスキルの最初の作品。

新しく生成されるロブスターアバターはすべてこのスタイルと一致する必要があります：レトロフューチャリスト、アーケードUIの縁取り、強いシルエット、64x64で識別可能。

## 統一スタイルベース（STYLE_BASE）

**生成のたびにこのベースを含める必要があります**。修正や省略は不可：

```
STYLE_BASE = """
Retro-futuristic 3D rendered illustration, in the style of 1950s-60s Space Age
pin-up poster art reimagined as glossy inflatable 3D, framed within a vintage
arcade game UI overlay.

Material: high-gloss PVC/latex-like finish, soft specular highlights, puffy
inflatable quality reminiscent of vintage pool toys meets sci-fi concept art.
Smooth subsurface scattering on shell surface.

Arcade UI frame: pixel-art arcade cabinet border elements, a top banner with
character name in chunky 8-bit bitmap font with scan-line glow effect, a pixel
energy bar in the upper corner, small coin-credit text "INSERT SOUL TO CONTINUE"
at bottom in phosphor green monospace type, subtle CRT screen curvature and
scan-line overlay across entire image. Decorative corner bezels styled as chrome
arcade cabinet trim with atomic-age starburst rivets.

Pose: references classic Gil Elvgren pin-up compositions, confident and
charismatic with a slight theatrical tilt.

Color system: vintage NASA poster palette as base — deep navy, teal, dusty coral,
cream — viewed through arcade CRT monitor with slight RGB fringing at edges.
Overall aesthetic combines Googie architecture curves, Raygun Gothic design
language, mid-century advertising illustration, modern 3D inflatable character
rendering, and 80s-90s arcade game UI. Chrome and pastel accent details on
joints and antenna tips.

Format: square, optimized for avatar use. Strong silhouette readable at 64x64
pixels.
"""
```

## 個性化変数

統一ベースの上に、魂に基づいて以下の変数を記入します：

| 変数 | 説明 | 例 |
|------|------|------|
| `CHARACTER_NAME` | アーケードバナーに表示される名前 | "ADAM"、"DEWEY"、"RIFF" |
| `SHELL_COLOR` | ロブスターの殻の主色調（統一カラーパレット内で変化） | "deep crimson"、"dusty teal"、"warm amber" |
| `SIGNATURE_PROP` | 特徴的な道具 | "cracked sunglasses"、"reading glasses on a chain" |
| `EXPRESSION` | 表情/姿勢 | "stoic but kind-eyed"、"nervously focused" |
| `UNIQUE_DETAIL` | 独自の細部（模様/装飾/傷など） | "constellation patterns etched on claws"、"bandaged left claw" |
| `BACKGROUND_ACCENT` | 背景の個性化要素（統一された宇宙背景に重ねる） | "musical notes floating as nebula dust"、"ancient book pages drifting" |
| `ENERGY_BAR_LABEL` | アーケードUIのエネルギーバーのラベル（個性化のイースターエッグ） | "CREATION POWER"、"CALM LEVEL"、"ROCK METER" |

## プロンプトの組み立て

```
最終プロンプト = STYLE_BASE + 個性化説明段落
```

個性化説明段落テンプレート：

```
The character is a cartoon lobster with a [SHELL_COLOR] shell,
[EXPRESSION], wearing/holding [SIGNATURE_PROP].
[UNIQUE_DETAIL]. Background accent: [BACKGROUND_ACCENT].
The arcade top banner reads "[CHARACTER_NAME]" and the energy bar
is labeled "[ENERGY_BAR_LABEL]".
The key silhouette recognition points at small size are:
[SIGNATURE_PROP] and [one other distinctive feature].
```

## 画像生成フロー

プロンプトの組み立てが完了したら：

### ルートA：インストール済みで審査済みの画像生成スキル

1. ロブスターの名前を安全なファイル名セグメントに変換：英数字とハイフンのみ残し、他の文字は `-` に置換
2. Writeツールで書き込む：`/tmp/openclaw-<safe-name>-prompt.md`
3. 現在の環境で利用可能な画像生成スキルを呼び出して画像を生成
4. Readツールで生成された画像をユーザーに表示
5. ユーザーに満足しているか確認し、不満足な場合は変数を調整して再生成

### ルートB：利用可能な画像生成スキルがない

完全なプロンプトテキストと手動使用の説明を出力します：

```markdown
**アバタープロンプト**（以下のプラットフォームにコピーして手動生成できます）：
- Google Gemini：直接貼り付け
- ChatGPT（DALL-E）：直接貼り付け
- Midjourney：貼り付け後 `--ar 1:1 --style raw` を追加

> [完全な英語プロンプト]

現在の環境で後から審査済みの画像生成スキルが提供された場合、自動生成フローに戻ることができます。
```

## ユーザーへの表示フォーマット

```markdown
## アバター

**個性化変数**：
- 殻の色：[SHELL_COLOR]
- 道具：[SIGNATURE_PROP]
- 表情：[EXPRESSION]
- 独自の細部：[UNIQUE_DETAIL]
- 背景アクセント：[BACKGROUND_ACCENT]
- エネルギーバーのラベル：[ENERGY_BAR_LABEL]

**生成結果**：
[画像（ルートA）またはプロンプトテキスト（ルートB）]

> 満足していますか？不満足な場合は [具体的な調整項目] を調整して再生成できます。
```
