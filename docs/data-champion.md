# 데이터 전용 챔피언

데이터 전용 챔피언은 시작하기에 가장 좋은 지점입니다. 이들은 `.data_champion` 확장자를 가진 JSON 파일이며, Rust 코드를 작성하지 않고도 플레이 가능한 챔피언을 추가할 수 있게 해줍니다.

모드 폴더 아래의 어느 위치에 두어도 되며, 일반적으로는 다음과 같습니다:

```text
mods/my_mod/champion/fire_mage.data_champion
```

게임은 해당 파일을 다음 에셋 경로로 인식합니다:

```text
asset/my_mod/champion/fire_mage
```

## 전체 스키마 참조

이 페이지는 튜토리얼 형식의 예시입니다. 각 필드, enum 값, 효과 유형, 시각 바인딩, 패치 가능한 필드에 대해서는 [데이터 챔피언 스키마](data-champion-schema/index.md)를 참조하십시오.

## 예시 챔피언

아래 예시는 기본 공격, 두 개의 기술, 궁극기, 기술 아이콘, 투사체 시각 효과를 갖춘 원거리 마법 챔피언을 정의합니다.

```json
{
  "id": "my_mod_fire_mage",
  "category": "Magician",
  "tags": ["AP", "Range"],
  "sprite": "asset/base/aseprite_resources/champions/pyromancer",
  "anim_prefix": "",
  "skill_icons": [
    "asset/my_mod/icons/fire_mage_skill",
    "asset/my_mod/icons/fire_mage_skill2",
    "asset/my_mod/icons/fire_mage_ult"
  ],
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
  "attack": {
    "action_name": "attack",
    "duration": 18,
    "cooltime": 60,
    "start_timing": 10,
    "cancelable": true,
    "range": 52000,
    "casting_type": "Targeting",
    "casting_target": "Enemy",
    "attack_type": "BaseAttack",
    "effect": {
      "type": "TargetProjectile",
      "speed": 4500,
      "name": "fire_mage_attack",
      "applied_target": "Enemy",
      "applied_effects": [
        {
          "effect": {
            "type": "Attack",
            "damage": 0,
            "attack_ratio": 100
          },
          "casting_type": "Targeting"
        }
      ]
    }
  },
  "skill": {
    "action_name": "skill",
    "description": "#asset/my_mod/text/champion?description.my_mod_fire_mage.skill",
    "duration": 20,
    "cooltime": 240,
    "start_timing": 10,
    "range": 65000,
    "casting_type": "Direction",
    "casting_target": "Enemy",
    "attack_type": "Skill",
    "effect": {
      "type": "LinearProjectile",
      "penetrate": true,
      "speed": 4200,
      "range": 65000,
      "name": "fire_mage_skill",
      "shape": { "Circle": { "radius": 8000 } },
      "applied_target": "Enemy",
      "applied_effects": [
        {
          "effect": {
            "type": "ApAttack",
            "damage": 50,
            "attack_ratio": 80
          },
          "casting_type": "Targeting"
        }
      ]
    }
  },
  "skill2": {
    "action_name": "skill2",
    "description": "#asset/my_mod/text/champion?description.my_mod_fire_mage.skill2",
    "duration": 16,
    "cooltime": 360,
    "start_timing": 8,
    "range": 0,
    "casting_type": "None",
    "casting_target": "AllyOnlySelf",
    "attack_type": "Skill",
    "effect": {
      "type": "AddCasterBuff",
      "buff_state": {
        "name": "fire_focus",
        "duration": { "Time": { "tick": 180 } },
        "magic_power": 20,
        "skill_cooldown_mult": 15
      }
    }
  },
  "ult": {
    "action_name": "ult",
    "description": "#asset/my_mod/text/champion?description.my_mod_fire_mage.ult",
    "duration": 25,
    "cooltime": 900,
    "start_timing": 12,
    "range": 42000,
    "casting_type": "None",
    "casting_target": "Enemy",
    "attack_type": "Skill",
    "effect": {
      "type": "RangeEffect",
      "shape": { "Circle": { "radius": 42000 } },
      "target": "Enemy",
      "apply_type": "AroundCaster",
      "effects": [
        {
          "type": "Combine",
          "effects": [
            {
              "type": "ApAttack",
              "damage": 120,
              "attack_ratio": 100
            },
            {
              "type": "Stun",
              "duration": 45
            }
          ]
        }
      ]
    }
  },
  "view_projectiles": [
    {
      "type": "Sprite",
      "name": "fire_mage_attack",
      "sprite": "asset/base/sprite/arrow"
    },
    {
      "type": "Sprite",
      "name": "fire_mage_skill",
      "sprite": "asset/base/sprite/arrow"
    }
  ]
}
```

## 최상위 필드

