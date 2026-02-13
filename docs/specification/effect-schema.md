# Effect Schema

## 目的
- `effect_ids` で参照する効果データの最小スキーマを定義する。
- `timing` / `params` / `stack_rule` を統一し、実装とデータ作成の解釈差を防ぐ。

## 対象範囲
- `game v0.1.0` の Battle で使用する効果データ。
- Card 本体にぶら下がる `effect_ids` の参照先定義。
- 実行エンジン内部の処理詳細（クラス設計・関数分割）は対象外。

## 本文
### Common Fields
| Field | Type | Required | Description |
|---|---|---|---|
| `effect_id` | string | yes | 効果の一意ID。 |
| `effect_type` | enum | yes | 例: `damage`, `add_shield`, `heal`, `add_status`, `add_attack_bonus`。 |
| `timing` | enum | yes | 発動タイミング。`on_play` / `on_install` / `on_turn_end` / `on_turn_start` / `passive`。 |
| `target_scope` | enum | yes | 効果対象。`self` / `ally_core` / `enemy_core` / `auto_target`。 |
| `stack_rule` | enum | yes | 重複時ルール。`refresh` / `additive` / `replace` / `ignore`。 |
| `params` | object | yes | 効果固有パラメータ。 |

### Params Minimum Contract
- `damage`: `{"amount": number}`
- `add_shield`: `{"amount": number}`
- `heal`: `{"amount": number}`
- `add_status`: `{"status_id": string, "stacks": number, "duration": number}`
- `add_attack_bonus`: `{"amount": number, "duration": number}`

### Validation Rules
- `effect_id` は全体で一意でなければならない。
- `params.amount` は 0 以上の整数とする。
- `params.duration` は 1 以上の整数とする（`effect_type` が永続効果の場合は不要）。
- `timing=passive` は `target_scope=self` を基本とする（例外は `TODO`）。

### Effect Type Execution Mapping
| `effect_type` | Resolver | Core Behavior |
|---|---|---|
| `damage` | `damage_resolver` | `FinalDamage` を計算し、`shield -> hp` の順で適用する。 |
| `add_shield` | `shield_resolver` | `params.amount` を `target.shield` に加算する。 |
| `heal` | `heal_resolver` | `params.amount` を `target.hp` に加算し `max_hp` で上限クリップする。 |
| `add_status` | `status_resolver` | `status_id` を `stack_rule` に従って付与・更新する。 |
| `add_attack_bonus` | `attack_bonus_resolver` | 攻撃補正値を `duration` 付きで付与する。 |

### Timing Execution Order
| Order | Timing | Resolve Phase | Resolve Rule |
|---|---|---|---|
| 1 | `on_turn_start` | `Resource` | ターン開始時に先に解決する。 |
| 2 | `on_play` | `Action` | Card 使用直後に即時解決する。 |
| 3 | `on_install` | `Action` | `Module Card` 設置・交換直後に解決する。 |
| 4 | `on_turn_end` | `End` | ターン終了時に解決する。 |
| 5 | `passive` | All Steps | 各 Step 解決時の修飾子として常時参照する。 |

### Target Scope Resolution Order
| `target_scope` | Primary Target | Tie Break Rule |
|---|---|---|
| `self` | 効果発生元の Card | なし |
| `ally_core` | 自陣 `Core Card` | なし |
| `enemy_core` | 敵陣 `Core Card` | なし |
| `auto_target` | `Target Priority` に従う対象 | 同率時は `Target Policy` -> `Module Slot` 番号昇順 |

### Minimal JSON Examples
```json
{
  "effect_id": "eff_guard_wall_v1",
  "effect_type": "add_shield",
  "timing": "on_install",
  "target_scope": "self",
  "stack_rule": "additive",
  "params": {
    "amount": 3
  }
}
```

```json
{
  "effect_id": "eff_auto_shot_v1",
  "effect_type": "damage",
  "timing": "on_turn_end",
  "target_scope": "auto_target",
  "stack_rule": "ignore",
  "params": {
    "amount": 2
  }
}
```

```json
{
  "effect_id": "eff_core_regen_v1",
  "effect_type": "heal",
  "timing": "on_turn_start",
  "target_scope": "ally_core",
  "stack_rule": "refresh",
  "params": {
    "amount": 1
  }
}
```

### TODO
- `target_scope` の候補追加（`all_enemies`, `all_allies` など）は拡張時に定義する。
- `effect_type` ごとの詳細上限値（例: `amount` 上限）はバランス調整で確定する。

## 決定事項
- `Card` は効果本体を持たず、`effect_ids` で `Effect` を参照する。
- `Effect` は `effect_id`, `effect_type`, `timing`, `target_scope`, `stack_rule`, `params` を必須とする。
- `effect_type` は `Effect Type Execution Mapping` の Resolver へ 1 対 1 で委譲する。
- `timing` の解決順は `on_turn_start -> on_play -> on_install -> on_turn_end` とし、`passive` は常時参照とする。
- `target_scope=auto_target` は `Target Priority` と同率時ルールに従って対象を決定する。
- `shield` 関連効果は `effect_type=add_shield` で表現する。
