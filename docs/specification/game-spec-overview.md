# Abyss Branch - Game Spec Draft r1

## 目的
- Abyss Branch のゲーム仕様の初期方針を定義する。
- MVP として `game v0.1.0`（1 Area で Battle 可能）までの到達条件を明確化する。

## 対象範囲
- スマホ縦画面向けのシングルプレイカード戦闘。
- Battle ループ、Status、カード循環、Module 運用、勝敗条件の基礎仕様。
- アート制作手段や最終世界観設定の詳細は対象外とする。

## 本文
- Document Version: `Draft r1`
- Target Game Version: `v0.1.0`
- Platform: スマホ（縦画面）
- Play Style: Offline Mode、シングルプレイ
- Genre: カードバトル + ローグ系
- Reference: Night of the Full Moon / Slay the Spire

### Related Specs
- Battle の詳細手順は `docs/specification/battle-flow.md` を参照する。
- `battle-flow` では `Initial` と `Turn -> Phase -> Step`、`Status` 処理順を定義する。
- Card データは `docs/specification/card-schema.md`、効果データは `docs/specification/effect-schema.md` を参照する。
- `effect-schema` では `Timing Execution Order` と `Target Scope Resolution Order` を定義する。
- 状態遷移モデルは `docs/specification/state-model.md` を参照する。
- `Event` payload 契約は `docs/specification/state-events.md` を参照する。

### Core Loop
- Player は Battle を繰り返して進行する。
- MVP では 1 Area 分の Battle 進行を実装対象とする。
- `Battle Tier` は `Normal` / `Elite` / `Boss` を使用する。
- `Normal` の目標時間は 1-2 分、`Boss` はそれより長くなる想定とする。
- 長期目標として、1 Run は約 15 分を想定する。

### Card Architecture
- Card は `Core Card` / `Module Card` / `Instant Card` で構成する。
- `Core Card` は Character に紐づく中核カードとする。
- `Module Slot` は 3 枠固定とする（MVP 時点）。
- `Module Card` の設置と交換は手動で行う。
- `Module Card` の効果発動は自動で行う。
- `Instant Card` は Slot を使わず即時発動する。
- Card 寿命タグとして `Persistent` / `Ephemeral` を扱う。
- `Ephemeral` は 1 回発動後に `Exhaust Pile` へ送る。

### Pile Rules
- `Deck` は戦闘前に保持するカード集合とする。
- Battle 開始時に `Draw Pile` を構成する。
- `Hand` は戦闘中に保持する手札とする。
- `Draw Pile` が 0 枚になった場合、`Discard Pile` を `Shuffle` して `Draw Pile` へ戻す。
- Module 交換で外した `Module Card` は `Discard Pile` へ送る。

### Targeting Rules
- 攻撃対象に `Enemy` / `Module Card` / `Core Card` を含める。
- 毎回の手動ターゲット選択は採用しない。
- 先行側の `Turn 1` は攻撃を発動しない。
- ターゲット選択は自動優先とし、通常攻撃の `Target Priority` は `Guard Module -> Core Card` とする。
- 相手側に `Guard Module` が 1 つでも生存中の場合、通常攻撃は `Core Card` を対象にできない。
- `Siege` タグ付き攻撃のみ、`Guard Module` を無視して `Core Card` を対象にできる。
- 複数候補がある場合は、攻撃側の `Target Policy` を先に適用し、同率なら `Module Slot` の番号昇順で決定する。

### Damage Resolution Rules
- ダメージ計算は `RawDamage = ATK + DMG Bonus` を用いる。
- 最終ダメージは `FinalDamage = max(1, RawDamage)` とする。
- 受ダメージ時は `shield` を先に減算し、余剰分だけ `hp` を減算する。
- `target.hp == 0` で即時 `Destroyed` とする。
- 攻撃解決は同時処理せず、攻撃側 `Module Slot` 番号昇順で逐次処理する。
- 反撃/貫通/分散ダメージは MVP では `TODO` とする。

### Progression Policy
- `In-Run Progression` は採用する。
- `Meta Progression` は MVP 範囲外とし、`TODO` として保留する。

### Save and Resume Policy
- 中断保存仕様は MVP 範囲外とし、`TODO` として保留する。

### World and Presentation
- 世界観トーンは無機質寄りを基調とする。
- 世界観の詳細設定は `TODO` とする。
- アセット制作は生成AI活用を想定するが、最終ルック（例: ドットライク）は `TODO` とする。

## 決定事項
- MVP の到達点は「1 Area で Battle 可能（game v0.1.0）」とする。
- 勝利条件は「1 Area 突破」とする。
- 敗北条件は `Player HP <= 0` または `Core Card HP <= 0` とする。
- オンライン要素は採用しない。