- `id`: 고유한 챔피언 id입니다. 저장 데이터와 패치가 이를 참조할 수 있으므로 출시 후에도 안정적으로 유지해야 합니다.
- `category`: `Melee`, `Range`, `Magician`, `Util`, 또는 `Assassin`.
- `tags`: `AD`, `AP`, `Heal`, `Shield`, `Dot`, `CC`, `Range`, `Melee`, `Tank`, `Magic` 중 하나입니다.
- `stat`: 레벨 1 기본 능력치입니다.
- `growth`: 레벨당 능력치 성장치입니다.
- `attack`, `skill`, `skill2`, `ult`: 필수 행동입니다.
- `sprite`: 챔피언 시각 요소에 사용하는 선택적 에셋 경로입니다. PNG, Aseprite 에셋 또는 수동 `#sheet` 와 `#anim` 애니메이션 쌍이 될 수 있습니다.
- `anim_prefix`: 애니메이션 스프라이트 원본과 함께 사용됩니다. 애니메이션 태그를 그대로 유지하려면 `""`로 설정하고, `"eagle_"` 같은 접두어를 제거하려면 해당 접두어로 설정합니다.
- `skill_icon`: 선택적 공용 아이콘 시트 정의입니다. skill, skill2, ult 아이콘이 태그와 함께 하나의 시트에 묶여 있을 때 사용합니다.
- `skill_icons`: 선택적 3개의 직접 아이콘 에셋 경로 목록입니다. 각 아이콘이 개별 PNG일 때 사용합니다.
- `view_effects`, `view_projectiles`, `view_buffs`: 선택적 화면 바인딩입니다.

## 행동 필드

- `action_name`: `attack`, `skill`, `skill2`, `ult` 같은 재생할 애니메이션 태그입니다.
- `duration`: 시뮬레이션 틱 기준 전체 행동 지속 시간입니다.
- `cooltime`: 틱 기준 재사용 대기시간입니다.
- `start_timing`: 행동 중 효과가 발동하는 틱입니다.
- `cancelable`: 행동이 중단될 수 있는지 여부입니다.
- `range`: 기본 시전 거리입니다.
- `growth_range`: 레벨당 추가 거리입니다.
- `casting_type`: `Targeting`, `Position`, `Direction`, 또는 `None`.
- `casting_target`: 아래 대상 목록을 참고하십시오.
- `attack_type`: `BaseAttack`, `Skill`, `Dot`, `DotIgnoreShield`, `Item`, 또는 `Well`.
- `can_use_with_move`: 이동 중에도 행동을 사용할 수 있는지 여부입니다.
- `description`: i18n 키 또는 일반 문자열입니다.
- `effect`: 효과 정의입니다.

## 애니메이션 태그

`action_name`은 게임플레이 행동 이름이면서 동시에 시각 조회 키이기도 합니다. 행동이 시작되면 챔피언 렌더러는 같은 이름의 애니메이션 태그를 재생하려고 시도합니다.

일반적인 챔피언 태그는 다음과 같습니다:

```text
idle
run
attack
skill
skill2
ult
dead
```

당신의 `skill` 동작이 다음을 사용한다면:

```json
{
  "action_name": "fire_cast"
}
```

그러면 스프라이트의 애니메이션 데이터에도 `fire_cast` 태그가 있어야 합니다. Aseprite에서는 타임라인 태그 이름이 `fire_cast`여야 합니다. 수동 `.fanim` 파일에서는 `anims.fire_cast` 아래에 항목이 있어야 한다는 뜻입니다.

특별한 애니메이션이 필요한 경우가 아니라면 먼저 익숙한 이름을 사용하십시오. 태그가 없으면 챔피언은 불러와지더라도 해당 동작 중에 기대한 애니메이션이 표시되지 않는 경우가 많습니다.

## 일반 대상

유용한 `casting_target` 값:

- `Enemy`
- `EnemyWithoutTower`
- `EnemyChampion`
- `EnemyChampionInCC`
- `Ally`
- `AllyChampion`
- `AllyNotSelf`
- `AllyOnlySelf`
- `Both`
- `BothChampion`
- `None`

## 효과 유형

데이터 챔피언은 다음 효과 유형을 사용할 수 있습니다:

- 피해 및 회복: `Attack`, `ApAttack`, `FixedAttack`, `Heal`, `Shield`.
- 군중 제어: `Stun`, `Airborne`, `Knockback`, `Grab`, `Pull`, `Fear`, `Charm`, `Bind`, `Taunt`, `BlockAttack`, `BlockSkill`, `BlockMoveSkill`, `Invisible`, `Banish`.
- 이동: `Rush`, `RushTime`, `Teleport`, `DirTeleport`, `MoveBack`, `MoveTo`, `MoveToTarget`, `RushMoveToBack`.
- 투사체: `LinearProjectile`, `TargetProjectile`, `TargetSplashProjectile`, `AutoTargetProjectile`, `RangeProjectile`, `ParabolicProjectile`, `BackToCasterLinearProjectile`, `TargetProjectileFromProjectile`, `LineRangeProjectile`, `RangePeriodProjectile`, `ApplyInProjectile`.
- 범위 및 장벽: `RangeEffect`, `ShrinkingBarrier`.
- 버프 및 시전 효과: `AddBuff`, `AddCasterBuff`, `RemoveCasterBuff`, `AddCasted`.
- 조합 및 분기: `Combine`, `Delayed`, `WithSelf`, `RandomTarget`, `SwitchByBuff`, `SwitchByLevel3`.
- 시각/음향 트리거: `ViewEffect`, `CasterViewEffect`, `CasterAnimation`, `RemoveCasterAnimation`, `Sfx`, `TargetSfx`.

