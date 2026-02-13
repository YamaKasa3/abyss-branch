# Card Schema

## 目的
- `Card` データの最小スキーマを定義し、実装時の項目解釈を統一する。
- `Core Card` / `Module Card` / `Instant Card` の責務差を明確化する。

## 対象範囲
- `game v0.1.0` の Battle で使用する Card データ。
- カード本体項目と `effect_ids` 参照方式。
- 効果マスタ（`effect_id` 側）の詳細は `docs/specification/effect-schema.md` を参照する。

## 本文
### Common Fields
| Field | Type | Required | Description |
|---|---|---|---|
| `card_id` | string | yes | Card の一意ID。 |
| `card_type` | enum | yes | `core` / `module` / `instant`。 |
| `name` | string | yes | 表示名。 |
| `lifetime_tag` | enum | no | `persistent` / `ephemeral`。未指定時は `persistent`。 |
| `tags` | string[] | no | 例: `guard`, `siege`。 |
| `effect_ids` | string[] | no | 効果マスタを参照するID配列。 |

### Type Specific Fields
| Field | Core Card | Module Card | Instant Card | Description |
|---|---|---|---|---|
| `mana_cost` | no | yes | yes | 消費Mana。`Core Card` には持たせない。 |
| `atk` | yes | yes | no | 攻撃値。 |
| `hp` | yes | yes | no | 現在HP。 |
| `max_hp` | yes | yes | no | 最大HP。 |
| `shield` | yes | yes | no | 受ダメージを先に吸収する防護値。 |
| `target_policy` | no | yes | no | 自動ターゲット方針。 |
| `slot_constraint` | no | no | no | 将来拡張用。MVPは未使用。 |

### Behavior Constraints
- `Core Card` は Battle 開始時に初期配置される（召喚操作なし）。
- `Instant Card` は `Module Slot` を使用しない。
- `Instant Card` は `atk` / `hp` / `max_hp` / `shield` を持たない。
- `effect_ids` はカード本体に処理詳細を持たせず、効果定義を外部参照する。

### Target Policy Allowed Values
| Value | Meaning | Candidate Set |
|---|---|---|
| `front` | 前方優先で対象を選ぶ。 | `auto_target` 候補 |
| `lowest_hp` | 現在HPが最も低い対象を選ぶ。 | `auto_target` 候補 |
| `random` | 候補からランダムに 1 体選ぶ。 | `auto_target` 候補 |
| `core_only` | `Core Card` のみを対象に選ぶ。 | `Core Card` |

### Minimal JSON Examples
```json
{
  "card_id": "core_001",
  "card_type": "core",
  "name": "Abyss Core",
  "atk": 2,
  "hp": 30,
  "max_hp": 30,
  "shield": 0,
  "effect_ids": ["eff_core_regen_v1"]
}
```

```json
{
  "card_id": "mod_001",
  "card_type": "module",
  "name": "Bulwark Node",
  "mana_cost": 2,
  "tags": ["guard"],
  "atk": 3,
  "hp": 8,
  "max_hp": 8,
  "shield": 2,
  "target_policy": "lowest_hp",
  "effect_ids": ["eff_guard_wall_v1"]
}
```

```json
{
  "card_id": "ins_001",
  "card_type": "instant",
  "name": "Overclock Burst",
  "mana_cost": 1,
  "lifetime_tag": "ephemeral",
  "effect_ids": ["eff_overclock_burst_v1"]
}
```

## 決定事項
- `Instant Card` は `atk` / `hp` / `max_hp` / `shield` を持たない。
- `Core Card` は `atk` / `hp` / `max_hp` / `shield` を持つ。
- `Core Card` は `mana_cost` を持たない。
- `Core Card` は Battle 開始時に初期配置する。
- 効果定義は `effect_ids` による ID 参照方式を採用する。
- `target_policy` の許可値は `front` / `lowest_hp` / `random` / `core_only` とする。
