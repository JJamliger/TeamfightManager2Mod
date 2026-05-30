# 버프 및 능력치

## `EntityStat`

최상위 `stat` 및 `growth`에 사용됩니다.

```json
{
  "attack": 40,
  "magic_power": 65,
  "hp": 620,
  "defence": 20,
  "magic_resistance": 30,
  "move_speed": 1050,
  "hp_regen": 2,
  "stack": 0,
  "crit_chance": 0
}
```

| 필드 | 유형 | 비고 |
| --- | --- | --- |
| `attack` | integer | 공격 능력치. |
| `magic_power` | integer | 마법 위력 능력치. |
| `hp` | integer | 체력. |
| `defence` | integer | 물리 방어력. |
| `magic_resistance` | integer | 마법 저항력. |
| `move_speed` | integer | 이동 속도. |
| `hp_regen` | integer | 체력 재생. |
| `stack` | integer | 일부 챔피언 로직에서 사용하는 일반 중첩 능력치. |
| `crit_chance` | integer | 치명타 확률 퍼센트, 0-100. |

Rust 기본값은 많은 필드에서 0이 아니지만, 모드에서는 챔피언이 읽기 쉽고 안정적으로 되도록 모든 필드를 명시적으로 설정해야 합니다.

## `DataBuffStateDef`

`AddBuff` 및 `AddCasterBuff`에 사용됩니다.

```json
{
  "name": "fire_focus",
  "duration": { "Time": { "tick": 180 } }, //지속시간
  "attack": 0,
  "attack_mult": 0,
  "magic_power": 20,
  "magic_power_mult": 0,
  "defence": 0,
  "defence_mult": 0,
  "hp": 0,
  "hp_mult": 0,
  "hp_regen": 0,
  "magic_resistance": 0,
  "magic_resistance_mult": 0,
  "move_speed_mult": 0,
  "attack_speed_mult": 0,
  "skill_cooldown_mult": 15,
  "ult_cooldown_mult": 0,
  "damaged_amplify": 0,
  "damaged_reduce": 0,
  "dot_amplify": 0,
  "base_attack_enemy_max_hp_damage": 0,
  "skill_enemy_max_hp_damage": 0,
  "self_max_hp_damage": 0,
  "base_attack_damaged_reduce": 0,
  "skill_damaged_reduce": 0,
  "defence_penetration": 0,
  "magic_resistance_penetration": 0,
  "range": 0,
  "heal_reduce": 0,
  "toughness": 0,
  "crit_chance": 0,
  "radius_mult": 0,
  "vamp": 0,
  "damage_reflect": 0,
  "cc_immune": false,
  "undying": false,
  "ignore_wall": false
}
```

Rust 구조체에서는 `name`만 필수입니다. 그 외 모든 필드에는 기본값이 있습니다.

| 필드 | 형식 | 기본값 | 비고 |
| --- | --- | --- | --- |
| `name` | 문자열 | 필수 | Buff id. 시각 효과나 로직이 이를 참조하는 경우 안정적으로 유지합니다. |
| `duration` | `BuffType` | `Permanent` | `Permanent`, `Time`, 또는 `WithShield`. |
| `attack` | 정수 | `0` | 고정 공격력. |
| `attack_mult` | 정수 | `0` | 공격력 백분율 보정치. |
| `magic_power` | 정수 | `0` | 고정 마력. |
| `magic_power_mult` | 정수 | `0` | 마력 백분율 보정치. |
| `defence` | 정수 | `0` | 고정 방어력. |
| `defence_mult` | 정수 | `0` | 방어력 백분율 보정치. |
| `hp` | 정수 | `0` | 고정 HP. |
| `hp_mult` | 정수 | `0` | HP 백분율 보정치. |
| `hp_regen` | 정수 | `0` | 고정 HP 재생. |
| `magic_resistance` | 정수 | `0` | 고정 마법 저항력. |
| `magic_resistance_mult` | 정수 | `0` | 마법 저항력 백분율 보정치. |
| `move_speed_mult` | 정수 | `0` | 이동 속도 백분율 보정치. |
| `attack_speed_mult` | 정수 | `0` | 공격 속도 백분율 보정치. |
| `skill_cooldown_mult` | 정수 | `0` | 기술 재사용 대기시간 보정치. |
| `ult_cooldown_mult` | 정수 | `0` | 궁극기 재사용 대기시간 보정치. |
| `damaged_amplify` | 정수 | `0` | 받는 피해 증폭. |
| `damaged_reduce` | 정수 | `0` | 받는 피해 감소. |
| `dot_amplify` | 정수 | `0` | 지속 피해 증폭. |
| `base_attack_enemy_max_hp_damage` | 정수 | `0` | 기본 공격 시 적 최대 HP 비례 피해. |
| `skill_enemy_max_hp_damage` | 정수 | `0` | 기술 사용 시 적 최대 HP 비례 피해. |
| `self_max_hp_damage` | 정수 | `0` | 자신의 최대 HP 비례 피해 보정치. |
| `base_attack_damaged_reduce` | 정수 | `0` | 기본 공격에 대한 피해 감소. |
| `skill_damaged_reduce` | 정수 | `0` | 기술에 대한 피해 감소. |
| `defence_penetration` | 정수 | `0` | 물리 방어 관통. |
| `magic_resistance_penetration` | 정수 | `0` | 마법 저항 관통. |
| `range` | 정수 | `0` | 사거리 보정치. |
| `heal_reduce` | 정수 | `0` | 회복량 감소. |
| `toughness` | 정수 | `0` | 군중 제어 저항 관련 능력치. |
| `crit_chance` | 정수 | `0` | 치명타 확률 보정치. |
| `radius_mult` | 정수 | `0` | 반경 수정치. |
| `vamp` | 정수 | `0` | 흡혈 방식 수정치. |
| `damage_reflect` | 정수 | `0` | 피해 반사. |
| `cc_immune` | 불리언 | `false` | 군중 제어 면역. |
| `undying` | 불리언 | `false` | 활성 상태 동안 사망을 방지합니다. |
| `ignore_wall` | 불리언 | `false` | 활성 상태 동안 벽을 무시할 수 있습니다. |

## `BuffType`

```json
"영구"
```

```json
{ "Time": { "tick": 180 } }
```

```json
"WithShield"
```

일반적인 일시 버프에는 `Time`을 사용합니다. `WithShield`는 관련 보호막이 사라지면 종료됩니다.