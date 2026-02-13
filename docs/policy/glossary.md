# 用語集

## 目的
プロジェクトで使用する主要用語の正本を定義し、文書間の表記ゆれを防ぐ。

## ルール
- 用語は英語を正とする。
- `Definition` は日本語で記述してよい。
- 新しい主要用語を使う場合は先に本書へ追加する。

## 用語
| Term | Definition | Related Code/Data |
|---|---|---|
| Player | プレイヤー本人。ゲームを操作する主体。 | player |
| Character | プレイヤーが選択する操作キャラクター。 | character |
| Card | カードの上位概念。`Core Card` / `Module Card` / `Instant Card` を含む。 | card |
| Core Card | Character に紐づく中核カード。将来の複数保持拡張を許容する。 | core_card |
| Module Card | `Module Slot` に配置して効果を自動発動するカード。 | module_card |
| Instant Card | `Module Slot` を使わず即時発動するカード。 | instant_card |
| Module Slot | `Module Card` を配置する枠。MVP は 3 枠固定。 | module_slot |
| Persistent | 発動後も戦闘中に残るカード寿命タグ。 | card_lifetime |
| Ephemeral | 1 回発動後に `Exhaust Pile` へ送るカード寿命タグ。 | card_lifetime |
| Deck | 戦闘前に保持するカード集合。 | deck |
| Draw Pile | 戦闘中にカードを引く山。 | draw_pile |
| Hand | 戦闘中に保持する手札。 | hand |
| Discard Pile | 使用後や交換後に送られる捨て札。 | discard_pile |
| Exhaust Pile | 再利用しないカードを退避する除外領域。 | exhaust_pile |
| Shuffle | `Draw Pile` が空のときに `Discard Pile` を混ぜて戻す処理。 | shuffle |
| Enemy | 戦闘で対峙する敵の総称。 | enemy |
| Enemy Unit | 戦場上の個別敵ユニット。 | enemy_unit |
| Status | Battle 中に継続適用される効果の総称（バフ/デバフ/DoT など）。 | status |
| Status Stack | 同一 Status の重なり数。 | status_stack |
| Status Tick | 定義タイミングで発生する Status 効果処理。 | status_tick |
| Status Duration | Status の残り有効ターン数または回数。 | status_duration |
| Battle | 戦闘の統一語。 | battle |
| Battle Tier | 戦闘難度区分。`Normal` / `Elite` / `Boss` を使用する。 | battle_tier |
| Effect | `effect_id` で参照される効果データ。 | effect |
| Effect ID | 効果を一意に識別するID。 | effect_id |
| Effect Timing | 効果が発動するタイミング定義。 | effect_timing |
| Target Scope | 効果の対象範囲定義。 | target_scope |
| Stack Rule | 同一効果重複時の解決ルール。 | stack_rule |
| Game State | Battle 進行の単一正本となる状態。 | game_state |
| Event | 状態更新を発生させる入力単位。 | event |
| Event Payload | `Event` ごとの入力データ本体。 | event_payload |
| Reducer | `state` と `event` から新しい `state` を返す純関数。 | reducer |
| Event Queue | 未解決 `Event` を保持するキュー。 | event_queue |
| Event Log | 解決済み `Event` の追記ログ。 | event_log |
| RNG Seed | 乱数再現性を担保するシード値。 | rng_seed |
| Determinism | 同一入力から同一結果を返す再現性特性。 | determinism |
| Shield | 受ダメージ時に HP より先に消費される防護値。 | shield |
| Target | カード効果の対象。`Enemy` / `Module Card` / `Core Card` を含む。 | target |
| Target Priority | 自動ターゲット時の優先順位ルール。 | target_priority |
| Guard Module | `Core Card` への通常攻撃を遮断する `Module Card`。 | guard_module |
| Siege | `Guard Module` を無視して `Core Card` を対象化できる攻撃タグ。 | siege |
| Run | 1 回の挑戦セッション。 | run |
| Area | Run 内の進行区画。MVP は 1 Area。 | area |
| In-Run Progression | Run 内でのみ有効な成長。 | in_run_progression |
| Meta Progression | Run をまたいで維持される恒久成長。現時点は未採用。 | meta_progression |
| Offline Mode | ネットワーク非依存で完結する動作方針。 | offline_mode |
| MVP | 最小実装到達点。現在は `game v0.1.0` を指す。 | mvp |
