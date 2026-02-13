# 変更履歴

自分向けのリリース履歴。プレイ体験や仕様に影響する変更を中心に記録する。
内部リファクタや命名整理など、体感に影響しない変更は省略する。

## 運用ルール
- リリース単位で記録する。
- 日付は `YYYY-MM-DD` 形式で記載する。
- 区分は必要なものだけ使う（Added / Changed / Fixed / Deprecated / Removed / Security）。

## 未リリース
### Changed
- `docs/specification/state-events.md` を新規作成し、`Event Envelope`、最小 `event_type` 一覧、payload 契約、`RNG_USED` を含む再現性ログ規約を定義。
- `docs/specification/state-model.md` を新規作成し、`state + reducer + event log` による Battle 状態遷移モデル（`GameState`/`Event`/`Reducer`/`Event Queue`/`Determinism`）を定義。
- `docs/specification/card-schema.md` で `target_policy` 許可値（`front` / `lowest_hp` / `random` / `core_only`）を確定し、`docs/specification/effect-schema.md` に `effect_type` 実行マッピング（Resolver 対応）を追加。
- `docs/specification/effect-schema.md` で `Timing Execution Order` と `Target Scope Resolution Order` を追加し、`timing` と `target_scope` の実行順を固定。
- `docs/specification/effect-schema.md` を新規作成し、`effect_id` の最小スキーマ（`effect_type` / `timing` / `target_scope` / `stack_rule` / `params`）と `shield` 効果表現を定義。
- `docs/specification/card-schema.md` / `docs/specification/battle-flow.md` / `docs/specification/game-spec-overview.md` / `docs/policy/glossary.md` で `def` を廃止し、`shield` 先行吸収方式へ変更。`Core Card` に `atk` と `shield` を定義。
- `docs/specification/card-schema.md` を新規作成し、`Core Card` / `Module Card` / `Instant Card` の最小スキーマ、型別必須項目、`effect_ids` 参照方式を定義。
- `docs/specification/battle-flow.md` / `docs/specification/game-spec-overview.md` に先行側 `Turn 1` の攻撃スキップ（設置・交換・非攻撃効果は実行可）ルールを反映。
- `docs/specification/battle-flow.md` / `docs/specification/game-spec-overview.md` にダメージ解決の最小仕様（`RawDamage = ATK + DMG Bonus`、`FinalDamage = max(1, RawDamage)`、`shield` 先行吸収、`Destroyed` 即時判定、`Module Slot` 番号昇順の逐次解決）を反映。
- `docs/specification/battle-flow.md` / `docs/specification/game-spec-overview.md` に `Target Priority` の確定ルール（通常攻撃は `Guard Module -> Core Card`、同率時は `Target Policy` 後に `Module Slot` 番号昇順）を反映。
- `docs/specification/battle-flow.md` / `docs/specification/game-spec-overview.md` / `docs/policy/glossary.md` に `Core Card` 直接攻撃条件（`Guard Module` による遮断と `Siege` 例外）および敗北条件（`Core Card HP <= 0`）を反映。
- `docs/specification/game-spec-overview.md` を `Abyss Branch - Game Spec Draft r1` として維持しつつ、用語統一（Battle / Battle Tier / Card種別 / Pile定義）を反映。
- `docs/policy/glossary.md` に主要用語を追加し、`Deck` と `Draw Pile` を分離して定義。
- 目標バージョンを `game v0.1.0`（1エリアで戦闘可能）に設定。
- デッキ再循環、モジュール交換時の捨て札移動、モジュール攻撃対象化を仕様へ反映。
- `docs/specification/battle-flow.md` を新規作成し、`Phase -> Step` と `Interrupt Check`（Win/Lose 割り込み判定）を追加。
- `docs/specification/game-spec-overview.md` に `battle-flow.md` 参照を追加。
- `docs/specification/battle-flow.md` を `Turn -> Phase -> Step` 構造へ再編し、`Status` 処理フェーズを追加。
- `docs/policy/glossary.md` に `Status` / `Status Stack` / `Status Tick` / `Status Duration` を追加。
- `docs/specification/battle-flow.md` で `Battle Start Phase` を Turn 外の `Initial` として分離し、手順をフラットな `Turn / Phase / Step` 形式へ修正。
- `docs/specification/battle-flow.md` の `Check Phase` を廃止し、`Interrupt Check` を各 Step 後の共通ルールとして統一。

## リリース 0.0.1 - 2026-02-11
### 追加
- ドキュメント初版を作成。