이 페이지는 일반적인 묶음만 요약합니다. 각 효과 유형의 정확한 필드와 중첩 예시는 [Effect Schema](data-champion-schema/effects.md)를 사용하십시오.

## 형태

투사체 및 범위 효과는 `ProjectileShape`를 사용합니다:

```json
{ "Circle": { "radius": 10000 } }
```

```json
{ "Rect": { "width": 12000, "height": 8000 } }
```

```json
{ "DirDot": { "radius": 8000, "range": 700 } }
```

`Line`도 지원되지만, 명시적인 끝점이 필요하며 작성된 챔피언 데이터에는 보통 덜 편리합니다.

## 텍스트

설명은 보통 i18n 키를 가리킵니다:

```json
"#asset/base/text/champion?description.my_mod_fire_mage.skill"
```

i18n 병합을 통해 해당 키를 추가하십시오. [Asset Overrides and i18n](asset-overrides-and-i18n.md)을 참조하십시오.

## 스프라이트 바인딩

`sprite` 필드는 `#sheet` 또는 `#anim` 없이 작성합니다:

```json
{
  "sprite": "asset/example/champions/fire_mage"
}
```

게임은 해당 경로에 존재하는 항목을 기준으로 이를 어떻게 바인딩할지 결정합니다.

### 정적 PNG

`sprite`가 PNG를 가리키는 경우 `anim_prefix`를 설정하지 마십시오:

```json
{
  "sprite": "asset/example/champions/fire_mage_idle"
}
```

전체 이미지가 1프레임 스프라이트로 사용됩니다. 게임은 `idle`, `attack`, `skill`, `skill2`, `ult`, `dead`, `run`에 대해 단순한 대체 애니메이션을 생성합니다.

이는 챔피언을 시험하는 동안 유용합니다. 완성된 애니메이션 챔피언에는 이상적이지 않습니다.

### 애니메이션 Aseprite 또는 수동 애니메이션 시트

`sprite`가 Aseprite 애니메이션 또는 수동 `#sheet`와 `#anim` 쌍을 가리키는 경우 `anim_prefix`를 설정하십시오.

태그가 이미 표준 챔피언 태그와 일치하는 경우 빈 문자열을 사용하십시오:

```json
{
  "sprite": "asset/example/champions/fire_mage",
  "anim_prefix": ""
}
```

이렇게 하면 `idle`, `attack`, `skill`, `ult` 같은 태그가 변경되지 않은 채 유지됩니다.

하나의 원본 파일에 여러 변형이 들어 있는 경우 실제 접두사를 사용하십시오. 예를 들면:

```json
{
  "sprite": "asset/base/aseprite_resources/champions/druid",
  "anim_prefix": "eagle_"
}
```

원본 태그 `eagle_idle`, `eagle_attack`, `eagle_ult`는 새 챔피언에서 `idle`, `attack`, `ult`가 됩니다.

접두사가 없는 태그도 함께 유지됩니다. 이렇게 하면 공유 원본 파일에 모든 변형이 사용할 수 있는 공통 태그를 담을 수 있습니다.

`#sheet`, `#anim`, Aseprite 메타데이터, 수동 PNG/JSON 스프라이트 시트에 대한 전체 설명은 [Assets and Sprite Sheets](assets-and-sprite-sheets.md)를 참조하십시오.

## 스킬 아이콘

skill, skill2, ult 아이콘을 정의하는 방법은 두 가지입니다.

각 아이콘이 개별 PNG일 때는 `skill_icons`를 사용하십시오:

```json
{
  "skill_icons": [
    "asset/example/icons/fire_skill",
    "asset/example/icons/fire_skill2",
    "asset/example/icons/fire_ult"
  ]
}
```

아이콘이 하나의 스프라이트 시트로 묶여 있을 때는 `skill_icon`을 사용하십시오:

```json
{
  "skill_icon": {
    "source": "asset/example/icons/fire_skills",
    "tags": ["fire_skill", "fire_skill2", "fire_ult"]
  }
}
```

두 번째 형식에서는, 게임이 다음을 찾습니다:

```text
asset/example/icons/fire_skills#sheet
asset/example/icons/fire_skills#data
```

그리고 해당 시트에서 태그된 사각형을 그립니다.