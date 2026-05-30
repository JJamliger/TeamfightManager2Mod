# 시각 결속

데이터 챔피언 효과는 `name`으로 시각 요소를 참조합니다. `view_effects`, `view_projectiles`, `view_buffs`는 게임 화면용으로 그 이름들을 등록합니다.

## 효과 시각 요소

`ViewEffect`와 `CasterViewEffect`에서 사용됩니다.

### `Animation`

```json
{
  "type": "Animation",
  "name": "fire_burst",
  "anim": "asset/my_mod/effects/fire_burst",
  "tag": "burst",
  "z": 0,
  "is_follow": false
}
```

### `LoopAnimation`

```json
{
  "type": "LoopAnimation",
  "name": "fire_aura",
  "anim": "asset/my_mod/effects/fire_aura",
  "tag": "loop",
  "z": 0,
  "is_follow": true
}
```

항목:

- `name`: `ViewEffect` 또는 `CasterViewEffect`에서 사용하는 효과 이름입니다.
- `anim`: 동작 자산 기본 경로입니다.
- `tag`: `anim#anim` 내부의 동작 태그입니다.
- `z`: 렌더링 깊이 오프셋이며, 기본값은 `0`입니다.
- `is_follow`: 동작이 대상을 따라가는지 여부이며, 기본값은 `false`입니다. 현재 반복 등록은 내부적으로 이 깃발을 무시하지만, 가독성을 위해 명시해 두십시오.

## 투사체 시각 요소

`name`이 결속과 일치하는 투사체 효과에서 사용됩니다.

### `Animated`

```json
{
  "type": "Animated",
  "name": "fire_skill",
  "anim": "asset/my_mod/projectiles/fire_bolt",
  "tag": "fly",
  "z": 0,
  "repeat": true
}
```

### `Sprite`

```json
{
  "type": "Sprite",
  "name": "fire_skill",
  "sprite": "asset/my_mod/projectiles/fire_bolt",
  "z": 0
}
```

### `ThreePhase`

```json
{
  "type": "ThreePhase",
  "name": "fire_orb",
  "anim": "asset/my_mod/projectiles/fire_orb",
  "pre_tag": "spawn",
  "loop_tag": "loop",
  "remove_tag": "remove"
}
```

## 버프 시각 요소

같은 `name`을 가진 버프가 활성화되어 있을 때 사용됩니다.

### `Animated`

```json
{
  "type": "Animated",
  "name": "fire_focus",
  "anim": "asset/my_mod/buffs/fire_focus",
  "tag": "loop",
  "z": 0
}
```

### `ThreePhase`

```json
{
  "type": "ThreePhase",
  "name": "fire_focus",
  "anim": "asset/my_mod/buffs/fire_focus",
  "pre_tag": "on",
  "loop_tag": "loop",
  "remove_tag": "off",
  "z": 0
}
```

## 이름 일치

이름은 반드시 정확히 일치해야 합니다:

```json
{
  "effect": { "type": "ViewEffect", "name": "fire_burst" },
  "view_effects": [
    { "type": "Animation", "name": "fire_burst", "anim": "asset/my_mod/effects/fire", "tag": "burst" }
  ]
}
```

투사체 이름도 같은 방식으로 작동합니다:

```json
{
  "effect": { "type": "LinearProjectile", "name": "fire_skill", "speed": 4200, "range": 65000 },
  "view_projectiles": [
    { "type": "Sprite", "name": "fire_skill", "sprite": "asset/my_mod/projectiles/fire_skill" }
  ]
}
```