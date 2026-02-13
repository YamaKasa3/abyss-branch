# Battle Flow

## 目的
- 1 回の `Battle` の進行を `Initial` と `Turn -> Phase -> Step` で定義する。
- `Interrupt Check` による `Win/Lose` の割り込み判定タイミングを固定する。

## 前提条件
- 用語は `docs/policy/glossary.md` を正本とする。
- 本書は `Abyss Branch - Game Spec Draft r1`（`game v0.1.0` 対象）に従う。
- MVP は `1 Area` での Battle 進行を対象とする。

## Trun
一連の `Phase` から構成され、Player と Enemy が交互に繰り返す。
- `Initial` は Battle 開始時の状態初期化のみを扱う。
- Character 選択や Relic 採用は Battle 外の `Pre-Battle Setup` として後続定義する（現時点では対象外）。

## Phase
- `Resource`: Draw、Mana Recovery、自動発動効果の処理を行う。
- `Action`: Card 使用、Module 設置・交換、効果解決を行う。
- `End`: ターン終了時効果の処理と次ターンに向けた Cleanup を行う。
- `Effect Timing` の実行順は `docs/specification/effect-schema.md` の `Timing Execution Order` を正とする。

## Attack Target Rules
- 攻撃対象は `Enemy` / `Module Card` / `Core Card` を扱う。
- 毎回の手動ターゲット選択は採用せず、自動ターゲットで解決する。
- 先行側の `Turn 1` は攻撃を発動しない（攻撃処理をスキップする）。
- 先行側の `Turn 1` でも、`Module Card` の設置・交換や非攻撃効果は通常どおり実行できる。
- 通常攻撃の `Target Priority` は次の順とする。
  1. 相手側の `Guard Module`
  2. 相手側の `Core Card`
- 相手側に `Guard Module` が 1 つでも生存中の場合、通常攻撃は `Core Card` を対象にできない。
- `Siege` タグ付き攻撃のみ、`Guard Module` を無視して `Core Card` を対象にできる。
- 複数の `Guard Module` が候補になる場合は、攻撃側の `Target Policy` を先に適用する。
- `Target Policy` 適用後も同率の場合は、`Module Slot` の番号昇順で対象を決定する。

## Damage Resolution Rules
- 攻撃 1 回あたりのダメージ計算は `RawDamage = ATK + DMG Bonus` とする。
- 最終ダメージは `FinalDamage = max(1, RawDamage)` とする。
- 受ダメージ時は `shield` を先に減算し、余剰分だけ `hp` を減算する。
- `shield` 更新は `target.shield = max(0, target.shield - FinalDamage)` とする。
- `hp` 更新は `target.hp = max(0, target.hp - overflow_damage)` とする。
- `target.hp == 0` になった対象は即時に `Destroyed` とする。
- 攻撃解決は同時処理せず、攻撃側の `Module Slot` 番号昇順で逐次処理する。
- 破壊された `Module Card` は、その時点以降の未解決行動を失う。
- `Core Card` が `Destroyed` になった時点で `Lose Check` を即時実行する。
- 反撃/貫通/分散ダメージなどの拡張要素は `TODO` とする。

## Step
- `Step` は各 `Phase` 内の最小処理単位とする。
- 各 `Step` の解決時に `Status` を修飾子として参照し、必要な効果を適用する。
- すべての `Step` の直後に `Interrupt Check` を実行し、`Win/Lose` を即時判定する。

## Status Rules
- `Status` は特定 `Phase` ではなく、`Step` 解決に横断的に作用するルールとして扱う。
- `Status` の適用順（Player/Enemy の優先）は `TODO` とする。
- `Status Tick` と `Status Duration` の更新タイミングは `TODO` とする。

## Interrupt Rules
- `Win Check`: Battle 内 `Enemy` が 0 になった時点で即時勝利とする。
- `Lose Check`: `Player HP` または `Core Card HP` が 0 以下になった時点で即時敗北とする。
- `Priority Rule`: `Win/Lose` 同時成立時の優先順は `TODO` とする。

## 検証方法
- `Turn` が `Resource -> Action -> End` の順で実行されることを確認する。
- すべての Step 後に `Interrupt Check` が実行されることを確認する。
- `Status Tick` 実行後に `Status Duration` が更新されることを確認する。
- `Draw Pile` が 0 のときに `Discard Pile` から再構成されることを確認する。
- `Ephemeral` が発動後に `Exhaust Pile` へ移動することを確認する。
- ダメージ計算が `max(1, ATK + DMG Bonus)` で解決されることを確認する。
- ダメージが `shield` に先に適用され、余剰分のみ `hp` に適用されることを確認する。
- `target.hp == 0` 到達時に即時 `Destroyed` となることを確認する。
