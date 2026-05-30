# 효과

`effect` 필드는 `DataEffectDef`를 사용합니다. 모든 효과 객체에는 `type` 문자열이 있습니다. 이 페이지는 현재 `.data_champion` JSON에서 허용되는 효과 유형을 설명합니다.

게임 엔진에는 이 페이지에 공개된 것보다 더 많은 기본 `EffectType` 구현이 포함되어 있습니다. 데이터 전용 챔피언은 게임 코드에서 다른 JSON 매핑을 추가하지 않는 한 여기 나열된 `DataEffectDef` 변형만 사용할 수 있습니다. 나머지 내부 전용 범주는 끝부분 근처의 [데이터에 노출되지 않은 엔진 효과](#engine-effects-not-exposed-to-data)를 참고하십시오.

## 공통 규칙

`tick`, `duration`, `delay`, `apply`, `period`, `travel_time` 등의 시간 값은 시뮬레이션 틱입니다.

대부분의 투사체 및 이동 효과에는 중첩 효과를 포함할 수 있습니다. `applied_effects` 항목은 `effect`와 선택적 `casting_type`을 갖는 객체입니다:

```json
{
  "effect": { "type": "Attack", "damage": 50 },
  "casting_type": "Targeting"
}
```

`casting_type`의 기본값은 `Targeting`입니다.

`end_effects`는 보통 원시 효과 객체 목록입니다. `RangePeriodProjectile.end_effects`는 예외로, 주기적 범위가 범위 안의 대상에게 종료 효과를 적용할 수 있기 때문에 `applied_effects`와 동일한 `{ "effect", "casting_type" }` 항목 형식을 사용합니다.

대상 선택기의 기본값은 Rust 열거형 기본값에서 오기 때문에 흔히 `Ally`입니다. 피해, 공격형 투사체, 적 대상 범위 효과의 경우 `casting_target`, `applied_target`, 또는 `target`을 명시적으로 설정하십시오.

## 지원 유형 요약

| 범주 | 유형 |
| --- | --- |
| 피해 및 유지 | `Attack`, `ApAttack`, `FixedAttack`, `Heal`, `Shield` |
| 군중 제어 및 행동 불가 | `Stun`, `Airborne`, `Knockback`, `Grab`, `Pull`, `Fear`, `Charm`, `Bind`, `Taunt`, `BlockAttack`, `BlockSkill`, `BlockMoveSkill`, `Invisible`, `Banish` |
| 이동 | `Rush`, `RushTime`, `Teleport`, `DirTeleport`, `MoveBack`, `MoveTo`, `MoveToTarget`, `RushMoveToBack` |
| 투사체 및 지연 범위 효과 | `LinearProjectile`, `BackToCasterLinearProjectile`, `TargetProjectile`, `TargetProjectileFromProjectile`, `TargetSplashProjectile`, `AutoTargetProjectile`, `RangeProjectile`, `LineRangeProjectile`, `RangePeriodProjectile`, `ApplyInProjectile`, `ParabolicProjectile` |
| 범위와 장벽 | `RangeEffect`, `ShrinkingBarrier` |
| 강화 효과와 지속 효과 | `AddBuff`, `AddCasterBuff`, `RemoveCasterBuff`, `AddCasted` |
| 조합과 분기 | `Combine`, `Delayed`, `WithSelf`, `SwitchByBuff`, `SwitchByLevel3`, `RandomTarget` |
| 시각 및 음향 발동 | `ViewEffect`, `CasterViewEffect`, `CasterAnimation`, `RemoveCasterAnimation`, `Sfx`, `TargetSfx` |

## 피해 및 유지

### `Attack`

물리 공격 피해를 줍니다. AD 유형의 기술 피해나 기본 공격과 유사한 피해에 사용합니다.

```json
{
  "type": "Attack",
  "damage": 50,
  "attack_ratio": 100,
  "hp_ratio": 0,
  "target_hp_ratio": 0,
  "attack_effect_type": "Target"
}
```

- `damage`: 고정 피해량.
- `attack_ratio`: 시전자의 공격력 중 피해에 추가되는 비율.
- `hp_ratio`: 시전자의 최대 HP 중 피해에 추가되는 비율.
- `target_hp_ratio`: 대상의 최대 HP 중 피해에 추가되는 비율.
- `attack_effect_type`: 대상 지정 방식. [Enums](enums.md#dataattackeffecttype)을 참조하십시오.

기본값: `damage = 0`, `attack_ratio = 100`, `hp_ratio = 0`, `target_hp_ratio = 0`, `attack_effect_type = "Target"`.

### `ApAttack`

마력 기반 피해를 가합니다. AP 계열 기술 피해에 사용하십시오.

```json
{
  "type": "ApAttack",
  "damage": 80,
  "attack_ratio": 100,
  "hp_ratio": 0,
  "attack_effect_type": "Target"
}
```

- `damage`: 고정 마법 피해.
- `attack_ratio`: 시전자의 마력 중 피해에 추가되는 비율.
- `hp_ratio`: 시전자의 최대 HP 중 피해에 추가되는 비율.
- `attack_effect_type`: 대상 지정 방식.

기본값: `damage = 0`, `attack_ratio = 100`, `hp_ratio = 0`, `attack_effect_type = "Target"`.

### `FixedAttack`

고정 피해 판정을 통해 피해를 가합니다. 효과가 일반적인 AD 또는 AP 피해처럼 작동하지 않아야 할 때 사용하십시오.

```json
{
  "type": "FixedAttack",
  "damage": 120,
  "attack_ratio": 0,
  "hp_ratio": 0,
  "target_hp_ratio": 0,
  "attack_effect_type": "Target"
}
```

필드와 기본값은 `Attack`과 같지만, 피해 계산은 고정 피해 판정을 따릅니다.

### `Heal`

체력을 회복합니다. `heal_type`을 사용하여 효과가 시전자 자신, 선택한 대상, 아군만, 또는 주변 아군을 회복할지 결정하십시오.

```json
{
  "type": "Heal",
  "amount": 80,
  "attack_ratio": 0,
  "ap_ratio": 40,
  "heal_type": "Ally"
}
```

- `amount`: 고정 회복량.
- `attack_ratio`: 시전자 공격력 중 회복량에 추가되는 비율.
- `ap_ratio`: 시전자 마력 중 회복량에 추가되는 비율.
- `heal_type`: 회복 대상 지정 방식. [Enums](enums.md#datahealtype)를 참조하십시오.

기본값: `amount = 0`, `attack_ratio = 0`, `ap_ratio = 0`, `heal_type = "Any"`.

### `Shield`

대상에게 일시적인 보호막을 부여합니다.

```json
{
  "type": "Shield",
  "amount": 120,
  "attack_ratio": 0,
  "ap_ratio": 50,
  "tick": 300
}
```

- `amount`: 고정 보호막 수치.
- `attack_ratio`: 시전자의 공격력 중 보호막에 추가되는 비율.
- `ap_ratio`: 시전자의 마력 중 보호막에 추가되는 비율.
- `tick`: 보호막 지속 시간.

기본값: `amount = 0`, `attack_ratio = 0`, `ap_ratio = 0`, `tick = 300`.

## 군중 제어 및 행동 불가

### `Stun`

`duration` tick 동안 대상이 행동하지 못하게 합니다.

```json
{ "type": "Stun", "duration": 60 }
```

### `Airborne`

`duration` tick 동안 대상을 띄워 행동 불가 상태로 만듭니다. 공중에 띄우는 강력한 군중 제어에 사용하십시오.

```json
{ "type": "Airborne", "duration": 45 }
```

### `Knockback`

주어진 `speed`로 `tick` tick 동안 대상을 밀쳐냅니다.

```json
{ "type": "Knockback", "speed": 2000, "tick": 10 }
```

### `Grab`

대상을 시전자 쪽으로 끌어당깁니다. `tick`은 선택 항목이며, 생략하면 엔진이 잡기 효과의 내부 타이밍 동작을 사용합니다.

```json
{ "type": "Grab", "speed": 3500, "tick": 12 }
```

### `Pull`

고정된 tick 수 동안 대상을 효과 지점/시전자 쪽으로 끌어당깁니다.

```json
{ "type": "Pull", "speed": 2500, "tick": 15 }
```

### `Fear`

`tick` tick 동안 대상이 공포 행동을 하게 만듭니다.

```json
{ "type": "Fear", "tick": 90 }
```

### `Charm`

`tick` tick 동안 대상이 매혹 행동을 하게 만듭니다.

```json
{ "type": "Charm", "tick": 90 }
```

### `Bind`

`duration` tick 동안 대상의 이동을 묶거나 봉쇄합니다.

```json
{ "type": "Bind", "duration": 90 }
```

### `Taunt`

`duration` tick 동안 대상을 도발합니다.

```json
{ "type": "Taunt", "duration": 90 }
```

### `BlockAttack`

`tick` tick 동안 기본 공격을 막습니다.

```json
{ "type": "BlockAttack", "tick": 90 }
```

### `BlockSkill`

`tick` tick 동안 기술 사용을 막습니다.

```json
{ "type": "BlockSkill", "tick": 90 }
```

### `BlockMoveSkill`

`tick` tick 동안 이동 기술을 막습니다.

```json
{ "type": "BlockMoveSkill", "tick": 90 }
```

### `Invisible`

대상을 `tick`틱 동안 보이지 않게 만듭니다.

```json
{ "type": "Invisible", "tick": 120 }
```

### `Banish`

잠금 시작 시점과 종료 시점의 시각 효과 이름을 선택적으로 지정하여, 대상을 일시적으로 제거하거나 잠급니다.

```json
{
  "type": "Banish",
  "duration": 120,
  "lock_effect_name": "banish_lock",
  "end_effect_name": "banish_end"
}
```

`lock_effect_name` 및 `end_effect_name`의 기본값은 빈 문자열입니다.

## 이동

### `돌진`

시전자를 입력한 위치를 향해 이동시킵니다. 돌진하는 동안 시전자는 `casting_target`에 맞는 개체에게 중첩 효과를 적용할 수 있습니다.

```json
{
  "type": "Rush",
  "speed": 3500,
  "range": 50000,
  "move_speed_ratio": 0,
  "casting_target": "Enemy",
  "penetrate": false,
  "applied_effects": []
}
```

- `speed`: 기본 돌진 속도입니다.
- `range`: 돌진 중 타격/판정 범위입니다.
- `move_speed_ratio`: 시전자의 이동 속도에 비례해 추가되는 속도 백분율입니다.
- `casting_target`: 돌진 중 영향을 받는 개체입니다.
- `penetrate`: 첫 번째 적중 대상에서 멈추지 않고 계속 관통할지 여부입니다.
- `applied_effects`: 충돌한 대상에게 적용되는 효과입니다.

기본값: `range = 0`, `move_speed_ratio = 0`, `casting_target = "Ally"`, `penetrate = false`, `applied_effects = []`.

### `시간돌진`

클릭한 목적지로 이동하는 대신, 입력한 방향으로 고정된 틱 수 동안 시전자를 이동시킵니다.

```json
{
  "type": "RushTime",
  "speed": 3500,
  "tick": 30,
  "range": 50000,
  "casting_target": "Enemy",
  "penetrate": false,
  "applied_effects": []
}
```

### `순간이동`

시전자를 입력한 위치 또는 대상 위치로 즉시 이동시킵니다.

```json
{ "type": "Teleport" }
```

### `DirTeleport`

시전자를 방향 입력을 따라 `moved` 거리만큼 즉시 이동시킵니다.

"ap_ratio": 50,
{ "type": "DirTeleport", "moved": 32000 }
- `tick`: 보호막 지속 시간.

### `MoveBack`

시전자를 대상의 위치로부터 멀어지게 `tick`틱 동안 `speed` 속도로 이동시킵니다.

```json
{ "type": "MoveBack", "speed": 2500, "tick": 12 }
```

### `MoveTo`

대상, 위치 또는 방향을 향해 이동 상태를 시작합니다. 이동이 끝나면 `end_effects`가 발동합니다.

```json
{
  "type": "MoveTo",
  "speed": 3500,
  "range": 50000,
  "end_effects": [
    { "type": "RangeEffect", "target": "Enemy", "effects": [{ "type": "Attack", "damage": 40 }] }
  ]
}
```

`range`는 방향 입력에 사용됩니다. 기본값: `range = 0`, `end_effects = []`.

### `MoveToTarget`

대상 개체를 추적하는 이동 상태를 시작합니다. 효과가 해소될 때 대상이 사라졌다면, `end_effects`는 원래 입력값을 대상으로 즉시 실행됩니다.

```json
{
  "type": "MoveToTarget",
  "speed": 3500,
  "range": 50000,
  "end_effects": []
}
```

### `RushMoveToBack`

대상의 뒤로 돌진한 뒤 이동 지연 후 `applied_effects`를 발동합니다. 돌진 후방 이동 또는 배후 찌르기 형식의 기술에 사용하십시오.

```json
{
  "type": "RushMoveToBack",
  "speed": 4500,
  "applied_effects": [
    { "type": "Attack", "damage": 80, "attack_ratio": 100 }
  ]
}
```

## 투사체 및 지연 영역

### `LinearProjectile`

시전자에게서 대상, 위치 또는 방향을 향해 직선으로 날아가는 투사체를 생성합니다. 명중 시 `applied_effects`를 적용하고, 종료될 때 `end_effects`를 실행합니다.

```json
{
  "type": "LinearProjectile",
  "penetrate": true,
  "speed": 4200,
  "range": 65000,
  "name": "fire_skill",
  "shape": { "Circle": { "radius": 8000 } },
  "applied_target": "Enemy",
  "applied_effects": [
    { "effect": { "type": "ApAttack", "damage": 50, "attack_ratio": 80 } }
  ],
  "end_effects": []
}
```

기본값: `penetrate = false`, `shape = { "Circle": { "radius": 10000 } }`, `applied_target = "Ally"`, `applied_effects = []`, `end_effects = []`.

### `BackToCasterLinearProjectile`

시전자의 뒤쪽 위치에서 시전자에게 되돌아오는 직선 투사체를 생성합니다. 투사체의 종료 효과는 위치 입력을 전달하므로, 이는 주로 다른 투사체의 `end_effect`로 유용합니다.

```json
{
  "type": "BackToCasterLinearProjectile",
  "penetrate": true,
  "speed": 4200,
  "range": 65000,
  "name": "return_blade",
  "shape": { "Circle": { "radius": 8000 } },
  "applied_target": "Enemy",
  "applied_effects": [],
  "end_effects": []
}
```

### `TargetProjectile`

선택한 대상을 추적하며 명중 시 효과를 적용하는 투사체를 생성합니다.

```json
{
  "type": "TargetProjectile",
  "speed": 4500,
  "name": "arrow_hit",
  "y_offset": 0,
  "applied_target": "Enemy",
  "applied_effects": []
}
```

### `TargetProjectileFromProjectile`

현재 투사체 위치에서 대상을 추적하는 투사체를 생성합니다. 이는 다른 투사체의 `applied_effects` 또는 `end_effects`처럼 투사체 선택 정보가 있는 효과 연계 내부에서만 작동합니다.

```json
{
  "type": "TargetProjectileFromProjectile",
  "speed": 4500,
  "name": "split_bolt",
  "y_offset": 0,
  "applied_target": "Enemy",
  "applied_effects": []
}
```

### `TargetSplashProjectile`

대상을 추적하는 투사체를 생성한 뒤, `range` 안의 다른 대상으로 도약/확산할 수 있습니다.

```json
{
  "type": "TargetSplashProjectile",
  "speed": 4500,
  "name": "splash_hit",
  "range": 22000,
  "y_offset": 0,
  "applied_target": "Enemy",
  "applied_effects": []
}
```

### `AutoTargetProjectile`

`range` 안의 적 대상을 자동으로 선택하며, 유효할 경우 시전자가 최근 공격한 대상을 우선한 뒤, 대상을 추적하는 투사체를 발사합니다.

```json
{
  "type": "AutoTargetProjectile",
  "speed": 4500,
  "range": 60000,
  "name": "auto_bolt",
  "y_offset": 0,
  "applied_target": "Enemy",
  "applied_effects": []
}
```

### `RangeProjectile`

위치 입력 지점에 지연 발동 범위를 생성합니다. `delay` 틱 이후, `shape` 안의 대상에게 `apply` 틱 동안 효과를 적용합니다.

```json
{
  "type": "RangeProjectile",
  "name": "delayed_zone",
  "delay": 30,
  "apply": 60,
  "shape": { "Circle": { "radius": 26000 } },
  "applied_target": "Enemy",
  "applied_effects": []
}
```

### `LineRangeProjectile`

시전자에서 입력 방향/대상/위치를 향해 지연 후 발동하는 선형 범위를 생성합니다.

```json
{
  "type": "LineRangeProjectile",
  "width": 8000,
  "length": 70000,
  "delay": 20,
  "apply": 30,
  "name": "line_blast",
  "applied_target": "Enemy",
  "applied_effects": []
}
```

### `RangePeriodProjectile`

위치 입력 지점에 고정된 주기 범위를 생성합니다. `tick`이 만료될 때까지 매 `period` 틱마다 `applied_effects`를 적용하고, 이후 범위 안에 아직 남아 있는 개체에 `end_effects`를 적용합니다.

```json
{
  "type": "RangePeriodProjectile",
  "name": "burning_ground",
  "tick": 180,
  "period": 30,
  "first_delay": 0,
  "shape": { "Circle": { "radius": 26000 } },
  "applied_target": "Enemy",
  "applied_effects": [
    { "effect": { "type": "ApAttack", "damage": 20 }, "casting_type": "Targeting" }
  ],
  "end_effects": []
}
```

`period`는 `0`보다 커야 하며, 불량 데이터를 방지하기 위해 로더가 최소 `1`로 보정합니다.

### `ApplyInProjectile`

보이지 않는 투사체/범위를 생성하여 `tick` 틱 동안 대기한 뒤, `shape` 안의 대상에게 효과를 적용합니다. `follow_caster`가 true이면 적용될 때까지 해당 범위가 시전자를 따라갑니다.

```json
{
  "type": "ApplyInProjectile",
  "name": "delayed_self_aura",
  "follow_caster": true,
  "tick": 45,
  "shape": { "Circle": { "radius": 24000 } },
  "applied_target": "Enemy",
  "applied_effects": []
}
```

### `ParabolicProjectile`

대상 또는 위치를 향해 고정된 포물선 궤적으로 이동하는 투사체를 생성합니다. 도착 시 명중 효과를 적용한 뒤 `end_effects`를 적용합니다. `range_effect_name`은 이동 중 미리보기/충돌 범위 시각 효과를 표시할 수 있습니다.

```json
{
  "type": "ParabolicProjectile",
  "name": "meteor",
  "travel_time": 45,
  "range": 70000,
  "range_effect_name": "meteor_impact",
  "shape": { "Circle": { "radius": 24000 } },
  "applied_target": "Enemy",
  "applied_effects": [],
  "end_effects": []
}
```

`range_effect_name`의 기본값은 빈 문자열입니다.

## 범위 및 장벽

### `RangeEffect`

`shape` 내부 대상에게 중첩 효과를 즉시 적용합니다. 범위는 시전자 중심으로 지정하거나 시전자 전방에 배치할 수 있습니다.

```json
{
  "type": "RangeEffect",
  "shape": { "Circle": { "radius": 42000 } },
  "target": "Enemy",
  "apply_type": "AroundCaster",
  "effects": [
    { "type": "ApAttack", "damage": 120, "attack_ratio": 100 },
    { "type": "Stun", "duration": 45 }
  ]
}
```

기본값: `target = "Ally"`, `apply_type = "AroundCaster"`, `effects = []`.

### `ShrinkingBarrier`

대상을 따라다니며 시간이 지날수록 축소되는 원형 장벽을 생성합니다. 효과는 엔진의 장벽 동작 방식에 따라 장벽 가장자리에 적용됩니다.

```json
{
  "type": "ShrinkingBarrier",
  "name": "closing_ring",
  "start_radius": 70000,
  "end_radius": 16000,
  "shrink_per_tick": 800,
  "tick": 120,
  "edge_thickness": 6000,
  "applied_effects": [
    { "effect": { "type": "Bind", "duration": 20 } }
  ]
}
```

## 강화 효과 및 지속 효과

### `AddBuff`

선택한 대상에게 능력치/상태 `buff_state`를 추가합니다.

```json
{
  "type": "AddBuff",
  "buff_state": {
    "name": "burning",
    "duration": { "Time": { "tick": 180 } },
    "damaged_amplify": 115
  }
}
```

모든 `buff_state` 필드는 [강화 효과 및 능력치](buffs-and-stats.md)를 참조하십시오.

### `AddCasterBuff`

시전자에게 능력치/상태 `buff_state`를 추가합니다. `only_to_enemy`가 true이면 원래 대상이 적일 때만 강화 효과가 추가됩니다.

```json
{
  "type": "AddCasterBuff",
  "only_to_enemy": false,
  "buff_state": {
    "name": "focus",
    "duration": { "Time": { "tick": 180 } },
    "magic_power": 20
  }
}
```

`only_to_enemy`의 기본값은 `false`입니다.

### `RemoveCasterBuff`

시전자에게서 이름이 지정된 일반 강화 효과를 제거합니다. 모드 강화 효과 종료처럼 일시적인 자기 상태를 정리할 때 사용합니다.

```json
{ "type": "RemoveCasterBuff", "name": "focus" }
```

### `AddCasted`

선택한 대상에게 주기적인 지속 효과를 추가합니다. `duration`까지 매 `period` 틱마다 각 중첩 효과가 해당 대상에게 적용됩니다.

```json
{
  "type": "AddCasted",
  "duration": 180,
  "period": 30,
  "casted_type": "Fire",
  "effects": [
    { "type": "ApAttack", "damage": 20, "attack_ratio": 20 }
  ]
}
```

`casted_type`은 지속 효과 범주/아이콘과 공격 유형을 제어합니다. 값: `Bleed`, `Poison`, `Fire`, `Heal`. 기본값: `Fire`. `period`는 `0`보다 커야 하며, 로더는 이를 최소 `1`로 보정합니다.

## 조합 및 분기

### `Combine`

동일한 입력으로 여러 효과를 즉시 실행합니다.

```json
{
  "type": "Combine",
  "effects": [
    { "type": "Attack", "damage": 50 },
    { "type": "Heal", "amount": 30, "heal_type": "Caster" }
  ]
}
```

### `Delayed`

중첩된 효과를 `tick`틱 후에 실행하도록 대기열에 넣습니다.

```json
{
  "type": "Delayed",
  "tick": 30,
  "effects": [
    { "type": "Stun", "duration": 30 }
  ]
}
```

### `WithSelf`

시전자를 대상/자기 자신 문맥으로 하여 중첩된 효과를 실행합니다.

```json
{
  "type": "WithSelf",
  "effects": [
    { "type": "AddCasterBuff", "buff_state": { "name": "self_mark" } }
  ]
}
```

### `SwitchByBuff`

시전자가 현재 `buff_name`을 보유하고 있는지에 따라 두 효과 중 하나를 선택합니다.

```json
{
  "type": "SwitchByBuff",
  "buff_name": "empowered",
  "effect_none": { "type": "Attack", "damage": 40 },
  "effect_buff": { "type": "Attack", "damage": 90 }
}
```

### `SwitchByLevel3`

3레벨 이전에는 한 효과를, 3레벨 이상에서는 다른 효과를 선택합니다.

```json
{
  "type": "SwitchByLevel3",
  "effect_start": { "type": "Shield", "amount": 80 },
  "effect_level3": { "type": "Shield", "amount": 160 }
}
```

### `RandomTarget`

`range` 내에서 `casting_target`과 일치하는 무작위 대상을 선택한 다음, 그 대상에게 중첩된 효과를 적용합니다.

```json
{
  "type": "RandomTarget",
  "range": 65000,
  "casting_target": "EnemyChampion",
  "from_projectile": false,
  "effects": [
    { "type": "ApAttack", "damage": 80 }
  ]
}
```

기본값: `casting_target = "Ally"`, `from_projectile = false`.

## 시각 및 음향 트리거

이는 이름이 지정된 시각/음향 계통을 작동시킵니다. 사용자 지정 계통은 [Visual Bindings](visual-bindings.md)를 통해 등록하십시오.

### `ViewEffect`

효과 대상/입력 위치에서 이름이 지정된 시각 효과를 재생합니다.

```json
{ "type": "ViewEffect", "name": "fire_burst" }
```

### `CasterViewEffect`

시전자 위치에서 이름이 지정된 시각 효과를 재생합니다.

```json
{ "type": "CasterViewEffect", "name": "caster_flash" }
```

### `CasterAnimation`

`tick`틱 동안 이름이 지정된 시전자 애니메이션 상태를 추가합니다.

```json
{ "type": "CasterAnimation", "name": "cast_pose", "tick": 30 }
```

### `RemoveCasterAnimation`

이름이 지정된 시전자 애니메이션 상태를 제거합니다.

```json
{ "type": "RemoveCasterAnimation", "name": "cast_pose" }
```

### `Sfx`

시전자 위치에서 이름이 지정된 음향 효과를 재생합니다.

```json
{ "type": "Sfx", "name": "fire_cast" }
```

### `TargetSfx`

대상, 위치 입력 지점 또는 현재 투사체 위치에서 이름이 지정된 음향 효과를 재생합니다.

```json
{ "type": "TargetSfx", "name": "fire_hit" }
```

## 데이터에 노출되지 않은 엔진 효과

엔진에는 현재 `.data_champion` JSON에 매핑되지 않은 추가 효과 구현이 있습니다. 위에서 지원되는 `type` 문자열로 의도적으로 나열하지 않았습니다.

| 내부 분류 | 아직 일반 데이터 효과가 아닌 이유 |
| --- | --- |
| `AddEffectBuff`, `AddCasterEffectBuff`, `DamageShare` | 이들은 런타임 버프 동작 특성인 `EffectBuff`를 사용합니다. 현재 데이터 JSON은 임의의 효과 버프 동작 객체가 아니라 일반 능력치/상태 버프를 모델링합니다. |
| `SpawnBear`, `SpawnEagle`, `SpawnGhoul`, `SpawnIllusion`, `SpawnRevenant` 같은 소환 | 이들은 하드코딩된 소환 개체 동작과 챔피언별 가정에 의존합니다. |
| `TowerAttack` | 장엄한 타이밍 동작을 포함한 타워/미니언 전투 규칙에 묶여 있습니다. 일반적인 챔피언 기술 효과가 아닙니다. |
| 바드, 드루이드, 늑대인간, 부두 주술사, 빙결 마법사 및 유사한 궁극기/기술 모듈과 같은 챔피언 전용 효과 | 이들은 맞춤형 챔피언 메커니즘을 인코딩합니다. 모드 제작자에게 안전하게 노출하기 전에 별도의 데이터 스키마 작업이 필요합니다. |
| 튕기는 대상 이동과 같은 추가 투사체 이동 내부 기능 | 일부는 하드코딩된 챔피언 효과를 통해서만 접근할 수 있습니다. 공개 지원 전에 명시적인 JSON 필드와 검증이 필요합니다. |

데이터 지원에 다른 효과를 추가할 때는 `game-core/src/setting/champion/data_driven.rs`의 `DataEffectDef`를 갱신하고, 해당 JSON 형식을 여기 문서화하십시오.