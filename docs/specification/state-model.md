# State Model

## 目的
- Battle 進行を `state + reducer + event log` で一貫管理する基準を定義する。
- 実装時に再現性（determinism）とテスト容易性を確保する。

## 対象範囲
- `game v0.1.0` の Battle で使用する状態遷移モデル。
- `GameState` の最小構造、`Event` の分類、`Reducer` 更新規則。
- `Event` の payload 定義は `docs/specification/state-events.md` を参照する。
- 画面演出や UI レイヤー固有の状態は対象外。

## 本文
### Core Principle
- Battle の正本は常に `GameState` とする。
- すべての進行は `Event` を介してのみ状態更新する。
- 状態更新は `reduce(state, event) -> new_state` の純関数で行う。
- `Event Log` は追記専用とし、デバッグと再現実行に利用する。

### GameState Minimum Shape
| Field | Type | Description |
|---|---|---|
| `battle_id` | string | Battle 識別子。 |
| `turn` | number | 現在ターン番号。 |
| `active_side` | enum | `player` / `enemy`。 |
| `phase` | enum | `resource` / `action` / `end`。 |
| `player_core` | object | `hp`, `max_hp`, `shield`, `atk` を持つ。 |
| `enemy_core` | object | `hp`, `max_hp`, `shield`, `atk` を持つ。 |
| `player_modules` | object[] | `Module Slot` 順の配列。 |
| `enemy_modules` | object[] | `Module Slot` 順の配列。 |
| `hands` | object | `player` / `enemy` の手札。 |
| `piles` | object | `draw`, `discard`, `exhaust` を保持。 |
| `status_map` | object | Status の現在値。 |
| `event_queue` | object[] | 未解決 `Event` キュー。 |
| `event_log` | object[] | 解決済み `Event` ログ。 |
| `rng_seed` | number | 乱数再現用シード値。 |

### Event Categories
| Category | Example Events | Description |
|---|---|---|
| Turn/Phase | `START_TURN`, `CHANGE_PHASE`, `END_TURN` | 進行制御。 |
| Card Action | `PLAY_CARD`, `INSTALL_MODULE`, `SWAP_MODULE` | 行動入力。 |
| Effect | `RESOLVE_EFFECT`, `APPLY_STATUS`, `ADD_SHIELD` | 効果解決。 |
| Combat | `SELECT_TARGET`, `APPLY_DAMAGE`, `DESTROY_CARD` | 戦闘解決。 |
| Result | `CHECK_INTERRUPT`, `BATTLE_WIN`, `BATTLE_LOSE` | 勝敗判定。 |

### Reducer Rules
- `Reducer` は副作用を持たない。
- `Reducer` は 1 回の `Event` につき 1 つの `new_state` を返す。
- `Reducer` は解決済み `Event` を `event_log` へ追記する。
- 未解決の後続処理は `event_queue` へ追加する。
- `CHECK_INTERRUPT` は各 `Step` 解決直後に必ず enqueue する。

### Queue and Timing Rules
- `event_queue` は FIFO を基本とする。
- 同一タイミング内の `auto_target` 同率は `Target Policy -> Module Slot` 番号昇順で決定する。
- `timing` 実行順は `docs/specification/effect-schema.md` の `Timing Execution Order` を正本とする。
- `passive` は `Event` 発行ではなく、該当 `Event` 解決時の修飾として適用する。

### Determinism Rules
- 乱数は必ず `rng_seed` から生成し、外部乱数を直接使用しない。
- 乱数使用時は `RNG_USED`（`seed_before`, `seed_after`, `roll`）を `event_log` に残す。
- 同一 `initial_state` と同一 `event` 列では、必ず同一結果を返す。

### Minimal Pseudo Flow
```text
dispatch(event):
  queue.push(event)
  while queue not empty:
    current = queue.pop_front()
    state = reduce(state, current)
    log.append(current)
```

## 決定事項
- Battle 進行は `state + reducer + event log` で管理する。
- `GameState` を単一正本とし、直接ミューテーションを禁止する。
- `timing` は `effect-schema` の順序に従い `event_queue` で解決する。
- 再現性担保のため `rng_seed` を `GameState` の必須項目とする。
