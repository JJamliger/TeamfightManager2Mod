# Champion 파일

데이터 챔피언은 `.data_champion` 확장자를 가진 JSON 파일입니다. 에셋 로더는 이를 `DataChampionInfo`로 파싱합니다.

## 최소 구조

```json
{
  "id": "my_mod_fire_mage",
  "category": "Magician",
  "tags": ["AP", "Range"],
  "stat": {
    "attack": 40,
    "magic_power": 65,
    "hp": 620,
    "defence": 20,
    "magic_resistance": 30,
    "move_speed": 1050,
    "hp_regen": 2,
    "stack": 0,
    "crit_chance": 0
  },
  "growth": {
    "attack": 3,
    "magic_power": 7,
    "hp": 75,
    "defence": 3,
    "magic_resistance": 3,
    "move_speed": 0,
    "hp_regen": 1,
    "stack": 0,
    "crit_chance": 0
  },
  "attack": { "action_name": "attack" },
  "skill": { "action_name": "skill" },
  "skill2": { "action_name": "skill2" },
  "ult": { "action_name": "ult" }
}
```

`attack`, `skill`, `skill2`는 로더에 필수입니다. `ult`는 선택 사항이며, 생략하면 해당 챔피언은 궁극기 행동이 없습니다.

## 최상위 필드

| 필드 | 유형 | 필수 | 기본값 | 비고 |
| --- | --- | --- | --- | --- |
| `id` | string | 예 | 없음 | 고유한 챔피언 id입니다. 출시 후에도 변경되지 않게 유지하십시오. |
| `category` | `ChampionCategory` | 아니오 | `Melee` | [열거형](enums.md)을 참조하십시오. |
| `tags` | `ChampionTag[]` | 아니오 | `[]` | UI/검색/균형 분류에 사용됩니다. |
| `stat` | `EntityStat` | 아니오 | 0이 아닌 기본 능력치 | 기본 1레벨 능력치입니다. [강화 효과 및 능력치](buffs-and-stats.md)를 참조하십시오. |
| `growth` | `EntityStat` | 아니오 | 0이 아닌 기본 능력치 | 레벨당 능력치 성장입니다. 보통 모든 필드를 명시적으로 설정합니다. |
| `attack` | `DataActionDef` | 예 | 없음 | 기본 공격 행동입니다. |
| `skill` | `DataActionDef` | 예 | 없음 | 첫 번째 기술 행동입니다. |
| `skill2` | `DataActionDef` | 예 | 없음 | 두 번째 기술 행동입니다. |
| `ult` | `DataActionDef` | 아니오 | 궁극기 없음 | 궁극기 행동입니다. |
| `sprite` | string | 아니오 | 없음 | `#sheet` 또는 `#anim`을 제외한 챔피언 시각 요소의 에셋 경로입니다. |
| `anim_prefix` | string 또는 null | 아니오 | null | 다시 바인딩해야 하는 애니메이션 소스를 사용할 때 필요합니다. 태그를 그대로 복사하려면 `""`를 사용하십시오. |
| `skill_icon` | `DataSkillIconDef` | 아니오 | 없음 | 공유 스프라이트 시트 아이콘 소스와 3개의 태그입니다. |
| `skill_icons` | string[] | 아니오 | 없음 | `skill`, `skill2`, `ult`에 대한 직접 아이콘 자산 경로 3개입니다. `skill_icon`보다 우선합니다. |
| `view_effects` | `DataViewEffectDef[]` | 아니오 | `[]` | 이름 있는 효과 시각 요소를 등록합니다. |
| `view_projectiles` | `DataViewProjectileDef[]` | 아니오 | `[]` | 이름 있는 투사체 시각 요소를 등록합니다. |
| `view_buffs` | `DataViewBuffDef[]` | 아니오 | `[]` | 이름 있는 강화 효과 시각 요소를 등록합니다. |

## 스킬 아이콘

각 아이콘이 개별 PNG일 때는 `skill_icons`를 사용합니다:

```json
{
  "skill_icons": [
    "asset/my_mod/icons/fire_skill",
    "asset/my_mod/icons/fire_skill2",
    "asset/my_mod/icons/fire_ult"
  ]
}
```

아이콘이 하나의 스프라이트 시트에 묶여 있을 때는 `skill_icon`을 사용합니다:

```json
{
  "skill_icon": {
    "source": "asset/my_mod/icons/fire_skills",
    "tags": ["fire_skill", "fire_skill2", "fire_ult"]
  }
}
```

UI는 `skill`에 아이콘 인덱스 `0`을, `skill2`에 `1`을, `ult`에 `2`를 사용합니다.

## 스프라이트 바인딩

`sprite`는 자산 기본 경로를 가리킵니다:

```json
{
  "sprite": "asset/my_mod/champions/fire_mage",
  "anim_prefix": ""
}
```

애니메이션 Aseprite/수동 애니메이션 소스의 경우 `anim_prefix`를 설정합니다. 게임은 다음을 생성하거나 다시 매핑합니다:

```text
asset/base/aseprite_resources/champions/<id>#sheet
asset/base/aseprite_resources/champions/<id>#anim
```

이렇게 하면 일반 챔피언 렌더러가 id로 당신의 데이터 챔피언을 찾을 수 있습니다.

정적 PNG의 경우 `anim_prefix`를 생략합니다. 게임은 `idle`, `attack`, `skill`, `skill2`, `ult`, `dead`, `run` 같은 공통 태그에 대해 1프레임 대체 애니메이션을 생성합니다.