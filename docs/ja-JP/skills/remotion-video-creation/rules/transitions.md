---
name: transitions
description: Remotionのフルスクリーンシーントランジション。
metadata:
  tags: transitions, fade, slide, wipe, scenes
---

## フルスクリーントランジション

`<TransitionSeries>` を使って複数のシーンやクリップ間をアニメーションで切り替えます。
これはchildren要素を絶対位置に配置します。

## 前提条件

まず、@remotion/transitionsパッケージをインストールする必要があります。
インストールされていない場合は、以下のコマンドを使用します：

```bash
npx remotion add @remotion/transitions # npmを使うプロジェクト
bunx remotion add @remotion/transitions # bunを使うプロジェクト
yarn remotion add @remotion/transitions # yarnを使うプロジェクト
pnpm exec remotion add @remotion/transitions # pnpmを使うプロジェクト
```

## 使用例

```tsx
import {TransitionSeries, linearTiming} from '@remotion/transitions';
import {fade} from '@remotion/transitions/fade';

<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={60}>
    <SceneA />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition presentation={fade()} timing={linearTiming({durationInFrames: 15})} />
  <TransitionSeries.Sequence durationInFrames={60}>
    <SceneB />
  </TransitionSeries.Sequence>
</TransitionSeries>;
```

## 利用可能なトランジションタイプ

それぞれのモジュールからトランジションをインポートします：

```tsx
import {fade} from '@remotion/transitions/fade';
import {slide} from '@remotion/transitions/slide';
import {wipe} from '@remotion/transitions/wipe';
import {flip} from '@remotion/transitions/flip';
import {clockWipe} from '@remotion/transitions/clock-wipe';
```

## 方向指定のスライドトランジション

入退場アニメーションのスライド方向を指定します。

```tsx
import {slide} from '@remotion/transitions/slide';

<TransitionSeries.Transition presentation={slide({direction: 'from-left'})} timing={linearTiming({durationInFrames: 20})} />;
```

方向: `"from-left"`、`"from-right"`、`"from-top"`、`"from-bottom"`

## タイミングオプション

```tsx
import {linearTiming, springTiming} from '@remotion/transitions';

// 線形タイミング - 一定速度
linearTiming({durationInFrames: 20});

// スプリングタイミング - 有機的な動き
springTiming({config: {damping: 200}, durationInFrames: 25});
```

## 長さの計算

トランジションは隣接するシーンを重ね合わせるため、コンポジション全体の長さはすべてのシーク長さの合計よりも**短く**なります。

例えば、2つの60フレームシーンと15フレームのトランジションの場合：

- トランジションなし：`60 + 60 = 120` フレーム
- トランジションあり：`60 + 60 - 15 = 105` フレーム

トランジション中は両シーンが同時に再生されるため、トランジションの長さが差し引かれます。

### トランジションの長さを取得する

タイミングオブジェクトの `getDurationInFrames()` メソッドを使用します：

```tsx
import {linearTiming, springTiming} from '@remotion/transitions';

const linearDuration = linearTiming({durationInFrames: 20}).getDurationInFrames({fps: 30});
// 20を返す

const springDuration = springTiming({config: {damping: 200}}).getDurationInFrames({fps: 30});
// スプリング物理に基づいて計算された長さを返す
```

明示的な `durationInFrames` なしの `springTiming` の場合、スプリングアニメーションが落ち着くタイミングを `fps` で計算するため、長さは `fps` に依存します。

### コンポジション全体の長さを計算する

```tsx
import {linearTiming} from '@remotion/transitions';

const scene1Duration = 60;
const scene2Duration = 60;
const scene3Duration = 60;

const timing1 = linearTiming({durationInFrames: 15});
const timing2 = linearTiming({durationInFrames: 20});

const transition1Duration = timing1.getDurationInFrames({fps: 30});
const transition2Duration = timing2.getDurationInFrames({fps: 30});

const totalDuration = scene1Duration + scene2Duration + scene3Duration - transition1Duration - transition2Duration;
// 60 + 60 + 60 - 15 - 20 = 145フレーム
```
