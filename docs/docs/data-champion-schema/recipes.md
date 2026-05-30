"tag": "burst"

이 코드 조각들은 행동 `effect`에 복사해 넣을 수 있는 작은 형식입니다.

## 기본 대상 투사체 공격

```json
{
  "type": "TargetProjectile",
  "speed": 4500,
  "name": "fire_attack",
  "applied_target": "Enemy",
  "applied_effects": [
    {
      "effect": {
        "type": "Attack",
        "damage": 0,
        "attack_ratio": 100
      }
    }
  ]
}
```

## 방향성 관통 기술

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
    {
      "effect": {
        "type": "ApAttack",
        "damage": 50,
        "attack_ratio": 80
      }
    }
  ]
}
```

다음 행동 필드와 함께 사용:

```json
{
  "action_name": "skill",
  "casting_type": "Direction",
  "casting_target": "Enemy",
  "attack_type": "Skill"
}
```

## 자신 강화

```json
{
  "type": "AddCasterBuff",
  "buff_state": {
    "name": "fire_focus",
    "duration": { "Time": { "tick": 180 } },
    "magic_power": 20,
    "skill_cooldown_mult": 15
  }
}
```

다음과 함께 사용:

```json
{
  "casting_type": "None",
  "casting_target": "AllyOnlySelf",
  "attack_type": "Skill"
}
```

## 시전자 주변 범위 궁극기

```json
{
  "type": "RangeEffect",
  "shape": { "Circle": { "radius": 42000 } },
  "target": "Enemy",
  "apply_type": "AroundCaster",
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
```

다음과 함께 사용:

```json
{
  "action_name": "ult",
  "casting_type": "None",
  "casting_target": "Enemy",
  "attack_type": "Skill"
}
```

## 돌진 후 지연 타격

```json
{
  "type": "Combine",
  "effects": [
    {
      "type": "Rush",
      "speed": 3500,
      "range": 50000,
      "casting_target": "Enemy",
      "penetrate": false
    },
    {
      "type": "Delayed",
      "tick": 12,
      "effects": [
        {
          "type": "Attack",
          "damage": 70,
          "attack_ratio": 100
        }
      ]
    }
  ]
}
```

## 무작위 적 번개탄

```json
{
  "type": "RandomTarget",
  "range": 65000,
  "casting_target": "EnemyChampion",
  "from_projectile": false,
  "effects": [
    {
      "type": "ApAttack",
      "damage": 80,
      "attack_ratio": 70
    }
  ]
}
```

## 등록된 시각 효과 발동

```json
{
  "type": "ViewEffect",
  "name": "fire_burst"
}
```

`view_effects`에 해당 시각 효과를 등록:

```json
{
  "view_effects": [
    {
      "type": "Animation",
      "name": "fire_burst",
      "anim": "asset/my_mod/effects/fire_burst",
      "tag": "burst"
    }
  ]
}
```