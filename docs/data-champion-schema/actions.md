# 행동

`attack`, `skill`, `skill2`, `ult`는 `DataActionDef`를 사용합니다.

## 형태

```json
{
  "action_name": "skill", //기술
  "description": "#asset/my_mod/text/champion?description.my_mod_fire_mage.skill",
  "duration": 20, //지속시간
  "cooltime": 240, //재사용시간
  "start_timing": 10,
  "cancelable": true,
  "range": 65000, //사거리
  "growth_range": 0,
  "casting_type": "Direction",
  "casting_target": "Enemy",
  "attack_type": "Skill",
  "can_use_with_move": false,
  "effect": {
    "type": "LinearProjectile", //직선투사체
    "name": "fire_skill",
    "speed": 4200, //속도
    "range": 65000, //사거리
    "applied_target": "Enemy",
    "applied_effects": []
  }
}
```

## 필드

| 필드 | 형식 | 기본값 | 비고 |
| --- | --- | --- | --- |
| `action_name` | string | `""` | 애니메이션 태그 및 패치 유형 이름. 가급적 `attack`, `skill`, `skill2`, `ult`를 사용하십시오. |
| `duration` | integer | `0` | 시뮬레이션 틱 기준 전체 행동 지속 시간. |
| `cooltime` | integer | `0` | 틱 기준 재사용 대기시간. |
| `start_timing` | integer | `0` | 행동 내부에서 효과가 발동하는 틱. |
| `cancelable` | boolean | `false` | 행동이 중단될 수 있는지 여부. |
| `range` | integer | `0` | 기본 시전/효과 범위. |
| `growth_range` | integer | `0` | 레벨당 추가 범위. |
| `casting_type` | `CastingType` | `Targeting` | 플레이어/AI 입력이 대상을 선택하는 방식. |
| `casting_target` | `CastingTarget` | `Ally` | 선택할 수 있는 개체 유형. 보통 이를 명시적으로 설정하십시오. |
| `attack_type` | `AttackType` | `BaseAttack` | 전투 계통에서 사용하는 피해 분류. |
| `can_use_with_move` | boolean | `false` | 이동 중에 행동을 사용할 수 있는지 여부. |
| `description` | string | `""` | 일반 텍스트 또는 i18n 키. i18n 키에는 전체 `#asset/...?...` 형식이 포함되어야 합니다. |
| `effect` | `DataEffectDef` 또는 null | null | `start_timing`에 발생하는 내용. |

## 슬롯

다음 공개 슬롯 이름을 일관되게 사용하십시오:

- `attack`: 기본 공격.
- `skill`: 첫 번째 기술.
- `skill2`: 두 번째 기술.
- `ult`: 궁극기.

`action_name`에는 제공한 어떤 애니메이션 태그도 사용할 수 있지만, 같은 이름을 사용하면 데이터, 애니메이션, 문서를 예측 가능하게 유지할 수 있습니다.

## 대상 지정 요령

- `casting_type: "Targeting"`은 대상 개체가 필요합니다.
- `casting_type: "Position"`은 위치를 대상으로 합니다.
- `casting_type: "Direction"`은 시전자로부터의 방향을 대상으로 합니다.
- `casting_type: "None"`은 대상 입력이 필요 없으며, 보통 `AllyOnlySelf`, `None` 또는 시전자 주변의 범위 효과와 함께 사용됩니다.

행동의 `casting_target`과 모든 투사체/효과의 `applied_target` 또는 `target`을 모두 설정하십시오. Rust에서 대상 열거형의 기본값은 `Ally`이며, 피해 투사체에는 거의 올바르지 않습니다.