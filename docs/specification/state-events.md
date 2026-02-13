# State Events

## 目的
- `Reducer` 実装で使用する `Event` の入力契約（payload）を固定する。
- `state-model` と `battle-flow` の接続点を明確化する。

## 対象範囲
- `game v0.1.0` Battle における `Event` 種別と payload 定義。
- `dispatch -> queue -> reduce` の進行で使用する最小イベントセット。
- UI 表示専用イベントや解析専用メタデータは対象外。

## 本文
### Event Envelope
| Field | Type | Required | Description |
|---|---|---|---|
| `event_id` | string | yes | Event 一意ID。 |
| `event_type` | enum | yes | Event 種別。 |
| `payload` | object | yes | Event 固有データ。 |
| `source` | enum | yes | `system` / `player` / `enemy`。 |
| `turn` | number | yes | 発行時ターン番号。 |
| `phase` | enum | yes | `resource` / `action` / `end`。 |
| `timestamp` | number | no | デバッグ用時刻。 |

### Event List
| Event Type | Payload Schema | Emitter | Enqueue Timing | Follow-up |
|---|---|---|---|---|
| `START_TURN` | `{ "side": "player|enemy" }` | `system` | ターン開始時 | `CHANGE_PHASE(resource)` |
| `CHANGE_PHASE` | `{ "phase": "resource|action|end" }` | `system` | フェーズ切替時 | `CHECK_INTERRUPT` |
| `PLAY_CARD` | `{ "actor_id": string, "card_id": string }` | `player|enemy` | Card 使用時 | `RESOLVE_EFFECT` |
| `INSTALL_MODULE` | `{ "actor_id": string, "card_id": string, "slot_index": number }` | `player|enemy` | 設置・交換時 | `RESOLVE_EFFECT` |
| `RESOLVE_EFFECT` | `{ "effect_id": string, "origin_card_id": string }` | `system` | 効果解決時 | `SELECT_TARGET` or state update |
| `SELECT_TARGET` | `{ "selector": "target_policy|target_scope", "candidate_ids": string[] }` | `system` | 攻撃・効果対象決定時 | `APPLY_DAMAGE` or effect apply |
| `APPLY_DAMAGE` | `{ "target_id": string, "final_damage": number }` | `system` | ダメージ確定時 | `DESTROY_CARD` 判定 |
| `DESTROY_CARD` | `{ "card_id": string, "owner": "player|enemy" }` | `system` | `hp==0` 到達時 | `CHECK_INTERRUPT` |
| `CHECK_INTERRUPT` | `{}` | `system` | 各 Step 解決直後 | `BATTLE_WIN` or `BATTLE_LOSE` |
| `BATTLE_WIN` | `{ "winner": "player" }` | `system` | 勝利条件成立時 | なし |
| `BATTLE_LOSE` | `{ "winner": "enemy" }` | `system` | 敗北条件成立時 | なし |
| `RNG_USED` | `{ "seed_before": number, "seed_after": number, "roll": number }` | `system` | 乱数使用直後 | なし |

### Reducer Contract
- `Reducer` は `Event Envelope` を入力として受ける。
- `Reducer` は `payload` を検証し、不正値は `INVALID_EVENT` 扱いで拒否する。
- `Reducer` 内で副作用（I/O、外部乱数、時刻取得）を実行しない。
- 後続処理は必ず `follow-up` に従って `event_queue` へ enqueue する。

### Validation Rules
- `event_type` は本書の `Event List` に含まれる値のみ許可する。
- `slot_index` は `0..2`（MVP の `Module Slot` 3 枠）に制限する。
- `final_damage` は 1 以上の整数とする。
- `candidate_ids` は重複を持たない。

### Minimal JSON Example
```json
{
  "event_id": "evt_000123",
  "event_type": "PLAY_CARD",
  "payload": {
    "actor_id": "player_core",
    "card_id": "ins_001"
  },
  "source": "player",
  "turn": 2,
  "phase": "action"
}
```

## 決定事項
- `Event` は `Event Envelope` 形式で統一する。
- `event_type` は本書の `Event List` を正本とする。
- 乱数処理は `RNG_USED` を `event_log` に残して再現性を担保する。
